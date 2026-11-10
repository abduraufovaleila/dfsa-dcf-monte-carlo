# DFSA: DCF Valuation with Scenario Analysis and Monte Carlo Simulation

A five-year discounted cash flow (DCF) valuation of an investment project, built from scratch in Python. The model projects sales, operating costs, capital expenditure and working capital, discounts the free cash flows at a WACC-based cost of capital, and then stress-tests the result with deterministic scenarios and a stochastic Monte Carlo simulation.

> The notebook's text and variable names are in Spanish (it was written as a university finance case). This README is in English for a wider audience.

## What the model does

| Stage | Description |
|---|---|
| **1. Sales forecast** | Base-year sales of 100,000 grown with year-by-year volume growth (6%, 6%, 8%, 12%, 15%) and compounded inflation (3.2% falling to 2.0%). |
| **2. Operating costs** | Costs derived from the incremental sales and a margin that improves from 20% to 24%. |
| **3. Non-current assets** | Initial investment and annual CAPEX for infrastructure, machinery, control system and licence, each with its own growth rate and depreciation period. A second, larger investment round is modelled in year 3. |
| **4. Income statement** | Depreciation, staff costs and training costs lead to EBIT and net income at a 20% tax rate. |
| **5. Working capital** | Inventories, receivables and payables built from days-of-activity assumptions (inventory 120 days, collection 90 days, payment 75 days). |
| **6. Free cash flow** | Net income, plus depreciation, minus working-capital change, plus a year-2 grant of 108,000, plus investment and CAPEX. |
| **7. Cost of capital** | Beta unlevering/relevering, CAPM cost of equity, after-tax cost of debt, and a nominal cost of capital that varies with expected inflation. |
| **8. Valuation** | NPV, IRR and a terminal value using a perpetual growth rate of 4%. |
| **9. Scenarios** | Pessimistic (0.8x), base (1.0x) and optimistic (1.2x) sales growth. |
| **10. Monte Carlo** | 5,000 simulated sales paths with uncertainty that grows over time, first as independent draws and then with serial correlation between years. |

## Key results

| Metric | Value |
|---|---|
| Levered beta | 1.69 |
| Cost of equity | 14.86% |
| Nominal cost of capital (year 1) | 8.34% |
| **NPV, base case** | **50,616** |
| NPV, pessimistic (sales growth x 0.8) | 40,322 |
| NPV, optimistic (sales growth x 1.2) | 60,808 |
| Modified IRR ("TMR") | 25.87% |

NPV stays positive in all three scenarios. The Monte Carlo histogram shows how the NPV distribution spreads when sales are uncertain.

## Monte Carlo design

- **Step 1: independent shocks.** Sales growth each year is drawn from a normal distribution centred on the base-case assumption. The standard deviation grows linearly from 10% in year 1 to 60% in year 5, reflecting that long-term forecasts are less certain.
- **Step 2: persistence.** Sales follow an AR(1)-style process, so a good or bad year carries over into the next:

  `Sales_t = mu_t + rho * (Sales_(t-1) - mu_(t-1)) + epsilon_t`, with `rho = 0.6`.
- Sales are floored at zero, and each simulated path is pushed through the same cash-flow model to produce an NPV.

## Notes and limitations

- **IRR vs. MIRR.** The cash flows change sign more than once (the year-2 grant turns a negative year positive), so the standard IRR (-46%) is not meaningful. The modified IRR (reinvestment/finance-rate approach) is reported instead.
- **Terminal value.** The year-5 free cash flow is negative, so the perpetuity-growth terminal value is also negative. This is worth revisiting, for example by normalising the final-year cash flow before applying the terminal value.
- **Units.** Monetary figures follow the units of the original case statement.
- **Randomness.** The simulation does not set a random seed, so Monte Carlo results vary slightly between runs. Add `np.random.seed(42)` for reproducibility.

## Tech stack

Python, pandas, NumPy, numpy-financial, Matplotlib.

## Run it yourself

```bash
git clone https://github.com/abduraufovaleila/dfsa-dcf-monte-carlo.git
cd dfsa-dcf-monte-carlo
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook dfsa_dcf_monte_carlo.ipynb
```

`yfinance` and `statsmodels` are imported in the first cell but not used in the analysis. They are listed so the notebook runs unchanged.

## Author

Leila Abduraufova
