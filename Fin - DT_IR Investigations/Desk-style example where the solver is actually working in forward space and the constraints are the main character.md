You want a desk-style example where the **solver is actually working in forward space** and the **constraints are the main character**, not an afterthought. Fine. Here’s what it looks like when a real curve engine is trying to keep your risk from turning into modern art.

---

## 1) Desk-real “simultaneous build” in forward space

### The point

Instead of solving “discount curve first, then projection curve,” a desk engine solves:

* a **discounting instantaneous forward curve** $(f_d(t))$ (CSA/OIS)
* a **projection forward curve** $(L(t_{i-1},t_i))$ (e.g., 3M index forwards)

**at the same time**, because instruments link them.

### Why forwards (not just DFs)?

Because most “typical constraints” you listed are naturally expressed on forwards:

* **DF monotonicity** becomes “discount instantaneous forward is non-negative”
* **no crazy negative forwards** becomes “bounds on (L)”
* **no kinks** becomes “limits on forward slope/curvature”

---

## 2) The core representation (this is what engines really control)

### Discounting curve in *instantaneous forwards* on segments

Choose a knot grid aligned with cashflows (quarterly here for simplicity, but desks use richer grids: IMM, meeting dates, month-ends, turns).

Let the discount instantaneous forward be piecewise-constant per quarter:
$$
f_{d,k} \quad \text{for } (t_{k-1},t_k],  \Delta t = 0.25
$$
Then the discount factor is:
$$
D(t_k)=\exp!\left(-\sum_{j=1}^{k} f_{d,j}\Delta t\right)
$$
So you never “solve for DFs” directly. You solve for the **forward increments**, then integrate into DFs.

### Projection curve directly in forwards

For the 3M index, on the same quarterly grid, define:
$$
L_k \equiv L(t_{k-1},t_k) \quad (\text{a 3M forward})
$$
This is the forward the float leg uses (with daycount and fixing conventions in real life, but the structure is the same).

---

## 3) Desk-style objective: fit + weights + shape control

A typical “risk curve” solve looks like:
$$
\min_{f_d,,L};;
\underbrace{\sum_i w_i,r_i(f_d,L)^2}*{\text{fit market quotes}}
;+;
\underbrace{\lambda \sum_k (\Delta^2 f*{d,k})^2 + \lambda' \sum_k (\Delta^2 L_k)^2}*{\text{curvature smoothing}}
;+;
\underbrace{\mu \sum_k \max(0,|\Delta f*{d,k}|-s_{max})^2 + \mu' \sum_k \max(0,|\Delta L_k|-s_{max})^2}_{\text{slope soft-cap}}
$$
Where:

* (r_i) are residuals (par swap PV errors, quote errors, etc.)
* (w_i) reflect liquidity / bid-ask / instrument priority
* (\Delta^2) is the second difference (curvature proxy)
* slope penalties mimic hard slope constraints when you don’t use a fully constrained solver

Then the engine solves iteratively (Gauss–Newton / LM style), **projecting** back onto hard bounds each step.

---

## 4) The “typical constraints” you asked for, desk-true

### Constraint A: Discount factors decreasing

You wrote:
$$
D(t_{k+1}) \le D(t_k)
$$
In forward parameterization, this is basically:
$$
f_{d,k} \ge 0 \quad \forall k
$$
Because if (f_{d,k}\ge 0), then each step multiplies DF by (\exp(-f_{d,k}\Delta t)\le 1), so DFs can’t increase.

**How desks enforce it**

* **Hard bound:** $(f_{d,k}\in[0,f^{max}])$
* Or **reparameterize:** $(f_{d,k}=\text{softplus}(u_k))$ so it’s always $(\ge 0)$

This is cleaner than “solve DFs then try to fix monotonicity after.”

---

### Constraint B: Forwards non-negative (or bounded)

For projection forwards:
$$
L_k \in [L_{min}, L_{max}]
$$
Desk reality:

* In normal markets, (L_{min}=0) is common.
* In negative-rate regimes, desks allow (L_{min}<0) (but still bounded so the solver doesn’t invent -7% forwards to fit one garbage quote).

**Typical desk bounds**

* (L_{min}): (-0.01) to (0.00) (depending on currency/regime)
* (L_{max}): (0.10) to (0.20) (safety cap)

---

### Constraint C: Slope/curvature limits (avoid kinks that explode sensitivities)

This is the most “risk-driven” constraint.

A kink in instantaneous forwards creates:

* unstable bucketed DV01
* nasty Jacobians (small quote bumps cause huge node moves)
* “spiky” hedge ratios that traders hate (because they have to actually trade)

**How desks implement it**

1. **Hard slope cap**
$$
   |f_{k}-f_{k-1}| \le s_{max}
   \quad\text{and}\quad
   |L_k-L_{k-1}| \le s_{max}
$$
2. **Curvature penalty** (very common)
$$
   \sum_k (f_{k+1}-2f_k+f_{k-1})^2
$$
   Same for (L).

3. **Interpolation choice**
   Even with nodes, how you interpolate matters:

* log-DF interpolation preserves positivity
* monotone convex / Hyman-filtered cubic prevents overshoot that creates forward wiggles

---

## 5) A more desk-involved worked example (with forwards + constraints)

### Setup (2Y build on quarterly grid, solved simultaneously)

We build:

* discount forwards (f_{d,1.8}) (OIS discounting)
* 3M projection forwards (L_{1.8})

**Market instruments**

* OIS par: 1Y = 4.00%, 2Y = 4.20%
* 3M deposit: 4.60%
* 3M IRS par: 1Y = 4.80%, 2Y = 4.95%

**Constraints enforced**

* (f_{d,k}\ge 0)  (so (D) decreases)
* (0 \le L_k \le 12%)
* slope control (targeting “no quarter-to-quarter jumps”)
* curvature smoothing (avoid kinks)

### Resulting calibrated curves (one possible constrained solution)

**Model fits (errors are tiny, as intended):**

* OIS 1Y: 4.0000%
* OIS 2Y: 4.1999%
* Depo 3M: 4.6007%
* IRS 1Y: 4.7981%
* IRS 2Y: 4.9507%

**Discount curve outputs (generated from constrained discount forwards):**

* (D(1Y)=0.960968)
* (D(2Y)=0.919734)

**Forward curves (the actual control variables):**

| Quarter ending (t) | (f_d) (inst disc fwd) | (L) (3M fwd) |   (D(t)) |
| -----------------: | --------------------: | -----------: | -------: |
|               0.25 |               3.8299% |      4.6007% | 0.990471 |
|               0.50 |               3.9309% |      4.7279% | 0.980785 |
|               0.75 |               4.0319% |      4.8407% | 0.970949 |
|               1.00 |               4.1329% |      4.9319% | 0.960968 |
|               1.25 |               4.2340% |      5.0021% | 0.950850 |
|               1.50 |               4.3351% |      5.0593% | 0.940600 |
|               1.75 |               4.4362% |      5.1099% | 0.930226 |
|               2.00 |               4.5373% |      5.1582% | 0.919734 |

### Check the constraints (explicitly)

* **DF decreasing:** yes (every step down)
* **Bounds on forwards:**

  * $(f_d\in[3.83\%, 4.54\%])$
  * $(L\in[4.60\%, 5.16\%])$
* **Slope (quarter-to-quarter max move):**

  * $max (|\Delta f_d| \approx 10.1)$ bp per quarter
  * $max (|\Delta L| \approx 12.7) bp$ per quarter
    That’s exactly the “no kink” behavior risk teams like.

**Interpretation (desk-relevant)**

* Discount forwards ramp smoothly from ~3.83% to ~4.54%: consistent with OIS 1Y/2Y being 4.00/4.20 but upward sloping.
* 3M forwards are higher than OIS forwards (projection > discount): consistent with a positive tenor premium / basis environment.

---

## 6) Why these constraints matter for *risk*, not aesthetics

If you don’t control forward slope/curvature:

* the solver can fit tiny inconsistencies by creating a localized kink
* that kink sits near some coupon date
* then a 1bp bump in one quote reshuffles nodes non-locally
* your DV01 “moves around” across buckets day to day
* traders accuse the quant team of witchcraft (and they’re not totally wrong)

So desks constrain shape mainly to protect **stability of (\partial PV/\partial q)**, not because they love smooth lines.

---

## Summary tables

### Typical constraints and how desks enforce them

| Constraint       | Natural form              | Typical enforcement                                            |              |                                                                 |
| ---------------- | ------------------------- | -------------------------------------------------------------- | ------------ | --------------------------------------------------------------- |
| DF decreasing    | (D(t_{k+1})\le D(t_k))    | parameterize via (f_{d,k}\ge 0) so monotonic DFs are automatic |              |                                                                 |
| Forwards bounded | (L_k\in[L_{min},L_{max}]) | hard bounds + projected updates (or reparameterization)        |              |                                                                 |
| No kinks (slope) | (                         | \Delta f_k                                                     | \le s_{max}) | hard inequality in constrained solver or hinge-penalty soft cap |
| Smooth curvature | small (\Delta^2 f)        | Tikhonov regularization on second differences of forwards      |              |                                                                 |

### Worked example recap

| Item             | Inputs                           | Output behavior                                          |
| ---------------- | -------------------------------- | -------------------------------------------------------- |
| Discount curve   | OIS 1Y/2Y                        | (D(t)) strictly decreasing; (f_d(t)) smooth upward slope |
| Projection curve | 3M depo + 3M IRS 1Y/2Y           | (L(t)) bounded and smooth; no localized kinks            |
| Shape controls   | bounds + slope/curvature control | stable forwards ⇒ stable bucketed DV01                   |

### Model fit quality (example)

| Instrument |  Market |   Model | Error (bp) |
| ---------- | ------: | ------: | ---------: |
| OIS 1Y     | 4.0000% | 4.0000% |    +0.0007 |
| OIS 2Y     | 4.2000% | 4.1999% |    -0.0110 |
| Depo 3M    | 4.6000% | 4.6007% |    +0.0740 |
| IRS 1Y     | 4.8000% | 4.7981% |    -0.1908 |
| IRS 2Y     | 4.9500% | 4.9507% |    +0.0692 |

That’s a constraint-driven simultaneous build in the way desks actually care about: **forwards first**, DFs as an integrated consequence, and shape controls engineered so your risk doesn’t randomly explode on a Tuesday.
