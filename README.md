# Options Pricer & Greeks Visualiser 

A self-contained Jupyter notebook that prices European options with the **Black-Scholes-Merton** model, computes all five Greeks via **finite differences**, solves **implied volatility** with Newton-Raphson, renders interactive 3-D Greek surfaces, and runs a **historical delta / delta-gamma hedge simulation** on real market data.

---

## Features

| Module | What it does |
|---|---|
| **BSM Pricer** | Prices European calls and puts (scalar, pure Python) |
| **Greeks (FD)** | Computes Δ, Γ, ν, Θ, ρ via central finite differences |
| **IV Solver** | Recovers implied volatility with Newton-Raphson iteration |
| **3-D Surfaces** | Interactive Plotly surfaces over a (S, τ) grid for every Greek |
| **Dashboard** | 2×3 subplot overview of all Greeks in one figure |
| **Export** | Saves every figure as HTML and bundles them into a ZIP |
| **Hedge Simulation** | Replays a delta or delta-gamma hedge on a real stock's historical prices |

---

## Quick Start

### 1. Install dependencies

```bash
pip install numpy pandas yfinance plotly matplotlib
```

### 2. Launch the notebook

```bash
jupyter notebook TEAM_1_DERIVABLE_B.ipynb
```

### 3. Configure your option

When prompted, choose between a **custom** set of parameters or fetch **live market data** for any Yahoo Finance ticker:

```
Use a REAL stock? [y/n] (default n):
Ticker (e.g. AAPL):
Spot S  |  Strike K  |  Days to expiry  |  Risk-free r  |  Div yield q  |  Vol σ
```

> In non-interactive environments (CI, nbconvert) the notebook falls back to the built-in defaults automatically.

---

## Default Parameters

| Parameter | Default |
|---|---|
| Spot S | 105.0 |
| Strike K | 105.0 |
| Days to expiry τ | 90 |
| Risk-free rate r | 3 % |
| Dividend yield q | 1.5 % |
| Volatility σ | 27 % |
| Historical vol window | 60 trading days |

---

## Module Details

### BSM Pricer
Implements the closed-form BSM formula with a custom Abramowitz & Stegun polynomial approximation for the normal CDF (no `scipy` dependency).

### Greeks — Finite Difference
Uses central differences for Δ and Γ (step = 1 % of spot), Vega and Rho (bp-level bumps), and a forward difference for Θ (1-day step).

### Implied Volatility
Newton-Raphson with a 30 % starting guess, clamped to `[1e-6, 10.0]`, converging to `1e-7` tolerance within 200 iterations.

### 3-D Surfaces & Dashboard
Plotly surfaces rendered over a configurable (S, τ) grid. Auto-detects the runtime renderer (Colab, classic notebook, terminal browser). Figures are exported as self-contained HTML files (Plotly CDN) and zipped for easy sharing.

### Historical Hedge Simulation
Fetches price history via `yfinance`, then steps through each trading day:

- **Delta hedge** — rebalances a stock position to keep net Δ ≈ 0.
- **Delta-gamma hedge** — adds a second option position to neutralise Γ, then adjusts the stock to neutralise the residual Δ.

Rebalancing frequency, contract multiplier, position direction (long/short), and hedge-option strike are all configurable. At the end the simulation prints a daily P&L table, a final summary, and two matplotlib charts (hedged P&L vs. time and hedged vs. unhedged comparison).

---

## Output Files

| File | Description |
|---|---|
| `greeks_export/<TICKER>_<greek>_<type>.html` | Individual interactive Greek surfaces |
| `greeks_export/<TICKER>_dashboard_<type>.html` | All-Greeks dashboard |
| `greeks_export.zip` | ZIP bundle of all HTML exports |
| `<TICKER>_hedge_history.csv` | Day-by-day hedge simulation log |

---

## Project Structure

```
TEAM_1_DERIVABLE_B.ipynb   # Main notebook (all code in one file)
README.md
```

---

## Dependencies

- Python ≥ 3.9
- `numpy`
- `pandas`
- `yfinance`
- `plotly`
- `matplotlib`

All standard-library modules used (`math`, `os`, `zipfile`) require no installation.

---

## Notes

- The hedge simulation requires a real ticker (not the custom mode). If `ASSET_NAME == "CUSTOM"`, the simulation section is skipped automatically.
- Live market data (spot, risk-free rate from `^IRX`, trailing dividend yield) is fetched at runtime; results will vary with the date of execution.
- Greek surfaces are computed with a pure-Python double loop and can be slow for large grid sizes (`n > 40`). Reduce `n` if needed.
