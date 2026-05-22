---
name: stock-picker
description: Run a stock analysis pipeline that screens S&P 500 stocks by industry, fetches price history, applies sentiment scoring, and generates SARIMAX forecasts. Use when the user wants to analyze stocks, screen an industry sector, get a stock report, or run the stock picker.
---

# stock-picker — S&P 500 Stock Analysis

Runs a multi-agent pipeline that screens S&P 500 constituents for a given industry, downloads OHLCV price history, scores news sentiment (if `NEWSAPI_KEY` is set), fits a SARIMAX forecast model, and writes a ranked report to `report/`.

## Location

```
/home/javastarchild/stock_picker/stock_picker_agents.py
```

## How to run

**Important:** Use `python3.12` directly — the `uv` venv crashes on this machine due to a CPU instruction-set mismatch (SIGILL).

### Interactive mode (prompts for industry and months)

```bash
cd /home/javastarchild/stock_picker
python3.12 stock_picker_agents.py
```

### Batch mode (programmatic, non-interactive)

```python
import sys
sys.path.insert(0, '/home/javastarchild/stock_picker')
from stock_picker_agents import create_orchestrator, create_custom_config

config = create_custom_config(industry='technology', lookback_months=6)
orchestrator = create_orchestrator(config)
results = orchestrator.run_analysis('technology', 6)
```

## Key parameters

- **Industry** — any sector string, e.g. `technology`, `healthcare`, `financials`, `energy`
- **Lookback months** — how many months of price history to use (default 6; minimum needed for 30 trading days)
- **Forecast days** — configured in `AnalysisConfig`; default is 30 business days ahead

## Outputs

Reports are written to `/home/javastarchild/stock_picker/report/`:

- `<industry>_<date>.csv` — one row per ticker with forecast, sentiment, and metrics
- `<industry>_<date>_summary.txt` — human-readable ranked summary

To view the latest report:

```bash
ls -t /home/javastarchild/stock_picker/report/*.txt | head -1 | xargs cat
```

## Notes

- Tickers with fewer than 30 trading days of history are skipped (`success=False`).
- Without `NEWSAPI_KEY`, sentiment defaults to neutral (`sentiment_score=0`); analysis still runs.
- S&P 500 constituent list is cached at `.cache/sp500_constituents.csv` for 24 hours. Delete it to force a refresh.
- Tests: `python3.12 -m pytest tests/` (all 6 pass, no network calls).
