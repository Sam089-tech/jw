# Intraday Strategy with Daily Overlay (GARCH + RSI/Bollinger)

A small research project that builds a **5‑minute intraday strategy** conditioned on a **daily overlay**.  
The intraday signal uses **RSI + Bollinger Bands**; the daily overlay can be any daily bias you prefer.  
For demo purposes this repo uses **simulated daily and 5‑minute data** and includes a notebook that you can run end‑to‑end.

> Notebook: `notebooks/Building_An_Intraday_Strategy_Using_GARCH_Model.ipynb`

## What this does

1. **Load (or generate) data**
   - Daily: `simulated_daily_data.csv` (has `date`, `price`, `log_ret`, etc.)
   - Intraday 5‑minute: `simulated_5m_data.csv` (has `datetime`, `price`, etc.)
2. **Daily overlay (example)**
   - You can plug in any daily bias (e.g., your GARCH‑based variance regime, SMA cross, etc.). In the notebook it is stored as `signal_daily ∈ {‑1, 1}`.
3. **Intraday signal**
   - RSI(20) + Bollinger(20, 2). Example rule:
     - `+1` if `RSI > 60` and `close > upper_band`
     - `‑1` if `RSI < 40` and `close < lower_band`
4. **Combine signals (Fade option shown)**
   - If `signal_daily == 1` and intraday == ‑1 → **buy dips** (+1)
   - If `signal_daily == ‑1` and intraday == +1 → **sell rips** (‑1)
   - Otherwise **flat (0)**
5. **No look‑ahead**
   - We act on the **next bar**: `position = signal.shift(1)`
6. **Returns & aggregation**
   - Intraday strategy P&L → grouped to daily with compounding: `(1 + r).prod() - 1`
   - Plot cumulative performance.

> Notes
> - Indicators are implemented in **pure pandas/numpy** (no `pandas_ta`) to avoid the `Numba / NumPy` version issue.
> - If you want to re‑enable `pandas_ta`, pin `numpy<=2.2.*` and use a matching `numba` version.

---

## Results (example)
The notebook renders:
- Cumulative strategy return chart (daily frequency)
- Quick diagnostics of signals and trading activity

> This project uses **simulated** data to keep it portable. Replace the CSVs with your real data and keep the same column names, or adapt the loader cells accordingly.

---

## Project structure

```
.
├── data/
│   ├── simulated_daily_data.csv
│   └── simulated_5m_data.csv
├── notebooks/
│   └── Building_An_Intraday_Strategy_Using_GARCH_Model.ipynb
├── README.md
├── requirements.txt
├── environment.yml
├── LICENSE
└── .gitignore
```

> Put your CSVs under `data/`. The notebook expects them there by default.

---

## Quick start

### 1) Create environment (Conda)
```bash
conda env create -f environment.yml
conda activate intraday-garch
```

### 2) Launch Jupyter
```bash
jupyter lab
# or
jupyter notebook
```

### 3) Run the notebook
Open `notebooks/Building_An_Intraday_Strategy_Using_GARCH_Model.ipynb` and run all cells.

---

## Dependencies

- Python 3.11 (tested)
- pandas, numpy, matplotlib, tqdm
- arch (optional; if you compute daily GARCH overlays)

Install via pip instead of conda (alternate):

```bash
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## Replace with your data (optional)

If you want to use real data, keep the following columns:

- **Daily CSV** (`data/simulated_daily_data.csv`)
  - `date` (YYYY‑MM‑DD), `price` (close)
- **Intraday 5‑minute CSV** (`data/simulated_5m_data.csv`)
  - `datetime` (ISO timestamp), `price` (close)

You can add OHLC if you like, but the notebook only needs `close`.

---

## Troubleshooting

- **`AttributeError: 'DataFrame' object has no attribute 'plt'`**  
  Use `.plot(...)` on Series/DataFrame, not `.plt(...)`. `plt` is from matplotlib.

- **Numba/NumPy version error** (e.g., *Numba needs NumPy 2.2 or less*)  
  This repo avoids numba by not importing `pandas_ta`. If you add `pandas_ta`, pin `numpy<=2.2.*` and a compatible `numba`, or just use the pure‑pandas indicators provided in the notebook.

- **`FileNotFoundError` for CSVs**  
  Check paths. The notebook expects CSVs in `./data/` by default.

---

## License

MIT — see `LICENSE`.

---

## Acknowledgements

- The `arch` package for volatility modeling
- pandas/matplotlib for data wrangling and plotting
- Inspiration from classic RSI/Bollinger intraday studies
