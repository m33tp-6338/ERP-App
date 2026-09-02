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

## If the app isn't showing recent updates

**This has actually happened, and here's exactly why:** the app was found to be
running on a version from months ago even though `index.html` had been
re-uploaded several times since. The cause was `sw.js` — the file that decides
whether the phone uses a cached copy of the app or fetches the new one. It had
been stuck at its very first version (`v1`) the whole time, because **only
`index.html` (and sometimes `README.md`) was being re-uploaded — `sw.js` itself
was never replaced.** The phone's browser only checks for a new version of the
app when `sw.js` itself changes; if that file is byte-for-byte the same as
before, it never even looks at whether `index.html` changed on the server, so
every update since the first one was silently ignored.

**Going forward, every update package includes three files — upload all
three, every time:** `index.html`, `sw.js`, and `README.md`. It's specifically
`sw.js` that's easy to skip because it looks like plumbing rather than the
actual app — but it's the one file that has to change for anything else to
take effect.

**To check it actually worked, on GitHub:** open your repo → click `sw.js` →
the line near the top should say `const CACHE_NAME = "erp-ledger-cache-vN"`
where `N` matches the version mentioned in that update's message from me. If
you're ever unsure, paste me the repo link and I can check the live file for
you before you go looking on the phone.

**On the phone, after uploading all three files:**
1. Close the app fully (swipe it away from your recent apps — don't just tap
   back or leave it running in the background).
2. Reopen it once while connected to the internet, and give it a few seconds.
3. Still showing the old version? This is the guaranteed fix: in Chrome, go to
   the site (long-press the app icon → App info → or visit the Pages URL in
   Chrome directly) → **⋮ → Site settings → Clear & reset** (or uninstall the
   home-screen icon and use **Add to Home screen** again from Chrome). This
   forces a completely fresh copy, bypassing any caching layer.

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

By default the backup file goes into your Drive's root folder. There are two
ways to move it into a folder you already use instead (for example a "PW"
folder with an "ERP" folder inside it) — a simple one that needs no setup at
all, and Google's own folder-picker dialog, which needs a one-time API key
but then lets you change the folder again from inside the app whenever you
like.

### The simple way (no setup, works right now)

The app always backs up to the same one file, always named **ERP-Data.json**
— it updates that file in place on every sync rather than creating a new one.
That means you can move it yourself, once, using the Drive app or
drive.google.com like any other file, and every future sync will keep
updating it in its new location:

1. Open Google Drive (the app or drive.google.com) and sign in with the same
   account you connected in the app.
2. Find **ERP-Data.json** — it's most likely sitting in your Drive's root
   ("My Drive"), unless you've already moved it.
3. Move it into your PW → ERP folder (drag it there, or use "Move to" /
   "Organize" from its menu).

That's it — nothing to change in the app, and nothing to re-upload. The next
automatic backup (or the next time you tap **Sync now** in Settings) updates
that same file wherever it now sits. The **Backup folder** line in Settings
will still say "My Drive (root)" since the app itself didn't do the move —
that line only updates when the app is the one that placed the file there
(the folder-picker way, below) — but the file's real location is unaffected
either way, so this is a cosmetic difference only.

### The folder-picker way (optional, one-time setup, needs a free API key)

This is the fancier route: a **Choose backup folder** button already exists
under Settings & Backup, but it needs one more free API key from the same
Google Cloud project as above before it will work.

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
  (currently `v19`) so installed phones pick up the new version instead of a
  cached old one — and **re-upload all three of `index.html`, `sw.js`, and
  `README.md` together, every time**, even if only one of them actually
  changed. Uploading just `index.html` leaves the live site serving an old
  `sw.js`, which then keeps installed phones (and the folder-picker/API-key
  fix) stuck on the previous version — this has happened more than once.

## Settings & Backup: smaller Google Drive sync control

The Google Drive backup card in Settings now shows a compact **Sync now**
button next to the last-synced time (instead of the earlier full-width "Sync
Now" button and a longer paragraph of text) — everything else there works the
same. The connected account's email now also shows right next to "Connected"
at the top of the card.

## Ledgers: sharing just the bank details

Open any ledger (Ledgers → tap it) and there's now a **Share Bank Details
Only** button, separate from the existing **Share** button. It shares just
the name, bank, account number and IFSC code — nothing else (no GSTIN, PAN,
address or category) — useful for quickly forwarding payment details to HO
or anyone else who only needs to make the transfer, not see everything else
on file for that ledger. It only appears when a bank account number or IFSC
is actually saved for that ledger.

## Ledgers: fixing entries that landed on the wrong same-named person

If two different real people (or a vendor and a person) share the exact same
name, the app used to have no way to tell their payment histories apart — an
entry recorded against one of them could visually look like it belonged to
both. This is now fixed with a **reassign** flow, and a warning that stops it
happening again:

**Warning when you save a ledger with a name already in use.** When you add
or rename a ledger to a name that another ledger already has, you'll see a
popup: *"A ledger named '...' already exists..."*. If it's genuinely the same
person, cancel and use the existing ledger instead of creating a duplicate.
If it's truly a different person who happens to share the name, add
something to tell them apart before saving — e.g. `Ramesh Kumar (Site B)` or
`Ramesh Kumar (Driver)` — then confirm to save anyway.

**Moving entries off a ledger they don't belong to.** Open the ledger
(Ledgers → tap it). If another ledger exists with the exact same name, you'll
see an amber notice above Transaction History naming how many other ledgers
share that name. To move a wrongly-attributed entry:

1. Tap the small square to the left of the entry (or **Select all** at the
   top of the list) to select the entry or entries that don't actually
   belong on this ledger.
2. A bar appears at the bottom showing how many are selected — tap
   **Options**, then **Reassign to another ledger**.
3. You'll see two ways to move them:
   - **Create a new ledger** for the different person — type a
     distinguishing name (e.g. `Ramesh Kumar (Site B)`) and tap **Create &
     Move**. This creates a brand-new ledger and moves the selected entries
     onto it in one step.
   - **Move to an existing ledger** — search and tap any other ledger already
     in your list (ledgers with the exact same name as the one you're on are
     left out of this list, since that would just recreate the mix-up) to
     move the selected entries there instead.

The payment record itself is never deleted or changed in amount — only which
ledger it's counted against changes. Nothing is lost, and both ledgers'
totals update immediately to reflect the move.

## Cash & Card: each card now has its own page

Tapping a card (or its **Entries →** button) on the Cash & Card screen no
longer just filters the shared list in place — it opens a **dedicated page
for that card only**:

- The page shows just that card's balance, badges (no-bill count, last
  count, negative-balance warning), and its own entries — no other card's
  entries are mixed in.
- **Filter and sort** work exactly as before, scoped to this card: search
  by payee/description/bill no., group by date/category/site/type, sort by
  date/amount/type/paid-to/site, filter by month, and the "No bill" toggle.
- **Select several entries and act on them together** — tap **Select all**
  (or tick individual rows) to bring up the selection bar, then **Options**
  for **Mark bill as on file** or **Delete**. A single entry can still be
  deleted from its own row (trash icon), and tapping a row opens it for
  editing, same as before.
- **Add Entry**, **Count cash**, **Import** and **Export** on this page are
  all pre-scoped to the open card — no need to pick it again.
- **Back to Cash & Card** returns to the card list, where the **All
  cards**/**Card name** chips below the cards still work as a quick filter
  if you'd rather browse everything on one screen without opening a card's
  own page.

Nothing about how entries are stored, matched or balanced changed — this is
purely how you get to and act on a card's entries.

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

## Syncing "Payment to be done", "Master" and "Office Master" from Google Sheets

This moves "Payment to be done", "Master" and "Office Master" into one Excel
workbook the app reads directly — no more entering the same payment twice,
once in the sheet and once in the app. As of v6.14 **all three** can be
synced, each as its own source:

- **"Payment to be done" → Requisitions** — the first place a payment usually
  appears, often still Pending (no Payment Date yet).
- **"Master" → Requisitions** — your month-end record of Head-Office-paid
  (non-cash) payments, once you've copied a row across and it has a Payment
  Date. Syncing it is now **optional, not required**: if you already fill in
  the Payment Date on "Payment to be done" itself before moving the row to
  Master (as you told me you do), that row is already marked Paid in the app
  the moment "Payment to be done" is re-synced — adding Master as a source
  too is a safety net, not a second thing you have to remember to do. Add it
  if you want the extra reassurance; skip it if one source feels like enough.
- **"Office Master" → Expenses** — unchanged from before.

**Why syncing the same row twice (once from "Payment to be done" while it's
still Pending, later from "Master" once it's Paid) is safe:** the app matches
an incoming row against what's already there by date + site + name + amount +
description, not by a reference number. The **first** time a row is seen, it's
added as a new Requisition — Pending if there's no Payment Date yet, Paid if
there is. **Every time after that**, a matching row does one of three things,
never more than one:
1. If the app's existing record is still Pending and the incoming row now has
   a Payment Date, the **existing Requisition is updated to Paid** — it does
   not get added again as a duplicate. (This was a real bug in earlier
   versions — a re-sync after adding a Payment Date used to be silently
   skipped instead of updating anything. Fixed in v6.14 — see the CHANGELOG
   in the app for the exact wording.)
2. If the app's existing record is already Paid (or On Hold), the incoming
   row is just skipped as "already in the app" — nothing changes.
3. If nothing matches yet, it's added as a brand-new Requisition.
   Either way, the sync preview screen (before you tap Import) always shows
   you exactly what it's about to do — how many new rows, how many existing
   ones will be marked Paid, and how many are being skipped — so you can
   check it before committing.

**How the pieces map onto the app:**

- **"Payment to be done"** (Date, Site, Category, Sub-Category, Name, Amount,
  Description, Payment Status, A/c No., IFSC Code) syncs straight into
  **Requisitions**, using the exact same rule already built into the app:
  a blank/"Pending" Payment Status stays a Requisition; a **date** in that
  column sends the row straight to the Payment Register as paid. This is the
  same behaviour Import from Excel already has for a requisition sheet — the
  sync just does it automatically from the live Google Sheet instead of a
  one-off file.
- **No Ref No. column on Payment to be done, deliberately.** Rows on that tab
  get edited or deleted before a payment is finalised, so a stable ID there
  would be misleading — the app just assigns its own Ref No. once a row
  syncs in, same as always. If you want a Ref No. you can trace later, add
  it on **"Master"** instead once that row is settled and copied across (see
  the Master template's Read Me tab) — Office Master rows get their own
  optional **ID** column there for the same reason.
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

**How the sync actually reads your data (as of v6.17):** each source now
points straight at one Excel file in Google Drive — you pick it once,
directly, using Google's own file picker. There's no import folder to
choose first and no file name to type in for the app to go searching for.
Picking a file through the picker is also what grants the app permission to
read that exact file, so there's no more guessing about whether the app can
"see" what's inside a folder — it's pointed straight at the file you chose,
wherever in Drive it lives (subfolders included, since you're picking the
file itself, not browsing for it by name).

*(Earlier versions worked differently — v6.12 through v6.16 used a chosen
import folder plus a typed file name the app searched for, and v6.16 added
searching inside subfolders. All of that is gone as of v6.17; if your setup
predates it, see the note at the bottom of this section.)*

**Setup — you need two or three sync sources (Master is optional, per the
"safety net" note above). No import folder to set up first.**

1. Keep "Payment to be done", "Master" and "Office Master" as tabs in one
   Excel workbook (this is exactly how the Excel template you were given is
   already laid out). Whenever you want to sync, export/save that workbook
   as `.xlsx` to Google Drive — anywhere in your Drive is fine, since you'll
   pick the file directly. **Re-save/replace that same file each time**
   (Google Drive keeps the same file, updated) rather than creating a
   brand-new file each time — see the note below on why that matters.
2. In the app: **Sync from Google Sheets** → **Add Source** → give it a name
   (e.g. "Payment to be done") → tap **Choose file**, sign in to Google if
   asked, and pick your workbook from Drive. Set **Tab inside that file** to
   `Payment to be done`, and **Syncs into** to **Requisitions**. Save.
3. **Add Source** again → tap **Choose file** and pick the *same* workbook
   again, **Tab inside that file** set to `Office Master`, **Syncs into** set
   to **Expenses**. Save.
4. **Optional — Add Source** a third time → pick the same workbook once
   more, **Tab inside that file** set to `Master`, **Syncs into** set to
   **Requisitions** (same option as step 2 — the dropdown reads
   "Requisitions (Payment to be done / Master)" to make this clear). Save.
   Only add this one if you want the extra safety net described above; skip
   it if you're comfortable relying on "Payment to be done" alone.
5. From then on: **save over that same file** in Drive with your updates
   (don't create a new file with a different name), come back to **Sync from
   Google Sheets**, and tap **Sync now** on each card (or use "Sync all ...
   sources" once you have two or more of the same kind). You do **not** need
   to pick the file again after this — re-saving/replacing the same file is
   picked up automatically on every sync.
6. **You only need to tap Choose file again if you create a genuinely new
   file** (a different file, a different name) instead of overwriting the
   old one — for example if you ever start a fresh workbook for a new year.
   In that case, edit the source, tap **Change**, and pick the new file.

**If your workbook is nested a few folders deep** (e.g. a Drive layout like
`PW → ERP → PR`), Google's file picker otherwise opens at the very top of
your whole Drive, forcing you to click through every folder to find it. As
of v6.19, **Sync from Google Sheets** has an optional **"Starting folder"**
card near the top: tap it once, pick the folder your workbook actually
lives in (e.g. `ERP`, or `PR` directly), and every **Choose file**/**Change**
tap after that opens straight there instead of your Drive's root. This is a
convenience, not a hard restriction — Google's picker still lets you browse
elsewhere in Drive from inside it if you ever need to; picking the file
itself is still what grants the app access, exactly as before. Tap **Clear**
on that same card to go back to browsing your whole Drive from the top.

**Why "re-save the same file" matters:** picking a file through the picker
gives the app permission for that one specific file — not a rule like "any
file in this folder" or "any file with this name." Overwriting/re-saving the
same file in Drive keeps that same file, so the app's permission still
applies and syncing keeps working with no extra steps. Creating a brand-new
file (even with the exact same name) is, to Google Drive, a different file
that the app has never been given permission to see — that's the one case
where you'd need to pick again.

**If you set this up before v6.17** (the folder + typed-file-name version):
your sources keep their name, tab, and "Syncs into" settings, but each one
needs its file picked once — open **Sync from Google Sheets**, and any
source without a file shows a clear **"No file chosen yet"** warning instead
of a sync error. Tap the pencil to edit it, tap **Choose file**, pick your
workbook, and Save — then it works exactly as described above from then on.

**As of v6.18, syncing also happens automatically.** Every source with a
file already picked is checked quietly the moment you open the app — you no
longer have to open Sync from Google Sheets and tap Sync now yourself every
time. If it finds anything new, a green banner appears on the Overview
screen ("Synced from Google Sheets — N rows ready to review") — tap it to
see the exact same preview screen described above and decide whether to
Import. **Nothing is ever added automatically** — the preview and the
Import tap are still required every time, this just saves you the trip to
go fetch the data in the first place. If nothing new is found, or Drive
isn't reachable at that moment, the app stays silent about it — no error
popups from a background check you didn't ask for; a source's own **Check
status** card still shows the real reason if something's actually wrong
with it. It also re-checks after the app has been sitting in the background
for a while and you bring it back to the front (at most once every 20
minutes), so leaving the app open overnight still catches a fresh sync the
next time you look at it.

## Optional: one-click move between "Payment to be done" and "Master"

Once a row is paid, you can move it out of "Payment to be done" and into
"Master" with one click, instead of copying it across by hand — and send a
row back the other way if a settled payment needs correcting. This lives
entirely inside the Google Sheet (a small menu Apps Script adds), separate
from the app's own sync.

**Why this isn't automatic (no button, just happens the moment you type a
payment date):** the app only reads "Payment to be done" when you tap Sync.
If a row got moved out to Master the instant you typed the date — before
you'd had a chance to sync — the app would never see it, and that payment
would silently never show up in Requisitions/Payment Register at all. So the
move only ever happens when **you** trigger it, which should be **after**
you've already synced that batch of payments into the app.

**This menu is completely separate from how the app syncs** — the app (since
v6.12) reads your Excel file straight out of a Google Drive folder, it does
not call anything in this Google Sheet or its Apps Script. So this setup
needs no deployment, no Web app URL, and nothing to connect to the app —
it's purely a convenience menu inside the sheet itself.

**Setup (one-time, or whenever you update the script below):**

1. Open the Google Sheet, **Extensions → Apps Script**. (If you still have
   an older project here from before v6.12's sync change, that's fine — this
   replaces its contents; nothing else depends on what was there before.)
2. Select everything in the code editor, delete it, and paste in the
   script below.
3. Save (Ctrl+S / Cmd+S). No deployment step, no "Anyone" access setting —
   this only ever runs from inside the sheet itself, for whoever has it open.
4. Close and reopen the Google Sheet in your browser (or just refresh the
   page). You should see a new **ERP Tools** menu appear next to Help.
   First time only: it may ask you to authorize again — click through
   **Advanced → Go to (project) → Allow**.

```javascript
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu("ERP Tools")
    .addItem("Move paid rows to Master", "movePaidRowsToMaster")
    .addItem("Send selected row(s) back to Payment to be done", "sendSelectedRowsBack")
    .addToUi();
}

function looksPaid_(val) {
  if (val instanceof Date) return true;
  if (typeof val !== "string") return false;
  var s = val.trim();
  if (!s) return false;
  if (/^(done|paid|yes|y|complete|completed|settled)$/i.test(s)) return true;
  return /^\d{1,4}[-\/]\d{1,2}[-\/]\d{1,4}$/.test(s);
}

function movePaidRowsToMaster() {
  var ui = SpreadsheetApp.getUi();
  var resp = ui.alert(
    "Move paid rows to Master",
    "Have you already opened the app and tapped Sync since entering these payments? " +
    "If not, cancel this and sync first — otherwise a payment could be moved out " +
    "of \"Payment to be done\" before the app ever gets to read it.",
    ui.ButtonSet.YES_NO
  );
  if (resp !== ui.Button.YES) return;

  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var src = ss.getSheetByName("Payment to be done");
  var dst = ss.getSheetByName("Master");
  if (!src || !dst) {
    ui.alert("Could not find both \"Payment to be done\" and \"Master\" tabs — check the tab names match exactly.");
    return;
  }
  var data = src.getDataRange().getValues();
  if (data.length < 2) { ui.alert("No rows to move."); return; }
  var headers = data[0];
  var col = {};
  headers.forEach(function (h, i) { col[String(h).trim()] = i; });
  var required = ["Date", "Site", "Category", "Sub-Category", "Name", "Amount", "Description", "Payment Status"];
  for (var r = 0; r < required.length; r++) {
    if (!(required[r] in col)) {
      ui.alert("\"Payment to be done\" is missing the \"" + required[r] + "\" column — check the header row hasn't been changed.");
      return;
    }
  }
  var rowsToDelete = [];
  var moved = 0;
  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    if (row.join("").toString().trim() === "") continue;
    var status = row[col["Payment Status"]];
    if (!looksPaid_(status)) continue;
    dst.appendRow([
      "",
      row[col["Date"]],
      row[col["Site"]],
      row[col["Category"]],
      row[col["Sub-Category"]],
      row[col["Name"]],
      row[col["Amount"]],
      row[col["Description"]],
      status
    ]);
    rowsToDelete.push(i + 1);
    moved++;
  }
  rowsToDelete.sort(function (a, b) { return b - a; });
  rowsToDelete.forEach(function (rowNum) { src.deleteRow(rowNum); });
  ui.alert(moved + " paid row(s) moved to Master." +
    (moved > 0 ? " Ref No. was left blank — copy it in from Requisitions → Export to Excel if you want one." : ""));
}

function sendSelectedRowsBack() {
  var ui = SpreadsheetApp.getUi();
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var master = ss.getSheetByName("Master");
  var dst = ss.getSheetByName("Payment to be done");
  if (!master || !dst) {
    ui.alert("Could not find both \"Master\" and \"Payment to be done\" tabs.");
    return;
  }
  var sel = ss.getActiveRange();
  if (!sel || sel.getSheet().getName() !== "Master") {
    ui.alert("Select the row(s) on the Master tab you want to send back first, then run this again.");
    return;
  }
  var startRow = sel.getRow();
  if (startRow === 1) {
    ui.alert("That selection includes the header row — select only the data row(s) you want to move.");
    return;
  }
  var resp = ui.alert(
    "Send back to Payment to be done",
    "This moves the selected row(s) out of Master and into Payment to be done for editing. " +
    "If this payment is already showing as paid in the app, this will NOT change that — " +
    "fix it directly in the app's Requisitions tab instead. Continue?",
    ui.ButtonSet.YES_NO
  );
  if (resp !== ui.Button.YES) return;

  var numRows = sel.getNumRows();
  var values = master.getRange(startRow, 1, numRows, master.getLastColumn()).getValues();
  var headers = master.getRange(1, 1, 1, master.getLastColumn()).getValues()[0];
  var col = {};
  headers.forEach(function (h, i) { col[String(h).trim()] = i; });

  var moved = 0;
  values.forEach(function (row) {
    if (row.join("").toString().trim() === "") return;
    dst.appendRow([
      row[col["Date"]],
      row[col["Site"]],
      row[col["Category"]],
      row[col["Sub-Category"]],
      row[col["Name"]],
      "",
      row[col["Amount"]],
      row[col["Description"]],
      "",
      "",
      ""
    ]);
    moved++;
  });
  master.deleteRows(startRow, numRows);
  ui.alert(moved + " row(s) sent back to \"Payment to be done\". Payment Status was cleared, so it shows as Pending again until you re-enter the payment date.");
}
```

**Using it:**

- **Move paid rows to Master:** ERP Tools → **Move paid rows to Master**.
  Confirm you've already synced, and it moves every row in "Payment to be
  done" that has a Payment Status/date filled in over to Master, removing it
  from "Payment to be done" — no more of that copy-paste at month end. Ref
  No. is left blank on the moved row, same as a manual copy would be — fill
  it in from Requisitions → Export to Excel if you want one (see the Excel
  template's Read Me tab).
- **Send a row back for correction:** click the row number(s) on **Master**
  to select them, then ERP Tools → **Send selected row(s) back to Payment to
  be done**. Payment Status is cleared, so it comes back as an ordinary
  Pending row ready to edit and re-enter a payment date for later.
- **Important limitation:** if that payment was already synced into the app
  as a paid Requisition *before* you sent it back, this does **not** touch
  that existing app record — re-syncing the corrected row will either be
  silently skipped (if it still looks like the same payment) or come in as a
  second, duplicate entry (if you changed the amount/date/description).
  Fix an already-synced payment directly in the app's own Requisitions tab
  instead — editing an existing Requisition there already works today and is
  the reliable way to correct something that's already made it into the app.

## Payroll: fixed salaries and attendance-based pay

A new **Payroll** tab handles the two different ways staff get paid, instead
of typing each month's salary requisition by hand:

- **Fixed** — the employee gets their full **Monthly Salary** every month,
  no matter what (typically top management / anyone on a flat salary).
- **Attendance-based** — the employee's pay is different every month,
  worked out from how many days they were actually present:

  ```
  that month's pay = Monthly Salary ÷ (actual number of days in that month) × days present
  ```

  So the same ₹31,000 Monthly Salary works out to a different daily rate in
  a 31-day August than in a 30-day September — the app always uses the real
  number of days in the specific month being paid, not a fixed 26 or 30.

**Set each employee's Pay Type once.** Open Ledgers/Employees → edit the
employee → there's now a **Pay Type** field right under Roll Type, defaulting
to **Fixed**. Set it to **Attendance-based** for anyone whose pay depends on
attendance. (Existing employees you haven't touched default to Fixed, so
nobody's pay changes on its own after this update — go through your
Attendance-based staff once and switch them over.)

**Get that month's attendance into the app.** Payroll only reads attendance
you've imported — it does not read attendance from anywhere else. From the
Payroll tab, tap **Import attendance from Excel**, and fill in a simple sheet
with one row per employee per month: **Month**, **Employee** (must match an
existing employee ledger's name exactly), and **Days Present**. The Month
column accepts pretty much any reasonable way of writing it — `August 2026`,
`Aug-26`, `08/2026`, `2026-08`, an Excel date, and so on — the app reads it
correctly either way, and rejects (rather than guesses at) anything it can't
make sense of. Re-importing the same employee and month again **corrects**
that record instead of creating a duplicate, so fixing a mis-typed
attendance figure is just a re-import. An import row is also rejected, with
a clear reason, if the named employee doesn't exist yet as a ledger, or if
the days present is more than that month actually has.

**Run payroll.** On the Payroll tab, pick the month (defaults to the current
one). Fixed employees and Attendance-based employees are listed separately,
each showing what they're due to be paid this month. Anyone with no Monthly
Salary set on their ledger, or (for Attendance-based staff) no attendance
imported yet for that month, is clearly flagged and skipped — fix the ledger
or import the attendance, and they'll pick up automatically. Tap **Generate**
on one employee, or **Generate this month's payroll** to raise a Pending
Requisition for everyone who's ready at once — each one is tagged with the
month it's for, so trying to generate the same employee's pay again for a
month already generated is blocked, showing the reference number of the one
already raised instead of risking a double payment. Generated requisitions
show up exactly like any other Pending Requisition — Export, mark paid, etc.
all work the same as before.

**How this interacts with Google Sheets sync (see the section above):** if
you're also syncing "Payment to be done" from Google Sheets and still typing
salary rows into that sheet by hand (Category = "Salary"), those come in as
ordinary Requisitions and are **not** linked to Payroll's month-tracking, so
Payroll won't know they've already been paid and won't block a duplicate for
that employee/month. To avoid paying someone twice, pick one path per
employee: either generate their salary from the Payroll tab each month, or
keep entering it by hand in the sheet — not both for the same person.

**One thing worth knowing:** the Fixed Cost Summary / Site Summary's
"Salaries & wages" figure still adds up every active employee's flat
Monthly Salary figure, same as before this update — including
Attendance-based staff, at their full salary. Treat that figure as a
**budgeted ceiling assuming full attendance**, not the actual amount paid
out that month; the real, attendance-adjusted total is what the Payroll tab
shows for the month you've selected there.
