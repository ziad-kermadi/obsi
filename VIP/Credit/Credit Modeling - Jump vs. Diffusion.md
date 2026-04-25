This image summarizes the fundamental paradigm of modern credit modeling, which treats credit not as a "volatile" asset like a stock, but as a regime-switching instrument. It transitions from "alive" to "defaulted" in a single, discrete jump.

Here is the step-by-step breakdown of the concepts, the math, and the underlying intuition.

---

## 1. The Fundamental Identity

The core formula presented is:

$$s_t = \lambda_t \times LGD_t$$

Where:

- **$s_t$ (Credit Spread):** The extra yield investors demand over a risk-free rate to compensate for credit risk.
    
- **$\lambda_t$ (Hazard Rate/Default Intensity):** The instantaneous probability of default. Think of this as the "frequency" of the event.
    
- **$LGD_t$ (Loss Given Default):** The percentage of the face value lost if a default occurs. It is defined as $1 - \text{Recovery Rate}$.
    

### The Intuition

In equity, you care about **volatility**—how much the price wiggles. In credit, you care about the **jump**.

Imagine you are insuring a house against fire. You don't care if the house's market value fluctuates by 2% every day (diffusion). You care about the probability of it burning down (the jump) and how much the insurance will have to pay out if it does (the recovery). Credit pricing is essentially an insurance premium calculation.

---

## 2. Credit vs. Equity (Diffusion vs. Jump)

The image highlights a critical distinction used in quantitative finance:

- **Equity is Diffusion (Geometric Brownian Motion):** Stock prices move in continuous, small increments. You can theoretically hedge or sell a stock as it drops.
    
- **Credit is a Binary Jump (Poisson Process):** A bond is either paying its coupons, or it isn't. When a "Jump to Default" (JTD) occurs, the price doesn't slide down gracefully; it gaps down instantly from, say, $95$ cents on the dollar to its recovery value of $40$ cents.
    

**Why Carry Trades Fail:**

A "carry trade" involves buying a high-yield bond (collecting the spread $s$) and financing it at a low rate.

- **The Trap:** You might earn $5\%$ a year in "carry."
    
- **The Reality:** If the bond defaults (the jump), you lose $60\%$ of your principal ($LGD$) instantly.
    
- **The Math:** It would take **12 years** of collecting that $5\%$ spread to make up for a single day's jump-to-default loss.
    

---

## 3. CDS vs. Cash Bonds

A **Credit Default Swap (CDS)** is a derivative that isolates "pure" credit risk. A **Bond** is a cash instrument.

- **Bond Spread:** Includes credit risk + liquidity risk + the cost of borrowing money to buy the bond (Repo rates).
    
- **CDS Spread:** Purely the cost of protection against the "jump."
    
    The difference between the two is known as the **Basis**.
    

---

## 4. Worked Example: Pricing a Spread

Let's calculate the fair credit spread for a corporate bond.

**Assumptions:**

1. **Probability of Default ($\lambda$):** 3% per year.
    
2. **Expected Recovery Rate ($R$):** 40%.
    
3. **$LGD$:** $1 - 0.40 = 0.60$ (or 60%).
    

**The Calculation:**

$$s = 0.03 \times 0.60 = 0.018$$

To convert this to **Basis Points (bps)**, we multiply by 10,000:

$$0.018 \times 10,000 = 180 \text{ bps}$$

**Result:** You should demand at least **180 bps** over the risk-free rate to break even on the expected loss. If the market is offering 150 bps, you are being underpaid for the risk.

---

## 5. Further Generalizations & Related Concepts

### Stochastic Recovery

The image mentions that "Recovery dominates spread moves in crisis." This is because $LGD$ is not a constant. In a systemic crash, many companies default at once. When the market is flooded with defaulted debt, the "Recovery Rate" drops because there are fewer buyers for distressed assets. This is known as **PD/LGD Correlation**.

### Default Clustering

Defaults are not independent. They cluster by sector (e.g., energy prices hitting all oil companies) or macro-environment (rising interest rates). This is why a portfolio of bonds is riskier than it looks; if one "jumps," the probability of others jumping increases.

---

## Summary Tables

### Credit vs. Equity

|**Feature**|**Equity (Diffusion)**|**Credit (Jump)**|
|---|---|---|
|**Primary Risk**|Volatility ($\sigma$)|Default Intensity ($\lambda$)|
|**Price Movement**|Continuous / Smooth|Discrete / Gapping|
|**Worst Case**|Price goes to zero gradually|Immediate loss of $(1 - \text{Recovery})$|
|**Model Type**|Black-Scholes / GBM|Reduced Form / Poisson|

### The Components of Credit

|**Component**|**What it answers**|**Market driver**|
|---|---|---|
|**Spread ($s$)**|"What is the total risk premium?"|Supply/Demand & Risk Appetite|
|**Intensity ($\lambda$)**|"How likely is the event?"|Earnings, Leverage, Macro factors|
|**LGD ($1-R$)**|"How much is left after the crash?"|Collateral quality, Seniority, Cycle|
