# Bermudian Swaption Pricing: A Trading Desk Walkthrough

## What Is a Bermudian Swaption?

A **Bermudian swaption** is an option on an interest rate swap that can be exercised on multiple discrete dates (typically quarterly or semi-annually) prior to expiry — sitting between a European swaption (one exercise date) and an American swaption (continuous exercise). The holder pays a premium upfront and, on any eligible exercise date, can choose to enter a pre-agreed swap.

They are common in **callable bond hedging**, **structured rate products**, and **mortgage prepayment modeling**.

---

## Step 0: What the Desk Actually Needs Before Pricing Anything

Before a single number is computed, the desk needs:

- **Trade economics**: notional, fixed rate (strike), swap tenor, exercise schedule, pay/receive direction, day count conventions, business day rules
- **A calibrated interest rate model** (more on this below)
- **A market data stack**: yield curves, swaption volatility surface (normal or lognormal), OIS discounting curves
- **An exercise strategy**: when is it optimal to exercise? This is not trivial.
- **A numerical engine**: analytical methods don't work here. You need a tree, PDE grid, or Monte Carlo.

---

## Step 1: Build and Bootstrap the Yield Curves

The desk maintains several curves simultaneously:

- **Projection curve**: forecasts floating rate fixings (e.g., SOFR, €STR, SONIA). Built by bootstrapping from OIS swaps, futures, and FRAs.
- **Discounting curve**: used to present-value cashflows. Post-2008, this is the OIS curve (collateralised trades use CSA discounting).
- **Basis curves**: if the swap references a tenor different from the OIS index, basis spreads are added.

Bootstrapping means solving, instrument by instrument in maturity order, for the discount factors that reprice each market instrument to par. This gives a set of continuously compounded zero rates at discrete pillars, which are interpolated (typically log-linear on discount factors or cubic spline on zero rates).

This curve is rebuilt **intraday** as market rates move.

---

## Step 2: Build the Swaption Volatility Surface

Bermudian swaptions require volatility inputs for **all the underlying European swaptions** that span the exercise dates and swap maturities embedded within them.

The vol surface is a grid of:

- **Option expiry** (rows): 1m, 3m, 6m, 1y, 2y, 5y, 10y, etc.
- **Swap tenor** (columns): 1y, 2y, 5y, 10y, 30y, etc.
- **Strike dimension**: at-the-money (ATM) vols plus a smile/skew via SABR or similar

The desk marks this surface in **normal (basis point) volatility** — i.e., the Bachelier model — since rates can go negative. Broker quotes for liquid ATM swaptions come in directly; strikes away from ATM require smile interpolation.

**SABR model** is standard for the smile:

Each expiry/tenor slice gets its own SABR parameters (α, β, ρ, ν) calibrated to broker quotes for that slice. β is typically fixed at 0 (normal SABR) or 0.5.

---

## Step 3: Choose the Interest Rate Model

This is the most consequential decision. A Bermudian swaption's value depends critically on the **joint evolution of rates at multiple future dates**, so you need a full term structure model — not just a single-rate model.

### The Industry Standard: LGM / Hull-White 1-Factor

**Linear Gaussian Model (LGM)**, also known as **Hull-White 1-factor**, is the dominant choice on most rates desks for Bermudian swaptions.

The short rate evolves as:

```
dr(t) = [θ(t) - κ·r(t)] dt + σ(t) dW(t)
```

Where:

- `κ` = mean reversion speed (scalar, often fixed)
- `σ(t)` = time-dependent volatility function
- `θ(t)` = drift, chosen to fit the initial yield curve exactly

**Why LGM/HW?**

- Analytically tractable: European swaption prices have closed-form solutions (Jamshidian decomposition)
- Can be implemented on a trinomial tree or PDE grid efficiently
- Calibrates cleanly to European swaption prices
- Single factor = fast

**Limitations:** one-factor models assume perfect correlation between all rates, which is unrealistic for long-dated products or swaptions with many exercise dates spread far apart.

### Upgrade Path: 2-Factor Models (G2++, LGM-2F)

Two-factor models add a second state variable, allowing **imperfect correlation** between short and long rates. This is more realistic and important for:

- Long-dated Bermudians (10y+ exercise schedules)
- Curve steepener/flattener risk

The desk pays a cost in complexity and calibration time.

### For Complex Books: LIBOR Market Model (LMM) / BGM

LMM models the evolution of **forward rates** directly (e.g., all SOFR forwards on a tenor grid). It naturally prices swaptions and caps/floors but:

- Requires Monte Carlo (no tree)
- Calibration is expensive
- Typically used for exotic books with heavy skew sensitivity, not standard Bermudians

---

## Step 4: Calibrate the Model to Market

Calibration = find model parameters such that the model **reprices a set of benchmark European swaptions** to their market prices (derived from the vol surface in Step 2).

### For LGM/HW:

**Mean reversion `κ`** is often fixed exogenously (0.03–0.10 is typical). It controls the term structure of volatility and cannot be reliably identified from a single maturity.

**Volatility function `σ(t)`** is calibrated by matching European swaption prices. The desk chooses a **calibration basket** — a set of European swaptions that are "co-terminal" with the Bermudian:

For a 5y×10y Bermudian (exercisable annually into a 10-year swap):

- 1y×10y European swaption
- 2y×9y European swaption
- 3y×8y European swaption
- 4y×7y European swaption
- 5y×6y European swaption

Each of these is priced analytically under HW using the Jamshidian decomposition (the swap is decomposed into a portfolio of zero-coupon bond options). The σ(t) is bootstrapped forward so each benchmark is repriced exactly.

This calibration runs in **milliseconds** for HW 1-factor. A 2-factor calibration is slower but still sub-second.

### Calibration choices matter enormously:

- **Co-terminal calibration** (above): captures the optionality of each exercise date well
- **Diagonal calibration**: matches swaptions along the diagonal of the vol surface; better for exotic path-dependency
- **Stability**: σ(t) can oscillate day-to-day as vols move; some desks add regularisation

---

## Step 5: Build the Numerical Engine

With calibrated parameters in hand, you compute the Bermudian price using backward induction.

### Trinomial Tree (most common for LGM)

1. Discretise time at each exercise date and coupon date
2. At each node, store the **state variable** x(t) (the Gaussian driver of the short rate)
3. Build the tree forward to the final maturity — each node has a probability of transitioning to three child nodes (up, middle, down)
4. **Roll back from maturity:**
    - At maturity: set terminal value (= swap value, typically 0 at par)
    - At each coupon date: add/subtract coupon cashflows
    - At each **exercise date**: apply the exercise decision

### The Backward Induction and Exercise Decision

At each exercise date `t_i`, at each node `x`:

```
V(t_i, x) = max [ Hold Value, Exercise Value ]
```

- **Exercise value** = PV of entering the swap at that node = swap NPV at (t_i, x)
- **Hold value** = continuation value = expected discounted value of keeping the option alive, computed by rolling back the next timestep's values

The swap NPV at any node is computed analytically under the model (since HW gives closed-form bond prices).

This backward sweep gives you `V(0, 0)` = the Bermudian price today.

### PDE Alternative

Instead of a tree, solve the backward Kolmogorov PDE on a spatial grid in x. More accurate for smooth payoffs, handles boundary conditions more cleanly. Computationally equivalent to a fine tree.

### Monte Carlo (for LMM or 2-factor)

If using Monte Carlo, you **cannot** do naive backward induction because you don't have a recombining lattice. Instead, you use:

**Longstaff-Schwartz (LSM)** regression:

1. Simulate thousands of forward paths of rates
2. At each exercise date, **regress** the continuation value against basis functions of the state variables (e.g., polynomials of the swap rate, annuity factor)
3. Exercise if swap NPV > fitted continuation value
4. Average discounted payoffs across paths

LSM is noisier (requires variance reduction: antithetic paths, control variates) and slower, but handles high-dimensional models that trees cannot.

---

## Step 6: Greeks and Risk Management

The desk doesn't just want a price — it needs a full **risk report** to hedge the position.

### Delta (DV01 / IR Delta)

Bump each curve pillar by 1bp, reprice, divide by 2. This gives a **risk ladder**: how much the position makes/loses if that particular maturity rate moves by 1bp.

A 10y Bermudian might have ~50 curve pillars, so you need ~50 repricings. This is why a fast model matters.

### Vega

Bump each vol surface point by 1 vol unit (1bp normal vol or 1% lognormal vol), reprice. This gives the sensitivity to each swaption on the vol surface — which drives **hedging with European swaptions**.

### Theta

Change the pricing date by one day, re-bootstrap everything, reprice. Theta tells you the daily time decay.

### Gamma

Second derivative of price with respect to rates. Measures how delta changes as rates move — important for understanding hedge rebalancing costs.

### Vanna / Volga

Cross-sensitivities between vol and delta. Relevant for skew hedging.

---

## Step 7: Model Risk and Validation

Every model used on a trading desk must be **independently validated** by a model risk / quant team before it goes live. For Bermudian swaptions this involves:

- Verifying calibration stability across market conditions
- Testing edge cases: deep ITM/OTM exercise, flat vol surface, inverted curves
- Benchmarking against alternative models (HW1F vs G2++ vs LMM)
- Stress-testing Greeks
- Checking put-call parity, early exercise boundaries, no-arbitrage conditions

A model reserve is often held against model uncertainty — particularly the choice of mean reversion and the calibration basket.

---

## Step 8: The Full Pricing Stack in Practice

On a real rates desk, a Bermudian swaption price flows through:

```
Trade Booking System
      ↓
Risk Engine (Murex / Calypso / Orchestrade / in-house)
      ↓
Market Data Server (curves, vol surface, fixings)
      ↓
Model Library (calibration + pricing kernel)
      ↓
P&L, Greeks, Risk Limits
      ↓
Hedging: European swaptions (vega), swaps (delta)
```

Pricing runs in **real-time** for small books (tree-based HW is fast), or **batch overnight** for large exotic books (Monte Carlo with thousands of paths).

---

## Summary Table

|Step|What Happens|Tool / Method|
|---|---|---|
|Curve building|Bootstrap yield curves|OIS, swap quotes|
|Vol surface|Mark swaption vols|SABR per slice|
|Model choice|Pick term structure model|LGM/HW (standard)|
|Calibration|Fit σ(t) to co-terminal basket|Jamshidian + bootstrap|
|Pricing|Backward induction|Trinomial tree / PDE|
|Exercise strategy|Optimal stopping|max(hold, exercise) at each node|
|Greeks|Bumping and repricing|Full grid per risk factor|
|Validation|Model risk sign-off|Independent quant team|

---

The core difficulty of Bermudian swaptions is that you need a model that is simultaneously **consistent with the vol surface** (European swaption prices), **tractable enough** to do backward induction on a tree or grid, and **realistic enough** in its rate dynamics that the exercise boundary it computes is economically sensible. That tension — between tractability and realism — is what keeps rates quants employed.