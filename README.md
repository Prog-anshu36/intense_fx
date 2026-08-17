# intensefxTRADES

A full rebuild of your trading replay/journal tool, restyled to match the
intensefxTRADES reference screenshots: dark, card-based UI with a Dashboard,
Performance Analytics, Trade Analysis, a Trade Journal, and the original
chart/replay/order screen — now called Backtesting.

## Stack

- **Backend:** Flask (Python). Stores your account balance, open/closed
  trades, and journal entries in `backend/store.json`, so everything
  survives a restart.
- **Frontend:** Plain HTML/CSS/JS (no build step). Candlestick price data is
  generated client-side with a seeded random-walk generator — no internet
  connection or `yfinance` dependency required, so it always works offline.

## Run locally

```bash
cd tradefxbook
pip install -r requirements.txt
python backend/app.py
```

Then open **http://127.0.0.1:5050** in your browser.

The server keeps running in your terminal; press `Ctrl+C` to stop it.
Your account state lives in `backend/store.json` (auto-created on first
run, and git-ignored so your data never gets committed) — delete that file
or hit the reset endpoint (see below) to start over with a fresh account.

## Deploy so it's live on the internet

GitHub itself only hosts the code — it can't run the Flask backend (no
Python execution on GitHub Pages). Push this repo to GitHub, then deploy
the backend on a host that runs Python. **Render's free tier** is the
easiest:

1. Push this folder to a new GitHub repo:
   ```bash
   cd tradefxbook
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```
2. Go to [render.com](https://render.com) → **New → Web Service** → connect
   your GitHub repo. Render will read `render.yaml` in this repo
   automatically and set the build/start commands for you.
3. Click **Deploy**. Render gives you a live URL
   (e.g. `https://tradefxbook.onrender.com`) once the build finishes.

**Note on data persistence:** the app stores trades/journal in a JSON file
on disk. Render's **free** plan has an ephemeral filesystem, so
`store.json` resets on every redeploy or when the free instance spins down
from inactivity. That's fine for a demo/portfolio link. If you want data to
actually persist between deploys, upgrade to a paid Render plan and enable
the disk block already stubbed out (commented) in `render.yaml`, or swap
the JSON-file store for a real database later.

Railway, Fly.io, or PythonAnywhere work the same way — any host that runs
`pip install -r requirements.txt` then `gunicorn backend.app:app` will work,
using the included `Procfile`.

## Pages

- **Dashboard** — total/unrealized/realized P&L, win rate, an equity
  performance chart with 1D/1W/1M/3M/1Y/ALL ranges, a monthly P&L calendar,
  open positions, and top performers.
- **Trades** — all open and closed positions in one sortable list.
- **Journal** — write reflections on your trades (mood, tags, notes),
  optionally linked to a specific closed trade.
- **Analysis → Performance** — win rate, profit factor, expectancy, quick
  stats (avg winner/loser, streaks, risk:reward), and an equity curve, all
  filterable by time period and winners/losers.
- **Analysis → Trade Analysis** — the full trade log, sortable by any
  column, filterable by symbol.
- **Backtesting** — the core screen: a candlestick chart for EURUSD,
  GBPUSD, USDJPY, XAUUSD, or BTCUSD, replay controls (step, auto-play,
  speed), and an order panel (lots, take-profit/stop-loss in pips,
  market buy/sell).

All five symbols and all six timeframes (1m/5m/15m/1h/4h/1d) are simulated
with a seeded generator, so reloading the page gives you the same "history"
each time, while replaying still advances it forward and appends new candles
once you reach the end.

## API (for reference)

| Method | Path                  | Purpose                                  |
|--------|-----------------------|-------------------------------------------|
| GET    | `/api/state`          | Full account state                       |
| POST   | `/api/settings`       | Update saved symbol/timeframe/etc.       |
| POST   | `/api/trades/open`    | Open a simulated position                |
| POST   | `/api/trades/close`   | Close a position at a given exit price   |
| POST   | `/api/trades/update`  | Push latest tick(s); auto-closes SL/TP   |
| GET    | `/api/journal`        | List journal entries                     |
| POST   | `/api/journal`        | Add a journal entry                      |
| DELETE | `/api/journal/<id>`   | Delete a journal entry                   |
| POST   | `/api/reset`          | Wipe everything back to a fresh account  |

## Notes / known limitations

- This is a **local single-user** tool — there's no login, and the
  "Traders Lounge", "Trade Rooms", "Leaderboard", and "Settings" nav items
  are intentionally inert placeholders (greyed out), same as the
  "Coming Soon" treatment in the reference design.
- The theme toggle currently just shows a toast — the app is dark-mode only
  for now.
- Contract sizes used for P&L math: 100,000 units/lot for FX pairs, 100
  oz/lot for XAUUSD, 1 BTC/lot for BTCUSD. Adjust `CONTRACT_SIZE` in
  `backend/app.py` if you want different conventions.
