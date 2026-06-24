# Trading Journal v2.0 — Project Memory (Claude.md)
_Last updated: June 2026 — UI v3 complete_

## Project Overview
Personal web-based Day Trading Journal for Mohammed.  
Current phase: **UI v3 Design complete → Ready for Development**.  
Deployed on: AWS S3 (static hosting).  
Backend: Firebase Firestore + Firebase Auth (Anonymous).

---

## Tech Stack
| Layer | Technology |
|---|---|
| Frontend | HTML5, Vanilla JavaScript (ES6 Modules) |
| Styling | Tailwind CSS (CDN) + Custom CSS variables |
| Icons | Font Awesome 6 (CDN) |
| Charts | Chart.js (CDN) |
| Database | Firebase Firestore (NoSQL) |
| Auth | Firebase Auth — Anonymous + Custom Token |
| MT5 Sync | Python bridge (`mt5_to_firebase.py`) |
| Hosting | AWS S3 (static) |

---

## Color Palette (v2.0)
```
--bg:       #0e1217   (page background)
--card:     #161b22   (card/header background)
--input:    #1e2738   (input/button background)
--border:   #2a3347   (borders, grid lines)
--text:     #e8eaf0   (primary text)
--muted:    #8b9ab5   (secondary text, labels)
--purple:   #7371ff   (active tab, primary CTA, NEW badges)
--blue:     #3b82f6   (Initial Balance)
--cyan:     #38bdf8   (Current Balance, equity curve)
--green:    #4ade80   (profit, wins)
--green2:   #22c55e   (win rate)
--red:      #f87171   (loss, danger, worst trade, reset)
--amber:    #fbbf24   (best trade, 4R+ bucket, warnings)
```

---

## Firebase Config
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyB4w3eFdIjigrLIT1MoXIm_WteVpL6gdfA",
  authDomain: "trading-journal-2025-09.firebaseapp.com",
  projectId: "trading-journal-2025-09",
  storageBucket: "trading-journal-2025-09.firebasestorage.app",
  messagingSenderId: "13238776469",
  appId: "1:13238776469:web:60ab5bb8b06035222e619c"
};
```

## Firestore Paths
```
Trades:   artifacts/{appId}/public/data/trades          (collection)
Settings: artifacts/{appId}/public/data/settings/config (document)
```

---

## Firestore Data Schema

### Trade Document (v2.0)
```javascript
{
  id:          string,   // auto-generated or MT5 ticket
  userId:      string,   // Firebase Auth UID
  symbol:      string,   // e.g. "XAUUSD"
  side:        string,   // "Long" | "Short"  ← UI label: "Direction"
  date:        string,   // "YYYY-MM-DD"
  entryTime:   string,   // "HH:MM"  ← NEW v2.0
  session:     string,   // "Asian" | "London" | "New York"
  setupType:   string,   // from user-managed list  ← NEW v2.0
  timeframe:   string,   // "M1"|"M5"|"M15"|"H1"|"H4"|"D1"  ← NEW v2.0
  profit:      number,   // Net P&L in USD
  url:         string,   // Optional chart screenshot URL
  notes:       string,   // Trade notes
  timestamp:   string,   // ISO string for sorting

  // --- Phase 2 fields ---
  entryPrice:  number,
  exitPrice:   number,
  stopLoss:    number,
  takeProfit:  number,
  lotSize:     number,
  exitTime:    string,
  outcome:     string,   // "Hit TP" | "Hit SL" | "Manual Profit" | "Manual Loss" | "Breakeven"
  emotion:     string,   // optional
  followedPlan: boolean, // optional
  marketCondition: string, // optional
  newsEvent:   boolean,  // optional
  htfConfluence: number, // 1–5, optional
  mistakeCategory: string // optional
}
```

### Settings Document (v2.0)
```javascript
{
  initialBalance: number,
  setupTypes: string[]   // user-managed list, saved to Firestore
                         // default: ["Order Block (OB)", "Fair Value Gap (FVG)",
                         //           "BOS / CHoCH", "Liquidity Sweep", "MSS",
                         //           "Support / Resistance"]
}
```

---

## Tab 1 — Journal (v3 changes)

| Field | Status | Notes |
|---|---|---|
| "New Trade" → "Journal Trade" | ✅ Done | |
| "Side" → "Direction" | ✅ Done | |
| Time (Entry) field after Date | ✅ Done | `type="time"`, saves as `entryTime` |
| Session + P&L flex row | ✅ Done | |
| Setup Type dropdown | ✅ Done | Loads from Firestore `setupTypes[]` |
| Timeframe dropdown | ✅ Done | |
| Settings button in header | ✅ Done | Shows settings modal |
| Reset All Data button in header | ✅ Done | Red/danger styled |

### Settings Modal (v2.0)
- Initial Account Balance input (existing)
- **Manage Setup Types** section:
  - Tags shown with × remove button
  - Text input + "+ Add" button for new entries
  - Saved to Firestore `settings/config.setupTypes[]`
  - Loaded dynamically into Setup Type dropdown on Journal form

---

## Tab 2 — Analytics (v3 sections, top to bottom)

1. **Filter Bar** — Symbol, Setup, Session, Timeframe, Direction, Date Range + Reset button
2. **8 Metric Cards** — Initial Balance, Current Balance, Total P&L, Win Ratio, Long P&L, Short P&L, Best Trade, **Worst Trade (fixed)**
3. **4 Risk Cards** — Max Drawdown, Max Daily Loss, **Expectancy (NEW)**, **Avg Hold Time (NEW)**
4. **Equity Curve** — full width
5. **Session (compact) + R:R Distribution table** — 0.7fr / 1.3fr split; Best Instruments nested inside Session card below the bar chart
6. **Setup Performance table + Win/Loss Streak** — 2-col
7. **Day of Week Analysis** — KPIs + bar chart
8. **Best Hours of Day (heatmap) + Best Week of Month** — 2-col, both NEW
9. **AI Pattern Analyzer** — requirements panel + sample insights
10. **Reset Confirmation Modal** — shown at page bottom (separate section in UI)

---

## Key Logic & Fixes

### Worst Trade (fixed)
```javascript
// v2.0 CORRECT logic — only shows actual losses
const losses = trades.filter(t => t.profit < 0);
const worstTrade = losses.length > 0
  ? Math.min(...losses.map(t => t.profit))
  : null;  // display "—" and "No losses yet" if no losses exist
```

### R:R Distribution Table (detailed)
Columns: R:R Bucket | Pair(s) | Volume bar | # Trades | % of Total | Avg P&L  
Row per bucket: -3R, -2R to -3R, -1R to -2R, 0 (Breakeven), +1R to +2R, +2R to +3R, +3R to +4R, +4R and above  
Footer row: Total trades · Positive R% · Negative R% · Expectancy  

### Best Hours of Day (heatmap)
- Grid: 24 cells (00–23 UTC), 2 rows of 12
- Color intensity = win rate for that hour
- Classes: h0 (no data), h1 (1–25%), h2 (26–50%), h3 (51–75%), h4 (76–100%), hn (net loss hour)
- Best Hour + Avoid Hour callout cards below legend

### Best Week of Month
- 5 cards (Week 1–5), best week highlighted with green border + "★ Best" badge
- Shows Avg P&L + Win Rate per week
- Mini bar chart below cards

---

## Reset Confirmation — 2-Step Flow
```
Step 1: User clicks "Reset All Data" button (red, in both header + filter bar)
        → Modal opens with step indicator (● ○)
        → Warning box: shows trade count, confirms irreversibility
        → "Delete All Data" button = DISABLED (greyed out)
        → User must type exactly "RESET" (case-sensitive) in input field

Step 2: Input matches "RESET" exactly
        → "Delete All Data" button becomes ACTIVE (red, clickable)
        → User clicks → Firestore data deleted
        → Modal closes, toast confirmation shown
```

---

## Setup Types Management
- Stored in Firestore: `settings/config.setupTypes[]`
- Default list: `["Order Block (OB)", "Fair Value Gap (FVG)", "BOS / CHoCH", "Liquidity Sweep", "MSS", "Support / Resistance"]`
- Accessed via ⚙ Settings button in header (both tabs)
- Tags shown with × remove button in Settings modal
- Text input + "+ Add" button for custom entries
- Dropdown in Journal Trade form loads dynamically from this list

---

## AI Pattern Analyzer — Phase 2

### Already captured by v2.0 form
Symbol, Direction, Date, Entry Time, Session, Setup Type, Timeframe, P&L

### Needed for Phase 2 (add to form)
Entry Price, Exit Price, Stop Loss, Take Profit, Position Size (lots), Exit Time, Trade Outcome

### Optional (enriches AI insights further)
Emotional State, Followed Plan?, Market Condition, News Event, HTF Confluence Score (1–5), Mistake Category

### Implementation
- Model: `claude-sonnet-4-6`
- Input: full trade history as JSON from Firestore
- System prompt: instructs Claude to find behavioral patterns, streaks, session/setup correlations
- Output parsed into 3 insight card types: Pattern Detected, Behavior Alert, Edge Confirmed
- API key: must NOT be hardcoded in HTML — use backend proxy or environment variable
- Minimum useful dataset: ~20–30 trades with Phase 2 fields populated

---

## MT5 Python Bridge (existing)
```python
# mt5_to_firebase.py
# Connects to local MetaTrader 5 terminal
# Fetches historical deals via mt5.history_deals_get()
# Filters out deposits/withdrawals
# Calculates net profit = profit + commission + swap
# Auto-assigns session from UTC timestamp
# Pushes to Firestore
# NOTE: v2.0 needs to also push: entryTime, setupType, timeframe
```

---

## Development Phase Checklist

### Phase 1 (v2.0 core — start here)
- [x] Add `entryTime`, `setupType`, `timeframe` to trade form + `saveTrade()`
- [x] Fix Worst Trade logic (losses only, show "—" if none)
- [x] Settings modal: add Setup Types manager (add/remove tags, save to Firestore)
- [x] Load `setupTypes[]` dynamically into Journal Trade dropdown
- [x] Implement 2-step Reset confirmation modal (type "RESET" to unlock)
- [x] Build R:R Distribution as detailed table (pair column, all buckets, footer stats)
- [x] Build Best Hours of Day heatmap (24-cell grid, win rate by UTC hour)
- [x] Build Best Week of Month (5-card grid + mini bar chart)
- [x] Build Setup Performance table
- [x] Build Win/Loss Streak tracker + Plan Adherence bars
- [x] Add Expectancy metric card
- [x] Add Avg Hold Time metric card
- [x] Implement Analytics Filter Bar (client-side, all filters)
- [x] Move Best Instruments chart inside Session card (space saving)
- [x] Export CSV updated to include new v2.0 fields

### Phase 2 (AI + advanced)
- [ ] Add Entry/Exit/SL/TP/LotSize/ExitTime to form
- [ ] Add Trade Outcome, Emotion, FollowedPlan optional fields
- [ ] Implement AI Pattern Analyzer using Anthropic API
- [ ] Secure API key via backend proxy (not hardcoded)
- [ ] Multi-account support

---

## Notes
- Mohammed handles all AWS/infrastructure himself
- Developer handles Firebase/JS code tasks
- UI mockups (JPG) produced as reference before each development sprint
- Primary instruments: XAUUSD and BTCUSD
- App v1.0 live on S3; v2.0 development starting next sprint

---
## v2.0 Phase 1 — Deployed
- File: trading_journal_v2.html (92KB)
- All Phase 1 checklist items complete
- Old data safe: same Firestore paths, no schema migration needed
- New fields (entryTime, setupType, timeframe) are additive — old trades show blank for them
- Deploy: upload trading_journal_v2.html to S3, replace old index.html
