# FANG Oil Price → EPS Calculator — Method Guide

> A dedicated calculator for **Diamondback Energy (FANG)**.
> Input: WTI crude oil price ($/bbl) → Output: Annualized EPS estimates via 6 independent methods.

---

## How It Works

You enter a WTI oil price. The calculator runs it through 6 different models, each using a different approach to estimate what FANG's annual earnings per share (EPS) would be at that oil price. You then pick which models you trust, and the tool gives you the average, spread, and a confidence interval.

---

## The 6 Methods

### M1: Constant ΔWTI (Sensitivity Method)

**Idea:** Start from a known quarter (Q4 2024) and scale up or down based on how far your oil price is from that quarter's average.

**How it works:**
1. We know FANG earned $3.64/share in Q4 2024, when WTI averaged $70.30/bbl
2. We calculate how much EPS changes per $1 change in WTI, using FANG's oil production volume, tax rates, and share count
3. Multiply that sensitivity by the difference between your input price and $70.30
4. Add it to the Q4 2024 base (annualized)

**Formula:**
```
sensitivity = (oil_production × 365 × (1 - prod_tax) × (1 - income_tax)) / shares
annual_eps = (3.64 × 4) + sensitivity × (your_WTI - 70.3)
```

**Sensitivity ≈ $0.43/bbl** — every $1 increase in WTI adds ~$0.43 to annual EPS.

**Strengths:** Simple, transparent, anchored to real data.
**Weaknesses:** Assumes everything else stays constant (NGL prices, gas prices, costs, production volumes). Linear extrapolation only.

---

### M2: Bottom-Up Cost Stack (Oil Only)

**Idea:** Build FANG's income statement from scratch — revenue minus costs — using actual production volumes and per-barrel costs.

**How it works:**
1. **Revenue:** Oil revenue (your WTI price × oil production) + NGL revenue (WTI × 0.305 ratio × NGL production) + Gas revenue (fixed $1.40/Mcf × gas production)
2. **Costs:** Production taxes (7% of revenue) + cash operating costs (LOE + GPT + G&A at $/BOE) + DD&A + interest
3. **Net income:** (Revenue - all costs) × (1 - 23% tax rate)
4. **EPS:** Net income ÷ 286M shares
5. **Bias correction:** Multiply by 0.848 (historical models overpredict by ~15%, this corrects for it)

**Key assumption:** Gas price is fixed at $1.40/Mcf (Q3 2025 realized price). This is the "low gas" scenario.

**Strengths:** Most detailed — models actual cost structure.
**Weaknesses:** Many parameters to get right. Gas price assumption matters.

---

### M3: Bottom-Up Cost Stack (Oil + Gas)

**Idea:** Identical to M2, but with a user-configurable gas price (default $3.00/Mcf). The label and description update dynamically to reflect the current gas price setting.

**Differences from M2:**
- Gas price: editable, default $3.00/Mcf (instead of M2's fixed $1.40/Mcf)
- Bias correction: editable, default 1 (no correction). Previously hardcoded at 0.861.

**Why it matters:** Lets you model any gas price scenario. At $3/Mcf gas, FANG earns ~$2/share more annually than at $1.40/Mcf. Adjusting the gas price and bias lets you explore different assumptions.

---

### M4: Regression-Calibrated Bottom-Up

**Idea:** Take M3's raw output (before bias correction) and run it through a calibration formula derived from comparing model predictions to actual results over 5 quarters.

**How it works:**
1. Calculate M3's raw quarterly EPS (no bias correction)
2. Apply calibration: `calibrated_q_eps = -1.464 + 1.290 × raw_q_eps`
3. Annualize: multiply by 4

**Why:** The raw bottom-up model has systematic errors — it tends to underpredict at low oil prices and overpredict at high ones. The regression calibration adjusts for this non-linear bias, producing a steeper slope than M2/M3.

**Strengths:** Corrects for known model bias.
**Weaknesses:** Calibration is based on only 5 quarters. Small sample.

---

### M5: Historical OLS Regression

**Idea:** Skip the cost modeling entirely. Just fit a straight line through actual quarterly EPS vs. WTI price data from the 5 post-Endeavor-merger quarters.

**Formula:**
```
quarterly_eps = -8.490 + 0.1779 × WTI
annual_eps = quarterly_eps × 4
```

**R² = 0.94** — oil price explains 94% of the variation in FANG's quarterly EPS.

**Strengths:** Purely empirical — captures all real-world effects (hedging, NGL pricing, cost changes) that theoretical models might miss.
**Weaknesses:** Only 5 data points. Post-Endeavor period only, so the relationship may not hold if cost structure changes.

---

### M6: Barrel-to-Earnings Chain (Annual Regression)

**Idea:** A two-step chain model using annual (not quarterly) data from FY2022–2025:

**Step 1 — Model I: Oil Price → Profit Margin**
```
Net_Profit_Margin(%) = 0.70 × WTI - 18.55     (R² = 0.87)
```
Every $1/bbl increase in WTI lifts FANG's net margin by 0.70 percentage points. The -18.55 intercept represents fixed costs: even at $0 oil, FANG still has costs to pay.

**Step 2 — Model II: Profit Margin → EPS**
```
EPS($) = 0.47 × Margin(%) + 0.99     (R² = 0.88)
```
Each 1pp of margin translates to ~$0.47 in EPS. The +0.99 intercept reflects non-operating income.

**Combined (Model III):**
```
EPS ≈ 0.33 × WTI - 7.68
```
Every $10/bbl change in WTI → ~$3.30 change in annual EPS.

**Breakeven:** WTI ≈ $23/bbl (EPS goes to zero).

**Strengths:** Uses annual data which smooths out quarterly noise. Two-stage model is interpretable — you can see margin as an intermediate step.
**Weaknesses:** Only 4 annual data points. The 2024 Endeavor merger changed FANG's share count (~180M → ~290M shares), creating a structural break in EPS.

---

## Data Editable vs Ineditable

Each method displays a badge indicating whether its underlying parameters can be modified:

| Methods | Badge | Meaning |
|---------|-------|---------|
| **M1–M4** | **Data Editable** (green) | Parameters can be edited in the Data Checklist. Adjust assumptions like gas price, bias correction, production volumes, etc. |
| **M5–M6** | **Data Ineditable** (red) | Parameters are derived from historical regression and cannot be edited. They are displayed in the Data Checklist for reference but inputs are disabled. |

**Why?** M5 and M6 are purely empirical models fitted to historical data. Their coefficients (slopes, intercepts, R²) are fixed regression outputs — editing them would break the statistical validity of the model. M1–M4 use assumption-based inputs that legitimately vary with market conditions.

---

## Statistical Summary

After you select which methods to include, the calculator computes:

| Metric | Formula | What It Means |
|--------|---------|---------------|
| **Mean** | Average of selected methods' EPS | Best single-point estimate |
| **Std Dev (SD)** | Sample standard deviation (n-1) | How much the methods disagree |
| **68% CI** | Mean ± 1 SD | ~68% chance EPS falls in this range under t-distribution |
| **95% CI** | Mean ± t × SE (t-distribution) | 95% confidence interval, uses t critical value for df = n-1 |

The t-distribution critical value adjusts automatically based on how many methods you select (df = n-1).

---

## Student's t-Distribution Chart

The chart plots a **Student's t-distribution** (not a normal/Gaussian distribution) centered on the mean EPS. We use the t-distribution because:

1. **Small sample size:** With only 2–6 methods, the normal distribution underestimates tail risk. The t-distribution has heavier tails, properly reflecting the greater uncertainty that comes with few data points.
2. **Degrees of freedom (df = n−1):** The df is displayed in the chart title and adjusts automatically as you select/deselect methods. Lower df → fatter tails → wider uncertainty.
3. **Converges to normal:** As you select more methods (higher df), the t-distribution approaches the normal distribution. At df ≥ 30 the difference is negligible.

The **red dot**:
- By default: sits at the mean
- Use the slider or input a Target EPS: the dot slides along the t-distribution curve
- Hover to see the **percentile** (computed using the t-distribution CDF, not the normal CDF)

Requires at least 2 methods selected (otherwise SD = 0, no curve to draw).

---

## PE Ratio & Stock Price

Four PE scenarios to convert EPS into an implied stock price:

| Card | PE | Scenario | Rationale |
|------|------|----------|-----------|
| **Grey** | 12x | Bear case | Oil prices expected to drop; market assigns lower multiple to cyclical earnings |
| **Blue** | 15x | Base case | Oil prices stay around current levels; fair value multiple for E&P |
| **Green** | 18x | Bull case | Oil prices expected to rise; market pays premium for growing earnings |
| **Yellow** | Custom | Your call | Enter any PE ratio you think is appropriate |

**Implied Stock Price = Mean EPS × PE Multiple**

---

## Data Validity

All hardcoded parameters are sourced from FANG's SEC filings (10-K, 10-Q) and public data (EIA oil prices). The **validity badge** in the header shows whether the data is still current:
- **Green:** Before the next earnings report (May 4, 2026) — data is up to date
- **Red:** After earnings — new quarterly data may change the parameters

Use the **Data Checklist** to view, edit, and save any parameter. Changes persist in your browser (localStorage).

---

## Limitations & Disclaimers

1. **Small samples:** Methods 5 and 6 rely on 4-5 data points. Statistical significance is limited.
2. **Structural break:** The 2024 Endeavor merger changed FANG's cost structure, production mix, and share count. Pre-merger relationships may not hold.
3. **Linear assumptions:** All models assume linear relationships. Reality has non-linearities (hedging gains/losses, production curtailments at very low prices).
4. **Missing variables:** NGL prices, natural gas prices, hedging positions, capex changes, and tax rate shifts all affect EPS but are held constant or simplified.
5. **Not investment advice:** This is an educational tool for directional estimation. Do your own research.
