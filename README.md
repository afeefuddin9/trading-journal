# AasimFxBook — Trading Journal v2.0.

> A professional single-file day trading journal with Firebase backend, real-time sync, and deep performance analytics. Built for serious traders who want to track, review, and improve their edge.

---

## Table of Contents

1. [What's New in v2.0](#whats-new-in-v20)
2. [Features](#features)
3. [Tech Stack](#tech-stack)
4. [Project Files](#project-files)
5. [Before You Deploy — What to Change](#before-you-deploy--what-to-change)
6. [Firebase Setup](#firebase-setup)
7. [AWS S3 Deployment](#aws-s3-deployment)
8. [First-Time App Setup](#first-time-app-setup)
9. [How to Use the App](#how-to-use-the-app)
10. [Data Structure (Firestore)](#data-structure-firestore)
11. [Customising Setup Types](#customising-setup-types)
12. [Exporting Your Data](#exporting-your-data)
13. [Resetting the Journal](#resetting-the-journal)
14. [Phase 2 Roadmap — AI Pattern Analyzer](#phase-2-roadmap--ai-pattern-analyzer)
15. [MT5 Sync Bridge](#mt5-sync-bridge)
16. [Troubleshooting](#troubleshooting)
17. [Changelog](#changelog)

---

## What's New in v2.0

| Area               | v1.0              | v2.0                                                                              |
| ------------------ | ----------------- | --------------------------------------------------------------------------------- |
| Trade form label   | "Side"            | **Direction**                                                                     |
| Trade entry        | No time field     | **Entry Time** field added                                                        |
| Trade notes        | Notes only        | Notes + **Mistake** + **Learning**                                                |
| Setup types        | Not tracked       | **Setup Type** dropdown (user-managed)                                            |
| Timeframe          | Not tracked       | **Timeframe** dropdown (M1–D1)                                                    |
| Trade expand       | No                | **Click any trade** to see chart image, notes, mistake, learning                  |
| Recent trades      | All trades listed | **Last 5 only** with "View all" link                                              |
| Analytics filter   | None              | **Full filter bar** — symbol, setup, session, TF, direction, P&L type, date range |
| Date filter        | None              | Last 7D / Last 30D / This Month / All Time / **Custom Date Range**                |
| Worst Trade logic  | Smallest profit   | **Biggest loss** (most negative P&L)                                              |
| R:R Distribution   | Basic chart       | **Interactive** — click a bar, see all trades in that bucket with pair names      |
| Setup performance  | None              | **Per-setup table** — win rate bar, avg P&L, best, worst                          |
| Streak tracker     | None              | **W/L dots**, max streak counts, **plan adherence bars**                          |
| Best Hours         | None              | **24-hour UTC heatmap** — green = profitable, red = loss                          |
| Best Week of Month | None              | **Week 1–5 breakdown** across all months                                          |
| Trade Logs         | None              | **Full searchable table** — all trades, expandable detail rows                    |
| Reset protection   | Single confirm    | **Double confirmation** — requires typing `RESET`                                 |
| Setup type manager | Not editable      | **Add/remove setup types** in Settings                                            |
| Branding           | Generic icon      | **AasimFxBook logo** + favicon                                                    |

---

## Features

### Journal Tab

- **Calendar** — monthly view, profit days green, loss days red, click a day to see P&L summary
- **Journal Trade form** — Symbol, Direction, Date, Entry Time, Session, P&L, Setup Type, Timeframe, Position URL, Notes, Mistake, Learning
- **Recent Trades** — last 5 trades, click any row to expand and view chart screenshot, notes, mistakes, and learnings
- **Stats bar** — Initial Balance, Current Balance, Total P&L, Win Rate

### Analytics Tab

- **Filter bar** — slice data by any combination of symbol, setup, session, timeframe, direction, P&L type, and date range
- **8 metric cards** — balance, P&L, win ratio, Long/Short P&L, best/worst trade
- **Risk cards** — max drawdown, max daily loss, expectancy (in R), avg hold time
- **Equity curve** — full-width account performance chart
- **Session performance** — bar chart by Asian / London / New York
- **Best instruments** — horizontal bar chart, top 5 symbols by P&L
- **R:R Distribution** — interactive histogram, click/select any R bucket to see all trades in that bucket with symbol, direction, date, and P&L
- **Setup performance table** — per-setup win rate, avg P&L per trade, total, best, worst
- **Win/Loss Streaks** — last 20 trades visualised, max win/loss streak counts, plan adherence bars
- **Day of Week analysis** — best day, day to avoid, average daily P&L, bar chart
- **Best Hours of Day** — 24-hour UTC heatmap with colour intensity showing profitability
- **Best Week of Month** — Week 1–5 P&L breakdown and win rate
- **Trade Logs** — full searchable table with expandable rows showing chart image, notes, mistake, and learning

### Settings

- Set initial account balance
- Manage setup types — add your own setups (OB, FVG, etc.), remove ones you don't use
- Reset all trade data with double confirmation

---

## Tech Stack

| Layer      | Technology                                  |
| ---------- | ------------------------------------------- |
| Frontend   | HTML5, Vanilla JavaScript (ES6 Modules)     |
| Styling    | Tailwind CSS via CDN + custom CSS variables |
| Charts     | Chart.js via CDN                            |
| Icons      | Font Awesome via CDN                        |
| Database   | Firebase Firestore (NoSQL, real-time)       |
| Auth       | Firebase Anonymous Auth                     |
| Hosting    | AWS S3 (static website)                     |
| MT5 Bridge | Python (`mt5_to_firebase.py`)               |

---

## Project Files

```
AasimFxBook/
├── day_trading_journal_v2.html   ← Main app (deploy this to S3)
├── README.md                     ← This file
├── CLAUDE.md                     ← Developer context for AI sessions
├── mt5_to_firebase.py            ← MetaTrader 5 sync bridge (optional)
├── logo_transparent.png          ← Logo asset (already embedded in HTML)
└── favicon_32.png                ← Favicon (already embedded in HTML)
```

> **Note:** The logo and favicon are already embedded as base64 inside `day_trading_journal_v2.html`. You only need to deploy the single HTML file to S3.

---

## Before You Deploy — What to Change

Open `day_trading_journal_v2.html` in a text editor and find the Firebase config block (around line 730). It looks like this:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyB4w3eFdIjigrLIT1MoXIm_WteVpL6gdfA",
  authDomain: "trading-journal-2025-09.firebaseapp.com",
  projectId: "trading-journal-2025-09",
  storageBucket: "trading-journal-2025-09.firebasestorage.app",
  messagingSenderId: "13238776469",
  appId: "1:13238776469:web:60ab5bb8b06035222e619c",
};
```

**If you are Mohammed (AasimFx):** This config already points to your Firebase project. No changes needed — your existing trade data will load automatically.

**If you are setting this up for a new user or new account:** Replace all six values with the config from your own Firebase project (see [Firebase Setup](#firebase-setup) below).

---

## Firebase Setup

> Skip this section if you already have a Firebase project configured.

### Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → give it a name → Continue
3. Disable Google Analytics if not needed → Create project

### Step 2 — Enable Firestore

1. In the left sidebar click **Build → Firestore Database**
2. Click **Create database**
3. Choose **Start in production mode** → select your nearest region → Enable

### Step 3 — Set Firestore security rules

In Firestore → Rules tab, paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /artifacts/{appId}/public/data/{document=**} {
      allow read, write: if true;
    }
  }
}
```

Click **Publish**.

> This allows public read/write to the `public/data` path. Since the app uses anonymous auth and is for personal use, this is acceptable. For multi-user setups you would tighten these rules.

### Step 4 — Enable Anonymous Authentication

1. In the left sidebar click **Build → Authentication**
2. Click **Get started**
3. Under Sign-in providers, click **Anonymous** → Enable → Save

### Step 5 — Get your config values

1. In the left sidebar click the ⚙️ gear → **Project settings**
2. Scroll down to **Your apps** → click **Web** (`</>`)
3. Register the app (give it any nickname)
4. Copy the `firebaseConfig` object shown and paste it into the HTML file replacing the existing values

---

## AWS S3 Deployment

### Step 1 — Create an S3 bucket

1. Open the [AWS S3 console](https://s3.console.aws.amazon.com)
2. Click **Create bucket**
3. Give it a unique name (e.g. `aasimfxbook-journal`)
4. Choose your preferred region
5. **Uncheck** "Block all public access" → confirm the acknowledgement
6. Leave everything else as default → **Create bucket**

### Step 2 — Enable static website hosting

1. Open your newly created bucket
2. Go to **Properties** tab
3. Scroll to **Static website hosting** → click **Edit**
4. Enable it
5. Set **Index document** to `index.html`
6. Click **Save changes**

### Step 3 — Upload the file

1. Go to the **Objects** tab
2. Click **Upload** → **Add files**
3. Select `day_trading_journal_v2.html`
4. **Important:** Before clicking Upload, click **Additional upload options** → under **Storage class** keep Standard, and under **Server-side encryption** leave as default
5. Click **Upload**

### Step 4 — Rename the file as index.html (optional but recommended)

If you want to access the app directly at your bucket URL without a filename:

1. After upload, you can either rename it to `index.html` before uploading, or
2. Use the bucket URL with the filename: `http://your-bucket.s3-website-region.amazonaws.com/day_trading_journal_v2.html`

To rename: delete the uploaded file, rename your local copy to `index.html`, and re-upload.

### Step 5 — Set bucket policy for public access

1. Go to the **Permissions** tab on your bucket
2. Scroll to **Bucket policy** → click **Edit**
3. Paste the following (replace `YOUR-BUCKET-NAME` with your actual bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

4. Click **Save changes**

### Step 6 — Get your app URL

1. Go to **Properties** → **Static website hosting**
2. Your URL will be in this format:
   ```
   http://YOUR-BUCKET-NAME.s3-website-REGION.amazonaws.com
   ```
   For example: `http://aasimfxbook-journal.s3-website-us-west-1.amazonaws.com`

Open that URL in your browser — the app should load.

### Optional: Custom domain with CloudFront

If you want to use a custom domain (e.g. `journal.aasimfx.com`) or HTTPS:

1. Create a **CloudFront distribution** pointing to your S3 bucket
2. Add an **SSL certificate** via AWS Certificate Manager (free)
3. Point your domain's DNS to the CloudFront distribution URL

---

## First-Time App Setup

When you open the app for the first time:

1. Click the **⚙️ gear icon** in the top-right corner
2. Enter your **Initial Account Balance** (e.g. `10000` for a $10,000 account)
3. Under **Setup Types**, you'll see the defaults: Order Block (OB), Fair Value Gap (FVG), BOS / CHoCH, Liquidity Sweep, MSS, S/R Flip
4. Add any setups you use that aren't listed, or remove ones you don't use
5. Click **Save Settings**

Your balance is saved to Firestore and will persist across sessions.

---

## How to Use the App

### Logging a trade

Fill in the **Journal Trade** form on the left:

| Field            | Notes                                                                |
| ---------------- | -------------------------------------------------------------------- |
| **Symbol**       | Ticker in uppercase — e.g. `XAUUSD`, `BTCUSD`, `EURUSD`              |
| **Direction**    | Long (buy) or Short (sell)                                           |
| **Date**         | Trade date                                                           |
| **Time (Entry)** | Entry time in your local timezone — used for hour-of-day analysis    |
| **Session**      | Asian, London, or New York                                           |
| **P&L ($)**      | Net profit or loss in dollars — use negative for losses (e.g. `-83`) |
| **Setup Type**   | The pattern/setup that triggered your entry                          |
| **Timeframe**    | Entry timeframe — M1, M5, M15, H1, H4, D1                            |
| **Position URL** | Link to a TradingView screenshot or any image URL of your chart      |
| **Notes**        | Entry reasoning, confluences, what you observed                      |
| **Mistake**      | What you did wrong on this trade (if anything)                       |
| **Learning**     | What you will do differently next time                               |

Click **Save Trade** — it saves to Firestore in real time and appears instantly in the Recent Trades list.

### Reviewing a trade

Click any trade row in Recent Trades to expand it. You will see:

- Chart image loaded from your Position URL
- Your notes, mistake, and learning
- A link to open the chart in a new tab

### Analysing performance

Switch to the **Analytics** tab. Use the filter bar at the top to narrow data by any combination of symbol, setup, session, timeframe, direction, profit/loss type, and date range.

For date ranges, select **Custom Date** from the dropdown to enter a specific from/to date range.

Click any bar in the **R:R Distribution** chart to filter the trade list on the right to only show trades in that R bucket.

---

## Data Structure (Firestore)

All data lives in your Firebase project under this path structure:

```
artifacts/
  {projectId}/
    public/
      data/
        trades/          ← collection, one document per trade
          {tradeId}
            symbol       string   "XAUUSD"
            direction    string   "Long" | "Short"
            side         string   same as direction (backward compat)
            date         string   "YYYY-MM-DD"
            entryTime    string   "HH:MM" (24h)
            session      string   "Asian" | "London" | "New York"
            profit       number   123.45 or -83.00
            setupType    string   "Order Block (OB)"
            timeframe    string   "M15"
            url          string   "https://..."
            notes        string   free text
            mistake      string   free text
            learning     string   free text
            userId       string   Firebase anonymous UID
            timestamp    string   ISO datetime of when logged

        settings/
          config
            initialBalance   number   10000
            setupTypes       array    ["Order Block (OB)", "FVG", ...]
```

> **Backward compatibility:** The v1.0 app stored trades with a `side` field instead of `direction`. The v2.0 app reads both (`direction || side`) so all your old trades load correctly. New trades write both fields.

---

## Customising Setup Types

Your setup types are stored in Firestore and loaded dynamically into the form dropdown each time the app opens.

**To add a setup:**

1. Click ⚙️ Settings
2. Type the setup name in the input at the bottom of the Setup Types section
3. Press **Enter** or click **Add**
4. Click **Save Settings**

**To remove a setup:**

1. Click ⚙️ Settings
2. Click the **✕** next to any setup type you want to remove
3. Click **Save Settings**

Changes take effect immediately for all new trades. Existing trades that used a removed setup type are not affected — they keep their original setup value.

---

## Exporting Your Data

Click **Export CSV** on the Journal tab to download all your trades as a CSV file named `aasimfxbook_journal.csv`.

The CSV includes these columns:

```
id, date, entryTime, session, symbol, direction, setupType,
timeframe, profit, url, notes, mistake, learning
```

You can open this in Excel, Google Sheets, or import it into any analysis tool.

---

## Resetting the Journal

The Reset button is in the **Filter bar** on the Analytics tab (red **Reset** button on the right side). This resets the **filter state only** — it does not delete any trades.

To **delete all trade data**, click the ⚙️ gear and inside the Settings modal you will find a reset option (if added), or you can delete documents directly in the Firebase Firestore console.

The app uses a double-confirmation system to prevent accidental deletes — you must type `RESET` exactly to confirm.

---

## Phase 2 Roadmap — AI Pattern Analyzer

The Analytics tab already shows a preview of the upcoming AI Pattern Analyzer. This feature will use the Claude API to read your full trade history and surface behavioral patterns.

**What it will detect:**

- Which setups have the highest win rate per session
- Whether you overtrade after consecutive losses
- Your best symbol + session + timeframe combination
- Patterns in your mistakes across trades

**Fields already captured that feed the AI (v2.0):**

- Symbol, Direction, Date, Time, Session, Setup Type, Timeframe, P&L, Notes, Mistake, Learning

**Additional fields to add in Phase 2 for richer analysis:**

| Field                  | Why it matters                                           |
| ---------------------- | -------------------------------------------------------- |
| Actual R:R achieved    | Measures execution quality vs plan                       |
| Planned R:R            | Tells the AI if you're cutting winners or widening stops |
| Trade Outcome          | Hit TP / Hit SL / Manual Close — detects emotional exits |
| Emotion Tag            | Calm / FOMO / Revenge / Hesitant — correlates with P&L   |
| Followed Plan (Yes/No) | Separates disciplined trades from reactive ones          |
| Exit Time              | Enables accurate hold duration calculation               |

The more fields you fill per trade, the deeper and more accurate the AI insights will be. Even with Phase 1 fields alone, the AI can detect session-level and setup-level patterns.

---

## MT5 Sync Bridge

The `mt5_to_firebase.py` script connects to a running MetaTrader 5 terminal on your local machine and automatically pushes historical deals to Firestore.

### Requirements

```bash
pip install MetaTrader5 firebase-admin
```

You also need a Firebase service account key file (JSON). Download it from:
Firebase Console → Project Settings → Service Accounts → Generate new private key

### Configuration

Edit the top of `mt5_to_firebase.py`:

```python
# Path to your Firebase service account key
SERVICE_ACCOUNT_KEY = "path/to/serviceAccountKey.json"

# Your Firebase project ID
PROJECT_ID = "trading-journal-2025-09"

# MT5 login credentials
MT5_LOGIN    = 12345678
MT5_PASSWORD = "your-password"
MT5_SERVER   = "YourBroker-Server"

# How many days of history to sync
DAYS_BACK = 30
```

### Running the sync

```bash
python mt5_to_firebase.py
```

The script:

1. Connects to your MT5 terminal
2. Fetches deals from the last N days
3. Filters out deposits/withdrawals
4. Calculates net P&L (profit + commission + swap)
5. Auto-assigns the session based on UTC trade time
6. Pushes each trade to Firestore

> Note: The v1.0 bridge does not populate the new v2.0 fields (setupType, timeframe, entryTime, mistake, learning). After syncing, you can open each trade in the journal and fill in those fields manually.

---

## Troubleshooting

### Trades not loading

- Check your internet connection — the app requires access to Firebase (CDN at `gstatic.com`) and chart/icon CDNs
- Open browser DevTools (F12) → Console and look for Firebase errors
- Verify your Firebase config in the HTML matches your actual project values
- Check that Anonymous Authentication is enabled in Firebase Auth
- Check that Firestore security rules allow public read/write to the `artifacts/` path

### Analytics tab not switching

- This usually means the JavaScript module failed to load. Open DevTools → Console for errors
- Most common cause: Firebase CDN blocked or unreachable. Try on a different network

### Charts not rendering

- Chart.js loads from `cdn.jsdelivr.net` — check if it's blocked by your network/firewall
- Open DevTools → Network tab and look for failed requests

### Old v1.0 trades showing wrong direction

The v1.0 app stored `side: "Long"/"Short"`. The v2.0 app reads `direction || side`, so old trades should display correctly. If a trade shows blank direction, open it in the Firestore console and check its `side` field value.

### Position URL images not loading in trade expand

- The image must be publicly accessible via the URL
- TradingView snapshot URLs (`tradingview.com/x/...`) work well
- If the image fails to load, the expand panel shows "No image loaded" and the link still opens in a new tab

### Settings not saving

- Check DevTools → Console for Firestore permission errors
- Ensure the Firestore rules include the `settings` path under `artifacts/{appId}/public/data/`

### Deployed to S3 but seeing a 403 error

- Verify the bucket policy is correctly set (see [Step 5](#step-5--set-bucket-policy-for-public-access) above)
- Verify "Block all public access" is disabled on the bucket
- If you renamed the file, make sure the Static Website Hosting index document matches the filename exactly

---

## Changelog

### v2.0 — June 2026

- Added: Entry Time field for hour-of-day analysis
- Added: Direction field (replaces "Side", backward compatible with v1.0 data)
- Added: Setup Type dropdown (user-managed from Settings)
- Added: Timeframe dropdown
- Added: Mistake and Learning fields per trade
- Added: Trade expand panel with chart image preview on Journal tab
- Added: Recent Trades limited to last 5 with "View all in Analytics" link
- Added: Analytics filter bar (symbol, setup, session, TF, direction, P&L type, date range)
- Added: Custom date range filter
- Added: Profits Only / Losses Only filter
- Added: Worst Trade = biggest loss logic (not smallest profit)
- Added: R:R Distribution interactive chart with per-bucket trade list and pair names
- Added: Setup Performance table with per-setup win rate, avg P&L, best, worst
- Added: Win/Loss Streak tracker with max streak counts and plan adherence bars
- Added: Best Hours of Day — 24-hour UTC heatmap
- Added: Best Week of Month — week 1–5 performance cards
- Added: Trade Logs — full searchable trade table with expandable detail rows
- Added: Double confirmation Reset (type RESET to confirm)
- Added: Setup Types manager in Settings — add/remove custom setup tags
- Added: AasimFxBook logo and favicon (embedded as base64, no external files needed)
- Fixed: Analytics tab switching reliability
- Fixed: Modal auto-opening bug on load
- Fixed: JavaScript syntax error in CSV export
- Fixed: Old v1.0 trade data compatibility (side → direction fallback)

### v1.0 — 2025

- Initial release
- Journal tab with calendar, trade form, recent trades
- Analytics tab with equity curve, session chart, best instruments, day of week chart
- Firebase Firestore backend with anonymous auth
- CSV export
- Settings modal for initial balance
- MT5 sync bridge (Python)

---

_Built for AasimFx — Trading Journal for serious traders._
