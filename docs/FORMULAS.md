# IBM Equity Dashboard — Calculation Formulas

## 📐 Overview

This document defines every financial calculation used in the application. All formulas are cited with sources and examples using the spoof data.

**Disclaimer**: These formulas are for informational purposes only. Consult a tax professional for actual tax liability and a financial advisor for investment decisions.

---

## 💰 RSU (Restricted Stock Unit) Calculations

### 1. Current RSU Value

**Formula:**
```
RSU Value = (Vested Units + Unvested Units) × Current Market Price
```

**Rationale**: RSUs represent actual shares that will be delivered upon vesting. Both vested and unvested units have value based on current market price.

**Example** (from spoof data):
```
Grant RSU-2021-00184:
- Vested Units: 180
- Unvested Units: 0
- Current Price: $247.85
- Value = (180 + 0) × $247.85 = $44,613.00
```

**Total RSU Value** (all grants):
```
Total = Σ(Vested Units + Unvested Units) × Current Price
      = 1,510 × $247.85
      = $374,253.50
```

---

### 2. RSU Vested vs. Unvested Breakdown

**Formula:**
```
Vested Value = Vested Units × Current Price
Unvested Value = Unvested Units × Current Price
```

**Example**:
```
Grant RSU-2024-01355:
- Vested Units: 105
- Unvested Units: 315
- Current Price: $247.85

Vested Value = 105 × $247.85 = $26,024.25
Unvested Value = 315 × $247.85 = $78,072.75
Total Value = $104,097.00
```

---

### 3. RSU Grant-Date Fair Value

**Formula:**
```
Grant-Date Value = Units Granted × Grant Price
```

**Purpose**: Used for calculating unrealized gains and IRR.

**Example**:
```
Grant RSU-2023-00912:
- Units Granted: 310
- Grant Price: $141.75
- Grant-Date Value = 310 × $141.75 = $43,942.50
```

---

### 4. RSU Unrealized Gain/Loss

**Formula:**
```
Unrealized Gain = Current Value - Grant-Date Value
Gain % = (Current Value / Grant-Date Value - 1) × 100%
```

**Example**:
```
Grant RSU-2023-00912:
- Current Value: 310 × $247.85 = $76,833.50
- Grant-Date Value: 310 × $141.75 = $43,942.50
- Unrealized Gain = $76,833.50 - $43,942.50 = $32,891.00
- Gain % = ($76,833.50 / $43,942.50 - 1) × 100% = 74.9%
```

---

## 📊 Stock Option Calculations

### 5. Intrinsic Value (In-the-Money Options)

**Formula:**
```
Intrinsic Value = MAX(0, Current Price - Strike Price) × Vested Options
```

**Rationale**: Intrinsic value is the profit you would realize if you exercised the options today and immediately sold the shares.

**Example** (from spoof data):
```
Grant NQ-2020-00091:
- Strike Price: $119.30
- Vested Options: 800
- Current Price: $247.85
- Intrinsic Value = MAX(0, $247.85 - $119.30) × 800
                  = $128.55 × 800
                  = $102,840.00

Note: Spoof data shows $77,130 because only 600 options are outstanding
(200 were exercised). Correct calculation:
Intrinsic Value = $128.55 × 600 = $77,130.00 ✓
```

---

### 6. Out-of-the-Money Options

**Formula:**
```
If Current Price < Strike Price:
    Intrinsic Value = $0
```

**Example**:
```
Hypothetical grant with Strike Price = $300:
- Current Price: $247.85
- Intrinsic Value = MAX(0, $247.85 - $300) × Vested
                  = $0 (underwater options have no intrinsic value)
```

---

### 7. Time Value (Not Implemented in MVP)

**Formula:**
```
Time Value = Option Premium - Intrinsic Value
```

**Note**: Requires Black-Scholes model or similar. Not included in MVP since we don't have option premiums. Future enhancement.

---

### 8. Total Options Intrinsic Value

**Formula:**
```
Total = Σ MAX(0, Current Price - Strike Price) × Vested Options
```

**Example** (from spoof data):
```
NQ-2020-00091: ($247.85 - $119.30) × 600 = $77,130.00
NQ-2022-00318: ($247.85 - $128.20) × 450 = $53,842.50
NQ-2024-00644: ($247.85 - $188.40) × 125 = $7,431.25
Total = $138,403.75 ✓
```

---

## 🏦 ESPP (Employee Stock Purchase Plan) Calculations

### 9. ESPP Current Value

**Formula:**
```
Current Value = Shares Held × Current Price
```

**Example** (from spoof data):
```
Purchase 2023-06-30:
- Shares Held: 38
- Current Price: $247.85
- Current Value = 38 × $247.85 = $9,418.30 ✓
```

---

### 10. ESPP Unrealized Gain

**Formula:**
```
Unrealized Gain = Current Value - Total Contribution
Gain % = (Current Value / Total Contribution - 1) × 100%
```

**Example** (from spoof data):
```
All ESPP Purchases:
- Total Contribution: $30,000
- Current Value: 200 × $247.85 = $49,570.00
- Unrealized Gain = $49,570 - $30,000 = $19,570.00
- Gain % = ($49,570 / $30,000 - 1) × 100% = 65.2%
```

---

### 11. ESPP Discount Benefit (Historical)

**Formula:**
```
Discount = (FMV at Purchase - Purchase Price) × Shares Purchased
Discount % = (FMV at Purchase - Purchase Price) / FMV at Purchase × 100%
```

**Rationale**: ESPP typically offers 15% discount off the lower of offering-date or purchase-date FMV.

**Example**:
```
Purchase 2023-06-30:
- FMV at Purchase: $139.05
- Purchase Price: $118.20
- Shares Purchased: 38
- Discount = ($139.05 - $118.20) × 38 = $791.30
- Discount % = ($139.05 - $118.20) / $139.05 × 100% = 15.0% ✓
```

---

## 📅 Vesting Schedule Calculations

### 12. Estimated Fair Market Value at Vest

**Formula:**
```
Estimated FMV = Current Price (or user-specified forecast)
```

**Note**: For future vests, we use current price as estimate. User can override with "what-if" price.

**Example**:
```
Upcoming vest on 2027-04-15:
- Units: 77
- Estimated FMV: $247.85 (current price)
- Estimated Gross Value = 77 × $247.85 = $19,084.45
```

---

### 13. Tax Withholding Estimate

**Formula:**
```
Tax Withheld = Gross Value × Tax Rate
Net Value = Gross Value - Tax Withheld
Shares Withheld = FLOOR(Tax Withheld / FMV at Vest)
```

**Default Tax Rate**: 37% (22% federal supplemental + 15.3% FICA, rounded)

**⚠️ IMPORTANT**: This is an estimate only. Actual withholding depends on:
- Your W-4 elections
- State/local taxes
- FICA cap ($160,200 for 2023)
- Your total income for the year

**Example** (from spoof data):
```
Vest on 2026-04-15 (RSU-2023-00912, Tranche 3/4):
- Units: 77
- Estimated FMV: $247.85
- Gross Value = 77 × $247.85 = $19,084.45
- Tax Withheld = $19,084.45 × 0.37 = $7,061.25
- Net Value = $19,084.45 - $7,061.25 = $12,023.20
- Shares Withheld = FLOOR($7,061.25 / $247.85) = 28 shares

Spoof data shows $7,061.2465 (more precise), which matches ✓
```

---

### 14. Configurable Tax Rate

**Formula:**
```
User can override default 37% with their actual marginal rate:
- Federal: 10%, 12%, 22%, 24%, 32%, 35%, 37% (2023 brackets)
- State: varies by state (0% to 13.3%)
- FICA: 7.65% (up to wage base)

Recommended: Use 37% as conservative estimate, then consult CPA
```

---

## 📈 Portfolio-Level Calculations

### 15. Total Equity Value

**Formula:**
```
Total Equity Value = RSU Value + Options Intrinsic Value + ESPP Value
```

**Example** (from spoof data):
```
RSU Value: $374,253.50
Options Intrinsic Value: $138,403.75
ESPP Value: $49,570.00
Total = $562,227.25 ✓
```

---

### 16. Today's Gain/Loss (Future Feature)

**Formula:**
```
Today's Gain = (Current Price - Yesterday's Close) × Total Shares
Today's Gain % = (Current Price / Yesterday's Close - 1) × 100%

Total Shares = RSU Units + ESPP Shares + Exercised Options
```

**Example**:
```
If IBM closed yesterday at $245.00 and is now $247.85:
- Total Shares: 1,510 (RSU) + 200 (ESPP) = 1,710
- Today's Gain = ($247.85 - $245.00) × 1,710 = $4,873.50
- Today's Gain % = ($247.85 / $245.00 - 1) × 100% = 1.16%
```

---

### 17. Unrealized Gain/Loss (Portfolio)

**Formula:**
```
Total Unrealized Gain = Current Value - Cost Basis

Cost Basis = Σ(RSU Grant-Date Values) + Σ(Option Strike Prices × Exercised) + Σ(ESPP Contributions)
```

**Example**:
```
RSU Cost Basis:
- RSU-2021-00184: 180 × $134.50 = $24,210
- RSU-2022-00477: 240 × $128.20 = $30,768
- RSU-2023-00912: 310 × $141.75 = $43,942.50
- RSU-2024-01355: 420 × $188.40 = $79,128
- RSU-2025-01821: 360 × $224.10 = $80,676
Total RSU Cost Basis = $258,724.50

Options Cost Basis (exercised only):
- NQ-2020-00091: 200 × $119.30 = $23,860

ESPP Cost Basis:
- Total Contributions: $30,000

Total Cost Basis = $258,724.50 + $23,860 + $30,000 = $312,584.50

Current Value = $562,227.25
Unrealized Gain = $562,227.25 - $312,584.50 = $249,642.75
Gain % = ($562,227.25 / $312,584.50 - 1) × 100% = 79.9%
```

---

## 🎯 What-If Modeling

### 18. What-If Total Value

**Formula:**
```
What-If Value(P) = RSU Value(P) + Options Value(P) + ESPP Value(P)

Where P = hypothetical price

RSU Value(P) = Total RSU Units × P
Options Value(P) = Σ MAX(0, P - Strike Price) × Vested Options
ESPP Value(P) = Total ESPP Shares × P
```

**Example**:
```
What if IBM trades at $300?

RSU Value = 1,510 × $300 = $453,000
Options Value:
  - NQ-2020-00091: ($300 - $119.30) × 600 = $108,420
  - NQ-2022-00318: ($300 - $128.20) × 450 = $77,310
  - NQ-2024-00644: ($300 - $188.40) × 125 = $13,950
  Total Options = $199,680
ESPP Value = 200 × $300 = $60,000

Total What-If Value = $453,000 + $199,680 + $60,000 = $712,680

Gain from current: $712,680 - $562,227.25 = $150,452.75 (+26.8%)
```

---

## 📊 Advanced Analytics (v2.0)

### 19. Internal Rate of Return (IRR)

**Formula:**
```
IRR = (Current Value / Grant-Date Value) ^ (1 / Years) - 1

Where Years = (Today - Grant Date) / 365.25
```

**Example**:
```
Grant RSU-2021-00184:
- Grant Date: 2021-04-15
- Grant-Date Value: 180 × $134.50 = $24,210
- Current Value: 180 × $247.85 = $44,613
- Years: (2026-05-26 - 2021-04-15) / 365.25 = 5.12 years
- IRR = ($44,613 / $24,210) ^ (1 / 5.12) - 1
      = 1.8423 ^ 0.1953 - 1
      = 1.1268 - 1
      = 12.68% annualized return
```

---

### 20. Cost Basis (for Tax Reporting)

**Formula:**
```
RSU Cost Basis = FMV at Vest (reported as W-2 income)
Option Cost Basis = Strike Price × Shares Exercised
ESPP Cost Basis = Purchase Price × Shares Purchased

For capital gains:
Short-term: Held < 1 year from vest/exercise
Long-term: Held ≥ 1 year from vest/exercise
```

**Example**:
```
RSU vested on 2025-04-15 at FMV $178.40:
- 45 shares vested
- Cost Basis = 45 × $178.40 = $8,028 (reported on W-2)
- If sold on 2026-05-26 at $247.85:
  - Holding Period = 1.12 years (long-term)
  - Capital Gain = (45 × $247.85) - $8,028 = $3,125.25
  - Tax Rate = 15% or 20% (long-term capital gains)
```

---

### 21. Dividend Projections (Future)

**Formula:**
```
Annual Dividend Income = Total Shares × Dividend per Share

Where Total Shares = Vested RSUs + ESPP Shares + Exercised Options
```

**Example**:
```
From spoof data:
- Dividend per Share: $6.80 (annual)
- Total Shares: 620 (vested RSUs) + 200 (ESPP) = 820
- Annual Dividend = 820 × $6.80 = $5,576.00
- Quarterly Dividend = $5,576 / 4 = $1,394.00
```

---

## 🧮 Validation Rules

### Data Integrity Checks:

1. **RSU Totals**:
   ```
   Units Granted = Vested + Unvested + Cancelled
   ```

2. **Options Totals**:
   ```
   Options Granted = Vested + Unvested + Exercised
   Outstanding = Granted - Exercised
   ```

3. **Vesting Schedule**:
   ```
   Σ(Tranche Units) = Units Granted (for each grant)
   ```

4. **Price Sanity Checks**:
   ```
   Current Price > $0
   Strike Price > $0
   Grant Price > $0
   Current Price should be within 50% of 52-week range
   ```

5. **Date Validations**:
   ```
   Grant Date < Vest Date
   Vest Date < Expiration Date (for options)
   Grant Date ≤ Today
   ```

---

## 📚 References

1. **IRS Publication 525** - Taxable and Nontaxable Income
   - RSU taxation: https://www.irs.gov/publications/p525#en_US_2022_publink1000229143

2. **IRS Publication 15** - Employer's Tax Guide
   - Supplemental wage withholding: https://www.irs.gov/publications/p15#en_US_2023_publink1000202352

3. **SEC Rule 10b-5** - Insider Trading
   - https://www.sec.gov/rules-regulations/1934-act-rules/rule-10b-5

4. **ESPP Taxation** - IRS Notice 2009-67
   - https://www.irs.gov/irb/2009-39_IRB#NOT-2009-67

5. **Stock Option Valuation** - Black-Scholes Model
   - https://en.wikipedia.org/wiki/Black%E2%80%93Scholes_model

---

## ⚠️ Important Disclaimers

1. **Tax Estimates**: All tax calculations are estimates only. Actual tax liability depends on your total income, deductions, state/local taxes, and other factors. Consult a CPA or tax professional.

2. **Investment Advice**: This application does not provide investment advice. Consult a financial advisor before making investment decisions.

3. **Accuracy**: While we strive for accuracy, calculation errors may occur. Always verify important calculations independently.

4. **Compliance**: You are responsible for complying with IBM's insider trading policy, SEC regulations, and Morgan Stanley's Terms of Service.

5. **Market Data**: Stock prices are delayed and may not reflect real-time values. Do not use for trading decisions.

---

## ✅ Formula Verification Checklist

When implementing or modifying calculations:

- [ ] Formula matches this documentation
- [ ] Formula tested with spoof data
- [ ] Edge cases handled (zero values, negative prices, etc.)
- [ ] Units are consistent (shares vs. dollars)
- [ ] Rounding is appropriate (2 decimal places for currency)
- [ ] Formula cited with source (IRS, SEC, etc.)
- [ ] Example calculation provided
- [ ] User-facing disclaimer added if needed

---

**Last Updated**: 2026-05-26  
**Version**: 1.0 (MVP)