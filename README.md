# dispersion_trading

**Strategy Overview**

Dispersion trading exploits the spread between the **implied volatility of an index** and the **implied volatilities of its component stocks**. The core signal is *implied dirty correlation* - a metric derived from index IV versus weighted constituent IVs. Because implied correlation tends to mean-revert, extreme readings signal a mispricing opportunity.

The strategy is structured as a **volatility trade** using ATM straddles (ATM call + ATM put per symbol):
- When implied correlation is **too high** → short NSEBANK straddle, long constituent straddles (*short dispersion*)
- When implied correlation is **too low** → long NSEBANK straddle, short constituent straddles (*long dispersion*)

**Universe**
<img width="395" height="148" alt="image" src="https://github.com/user-attachments/assets/a7294c0d-baf5-4b37-badc-dda5e0b0f9cd" />

**Implementation Details**

**1. Data Preprocessing**

Options data is read from `dataset.xlsx`. Only rows with non-zero open interest are kept, and data is filtered to a **single expiry** (`5/26/2026`). The closest-to-money strike (one CALL + one PUT per date) is selected, forming an ATM straddle per symbol.

**2. Implied Volatility**

IV is calculated using the **Black-Scholes model** via the `mibian` library, with the futures price as the underlying and risk-free rate = 0.

**3. Implied Dirty Correlation**

$$\rho_{dirty} = \frac{\sigma_{index}^2 - \sum w_i^2 \sigma_i^2}{\left(\sum w_i \sigma_i\right)^2 - \sum w_i^2 \sigma_i^2}$$

**4. Trading Signal (EWMA Bands)**

An **EWMA** (λ = 0.94) is applied to dirty correlation to produce a rolling mean and standard deviation. Entry/exit bands are set at ±0.5 standard deviations:

| Condition | Action |
|---|---|
| `dirty_corr > upper_band` | Short dispersion: short NSEBANK straddle, long constituent straddles |
| `dirty_corr < lower_band` | Long dispersion: long NSEBANK straddle, short constituent straddles |
| `dirty_corr crosses EWMA mean` | Exit position |
| Expiry date | Force flat (position = 0) |

<img width="1214" height="605" alt="image" src="https://github.com/user-attachments/assets/43de9624-9f63-487b-afde-c05e5ea37718" />


**5. Delta Hedging**

Delta hedging is actually very important in option trading. However, in this example, I intentionally trade ATM straddle (ATM call + ATM put), so the add-up delta in each leg is almost 0. 
This can be considered as one of improvement points to increase the effectiveness of the model. 
<img width="292" height="563" alt="image" src="https://github.com/user-attachments/assets/0887571b-9e6a-4453-9cd5-eacefe28ebc2" />

**6. P&L Calculation**

Daily P&L per option = next trading day close − current close. Both ATM call and put are included, so each symbol contributes a combined long/short straddle P&L per day.
Total weighted P&L across all legs: Total PnL =  NSEBANK_PnL × lot_size - Total of (stock_PnL × lot_size × index_weight) 

**Evaluation**
<img width="1047" height="532" alt="image" src="https://github.com/user-attachments/assets/76d16535-fbcf-4da4-b466-3debe0238277" />
<img width="362" height="130" alt="image" src="https://github.com/user-attachments/assets/df59566a-b077-4063-9c76-3a2e60c44075" />

This strategy gives out positive result with cumulative PnL up to 20,000 PnL within 12 trading days. 
However, there are some improvement points: 
* The testing sample is small, this lead to abnormal Sharpe ratio. We can improve by increasing the sample with more expiry dates and more trading dates per expiry. The reason for small dataset is that the dataset.xlsx was built mannually due to lack of available information. 
* Risk-free rate in mibian.BS() is set at 0.
* EWMA parameters are not sensitivity-tested. λ=0.94 and ±0.5σ bands are standard defaults, not validated choices

**Disclaimer**

For **educational and research purposes only**. Does not constitute financial advice. Options trading involves substantial risk of loss.
