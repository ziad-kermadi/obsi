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

![[IMG-20260329214355023.png]]
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

This image describes a core problem in quantitative finance: **Intensity Calibration**.

While the "Bootstrap" method I explained earlier is the standard starting point, it has a major flaw in the real world: it's "noisy." If you follow the market quotes too strictly, your hazard curve ($\lambda(t)$) ends up looking like a jagged, unrealistic staircase that jumps wildly every day.

Here is the breakdown of the "Practitioner Insight" provided in the image.

---

### 1. The Under-Determined Problem

The image notes that you have 7 data points (3M to 10Y) to find 7 parameters (the hazard rates for each period).

- **The Theory:** Mathematically, you have 7 equations and 7 unknowns. You can find a "perfect" fit.
    
- **The Reality:** There are infinitely many "shapes" the curve could take between those 7 points that would all result in the same market prices. If you just let the computer pick any curve that fits, it might choose one with massive, nonsensical spikes.
    

### 2. The Desk Solution: Regularization

To stop the curve from "bouncing wildly," desks use **Regularization**. This is basically telling the math: "Find a curve that fits the market prices, **but make it pretty (smooth).**"

#### **The Regularization Formula**

The image provides this minimization objective:

$$\min_{\lambda} \sum_{i} (CDS_i^{mkt} - CDS_i^{model})^2 + \eta \int_{0}^{T} (\lambda'(t))^2 dt$$

1. **The First Term (The "Fit"):** This ensures the model prices match the market prices as closely as possible.
    
2. **The Second Term (The "Smoothness Penalty"):** This is the magic. It calculates the **derivative** (the slope) of the hazard rate.
    
    - If the curve is flat, the derivative is zero (no penalty).
        
    - If the curve has sharp spikes, the derivative is huge, and the penalty becomes very high.
        
3. **$\eta$ (Eta):** This is the "volume knob." A high $\eta$ creates a very smooth, almost flat curve (at the expense of fitting market prices perfectly). A low $\eta$ fits the market perfectly but results in a jagged curve.
    

---

### 3. The Intuition: The "Ironing" Metaphor

Imagine the market quotes are a series of pegs stuck in a board at different heights.

- **Bootstrapping** is like pulling a string through those pegs. It works, but the string has sharp, awkward angles at every peg.
    
- **Regularization** is like using a stiff rubber band. You still want it to pass near the pegs, but the rubber band’s internal tension naturally resists sharp bends. It forces a "smooth" path that makes more sense economically.
    

---

### 4. The Interview Question Breakdown

The image mentions an interview question about a calibration that "bounces wildly day-to-day."

- **The Cause:** "Overfitting." The model is trying so hard to hit every tiny tick in market noise that it changes the entire shape of the curve every time a quote moves by 0.1bp.
    
- **The Fix 1: $H^1$ Regularization.** This is exactly what the formula above does—it penalizes the "energy" or "slope" of the curve.
    
- **The Fix 2: Parametric Form (e.g., CIR).** Instead of a staircase, you force the curve to follow a specific mathematical shape (like the Cox-Ingersoll-Ross model). This is like saying, "The curve _must_ be a smooth arc; just tell me how high the arc should be."
    

---

### 5. Synthetic Table: Bootstrap vs. Regularized Calibration

|**Feature**|**Simple Bootstrap**|**Regularized Calibration (Desk)**|
|---|---|---|
|**Logic**|Exact match of market quotes.|Trade-off between "fit" and "smoothness."|
|**Curve Shape**|Piece-wise constant (staircase).|Smooth, continuous curve.|
|**Stability**|Very low (bounces with every quote).|High (filters out market noise).|
|**Greeks (CS01)**|Can be "noisy" or inconsistent.|Stable and predictable.|

### Further Generalization: Why do we care about smoothness?

If your hazard curve isn't smooth, your **Greeks (like CS01)** will be garbage. If a 1bp move in the market causes your "staircase" to jump to a completely different level, your risk management system will tell you that you've suddenly gained or lost millions in risk for no real reason. A smooth curve ensures that small market moves result in small, logical risk adjustments.

------

---

## Part 1: What a hazard rate actually is

Before any algorithm, you need to understand what you're solving for. The hazard rate h(t) — also called the default intensity — is the instantaneous conditional probability of default per unit time, _given_ survival to time t.

Formally, if τ is the random default time:

**h(t) = lim_{Δt→0} P(τ ≤ t+Δt | τ > t) / Δt**

This is a hazard function from survival analysis, borrowed wholesale into credit. From h(t) you get everything:

- **Survival probability:** Q(t) = exp(−∫₀ᵗ h(u) du) — probability entity is alive at t
- **Default probability density:** f(t) = h(t) · Q(t) — probability of defaulting _exactly_ at t
- **Cumulative default probability:** P(τ ≤ t) = 1 − Q(t)

The desk doesn't observe h(t) directly. It observes CDS par spreads at quoted tenors. The bootstrap is the inversion: given observed spreads, solve for the piecewise-constant hazard rates that reprice them exactly.

---

## Part 2: The exact market inputs and their meaning

The desk receives a spread quote sheet — here is what it looks like in practice:Every spread on that sheet is a _par_ spread — the coupon that makes that specific maturity CDS worth zero at inception. The bootstrap finds the unique piecewise-constant hazard curve that reprices all of them simultaneously.

---

## Part 3: The piecewise-constant hazard rate model — the exact assumption

The desk uses the simplest model that is consistent: between two consecutive pillar dates tᵢ₋₁ and tᵢ, the hazard rate is a constant hᵢ.

This means:

- Q(t) = Q(tᵢ₋₁) · exp(−hᵢ · (t − tᵢ₋₁)) for t ∈ [tᵢ₋₁, tᵢ]
- Q(0) = 1 always (entity is alive today by definition)
- The hazard rate function is a step function with jumps at pillar dates

This is _not_ the same as saying the hazard rate is constant forever. It is piecewise-constant in the log-survival dimension. The implication is that the default intensity can be different in each interval, but within each interval it's flat.

**Why this convention?** Because it makes the bootstrapping exactly recursive: h₁ is determined solely by the 1Y spread, h₂ is then determined by the 2Y spread _given_ h₁ is already known, and so on. No simultaneous equation systems. Clean and fast.

---

## Part 4: The exact legs — integration in full detail

For a CDS with coupon C (the running spread, in decimal), maturity T, and notional N=1, with recovery rate R:

**The fee leg** collects coupon payments on each period [tᵢ₋₁, tᵢ], but only if the entity has survived to the payment date tᵢ. The PV of the fee leg includes two components:

_Scheduled payments:_ $$\text{PV}_{\text{fee}}^{\text{scheduled}} = C \sum_{i=1}^{n} \Delta t_i \cdot DF(t_i) \cdot Q(t_i)$$

_Accrued on default_ (the stub payment owed if default happens mid-period): $$\text{PV}_{\text{fee}}^{\text{accrued}} = C \sum_{i=1}^{n} \int_{t_{i-1}}^{t_i} (t - t_{i-1}) \cdot DF(t) \cdot h \cdot Q(t) , dt$$

Under piecewise-constant h and flat discount: $$= C \sum_{i=1}^{n} \frac{\Delta t_i}{2} \cdot \overline{DF}_i \cdot [Q(t_{i-1}) - Q(t_i)]$$

where $\overline{DF}_i$ is the discount factor at the midpoint of the period. This mid-period approximation is standard on most desks.

**The protection leg** pays (1−R) at the moment of default. Its PV: $$\text{PV}_{\text{prot}} = (1-R) \int_{0}^{T} DF(t) \cdot h(t) \cdot Q(t) , dt$$

Numerically this integral is computed by summing over the same coupon periods: $$\approx (1-R) \sum_{i=1}^{n} \overline{DF}_i \cdot [Q(t_{i-1}) - Q(t_i)]$$

**At par, fee leg PV = protection leg PV.** Solving this for the hazard rate in each interval is the bootstrap.

---

## Part 5: The bootstrap algorithm — step by step

Here is the exact Newton-Raphson loop as it runs. Step through each pillar:Click any pillar button to inspect the Newton-Raphson convergence for that specific node. Drag the sliders to watch the whole curve reprice live.

![[hazard_bootstrap_stepper.html]]

---

## Part 6: The Newton-Raphson solver in exact detail

Here is precisely what the solver does at each pillar n. At this point, h₁ through h_{n-1} are already known. You need to find hₙ.

**Define the objective function:**

f(hₙ) = PV_protection(hₙ) − PV_fee(hₙ) = 0

where both PVs are computed using the full survival curve — previously solved segments use their known hazard rates, and the current segment [t_{n-1}, tₙ] uses hₙ.

**Initial guess:** The closed-form approximation h₀ = S_n / LGD (which assumes flat discounting and continuous compounding). This is usually within 5–20bps of the solution for normal names, which means Newton converges in 3–5 iterations.

**Newton step:** At each iteration k:

hₙᵏ⁺¹ = hₙᵏ − f(hₙᵏ) / f'(hₙᵏ)

The derivative f'(h) is computed numerically by a finite bump: f'(h) ≈ [f(h·1.001) − f(h)] / (h·0.001). Analytic derivatives are possible but rarely used in production — the numerical bump is stable and fast enough given the problem is 1-dimensional.

**Convergence criterion:** |f(hₙ)| < ε where ε is typically 1e-6 in yield units (i.e., 0.0001 bps). Newton converges in 3–8 iterations for normal names. Distressed names may need 10–15 iterations if the initial guess is far away.

**What breaks it:** If the implied hazard rate is extremely high (distressed name, 1000+ bps), the initial guess S/LGD can overshoot and h becomes negative in the first Newton step. The fix is to clamp h > 0 and add a damped step (taking 50% of the Newton step) for the first few iterations.

---

## Part 7: Interpolation between pillars — what happens for off-pillar dates

The bootstrap gives you hᵢ at the 7 quoted tenors. But a trade might mature on any date — e.g., a 4Y trade, or a 2.5Y trade. You need Q(t) for any t.

**Between two pillar dates:** Use the piecewise-constant rule — the hazard rate in the interval [tᵢ, tᵢ₊₁] is hᵢ₊₁ (the hazard rate solved at the _right_ endpoint). So for t ∈ [3Y, 5Y], you use h₅Y.

Q(t) = Q(3Y) · exp(−h₅Y · (t − 3Y))

**Before the first pillar (t < 6M):** Use h₁ (the 6M hazard rate) all the way back to today. This is the standard desk convention.

**Beyond the last pillar (t > 10Y):** Most systems either extrapolate flat (hold the 10Y hazard rate constant) or linearly extrapolate the log-survival curve. Flat extrapolation is the more conservative and common choice.

**Flat forward hazard rate:** Note this is _flat forward_ in the hazard rate, not in the spread. The two-year par spread implies a hazard rate averaged over 2 years, not the same as the one-year-into-one-year forward hazard rate. The bootstrap does extract the true forward hazard rate in each interval — this is where the information is.

---

## Part 8: The accrued interest on default — why it matters more than people think

The accrued premium correction is the term most textbooks skip, but on the desk it matters for:

- Long-dated trades (10Y): correction is up to 30bps in spread equivalent
- Names trading very wide: the probability of default mid-period is non-negligible
- Marking existing trades mid-period, where the current accrual period is partially elapsed

The mid-period approximation used in practice:

PV(accrued) ≈ C · Σᵢ (Δtᵢ/2) · DF(midᵢ) · [Q(tᵢ₋₁) − Q(tᵢ)]

This treats the default as occurring at the midpoint of each period on average. More accurate systems use the exact integral under piecewise-constant h, which has a closed form:

For the segment [a, b] with hazard h and discount rate r:

![[IMG-20260329220853788.png]]

![[IMG-20260329221001199.png]]

Most production systems use the mid-period approximation for speed. The difference is sub-0.1bp for investment-grade names, but can reach 0.5–1bp for high-yield.

---

## Part 9: Day count and calendar — the unglamorous but critical part---

![[IMG-20260329142601328.png]]

## Part 10: The discount curve — OIS stripping in practice

The bootstrap uses a risk-free discount curve DF(t), which on a modern desk is the OIS curve — SOFR for USD, €STR for EUR, SONIA for GBP. This is itself bootstrapped from OIS swap quotes using the same type of root-finding, but it's a separate process that the credit desk consumes, not builds.

In practice: the credit desk calls the rates desk or pulls from the curve server. The discount factors used are:

**DF(t) = exp(−r(t)·t)** under flat rate approximation (fine for relative value work)

or more precisely from the stripped zero rates:

**DF(t) = product of overnight forward rates compounded from today to t**

The choice matters for long-dated trades. At 10Y, a 5bp error in the discount rate translates to ~0.5% error in DF(10Y), which can be 5–10bps in the implied CDS spread. The desk always uses the strapped zero curve from the swaps desk, never a flat rate, for live trading.

---

## Part 11: What the bootstrapped curve actually tells you — the forward hazard rate

The key insight that the bootstrap delivers is the **forward default intensity** in each interval. The bootstrapped hᵢ is the piecewise-constant hazard rate in [tᵢ₋₁, tᵢ] — it is the market's implied annualised default probability in that specific interval, conditional on survival to tᵢ₋₁.

When the hazard curve is upward sloping (h₁ < h₂ < ... < h₁₀), the market is saying: defaults are more likely to occur in the distant future than the near future. This is the normal shape for investment-grade names — near-term risk is low, long-term accumulates.

When the hazard curve is **inverted** — h₁ > h₂ — this is a serious stress signal. The market is pricing a high probability of near-term default. For distressed names, the 1Y hazard rate can be 3000+ bps while the 5Y is only 1500bps, because the market thinks: if they survive 1 year, they'll probably survive longer.

This forward structure is exactly why you need a proper bootstrap rather than just dividing par spreads by LGD. The 5Y par spread is an average of all the forward hazard rates from year 0 to year 5, weighted by survival probabilities and discount factors. The bootstrap disentangles them.

---

## Part 12: Practical implementation details — what changes between systems

The description above is ISDA-standard. In practice, banks diverge in several ways:

**Integration granularity.** The protection leg integral can be computed at quarterly coupon frequency (4 points per year), monthly (12 per year), or daily (365 per year). ISDA recommends quarterly for the fee leg and either monthly or daily for the protection leg. More points = more accuracy but slower. For distressed names near default, daily integration matters. For IG names, quarterly is indistinguishable.

**Discount curve:** Some older systems still use LIBOR-based discounting (pre-2022). Any system still doing this is wrong and will be out-of-market. SOFR/OIS discounting is mandatory post-Libor transition.

**Recovery rate source:** Some desks use sector-default recovery data from Moody's or S&P (the "Markit recovery rate"). Others use a single flat 40% for all corporates and 25% for sovereigns. The single-name CDS convention is to treat R as a fixed input; the market quotes spread flat given R=40%.

**Interpolation conventions:** Some systems interpolate the log-survival curve linearly between pillars (equivalent to piecewise-constant hazard). Others use cubic spline interpolation on the log-survival, which gives smoother forward hazard rates but can generate oscillations. The desk standard is piecewise-constant hazard (linear log-Q interpolation) — it's the ISDA standard and avoids spurious wiggles.

**Curve bumping for CS01:** The CS01 is computed by parallel-bumping all input spreads by 1bp, re-bootstrapping the entire curve, and revaluing. This is a full re-bootstrap per trade per risk run. On a large book (10,000+ CDS positions, 200+ reference entities), this is the dominant computational cost. Analytic sensitivities exist but most desks still use the bump-and-reprice approach for accuracy.

The entire bootstrap for a single name — 7 pillar tenors, Newton-Raphson at each, quarterly integration — runs in under a millisecond. A full EOD risk run on 200 reference entities with 10 CS01 bumps each is still under a second. Performance is not the constraint; correctness and consistency with ISDA analytics is.

----
Great question — and one that separates a proper production curve build from a naive bootstrap. There are five distinct problems the desk has to solve, and each requires a different technique.

---

## The core tension: exact fit vs. smooth shape

The vanilla piecewise-constant bootstrap fits every pillar spread _exactly_ — by construction. But the resulting forward hazard rate curve is a step function that jumps discontinuously at every pillar date. That creates real problems: off-pillar trades misprice, the risk ladder has cliffs in it, and hedges don't behave continuously as time passes.

The desk has to choose where to sit on the spectrum between _exact fit_ and _smooth shape_, and then enforce a set of hard constraints regardless of where they sit.

---

## Constraint 1: Monotone survival probabilities — the no-arbitrage condition

This is the hardest constraint and the one the bootstrap can violate if spreads are noisy or inverted. The survival probability Q(t) must be monotonically _decreasing_ in t. If Q(3Y) > Q(2Y), you have a negative forward hazard rate in the [2Y, 3Y] interval — which means the model is saying defaults become _less likely_ over time in that window on an absolute basis. That is a risk-neutral arbitrage.

**Why it happens:** Market quotes are composite mid prices assembled from multiple dealers. Bid-offer noise can easily make the 2Y spread wider than the 3Y spread by a few basis points. A naive bootstrap then produces a negative h₃Y. In stress periods this is common across many parts of the curve.Try dragging the 3Y spread above the 5Y spread and watch the violation fire. The three repair strategies show what the desk actually does in practice — clamping is the most common.

![[monotone_positivity_correct (1).html]]

---

## Constraint 2: Positivity of hazard rates

This follows directly from monotonicity, but it is coded as an independent check because the failure mode is different. A negative hazard rate means the model is pricing a _rebate_ for surviving — which is incoherent. In code, it surfaces as Q(t) sometimes exceeding 1.0 for distant tenors, which is catastrophic.

**Production fix:** Every bootstrap loop has an explicit floor: `h = max(h, ε)` where ε is typically 0.01bps (1e-6 in decimal). Some desks use 0.1bps as the floor for operational robustness. This floor does mean the bootstrapped spread at that pillar will not exactly reprice — the pricer accepts this when the market quote is clearly arbitrageable.

---

## Constraint 3: Forward hazard rate smoothness — the hard problem

Even a perfectly monotone piecewise-constant curve has a brutal property: the **forward hazard rate** looks like a staircase. When you bump from one pillar to the next, the forward hazard jumps instantaneously. For a curve with quoted pillars at 1Y, 2Y, 3Y, 5Y, 7Y, 10Y — there are literally no quotes between those nodes, so the piecewise-constant assumption fills in those gaps with flat forwards.

The consequence: trades maturing at 4Y price off h₅Y. Trades at 4.9Y also price off h₅Y. A trade at 5.1Y suddenly uses a completely different hazard rate. The _discontinuity in the forward_ creates hedging discontinuities — a trade rolling from just past a pillar to just before will show a sudden jump in CS01 with no change in the underlying market.

There are four main approaches the desk uses:The right panel is the critical one. The blue step function is what almost every desk uses. The dashed amber cubic spline is smooth but can go _negative_ — visible in the chart. The dotted red monotone convex method guarantees smoothness _and_ positivity.

![[interpolation_comparison.html]]

---

## Constraint 4: The monotone convex method — what it actually does

The monotone convex interpolation, introduced by Hagan and West (2006) for interest rate curves and adapted for credit, works by interpolating on the _instantaneous forward hazard_ rather than on log-Q directly.

The key idea: define the discrete forward hazard rate in interval [tᵢ₋₁, tᵢ] as:

**fᵢ = −log(Q(tᵢ)/Q(tᵢ₋₁)) / (tᵢ − tᵢ₋₁)**

This is already what piecewise-constant h gives you — a flat forward in each interval. Monotone convex then interpolates the fᵢ values using a constrained quadratic scheme that guarantees f(t) ≥ 0 everywhere and is continuous (though not differentiable at pillar dates).

The constraint is: for any sub-interval, the interpolated forward hazard must stay within the bounds set by the adjacent discrete forwards. This prevents oscillation — the pathology that makes cubic splines dangerous for credit curves.

**Why most desks don't use it anyway:** It's more complex to implement, harder to explain to risk management, and the ISDA standard is piecewise-constant. For vanilla CDS trading books, the step-function error is below bid-offer on any given trade. Monotone convex matters most for:

- Bespoke CDS options or credit-linked notes with exotic payoffs sensitive to the path of Q(t)
- XVA calculations where the hazard curve is integrated over many paths
- Structured credit where the full survival curve shape drives tranche pricing

---

## Constraint 5: Handling missing or illiquid pillars

Not every reference entity has quotes at all 7 standard pillars. A small corporate might only have 5Y and 10Y. A distressed name might only have 1Y. The desk has to decide what to do with the gaps.Toggle pillars off to simulate a sparse quote sheet. The triangle markers show where the curve is interpolated vs market-observed. The desk would mark those interpolated nodes as "model price" rather than "market price" in the risk system — a distinction that matters for P&L attribution.

![[interpolation_smoothness_correct.html]]

---

## Constraint 6: Regularisation — when you don't want an exact fit

For illiquid names, the market quotes are noisy composites assembled from 2–3 broker axes. Forcing an exact fit to each pillar propagates noise directly into the hazard curve, creating wild forward hazard rates that flip between pillars. The desk can instead solve a regularised problem:

**Minimise:** Σᵢ (model_spreadᵢ − market_spreadᵢ)² / σᵢ² + λ · ∫ [h'(t)]² dt

The first term is the fit quality (weighted by quote reliability σᵢ). The second term is a smoothness penalty — it penalises the integral of squared gradient of the hazard rate, pulling the curve toward a smoother shape. λ controls the trade-off.

Setting λ = 0 recovers the exact fit bootstrap. Setting λ large gives a very smooth curve that ignores noise but misfits every pillar slightly. In practice, desks set λ to be just large enough to prevent the forward hazard from changing sign.

**In code, this becomes a QP (quadratic programme):** the squared spread misfit is a quadratic objective, the positivity of hazard rates is a linear inequality constraint, and the smoothness penalty adds a regularisation matrix to the quadratic term. Most desks implement this with a standard constrained least-squares solver rather than rolling their own Newton loop.

---

## Constraint 7: Curve stability — the EOD vs. intraday problem

A correctly bootstrapped curve should move continuously as spreads move. But in practice, the piecewise-constant bootstrap can produce _jumps_ when a spread crosses a threshold that triggers a different hazard rate regime — especially around the zero-hazard constraint.

The desk checks for stability by running the curve at yesterday's spreads plus a small perturbation and ensuring the hazard rates move proportionally. If h₃Y jumps by 50bps when the 3Y spread moves by 1bp, something is wrong in the integration grid or the solver tolerance.

**The three stability checks run at EOD:**

- **Spread sensitivity check:** bump each pillar spread by 1bp, verify hᵢ moves by approximately 1bp / LGD ≈ 1.67bps for a 40% recovery name. Deviations beyond ±5bps signal a solver issue.
- **Time decay check:** roll the curve forward one day without changing spreads. Q(t) should decrease by approximately h(t)/365 per day for each tenor. A sudden jump indicates a grid mis-specification around the IMM date roll.
- **Cross-pillar consistency:** for a normal upward-sloping curve, verify that the 1Y-into-1Y forward hazard is between the 1Y and 2Y par hazard rates. Large violations indicate oscillation in the interpolation.

---

## Summary: what the production bootstrap actually enforces

The full production constraint set, in priority order:

|Constraint|Hard or soft|How enforced|
|---|---|---|
|Q(t) monotone decreasing|Hard|Positivity clamp on h; adjust input spreads if violated|
|h(t) ≥ 0 everywhere|Hard|Floor of 0.01bps in every Newton step|
|Exact reprice at liquid pillars|Hard (liquid), Soft (illiquid)|Newton-Raphson per pillar; regularisation for illiquid|
|Continuous forward hazard|Soft|Monotone convex or log-linear interpolation between pillars|
|Smooth forward hazard|Soft|Cubic spline or regularised bootstrap; only for exotics desks|
|Stable daily P&L attribution|Operational|EOD stability checks, curve version control|
|Consistent with ISDA analytics|Regulatory/legal|Piecewise-constant h, ACT/360, quarterly IMM payments|

The piecewise-constant bootstrap is not perfectly smooth, but it is exact at pillar dates, guaranteed positive (with the clamp), and fully consistent with the ISDA standard that ICE Clear Credit and LCH use for margin calculation. Departing from it requires a very good reason — usually either an exotics book that needs the forward structure, or an XVA desk that needs the full survival density without kinks.

