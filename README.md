# 📈 Overnight Gap Continuation & Fill Strategy (Gap-and-Go)

**An empirical backtesting study of the opening gap anomaly across stocks, cryptocurrencies, and forex.**

> Algorithmic Trading — Final Project
> **Muhammad Shayan Shahid** (2212325) · **Muhammad Amir** (2212295)
> Department of Computer Science, SZABIST Karachi · Instructor: Asif Khalid

🔴 **Live interactive dashboard:** https://gap-strategy-szabist-wbz9gaayvenqbmdxx9mjod.streamlit.app/

---

## 💡 The Idea

When a market closes overnight, news keeps arriving — so the next day's **open often differs from the previous close**, creating an *opening gap*. Trading folklore offers two contradictory rules: *"gaps get filled"* and *"gaps go."* We encode **both** into one conditional strategy and let the data decide which is true, and when:

| Condition (checked at the open) | Action |
|---|---|
| \|Gap%\| ≥ K1 × ATR% **and** Volume ≥ 1.5 × average | **Continuation** — trade *with* the gap |
| \|Gap%\| ≤ K2 × ATR% **and** Volume < 1.5 × average | **Fill** — fade the gap, target previous close |
| Anything in between | **No trade** (the ambiguous zone) |

All trades enter at the open and exit the same day — zero overnight risk. The gap size is normalized by the Average True Range (ATR) so a "2% gap" means something different for TSLA than for a forex pair.

**The control-group trick:** cryptocurrencies trade 24/7 and therefore *cannot* gap. Including BTC/ETH lets us prove gaps are created by market closures — if the strategy "worked" on crypto, it would be an artifact.

## 📁 Repository Structure

| File | What it is |
|---|---|
| `Gap_Strategy_Notebook.ipynb` | Full Databricks pipeline: yfinance → PySpark ETL → signals → trade-level backtest → benchmarks (Buy & Hold, SuperTrend 10/3) → charts. 29 cells. |
| `app.py` + `requirements.txt` | Interactive Streamlit dashboard — parameter sliders, decluttered candlestick chart, equity curves, gap analysis, downloadable trade log. |
| `gap_strategy.pine` | TradingView Pine Script v5 — plots live signals on any real chart with a built-in stats table and alerts. |
| `Gap_Strategy_Research_Paper.docx` | Full research paper (IMRaD format, 12 verified academic references). |

## 🚀 How to Run

**Notebook (Databricks):** Workspace → Import → upload the `.ipynb` → attach to a cluster → select all 9 tickers in the widget → Run All. Data downloads automatically via yfinance — no dataset files needed.

**Dashboard (local):**
```bash
pip install -r requirements.txt
streamlit run app.py
```

**TradingView:** open any daily chart → Pine Editor → paste `gap_strategy.pine` → Add to chart. Turn off the volume filter for forex pairs.

## 🔬 Methodology Highlights

- **No look-ahead bias** — ATR and average volume are lagged so every input is known at the open
- **70/30 chronological in-sample/out-of-sample split** to guard against backtest overfitting
- **Transaction costs** — all results reported gross and net of 10 bps per round trip
- **Parameter sensitivity heatmap** over the K1 × volume-multiplier grid
- Benchmarked against **Buy & Hold** and the **SuperTrend(10,3)** baseline on identical data
- **9 instruments · 3 asset classes · 60,781 daily records · 2018–2026**

## 📊 Key Results

**1. Crypto barely gaps — the control group works.** Opening gaps are created by market closures, and the 24/7 crypto market confirms it:

| Asset Class | Trading Days | Gaps > 0.5% | Mean \|Gap%\| |
|---|---|---|---|
| Equity | 10,615 | 6,178 (58%) | 1.03% |
| Forex | 4,399 | 1,081 (25%) | 0.36% |
| Crypto | 6,172 | 64 (1%) | **0.05%** |

**2. Neither folklore rule holds cleanly.** In equities, both continuation and fill trades win ≈50% of the time — indistinguishable from a coin flip:

| Asset Class | Signal | Trades | Win Rate | Avg ROI |
|---|---|---|---|---|
| Equity | Continuation | 235 | 51.1% | −0.23% |
| Equity | Fill | 8,302 | 50.7% | +0.02% |
| Forex | Fill | 2,813 | 28.1% | +0.001% |

**3. Transaction costs destroy the edge.** The forex case is the clearest: profitable gross, wiped out net of a 10 bps cost — a textbook demonstration of Park & Irwin (2007):

| Instrument | Gross Return | Gross Sharpe | Net Return (10 bps) | Net Sharpe |
|---|---|---|---|---|
| EURUSD=X (baseline) | +0.88% | 0.60 | **−79.3%** | **−24.7** |
| AAPL (improved) | −62.6% | −0.80 | −93.2% | −2.30 |

**4. Nothing beat Buy & Hold.** Out-of-sample, every instrument posted negative net returns and negative Sharpe ratios. For reference, NVDA buy-and-hold returned +4,017% over the period while the gap strategy bled out.

**Conclusion:** The opening-gap anomaly is statistically visible in equities and forex but was **not economically exploitable on daily data at retail cost levels** — a result fully consistent with the Efficient Market Hypothesis (Fama, 1970).

## ⚠️ Disclaimer

Educational research project. Historical backtest only — not investment advice. Data from Yahoo Finance via the open-source `yfinance` library.

## 🙏 Acknowledgements

Course framework and SuperTrend baseline from our Algorithmic Trading instructor, **Asif Khalid**, at SZABIST. Key literature: Caporale & Plastun (2017), Plastun et al. (2020), Lou, Polk & Skouras (2019), Park & Irwin (2007), Fama (1970).
