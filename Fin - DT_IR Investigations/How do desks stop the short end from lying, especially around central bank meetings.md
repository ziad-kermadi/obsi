You’re basically asking: “How do desks stop the short end from lying, especially around central bank meetings, and how does that feed into the whole multi-curve + multi-currency machine?”

Here’s the desk-true version.

---

## 1) Where meeting dates sit in the curve stack

### In multi-curve (single currency)

You typically have:

* **Discount curve** (CSA/OIS): built from **overnight** instruments (cash ON/TN/SN, short OIS, OIS swaps, and often **Fed Funds futures** historically in USD).
* **Projection curves** (e.g., 3M): built from IBOR/term-index swaps, FRAs, futures, basis vs OIS, etc.
* **Tenor basis curves** (1s3s, 3s6s): built from basis swaps, depends on both projection curves and discounting.

Meeting dates are mostly a **discount-curve short-end** issue because overnight rates are literally central-bank-policy-driven. But the effect propagates everywhere because discounting is in every PV.

### In multi-currency

You have multiple discount curves (one per currency *and collateral/CSA*), plus **XCCY basis**, plus **FX forwards**. Any distortion in the short-end DFs around meeting dates changes:

* FX forward points via CIP-like relations
* XCCY basis PVs (especially front-end)
* and thus the dependent non-USD CSA curves in the dependency tree

So meeting-date handling is not a “USD-only detail.” It’s a stability requirement for cross-currency consistency.

---

## 2) The market reality that forces meeting-date “marks”

Central banks move rates on **discrete dates** (FOMC, MPC, ECB meetings). The expected overnight path is therefore **piecewise-ish**, with jumps near meetings.

If you build a smooth curve with no special nodes at meetings, the solver has only two choices:

1. smear the expected jump across weeks (fake slope/curvature), or
2. create a kink somewhere random to force-fit futures and short OIS.

Both are bad:

* you misprice instruments that straddle the meeting
* you generate unstable risk (DV01 shifts between buckets day-to-day)
* you break “economic interpretability” of the front-end

So desks add **meeting-date knots/marks** so the curve can jump *where the world actually jumps*.

---

## 3) What “meeting date marks” actually mean mathematically

### A common desk parameterization (discount curve short end)

Represent the **overnight forward rate** as piecewise-constant between “important dates”:
$$
f_{ON}(t) = r_j \quad \text{for } t \in (T_{j-1}, T_j]
$$
Where the grid ({T_j}) includes:

* today, spot dates
* month-ends
* IMM dates (if relevant to your instrument set)
* **central bank meeting dates**
* coupon dates of major OIS instruments

Then discount factors are:
$$
D(t) = \exp\left(-\int_0^t f_{ON}(u),du\right)
\approx \exp\left(-\sum_j r_j,\Delta_j\right)
$$
**Key point:**
Meeting dates are just additional grid points (T_j) that allow (r_j) to change across that boundary.

---

## 4) How Fed Funds futures constrain the curve (and why meeting dates matter)

### What a Fed Funds futures quote means (desk level)

A Fed Funds futures contract for month (M) is quoted as:
$$
P = 100 - R^{mkt}_M
$$
where (R^{mkt}_M) is the market-implied **average** effective overnight rate over that calendar month (EFFR average, with day-count conventions).

Your model must produce:
$$
R^{model}*M = \frac{1}{N_M}\sum*{d \in M} E[r_d]
$$
Under a deterministic curve representation, this is basically:
$$
R^{model}_M \approx \frac{\sum_j r_j \cdot \text{DayCount}( \text{segment}_j \cap M)}{\text{DayCount}(M)}
$$
### Why meeting dates are essential here

If a meeting occurs inside the month, the month’s average is a **blend** of:

* pre-meeting expected rate
* post-meeting expected rate

So for a meeting month:
$$
R^{model}_M
=

\frac{n_{\text{pre}},r_{\text{pre}} + n_{\text{post}},r_{\text{post}}}{n_{\text{pre}}+n_{\text{post}}}
$$
Without a knot at the meeting date, you can’t represent (r_{\text{pre}}) vs (r_{\text{post}}) cleanly. The solver will “invent” curvature to fake it.

---

## 5) How “Cash” (ON/TN/SN, short deposits, short OIS) is used

### Cash instruments anchor the *very front*

They do three things:

1. **Fix the first few days/weeks** (ON/TN/SN, short OIS) so the curve is consistent with near-term funding reality.
2. Provide an anchor for the first segment(s) (r_1), preventing the solver from using futures to distort the very front end.
3. Handle date mechanics: spot lags, weekends/holidays, turn-of-month/quarter effects.

In a real engine:

* ON/TN/SN are treated almost like “hard” anchors (high weights)
* very short OIS (1W, 2W, 1M) pin the initial shape
* futures then shape the expected path out the strip

---

## 6) How this enters a simultaneous multi-curve solve

In a simultaneous framework you solve a big parameter vector that includes:

* discount curve parameters (r_j) (overnight forwards between grid dates)
* projection curve parameters (e.g., 3M forwards or pseudo-DF nodes)
* basis curve parameters (if included)
* in multi-ccy: FX forward nodes / cross-currency basis nodes too

You minimize something like:
$$
\min_x \sum_i w_i , r_i(x)^2 + \lambda |Lx|^2
$$
Where residuals (r_i(x)) include:

* OIS swap PV/par errors (discount curve)
* Fed Funds futures month-average errors (discount curve short-end)
* cash instrument errors (front anchor)
* IRS / FRAs / futures for the projection curve
* basis swap residuals coupling curves

Meeting date marks appear because the discount curve parameterization is built on a grid that includes those meeting dates, so the Jacobian has “degrees of freedom” exactly where futures need them.

---

## 7) Worked example: a meeting month Fed Funds future

Assume (toy numbers, but desk-real logic):

* Current effective overnight: (r_{\text{pre}} = 5.33%)
* FOMC meeting on the 18th of the month
* Month has 30 days total
* Days pre-meeting in that month: (n_{\text{pre}}=17)
* Days post-meeting: (n_{\text{post}}=13)
* Fed Funds future implies month average (R^{mkt}_M = 5.25%)

Model relation:
$$
5.25% = \frac{17\cdot 5.33% + 13\cdot r_{\text{post}}}{30}
$$
Solve:

* (17 \cdot 5.33% = 90.61%) (in “day-%” units)
* (30 \cdot 5.25% = 157.50%)
* So (13\cdot r_{\text{post}} = 157.50% - 90.61% = 66.89%)
* Therefore:
$$
  r_{\text{post}} = \frac{66.89%}{13} \approx 5.145%
$$
**Interpretation:** the strip implies a cut (or at least a drop in expected average effective) after the meeting.

Now imagine you *don’t* have a meeting-date knot. The solver can only fit this by bending the forward curve smoothly downward across the month, which:

* misallocates risk into random dates
* screws up pricing of short OIS that settle around the meeting
* makes FX forwards / XCCY basis front-end inconsistent (in multi-ccy)

With meeting knots, you get the economically correct shape: mostly flat pre-meeting, jump/change at meeting, new level after.

---

## 8) How it plugs into the “grand scheme” multi-currency framework

### A) Discount curves drive FX forwards (CIP structure)

In clean form (ignoring basis), FX forwards satisfy:
$$
F(0,T)=S_0 \frac{D_{dom}(T)}{D_{for}(T)}
$$
In reality, cross-currency basis breaks perfect CIP, but the dependence is still through discounting and basis instruments.

If your USD discount curve around FOMC is wrong, then:

* your implied USD DF ratios (D_{USD}(T_1)/D_{USD}(T_2)) are wrong
* FX forward points for short maturities shift
* XCCY basis calibration (which uses FX forwards + discounting) has to “compensate”
* compensation shows up as weird basis kinks or unstable short-end basis risk

So meeting date handling in the collateral currency (often USD) is a first-order input to stability of the entire dependency tree.

### B) Dependency tree vs simultaneous global solve

Even if your architecture is “build collateral currency first, then others,” the *reason* it works is because the collateral curve is well-posed and economically shaped.

In a more global simultaneous solve (like your slide), meeting date marks are applied per currency’s OIS curve:

* USD: FOMC dates
* GBP: MPC dates
* EUR: ECB dates
* JPY: BoJ dates

Then XCCY + FX instruments tie them together. The solver can reconcile cross-currency constraints without forcing absurd curvature in any one curve.

---

## 9) Practical desk nuances you actually see

* **Meeting date may not equal effective date**: some central banks move on meeting day but effective overnight changes can have operational conventions. Engines align to the actual accrual/fixing definitions used by the overnight index.
* **Calendar/turn effects**: month-end, quarter-end, year-end “turn” can cause genuine front-end spikes. Desks often add *turn nodes* too, not just meeting nodes.
* **Futures convexity**: some desks apply small model-based convexity adjustments; many treat FF futures as close-to-linear and rely on tolerances/weights. Either way, the meeting-date granularity still matters because the payoff is an average.
* **Weights matter**: cash and very short OIS often get higher weight than a thinly traded deferred future contract.

---

## Summary tables

### How instruments contribute (multi-curve, USD example)

| Instrument type   | Examples                 | Primary role                                    | Why meeting knots matter                                           |
| ----------------- | ------------------------ | ----------------------------------------------- | ------------------------------------------------------------------ |
| Cash / Fixings    | ON/TN/SN, very short OIS | anchor front-end level                          | ensures immediate DF shape is correct and stable                   |
| Fed Funds futures | monthly futures strip    | pins expected avg overnight path month-by-month | meeting months require pre/post split to avoid fake curvature      |
| OIS swaps         | 1W..30Y OIS              | pins integrated discounting                     | meeting kinks must sit on correct dates to price short OIS cleanly |
| 3M instruments    | FRAs, 3M IRS, futures    | build projection curve                          | discount curve affects PV and calibration trade-offs               |
| Basis swaps       | OIS vs 3M, 1s3s, 3s6s    | couples curves                                  | bad short-end discounting forces basis distortions                 |

### How it propagates in multi-currency

| Component                 | Depends on                     | Meeting-date impact                                               |
| ------------------------- | ------------------------------ | ----------------------------------------------------------------- |
| USD discount (collateral) | cash + FF futures + USD OIS    | sets front-end DF ratios and risk allocation                      |
| FX forwards               | (D_{USD}, D_{EUR}) + basis     | short-end forward points sensitive to DF shape near meetings      |
| XCCY basis                | FX fwds + both discount curves | wrong meeting handling shows as basis kinks / unstable basis DV01 |
| Non-USD CSA curves        | USD CSA + XCCY + local OIS     | clean USD meeting structure stabilizes the whole tree             |

Meeting-date marks are basically the curve engine admitting reality: policy changes happen in lumps, so your curve needs degrees of freedom at those dates, otherwise the solver will manufacture nonsense somewhere else and your multi-currency system will “balance” by breaking something you care about.
