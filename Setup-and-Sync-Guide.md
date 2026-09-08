# Business Ledger ERP — Setup & Sync Guide (v6.25)

A plain-language walkthrough for setting up the Excel-linking (Sync from
Google Sheets) from scratch on your PC, and using it day to day. Written
assuming you've never done this before — nothing here assumes prior
technical knowledge.

---

## Part 1 — What this actually does

Right now you enter each payment once in your Excel sheet ("Payment to be
done") and, separately, once in the app. Syncing removes the second step:
you keep entering payments in Excel like you already do, and the app reads
them straight out of the Excel file you pick in Google Drive, whenever you
tap **Sync now**.

**As of v6.17:** you pick that Excel file directly, once, using Google's own
file picker — there's no folder to choose first and no file name to type in
for the app to search for. This replaces the older folder + typed-file-name
setup (v6.12–v6.16). If you set this up before v6.17, see the note at the
end of Part 3.

Three tabs in your workbook can each be linked, one at a time:

| Your Excel tab | Lands in the app as | Required? |
|---|---|---|
| **Payment to be done** | Requisitions (Pending until a Payment Date is filled in, then Paid → shows in Payment Register) | Yes — this is the main one |
| **Master** | Requisitions (same as above — it recognises a row already brought in from Payment to be done and marks it Paid, instead of adding it twice) | Optional — a safety net |
| **Office Master** | Expenses | Yes, if you want travel/office expenses to show individually in the app |

You do **not** need to change how you fill in Excel. The three tabs, their
column headings, and your day-to-day habit of typing payments into "Payment
to be done" first all stay exactly as they are.

---

## Part 2 — One-time setup on your PC

### Step 1 — Confirm your Excel workbook is laid out correctly

Open your workbook and check it has these tabs, with these exact column
headings in the first row of each (the app matches columns by their
heading, so exact spelling matters more than the order they're in):

**Payment to be done:**
`Date`, `Site`, `Category`, `Sub-Category`, `Name`, `Amount`, `Description`,
`Payment Status`, `A/c No.`, `IFSC Code` — and optionally `Paid By` (leave
blank, or write "Company", unless a partner paid on the company's behalf).

**Master:**
`Date`, `Site`, `Category`, `Sub-Category`, `Name`, `Amount`, `Description`,
`Payment Date`, `Ref No.`

**Office Master:**
`Date`, `Expense Type`, `Paid By`, `Amount`, `Description`, `Site` (and
optionally `ID`).

If any of these are missing or spelled differently, fix the heading row
before continuing — everything downstream depends on it.

### Step 2 — Get your workbook into Google Drive, as either a Sheet or an .xlsx

**As of v6.21, you have two options — pick whichever matches how you
actually work, no need to change your habit:**

**Option A — you keep this as a live Google Sheet (edited directly in
Sheets, on phone or PC) and never download it.** As of v6.21 the app can
read this directly — there is nothing to export. Just keep the Sheet
updated in Google Sheets the way you already do, and in Part 3 you'll pick
the Sheet itself, not a downloaded copy. *(Before v6.21, the app could only
read a real .xlsx file, so this path didn't work — if you're updating from
an older version and were manually exporting to .xlsx as a workaround, you
can stop doing that from now on and switch to picking the Sheet directly.)*

**Option B — you maintain the workbook in real Excel (on a PC, or as an
uploaded .xlsx in Drive) rather than in Google Sheets.** This still works
exactly as before:

1. Save/export your workbook as an actual `.xlsx` file (in Excel: **File →
   Save As → .xlsx**; from Google Sheets instead: **File → Download →
   Microsoft Excel (.xlsx)**, or on phone: **⋮ → Share & export → Save as →
   Excel (.xlsx)**).
2. Upload/save that `.xlsx` file to Google Drive — anywhere in your Drive is
   fine, since you'll pick the file itself rather than a folder.
3. Each time you want fresh data in the app: **save over that same file**
   (replace it, don't create a new file with a different name) and drop the
   updated version in the same place in Drive. Re-saving the same file is
   what lets the app keep reading it automatically — see the note at the end
   of this part for why that matters.

One thing to know either way: Google caps how large a file it will convert
behind the scenes at 10MB — far more than a normal ledger workbook needs,
but worth knowing if yours is unusually large and something fails
unexpectedly.

### Step 3 — Update the app itself

1. On your Android phone, open the ERP app.
2. If it doesn't already show version **6.22** on the **What is New** page
   (menu → What is New), you need to re-upload the files from
   `ERP-App-Update-v22.zip` to your GitHub repository, replacing the
   existing `index.html`, `sw.js`, `README.md`, `manifest.json`, and the
   `vendor/` and `icons/` folders exactly as before. Then close the app
   fully and reopen it once you have an internet connection, so it picks up
   the new version.

---

## Part 3 — Connecting Google Drive and adding your three sources

Do this once. Open the app menu → **Sync from Google Sheets**. There's no
import folder to choose first — you'll pick your Excel file directly, once
per source.

**If your workbook lives a few folders deep** (e.g. `PW → ERP → PR`), do
this first so the file picker doesn't dump you at the top of your entire
Drive: tap **Set a starting folder** near the top of the screen, pick the
folder your workbook actually sits in (e.g. `ERP`, or `PR` itself), and
every **Choose file** tap from then on opens right there. You can change or
clear it any time from that same card — it's just a shortcut for where the
picker opens, not a restriction on what you can pick.

### Step 1 — Add "Payment to be done" as a source

Tap **Add Source** and fill in:

- **Name:** `Payment to be done` (just a label for you)
- **Excel file or Google Sheet:** tap **Choose file**. Sign in with your
  Google account if asked (approve the permission it asks for — this only
  lets the app read the one file you pick, nothing else in your Drive). Pick
  your workbook from Part 2, Step 2 — either the Google Sheet itself
  (Option A) or the uploaded .xlsx (Option B), whichever you set up.
- **Tab inside that file:** `Payment to be done`
- **Syncs into:** Requisitions (Payment to be done / Master)

Tap **Save Source**.

### Step 2 — Add "Office Master" as a source

Tap **Add Source** again:

- **Name:** `Office Master`
- **Excel file:** tap **Choose file** and pick the **same workbook** again
  (it's one file with multiple tabs — you pick it separately for each
  source, but it's the same file each time).
- **Tab inside that file:** `Office Master`
- **Syncs into:** Expenses

Tap **Save Source**.

### Step 3 — Optional: add "Master" as a third source

Only do this if you want the extra safety net described in Part 1. If you
always fill in the Payment Date on "Payment to be done" itself before
moving a row to Master (which is what you told me you do), this step is
optional — you're already covered without it. Add it anyway if you'd rather
have the belt-and-braces version:

- **Name:** `Master`
- **Excel file:** tap **Choose file** and pick the same workbook again
- **Tab inside that file:** `Master`
- **Syncs into:** Requisitions (Payment to be done / Master)

Tap **Save Source**.

### Step 4 — Check each source works

For each source card, tap **Check status**. It should turn green
("Active") and tell you how many rows it can see. If it turns red
("Not working"), the card will tell you exactly why — almost always the tab
name is spelled differently than in the app, or (if you're updating an
older setup) no file has been picked yet. Fix the source (tap the pencil
icon) and check again.

**If a source shows "File not found" on a Google Sheet you're sure is
correct:** as of v6.22, this can happen for a source whose file was picked
*before* this update, especially a Sheet reached through a shared company
folder or a shortcut. Edit that source (pencil icon), tap **Change**, and
pick the same file again — this refreshes some extra information the fix
needs that older picks don't have. You only need to do this once per
affected source; sources that already show green don't need anything.

### A note on why re-saving the same file matters

Picking a file through **Choose file** gives the app permission to read
that one specific file — not "anything in this folder" or "anything with
this name." As long as you keep **saving over that same file** in Drive
(replacing its contents, not creating a new file), that permission still
applies and every sync afterward just works, with no need to pick again.
The only time you'd need to tap **Change** and pick again is if you
genuinely start a new, different file — for example a fresh workbook for a
new year — because Drive treats that as a brand-new file the app hasn't
been given permission to see yet, even if you gave it the exact same name.

**If you set this up before v6.17** (the folder + typed-file-name version):
your sources keep their name, tab, and "Syncs into" settings, but each one
will show **"No file chosen yet"** until you edit it, tap **Choose file**,
and pick your workbook — then it's fully switched over.

---

## Part 4 — Using it day to day

**As of v6.18, you mostly don't need to do step 3 below by hand anymore.**
Every source with a file already picked is checked automatically the
moment you open the app — no tapping into Sync from Google Sheets required.
If it finds anything new, a green banner shows up right on the Overview
screen ("Synced from Google Sheets — N rows ready to review") — tap it and
you land straight on the same preview screen described in step 4. Nothing
is ever imported without you tapping **Import** yourself; the automatic
part only saves you the trip to go fetch the data. You can still tap
**Sync now** yourself any time — useful right after you've just updated the
sheet and don't want to wait for the next automatic check.

1. Keep entering payments in **Payment to be done** exactly as you do now.
   Leave **Payment Status** blank until the payment is actually made.
2. When a payment is made, fill in the date in **Payment Status** (or your
   dedicated Payment Date column) as you already do.
3. If you're on **Option A** (a live Google Sheet), there's nothing extra to
   do here — it's already saved. If you're on **Option B** (uploaded
   `.xlsx`), **save over that same file** in Drive (Part 2, Step 2) — don't
   create a new file. From here, either wait for the app to pick it up
   automatically next time you open it, or go to **Sync from Google
   Sheets** yourself and tap **Sync now** on each source card (or **Sync
   all Requisition sources** if you've set up more than one) to check
   right away.
4. Before anything is actually added, you'll see a **preview screen**:
   - **New requisitions / expenses** — brand-new rows about to be added.
   - **Existing rows to mark Paid** — rows already in the app that will be
     switched from Pending to Paid because a Payment Date has now appeared
     for them. This is the part that used to silently do nothing — it's
     fixed now, and this section is exactly where you can double-check it
     before committing.
   - **Skipped as duplicates** — rows already correctly reflected in the
     app; nothing needs to happen to them.
   - **Rows with problems** — a row missing something required; fix it in
     Excel and sync again.
   Review it, then tap **Import** to apply it.
5. Check **Payment Register** and **Expenses** in the app to confirm what
   you expected to see actually landed.

You can still use **Import from Excel** any time as a one-off, manual
alternative — syncing is a convenience on top of it, not a replacement.

---

## Part 5 — What the status dot in the top-right now means

As requested, it's a plain coloured dot now — no text.

| Colour | Meaning |
|---|---|
| 🟢 Green | Saved / synced with Google Drive, nothing to worry about |
| 🟠 Amber | Not backed up yet, or backup is getting old — worth checking |
| 🔴 Red | Something needs your attention (reconnect needed, or a Drive sync problem) |
| Spinning | Saving right now |

Tap it any time to open **Settings & Backup**, which shows the full detail
in writing. On a PC/larger screen you can also hover over it for a moment
to see the same detail as a tooltip.

---

## Part 6 — Syncing Cash & Card or Diesel & Machinery too (v6.25)

Everything above was written for your three Requisitions/Expenses
sources. As of v6.25, the same **Sync from Google Sheets** screen can also
sync **Cash & Card**, **Diesel & Machinery**, and Diesel Log's three tabs
(Issued Log / Purchases / Machinery Master) — you don't have a Google
Sheet set up for any of these yet, so this is here for when you're ready.

The setup is the same shape as Parts 2–3 above: put your data in a Google
Sheet (or keep it as an .xlsx in Drive) with the right column headers, then
**Add Source** → **Choose file** → pick it → set **Syncs into** to the
kind you want. Two new template files are included alongside this guide —
`Cash_Card_Sync_Template.xlsx` and `Diesel_Machinery_Sync_Template.xlsx` —
each with a "Read Me" tab and the exact headers to use; the quickest start
is to open one, read its Read Me tab, delete the yellow example row, and
fill in your real data (or upload the template itself to Drive as a first
test). For Diesel Log, the `Diesel_Log_TEMPLATE.xlsx` you already have
works the same way — add one source per tab you want synced.

Each of these sections also got its own **Sync Now** button (next to
Import/Export), so once a source is set up you don't need to come back to
this screen every time — see README.md's "Sync from Google Sheets now
also covers Cash & Card, Diesel & Machinery and Diesel Log" section for
the full detail, including the one dedup limitation worth knowing about
for Cash & Card (same-day identical entries).

## One thing still open

You mentioned a "diesel analysis book" that's been removed from the app —
I looked through the app's full build history and couldn't find any past
version of it, so I don't want to guess and build the wrong thing. If you
built it in a different chat about this same project, the most reliable way
forward is either: paste what that chat produced (the description of what
it showed, or the code if you have it), or describe what columns/numbers it
displayed, and I'll add it properly in the next update.
