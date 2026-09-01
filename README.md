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
   file named `ERP-Data.json` in your Drive's root folder.

**Scope used:** the app only ever requests `drive.file` access — meaning it can
only see/edit files *it* created, never your other Drive files.

**Known limits of browser-only sign-in (no backend server exists to keep you
signed in forever):** occasionally — after a longer stretch away from the app —
the automatic silent reconnect can fail, in which case the Sync Now button (or
just reopening the app) will prompt a quick Google re-sign-in. This is a normal
property of client-side-only OAuth, not a bug.

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
  (currently `v4`) so installed phones pick up the new version instead of a
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
