# Budget Tracker

A personal budget tracker that runs as a static web app on GitHub Pages and keeps its data in a **Google Sheet in your own Google account**. It works on desktop and phone, can be installed to the home screen, and keeps working offline.

**Live app:** https://dariomr98.github.io/budget-tracker/

This repository contains **code only**. It holds no financial data, exports, backups, tokens or secrets, and must never hold any.

## How it works

- **Sign-in:** Google Identity Services with the minimal scope `https://www.googleapis.com/auth/drive.file`. The app can only see files it created itself.
- **Storage:** On first sign-in the app creates a spreadsheet named **"Budget Tracker"** with the tabs `Expenses`, `Categories`, `Budget`, `Income`, `Cards` and `ImportLog`, and remembers its ID in `localStorage`. On another device, it finds the same spreadsheet in your Drive.
- **Sync:** The app reads the sheet when it opens and writes on every save. Offline, writes are queued on the device (`localStorage`) and synced when the connection returns. The chip at the top right shows the sync status. Tap it to sign in or retry.
- **Access token:** It is kept only in memory and in the browser tab's `sessionStorage`, and lasts about 1 hour. When it expires, the chip shows "Sign in to sync": tap it to reconnect. Local entries are never lost.
- **Offline shell:** `sw.js` caches the app files only. Google API requests and your data are never cached by the service worker.

### Accrued spending vs. cash flow

| | Based on | Includes | Excludes |
|---|---|---|---|
| Accrued spending | `date` (purchase date) | Counted expenses | Transfers, Credit Card Payments, Bank Fees (shown separately) |
| Cash flow | `cash_date` (money leaves the bank) | Counted expenses, Bank Fees | Transfers, Credit Card Payments |

**Unallocated cash** = income dated in the month − cash flow of the month − planned savings/debt lines (Fondo de Emergencia, Diezmo, Otras Metas de Ahorro, Pagos Extra a Deuda).

Each category has a *type* (editable on the Budget screen): Spending, Bank fee, Transfer, Credit card payment, or Savings/debt plan line. The exclusions follow the type, so renaming a category doesn't break them. Savings lines are budget-only: they are not offered in the Add Expense form.

### Credit card cash dates

For a credit card purchase, the statement closes on `cutoff_day` of the purchase month if the purchase day is on or before `cutoff_day`, otherwise on `cutoff_day` of the next month. The cash date is the first `payment_day` after the close date. Days past the end of a short month fall on its last day. All values come from each card's settings. Changing a card's cutoff or payment day recalculates the cash dates of that card's existing expenses.

## Publish on GitHub Pages

1. Push this repository to `https://github.com/dariomr98/budget-tracker` (public).
2. On GitHub, open the repository → **Settings → Pages**.
3. Under **Build and deployment**, set **Source: Deploy from a branch**, **Branch: `main`**, **Folder: `/ (root)`**, then **Save**.
4. Wait 1–2 minutes, then open https://dariomr98.github.io/budget-tracker/.

There is no build step. To update the app, commit and push. When you ship a change, bump `CACHE` in `sw.js` (for example `budget-tracker-v2`) so installed copies refresh their cached files.

## Google Cloud setup (one time)

In the [Google Cloud Console](https://console.cloud.google.com/), select the project that owns the OAuth client `752389430086-pn61oondfooggopt9eislq6dcujme4lh.apps.googleusercontent.com`:

1. **APIs & Services → Library:** enable **Google Sheets API** and **Google Drive API**. The Drive API lets a second device find the spreadsheet the app created.
2. **APIs & Services → Credentials →** open the OAuth 2.0 Client ID (type *Web application*). Under **Authorized JavaScript origins** add:
   - `https://dariomr98.github.io`

   Only the origin goes here, with no path and no trailing slash. No redirect URI is needed.
3. **OAuth consent screen** (Google Auth Platform → Audience): while the app is in **Testing**, add your own Gmail address under **Test users**. Only the scope `.../auth/drive.file` is needed.
4. Testing-mode consent can expire after 7 days. If Google asks you to sign in again, that's expected. Publishing the consent screen ("In production") avoids it. With only the `drive.file` scope, no verification is required for personal use.

The Client ID is a public identifier and is safe in the code. **Never** add a client secret, API key, access token or service-account file to this repository.

## Install on your phone

- **Android (Chrome):** open the app → menu ⋮ → **Install app** / **Add to Home screen**.
- **iPhone (Safari):** open the app → Share → **Add to Home Screen**.

## Backup

- The Google Sheet *is* the backup, and Google keeps its version history (File → Version history).
- **Backup → Export CSV** downloads every table in one CSV file. **Restore from CSV** replaces the tables found in the file and syncs them to the sheet. Keep exported files private, and never commit them: `.gitignore` blocks `*.csv`.

## Tests

Open `tests.html` (for example https://dariomr98.github.io/budget-tracker/tests.html, or locally with `python3 -m http.server` and http://localhost:8000/tests.html). It loads the app's own functions from `index.html?test` and checks:

- The required card examples (Bancomer Oct 2/Oct 4; Amex Oct 23/Oct 24/Nov 24).
- A cutoff on the 31st in short months, December → January, February (including leap years), and recalculation after a card change.
- MXN and es-MX formatting, and income schedules for every frequency.
- Accrued spending, cash flow, unallocated cash, savings rate, the essential split and 3-month averages.
- Budget colour thresholds, advice rules, sheet row mapping and CSV round trip.

The page shows a pass/fail table and sets the title to `PASS n/n`. Nothing on it talks to Google.

## Security notes

- Cards store only the **last 4 digits**: the form rejects anything else. There are no fields for full card numbers, CLABE, RFC or email.
- Advice is rule-based on your own data and is **not professional financial advice**.
