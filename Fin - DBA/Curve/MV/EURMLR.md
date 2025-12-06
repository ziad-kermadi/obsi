 ![[IMG-20250714200536764.jpg]]
![[IMG-20250714200547324.jpg]]
Excellent — you're spot on again.
What you're seeing is a **live trading and curve calibration environment** that’s implicitly solving a **constrained multi-linear regression problem** to:

> ✳️ **Infer fair values of illiquid swaps** using surrounding liquid market data — while fitting a smooth, arbitrage-free curve.

Let’s now **break it down step-by-step**, in **boring, exhaustive detail**.

---

## 🎯 What’s the Goal of This Screen?

To build and validate multiple **interest rate curves** (e.g., ESTR OIS, EUR6M, basis), and to **infer quotes for illiquid instruments** by using regression-style fitting from more liquid inputs.

---

## 🔍 Step-by-Step: What’s Happening in the UI

---

### 🔲 1. Panel: `Swaps 6s` (Upper Left Block)

This is your **raw input data** for the EUR 6M swap curve.

| Column  | Meaning                                            |
| ------- | -------------------------------------------------- |
| `Inst`  | Swap tenor (1y, 2y, ..., 100y)                     |
| `Value` | The **market-observed swap rate** (e.g. 2.8382%)   |
| `Diff`  | Change from last snapshot                          |
| `Int`   | Interpolated or internal flag (not important here) |

➡️ For tenors like 30y, 40y, 50y, you’ll notice **some values are stale, zeroed, or missing** — these are likely **illiquid**.

---

### 🔲 2. Panel: `SwapFly 6s`, `SwapSwitch`, `Rebucket`, `CrvSprd`

These show **cross-tenor spreads and convexity** of the curve.

| Panel        | What's it doing?                                          |
| ------------ | --------------------------------------------------------- |
| `SwapFly 6s` | Checking curve convexity: e.g., 2s5s2s (fly = 2y, 5y, 2y) |
| `SwapSwitch` | Checks steepness between tenors (e.g. 5s10s, 10s30s)      |
| `CrvSprd`    | Shows slope across regions (e.g. 1y–5y vs 5y–10y)         |
| `Rebucket`   | Weighted average of curve segments (used in macro hedges) |

✅ The **key idea**: if your curve is well-fit, these spreads should be **smooth and stable** — any large jumps or red values mean calibration tension, typically from **bad or missing quotes**.

---

### 🔲 3. Panel: `Esters3s Swap`, `Esters3s FRA OIS`, `3s6s`

This is your **OIS and basis curve input universe**, and where most of the regression logic kicks in.

#### Let's look at a row example:

| Tenor   | Value | Implied | Diff   |
| ------- | ----- | ------- | ------ |
| 12Y OIS | 2.80% | 2.77%   | -3 bps |

✅ This shows that the **observed quote** is 2.80%
✅ The curve fit (from the calibration solver) gives 2.77%
🔁 → So the system **is adjusting the curve slightly** to best fit **all** instruments, even if some quotes are a bit off.

This is where **multi-linear regression-like optimization** is happening:

### 📐 Solver Objective

$$
\min_{\mathbf{x}} \sum_i w_i \cdot \left(\text{ModelPrice}_i(\mathbf{x}) - \text{MarketPrice}_i\right)^2
$$

Where:

| Symbol            | Meaning                                                     |
| ----------------- | ----------------------------------------------------------- |
| `x`               | The vector of curve node levels (e.g. 1Y DF, 5Y DF, 10Y DF) |
| `i`               | Indexes the instruments (e.g. 5y swap, 6M/3M basis)         |
| `w_i`             | Weighting depending on liquidity / trust in quote           |
| `ModelPrice_i(x)` | Price implied by the current curve nodes                    |
| `MarketPrice_i`   | Quote from dealer, broker, or screen                        |

➡️ **Illiquid instruments (e.g. 40Y swap, 25Y basis)**:

* If no `MarketPrice_i` is available, their pricing error is excluded from the sum.
* But their **price still depends** on the curve, so they are **predicted via regression** (linear in the DFs).

---

### 🔲 4. Panel: `Package Axes` (Far Right)

These are your **trade packages** — like 2s10s curve steepeners — being **monitored for pricing accuracy and risk hedging**.

| Column       | Meaning                                      |
| ------------ | -------------------------------------------- |
| `Active`     | Whether this package is tracked in live book |
| `Short Form` | e.g. "pay 400k ester eur 2s10s"              |
| `Auto Hedge` | Whether it auto hedges if misaligned         |
| `Commentary` | Manual tag or trading rationale              |

This helps traders know whether a strategy (e.g., 2s10s steeper) is **still trading at fair value** according to the curve built.

If not — e.g., **curve slope implied from swaps ≠ curve slope from model** — then this is **either an opportunity or a curve calibration issue**.

---

## 🧠 What Is Actually Happening Underneath

Behind the UI:

1. Curve structure (e.g. 15 nodes on ESTR) defines the **model parameters**
2. Quotes on OIS swaps, FRA-OIS, and basis swaps define the **data vector `b`**
3. The solver constructs a **sensitivity matrix `A`**:

   * Row = instrument
   * Column = curve node
   * Entry = how instrument PV changes with each node
4. The system **solves the least squares system** to minimize error:

$$
A x \approx b
$$

✅ Illiquid instruments have **missing `b_i`**, so they're **implied** from neighboring instruments
✅ Can also include **penalties or regularization**, e.g. for curve smoothness, convexity, or no-arbitrage

---

## 📊 Final Summary Table

| Component                          | Function                                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------------------ |
| Swap Rates (Left Yellow)           | Raw market inputs for EUR 6M IRS                                                           |
| FRA OIS & Basis (Middle)           | Calibration targets for ESTR curve                                                         |
| Implied vs Value columns           | Show how well model curve matches observed quotes                                          |
| Spreads & Flies (Top middle)       | Structural checks for curve smoothness                                                     |
| Package Axes (Right)               | Trade packages tracked vs curve shape                                                      |
| Regression Engine (Under the hood) | Fits best curve by minimizing pricing errors across all instruments, implying missing ones |

---

Let me know if you'd like to see **a simulated Python implementation of this calibration + regression logic** — with a missing quote at 40Y, and how it gets inferred!
