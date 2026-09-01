# Business Ledger ERP — Offline App

This is the same `ERPApp2.html` app, packaged so it can be installed as an offline app
on a phone with a real home-screen icon, instead of only running as a browser tab.

## What changed from the original file

- `index.html` — identical app, except the four library `<script>` tags now point to
  local copies in `vendor/` instead of `unpkg.com` / `cdnjs.cloudflare.com`, and a
  service worker registration was added at the bottom. No application logic was touched.
- `vendor/` — exact copies of React 18.3.1, ReactDOM 18.3.1, Babel Standalone 7.29.8,
  and SheetJS xlsx 0.18.5 (the same versions the app was already loading from the CDN).
- `manifest.json` + `icons/` — makes the app installable ("Add to Home screen") with
  its own icon, using the app's existing ₹ mark.
- `sw.js` — a service worker that caches the app shell on first visit so it keeps
  working with zero internet connection afterwards.

**No business data is included in this repository.** The app starts empty. Data is
loaded on-device after install, using the app's own "Restore From a Saved File"
button under Settings & Backup — nothing is ever uploaded anywhere.

## Hosting on GitHub Pages

1. Repo must be **public** (free GitHub Pages can't serve a private repo).
2. Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save.
3. Wait 1–2 minutes, then the app is live at:
   `https://<your-username>.github.io/<repo-name>/`

## Installing on Android

1. Open the Pages URL above in Chrome on the phone (needs internet once, to download
   the app — about 4 MB total).
2. Chrome's menu (⋮) → **Install app** (or **Add to Home screen**).
3. Open the app from the home screen icon from now on.
4. Get today's data file onto the phone (USB transfer from the PC is the safest way —
   this keeps it off the internet entirely), then inside the app: ☰ / **More** →
   **Settings & Backup** → **Restore From a Saved File** → pick the file.
5. Turn on Airplane Mode and reopen the app to confirm it still works — it should.

## Google Drive backup setup (one-time, ~15–20 minutes)

The app can now back itself up to your own Google Drive automatically after each
change — but it needs its own registration with Google first (a "Client ID"),
which only you can create, using your own Google account. This is free and does
not need a credit card. The Client ID is not a secret — it's normal for it to sit
in plain view inside `index.html`, that's how Google's client-side sign-in is
designed to work (unlike a password, nobody can do anything with just this ID).

1. Go to **console.cloud.google.com/projectcreate**, sign in with your Google
   account, give the project a name (e.g. "ERP Ledger"), and create it. Skip
   anything about billing — not needed for this.
2. Make sure the new project is selected (top-left project switcher).
3. Go to **console.cloud.google.com/auth/branding** (this is the "OAuth consent
   screen" / "Google Auth Platform" setup) → **Get started**:
   - App name: anything, e.g. "Business Ledger ERP"
   - Support email: your email
   - Audience: **External**
   - Contact email: your email
   - Accept the policy → **Create**
4. Go to **console.cloud.google.com/auth/audience** → **Add users** (under Test
   users) → add your own Google account's email. (While the app is in "Testing"
   mode — which is fine forever for personal use — only test users you list here
   can sign in.)
5. Go to **console.cloud.google.com/auth/clients/create**:
   - Application type: **Web application**
   - Name: anything, e.g. "ERP Ledger Web"
   - **Authorized JavaScript origins** → **Add URI** → enter your GitHub Pages
     address exactly, with no trailing slash, e.g.
     `https://m33tp-6338.github.io`
   - Leave "Authorized redirect URIs" empty — not needed here.
   - Click **Create**.
6. A **Client ID** appears, looking like
   `123456789-abc...xyz.apps.googleusercontent.com`. Copy it.
7. Paste that Client ID into `index.html`, replacing the placeholder on this line
   near the top of the file:
   ```js
   const GOOGLE_DRIVE_CLIENT_ID = "PASTE_YOUR_GOOGLE_CLIENT_ID_HERE.apps.googleusercontent.com";
   ```
8. Re-upload `index.html` to the repo (same "Add file → Upload files" as always),
   wait for Pages to redeploy, then open the app on the phone once online.
9. Settings & Backup → **Connect Google Drive** → sign in with the same Google
   account you added as a test user → approve access. From then on, changes
   back up automatically about 10 seconds after you stop typing/editing, to a
   file named `ERP-Data.json` — in your Drive's root folder unless you point it
   at a specific folder using the next section.

**Scope used:** the app only ever requests `drive.file` access — meaning it can
only see/edit files *it* created, never your other Drive files. This stays true
even when you pick a specific existing folder below (see how that works there).

**Known limits of browser-only sign-in (no backend server exists to keep you
signed in forever):** occasionally — after a longer stretch away from the app —
the automatic silent reconnect can fail, in which case the Sync Now button (or
just reopening the app) will prompt a quick Google re-sign-in. This is a normal
property of client-side-only OAuth, not a bug. If it keeps failing repeatedly
with the exact same message, tapping **Sync Now** once forces a completely
fresh sign-in check rather than reusing whatever failed before.

## Saving the backup into a specific Drive folder (optional, one-time)

By default the backup file goes into your Drive's root folder. To have it
land inside a folder you already use instead (for example a "PW" folder with
an "ERP" folder inside it), the app needs Google's own folder-browser dialog —
this needs one more free API key from the same Google Cloud project as above.

1. Go to **console.cloud.google.com/apis/library**, make sure the same project
   from the steps above is selected (top-left), search for **Google Picker
   API**, open it, and click **Enable**.
2. Go to **console.cloud.google.com/apis/credentials** → **Create Credentials**
   → **API key**. A key appears immediately — copy it.
3. (Recommended, not required) Click the new key to restrict it: under
   **Application restrictions** choose **Websites**, add your GitHub Pages
   address with `/*` on the end, e.g. `https://m33tp-6338.github.io/*`; under
   **API restrictions** choose **Restrict key** and tick only **Google Picker
   API**. Save.
4. Paste that key into `index.html`, replacing the placeholder near the top:
   ```js
   const GOOGLE_DRIVE_PICKER_API_KEY = "PASTE_YOUR_GOOGLE_PICKER_API_KEY_HERE";
   ```
5. Re-upload `index.html`, wait for Pages to redeploy, open the app once online.
6. Settings & Backup → **Choose backup folder** → Google's own folder browser
   opens → navigate into your folder (e.g. PW → ERP) → select it → Open/Select.
   The app remembers this folder from then on, and if a backup file already
   existed elsewhere it is moved into the newly chosen folder automatically.

**Why this needs its own dialog instead of the app just searching your Drive
for the folder by name:** the app's narrow `drive.file` permission (see above)
means it normally can't see any file or folder it didn't create itself — that
is deliberate, for your privacy. Google's own picker dialog is the standard,
safe way around that: it runs as Google's own UI (not this app's code), so it
can show you your real folders, and the moment you pick one, only *that one
folder* becomes visible to the app — everything else in your Drive stays
exactly as invisible to it as before.

## Known limitations of this feasibility build

- **Single device for data entry.** There's no shared/multi-user access — this
  is one person's ledger, syncing to one person's Drive.
- **Manual backups on mobile without Drive connected.** The app's "auto-save to
  a connected file" feature only works on desktop Chrome/Edge (a browser API
  Android doesn't support) — use **Save a Copy Now** on the phone if you haven't
  set up Google Drive backup above.
- **Optional PIN lock** exists under Settings if you want to lock the app on the
  phone — off by default.
- If the app is ever updated (a new `index.html`), bump `CACHE_NAME` in `sw.js`
  (currently `v6`) so installed phones pick up the new version instead of a
  cached old one, and re-upload everything to the repo.

## Importing Cash & Card transactions for a specific card

If you use more than one card/cash account, an imported file's rows need to
know which card they belong to:

- If your Excel/CSV file already has a **Card** column filled in on every
  row, nothing else is needed — each row goes to the card named in that row
  (a new card is created automatically if the name doesn't match one you
  already have).
- If the file has no Card column, or some rows leave it blank: open
  **Cash & Card → tap the card → Import transactions for this card** (this
  preselects that card for every blank row), or use the regular **Import**
  button and pick the card from the new **"Which card is this for?"**
  dropdown on the column-matching screen. With two or more cards saved, this
  choice is required — the app blocks the import until it's set, so a file
  can never land silently on the wrong card.

## Syncing "Payment to be done" and "Office Master" from Google Sheets

This replaces the two-file Excel workflow (Payment to be done + Master, with
Office Master as a separate backing sheet) with one Google Sheet the app reads
directly — no more entering the same payment twice, once in the sheet and once
in the app.

**How the pieces map onto the app:**

- **"Master" goes away.** It doesn't need its own sheet or tab any more — the
  app's own **Payment Register** is already exactly that: any Requisition
  whose Payment Status holds a date (not "Pending") shows there automatically,
  Account/IFSC included in the underlying record but not surfaced in that
  view. Keep "Payment to be done" as your one working sheet; stop maintaining
  Master by hand.
- **"Payment to be done"** (Date, Site, Category, Sub-Category, Name, Amount,
  Description, Payment Status, A/c No., IFSC Code) syncs straight into
  **Requisitions**, using the exact same rule already built into the app:
  a blank/"Pending" Payment Status stays a Requisition; a **date** in that
  column sends the row straight to the Payment Register as paid. This is the
  same behaviour Import from Excel already has for a requisition sheet — the
  sync just does it automatically from the live Google Sheet instead of a
  one-off file.
- **Site expenses paid by your partner on their card** (electricity, etc.):
  add a **"Paid By"** column to Payment to be done. Leave it blank (or write
  "Company") for normal payments; write the partner's name for anything they
  paid on the company's behalf. The app already treats "Paid By" anyone other
  than Company as **partner-paid** and tracks it as a recoverable/receivable
  amount from that partner automatically (Receivables tab) — nothing new to
  build, this column is all that's needed.
- **House/Land/Machinery rent and instalments:** if a paid row's **Name**
  matches an existing, active Rent & EMI item exactly (same spelling), syncing
  it also settles that Rent & EMI item automatically — advances its next due
  date, and counts an instalment if it's an EMI/instalment type — the same
  thing that happens when you generate and pay it by hand from Rent & EMI.
  If the name doesn't match anything (a brand-new lease, a typo, or the name
  matches more than one item), the row still comes in as a normal paid
  Requisition — it's just not linked, and you can set up or fix the matching
  Rent & EMI item separately. **The name has to match exactly**, so keep the
  spelling in your sheet consistent with what's saved in Rent & EMI.
- **Office Master** (the travel/office expense bifurcation) syncs into the
  **Expenses** tab as its own sheet source, with each row becoming one
  Expense entry (Date, Expense Type, Paid By, Amount, Description, Site) —
  giving you the full breakdown in-app, not just a lump total. Keep recording
  the monthly *total* of these as one ordinary line in Payment to be done (so
  it still becomes a paid Requisition/Payment Register entry), while the
  detail lives in Expenses via this second sync.
- **Salary, Site Purchases, Installments, Service/Software Purchase** need no
  special handling — they're just different values in the Category column of
  ordinary Payment to be done rows.

**Setup — you need two sync sources, both pointing at the same spreadsheet:**

1. Move "Payment to be done" and "Office Master" into one Google Sheet (as two
   tabs/sheets within it — drop "Master", it's no longer needed).
2. In that Google Sheet: **Extensions → Apps Script**, paste in the script
   from **Sync from Google Sheets → Set up automatic sync (advanced)** in the
   app (see the note below — use this simple path, not the multi-site folder
   wizard on that same screen), then **Deploy → New deployment → Web app**,
   execute as **Me**, access **Anyone**, and copy the Web app URL it gives you
   (ends in `/exec`).
3. In the app: **Sync from Google Sheets → Add Folder** → give it a name,
   paste that same Web app URL, set **Tab name** to `Payment to be done`, and
   set **Syncs into** to **Requisitions**. Save.
4. **Add Folder** again → same Web app URL, **Tab name** set to
   `Office Master`, **Syncs into** set to **Expenses**. Save.
5. From then on, tap **Sync now** on either card whenever you've added new
   rows — it only pulls in rows it hasn't seen before (matched by date, site,
   name and amount together, not by the sheet having its own reference
   number, so re-syncing the same sheet repeatedly is safe).

**A known issue on that same "Sync from Google Sheets" screen, found while
building this — not something you did wrong:** the "Set up automatic sync
(advanced) → Step 1 — name each site and paste its folder" wizard (for
pulling many *separate* site workbooks out of Drive folders automatically)
generates a script that doesn't actually use the folder names/links you type
in — a pre-existing gap in the app, unrelated to this update. You don't need
that wizard for this workflow: the plain single-spreadsheet setup above (copy
the script once, paste it as-is, no folders to fill in) works correctly today
and is exactly what steps 1–5 use. If multi-site folder scanning is ever
needed, that's a separate fix.
