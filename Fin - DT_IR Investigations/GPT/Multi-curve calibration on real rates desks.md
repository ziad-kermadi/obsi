I’ll unpack what “simultaneous” multi-curve calibration means on real rates desks, how the math/optimization actually works (with constraints, weighting, and Jacobians), then I’ll walk through a concrete toy-but-realistic worked example with numbers.


You’re looking at the grown-up version of curve building: instead of “build curve A, then pretend it’s perfect while building curve B,” desks often solve **all the coupled curves in one go**. Because humans love quoting inconsistent markets and then demanding stable risk.

Below is what “calibrate curves simultaneously” means **in real trading systems**, how it’s implemented, and a **worked example with numbers**.

---

## 1) What “simultaneous multi-curve build” actually is

### The objects you’re solving for (the unknowns)

In a post-crisis rates setup, a “curve set” usually means:

* **1 discount curve** (CSA/OIS discounting): discount factors
$$
  D(t)=\text{DF}(0,t)
$$
* **N projection curves** (one per index tenor): forward rates (or an equivalent representation)
  e.g. for USD: 1M, 3M, 6M.
* **Basis curves** (tenor basis like 1s3s, 3s6s) when you build multiple tenors.
* In multi-ccy: **FX forward curve + XCCY basis** couples discounting across currencies.

In the “simultaneous” picture, you don’t treat “OIS first then 3M then 6M then basis” as separate worlds. You solve a **single coupled problem**.

---

## 2) Why bootstrapping “one curve at a time” breaks down in the real world

Sequential bootstrapping assumes the earlier curve is *truth*, so later curves inherit its errors.

That’s not just philosophically ugly; it causes desk pain:

### A) Coupled instruments *force* coupling in calibration

A **3s6s basis swap** PV depends on *both* 3M and 6M forwards **and** the discount curve:
$$
PV_{\text{basis}}(x)=PV_{6M}(x)-PV_{3M}(x)-s\cdot A(x)
$$
If you build 6M first and lock it, then solve 3M, you’re implicitly choosing who “absorbs” inconsistencies (and you can create weird risk).

### B) Inconsistent quotes are normal

Your calibration set is overdetermined (and sometimes contradictory):

* stale/illiquid quotes
* bid/ask differences
* calendar mismatches
* broker runs disagreeing
  So “solve exactly” is often impossible; you need **best fit with controls**.

### C) Risk stability (the real reason)

Risk needs a stable mapping from **market quotes → curve nodes**.
Sequential methods can create hidden “ghost dependencies” (especially with intermediate implied instruments). That’s what the slide is hinting at with “No Ghost Instruments?”

Simultaneous calibration gives you a single coherent Jacobian for the full system.

---

## 3) The real-world simultaneous build: math + constraints

### Step 1: Choose a parameterization (the curve nodes)

Production systems rarely solve for every daily DF. They solve for a vector of parameters:
$$
x = (x_1,\dots,x_m)
$$
Typical choices:

* **log discount factors** at knot dates: $(x_k=\ln D(t_k))$ (nice because $(D(t)>0)$ automatically)
* **instantaneous forwards** at knots (then integrate to get DFs)
* **zero rates** at knots (then interpolate)

Projection curves often use “pseudo discount factors” $(P^{(3M)}(t))$ purely to generate forwards:
$$
F^{(3M)}(t_{i-1},t_i)=\frac{P^{(3M)}(t_{i-1})/P^{(3M)}(t_i)-1}{\delta_i}
$$
### Step 2: Define pricing residuals for *all* calibration instruments

For each market quote $(q_i^{mkt})$, define a model quote $(q_i(x))$ or PV:

* Either enforce **par instruments**: $(PV_i(x)=0)$
* Or match model quote: $(q_i(x)-q_i^{mkt}=0)$

Collect residual vector:
$$
r(x)=\begin{bmatrix} r_1(x)\ \vdots \ r_n(x)\end{bmatrix}
$$
### Step 3: Solve a weighted least-squares + regularization problem

Common objective:
$$
\min_x ;; \sum_{i=1}^n w_i , r_i(x)^2 ;+; \lambda , |Lx|^2
$$
* $(w_i)$: instrument weights (liquidity / bid-ask / priority)
* $(Lx)$: smoothness penalty (e.g., second differences of instantaneous forwards)
* $(\lambda)$: regularization strength (controls wiggles)

### Step 4: Enforce hard constraints (no-arb / sanity)

Typical constraints:

* Discount factors decreasing: $(D(t_{k+1}) \le D(t_k))$
* Forwards non-negative (or bounded) in chosen domain
* Slope/curvature limits (avoid kinks that explode sensitivities)
* Monotone interpolation (Hyman-filtered cubics etc.)

In practice this is done via:

* constrained solvers (SQP / interior point), or
* penalties/barriers, or
* projection after each iteration

### Step 5: Numerical method (what actually runs)

Most curve engines use a variant of **Gauss–Newton / Levenberg–Marquardt**:

* Compute Jacobian $(J=\frac{\partial r}{\partial x})$
* Solve:
$$
  (J^\top W J + \lambda R)\Delta x = -J^\top W r
$$
* Update $(x \leftarrow x + \Delta x)$
* Repeat until residuals and updates are small

This is why simultaneous builds are popular: you get a single system with a controlled Jacobian.

---

## 4) Worked example (single-currency multi-curve, solved simultaneously)

This is a **toy but desk-faithful** example: USD OIS discounting + USD 3M projection, calibrated **together**.

### Market quotes (inputs)

Assume:

* OIS pays quarterly fixed (simplified)
* IRS fixed leg semiannual, float leg quarterly (realistic-ish)

**Quotes**

* 1Y OIS par rate: $(S_{OIS,1Y}=4.00pct)$
* 2Y OIS par rate: $(S_{OIS,2Y}=4.20pct)$
* 6M deposit (3M index style): $(R_{dep,6M}=4.50pct)$
* 1Y 3M IRS par fixed: $(K_{IRS,1Y}=4.70pct)$
* 2Y 3M IRS par fixed: $(K_{IRS,2Y}=4.85pct)$

### Unknown curve nodes (what we solve for)

Discount curve nodes:

* $(D(1Y)), (D(2Y))$

3M projection “pseudo-DF” nodes (to generate forwards):

* $(P^{3M}(0.5Y)), (P^{3M}(1Y)), (P^{3M}(2Y))$

So:
$$
x = \big(D(1),D(2),P^{3M}(0.5),P^{3M}(1),P^{3M}(2)\big)
$$
Assume **log-linear interpolation** on $(\ln D(t))$ and $(\ln P(t))$ between nodes.

### Instrument equations (residuals)

**(1) OIS par condition** (simplified):
$$
PV^{OIS}*{fixed}(x)-PV^{OIS}*{float}(x)=0
$$
with
$$
PV^{OIS}*{float}=1-D(T),\quad PV^{OIS}*{fixed}=S\sum_k \delta_k D(t_k)
$$
**(2) Deposit** pins (P^{3M}(0.5)):
$$
P^{3M}(0.5)=\frac{1}{1+R_{dep,6M}\cdot 0.5}
$$
**(3) 3M IRS par condition**:
$$
K\sum_{j \in \text{fixed pay}} \delta^{fix}*j D(t_j)
;-;
\sum*{i \in \text{float pay}} \delta^{flt}*i D(t_i),F^{3M}(t*{i-1},t_i)
=0
$$
and
$$
F^{3M}(t_{i-1},t_i)=\frac{P^{3M}(t_{i-1})/P^{3M}(t_i)-1}{\delta_i}
$$
### Simultaneous calibrated solution (outputs)

Solving the 5 coupled equations gives:

* $(D(1Y)=0.96098)$

* $(D(2Y)=0.91976)$

* $(P^{3M}(0.5Y)=0.977995)$  (this one is basically the deposit)

* $(P^{3M}(1Y)=0.954551)$

* $(P^{3M}(2Y)=0.908466)$

From these, the implied **3M forwards** (quarterly) come out piecewise (because of the knot placement):

* 0–0.5Y: $(F \approx 4.475\%)$
* 0.5–1Y: $(F \approx 4.882\%)$
* 1–2Y: $(F \approx 4.979\%)$

And by construction, the model reprices:

* OIS 1Y = 4.00%
* OIS 2Y = 4.20%
* IRS 1Y = 4.70%
* IRS 2Y = 4.85%

### Why this example matters

Even in this small setup, the IRS par rates depend on **both**:

* discount factors (D(t))
* forward generation via $(P^{3M}(t))$

A sequential method would “lock” (D(t)) first. Simultaneous solving lets the optimizer distribute tension between curves (guided by weights/regularization), which is exactly what you want when instruments disagree.

---

## 5) What this looks like on an actual desk (practical details)

### A) They don’t solve “exactly”, they solve “best fit with rules”

Real calibration sets are large and messy:

* OIS: deposits/ON, OIS swaps, futures (some ccys)
* Projection: deposits, FRAs, futures, swaps
* Basis: 1s3s, 3s6s, etc.
* Optional shape constraints: swaps flies/switches as soft constraints (your slide’s “switches/fl ies” vibe)

So the build is typically:

* **weighted least squares** on PV/quote errors
* **regularization** on instantaneous forward curvature
* **hard/soft constraints** for no-arb and stability

### B) Weighting is desk politics disguised as math

Typical weighting logic:

* tight bid/ask + high volume = high weight
* broker runs / illiquid tenors = low weight
* calendar rolls / stubby tenors = controlled weights
* outliers get down-weighted or kicked

### C) Risk is computed consistently from the same solve

After calibration, for risk you want:
$$
\frac{\partial PV}{\partial q}
$$
Most robust method:

* bump market quotes (q)
* rebuild curves (same solver)
* reprice PV

Simultaneous build helps because the “rebuild mapping” is coherent, and you avoid weird stepwise dependencies that show up as jagged bucketed risk.

---

## Summary tables

### Sequential vs simultaneous build

| Item                             | Sequential (bootstrap curve-by-curve) | Simultaneous (global solve)               |
| -------------------------------- | ------------------------------------- | ----------------------------------------- |
| Handles inconsistent quotes      | Poorly (forces exact fit per step)    | Naturally (weights + best fit)            |
| Coupled instruments (basis/XCCY) | Awkward dependency order              | Native (coupling is explicit)             |
| Smoothness control               | Add-on after the fact                 | Built-in via regularization               |
| No-arb constraints               | Harder across curves                  | Enforced jointly                          |
| Risk stability (Jacobians)       | Can be jumpy / “ghosty”               | Typically cleaner & more stable           |
| Debuggability                    | Easy locally, hard globally           | Harder locally, but consistent end-to-end |

### Worked example recap

| Component       |                            Market inputs |                                                       Calibrated outputs |
| --------------- | ---------------------------------------: | -----------------------------------------------------------------------: |
| OIS discount    |                       1Y 4.00%, 2Y 4.20% |                                           (D(1)=0.96098), (D(2)=0.91976) |
| 3M projection   | 6M dep 4.50%, 1Y IRS 4.70%, 2Y IRS 4.85% | $(P^{3M}(0.5)=0.977995)$, $(P^{3M}(1)=0.954551)$, $(P^{3M}(2)=0.908466)$ |
| Implied 3M fwds |                              (generated) |                           0–0.5Y: 4.475% ; 0.5–1Y: 4.882% ; 1–2Y: 4.979% |

### What “simultaneous” really means in code

| Piece                            | What it does                                                  |
| -------------------------------- | ------------------------------------------------------------- |
| Parameter vector (x)             | stores all curve nodes for all curves in the build set        |
| Residuals (r(x))                 | PV/quote errors for every calibration instrument              |
| Weights (W)                      | liquidity / bid-ask / priority control                        |
| Regularization ( \lambda|Lx|^2 ) | prevents wiggles / kinks / unstable risk                      |
| Solver step                      | solves ((J^\top WJ+\lambda R)\Delta x=-J^\top Wr) iteratively |

---

If you want this upgraded from “single currency multi-curve” to the **multi-currency dependency tree in your second slide** (GBP collateral → USD CSA → EUR CSA with XCCY + FX forwards), the exact same machinery applies, you just add:

* FX forward residuals (CIP consistency)
* XCCY basis swap residuals
* multiple discount curves under different collateral assumptions

It becomes the same optimization problem, just with more ways for the market to contradict itself.
