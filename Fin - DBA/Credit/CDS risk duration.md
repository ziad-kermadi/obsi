In the world of Credit Default Swaps (CDS), **Risk Duration** (often called **Risky PV01**, **RPV01**, or **Risky Duration**) is the most critical metric for managing a position.

Unlike a standard Treasury bond, where duration is primarily a function of interest rates, **CDS Risk Duration measures the sensitivity of the contract's value to a 1 basis point change in the credit spread.**

---

## 1. The Intuition: Probability-Weighted Time

In a risk-free bond, you are certain to receive all future coupons. In a CDS, you only receive (or pay) the premium **as long as the reference entity has not defaulted.**

Therefore, Risk Duration is the **expected present value of a 1bp stream of premiums paid until the earlier of maturity or default.**

- **Higher Credit Risk = Lower Duration:** If a company is likely to default soon, you don't expect to be paying/receiving premiums for very long. The "expected life" of the contract shrinks, which lowers the duration.
    
- **Lower Credit Risk = Higher Duration:** If a company is "AAA" rated, you expect to pay/receive premiums for the full 5 years. The duration will be very close to the actual time to maturity.
    

---

## 2. The Formula: The Math Behind the Sensitivity

The value of a CDS is the difference between the **Premium Leg** (the coupons) and the **Protection Leg** (the contingent payment upon default). Risk Duration ($\text{RD}$) focuses on the Premium Leg.

If we let $P(t)$ be the probability of survival until time $t$, and $Z(t)$ be the discount factor (interest rates), the Risk Duration is:

$$\text{RD} = \int_{0}^{T} P(t) \cdot Z(t) \, dt$$

In practice, we use this to calculate the change in the Mark-to-Market (MTM) value:

$$\Delta \text{MTM} \approx \text{Notional} \times \text{RD} \times \Delta \text{Spread}$$

---

## 3. The Nuances: Why it’s not "Normal" Duration

### A. The Negative Gamma Connection

As credit spreads widen (the company gets riskier), the probability of survival ($P(t)$) drops. This causes the Risk Duration to **decrease**.

- When spreads are low, RD is high.
    
- When spreads are high, RD is low.
    

This is the mathematical root of the **negative gamma** we discussed earlier. If you have sold protection and spreads widen, your "delta" (RD) is actually shrinking as the price falls, but your JTD (Jump-to-Default) risk is exploding. You are losing "sensitivity" to small spread moves just as you are becoming most vulnerable to the "Big Jump."

### B. Interest Rate Sensitivity (IR01 vs. CS01)

A CDS has two durations:

1. **CS01 (Credit Spread Sensitivity):** How much you lose/gain if the credit spread moves 1bp.
    
2. **IR01 (Interest Rate Sensitivity):** How much you lose/gain if the risk-free yield curve moves 1bp.
    

Because $Z(t)$ (the discount factor) is inside the integral, higher interest rates reduce the present value of future premiums, slightly lowering the Risk Duration. However, in credit markets, the **CS01 is usually 10x to 50x more impactful** than the IR01.

### C. The "Default Accrual" Nuance

If a company defaults halfway through a quarter, the protection seller is still entitled to the premium earned up to that day. Sophisticated RD models include this "accrued premium" logic, which slightly increases the duration compared to a simple bond.

---

## 4. Worked Numerical Example

Let’s calculate the **CS01** (the dollar value of the Risk Duration) for a trade.

**Inputs:**

- **Notional:** $100,000,000
    
- **Maturity:** 5 Years
    
- **Current Risk Duration (RD):** 4.2 (Note: It’s less than 5 because of default probability and discounting).
    

**Scenario:** The credit spread widens by **5 bps**.

**Calculation:**

1. **Find the dollar value of 1bp (CS01):**
    
    $$\text{CS01} = \text{Notional} \times \text{RD} \times 0.0001$$
    
    $$\text{CS01} = 100,000,000 \times 4.2 \times 0.0001 = \$42,000$$
    
2. **Calculate the Total MTM Change:**
    
    $$\text{MTM Loss} = \text{CS01} \times \Delta\text{Spread}$$
    
    $$\text{MTM Loss} = \$42,000 \times 5 = \$210,000$$
    

---

## 5. Synthetic Summary Table

|**Feature**|**Interest Rate Duration (Bond)**|**Risk Duration (CDS)**|
|---|---|---|
|**Measures Sensitivity To**|Benchmark Yields ($y$)|Credit Spreads ($s$)|
|**Impact of Higher Risk**|Minimal change to duration.|**Duration decreases significantly.**|
|**Max Value**|Close to Maturity ($T$).|Always less than Maturity ($T$).|
|**Primary Driver**|Time and Coupon rate.|**Default Probability ($P_{def}$).**|
|**Price Relation**|Convex (Positive Gamma).|**Concave (Negative Gamma).**|

---

## Further Generalizations

1. **Curve Risk:** A 5-year CDS doesn't just have one duration; it has sensitivity to the 1Y, 2Y, 3Y, and 5Y points of the credit curve. This is called "Key Rate Credit Duration."
    
2. **Credit Convexity:** Because RD changes as spreads change, for large spread moves (e.g., 500bps), the simple RD calculation will be wrong. You must include a "Convexity Adjustment" to account for the fact that the duration is shrinking as the spread widens.
    
