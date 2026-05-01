# dispersion_trading


# Strategy Overview

Dispersion trading is a form of correlation trading that profits from the difference between *implied* and *realized* correlation among index components. The core insight is that index options tend to overprice correlation relative to what actually materializes across individual stocks.

The strategy works by simultaneously:
- **Selling** straddles on an index (short index volatility)
- **Buying** straddles on the index's constituent stocks (long single-stock volatility)

or the reverse, depending on the prevailing implied correlation signal.

Because the positions are structured to be **delta-neutral**, the trade is insulated from directional market moves and profits purely from the volatility differential.



# How It Works

**1. Compute Implied Correlation ("Dirty Correlation")**

Implied correlation `ρ` is derived from the index implied volatility and the weighted implied volatilities of its components:

$$\rho = \frac{\sigma_{index}^2 - \sum w_i^2 \sigma_i^2}{\sum_{i \neq j} w_i w_j \sigma_i \sigma_j}$$

**2. Generate Entry Signals**

Signals are triggered when implied correlation reaches an extreme — above or below a threshold relative to its historical mean. Because implied correlation tends to mean-revert, extreme readings indicate a mispricing to be faded.

| Signal | Implied Correlation | Action |
|---|---|---|
| Short Dispersion | High (index vol cheap vs. components) | Sell index straddle, buy component straddles |
| Long Dispersion | Low (index vol rich vs. components) | Buy index straddle, sell component straddles |

**3. Delta Hedge**

The options portfolio is delta-hedged at regular intervals using index futures. When portfolio delta drifts beyond ±1, a futures contract is bought or sold to bring delta back near zero. This is done continuously throughout the life of the trade.

**4. Exit**

Positions are unwound when implied correlation reverts to its mean (`ρ = 0`).

**5. P&L Calculation**

Total P&L accounts for four components:
- Premium received/paid at trade entry
- Futures hedging costs
- Futures settlement at exit
- Options square-off value at exit

---

# Repository Structure

```
dispersion_trading/
├── dispersion_trading.ipynb   # Main notebook: full strategy implementation
├── dataset.xlsx               # Input data (prices, implied vols, weights)
└── README.md
```

---

# Requirements

```bash
pip install numpy pandas scipy matplotlib openpyxl jupyter
```

| Package | Purpose |
|---|---|
| `numpy` / `pandas` | Data manipulation and numerical computation |
| `scipy` | Statistical calculations |
| `matplotlib` | Visualization of correlation, signals, and P&L |
| `openpyxl` | Reading the `.xlsx` dataset |

---

## Usage

```bash
git clone https://github.com/duyenan0503/dispersion_trading.git
cd dispersion_trading
jupyter notebook dispersion_trading.ipynb
```

The notebook is self-contained. Run all cells in order — the dataset is loaded from `dataset.xlsx` in the same directory.

---

## Key Concepts

- **Implied Volatility (IV):** The market's forward-looking estimate of volatility, extracted from option prices.
- **Realized Volatility:** The volatility that actually occurred over a period.
- **Implied Correlation:** A measure of how correlated the market expects index components to be, derived from index vs. single-stock IV.
- **Straddle:** A simultaneous long call and long put at the same strike — a pure volatility bet with no directional bias.
- **Delta Neutrality:** Keeping the portfolio's sensitivity to the underlying price near zero, so P&L depends only on volatility, not direction.
- **Vega:** Sensitivity of an options position to changes in implied volatility — the primary exposure in this strategy.

---

## Important Notes

- Transaction costs are **not included** in the baseline P&L calculation.
- The strategy is most profitable in **low-correlation regimes** (individual stocks moving independently) and loses during **stress periods** when correlations spike.
- Delta hedging frequency has a significant impact on hedging costs and should be tuned to the underlying's liquidity.

---

## Disclaimer

This project is for **educational and research purposes only**. It does not constitute financial advice. Options trading involves substantial risk of loss.
