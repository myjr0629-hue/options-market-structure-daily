# US Options Market Structure — Daily Snapshots

Daily snapshots of **options market structure** for large-cap US stocks and ETFs — the same numbers shown in the free **SIGNUM HQ** app:

- **Max pain** — the strike where total option-holder value is minimized at expiration
- **Net GEX** — net dealer gamma exposure (sign shows whether dealers are net long or short gamma)
- **Gamma flip** — the spot level where net GEX changes sign
- **Call wall / put floor** — strikes with the largest call / put open interest

Files land in this repo as `YYYY-MM-DD.json` (one object per ticker, plus field definitions). Coverage today: SPY, QQQ, AAPL, NVDA, TSLA, MSFT, AMZN, META, AMD, GOOGL, NFLX, AVGO.

<p align="center">
  <img src="signum-command-screen.png" alt="SIGNUM HQ — Command screen showing max pain, gamma exposure and dark pool share for a single ticker" width="360">
</p>

## Where the numbers come from

Every value is a **derived metric** computed by the SIGNUM HQ app from the current listed options chain (open interest and greeks). No quotes or raw chain data are redistributed here — only the structure levels. The app shows the same levels live, with the expiration selector, dark-pool share and AI briefings that don't fit in a JSON file.

- **iOS / Android:** https://www.signumhq.com/app?from=github
- **Web:** https://www.signumhq.com

## How people use this

Traders and researchers use structure levels to *describe* where positioning is concentrated — for example, how far spot sits from max pain into a weekly expiration, or whether net GEX is positive (dealers hedge against moves) or negative (dealers hedge with moves). These are descriptions of current positioning, not forecasts.

## Congress trades (last 90 days)

`congress-trades-90d.csv` keeps one row per stock trade disclosed by members of the US Senate and House under the STOCK Act in the last 90 days: ticker, side, transaction date, disclosure date, reporting lag in days, amount range, range midpoint, member, chamber, a merged `member_key` (the same member often appears under different spellings in the source data) and a link to the official filing. `congress-by-ticker-90d.json` folds it per ticker: buys, sells, estimated net flow from the range midpoints and the number of distinct members.

Browse it at https://myjr0629-hue.github.io/options-market-structure-daily/congress.html, with one page per member.

## Usage

```python
import pandas as pd
import requests

BASE = "https://raw.githubusercontent.com/myjr0629-hue/options-market-structure-daily/main"

# One day of options structure: {"snapshotDateET", "fields", "tickers": {"SPY": {...}, ...}}
snap = requests.get(f"{BASE}/2026-09-22.json", timeout=30).json()
levels = pd.DataFrame.from_dict(snap["tickers"], orient="index")
print(levels[["expiration", "spot", "maxPain", "netGex", "gammaFlip", "callWall", "putFloor"]])

# How far spot sits from max pain, in percent
levels["spot_vs_max_pain_pct"] = (levels["spot"] / levels["maxPain"] - 1) * 100

# Congress trades, one row per disclosed trade
trades = pd.read_csv(f"{BASE}/congress-trades-90d.csv")
per_member = trades.groupby("member_key").agg(trades=("ticker", "size"), tickers=("ticker", "nunique"), median_lag_days=("lagDays", "median"))
print(per_member.sort_values("trades", ascending=False).head())
```

Field definitions for the daily files are in the `fields` object of each JSON file.

## Notes & license

- Snapshots are taken after the US close; the `expiration` field states which expiration the levels refer to.
- Data (JSON) is released under **CC BY 4.0** — cite “SIGNUM HQ (signumhq.com)”. The screenshot is © SIGNUM HQ, LLC.
- Nothing here is investment advice. Past positioning does not determine future prices.
