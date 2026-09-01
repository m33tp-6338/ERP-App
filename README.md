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

## Known limitations of this feasibility build

- **Single device.** Data lives only in this phone's browser storage. There is no
  sync between devices and no shared/multi-user access yet.
- **Manual backups on mobile.** The app's "auto-save to a connected file" feature
  only works on desktop Chrome/Edge (it depends on a browser API Android doesn't
  support). On the phone, use **Save a Copy Now** periodically and move that file
  somewhere safe (USB to a PC, personal Drive, etc.) — otherwise clearing the
  browser's data or uninstalling the app erases everything.
- **Optional PIN lock** exists under Settings if you want to lock the app on the
  phone — off by default.
- If the app is ever updated (a new `index.html`), bump `CACHE_NAME` in `sw.js`
  (e.g. `v1` → `v2`) so installed phones pick up the new version instead of a
  cached old one, and re-upload everything to the repo.
