This is a significant step in your career. For a VP-level role at a firm like Rothesay—which manages over **£70 billion** in pension assets —the interviewers will look beyond your ability to code. They want to see if you understand the "why" behind the models, their failure modes in a live production environment, and how they impact the balance sheet.

Since you mentioned being "grilled," these questions are designed to challenge your technical depth and your experience at Deutsche Bank and Société Générale.

---

## 1. Credit Derivatives: CDS Index Options & Tranche Pricing

**The Grilling:** "You've worked on CDS Index Options (iTraxx/CDX) and Tranche pricing using Gaussian Copulas. In a live desk environment, why is the Base Correlation approach fundamentally flawed, and how do you handle the 'Negative Expected Tranche Loss' problem during calibration?"

### The Intuition

The **Gaussian Copula** assumes that defaults are linked via a multivariate normal distribution. However, markets aren't normal; they have fat tails. **Base Correlation** was a "hack" to make the Gaussian Copula fit market prices for different tranches of the same index. The problem is that it treats each tranche in isolation rather than as part of a single, consistent loss distribution.

### Detailed Answer

- **The Flaw:** Base correlation is not a true model of the underlying pool; it is an interpolation scheme. It often leads to **non-monotonicity** in the loss distribution. If your base correlations aren't "stripped" or interpolated perfectly, you can mathematically imply that the expected loss of a senior tranche is lower than it should be, or even negative, which is a physical impossibility.
    
- **Live Context:** On a trading desk, if the index moves sharply, your base correlation curve might "break." You handle this by using **Stochastic Intensity Models** or ensuring your interpolation (e.g., cubic splines on the mapping function) preserves the convexity of the expected loss curve.
    

### Worked Example: Pricing a CDS

To price a CDS, we need the **Hazard Rate** ($\lambda$), representing the instantaneous probability of default.

The survival probability $P(t)$ is given by:

$$P(t) = e^{-\int_0^t \lambda(s) ds}$$

If we assume a constant hazard rate and a constant recovery rate $R$, the break-even spread $S$ is roughly:

$$S \approx \lambda(1 - R)$$

If you are pricing a **CDX Index Option**, you are essentially pricing a "Receiver" or "Payer" option on this spread, requiring a Black-style model or a SABR-type volatility surface for the index spread itself.

---

## 2. Interest Rates: CMS Spread Options & Correlation

**The Grilling:** "You implemented a CMS Spread Option pricing framework. Rothesay cares deeply about long-dated liabilities. When pricing a CMS Spread (e.g., 10y-2y), why can't you just use a simple Black model, and how does the 'Copula' choice for the joint distribution affect your Greek sensitivities?"

### The Intuition

A **CMS Spread Option** depends on two different points on the yield curve. You aren't just betting on rates going up; you are betting on the **slope** of the curve. Because the payoff involves the difference between two rates, you must account for the **correlation** between them. If the correlation is 1.0, the spread never changes; if it is 0.0, the spread is highly volatile.

### Detailed Answer

- **The Problem:** You must use a **Marginal Mapping** approach. You take the individual CMS rate distributions (usually calibrated from the Swaption smile using SABR ) and then "glue" them together using a Copula.
    
- **The Choice:** Using a **Gaussian Copula** for the rates is common, but it misses "tail dependence." If both rates spike (a market crash), the correlation usually increases. On a live desk, if you underprice this "correlation risk," your Delta and Vega hedges will be wrong. You need to perform a **Convexity Adjustment** because the CMS rate is a "rate" paid at a frequency that doesn't match its tenor.
    

---

## 3. Volatility Modeling: SABR & The RFR Transition

**The Grilling:** "You maintained SABR pricing and worked on the IBOR to RFR transition. Since RFRs (like SONIA) can technically go negative, how did you adapt the standard SABR model, and what happens to the 'Backbone' of the model when the shift parameter is poorly chosen?"

### The Intuition

The original SABR model assumes rates are log-normal (cannot be negative). When rates hit zero or go negative, the math breaks (you can't take the log of a negative number). To fix this, we "shift" the distribution.

### Detailed Answer

- **The Shifted SABR:** We replace the forward rate $F$ with $(F + \theta)$, where $\theta$ is the shift.
    
    $$\sigma(F) = \alpha (F + \theta)^\beta$$
    
- **The "Backbone":** This refers to how the ATM (At-The-Money) volatility moves as the forward rate moves. If your shift $\theta$ or your $\beta$ (the elasticity) is wrong, your model will predict that volatility will drop when rates rise (or vice versa) in a way that doesn't match the market.
    
- **Live Context:** At a VP level, you should know that **$\beta$** is often fixed (e.g., $\beta=0.5$ or $\beta=0$) because it is difficult to calibrate simultaneously with the volatility of volatility ($\nu$).
    

---

## 4. Curve Construction: Fitting & Interpolation

**The Grilling:** "Rothesay is looking for someone to maintain and extend curve fitting engines. Given their pension focus, they likely have very long-dated instruments. Why would you prefer **Monotone Convex Splines** over standard **Cubic Splines** for discount factor interpolation?"

### The Intuition

A curve fitting engine is the "source of truth" for the whole bank. If the curve has "wiggles" (artifacts of the math), your risk measures (Greeks) will be "noisy."

### Detailed Answer

- **Cubic Splines:** While smooth, they can "overshoot." A small change in a 10-year rate could mathematically cause a weird dip in the 9-year rate just to keep the curve smooth.
    
- **Monotone Convexity:** This ensures that the **Forward Rates** are continuous and, more importantly, that the discount factors are always decreasing (staying positive). For a pension fund, you cannot have a model that implies negative forward rates in the distant future unless the market explicitly prices them.
    

---

## Comparative Analysis of Models

|**Model**|**Primary Use Case**|**Key Strength**|**Live Desk Limitation/Risk**|
|---|---|---|---|
|**SABR**|Swaption Volatility|Captures the "smile" and "skew"|Fails near zero/negative rates without a shift.|
|**Gaussian Copula**|Credit Tranches|Fast, industry standard|Ignores tail dependence (correlation risk).|
|**Hazard Rate**|CDS Pricing|Direct modeling of default prob|Assumes recovery rate is known and constant.|
|**Hull-White**|Bermudian Swaptions|Mean reverting, analytic|Poor fit to the long-term volatility smile.|

---

## Next Step for Your Interview

Would you like me to generate a **technical implementation plan** for how you would migrate a legacy C++ credit analytics stack to a modern **Rust/Python** hybrid environment, as requested in the Rothesay job description?