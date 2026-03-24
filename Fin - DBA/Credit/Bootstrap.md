To explain how a CDS curve is calibrated on a live trading desk, we have to look at the **Bootstrap** process. On a desk, this happens in milliseconds, but the logic follows a very specific sequence of "solving for the unknown."

### 1. The Goal: The Hazard Rate Curve

We want to find the **Hazard Rate ($\lambda$)**. This is the mathematical representation of the probability of default at any given moment.

- We don't assume $\lambda$ is a single number for all time.
    
- We assume it is **piece-wise constant**. This means $\lambda$ is flat between market pillars (e.g., 6M, 1Y, 3Y) but "jumps" to a new level at each pillar.
    

---

### 2. The Step-by-Step Calibration Process

#### **Step 1: Data Gathering (The Inputs)**

The desk pulls the following from the terminal:

- **The Term Structure of Interest Rates:** Usually the OIS (Overnight Index Swap) curve or LIBOR/SOFR. This gives us the **Risk-Free Discount Factor $P(0, t)$**.
    
- **The Market CDS Quotes:** Quoted spreads ($S_{quoted}$) for standard tenors: 6M, 1Y, 2Y, 3Y, 5Y, 7Y, 10Y.
    
- **Recovery Rate ($R$):** Usually a standard assumption (e.g., 40% for senior unsecured debt).
    

#### **Step 2: Conversion to Upfronts**

Because the desk quotes in **Quoted Spreads**, but the math requires **Present Value (PV)**, each quote is converted into an **Upfront Payment** using the ISDA Standard Model.

$$Upfront_T = (S_{quoted, T} - S_{coupon}) \times RPV01_{ISDA}(S_{quoted, T})$$

This ensures that every market quote is now expressed as a "dollar" value (PV).

#### **Step 3: Bootstrapping the First Pillar (6M)**

We start with the shortest maturity. At 6 months ($T_1$), we only have one unknown: the hazard rate $\lambda_1$.

We set the **PV of the Premium Leg** equal to the **PV of the Protection Leg** (plus the Upfront).

The formula for the Protection Leg (the "Insurance" payout) is:

$$PV_{prot} = (1 - R) \int_{0}^{T_1} P(0, t) \cdot Q(0, t) \cdot \lambda(t) \, dt$$

Where $Q(0, t)$ is the **Survival Probability**: $Q(0, t) = e^{-\int_{0}^{t} \lambda(u) du}$.

Since this is the first pillar, we solve for the $\lambda_1$ that makes the equation match the market 6M Upfront.

#### **Step 4: The Recursive Leap (1Y, 2Y... 10Y)**

Now we move to the 1-year pillar ($T_2$).

- We already know $\lambda_1$ for the period $[0, 6M]$.
    
- We now need to find $\lambda_2$ for the period $[6M, 1Y]$.
    
- We "plug in" the known $\lambda_1$ for the first half of the integral and solve for the $\lambda_2$ that makes the total 1Y CDS PV match the 1Y market quote.
    

---

### 3. The Intuition: Building a Ladder

Imagine you are building a ladder.

1. You fix the first rung (6M) based on how high the ground is right there.
    
2. To fix the second rung (1Y), you can't move the first rung—it's already bolted in. You can only adjust the distance between the first and second rung to reach the total height required by the 1Y market price.
    
3. You repeat this until you reach 10 years. This ensures the curve is **internally consistent**—the 5Y price "knows" about the 1Y and 3Y prices.
    

---

### 4. Why the Stack Exchange user was confused (Detailed Math)

The user was comparing a **Par Spread** to a **Quoted Spread**.

- **Quoted Spread:** Assumes a single $\lambda$ for the whole 6 months.
    
- **Par Spread:** In their data, there was a "hidden" 3-month quote. The Bootstrap model was using $\lambda_1$ for $[0, 3M]$ and solving for $\lambda_2$ for $[3M, 6M]$.
    
- Because the Par model had two "rungs" in the first 6 months and the Quoted model only had one, the "average" height (the spread) came out differently.
    

---

### 6. Synthetic Comparison: Curve Calibration Methods

| **Feature**             | **Standard "Quoted" Model**                     | **Desk "Bootstrap" Model**                        |
| ----------------------- | ----------------------------------------------- | ------------------------------------------------- |
| **Pillar Independence** | Each tenor is calculated in a vacuum.           | Each tenor depends on all shorter tenors.         |
| **Hazard Rate Shape**   | **Flat:** $\lambda$ is a horizontal line.       | **Piece-wise:** $\lambda$ looks like a staircase. |
| **Accuracy**            | Good for quick quotes and cash settlement.      | Essential for risk management and Greeks.         |
| **Input**               | Only uses the quote for that specific maturity. | Uses the entire term structure of CDS and Rates.  |

### Further Generalization: What about "Curve Slopes"?

On a desk, traders watch the "slope" of this calibrated curve.

- **Normal Curve:** $\lambda$ increases over time (the future is more uncertain than today).
    
- **Inverted Curve:** $\lambda$ is very high for 6M and lower for 5Y. This usually means a company is in **distress**—it might blow up tomorrow, but if it survives the next 6 months, it might actually be okay. The Bootstrap process is the only way to accurately see this "Front-end stress."
    

**Would you like me to explain how "Greeks" like CS01 (Credit Spread Sensitivity) are calculated once this curve is calibrated?**

To explain **CS01** (Credit Spread 01), we first need to understand that once the desk has calibrated the hazard curve ($\lambda$), they don't just sit there—they need to know how much money they lose if the market moves by a single "tick" (1 basis point).

### 1. The Definition of CS01

**CS01** (also called **DV01** or **CR01** in some contexts) is the change in the **Market Value (PV)** of a CDS position for a **1 basis point (0.01%) parallel shift** in the credit spread curve.

- If you are **Long Protection** (you bought insurance), you want the spread to go **up** (the company gets riskier). Your CS01 tells you how much your profit increases per bp.
    
- If you are **Short Protection** (you sold insurance), a spread increase is a loss.
    

---

### 2. The Step-by-Step Calculation (The "Bump and Re-price")

A trading desk calculates CS01 using a numerical "sensitivity" approach. It isn't a simple derivative formula because the curve is a complex, piece-wise structure.

#### **Step A: The Base Case**

The system calculates the current PV ($PV_{base}$) using the market spreads ($S_{6M}, S_{1Y}, S_{3Y}, \dots$).

#### **Step B: The "Bump"**

The system creates a "shocked" version of the market. It adds **1 basis point** to **every** pillar on the curve simultaneously.

- $S_{6M}^{new} = S_{6M} + 0.0001$
    
- $S_{1Y}^{new} = S_{1Y} + 0.0001$
    
- ...and so on.
    

#### **Step C: Re-calibration (The "Boring" Detail)**

This is the most critical part. The system **re-bootstraps** the entire hazard curve using these new "bumped" spreads. This generates a slightly higher set of hazard rates ($\lambda_{new}$).

#### **Step D: Re-pricing**

The system re-calculates the PV of your specific trade ($PV_{bumped}$) using the new $\lambda_{new}$ curve.

#### **Step E: The Result**

The difference is your CS01:

$$CS01 = PV_{bumped} - PV_{base}$$

---

### 3. The Intuition: "Credit Duration"

CS01 is essentially the **duration** of a credit contract.

- A **5Y CDS** typically has a CS01 of about **4.5 to 4.8**.
    
- This means if the spread moves from 100bps to 101bps, the value of a $10M contract changes by roughly **$4,500**.
    

Why is it less than 5? Because there is a chance the company defaults _before_ year 5. If they default in Year 2, you stop paying premiums. Therefore, the "expected" life of the payments is shorter than the actual maturity.

---

### 4. Advanced "Tenor CS01" (The Bucket Risk)

On a real desk, a "Parallel CS01" isn't enough. If the 1Y spread explodes but the 10Y spread stays flat, a parallel shift won't tell you your true risk. Traders look at **Bucketed CS01**:

1. Bump **only** the 1Y pillar.
    
2. Re-bootstrap and re-price.
    
3. This shows you exactly which part of the curve you are "exposed" to.
    

---

### 5. Comparative Table: Parallel vs. Bucketed Risk

|**Feature**|**Parallel CS01**|**Bucketed (Tenor) CS01**|
|---|---|---|
|**Action**|Bump the whole curve up by 1bp.|Bump only one maturity (e.g., 5Y).|
|**Desk Use**|Measuring total "Macro" risk.|Hedging specific curve "twists" or "steepening."|
|**Analogy**|Feeling the room get 1 degree warmer everywhere.|Feeling a heat lamp pointing only at your feet.|

---

### 6. Further Generalization: Convexity (Gamma)

The relationship between Spread and PV is **not a straight line**; it's a curve.

As spreads go higher, the CS01 actually **decreases**. This is because as the probability of default approaches 100%, an extra 1bp of "spread" doesn't matter much—the company is already a "dead man walking." This change in CS01 is known as **Credit Gamma** or **Convexity**, and desks monitor it closely during market crashes to avoid "under-hedging."

**Would you like me to show you a synthetic table of how CS01 changes across different tenors (1Y vs 5Y vs 10Y) for a typical high-yield name?**