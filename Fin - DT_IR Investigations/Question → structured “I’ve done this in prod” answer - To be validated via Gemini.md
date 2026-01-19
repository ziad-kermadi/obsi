Fine. We’ll do this like an actual VP interview: **question → structured “I’ve done this in prod” answer**, with the stuff people forget (accrual mechanics, calendars, tolerances, and the parts that ruin risk).

There are **50+ questions** across the two lists, so I’m starting with **Q1–Q12** (the ones most likely to show up early in an interview, and the ones you can’t fake). Next up after that would be Q13 onward.

---

## Q1) “How do you model an accrual period that crosses a central bank meeting date?”

If an accrual period crosses a meeting, the underlying overnight rate path is **piecewise**: pre-meeting expected rate, post-meeting expected rate (and potentially more than one meeting for longer periods). The correct way to price is to **split the accrual integral** at the meeting effective boundary, not “smooth it out.”

### What I do in practice

1. **Put meeting effective dates into the front-end grid**
   My discount/ON forward representation uses segment boundaries:
$$
   {T_j} \supset {\text{spot}, \text{EOM}, \text{turns}, \text{meeting effective dates}}
$$
   With piecewise ON forwards (r_j) on ((T_{j-1},T_j]).

2. **Compute OIS coupon accrual as an integral/sum over segments**
   For an OIS-style compounded coupon over ([S,E]):
$$
   \text{Accrual} \approx \prod_{d\in [S,E)}(1+r_d \Delta_d)-1
$$
   In deterministic curve engines this becomes “multiply segment contributions” (or log-sum them).

3. **Ensure the solver has the DOF exactly where the market needs it**
   Meeting-month futures and short OIS depend on month-average or period-average overnight. Without a meeting boundary, the engine can’t produce correct averages without inventing curvature.

### Failure mode (what goes wrong)

* You smear a policy jump into a gentle slope.
* Short instruments that straddle the meeting misprice.
* Risk concentrates in “random” buckets because the curve is forced to kink somewhere else.

### Production check

* Reprice a 1W/2W/1M OIS that straddles the meeting and compare vs vendor.
* Verify the **difference** between “meeting-aware” and “smooth” build is localized, not leaking into long end.

---

## Q2) “What is a ‘meeting stub’ in OIS, and how does it impact calibration?”

A **meeting stub** is an irregular accrual segment created because you intentionally insert a knot at a meeting date (or meeting effective date), which creates a short or long mini-period inside what would otherwise be a regular schedule.

### Why it matters

Even if the trade coupon schedule is regular (e.g., quarterly), the **discount curve’s internal segmentation** is not. You’re integrating the ON forward over a boundary that wasn’t aligned to the coupon grid.

### Impact on calibration

* Meeting-month futures imply a **weighted average overnight rate** across the month.
* The only clean way to satisfy that constraint is:
$$
  \bar r_{month}=\frac{n_{\text{pre}} r_{\text{pre}} + n_{\text{post}} r_{\text{post}}}{n_{\text{pre}}+n_{\text{post}}}
$$
  So you need both (r_{\text{pre}}) and (r_{\text{post}}) as separate segments, which creates the “stub.”

### Failure mode

If you ignore stub mechanics, you can fit the strip but you’ll see:

* systematic residuals in short OIS,
* weird “micro kinks” that make front-end DV01 unstable.

### Production tip

Treat meeting stubs like turns: you often want **localized freedom** (low smoothing penalty across the meeting boundary) while keeping the rest smooth.

---

## Q3) “How do payment lags and fixing lags enter OIS and IBOR legs and affect the curve?”

This is where most “curve shape problems” are actually **date bugs**.

### OIS / overnight legs

* OIS uses daily overnight fixings; the “fixing lag” is usually baked into the index definition (published rate applies to a day).
* **Payment lag** affects the *discounting of the coupon cashflow*:

  * accrual over ([S,E])
  * paid at (E + \text{payLag})
    So PV uses (D(E+\text{payLag})), not (D(E)).

### IBOR / term legs

* A 3M coupon is typically:

  * fixed at (S - \text{fixLag})
  * accrues over ([S,E])
  * paid at (E+\text{payLag})

### How this impacts calibration

If you ignore pay lag:

* you will *systematically* misprice short instruments where (D(E)) and (D(E+\text{lag})) differ materially.
* the solver will compensate by bending the front end, which looks like “bad curve” but is just “wrong cashflow date.”

### What I say in an interview

“I don’t tune regularization to fix a date problem. I validate schedule generation first: fixing dates, accrual dates, payment dates, holiday rolls, stub rules.”

---

## Q4) “How do you prevent the meeting-date grid from creating discontinuities when you transition to the long-end swap grid?”

You’re balancing two conflicting needs:

* **High-resolution segmentation** at the front end (meetings/turns)
* **Low-dimensional smooth structure** in the long end (where instruments are broader and you don’t want overfit)

### Desk-true solution

1. **C0 continuity is non-negotiable**: discount factors must remain continuous and positive.
   Using log-DF integration from forwards guarantees continuity by construction.

2. **Control where curvature can live**

   * Make smoothing penalties maturity-dependent: stronger in long end, weaker near front-end events.
   * Or segment the curve into “front” and “back” zones with a bridge.

3. **Bridge knot(s)**
   Often you place a knot at a well-supported maturity (e.g., 2Y, 3Y) that is liquid and stabilizes the transition. This prevents the solver from dumping curvature into the first long-end interval.

### Diagnostic

Plot instantaneous forwards; if you see a kink right at the boundary between “meeting grid” and “swap grid,” your penalty structure is too uniform or your knot choice is naive.

---

## Q5) “How do you convert a futures quote into a curve constraint in practice?”

Futures are not “a forward point.” They’re a constraint on an **average rate over a window** (for FF) or a specific contract-defined settlement logic (other IR futures).

### For Fed Funds futures (month average)

Quoted price (P=100-R^{mkt}). Model-implied:
$$
R^{model}*{month}=\frac{\sum_j r_j \cdot \text{DC}((T*{j-1},T_j]\cap \text{month})}{\text{DC}(\text{month})}
$$
Residual:
$$
r_i(x)=R^{model}_{month}-R^{mkt}
$$
### Implementation detail that matters

* You compute the weights using the **actual calendar** (weekends/holidays matter).
* You do **not** create daily calibration degrees of freedom; you integrate your segmented curve over the window.

### Interview-grade phrasing

“I map each futures contract into a linear functional of my forward segments. That keeps the Jacobian stable and makes meeting-date effects explicit.”

---

## Q6) “Explain why daily margining matters for futures-to-forward conversion.”

Because futures are **marked to market daily**, the futures price reflects the expectation under a measure influenced by the correlation of rates with discounting/numeraire effects. In plain desk terms: **futures ≠ forwards** when rates are stochastic.

### What’s the effect?

There’s a convexity bias:

* if rates are volatile, the futures rate differs from the corresponding forward rate
* the size depends on vol and mean reversion (model-dependent)

### Desk reality

* For some products/tenors it’s small and absorbed in tolerances.
* For others (e.g., STIR futures in certain regimes) it’s systematic enough that ignoring it produces residual patterns.

### What I’d say as a VP strat

“I’m less worried about the existence of convexity than about being consistent: either apply a convexity framework and monitor residuals, or explicitly widen tolerances and don’t pretend the fit is telling you curve shape.”

---

## Q7) “What happens if your futures convexity adjustment model is wrong?”

You don’t just misprice futures. You contaminate the curve:

### Observable symptoms

* The strip fits but swaps show a systematic hump (or vice versa).
* Residuals are tenor-patterned: specific contract months persistently off.
* The curve develops artificial curvature where the solver tries to reconcile mismatched functional forms.

### Risk symptom (worse)

Your mapping (q \mapsto x) becomes unstable because the solver keeps “fighting” the wrong convexity correction.

### What I’d do

* Run residual heatmaps by contract month.
* Compare two builds: with convexity on vs off, see if misfit rotates predictably.
* If model uncertainty is high: treat convexity as a **soft effect** (penalty/tolerance) rather than hard-correcting aggressively.

---

## Q8) “How do you handle a contract whose accrual window doesn’t align with curve nodes?”

You value it by **integrating over the window**, not by adding nodes just to make your life easier.

### Practical method

1. Keep calibration DOFs at sensible grid points (meetings/turns/IMM/liquid maturities).
2. For valuation of a contract with window ([S,E]), compute:
$$
   \int_S^E f(t),dt
$$
   by splitting across whatever segments intersect ([S,E]).

### Key principle

**Valuation granularity ≠ calibration granularity.**
Adding nodes to fit one oddball contract is how you end up with a fragile curve and ugly risk.

### Exception (real desks do this)

If that contract family is highly liquid and central to the desk (e.g., a dominant futures strip), you may promote some boundaries to calibration knots. Liquidity earns degrees of freedom.

---

## Q9) “How does the CSA collateral rate choice change the discount curve?”

Discounting is CSA-specific because the trade’s PV is computed under the **collateral remuneration** (plus details like thresholds, MTAs, dispute timing, etc., but the core is the collateral rate).

### In clean form

If collateral accrues at OIS, then your discount curve is the OIS curve consistent with that collateral index:
$$
PV = \sum_k D_{CSA}(t_k),CF_k
$$
### What changes in practice

* If you switch collateral index/currency, you switch discount curve.
* The same trade can have different PVs under different CSAs.
* This feeds directly into curve-set architecture: “one curve per currency” is wrong; it’s “one curve per currency per CSA regime.”

### VP tip

I explicitly separate:

* “discount curve used for PV”
* “forecast curve used for floating coupons”
  and I tie discount curve selection to trade-level CSA metadata, not trader preference-of-the-day.

---

## Q10) “What’s the practical difference between ‘discount curve’ and ‘funding curve’ in XVA systems?”

Discount curve in pricing = collateral/CSA discounting (market-consistent under the CSA assumptions).
Funding curve = the desk’s internal cost of funding uncollateralized exposure / funding spreads used in FVA/FTP.

### Why people mess this up

If you mix them, you double-count funding or you price with an internal curve that isn’t hedgeable.

### VP-grade answer

* Base PV uses CSA discounting.
* Funding effects are handled as an adjustment (FVA/FTP) with its own modeling and governance.
* Keep responsibilities clean: discounting curve is for fair value under collateral; funding curve is for cost allocation/valuation adjustments.

---

## Q11) “How do you support CSA switching or collateral optionality?”

Collateral optionality is **literally optionality**. It’s not a static spread.

### Correct conceptual approach

* If the counterparty can switch collateral, the effective discounting is closer to a **cheapest-to-deliver** style problem:
  the party posting collateral chooses the cheapest funding/collateral source (subject to rules).
* The value depends on future states (rates, spreads), so you need scenarios.

### Practical desk approximations

* Scenario-based valuation: compute PV under a few plausible collateral regimes, approximate option value.
* Add-on approach: approximate benefit as bounded between two curve sets:
$$
  PV \in [PV^{CSA1}, PV^{CSA2}]
$$
  and estimate optionality premium from vol/spread assumptions.

### VP phrasing

“I don’t collapse collateral optionality into one curve and call it done. I either model the switching as optionality under scenarios or I produce a controlled approximation with governance.”

---

## Q12) “Explain how compounding-in-arrears vs simple compounding changes your forward construction.”

This is about matching the **index definition**.

### Simple (IBOR-like) forward coupon

For a period ([S,E]):
$$
CF = N \cdot L(S,E)\cdot \delta
$$
You just need the forward (L(S,E)).

### Compounded-in-arrears (common in RFR term constructions / some legs)

Coupon depends on daily compounded overnight rates:
$$
CF = N \left(\prod_{d\in [S,E)} (1+r_d \Delta_d)-1\right)
$$
So the “forward” is not a single point. It’s a functional of the entire path over the period.

### What changes in curve building

* The projection curve object might be an ON forward curve integrated over the coupon window.
* If you try to represent compounded coupons with a single simple forward naively, you introduce biases that show up as systematic residuals or incorrect risk.

### Desk-level statement

“Whenever compounding changes, the instrument-to-model mapping changes. I don’t tune the curve to fit a coupon definition mismatch.”

---

# Summary table (Q1–Q12)

| Q# | Topic                   | Key point the interviewer wants           | Classic trap                                         |
| -: | ----------------------- | ----------------------------------------- | ---------------------------------------------------- |
|  1 | Accrual crosses meeting | split accrual at meeting boundary         | smearing jumps into smooth curve                     |
|  2 | Meeting stubs           | irregular segments are intentional DOFs   | mishandling stubs → front-end misfit                 |
|  3 | Lags                    | payment/fixing lags shift PV timing       | tuning regularization to fix date bugs               |
|  4 | Grid transition         | dense front end, smooth back end          | kink at the join due to uniform penalty              |
|  5 | Futures constraint      | futures = average over window             | treating as a single forward point                   |
|  6 | Daily margining         | futures ≠ forwards under stochastic rates | ignoring convexity then “discovering” fake curvature |
|  7 | Wrong convexity model   | contaminates curve + risk stability       | forcing hard corrections with poor model             |
|  8 | Window misalignment     | integrate over window, don’t add DOFs     | adding nodes for convenience = unstable curve        |
|  9 | CSA discounting         | discount curve is CSA-specific            | “one curve per currency” simplification              |
| 10 | Discount vs funding     | separate fair value vs XVA/FTP            | double-counting funding                              |
| 11 | Collateral optionality  | it’s optionality, scenario-dependent      | collapsing into a static spread                      |
| 12 | Compounding             | coupon definition changes mapping         | fitting the wrong payoff with “better curve”         |

Next up in the sequence: **Q13–Q20** (multi-curve coupling, basis consistency, multi-currency truth hierarchy, and propagation paths).

----

---

## Q13) “How do you ensure internal consistency between projection curves of different tenors (1M/3M/6M) without creating arbitrage across tenors?”

The right mental model is: **tenor curves are not independent curves**. They’re different projections of the same underlying rate environment, and the glue is **basis instruments** plus **shape/consistency constraints**.

### What I do in practice

1. **Choose a clean responsibility split**

* Discount curve: OIS/CSA discounting.
* Projection curves: one per index tenor (e.g., 1M/3M/6M).
* Basis curves (optional explicit): represent “excess spread” between tenors.

2. **Calibrate with explicit coupling instruments**

* 1s3s basis swaps, 3s6s basis swaps, OIS-vs-IBOR basis, etc.
* These instruments directly constrain relationships like:
$$
  PV(\text{6M float}) - PV(\text{3M float}) - s\cdot A = 0
$$
  meaning both curves are co-determined.

3. **Prevent pathological cross-tenor shapes**
   Even with basis, you can get “tenor arbitrage” style weirdness if one curve develops spikes to absorb noise.
   So I enforce:

* bounds on forwards per tenor (regime-aware),
* smoothness penalties on each curve,
* and (this is the key) **a coupling prior**: basis shouldn’t oscillate wildly when basis quotes are sparse.

A common desk way is to penalize curvature of the *basis function*:
$$
\sum_k (\Delta^2 (f^{6M}_k - f^{3M}_k))^2
$$
so the relationship across tenors stays stable.

### What “arbitrage” means here (desk interpretation)

It’s not a formal arbitrage like static replication across tenors (because conventions and credit/liquidity premia differ), but you still want to avoid:

* basis changing sign back and forth every few months for no reason,
* or projection curves crossing in nonsensical ways purely due to overfit.

### Great answer bonus

I explicitly say: “I keep **basis risk** localized and explainable. If a 3s6s basis quote moves, the basis curve should move; it shouldn’t randomly reshape the whole 3M curve.”

---

## Q14) “Describe how you would attribute a misfit in a basis swap: is it discounting, forwarding, or basis curve error?”

Misfit attribution is a **block-Jacobian debugging problem**, not vibes.

### The practical method

Let the full state be:
$$
x = (x_d,;x_{3M},;x_{6M},;x_{basis})
$$
Residuals include OIS, IRS, basis swaps. Suppose a 3s6s basis swap has a residual.

1. **Check the obvious first (conventions/data)**

* schedule generation, daycount, stub rules, effective dates, fixing/payment lags
* especially around turns/meetings

2. **Sensitivity decomposition**
   Compute the basis swap PV sensitivities to the blocks:
$$
   \frac{\partial PV}{\partial x_d},\quad \frac{\partial PV}{\partial x_{3M}},\quad \frac{\partial PV}{\partial x_{6M}},\quad \frac{\partial PV}{\partial x_{basis}}
$$
   This tells you who *can* move to fix the residual.

3. **Controlled re-solves**
   This is what I actually do in prod when debugging:

* **Freeze discount** (x_d), re-solve projection/basis → see if misfit disappears.
* Freeze 3M, re-solve 6M/basis.
* Freeze basis curve (or set it to 0), see how much misfit becomes “unavoidable.”

If freezing one block causes the residual to remain, you’ve identified which component must move.

### Great answer bonus

I mention **instrument clustering**: if the misfit is isolated to basis swaps but IRS are fine, it’s likely basis curve/weights. If basis and IRS both misfit in the same maturity region, suspect projection curve or discounting.

---

## Q15) “When basis quotes are sparse: do you build an explicit basis curve or embed basis into projection curves?”

Both are viable; the VP answer is: it depends on what you need operationally, but you must avoid mixing responsibilities silently.

### Option A: Explicit basis curve (my default on a desk)

* Build discount + a “primary projection” (say 3M).
* Build 6M as 3M plus an explicit basis function.
  Pros:
* interpretable basis risk
* easier PnL explain
* cleaner hedging story
  Cons:
* more dimensions in the solve

### Option B: Embed basis into each projection curve

You just build separate 3M and 6M curves with no explicit basis object.
Pros:

* fewer moving parts conceptually
  Cons:
* basis risk becomes “mystery reshaping” across curves
* harder to stabilize when quotes are sparse

### What I say in an interview

“If basis is sparse, I still prefer an explicit basis object with **strong regularization**. Otherwise the solver hides basis misfit inside the projection curve and you get unstable risks that nobody can explain.”

### Great answer bonus

I mention the governance angle: explicit basis curve makes it much easier to justify why a hedge is a basis swap rather than some random IRS bucket.

---

## Q16) “When calibrating multi-currency, what do you treat as the ‘truth’: FX forwards, XCCY basis swaps, or the two discount curves?”

None of them is “the truth” in isolation. You have a coupled system and you reconcile with weights and tolerances that reflect liquidity by tenor.

### The actual coupled structure

* Each currency has discounting curve(s) under relevant CSA(s).
* FX forwards link discount factors across currencies (CIP-style relationship, but basis exists).
* XCCY basis swaps tie the two legs (discounting + projection + FX conversion + basis).

So the solve is:

* fit FX forwards (often very liquid front-end)
* fit XCCY basis swaps (liquid at certain tenors, weaker at others)
* fit each currency’s OIS instruments
  with **tenor-dependent weights**.

### My desk policy (typical)

* **Front end (1D to ~1Y):** FX swaps/forwards are usually most reliable and tight, so they get higher weight.
* **Core maturities (1Y–10Y):** XCCY basis swaps might dominate for basis, while OIS dominates for local discount.
* **Long end:** liquidity can be patchy, so regularization plays a bigger role and tolerances widen.

### Great answer bonus

I explicitly say: “I don’t force exact CIP in a world with basis. I fit the tradables with realistic weights, and I monitor where the inconsistency is being absorbed.”

---

## Q17) “How do you handle inconsistent FX forward points vs XCCY basis quotes?”

This happens constantly because they’re marked at different times/sources and have different liquidity profiles.

### My triage order (desk-real)

1. **Cut alignment**

* ensure FX forwards and XCCY basis are from the same timestamp/market cut
* check source: broker composite vs single venue

2. **Conventions sanity**

* spot lag mismatch (T+2 vs T+1 in some markets)
* daycount and payment lags
* holiday calendars (this bites hard at year-end)

3. **Reconcile via weights/tolerances**

* treat the less reliable leg as softer
* widen tolerances for the thinner market
* possibly use a robust loss so an outlier doesn’t wreck the curve

4. **Validate out-of-sample**
   Reprice:

* FX swaps not in calibration
* XCCY swaps at nearby maturities
  If the reconciliation is “cheating,” you’ll see it immediately.

### Great answer bonus

I mention what I *don’t* do: I don’t “fix” it by bending the USD OIS curve, because that poisons everything downstream.

---

## Q18) “How do collateral currencies change your cross-currency curve set architecture?”

Collateral currency changes discounting. That creates **multiple discount curves per currency**, not one.

### Architecture

For each currency (C), you can have discount curves:

* (D^{C|C}(t)): currency C discounted under collateral in C (local CSA)
* (D^{C|USD}(t)): currency C discounted under USD collateral (common in practice)
  And similarly other collateral currencies, depending on CSA population.

### Why it matters

* A EUR trade under USD CSA discounts EUR cashflows using a curve consistent with USD-collateral economics (through XCCY relationships).
* If you use the wrong curve set, PV and risk are wrong, and PnL explain becomes nonsense.

### Great answer bonus

I mention operational controls: trade metadata must select the curve set deterministically (CSA id, collateral currency, clearing house), and the build system must support parallel curve sets.

---

## Q19) “How do you design your curve so PnL explain is stable day-to-day?”

Stability is mainly about controlling the mapping:
$$
\text{market quotes } q ;\rightarrow; \text{curve parameters } x ;\rightarrow; PV
$$
PnL explain breaks when (x(q)) changes “too much” for small changes in (q), or when model degrees of freedom shift around.

### What I enforce

1. **Consistent instrument set / hierarchy**

* Don’t let today’s curve depend on a random quote that disappears tomorrow without a controlled fallback.

2. **Regularization tuned to stability**

* Enough to prevent spiky node responses
* Not so much that misfit gets pushed into wrong places systematically

3. **Temporal smoothness (for live curves especially)**

* Penalize deviation from prior curve during intraday streaming:
$$
  \eta |x - x_{\text{prev}}|^2
$$
  with safeguards so true market moves still pass.

4. **Constraint activity monitoring**
   If constraints bind intermittently, risk can “snap.”
   So I track:

* which bounds are active
* where curvature penalties are dominant
* and alert when the regime changes.

### Great answer bonus

I mention “explain-friendly decomposition”: discounting vs forwarding risk attribution and consistent bucket mapping.

---

## Q20) “How do you explain to a trader why two curve builds with similar fit give very different bucket risk?”

Because **fit is not risk**. Risk depends on the calibration mapping, i.e. how the curve parameters move when quotes move.

### The clean explanation

Two builds can have similar residuals but different:

* node parameterization (DFs vs forwards)
* regularization strength / penalty structure
* knot placement
* constraint activity

All of those change:
$$
\frac{\partial x}{\partial q}
$$
and hence bucket DV01 / key-rate risks.

### How I demonstrate it (no handwaving)

* Run a 1bp bump to a key quote.
* Compare the resulting node moves (\Delta x) between the two builds.
* Show that build A concentrates response locally (stable) while build B spreads or spikes it (unstable).

If I want to be very explicit:

* compare norms or principal components of node move vectors.

### Great answer bonus

I mention a practical outcome: “If you’re hedging, you want the curve where the sensitivity mapping is stable. A perfect fit that produces jumpy bucket risk is operationally expensive.”

---

# Summary table (Q13–Q20)

| Q# | Topic                            | What I’m proving                       | What breaks if you’re wrong          |
| -: | -------------------------------- | -------------------------------------- | ------------------------------------ |
| 13 | Multi-tenor consistency          | tenors are coupled; basis + priors     | oscillating basis, nonsense crossing |
| 14 | Basis misfit attribution         | block-Jacobian debugging               | blind tuning of weights/penalties    |
| 15 | Sparse basis strategy            | explicit basis vs embedded trade-off   | “mystery basis risk” in projection   |
| 16 | Multi-ccy “truth”                | reconcile tradables by tenor liquidity | forced CIP, unstable basis/FX        |
| 17 | FX vs XCCY inconsistency         | cut/conventions then weights           | poisoning anchor curves              |
| 18 | Collateral currency architecture | multiple CSA curve sets                | wrong PV/risk, PnL explain fails     |
| 19 | PnL explain stability            | stable (q\to x\to PV) mapping          | snapping risk, daily explain noise   |
| 20 | Fit vs risk                      | risk = (\partial x/\partial q)         | traders hedge the wrong thing        |

Next batch is **Q21–Q27** (data hygiene, validation packs, regime shifts, determinism, and the “advanced subtleties” that make people trust you in prod).

----
---

## Q21) “You see a persistent residual pattern: futures strip fits, but short OIS doesn’t (or vice versa). What do you check first?”

First thing: I assume it’s **not** a “curve shape problem.” It’s almost always **conventions, calendars, or window definitions**.

### My triage order (the order that saves hours)

1. **Cut / timestamp alignment**

* Are the futures and OIS quotes from the same market cut?
* Are you mixing “live streaming futures” with “end-of-day OIS runs”?

2. **Calendar + accrual windows**

* For FF futures: month-average means *actual calendar month* with correct business-day/daycount handling.
* For OIS: coupon accrual schedule uses business day adjustments, payment lag, stubs.
* If your futures month window is off by even 1 day (holiday/weekend handling), residuals appear as a clean pattern across contracts.

3. **Meeting-date / turn nodes present and correctly placed**

* If you’re missing a meeting effective date knot, futures month averaging forces the solver to smear a jump; OIS that straddles the meeting will misfit.
* Same for **turns** (month-end/quarter-end/year-end): if turn treatment is missing, you’ll see systematic short-end residuals.

4. **Daycount + compounding conventions**

* OIS compounding vs simple averaging differences.
* Act/360 vs Act/365F inconsistencies.
* Payment lag applied incorrectly (discounting at wrong date).

5. **Futures convexity treatment consistency**

* If you apply convexity on futures but not on OIS (or vice versa) in a mismatched way, you can create a persistent strip-vs-swaps mismatch.

### A quick diagnostic that’s very telling

* Reprice a **1W OIS**, **2W OIS**, and a **meeting-month FF future**.
  If those three can’t all be made consistent with sane forwards, the bug is usually in the date/window plumbing, not the solver.

---

## Q22) “How do you handle outliers without destabilizing the curve?”

Outliers are guaranteed. The question is whether your engine responds like a professional system or like a toddler throwing the curve across the room.

### My approach (robust + auditable)

1. **Do not delete quotes silently**
   If a quote is “ignored,” it must be logged with a reason: stale, off-market, wide bid/ask, known bad source.

2. **Use robust loss (instead of pure least squares)**
   Classic least squares overreacts because large residuals get squared.

A common robust choice is **Huber**:
$$
\rho(r)=
\begin{cases}
\frac{1}{2}r^2 & |r|\le \delta\
\delta|r|-\frac{1}{2}\delta^2 & |r|>\delta
\end{cases}
$$
So big misses don’t dominate the solve.

Implementation: IRLS (iteratively reweighted least squares) where weights shrink for large residuals:
$$
w_i \leftarrow \frac{\psi(r_i)}{r_i}
$$
3. **Anchor hierarchy**

* Cash/very front-end anchors usually get higher trust.
* Illiquid basis quotes get lower weight and wider tolerances.

4. **Outlier containment**
   Even with robust loss, I also apply **shape guardrails** (bounds + curvature penalties) so a single bad quote can’t create a localized kink.

### “Great answer bonus”

I mention **bid/ask-aware tolerances**: normalize residuals by bid/ask so the objective is really “how many bid/ask widths off am I,” not “how many bps off.”

---

## Q23) “What’s your minimum daily validation pack for curves used for risk?”

If the curve is used for risk, the validation pack is basically “prove this won’t embarrass us.”

### Minimum pack I’d insist on

**A) Fit diagnostics**

* Residuals by instrument type/tenor (OIS, futures, swaps, basis, FX fwd, XCCY)
* Summary stats: mean, RMS, percentiles
* Count of instruments outside tolerance

**B) Shape / no-arb / sanity**

* DF positivity and monotonicity
* Forward bounds (regime-aware)
* Kink metrics:

  * (\max |\Delta f|)
  * (\max |\Delta^2 f|)
* Turn and meeting behavior sanity (no “fake turns” showing up mid-month)

**C) Stability diagnostics (the stuff that actually matters)**

* Bump/rebuild response norms:

  * bump key quotes by 1bp, rebuild, record max node move and where it occurs
* Active-set / constraint binding report:

  * where are bounds binding today?
* Conditioning proxy:

  * warning if (J^\top WJ) is near-singular (or a proxy metric)

**D) Out-of-sample repricing**

* Reprice a set of “checks” not used in calibration:

  * nearby tenors, alternative broker sets, secondary instruments

**E) Multi-currency consistency checks (if relevant)**

* FX forwards repricing errors
* XCCY basis repricing errors
* Identify which curve is absorbing inconsistencies (USD anchor poisoning detector)

### “Great answer bonus”

I explicitly say: “I want **alerts**, not pretty reports.” Thresholds with escalation rules.

---

## Q24) “How do you compute risk fast if a full rebuild per bump is expensive?”

Full bump-and-rebuild is the gold standard, but it’s slow. The VP answer is: **use linearization intelligently**, validate it, and know when it breaks.

### The main speed-up: Jacobian-based “implicit risk”

Curve calibration solves:
$$
\min_x ; |r(x,q)|^2 + \lambda |Lx|^2
$$
At the solution, small quote bumps (\Delta q) cause parameter changes (\Delta x) given by a linear system.

Under Gauss–Newton/LM, locally:
$$
(H)\Delta x \approx -J^\top W \Delta r_q
$$
where:

* (J=\frac{\partial r}{\partial x})
* (\Delta r_q = \frac{\partial r}{\partial q}\Delta q)
* (H = J^\top W J + \lambda R) (plus LM damping if used)

So instead of rebuilding for each bump, you:

1. factor (H) once (Cholesky/LDL)
2. for each bump compute RHS and solve for (\Delta x) fast

Then PV sensitivity:
$$
\Delta PV \approx \frac{\partial PV}{\partial x}\Delta x
$$
### Where this works well

* small bumps (1bp)
* constraints not changing status
* stable market regimes

### Where it fails (and you must say this)

* when constraints bind/unbind (active set changes)
* big bumps / stress scenarios
* nonlinearity in instruments (esp. exotic mapping or large convexity effects)

### “Great answer bonus”

I mention a hybrid policy:

* Use Jacobian risk intraday for speed
* Daily EOD checks with full rebuild bumps to validate linearization drift

---

## Q25) “What do binding constraints do to sensitivities, and how do you explain that to a trader?”

Constraints make the mapping (q \to x) **piecewise** and sometimes **non-smooth**.

### What happens mathematically

If you enforce bounds like:
$$
f_k \ge 0,\quad | \Delta f_k| \le s_{\max}
$$
the solution can hit a boundary. When that happens:

* the solver is effectively operating on a reduced set of free variables (active set)
* infinitesimal bumps may not move constrained nodes at all
* when a bump causes the active set to change, sensitivities can “jump”

So (\partial x/\partial q) is not globally smooth.

### How I explain it to a trader (practical language)

“When the curve is up against a hard guardrail, small market moves can’t push it further in that direction, so the risk re-routes to neighboring buckets. If the guardrail stops binding, bucket risk can shift. That’s why we monitor constraint activity.”

### Desk tip

I report:

* which constraints are binding
* and where
  because that explains “why did DV01 move buckets today without prices moving much?”

---

## Q26) “How do you keep intraday live curves from flip-flopping when quotes stream?”

Live curves flip-flop because:

* quotes update asynchronously
* some prints are stale or spiky
* liquidity varies by tenor
* the solve is underdetermined locally

### Production tactics I use

1. **Warm-start the solver**
   Start from the last good curve so it doesn’t rediscover the universe every tick.

2. **Time smoothing / prior anchoring**
   Add a penalty to not move too far from the previous curve:
$$
   \eta |x - x_{\text{prev}}|^2
$$
   This is not “cheating.” It’s acknowledging that microstructure noise shouldn’t reshape the curve.

3. **Quote filtering + hysteresis**

* ignore tiny changes within bid/ask noise
* require persistence before accepting a move (especially for illiquid instruments)

4. **Tenor-dependent weights**
   Front-end liquid quotes move the curve more than illiquid long-end marks.

5. **Publish gating**
   If diagnostics fail (residual explosion, forward negativity, constraint thrash), don’t publish a new curve. Use last valid.

### “Great answer bonus”

I mention “constraint thrash”: if bounds are binding/unbinding repeatedly intraday, you need either stronger priors or better filtering.

---

## Q27) “What are the most common convention mismatches that specifically corrupt the front-end meeting-month fit?”

This is the list of classic landmines.

### The usual suspects

1. **Wrong accrual window for the futures month**

* mishandling month boundaries with holidays/weekends
* wrong daycount

2. **Meeting date vs meeting effective date**

* knot placed on announcement day but rate effective from next business day (or vice versa, depending on index rules)

3. **Spot lag mismatch**

* FX and XCCY spot conventions inconsistent (T+2 vs local variations)
* short-end FX forwards then conflict with curve expectations

4. **Payment lag not applied**

* discounting at (E) instead of (E+\text{payLag})

5. **Stub rule mismatch**

* short stubs vs long stubs
* EOM rule inconsistencies

6. **Compounding convention mismatch**

* compounded overnight vs simple average approximation used inconsistently

7. **Calendar updates**

* exchange calendar differs from internal calendar (yes, this happens, and yes, it ruins lives)

### “Great answer bonus”

I say: “I keep a micro test suite”: one meeting-month future, one short OIS straddling meeting, one turn-window instrument. If any of those reprices drift, it’s a conventions regression.

---

# Summary table (Q21–Q27)

| Q# | Topic                       | What a VP answer proves              | Key deliverable in production                          |
| -: | --------------------------- | ------------------------------------ | ------------------------------------------------------ |
| 21 | Strip vs short OIS mismatch | debug discipline (conventions first) | deterministic triage checklist + micro tests           |
| 22 | Outliers                    | robustness without hiding reality    | robust loss + bid/ask-aware tolerances + logging       |
| 23 | Validation pack             | you can run curves in risk           | residuals + shape + stability + out-of-sample + alerts |
| 24 | Fast risk                   | you know the math and its limits     | Jacobian/implicit risk + rebuild validation            |
| 25 | Binding constraints         | you understand non-smooth risk       | active-set reporting + trader explanation              |
| 26 | Intraday stability          | you can ship a live curve            | warm-start + time priors + filtering + publish gating  |
| 27 | Front-end convention traps  | you’ve been burned before            | regression tests + calendar/convention controls        |

Next batch is the **next set of questions from the second list** (representation/continuity/conditioning, meeting-knot policy, etc.) if you want to keep going in strict order.

Continuing. **Q21–Q27**, same “I’m the VP strat being interviewed” style, with the gritty production details people pretend don’t exist.

----
---
Got it. Since we already got up to **Q27** in the previous run, I’ll continue numbering here. Your pasted list’s **Q1–Q20** becomes **Q28–Q47**.

(Yes, humans would restart at 1 and pretend nothing happened. We won’t.)

---

## Q28) “Explain the difference between discounting and projection curves. What breaks if you mix them?”

On a modern rates desk:

* **Discounting curve** answers: *“What’s the PV today of $1 paid at time (t) under the trade’s collateral/funding rules?”*
  That’s the curve that gives you discount factors (D(t)). Under CSA/OIS discounting, this is typically an OIS curve consistent with the collateral rate.

* **Projection curve** answers: *“What forward rate do I expect for the index that sets my floating coupons?”*
  That gives you forwards (F(t_{i-1},t_i)) (e.g., 3M, 6M).

### The clean pricing split

For a vanilla swap (simplified):
$$
PV = \sum_i D(t_i),\alpha_i,(F^{(idx)}(t_{i-1},t_i) - K)
$$
* (D(\cdot)) comes from **discount curve**
* (F^{(idx)}) comes from the **projection curve for that index**

### What breaks if you mix them

1. **Basis instruments misprice immediately**
   Anything like OIS-vs-3M or 3s6s basis is *literally* the statement that discounting and forwarding are not the same object. If you use one curve for both, basis PVs won’t line up.

2. **Wrong hedge ratios (the real desk pain)**
   If you discount with the projection curve (or project with the discount curve), your risk decomposition is wrong:

   * “funding DV01” vs “index DV01” gets blended into nonsense
   * you hedge with the wrong instruments (or wrong tenors)

3. **PnL explain becomes a crime scene**
   You’ll see stable PV but unstable attribution, because your curve mapping is economically inconsistent with how the market trades collateral and term funding.

---

## Q29) “Why would you solve curves simultaneously rather than bootstrap sequentially?”

Sequential bootstrapping is fine in a fantasy world where markets are perfectly consistent and instruments are independent. Desks do not live there.

### Why simultaneous is used

1. **Coupled instruments create coupled unknowns**

* Basis swaps depend on multiple curves.
* XCCY depends on two discount curves + FX forwards + basis.
  So your residual system is inherently joint.

2. **Quotes are inconsistent; you need best-fit with priorities**
   You almost always have:

* bid/ask noise
* stale points
* conflicting broker runs
  So you solve:
$$
  \min_x \sum_i w_i r_i(x)^2 + \lambda|Lx|^2
$$
  instead of forcing exact sequential fits that push inconsistencies into hidden corners.

3. **Risk stability is better**
   Simultaneous solving typically yields a more coherent mapping:
$$
   q \rightarrow x(q)
$$
   so bump/rebuild risk behaves. Sequential builds often create “frozen errors” that show up as unstable Jacobians.

### When desks still use sequential

* For speed
* For legacy reasons
* When the instrument set is mostly hierarchical (e.g., discount curve is cleanly determined, basis is sparse)

But even then, they usually add *some* global fit/regularization layer.

---

## Q30) “What are ‘ghost instruments’ in curve building?”

Ghost instruments are **implied constraints** created by the *method*, not by the market.

### Typical examples

* In a sequential bootstrap you create an “implied forward” or “implied FRA” as an intermediate step, then later treat that as if it were a real constraint.
* Some systems build synthetic nodes that effectively become pseudo-quotes.

### Why they’re dangerous

* **Risk attaches to the ghost**, not the actual market quote.
  So the desk bumps a real quote and the curve “moves” somewhere unexpected.
* They make your curve system **path-dependent** (depends on build order).
* They can produce “looks fine in PV, weird in DV01” behavior.

### How I avoid them (or at least tame them)

* Prefer **instrument-based global solves** where residuals correspond to real tradables.
* If I must use implied constructs, I keep them internal and ensure risk attribution maps back to the true quote set.

---

## Q31) “How do Fed Funds futures constrain the OIS discount curve? What’s the model equation?”

Fed Funds futures (conceptually) pin the **month-average** effective overnight rate.

If the futures implies monthly average (R^{mkt}*M), the model produces:
$$
R^{model}*M = \frac{\sum*{d\in M} r(d),\Delta_d}{\sum*{d\in M}\Delta_d}
$$
With a segmented forward representation (r(d)=r_j) over segments ((T_{j-1},T_j]), this becomes:
$$
R^{model}*M = \frac{\sum_j r_j \cdot \text{DC}((T*{j-1},T_j]\cap M)}{\text{DC}(M)}
$$
Residual:
$$
r_i(x)=R^{model}_M - R^{mkt}_M
$$
### Why meeting knots matter here

If a meeting happens inside month (M), you need the curve to express different pre/post levels:
$$
R^{model}*M = \frac{n*{pre}r_{pre}+n_{post}r_{post}}{n_{pre}+n_{post}}
$$
Without a knot at the boundary, the solver has to invent curvature to mimic that weighted average.

### Desk realism

* You must get **calendar/daycount/windows** exactly right.
* Many “FF strip doesn’t fit” problems are just window bugs.

---

## Q32) “What happens if you don’t include meeting-date nodes?”

Then the solver has two options, both bad:

1. **Smear the jump across time**

* Produces a fake slope across weeks
* Misprices short OIS or instruments that straddle the meeting window

2. **Create a random kink elsewhere**

* Fits some instruments by introducing local curvature in a non-economic spot
* Bucket DV01 shifts unpredictably (risk instability)

### Cross-currency knock-on

If USD front-end discounting is wrong, it leaks into:

* FX forward points (short maturities are sensitive to DF ratios)
* XCCY basis calibration (front-end basis kinks appear as compensation)
* “Why is EURUSD basis DV01 moving when nothing moved?” kind of desk nightmares

---

## Q33) “Cash ON/TN/SN: do you treat them as hard anchors?”

Practically: **yes, as near-hard anchors**, meaning very high weight or explicit pinning.

### Why

* The very front end is tightly known: realized fixings and near-term funding are not ambiguous.
* Letting the solver distort ON/TN/SN to fit noisy deferred instruments is backwards.

### Implementation patterns

* Pin the first segment(s) (r_1) to ON/TN/SN with tight tolerances
* Or apply massive weights (w_{cash}) so they dominate locally
* Ensure spot lag and holiday calendars are correct (cash instruments are brutal about calendars)

### Failure mode if you don’t anchor

Your curve will “solve” disagreements by shifting the very front end, which cascades into every PV and turns risk into soup.

---

## Q34) “How do you handle turns (month-end/quarter-end/year-end) vs meeting effects?”

Both are **localized rate distortions**, but they come from different economics.

* **Turns**: funding/collateral scarcity and balance-sheet effects around reporting dates
* **Meetings**: policy expectation jumps

### How desks handle both

1. Include **turn nodes** (special short segments around month-end/quarter-end/year-end)
2. Include **meeting nodes** (policy effective boundaries)
3. Use **localized regularization**

* Don’t let global smoothing flatten these real spikes/jumps.
* Penalize curvature generally, but relax across known structural boundaries.

### Practical sanity check

* Turn effects should be localized and not “echo” months later.
* Meeting jumps should align to meeting effective date, not drift into adjacent weeks.

---

## Q35) “Where do slope/curvature caps come from? How do you justify them?”

They come from three places: **market resolution, stability requirements, and governance**.

### A) Market resolution (liquidity + instrument spacing)

If quotes don’t trade at weekly granularity, allowing the curve to wiggle weekly is just overfitting. Caps reflect the maximum plausible change between adjacent knots.

### B) Risk stability targets

Caps/penalties are tuned so bump/rebuild doesn’t create node spikes:

* monitor (|\Delta x|) for 1bp bumps
* set caps so spikes don’t happen unless a real structural event (turn/meeting) is present

### C) Historical distributions

Compute (|\Delta f|) and (|\Delta^2 f|) from historically accepted curves, set caps near 95–99th percentile, regime-aware.

### What I say to justify it

“These are **shape priors** that prevent the solver from manufacturing microstructure unsupported by the market. They’re calibrated against bid/ask tolerances and risk stability diagnostics.”

---

## Q36) “Explain how regularization affects hedge ratios.”

Regularization changes the **mapping from quotes to curve nodes**, and that mapping drives hedge ratios.

At the linearized level:
$$
\Delta x \approx -(J^\top WJ + \lambda R)^{-1}J^\top W,\Delta r
$$
So (\lambda) and (R) directly shape (\Delta x), hence:

* **bucketed DV01**
* **key-rate risk**
* how risk migrates across maturities

### What regularization does (good)

* reduces spiky forwards
* stabilizes (\partial x/\partial q)
* makes hedges more consistent day-to-day

### What too much regularization does (bad)

* forces misfit into other instruments/maturities
* creates systematic bias (curve becomes “too smooth” and economically wrong)
* can hide real structure (turns/meeting jumps)

### How I tune it

* fit within bid/ask on core instruments
* minimize instability metrics (bump/rebuild spikes)
* ensure structural features (turns/meetings) are preserved where appropriate

---

## Q37) “What constraints are truly ‘no-arb’ vs just ‘shape control’?”

### No-arb-ish (hard sanity)

* (D(t)>0)
* Usually monotonic DF: (D(t_{k+1})\le D(t_k)) (for non-negative discount rates)
* Bounds to prevent impossible rates in a given regime (sometimes)

### Shape control (priors, not laws)

* slope caps: (|\Delta f|\le s_{max})
* curvature penalties: (\sum (\Delta^2 f)^2)
* monotone interpolation filters to prevent overshoot

### VP framing

“No-arb constraints prevent impossible economics; shape constraints prevent the optimizer from overfitting noise and destroying risk stability.”

---

## Q38) “How do you detect you’re overfitting the curve?”

Overfitting shows up more in **forwards and risk** than in PV residuals.

### Red flags

1. **Forwards oscillate between sparse instruments**

* wiggles where there is no liquidity support

2. **Tiny quote bumps cause huge node moves**

* bump/rebuild response spikes
* unstable Jacobians

3. **Residuals look perfect but out-of-sample checks fail**

* instruments not in calibration misprice noticeably

4. **Constraint thrash**

* bounds binding/unbinding across small market moves

### Practical tests

* Remove one instrument (“leave-one-out”), rebuild: does the curve stay sane?
* Compute kink metrics and bump response norms daily.
* Check stability across days with similar market levels.

---

## Q39) “You have inconsistent front-end quotes: cash says one thing, futures another. What do you do?”

I do not panic. I assume a convention bug first, then a market microstructure issue.

### Step-by-step triage

1. **Data hygiene**

* calendars, daycount, accrual windows
* meeting date placement
* pay/fix lags
* timestamps / market cuts

2. **Liquidity hierarchy**

* ON/TN/SN and very short OIS usually more reliable than far-deferred futures prints
* identify which quotes are actually tradable and tight

3. **Robust fitting**

* widen tolerance on suspicious quotes
* downweight them
* use robust loss (Huber) so one bad quote doesn’t dominate

4. **Degree-of-freedom sanity**

* ensure meeting/turn nodes exist so genuine structure can be represented
* don’t create extra freedom just to force-fit a bad point

### Outcome I aim for

* fit core liquid set within bid/ask
* keep shape stable and risk sane
* log and explain any downweighting

---

## Q40) “How do you choose which instruments go into the calibration set vs used as checks?”

### Calibration set (solve)

* most liquid, most reliable, tightest markets
* instruments that define the curve’s main structure
* stable sources, consistent conventions

### Check set (validate)

* slightly less liquid
* alternative sources (broker A vs broker B)
* nearby tenors not in solve (out-of-sample)
* anything you suspect might be noisy but still informative

### Key principle

The calibration set should be **small enough to be stable** and **rich enough to define the curve**. Everything else is a diagnostic, not a steering wheel.

---

## Q41) “Explain the dependency tree: why build the collateral currency first?”

If you have a common collateral currency (often USD), its discount curve becomes the **anchor numeraire** for many CSA discounting relationships.

### Why build it first (practical)

* You need a stable USD collateral discount curve to value cashflows discounted under USD CSA.
* XCCY instruments reference both discount curves + FX forwards; a stable anchor reduces degrees of freedom and helps convergence.

### Important nuance

This is **architecture**, not economics. Economically it’s a coupled system. But building the anchor first can make the system more stable and faster in production.

---

## Q42) “Where does cross-currency basis live in the equations?”

In an XCCY swap PV, you discount each leg in its currency and convert via FX.

Simplified idea (not full legal detail):
$$
PV = PV_{\text{dom leg}}(D_{dom},F_{dom}) - S_0 \cdot PV_{\text{for leg}}(D_{for},F_{for},FX_fwd) + PV_{\text{basis}}
$$
The “basis” is the spread you add on one leg so the PV clears to par.

FX forwards relate discount curves through a CIP-like relationship adjusted by basis:
$$
F(0,T) \approx S_0 \frac{D_{dom}(T)}{D_{for}(T)} \times \text{(basis adjustment)}
$$
### Desk reality

The basis object is “whatever function makes the market XCCY swaps and FX forwards consistent with the curve set you’re using,” within tolerances and weights.

---

## Q43) “How does a wrong USD meeting-date shape propagate into EUR-USD XCCY?”

Front end XCCY is very sensitive to **short-dated DF ratios**.

### Propagation chain

1. Wrong USD meeting shaping → wrong USD short-end (D_{USD}(T)) ratios
2. FX forward points implied by those ratios shift
3. When calibrating XCCY, the solver must reconcile:

* EUR discount curve
* USD discount curve
* FX forwards
* XCCY basis quotes

4. It “fixes” the inconsistency by kinking the **basis curve** or distorting another component

### Symptoms you see

* basis DV01 instability around meeting months
* FX forward repricing errors clustered in front-end tenors
* “basis kinks” that disappear if you rebuild USD with meeting nodes correctly

---

## Q44) “What solver do you use for simultaneous calibration and why?”

### Standard choice

* **Gauss–Newton / Levenberg–Marquardt** for weighted least squares:
$$
  (J^\top WJ + \lambda R + \gamma I)\Delta x = -J^\top Wr
$$
* (\gamma) is LM damping to handle nonlinearity / poor conditioning.

### Constraints handling

* projection (clip to bounds) for simple box constraints
* barriers / penalties for smooth enforcement
* SQP / interior point if you need strict constraints and complex inequalities

### Why this is VP-relevant

* It’s fast
* Jacobians can be computed efficiently (analytic/AD)
* You can monitor conditioning, convergence, and constraint activity

---

## Q45) “How do you compute bucketed risk correctly in a multi-curve system?”

Correct answer: **bump market quotes, rebuild, reprice**.

### Why bumping nodes is wrong

Because the curve nodes are not tradable quotes. Risk must reflect:
$$
q \rightarrow x(q) \rightarrow PV
$$
If you bump (x) directly, you’re assuming (x) is independent, which ignores the calibration mapping.

### Practical workflow

1. choose quote set (OIS, futures, swaps, basis, FX fwd, XCCY)
2. for each quote (q_i), bump (q_i \pm 1bp)
3. rebuild the entire coupled curve set
4. compute PV difference

### Speed-ups desks use

* cached instrument schedules and accrual factors
* parallel bumps
* Jacobian/implicit risk approximations for small bumps (validated against full rebuild)

---

## Q46) “What production checks do you run daily?”

Minimum daily checks for a risk curve:

1. **Residual dashboard**

* by instrument type and tenor
* outliers flagged vs tolerance

2. **Sanity / no-arb**

* DF > 0
* DF monotonicity (as applicable)
* forward bounds (regime-aware)

3. **Kink metrics**

* (\max|\Delta f|)
* (\max|\Delta^2 f|)
* join discontinuities (front grid to long grid)

4. **Stability**

* bump/rebuild response norms
* constraint binding report

5. **Out-of-sample**

* reprice checks not used in calibration (vendor/broker alternatives)

---

## Q47) “What are the biggest curve build ‘gotchas’ that break desks?”

The classics. If you say these cleanly, you sound like someone who has suffered in production (which is, unfortunately, the job requirement).

* **Calendar/date logic bugs**: holidays, spot lags, stub rules, roll conventions
* **Meeting/turn nodes missing or mislocated**
* **Wrong daycount or compounding**: Act/360 vs Act/365F, daily compounding vs average
* **Interpolation choice creating negative forwards or oscillations**
* **Equal weighting of illiquid quotes** (how to destroy stability in one line)
* **Regime shifts**: negative rates, crisis funding, benchmark transitions (LIBOR→RFR), policy shock weeks
* **Timestamp mixing**: using quotes from different cuts and forcing reconciliation

---

# Summary tables

### Mapping from your pasted list to this numbering

| Your pasted Q# | Here | Topic                      |
| -------------: | ---: | -------------------------- |
|              1 |   28 | Discount vs projection     |
|              2 |   29 | Simultaneous vs sequential |
|              3 |   30 | Ghost instruments          |
|              4 |   31 | FF futures constraint      |
|              5 |   32 | Missing meeting nodes      |
|              6 |   33 | Cash anchors               |
|              7 |   34 | Turns vs meetings          |
|              8 |   35 | Slope/curvature caps       |
|              9 |   36 | Regularization & hedges    |
|             10 |   37 | No-arb vs shape control    |
|             11 |   38 | Overfitting detection      |
|             12 |   39 | Cash vs futures conflict   |
|             13 |   40 | Calibration vs checks      |
|             14 |   41 | Dependency tree            |
|             15 |   42 | Where basis lives          |
|             16 |   43 | USD meeting propagation    |
|             17 |   44 | Solver choice              |
|             18 |   45 | Correct bucket risk        |
|             19 |   46 | Daily production checks    |
|             20 |   47 | Gotchas                    |

### “VP answer ingredients” checklist (desk version)

| Ingredient                                  | What it signals                                             |
| ------------------------------------------- | ----------------------------------------------------------- |
| Conventions-first debugging                 | you won’t waste a day tuning (\lambda) for a calendar bug   |
| Quote-to-functional mapping                 | you actually know what instruments constrain                |
| Stability framing ((\partial x/\partial q)) | you understand why traders scream about DV01 buckets        |
| Tenor- and liquidity-aware weighting        | you know markets aren’t equally real everywhere             |
| Governance/logging                          | you can run this in production without gaslighting everyone |

If you want to continue, the next step is answering the *remaining* question set we created later (representation/continuity/conditioning, collateral optionality, determinism tests, etc.) while keeping numbering consistent after **Q47**.

---
---
Sure. You want the **“change one knob → three other things catch fire”** questions. That’s basically half the job on a multi-curve / multi-currency rates desk.

I’ll continue numbering from **Q47**, and for each one I’ll **ask the question and answer it in depth** like I’m being interviewed.

---

## Q48) “If you increase regularization on the discount curve, what else changes in a multi-curve system?”

### What you changed

You increased smoothness/curvature penalty on the **discount** curve (CSA/OIS), e.g.
$$
\min_x \sum_i w_i r_i(x)^2 + \lambda_d |L f_d|^2
$$
with larger (\lambda_d).

### What it affects downstream (mechanism)

1. **All PVs change** (because every cashflow is discounted with (D(t))).
   Even projection instruments (IRS, basis swaps) depend on discounting via (D(t_i)).

2. **Projection curve calibration shifts**
   If discounting is “held smoother,” the solver has fewer degrees of freedom to fit OIS instruments. Residual tension gets pushed into:

* projection curves (3M/6M)
* basis curves (OIS–IBOR, 3s6s, etc.)
  because those are also allowed to move to reduce global residuals.

3. **Risk allocation changes**
   Regularization changes the mapping:
$$
   \frac{\partial x}{\partial q}
$$
   So DV01 buckets that were “discount-driven” may migrate into “forward-driven” risk.

### What you see on a desk

* OIS residuals might worsen slightly (still within tolerance), but
* basis swaps start showing kinks or more curvature,
* IRS par fits shift in specific maturities,
* bucketed DV01 looks “smoother” on the discount curve but more jumpy on basis.

### How to do it safely

* Tighten discount regularization **only if** OIS fit remains within bid/ask **and** basis doesn’t get polluted.
* Monitor “residual migration”: if tightening (\lambda_d) increases basis residuals systematically, you’ve just moved the problem, not solved it.

---

## Q49) “If you add a meeting-date knot (or move it by 1 day), what else breaks besides the front end?”

### What you changed

You inserted/shifted a knot in the discount curve grid at a meeting effective date.

### Downstream effects

1. **FF futures constraints change**
   Futures are averages over a month. Changing a knot changes the weights in:
$$
   R^{model}_M=\frac{\sum_j r_j \cdot DC(segment_j \cap month)}{DC(month)}
$$
   A one-day shift can change the implied pre/post split.

2. **Short OIS swaps reprice**
   Anything spanning the meeting period changes because the integral of ON forward differs.

3. **Cross-currency FX forward points shift (multi-currency)**
   FX forwards depend strongly on short-dated DF ratios. If USD discount DFs around (T) move, then:
$$
   F(0,T) \approx S_0 \frac{D_{USD}(T)}{D_{EUR}(T)} \times (\text{basis adj})
$$
   So your FX forwards repricing errors and XCCY basis fit will react.

4. **Bucket risk “moves”**
   Meeting knots concentrate sensitivity into a narrower time region, which changes key-rate allocations.

### What you see

* Better fit for meeting-month futures but suddenly worse fit for adjacent short OIS or FX swaps (if your knot placement is inconsistent with index conventions).
* XCCY front-end basis kinks appear if USD is the anchor.

### Safe approach

* Tie meeting knots to the **index-effective boundary**, not the announcement day.
* Run a “micro suite”: meeting-month future + short OIS straddling meeting + 1W/2W OIS.

---

## Q50) “If you tighten bounds on projection forwards (e.g., enforce (L_k \ge 0)), what else changes?”

### What you changed

A hard constraint like:
$$
L_k \ge 0 \quad \forall k
$$
(or tightened (L_{min})) on a projection curve (3M/6M).

### Downstream effects

1. **Basis absorbs tension**
   If the market implies slightly negative forwards (or simply needs low forwards for fit), the constraint blocks the projection curve from moving. The solver compensates by:

* shifting discount curve (if allowed),
* bending basis curve,
* or degrading fit on instruments that depend on those forwards.

2. **Discount curve may distort**
   Because PV has two knobs (discounting and forwarding), blocking forwarding forces discounting to “help” fit IRS par conditions.

3. **Risk becomes non-smooth**
   When constraints bind, sensitivities become piecewise. A tiny quote move can switch which nodes are binding → bucket DV01 can jump.

### What you see

* Constraint binding flags turn on in specific maturities.
* IRS fits stay okay, but basis curve becomes unnatural.
* Sensitivities become sensitive to solver state (“constraint thrash”).

### Desk rule

Bounds must be **regime-aware**. If the currency historically traded negative, forbidding negative forwards is just wrong. Use soft floors with penalties instead of hard clamps if you must.

---

## Q51) “If you change interpolation on the discount curve (e.g., cubic → monotone), what else changes?”

### What you changed

Interpolation scheme between knots.

### Downstream effects

1. **Instantaneous forwards change a lot even if DFs look similar**
   Two interpolations can match the same knot DFs but produce very different (f(t)). That affects instruments sensitive to short segments or local behavior (front-end, turns).

2. **Sensitivity distribution changes**
   Even if PVs remain close, key-rate DV01 buckets can shift because (\partial D(t)/\partial x_k) depends on interpolation.

3. **Cross-currency propagation**
   If short-dated DFs shift even mildly, FX forwards and XCCY basis can show measurable repricing differences at the front end.

### What you see

* Same fit quality, different DV01 ladder.
* Different “kink metrics” (\max|\Delta f|), (\max|\Delta^2 f|).
* Traders complain risk “moved buckets” despite similar market.

### Best practice

* Use monotone schemes for stability in risk curves.
* Validate with bump/rebuild response norms; interpolation should not create unstable Jacobians.

---

## Q52) “If you reweight one instrument bucket (e.g., give more weight to basis swaps), how does the whole system react?”

### What you changed

Weights (w_i) for a class of instruments in:
$$
\min_x \sum_i w_i r_i(x)^2 + \text{regularization}
$$
### Downstream effects

1. **Curve responsibility shifts**
   Higher basis weight means basis instruments get fit more tightly, so other instruments are sacrificed within tolerances:

* IRS residuals may worsen
* OIS residuals may worsen
* projection curves may twist to satisfy basis

2. **Risk redistributes**
   The set of “active constraints” changes. Increased weight can cause localized overfitting and hence spiky (\partial x/\partial q).

3. **Multi-currency coupling gets more fragile**
   Overweighting XCCY basis or FX forwards can cause distortions in a dependent currency curve, especially if the anchor curve is slightly off.

### What you see

* Residual migration patterns.
* Increased curvature in some curve component where degrees of freedom are available.
* Better fit for the overweighted bucket but worse out-of-sample repricing.

### Desk-style answer

Weights encode liquidity and governance. If you change weights, you must show:

* fit remains within bid/ask where it matters,
* stability metrics don’t worsen,
* out-of-sample checks remain sane.

---

## Q53) “If you remove one futures contract (missing quote) and replace it with a proxy/last value, what else changes?”

### What you changed

You altered the calibration set and the effective information content of the strip.

### Downstream effects

1. **Loss of local anchoring**
   Futures are averages over windows. Removing one month removes a constraint on the average, so the solver can redistribute rate levels across adjacent months.

2. **Meeting-month inference can degrade**
   If the missing contract is a meeting month, you can lose the clean pre/post signal and the solver will smear the jump.

3. **Basis/XCCY front end can drift**
   Because short-end DFs shift subtly, cross-currency instruments can show slightly larger repricing changes than you expect.

### What you see

* Adjacent futures residuals move to compensate.
* Short OIS fit changes.
* DV01 in the 1Y bucket migrates.

### Safe fallback rules

* Use last good only if market hasn’t moved materially.
* Otherwise use a proxy with wider tolerance and log it.
* Avoid “silent fill”; it’s how you get unexplained PnL.

---

## Q54) “If you change the anchor curve build cut (timestamp) but leave dependent curves at another cut, what happens?”

### What you changed

You introduced time inconsistency across inputs.

### Downstream effects

1. **FX forwards and XCCY basis become inconsistent**
   Because those markets move intraday. The system will try to reconcile inconsistent snapshots by bending whichever curve is allowed to move.

2. **Dependent currencies get contaminated**
   If USD anchor is at 16:00 and EUR instruments are at 17:00, the dependent EUR|USD collateral curve will “absorb” that mismatch as basis/FX misfit.

### What you see

* Clean residuals in each local curve but persistent residuals in cross instruments.
* Strange basis kinks or FX forward repricing errors.

### Desk rule

All coupled instruments must use the same cut (or you explicitly treat mismatched buckets as soft with tolerances and don’t pretend it’s a consistent system).

---

## Q55) “If you change the collateral currency for a portfolio (USD CSA → EUR CSA), what else changes besides discounting?”

### What you changed

The discounting numeraire and therefore the entire curve set used for PV.

### Downstream effects

1. **Discount factors change** (obvious)
2. **Implied FX forward consistency changes**
   Because under different collateral, the relevant curve set differs; the relationship between discount curves and FX forwards shifts.
3. **XCCY basis usage and risk attribution changes**
   The XCCY basis curve implied by market instruments under that collateral regime is different.

### What you see

* PV changes not just because discounting changes, but because the curve set used for cross-currency consistency changes.
* Hedge ratios shift: what used to be hedged with OIS may now require more basis/FX hedging.

### VP-grade way to say it

“Changing CSA is changing the pricing measure and the curve set. It affects PV, risk decomposition, and which market instruments are consistent hedges.”

---

## Q56) “If you ‘fix’ a negative forward issue by increasing smoothing, what else can you accidentally break?”

### What you changed

You used regularization to eliminate negative forwards.

### Downstream damage

1. **You can hide real structure**
   Turns and meeting jumps can get flattened.
2. **You can shift misfit into long end**
   If smoothing is global, front-end noise can contaminate far maturities.
3. **You can bias the curve**
   Everything fits “okay” but systematically off vs checks and vendor.

### What you see

* Better no-arb metrics but worse out-of-sample pricing.
* Bucket DV01 becomes smoother but less aligned with tradable hedges.
* Residual pattern emerges by instrument type.

### Better fix

Use targeted:

* monotone interpolation / positivity-preserving parametrization
* local penalties
* explicit event nodes (meeting/turn)
  instead of “turn the smoothing knob to 11.”

---

## Q57) “If you add more knots to improve fit, why can that make risk worse?”

### What you changed

Increased degrees of freedom.

### Downstream effects

1. **Ill-conditioning**
   More parameters with the same instruments → (J^\top WJ) closer to singular.
2. **Overfit**
   Curve starts fitting noise; forwards oscillate.
3. **Risk becomes spiky**
   Small quote bumps produce large node moves (unstable (\partial x/\partial q)).

### What you see

* Similar PV residuals but uglier forward curve.
* Higher bump/rebuild response norms.
* Greater day-to-day DV01 bucket migration.

### Desk rule

Liquidity earns knots. If the market can’t support the DOFs, don’t create them.

---

## Q58) “If you change the basis curve smoothness (make basis ‘very smooth’), what does that do to projection curves?”

### What you changed

You forced basis to be smooth via penalty:
$$
\lambda_b |\Delta^2 b|^2
$$
### Downstream effects

If basis is prevented from expressing local bumps that are real (or at least present in quotes), then the solver will fit those bumps by reshaping:

* 3M projection curve
* 6M projection curve
  because the basis relationship is “stiff.”

So you effectively **push basis structure into projection curves**, which is usually bad operationally because it changes hedging interpretation.

### What you see

* Basis instruments fit okay but projection curve gets unnatural curvature.
* IRS fits shift in maturity zones where basis was stiffened.
* Hedging becomes confusing: you expected basis risk, you got projection risk.

### Best practice

Basis smoothness should reflect basis market liquidity. Too smooth basis in a noisy basis market is just hiding basis risk in the wrong place.

---

## Q59) “If you change the FX forward calibration priority, how does that impact XCCY basis and local curves?”

### What you changed

You increased weight/tolerance tightness on FX forwards.

### Downstream effects

1. **CIP reconciliation shifts**
   To fit FX forwards tightly, the system may need to adjust:

* one discount curve
* or XCCY basis curve
* or both

2. **Basis curve can kink**
   Especially front end where FX forwards are liquid and tight. If you force FX perfect, basis becomes the “garbage collector” for all remaining inconsistency.

3. **Local OIS curve can get contaminated**
   If the dependent curve is allowed to move, it might bend away from its local OIS instruments to satisfy FX.

### What you see

* FX forwards repricing improves
* XCCY basis residuals worsen or curve kinks
* local OIS residuals drift slightly (bad sign)

### Desk approach

Tenor-dependent: fit FX very tight at short tenors where it’s liquid, relax where it’s not, and avoid letting FX dominate long end basis unless that’s truly the market reality.

---

## Q60) “If you change a single daycount or compounding convention in one instrument type, why can it look like a ‘macro curve shift’ elsewhere?”

### What you changed

A convention bug fix (Act/360 vs Act/365F, compounding style, payment lag).

### Downstream effects

Because calibration is global, a systematic convention change changes residuals across a whole instrument bucket. The solver re-optimizes and reassigns structure across curves, which can:

* shift discount and projection levels
* change basis curve shape
* shift FX/XCCY reconciliation

### What you see

* residual patterns disappear in the corrected bucket
* but other buckets now show slight drift because the system is rebalanced
* risk ladders shift because the mapping changed

### The VP point

“This is why convention changes require controlled releases and before/after comparison packs. Otherwise it looks like ‘the market moved’ when it was just math.”

---

# Summary tables

## Change → where the pain shows up

| Q# | You change…                | Primary downstream impact               | Typical symptom                 |
| -: | -------------------------- | --------------------------------------- | ------------------------------- |
| 48 | discount regularization ↑  | basis/projection absorb tension         | basis kinks, DV01 migration     |
| 49 | meeting knot add/shift     | FF/OIS + FX/XCCY front end              | FX fwd misfit, basis DV01 jump  |
| 50 | forward floors tightened   | basis/discount distort, non-smooth risk | binding constraints, thrash     |
| 51 | interpolation scheme       | risk redistribution, forward behavior   | same fit, different DV01        |
| 52 | weights on a bucket ↑      | other buckets sacrificed                | residual migration              |
| 53 | missing futures contract   | local anchoring lost                    | smear around missing month      |
| 54 | inconsistent market cut    | cross-instruments inconsistent          | persistent XCCY/FX residuals    |
| 55 | collateral currency change | curve set change (PV + hedges)          | hedge ratios shift meaningfully |
| 56 | smoothing to fix negatives | hides real structure / biases           | checks fail, residual patterns  |
| 57 | add knots                  | conditioning worsens, overfit           | spiky bump/rebuild response     |
| 58 | basis made too smooth      | projection curves get distorted         | “basis risk becomes IRS risk”   |
| 59 | FX forwards priority ↑     | XCCY basis bends to compensate          | front-end basis kinks           |
| 60 | convention change          | global rebalance                        | looks like macro shift          |

## What I monitor whenever one knob changes

| Monitor                                 | Why it catches propagation               |          |            |    |                                 |
| --------------------------------------- | ---------------------------------------- | -------- | ---------- | -- | ------------------------------- |
| Residual migration by instrument bucket | shows who absorbed the change            |          |            |    |                                 |
| Kink metrics ((\max                     | \Delta f                                 | ), (\max | \Delta^2 f | )) | detects “solver invented shape” |
| Bump/rebuild node response norms        | detects unstable (\partial x/\partial q) |          |            |    |                                 |
| Constraint binding map                  | explains non-smooth risk                 |          |            |    |                                 |
| Out-of-sample repricing set             | catches hidden bias and overfit          |          |            |    |                                 |
| Cross-currency consistency checks       | catches anchor poisoning                 |          |            |    |                                 |

If you want to drill this properly, the best way is: **you pick one of Q48–Q60**, and I’ll push back like an interviewer (“ok, show me exactly which residuals migrate and how you’d prove it’s not a calendar bug”).

---
Fine. Here are **instrument-type-specific “change X → Y breaks”** questions, with **answers**, continuing numbering from **Q60**. This is the stuff desks actually argue about because it’s where “the curve” stops being a line and becomes a dependency graph with attitude.

---

## Q61) “If you tighten the weight on **ON/TN/SN cash** (or enforce it as hard), what happens to **1W–1M OIS** and the **first few futures contracts**?”

**Mechanism**

* ON/TN/SN essentially pins the **very first segment(s)** of the ON forward curve (r(t)).
* Short OIS (1W/2W/1M) PV is mostly an integral of those first segments:
$$
  PV_{OIS} \sim \sum D(t_i)\cdot \text{comp}(r(t)\text{ over }[S,E])
$$
* Futures (Fed Funds / STIR) near-term contracts price averages over windows that include those early days.

**What changes**

* Short OIS fit improves (or becomes forced), but
* the solver must compensate elsewhere:

  * it may bend **post-spot** segments harder,
  * shift **meeting-month** post-meeting segment,
  * or push misfit into the first futures month/IMM contract.

**Desk symptom**

* Great ON fit, but you suddenly see “why did the first futures contract go off by 0.3bp?” and the answer is “because you nailed cash to the wall.”

**Checks**

* Residual migration: ON bucket → first OIS or first futures.
* Bump/rebuild stability in the first 2 weeks.

---

## Q62) “If you correct a **Central Bank meeting effective date** (knot moves 1 business day), what happens to **meeting-month futures**, **short OIS**, and **IMM futures**?”

**Mechanism**

* Meeting-month futures and short OIS are sensitive to the **pre/post split**:
$$
  \bar r = \frac{n_{pre} r_{pre} + n_{post} r_{post}}{n_{pre}+n_{post}}
$$
* Moving the knot changes (n_{pre}, n_{post}) and therefore the implied (r_{post}) needed to fit the same quoted average.

**What changes**

* Meeting-month future residuals can flip sign.
* OIS instruments spanning the meeting reprice.
* IMM futures (if they straddle the same boundary or overlap with that window) shift because their implied forward over the quarter changes.

**Desk symptom**

* “We fixed a calendar bug and now front-end DV01 moved buckets.” Yes. That’s what correct math does.

**Checks**

* Micro-suite: meeting-month future + 1W/2W OIS straddling meeting + nearest IMM future.

---

## Q63) “If you switch the short-end discounting inputs from **Fed Funds futures** to **short OIS** (or vice versa), what changes in the implied **ON forward shape** and risk?”

**Mechanism**

* FF futures constrain **monthly averages**.
* OIS swaps constrain **compounded accrual PV** over specific start/end windows.
  These are different functionals of the same curve.

**What changes**

* OIS-heavy build tends to localize structure around coupon windows.
* Futures-heavy build tends to distribute structure across month/IMM windows.
* Risk allocation changes: the same 1bp quote move maps into different node movements.

**Desk symptom**

* PV fits are similar, but the DV01 ladder is different. Traders call that “curve voodoo.” It’s actually “different measurement operators.”

**Checks**

* Compare (\partial x/\partial q) norms by instrument type.

---

## Q64) “If you add an extra **1W OIS** quote (or tighten its tolerance), what happens to the **1M OIS** and the **first IMM future**?”

**Mechanism**

* 1W OIS pins a very short integral of (r(t)).
* 1M OIS and IMM futures also integrate (r(t)), but over longer windows.
  If 1W is forced, the solver must adjust later segments to keep longer-window instruments fit.

**What changes**

* The curve often creates a **micro ‘step’** around week 1–2 (if allowed), or shifts post-week segments.
* IMM future fit may worsen slightly unless you have enough DOFs.

**Desk symptom**

* “Short OIS got perfect, IMM future got slightly worse.” That’s expected coupling.

---

## Q65) “If you change **IMM date handling** (e.g., include IMM nodes explicitly vs not), what happens to **STIR futures strip fit** and **FRA-like implied forwards**?”

**Mechanism**

* Many STIR/IMM futures are effectively forwards over IMM-aligned periods.
* If your node grid doesn’t align to IMM boundaries, the solver approximates those forwards by interpolating through misaligned segments.

**What changes**

* Adding IMM nodes reduces interpolation error and improves strip fit *without* inventing curvature.
* But it increases DOFs, so if the market doesn’t support them you risk overfit.

**Desk symptom**

* Strip fit improves, but you must check stability: extra nodes can make Jacobians nastier.

**Checks**

* Condition proxy of (J^\top WJ).
* Kink metrics around IMM boundaries.

---

## Q66) “If you apply (or stop applying) a **futures convexity adjustment** for IMM futures, what shifts between **futures** and **swap** instruments?”

**Mechanism**

* Futures are margined daily; forwards are not. Under stochastic rates:
$$
  \text{Futures rate} \neq \text{Forward rate}
$$
* A convexity correction changes the mapping from futures quote to curve-implied forward.

**What changes**

* With convexity “on,” the curve will attribute some level differences to convexity rather than curve shape.
* With it “off,” the curve bends to absorb the difference, and that bend bleeds into swaps and basis.

**Desk symptom**

* Persistent residual pattern by contract month if convexity is wrong.
* Swap curve hump that “mysteriously” appears when you change convexity setting.

**Checks**

* Residual heatmap by futures contract.
* Compare implied forwards from futures vs FRAs/swaplets.

---

## Q67) “If you change the **daycount** on a cash instrument (Act/360 vs Act/365F) but leave swaps unchanged, why does it propagate into the whole curve?”

**Mechanism**

* The cash instrument PV becomes systematically shifted:
$$
  1 + r\cdot \delta \quad \text{(wrong }\delta\text{ means wrong implied }r)
$$
* The solver rebalances globally to fit this new anchor, affecting all instruments sensitive to early discount factors.

**What changes**

* Short OIS and first futures contracts reprice.
* Curve may rotate slightly: front-end level changes, slope adjusts to keep longer maturities fit.

**Desk symptom**

* Looks like a macro move, but it’s a convention correction.

**Checks**

* Before/after repricing pack and a strict conventions regression suite.

---

## Q68) “If you increase weight on **2Y–5Y par swaps** in discounting calibration, what happens to the **futures strip** and the **1Y swap**?”

**Mechanism**

* Par swaps constrain integrated discount factors at specific maturities.
* Futures strip constrains average forward rates at shorter horizons.
  If swaps are overweighted, the solver will satisfy them by shifting the curve shape, potentially sacrificing short-end strip fit.

**What changes**

* Strip residuals grow (often in a systematic pattern).
* 1Y region may kink to reconcile both.

**Desk symptom**

* Long-end looks ‘clean’, short-end looks ‘off’. Then risk becomes inconsistent across buckets.

**Checks**

* Residual migration: strip → short swaps.
* Short-end kink metrics.

---

## Q69) “If you change the **fixed-leg frequency** assumption on swaps (annual vs semiannual), how does that affect the curve even if the par rate is the same?”

**Mechanism**

* Cashflow timing changes. PV equation changes:
$$
  \sum D(t_i)\alpha_i K \quad \text{(different }t_i,\alpha_i\text{)}
$$
* Same quoted par rate doesn’t mean same PV constraint if the schedule differs.

**What changes**

* Discount curve adjusts to match the new cashflow distribution.
* This shifts sensitivities and can create small but systematic residual patterns in adjacent instruments.

**Desk symptom**

* “Why did curve move from just changing a schedule parameter?” Because PV depends on payment timing.

**Checks**

* Reprice a handful of swaps across maturities vs vendor using the same schedule assumptions.

---

## Q70) “If you tighten fit on **3M FRAs / IMM futures** for the projection curve, what happens to **3M IRS** and **OIS–3M basis swaps**?”

**Mechanism**

* FRAs/futures constrain forward rates over specific short windows.
* 3M IRS constrains compounded/projection-integrated cashflows over longer horizons.
* Basis swaps tie projection and discounting together.

If you force the FRA strip fit too tightly, the solver may:

* introduce curvature in the 3M projection curve,
* and basis swaps will either misfit or force discount curve shifts.

**Desk symptom**

* Perfect FRA strip, but basis curve becomes ugly or IRS fit worsens in mid tenors.

**Checks**

* Basis residual drift and basis smoothness metrics.
* Out-of-sample: check a few IRS maturities not used in the solve.

---

## Q71) “If a **3s6s basis swap** at 2Y moves 1bp, how does that propagate into the **3M curve**, **6M curve**, and their DV01 split?”

**Mechanism**
A 3s6s basis swap PV depends on both projection curves under the same discounting:

* If you keep discounting fixed, the adjustment must be absorbed by either 3M, 6M, or an explicit basis function.

**What changes**

* With explicit basis curve: basis curve shifts locally, 3M and 6M stay relatively stable.
* Without explicit basis curve: one of 3M/6M curves deforms, spreading the impact.

**DV01 split**

* Risk migrates between “3M forwarding risk” and “6M forwarding risk” buckets depending on parameterization and regularization.

**Desk symptom**

* Traders ask: “Why did a basis move change my IRS DV01?” Because projection curves are shared across instruments.

---

## Q72) “If you change the **OIS–IBOR basis swap** weighting, what happens to **discount curve**, **projection curve**, and even **swap spread-like behavior**?”

**Mechanism**
OIS–IBOR basis is one of the strongest couplers:

* Discount curve affects PV of both legs.
* Projection curve sets IBOR coupons.

Tightening basis fit forces the system to reconcile differences by shifting whichever curve is more flexible (often projection, sometimes discount).

**What changes**

* Projection curve level/slope changes to satisfy basis.
* Discount curve may adjust slightly if allowed.
* Par swap rates implied by your curves shift subtly, affecting perceived “swap spreads” vs alternative curve sets.

**Desk symptom**

* “Basis got better, but swaps moved.” Yes. You tightened the coupler.

---

## Q73) “If you change the **stubbing rule** (short stub vs long stub) for swaps around IMM dates, what breaks first: futures strip or swaps?”

**Mechanism**
Stub choice changes coupon period lengths and payment dates, affecting PV.
Futures strip is linked to IMM boundaries. If your swap schedule shifts around IMM, the mapping between “futures-implied forwards” and swap cashflows changes.

**What changes**

* Swaps around that region misfit if your stubs are inconsistent with market conventions.
* If swaps are forced to fit, the projection curve will kink near IMM.

**Desk symptom**

* Localized residual cluster around IMM region.

**Checks**

* Validate schedule generation against a known reference for IMM-adjacent swaps.

---

## Q74) “If you reprice using **fixings** incorrectly (e.g., treat known past fixings as curve-implied), what propagates into calibration?”

**Mechanism**
Fixings are **realized history**, not model expectation.

* Past fixings determine accrued coupon amounts.
* The curve should determine only the *unknown* future part.

If you let the curve “explain” past fixings, you create artificial constraints and distort the front end.

**What changes**

* Front-end projection curve shifts to fit historical realized rates (nonsense).
* Short instruments look off because accruals are wrong.
* PnL explain becomes noisy because accrual error masquerades as curve move.

**Desk symptom**

* “Why is yesterday’s fixing affecting today’s calibration?” It shouldn’t.

**Checks**

* Hard separation in implementation: realized fixings feed cashflows, not calibration targets.

---

## Q75) “If you change **FX swap points** at 3M (same currency pair), how does it move **XCCY basis** vs **local OIS curves**?”

**Mechanism**
FX swaps imply short-dated FX forwards. Those relate to DF ratios (plus basis effects). Tight FX fit pushes reconciliation into XCCY basis or one of the discount curves.

**What changes**

* If you treat FX as very reliable: XCCY basis curve adjusts (often front-end kink).
* If you allow local OIS to move: the dependent currency discount curve can get contaminated.

**Desk symptom**

* FX looks perfect but basis curve becomes weird, or vice versa depending on weights.

---

## Q76) “If you change the **basis swap payment frequency** (quarterly vs semiannual) assumption, why does the basis curve shape change?”

**Mechanism**
Different payment dates change DF weighting in PV:
$$
PV_{\text{basis}} \sim \sum D(t_i)\alpha_i,s
$$
So the same quoted spread corresponds to a different PV constraint when cashflow timing shifts.

**What changes**

* Basis curve levels may shift slightly to compensate.
* Residual patterns appear at tenors where payment schedule differences matter.

**Desk symptom**

* Looks like a basis market move, but it’s schedule mechanics.

---

# Summary table

| Q# | Instrument change                 | Main propagation path            | Typical “desk pain” symptom                |
| -: | --------------------------------- | -------------------------------- | ------------------------------------------ |
| 61 | Cash ON/TN/SN weight ↑            | cash → short OIS/futures         | first futures contract residual shifts     |
| 62 | Meeting effective date knot shift | meeting-month futures/OIS/IMM    | DV01 bucket migration, micro misfits       |
| 63 | FF futures ↔ short OIS dominance  | short-end operator changes       | same PV, different DV01                    |
| 64 | Add/tighten 1W OIS                | 1W pins early segments → 1M/IMM  | slight strip deterioration                 |
| 65 | Add IMM nodes                     | strip fit vs stability           | better fit, risk conditioning check needed |
| 66 | Futures convexity on/off          | strip ↔ swaps ↔ basis            | residual pattern by contract month         |
| 67 | Cash daycount fix                 | front-end anchors move           | “macro move” that’s just a bug fix         |
| 68 | Swap weights ↑                    | swaps dominate strip             | strip residuals rise systematically        |
| 69 | Swap fixed-leg freq change        | cashflow timing changes PV       | curve shifts “for no reason”               |
| 70 | FRA/IMM weight ↑                  | projection bends → basis tension | perfect strip, ugly basis                  |
| 71 | 3s6s basis move                   | 3M/6M + basis risk split         | IRS DV01 shifts after basis move           |
| 72 | OIS–IBOR basis weight change      | discount/projection coupling     | swaps shift when basis tightened           |
| 73 | Stub rule change near IMM         | schedule ↔ futures mapping       | residual cluster around IMM                |
| 74 | Fixings mishandled                | accrual error → curve distortion | yesterday’s fixing “moves” today’s curve   |
| 75 | FX swap points change             | FX ↔ XCCY ↔ discount curves      | front-end basis kink                       |
| 76 | Basis pay frequency change        | DF weighting in PV               | basis curve shifts mechanically            |

If you want the next batch, I’ll go deeper into **swap vs swap (different maturities)** interactions (like “tighten 1Y IRS fit and watch 2Y basis go weird”), plus **IMM roll and stub corner cases** that show up only on real desks.

---
I’ll continue with another batch of instrument-specific “change X → Y moves” questions (Q77 onward), covering swap-tenor interactions, basis/IMM roll corner cases, and multi-currency couplings, each answered like an interview response.


Alright, next batch. **Q77–Q92**, instrument-specific, “touch one quote and the rest of the curve system starts gossiping” style. Each question answered in depth.

---

## Q77) “If you tighten fit on the **1Y OIS swap** (discount curve), what happens to **2Y–5Y OIS**, and why can the **1Y–2Y forward segment** become unstable?”

### Mechanism

A 1Y OIS par swap is a strong constraint on the **integral** of overnight forwards over ([0,1Y]). If you tighten its tolerance a lot, the solver must make sure the area under the curve matches exactly.

* If your front-end has meeting/turn nodes, the solver may reallocate shape within ([0,1Y]) without moving the 1Y DF much.
* But if the curve lacks enough degrees of freedom (or regularization is too strong), the solver pushes curvature into the interval **right after 1Y** to reconcile longer OIS instruments.

### Downstream effects

* **2Y–5Y OIS**: slight residual drift or shape adjustments to preserve par.
* **1Y–2Y forward**: can kink because that’s where the solver finds “free curvature” to satisfy multiple constraints.

### Desk symptom

* 1Y OIS fits perfectly.
* Forward curve shows a kink around 1Y.
* DV01 around 1Y bucket “moves” even if market barely moved.

### Fix

* Add a supporting knot around 18M/2Y or adjust regularization locally so the solve doesn’t dump curvature at 1Y boundary.

---

## Q78) “If the **2Y IRS** quote moves but **1Y IRS** doesn’t, why does that often show up as a reshaping of the **1Y–3Y projection forwards** rather than a local 2Y bump?”

### Mechanism

IRS constraints are not point constraints. A 2Y IRS par rate is a weighted sum of forwards and discount factors over multiple coupon dates. The sensitivity is spread over the curve.
$$
\text{Par}(2Y) \approx \frac{\sum D(t_i)\alpha_i F(t_{i-1},t_i)}{\sum D(t_i)\alpha_i}
$$
So the “right” adjustment is typically a **rotation/hump** over the interval where the annuity weights are largest, not a delta at 2Y.

### Downstream effects

* Projection forwards reshape in 1Y–3Y zone.
* Basis vs OIS coupling instruments respond too (if used), because discounting is shared.

### Desk symptom

* Traders expect “2Y moved so 2Y forward moved.”
* You show them the annuity weights and they reluctantly stop blaming the curve engine.

---

## Q79) “If you force a perfect fit on the **ED/SOFR IMM strip** (projection curve), what happens to **swap spreads** and **OIS–IRS consistency**?”

### Mechanism

IMM futures are short-window forward constraints; swaps are long-window par constraints. If you force the strip to fit tightly, you’re effectively imposing a very specific forward path at the front/mid curve.

If the swap market disagrees slightly (because of liquidity, convexity, stub conventions, or microstructure), the system reconciles by adjusting:

* discount curve a bit,
* or basis curves,
* or long-window forwards in a way that can change implied par swap levels.

### Downstream effects

* **IRS par fits** degrade or require compensating curvature.
* OIS–IRS basis residuals can grow.
* The desk’s observed “swap spread-ish” relationships can shift in your internal curve set.

### Symptom

* Strip looks immaculate.
* Swaps become slightly off or basis becomes uglier.
* DV01 becomes strip-dominated and less stable.

### Safe policy

Don’t treat the strip as a religion. Use bid/ask-aware tolerances and validate against swaps and basis.

---

## Q80) “If you change **roll conventions** (Modified Following vs Following) for short instruments, why can it distort the **turn effect** and the **meeting-month fit**?”

### Mechanism

Roll conventions change the actual accrual dates and payment dates, which changes:

* which days belong to the window,
* how many days are pre/post meeting,
* whether you land on month-end/quarter-end dates.

A one-day shift can change the weighting in a month-average constraint or turn window.

### Downstream effects

* Turn effect can be over- or under-expressed.
* Meeting-month futures fit can change even if quotes didn’t.

### Desk symptom

“Nothing moved, but the curve did.”
Yes. Dates moved.

---

## Q81) “If you add a new instrument: **a 9M OIS** (between 6M and 1Y), why might it change the fit of **the first IMM future** even though they’re different products?”

### Mechanism

Both are constraints on the same underlying ON forward curve. Adding 9M OIS inserts an additional PV constraint on the integral of forwards up to 9M. That can redistribute the shape required to fit monthly/quarterly futures windows.

### Downstream effects

* IMM future fit can shift if the previously implied shape was using that region as a “buffer.”
* The curve may now localize shape differently, affecting window-averaged futures.

### Symptom

* You add a “helpful” OIS quote and the IMM strip gets slightly worse.
  This usually means your curve was previously over-relying on interpolation.

---

## Q82) “If you switch from **single-curve projection for all tenors** to explicit **1M/3M/6M projection curves**, what happens to **basis swap DV01** and why does it often ‘appear’ out of nowhere?”

### Mechanism

In a single projection curve world, basis is implicitly collapsed into one forwarding object. When you split into multiple projection curves, basis becomes an explicit degree of freedom and risk becomes explicit.

### Downstream effects

* Basis swap DV01 becomes meaningful and often larger.
* Risk decomposes into separate blocks: 3M vs 6M forwarding risk, plus basis risk.
* Hedging becomes clearer but more complex.

### Desk symptom

“Why did basis risk increase?”
It didn’t. You just stopped hiding it inside a single curve.

---

## Q83) “If a **2Y 3s6s basis** quote is stale but you overweight it, what happens to **6M swaps** and **6M projection forwards**?”

### Mechanism

3s6s basis ties 3M and 6M projection curves. If you force-fit a stale basis quote, the solver will deform the 6M curve (or 3M curve) to satisfy it.

### Downstream effects

* 6M forward curve bends unnaturally around maturities where that basis quote has sensitivity.
* 6M swap par rates may drift vs market.

### Symptom

Basis fits “great” but 6M instruments start looking wrong out-of-sample.

### Fix

Robust weighting and “staleness-aware” tolerances. You do not let a stale basis quote steer the curve hard.

---

## Q84) “If you change **swap coupon frequency** for projection instruments (e.g., 3M float leg accrues quarterly but you mis-handle stub), how does it leak into the **FRA strip fit**?”

### Mechanism

FRAs pin specific forward intervals; swaps depend on a weighted average of those intervals (via their coupon schedule). If the coupon schedule generation is wrong, the swap’s “implied” forwards do not line up with FRA-defined periods.

The solver will then try to reconcile by bending the projection curve, which can distort FRA fit too.

### Symptom

* FRA strip residual pattern emerges near stub region.
* Swaps around that maturity become inconsistent.

### Desk truth

This is almost always a scheduling bug, not a market phenomenon.

---

## Q85) “If you change the **start date convention** for swaps (spot vs IMM start), how does it affect the **relationship between futures and swaps**?”

### Mechanism

IMM-start swaps align cashflows to the IMM grid, making them naturally consistent with IMM futures. Spot-start swaps don’t align perfectly.

If you change start convention in the curve engine (or misapply it), you change which forward intervals swaps are actually constraining.

### Downstream effects

* Futures-implied forwards may stop lining up with swap-implied forwards.
* You see systematic basis-like residuals between strip and swaps.

### Symptom

Strip and swaps disagree “for no reason.”
The reason is your start convention drifted.

---

## Q86) “If you introduce a **turn adjustment** (e.g., year-end turn node), how does that affect **swap par rates** out to 5Y even though the turn is just a few days?”

### Mechanism

Even a few days of elevated overnight rates change discount factors slightly. Those DFs enter every annuity:
$$
A = \sum D(t_i)\alpha_i
$$
A small change in (D(t)) early changes the annuity for many maturities, and therefore the par rate consistency conditions.

### Downstream effects

* Small but systematic changes in par rates (most noticeable in shorter maturities where the early DF weight is larger).
* Risk distribution changes, especially front-end DV01.

### Symptom

Traders: “How is a year-end turn affecting 2Y swaps?”
Answer: because discounting weights everything.

---

## Q87) “If you change the **OIS compounding convention** (daily compounding vs simplified approximation), what shows up first: **OIS swaps**, **cash instruments**, or **FF futures**?”

### Mechanism

* OIS swaps depend heavily on accurate compounding across business days.
* Cash instruments are simpler (often simple interest).
* FF futures are month averages of overnight effective rates (not compounded in the same way as OIS coupons).

### What breaks first

* **OIS swaps**: immediate mispricing, especially in short maturities.
* Cash: less sensitive unless you also mis-handle daycount.
* FF futures: affected indirectly via the ON forward path that the solver re-optimizes.

### Symptom

OIS residuals blow up; strip might still look “okay” but becomes inconsistent.

---

## Q88) “If you calibrate the discount curve to **OIS swaps only** and exclude **cash** instruments, what propagates into **short-end risk**?”

### Mechanism

Without cash anchors, the solver uses OIS instruments (often 1W/1M/3M) to infer the very front end. That can work, but it increases sensitivity to:

* schedule conventions,
* local quote noise,
* and meeting/turn node placement.

### Downstream effects

* Short-end DV01 becomes noisier.
* The first few days can drift to satisfy OIS PV constraints.

### Symptom

Intraday, the curve flips more; day-to-day DV01 buckets migrate.

### Fix

Include cash anchors or impose a strong prior for the very front end.

---

## Q89) “If you change the **tolerance** on a single mid-curve instrument (say 5Y swap), why can that move the **entire curve level** rather than just local shape?”

### Mechanism

A tolerance change is effectively changing how “important” that constraint is relative to others. The optimizer can choose to satisfy it by:

* a local bump (costly in curvature penalties), or
* a gentle global level shift (cheap if regularization prefers smoothness).

So a small change in tolerance can cause the solution to pick a different “mode”: level shift vs local bend.

### Symptom

“Why did loosening 5Y tolerance move 2Y forwards?”
Because the global best-fit rebalanced.

---

## Q90) “If you tighten the **basis swap fit** in the front end (e.g., 1Y XCCY basis), what happens to **FX forward points** and **local OIS curves**?”

### Mechanism

Front-end XCCY basis PV is extremely sensitive to:

* short-dated DF ratios,
* FX forwards,
* and the basis spread.

If you force basis fit, the solver can reconcile by:

* bending basis curve,
* adjusting FX forward calibration,
* or slightly distorting one local discount curve (often the dependent one).

### Symptom

Either FX forwards start misfitting, or local OIS curve residuals degrade, or basis curve kinks. Something has to give.

### Desk policy

Tenor-dependent weighting: don’t force all three to be perfect if markets are inconsistent.

---

## Q91) “If you change the **FX spot lag** assumption (T+2 vs T+1 style), why does it corrupt the entire **short-end cross-currency** calibration?”

### Mechanism

FX forwards and FX swaps depend on the spot date. If spot date shifts, all forward maturities effectively shift. That changes the implied DF ratio constraints.

### Downstream effects

* FX forward repricing errors become systematic.
* XCCY basis tries to compensate.
* Dependent currency curves get polluted.

### Symptom

Everything cross-currency in the front end looks “consistently wrong” by a small amount.

### Fix

Spot lag is a hard convention. You don’t negotiate with it.

---

## Q92) “If you change **which instruments are used as targets** (e.g., switch broker runs) while keeping methodology fixed, why does the curve sometimes ‘rotate’ rather than shift?”

### Mechanism

Different broker runs have different microstructure: some quote sets are systematically higher in front end, others in belly, etc. The optimizer finds the best-fit shape consistent with smoothness constraints, often leading to a **rotation** (front-end up, mid down, or vice versa) rather than a parallel shift.

### Symptom

The curve’s level appears stable at one maturity but different at another, and DV01 shifts across buckets.

### Correct posture

Treat “data source change” as a model change: produce before/after packs, residual comparisons, and risk impact reports.

---

# Summary table

| Q# | Change                      | Propagates into           | First visible symptom            |
| -: | --------------------------- | ------------------------- | -------------------------------- |
| 77 | tighten 1Y OIS fit          | 2Y–5Y OIS, 1Y–2Y fwd      | kink around 1Y                   |
| 78 | 2Y IRS quote move           | 1Y–3Y forwards            | hump/rotation not local bump     |
| 79 | force IMM strip fit         | swaps + OIS–IRS basis     | strip perfect, swaps/basis worse |
| 80 | roll convention change      | turns + meeting-month fit | “market didn’t move” curve did   |
| 81 | add 9M OIS                  | IMM future fit            | IMM residual shifts              |
| 82 | split 1M/3M/6M curves       | basis DV01                | basis risk becomes explicit      |
| 83 | overweight stale 3s6s basis | 6M forwards + swaps       | 6M instruments drift             |
| 84 | stub bug on float leg       | FRA strip fit             | residual cluster near stub       |
| 85 | spot vs IMM start change    | futures-swap consistency  | systematic strip vs swap gap     |
| 86 | add turn node               | par swaps out to 5Y       | small systematic par shifts      |
| 87 | OIS compounding change      | OIS swaps first           | OIS residual blow-up             |
| 88 | exclude cash anchors        | short-end risk            | DV01 noisier                     |
| 89 | change one tolerance        | global curve mode         | level shift/rotation             |
| 90 | tighten front XCCY basis    | FX fwd + local OIS        | something kinks/misfits          |
| 91 | FX spot lag change          | all short XCCY            | systematic front-end wrong       |
| 92 | change quote source         | curve shape rotates       | DV01 bucket migration            |

If you want the next batch, I can go into: **swap switches/flies constraints**, **hedge impact when you change curve node placement**, and **how a change in one currency’s meeting calendar pollutes another currency through USD collateral**.

---
Continuing. **Q93–Q108**. Same format: instrument-specific, “change one thing → something else moves” questions, with full interview-style answers.

---

## Q93) “If you update **today’s overnight fixing** (or correct a bad fixing), what changes: cashflows only, or the curve too?”

### What should happen (correct behavior)

* **Fixings should affect cashflows**, not the forward curve.
* Any coupon portion that is already fixed uses realized fixings. The curve only supplies **unknown future** rates.

### What *actually* breaks if the system is sloppy

1. **Curve gets contaminated**
   If the engine mistakenly treats fixings as curve targets (or re-infers the front-end from fixings), it forces the ON forward curve to “explain” yesterday. That’s nonsense.

2. **Short OIS and front-end futures get distorted**
   Because the first segment of the ON curve is highly sensitive, a single fixing error can masquerade as a “front-end market move.”

3. **PnL explain gets wrecked**

* Accrual PnL should move with fixing correction.
* If curve also shifts, you get fake “curve PnL” and wrong risk attribution.

### How I prove it in an interview

* Show separation: fixings feed coupon accrual; curve solve uses tradable instruments only.
* Run a test: change a past fixing and confirm curve parameters don’t move (within numerical noise).

---

## Q94) “If you add a **meeting-date step constraint** (explicit jump) instead of just a knot, what happens to fitting and risk?”

### What you changed

Instead of allowing the curve to freely shape around the meeting boundary, you impose something like:
$$
f(t) = f_{pre}\ \text{for } t<T_m,\quad f(t)=f_{post}\ \text{for } t\ge T_m
$$
or a constrained jump size prior.

### Downstream effects

1. **Better interpretability**

* You get a clean pre/post decomposition.
* Meeting-month futures fit becomes more stable.

2. **Less flexibility elsewhere**

* If the market implies a more gradual repricing (or multiple events/uncertainty), the hard step can force misfit into:

  * adjacent futures months,
  * short OIS spanning the period,
  * or turns if those aren’t modeled well.

3. **Risk concentrates**

* A hard step makes sensitivities cluster around the meeting node. Good for clarity, bad if it causes bucket DV01 to become too “spiky.”

### Desk symptom

Traders love the clean narrative until DV01 collapses into one date bucket and hedging gets jumpy.

---

## Q95) “If you switch the discount curve’s short end from **SOFR OIS** to a proxy built from **SOFR futures**, what changes in swap pricing and hedging?”

### Mechanism

* OIS instruments constrain discounting through compounded overnight PVs.
* Futures constrain averages of expected overnight (plus convexity effects).

### Downstream effects

1. **Discount factors shift subtly**
   Even if the overall level matches, the *shape* changes because the measurement operator differs.

2. **Swap par rates change**
   Swaps discount all fixed and float cashflows with (D(t)). So a discount curve built from different inputs changes:

* par swap levels
* annuities
* DV01 ladders

3. **Hedge set changes**

* If discount curve is futures-driven, the system tends to allocate risk into futures buckets.
* If it’s OIS-driven, risk aligns more with OIS instruments.

### Desk symptom

“PV is close but hedge ratios differ.”
That’s exactly what you’d expect: same PV doesn’t imply same (\partial PV/\partial q).

---

## Q96) “If you tighten the fit on **2Y–3Y IMM futures** (belly of the strip), why does it sometimes distort **5Y swap** fit?”

### Mechanism

Even though IMM is “shorter,” those futures constrain forward expectations across a region that overlaps the swap annuity’s most sensitive period.

* A 5Y swap’s par rate is a weighted average of forwards out to 5Y.
* If you force the 2–3Y forwards to a specific shape, the optimizer may have to compensate by reshaping 3–5Y forwards or discounting.

### Downstream effects

* 5Y par fit shifts or needs curvature elsewhere.
* Basis instruments (if present) may absorb some inconsistency.

### Desk symptom

The 5Y point gets “pulled” by strip constraints. Traders will accuse you of witchcraft. You respond with annuity weights and carry on.

---

## Q97) “If you change **the roll date handling** for IMM futures (e.g., last trade date, effective start), what breaks in the FRA vs futures alignment?”

### Mechanism

IMM futures price a forward over a contract-defined interval. If you shift effective dates:

* the interval changes,
* the mapping from futures price → implied forward rate changes.

### Downstream effects

1. **Implied FRAs derived from futures stop matching actual FRA quotes**
2. **Projection curve gets forced to kink around the roll**
3. **Strip residuals become patterned**
   You see repeated residuals on contracts around the roll boundary.

### Desk symptom

You get a “comb pattern” in residuals across adjacent contracts. That’s almost always date handling, not market.

---

## Q98) “If you allow **negative discount rates** but still enforce **DF monotonicity**, what contradiction can appear?”

### Mechanism

DF monotonicity (D(t_{k+1})\le D(t_k)) assumes non-negative instantaneous discount rates on the interval.

But with negative rates:

* you can have (D(t)) increasing over time.

### Downstream effects

* Hard monotonic DF constraints become incompatible with a negative-rate regime.
* The solver compensates by contorting other curves (projection/basis) or failing convergence.

### Desk symptom

“Solver can’t converge” or “constraints binding everywhere.”
It’s not a numerical issue. It’s a conceptual constraint mismatch.

### Proper approach

* Use regime-aware constraints:

  * keep (D(t)>0) always
  * allow mild DF increases if negative rates are plausible
  * enforce forward bounds instead of strict DF monotonicity.

---

## Q99) “If you add a **swap switch** constraint (e.g., 5Y vs 10Y spread) to the live curve, what happens to the risk curve and why can PnL explain diverge?”

### Mechanism

A swap switch is a constraint on *relative levels*:
$$
S(10Y)-S(5Y)
$$
Adding it to a live curve introduces an extra “shape target” not necessarily present in the risk curve.

### Downstream effects

1. **Live curve bends to match relative-value inputs**

* It may shift curvature in the belly/long end to satisfy the switch.

2. **Risk curve may remain more “pure”**

* Built mostly from primary par instruments with stronger regularization.
* So it won’t reflect the same relative-value constraints.

3. **PnL explain divergence**

* Trades priced/managed off live curve show different sensitivities than risk curve-based reporting.
* You see “curve PnL” differences purely from methodology divergence.

### Desk symptom

Trader: “Why did I make money on live but risk says flat?”
Because you’re comparing two different curve objectives.

---

## Q100) “If you add a **swap fly** constraint (e.g., 2s5s10s), why can it increase the chance of ‘kinks’ and unstable sensitivities?”

### Mechanism

A fly constraint targets curvature directly:
$$
S(5Y) - \frac{1}{2}(S(2Y)+S(10Y))
$$
If you force the fly to match a quote tightly, you’re imposing a second-derivative-like condition.

### Downstream effects

* The optimizer may introduce localized curvature (kinks) to hit the fly while still fitting par swaps.
* This can blow up (\partial x/\partial q) locally.

### Desk symptom

* Fly fits perfectly.
* Instantaneous forwards show a wiggle.
* Bucket DV01 becomes unstable around the fly maturities.

### Safe use

* Treat flies as **soft constraints** (moderate weight) and always monitor kink metrics and bump/rebuild stability.

---

## Q101) “If you switch the projection curve input set from **FRAs** to **short swaps**, what changes in the forward curve’s locality and why?”

### Mechanism

* FRAs constrain specific forward intervals (localized).
* Swaps constrain weighted averages of many intervals (global-ish).

### Downstream effects

* FRA-driven curve tends to have strong local anchoring (good locality, potentially noisy).
* Swap-driven curve spreads adjustments across maturities (smoother, but less directly tied to short-window tradables).
* Hedging instruments shift: FRA risk vs swap risk.

### Desk symptom

Same PV quality but different bucket risk distribution. Again: operators differ.

---

## Q102) “If you change the **spot lag** on swaps (T+2 vs T+1 style in some markets), what does it do to the first 1–3 coupons and why does that matter?”

### Mechanism

Spot lag changes the effective start date. That shifts:

* first accrual start,
* stub length,
* coupon schedule alignment.

### Downstream effects

* Short swap PV constraints change materially for 1Y-ish instruments.
* This propagates into both discount and projection curves because the first few coupons carry significant PV weight.

### Desk symptom

* Short-end par swaps shift even if market rates didn’t.
* Residual clusters around 6M–2Y appear.

---

## Q103) “If you change the **basis swap spread sign convention** or leg designation (who pays spread), how does that silently corrupt calibration?”

### Mechanism

Basis swaps are quoted with desk conventions (add spread on one leg). If you flip sign or attach spread to wrong leg:

* your PV equation is wrong, but it still “solves” because the optimizer will contort curves to fit.

### Downstream effects

* Basis curve becomes inverted/weird.
* Projection curves deform to compensate.
* Cross-currency basis can go completely off if this is in XCCY.

### Desk symptom

Everything “fits” but the basis curve looks like a lie detector result. This is a classic “it calibrates therefore it’s correct” trap.

### Fix

Hard-coded convention unit tests. This is not optional if you like sleep.

---

## Q104) “If you change the collateral rate from **SONIA** to **€STR** for GBP/EUR-linked books (multi-currency), what propagates into FX forwards and XCCY basis?”

### Mechanism

Collateral choice changes discounting. Discount curves feed DF ratios, which feed FX forwards consistency and XCCY PVs.

### Downstream effects

* Cross-currency curve set changes: the implied basis needed to reconcile FX forwards and XCCY swaps changes.
* FX forward repricing can shift at short maturities.
* XCCY basis curve may rotate or kink if weights are tight.

### Desk symptom

“Why did changing collateral change my FX forwards model fit?”
Because the model is enforcing cross-asset consistency under the chosen collateral measure.

---

## Q105) “If you update **central bank meeting calendar** (e.g., unscheduled meeting, or corrected date), why can it impact months far away via the strip?”

### Mechanism

Meeting nodes affect segmentation of the curve, and futures constraints are averages over months/quarters. Changing segmentation changes how those averages are represented.

### Downstream effects

* Not only the meeting month, but adjacent months can shift because the solver rebalances pre/post rates.
* IMM quarter averages can shift too, propagating into several contracts.

### Desk symptom

You fix a meeting date and 2–3 nearby futures contracts change residuals. That’s normal. The only question is whether it’s localized and economically sensible.

---

## Q106) “If you change the **bootstrapping order** (even with the same instruments), how can that change risk in a sequential system?”

### Mechanism

Sequential bootstrap is path-dependent:

* earlier instruments determine earlier nodes,
* later instruments fit on top of that.

If you change the order, you change which errors get “locked in” early.

### Downstream effects

* PV fit may remain similar.
* Risk attribution changes because node exposure reflects the chosen build path.
* You can create “ghost dependencies” where one quote dominates more than intended.

### Desk symptom

“Same inputs, different DV01 ladder.”
That’s why global solves exist.

---

## Q107) “If you add **a single new basis swap tenor** (say 7Y 3s6s), why can it change the **entire basis curve level**, not just locally?”

### Mechanism

Basis curve is often weakly identified in sparse markets. Adding a new tenor can:

* shift the best-fit level of the whole basis curve (a ‘mode’ change),
* especially if regularization penalizes curvature strongly (so the cheapest change is a level shift).

### Downstream effects

* 3M and 6M curve relationship adjusts across many maturities.
* Swaps referencing those curves shift slightly.
* DV01 split between 3M and 6M buckets changes.

### Desk symptom

One new quote arrives and suddenly the whole basis curve “re-centers.” That’s a sign the curve was under-anchored.

---

## Q108) “If you tighten the tolerance on **short-dated XCCY basis** but keep **local curves** fixed-ish, where does the inconsistency end up?”

### Mechanism

XCCY basis PV depends on:

* both discount curves,
* FX forwards,
* basis spread.

If you force XCCY fit and freeze local curves, the remaining knobs are:

* FX forwards calibration,
* basis curve shape (especially front end).

### Downstream effects

* FX forward points fit may degrade (systematic errors).
* Basis curve kinks near front end.
* If neither is allowed, solver fails or constraints thrash.

### Desk symptom

You can make one thing perfect, but only by making something else worse. Anyone claiming otherwise is selling you a bridge.

---

# Summary tables

## Q93–Q108: what you tweak vs what it hits

|  Q# | You change…                  | Propagates into…                     | Classic symptom               |
| --: | ---------------------------- | ------------------------------------ | ----------------------------- |
|  93 | fixings update               | accruals vs curve contamination      | fake “curve move” from fixing |
|  94 | hard meeting step            | futures/OIS fit + risk concentration | DV01 spikes at meeting        |
|  95 | OIS → futures proxy discount | swap PV/hedges                       | PV close, hedges differ       |
|  96 | tighten belly IMM strip      | 5Y swap fit                          | 5Y pulled by strip            |
|  97 | IMM roll date handling       | FRA vs futures alignment             | comb residual pattern         |
|  98 | DF monotone with neg rates   | solver / other curves                | constraints bind everywhere   |
|  99 | add swap switch              | live vs risk divergence              | PnL explain mismatch          |
| 100 | add swap fly                 | kink risk + stability                | fly perfect, DV01 unstable    |
| 101 | FRAs → short swaps           | locality/hedge set                   | risk distribution shifts      |
| 102 | swap spot lag                | early coupons                        | residual cluster 6M–2Y        |
| 103 | basis sign/leg bug           | everything basis touches             | “fits” but basis curve insane |
| 104 | collateral SONIA↔€STR        | FX/XCCY consistency                  | FX fit shifts unexpectedly    |
| 105 | meeting calendar update      | adjacent strip months                | multiple contracts shift      |
| 106 | bootstrap order              | risk mapping                         | same PV, diff DV01            |
| 107 | add new basis tenor          | basis curve level                    | global re-centering           |
| 108 | tighten XCCY basis tol       | FX forwards / basis kinks            | something else must give      |

## What to monitor whenever you touch any of this

| Monitor                                      | Why it catches propagation early         |
| -------------------------------------------- | ---------------------------------------- |
| Residual migration by instrument type/tenor  | shows who “paid” for the change          |
| Active constraint map                        | explains non-smooth risk and snaps       |
| Kink metrics on instantaneous forwards       | detects solver-invented curvature        |
| Bump/rebuild response norms                  | detects unstable (\partial x/\partial q) |
| Cross-currency repricing set (FX fwd + XCCY) | catches anchor poisoning fast            |
| Fixings separation test                      | prevents “curve explains yesterday” bugs |

If you want the next batch, I can go into the really annoying stuff: **coupon day adjustments around holidays**, **stub-at-IMM edge cases**, and **how one currency’s front-end meeting segmentation leaks into another via USD collateral** (aka “why everything is USD’s fault”).

---
I’ll extend the list with the real desk “annoying stuff”: holiday/weekend coupon adjustments, stub-at-IMM edge cases, turns vs meeting overlaps, end-of-month quirks, and how USD-collateral front-end segmentation leaks into other currencies. Continuing numbering from Q108.


Fine. Here’s the **annoying stuff**. The stuff that breaks silently, produces “why is this off by 0.2bp,” and consumes your evening. Continuing numbering from **Q108**.

---

## Q109) “If a holiday calendar update shifts one business day in the front end, why can that move both **FF month averages** and **OIS PVs** in different directions?”

### Mechanism

* **FF futures** constrain a **calendar-month average** of overnight effective rates.
* **OIS coupons** accrue on business-day schedules with business-day adjustments and compounding rules.

If a calendar changes (new holiday, corrected holiday), it can:

* change the number of accrual days in the month window for FF averaging,
* and separately change OIS coupon accrual/payment dates (especially if roll conventions push dates).

So your two “measurement operators” shift differently.

### Desk symptom

* Meeting-month futures residual shifts one way,
* short OIS residual shifts another way,
* and it *looks* like a curve shape issue, but it’s just the calendar.

### What I do

* Run a “calendar diff” test: list changed dates and their impacts on windows.
* Reprice a micro set: 1W OIS + meeting-month future + cash ON/TN/SN.

---

## Q110) “How does **Modified Following vs Following** create phantom ‘turn’ effects at month-end?”

### Mechanism

At month-end, Modified Following can roll **back** into the prior month instead of forward into next month if rolling forward crosses into next month. That changes:

* accrual end dates,
* payment dates,
* whether cashflows land in the month-end funding stress zone.

### Propagation

* OIS par conditions shift around month-end.
* Turn adjustments (if modeled) can be double-counted or missed.
* The curve may invent a micro kink to reconcile.

### Desk symptom

You see a “turn spike” appear mid-month or disappear at actual month-end. That’s almost always a roll rule mismatch.

---

## Q111) “If you mis-handle **end-of-month (EOM) rule** for swap schedules, why does it break **swap vs swap** consistency across tenors?”

### Mechanism

EOM rule decides whether a schedule sticks to month-end dates. A 5Y swap and a 10Y swap can end up with different coupon alignment if EOM is inconsistently applied.

Even with the same par quote, the cashflow grid differs:
$$
PV_{fixed}=\sum D(t_i)\alpha_i K
$$
Different (t_i) and (\alpha_i) means different PV constraint.

### Desk symptom

* 5Y fits, 10Y fits, but **5s10s switch** repricing is off.
* DV01 distribution differs vs vendor curve.

### Fix

Schedule generation unit tests by tenor and EOM flag, verified against a reference system.

---

## Q112) “What’s the ‘stub-at-IMM’ edge case, and why can it poison your **IMM futures ↔ swap** consistency?”

### Mechanism

A swap starting spot but near IMM dates can have a first coupon that becomes a **short/long stub** that overlaps an IMM boundary. If your system assumes clean IMM-aligned quarters but the swap has a stub, the mapping from IMM futures forwards to swap coupon forwards breaks.

### Propagation

* IMM strip suggests forwards over IMM-to-IMM.
* Swap coupon needs forward over stub-to-date.
* Engine tries to reconcile by bending projection curve around IMM.

### Desk symptom

Residual cluster around the IMM region, often only for specific start dates (the ones traders actually trade).

### Fix

Explicitly handle stub coupons as their own accrual windows; don’t approximate them as standard IMM intervals.

---

## Q113) “If you change the rule for **short stub vs long stub**, why can it flip the sign of misfit for **the same swap tenor**?”

### Mechanism

Short stub vs long stub changes where you put the irregular period:

* Short stub: irregular coupon near start
* Long stub: irregular coupon near end

That changes the timing weights of discounting and forwards and can change whether the solver wants to raise or lower forwards in that region.

### Desk symptom

Same instrument quote, different stub rule, residual flips sign and the curve ‘rotates’ to compensate.

### Fix

You must match market convention for that instrument set. Stub rule is not a preference, it’s part of the contract spec.

---

## Q114) “How do overlapping **turn + meeting** effects create identification problems, and what happens when you ‘fix’ one but not the other?”

### Mechanism

If a central bank meeting is near quarter-end/year-end, market pricing contains both:

* policy expectation jump
* funding/turn premium

If you model only meeting nodes but not turns (or vice versa), the solver misattributes one effect to the other.

### Propagation

* Turn premium gets smeared into meeting jump, or
* meeting jump gets smeared into turn spike.

This affects:

* meeting-month futures
* short OIS
* and later cross-currency because short-end DFs move.

### Desk symptom

Your curve shows a “meeting jump” that is too big and a “turn” that is missing (or reversed). Traders call it “nonsense” and they’re right.

### Fix

Include both structures explicitly and apply localized regularization so neither gets flattened into the other.

---

## Q115) “Why do year-end holidays and ‘broken weeks’ cause strange behavior in **weekly OIS** and **cash instruments**, even when monthly products look fine?”

### Mechanism

Weekly instruments are sensitive to:

* exact business day count
* weekend/holiday placement
* compounding over non-business days rules

Monthly averages (like FF futures) can look fine because errors partially average out.

### Propagation

* 1W OIS PV changes materially with a single holiday shift.
* Cash ON/TN/SN schedules change.
* The curve then rebalances and can shift post-week segments to keep monthly fits.

### Desk symptom

Weekly OIS outliers around holidays while the rest of the curve “looks normal.”

### Fix

Treat holiday weeks as special: ensure correct rate application on weekends/holidays (per index rules) and ensure schedules are generated exactly.

---

## Q116) “What’s the classic **business day adjustment** bug that makes PV wrong but only for a few tenors?”

### Mechanism

Mixing:

* adjusted dates for accrual start/end
* but unadjusted dates for payment
  or inconsistent roll adjustments between legs.

This often only hits tenors where coupon dates land near holidays (e.g., 18M, 2Y around a specific holiday pattern).

### Desk symptom

A few tenors show persistent misfit, everything else ok. People waste time tuning weights. It’s a date bug.

### Fix

Schedule consistency checks: accrual start/end, fixing dates, payment dates all generated from the same rule set and validated.

---

## Q117) “How can **daycount** differences across instruments create apparent basis where none exists?”

### Mechanism

If you mis-apply daycount (Act/360 vs Act/365F vs 30/360), the implied rate from the same DF change differs:
$$
1+r\cdot \delta
$$
Wrong (\delta) means wrong (r).

### Propagation

* Cash instruments imply different levels.
* Short swaps and FRAs disagree.
* Solver introduces basis-like shape or curvature to reconcile.

### Desk symptom

You “discover” basis in a market that shouldn’t have it. That’s a bug, not alpha.

---

## Q118) “Why can a one-day shift in **spot date** create a visible kink in the front-end **XCCY basis** curve?”

### Mechanism

FX swaps/forwards are anchored at spot. If spot date shifts, every short-dated FX forward maturity shifts. That changes the DF ratio constraint at those tenors.

XCCY basis PV is extremely sensitive to those DF ratios at short maturities, so the basis curve bends to compensate.

### Desk symptom

Front-end basis kink appears around 1W–3M “out of nowhere.” It’s spot date convention drift.

---

## Q119) “How does USD-collateralization cause **USD meeting segmentation** to leak into other currencies’ discount curves?”

### Mechanism (the actual leakage path)

If a EUR trade is discounted under USD CSA, the EUR discounting curve under USD collateral is effectively anchored by relationships involving:

* USD discount curve
* FX forwards (EURUSD)
* XCCY basis instruments

If USD meeting segmentation changes USD short-end DFs, then DF ratios shift, and the implied EUR|USD-collateral discounting curve must adjust to preserve consistency.

### Desk symptom

You “only changed USD meeting nodes” and suddenly EUR|USD-collateral discount curve shifts at the front end and EURUSD basis PVs change.

### Proper interpretation

That’s not a bug. That’s cross-currency collateral economics being correctly enforced.

---

## Q120) “What happens if you forget to include **turn nodes** in USD, but you build EUR under USD CSA?”

### Mechanism

USD turn premium is real. If USD curve misses it, USD DFs are wrong at those dates. Under USD CSA, every currency curve linked through USD collateral inherits that mis-shape through FX forward and XCCY constraints.

### Propagation

* EUR|USD-collateral curve develops compensating kinks.
* XCCY basis curve tries to absorb mismatch.
* FX forwards repricing errors cluster around turn dates.

### Desk symptom

Cross-currency front-end looks “dirty” around quarter-end even if local curves look okay.

---

## Q121) “How do **coupon date clustering** and annuity weights amplify tiny schedule bugs into visible par-rate shifts?”

### Mechanism

Par swap constraints are weighted by annuity:
$$
A=\sum D(t_i)\alpha_i
$$
If schedule bugs move a few payment dates into a region with different discounting (turn, holiday adjustment), the annuity changes and par condition shifts, even if the DF curve itself didn’t change much.

### Desk symptom

A tiny scheduling difference produces a consistent 0.1–0.3bp shift in some maturities and not others.

### Fix

Compare coupon date lists directly against reference system output for representative maturities.

---

## Q122) “Why does changing **weekend rate application** (whether Friday fixing applies through weekend) matter in OIS compounding?”

### Mechanism

Overnight indices typically apply a fixing across multiple days over weekends/holidays (per index rules). In OIS compounding, those multi-day accrual factors matter:
$$
\prod (1+r_d \Delta_d)
$$
If you treat weekend days incorrectly (e.g., using separate fixings or ignoring multi-day accrual), you misprice OIS coupons.

### Desk symptom

Short OIS misfit and odd behavior around long weekends.

### Fix

Hard-code correct “rate effective over non-business days” logic consistent with the index.

---

## Q123) “What’s the ‘month-end + IMM’ double-whammy and how does it show up in futures vs swaps?”

### Mechanism

Sometimes IMM dates land near month-end, and schedules/roll rules decide which side of month-end you land on. You get:

* IMM boundary effects (strip)
* month-end funding/turn effects (turn)
* schedule adjustments (roll)

If you don’t model all three properly, the solver smears one into the other.

### Desk symptom

* Futures strip around that IMM looks off.
* Swaps around that region show an unexplained kink.
* Basis swaps get ugly because they’re the couplers.

---

## Q124) “Why do **‘broken date’ swaps** (off-cycle starts) expose curve engine weaknesses more than standard swaps?”

### Mechanism

Broken-date swaps create coupon periods that don’t align with your node grid (IMM, EOM, meeting). Pricing requires accurate integration of forwards over irregular windows.

If your curve engine relies on “nice alignment” assumptions, broken-dates reveal:

* window integration errors
* interpolation artifacts
* missing sub-grid valuation

### Desk symptom

Standard swaps fit fine, broken-dates misprice persistently.

### Fix

Ensure valuation uses exact cashflow dates and integrates the curve across those intervals, independent of calibration knot placement.

---

## Summary table

### Annoying stuff: what it is and what it breaks

|      Q# | Annoying thing           | Breaks first               | Common wrong response             |
| ------: | ------------------------ | -------------------------- | --------------------------------- |
|     109 | calendar update          | FF vs OIS mismatch         | “tune lambda”                     |
|     110 | ModFollowing/EOM         | phantom turns              | “market is weird”                 |
|     111 | EOM schedule mismatch    | switches/flies             | “curve bug” (it is, but in dates) |
| 112–113 | stub-at-IMM rules        | strip ↔ swaps              | “futures convexity”               |
|     114 | turn + meeting overlap   | meeting jump size          | “meeting expectations changed”    |
| 115–116 | holiday broken weeks     | weekly OIS                 | “quotes are noisy”                |
|     117 | daycount mismatch        | fake basis                 | “basis widened”                   |
|     118 | spot date drift          | front XCCY basis kink      | “FX moved”                        |
| 119–120 | USD CSA leakage          | other currencies front end | “EUR curve is wrong”              |
|     121 | coupon clustering        | par shifts                 | “solver instability”              |
|     122 | weekend rate application | short OIS                  | “compounding approx is fine”      |
|     123 | IMM+month-end collision  | strip + swaps + basis      | “illiquid quotes”                 |
|     124 | broken-date swaps        | irregular window pricing   | “add knots everywhere”            |

### Quick “desk sanity suite” I’d run after any of these changes

| Test                                      | Why it catches this batch                   |
| ----------------------------------------- | ------------------------------------------- |
| 1W/2W OIS around holidays                 | detects weekend/holiday compounding bugs    |
| Meeting-month FF future + 1W OIS straddle | detects meeting effective date placement    |
| Turn window instrument (EOM/QE/YE)        | detects turn modeling + roll rules          |
| One broken-date swap repricer             | detects window integration/interp artifacts |
| FX swap points + 1Y XCCY basis            | detects spot lag / cross-currency leakage   |

That’s the batch. If you want to keep going, the next natural set is **“debug playbooks”**: for each symptom (comb residuals, basis kink, DV01 snapping), the exact triage steps and which “one-line check” rules things out fastest.

----
Sure. Here’s the **debug playbook batch**, continuing numbering from **Q124**. Each one is a **symptom → fastest triage path → root causes → confirm/kill tests → fix**. This is what you want at 4:59pm when someone says “curve’s weird” and expects you to be a wizard.

---

## Q125) “Symptom: **Comb pattern** residuals across adjacent futures/IMM contracts. What do you check first?”

### What a comb pattern usually means

A repeating +/− residual across neighboring contracts screams **date/window mapping error**, not “market moved.”

### Triage in the right order

1. **Contract effective start/end dates**

* last trade date, settlement date, accrual window
* IMM boundary alignment

2. **Daycount and calendar**

* does the contract window include weekends/holidays correctly?
* are you counting days consistently across contracts?

3. **Convexity toggle consistency**

* if convexity is applied inconsistently by contract, it can mimic a pattern

### Kill test (fast)

* Recompute model-implied rate for each contract using a frozen curve and just the contract window logic.
  If the pattern persists with fixed curve, it’s the mapping.

### Fix

* Correct the contract date schedule, then rebuild.
* Only after that consider weight/tolerance changes.

---

## Q126) “Symptom: **Meeting-month futures** misfit spikes, but other months are fine. What’s the shortest path to root cause?”

### Most likely causes (ranked)

1. **Meeting effective date knot missing or mislocated** (announcement vs effective)
2. **Month window definition wrong** (end-of-month calendar mishandling)
3. **Turn overlap** (quarter-end/year-end inside that month)
4. **Weekend/holiday rate application** inconsistent with index rules

### Confirm/kill tests

* Price the meeting-month contract by explicitly computing:
$$
  \bar r_M=\frac{n_{pre}r_{pre}+n_{post}r_{post}}{n_{pre}+n_{post}}
$$
  If you can’t produce a clean pre/post decomposition, your segmentation is wrong.

* Reprice a 1W OIS that straddles the meeting. If it also breaks, it’s almost certainly meeting boundary logic.

### Fix

* Insert the meeting effective boundary in the ON forward grid and relax smoothing across it.

---

## Q127) “Symptom: **Short OIS (1W/2W)** is off, but the futures strip looks fine. What do you do?”

### What that usually means

Weekly OIS is brutally sensitive to **weekend/holiday compounding** and schedule mechanics. Month averages can look fine because errors average out.

### Triage steps

1. **Weekend/holiday compounding rule**

* verify multi-day accrual factors (Friday fixing applying over weekend, etc.)

2. **Business day adjustments**

* accrual start/end dates and payment date adjustments

3. **Fixings handling**

* ensure realized fixings are used for past days
* curve only for future portion

### Kill test

* Take the curve as fixed and compute the OIS coupon accrual day-by-day (or segment-by-segment).
  If PV still off, it’s schedule/compounding logic.

### Fix

Correct rate application on non-business days and verify against a reference OIS accrual calculator.

---

## Q128) “Symptom: **DF monotonicity constraint binding everywhere** and solver struggles. What’s your diagnosis path?”

### What it screams

You’re enforcing constraints inconsistent with the regime or instrument set.

### Likely causes

1. Negative-rate regime but still enforcing strict (D(t_{k+1})\le D(t_k))
2. Curve representation/interpolation causing oscillations in DF space
3. Too many hard bounds causing active-set thrash

### Confirm/kill

* Check sign of implied zero rates and whether DF increases are actually economically plausible.
* Temporarily relax DF monotonicity and keep positivity only; if solve stabilizes, your monotonicity constraint was wrong for the regime.

### Fix

* Use regime-aware constraints:

  * (D(t)>0) always
  * allow mild DF increases if negative rates plausible
  * enforce forward bounds / smoothness instead of strict monotonic DF.

---

## Q129) “Symptom: **Basis curve suddenly kinks at 3M/6M** and DV01 jumps buckets. What’s your playbook?”

### What it usually means

1. **Constraint thrash** (bounds binding/unbinding)
2. **FX/spot lag or XCCY coupling inconsistency** (if cross-currency)
3. **Stale overweighted basis quote**
4. **Interpolation changed** (same PV fit, different sensitivity distribution)

### Triage

1. Check **constraint binding map**: which nodes are clamped?
2. Check **outlier detection**: is one basis quote outside bid/ask?
3. Check **cut alignment**: are basis, FX forwards, and OIS from the same timestamp?
4. Check **calendar/spot lag**: especially if kink is front end

### Kill test

* Freeze discount curve and rebuild only projection/basis.
  If kink persists, it’s in basis/projection space. If kink disappears, discount curve or cross-currency linkage is the driver.

### Fix

* Downweight stale quote, add localized smoothing on basis, and confirm FX/spot conventions.

---

## Q130) “Symptom: **FX forwards reprice poorly** after a USD curve change, but USD local OIS residuals are still fine. What’s happening?”

### Most common root cause

USD curve change modified short-dated DF ratios (meeting/turn segmentation), and FX forwards are extremely sensitive to those.

### Triage

1. Check if USD change occurred near:

* meeting month
* quarter-end/year-end turn window

2. Check whether FX forward build uses same cut/calendar and spot lag.

3. Check whether cross-currency system is “forcing” consistency or allowing tolerances.

### Confirm/kill

Reprice FX forwards with:

* old USD curve vs new USD curve
* same EUR curve
  If error tracks USD curve delta, it’s propagation, not FX data.

### Fix

* Ensure meeting/turn modeling is correct and localized.
* Rebalance weights between FX forwards and XCCY basis so one doesn’t become the garbage collector.

---

## Q131) “Symptom: Live curve and risk curve diverge and PnL explain looks ‘wrong’. What’s your debugging flow?”

### First: decide if it’s expected

Live curve often uses extra constraints (flies/switches, smoothing-to-prior, streaming filters). Risk curve may be more “pure.”

### Triage

1. Compare objectives:

* instrument sets included/excluded
* weights/tolerances
* regularization and priors

2. Compare constraint activity:

* are flies/switches binding?
* are bounds binding intraday?

3. Compare mapping stability:

* bump/rebuild response norms for the same quote bump across both curves.

### Confirm/kill

Rebuild live curve using risk curve instrument set (temporarily).
If divergence vanishes, it’s methodological by design.

### Fix

* Governance: define which curve drives what (pricing, risk, PnL explain) and ensure consistent reporting mapping.

---

## Q132) “Symptom: **One tenor (18M/2Y)** has persistent misfit across days. What’s your likely culprit?”

### Most likely causes

* Schedule generation edge case around a recurring holiday window
* EOM/Modified Following inconsistency
* Stub rule mismatch that hits that tenor boundary

### Triage

1. Print coupon date schedule for that tenor and compare to reference.
2. Check whether those coupons land near holidays consistently across days.
3. Compare daycount and accrual factors for that tenor vs neighbors.

### Kill test

Use the same curve and reprice the instrument with:

* your schedule logic
* reference schedule logic
  If PV differs, it’s schedule.

### Fix

Correct date adjustment logic; don’t touch weights until schedules match.

---

## Q133) “Symptom: You fixed a calendar bug and now **DV01 ladder shifted** everywhere. How do you show it’s a ‘good’ change?”

### Reality

A correct calendar fix changes cashflow timing and window weights. That can legitimately shift:

* PV
* risk allocation

### What you do

1. Produce **before/after** pack:

* residuals by instrument type
* out-of-sample checks
* DV01 differences by bucket

2. Show stability improvement:

* bump/rebuild response norms improved (or at least didn’t worsen)
* kink metrics not worse

3. Explain causality:

* list the exact dates that changed and which instruments they affect

### Key point to say

“This is a deterministic mechanical change, not a market view. The new risk ladder is more correct because cashflows are now timed correctly.”

---

## Q134) “Symptom: After adding an instrument (new quote), curve PV fit improves but **risk becomes unstable**. What’s the playbook?”

### Cause

Added DOF pressure or ill-conditioning. More constraints can create sharper sensitivity mapping if regularization isn’t adapted.

### Triage

1. Compute condition proxy of (J^\top WJ) or monitor solver warnings.
2. Check whether the new instrument is redundant/overlapping.
3. Check whether it is noisy/stale but overweighted.

### Kill test

Rebuild without the new instrument (or set it as a check).
If risk stabilizes, the instrument is destabilizing the solve.

### Fix

* Use the new quote as a check or downweight it.
* Add localized regularization or adjust knot placement to absorb it smoothly.

---

## Q135) “Symptom: **Kinks appear** right at the join between front-end meeting grid and long-end swap grid. What do you do?”

### Causes

* Uniform regularization across the curve despite regime change in knot density
* Knot placement too sparse just after the join
* Interpolation scheme overshooting

### Triage

1. Plot instantaneous forwards and second differences around the join.
2. Check penalty structure: is (\lambda) maturity-dependent?
3. Check if there is a “bridge knot” near 18M/2Y.

### Fix

* Add a bridge knot or strengthen smoothing just around the join.
* Ensure meeting boundary relaxations don’t spill into long-end.

---

## Q136) “Symptom: You see **negative forwards** only in a narrow region, and only on some days. What’s your root cause shortlist?”

### Shortlist

1. Cubic interpolation overshoot between sparse knots
2. Local outlier quote forcing curvature
3. Active constraint switching (some days binds, some days not)
4. Missing structural node (turn/meeting), causing solver to fake curvature

### Kill tests

* Switch to monotone interpolation temporarily.
* Downweight suspected outlier and rebuild.
* Add missing event knot and see if negativity disappears.

### Fix

Prefer positivity-preserving representation (log-DF, monotone forward interpolation) + event nodes + robust loss.

---

## Q137) “Symptom: **Cross-currency basis DV01 explodes** after a small USD front-end quote move. What’s the playbook?”

### What it indicates

The cross-currency system is hyper-sensitive because:

* USD front end is poorly segmented (meeting/turn missing)
* or FX forward constraints are too tight
* or XCCY basis is used as the error sink

### Triage

1. Check USD meeting/turn node configuration.
2. Check FX forward weights/tolerances at short tenors.
3. Check whether dependent curves are overly constrained.

### Confirm/kill

Freeze USD curve and bump FX forwards instead: does basis DV01 still explode?
If yes, coupling weights are the culprit. If no, USD shape is.

### Fix

Stabilize USD segmentation, rebalance weights, and ensure basis curve has sensible smoothness and bounds.

---

## Q138) “Symptom: A trader override on a single quote causes a **non-local curve move**. How do you debug whether that’s expected?”

### Expected reasons

* The overridden quote is a strong anchor (cash/OIS) affecting annuities broadly.
* Regularization favors smooth global moves over local kinks.

### Unexpected reasons

* The quote is mis-mapped (wrong instrument definition)
* Wrong schedule logic causing broad PV differences
* Constraint binding causing solver to reroute motion globally

### Debug steps

1. Compute sensitivity of curve nodes to that quote bump: which nodes should move?
2. Compare to actual node move after override.
3. Check constraint activity changes (active set switch).

### Fix

If motion is too global, consider:

* adding a local knot near that quote’s maturity
* adjusting penalty to allow local adjustment rather than global rotation.

---

# Summary tables

## Symptom → Most likely culprit

|  Q# | Symptom                        | Top culprit                              |
| --: | ------------------------------ | ---------------------------------------- |
| 125 | comb residuals (strip)         | contract window/IMM date mapping         |
| 126 | meeting-month spike            | meeting effective date / window logic    |
| 127 | weekly OIS off                 | weekend/holiday compounding & schedule   |
| 128 | monotonicity binds             | regime mismatch (neg rates)              |
| 129 | basis kink + DV01 jump         | constraints/outliers/cut misalign        |
| 130 | FX forwards degrade            | USD front-end DF ratio propagation       |
| 131 | live vs risk diverge           | different objectives/constraints         |
| 132 | persistent one tenor off       | schedule/EOM/stub edge case              |
| 133 | calendar fix changed DV01      | correct timing changed mapping           |
| 134 | new quote destabilizes risk    | ill-conditioning/overfit                 |
| 135 | kink at grid join              | penalty/knot density mismatch            |
| 136 | intermittent negative forwards | interpolation/outlier/event knot missing |
| 137 | XCCY DV01 explodes             | coupling weights + USD segmentation      |
| 138 | override causes global move    | penalty structure or mis-mapped quote    |

## The “5-minute sanity kit” at 4:59pm

| Step                                                     | What it tells you fast                  |
| -------------------------------------------------------- | --------------------------------------- |
| Freeze curve, reprice with pure schedule logic           | schedule/compounding bug vs curve shape |
| Residual heatmap by tenor/type                           | which bucket is paying for the change   |
| Constraint binding report                                | whether risk is non-smooth              |
| Rebuild with robust loss                                 | outlier-driven distortion check         |
| Micro-suite: 1W OIS + meeting-month future + turn window | catches 80% of front-end disasters      |

If you want an even more “desk-operational” version, I can turn each of these into a **literal checklist** (exact printouts, plots, and one-line assertions you’d automate in CI for curve builds).

----
Alright. Here’s the “**trader walks over / pings you** with a messy request” batch. Same universe: multi-curve, meetings, IMM/futures, cash, swaps, basis, XCCY. Continuing numbering from **Q138**.

---

## Q139) Trader: “My **front-end DV01** moved buckets today but the market barely moved. Explain it. Now.”

### What I answer (calmly, because yelling doesn’t change Jacobians)

This is almost always one of three things:

1. **Active constraints changed** (a floor/monotonicity bound started binding or stopped binding)
2. **A short-end structural node changed** (meeting effective date knot, turn node, holiday window)
3. **Methodology changed** (live vs risk curve settings, smoothing/prior, instrument weights)

### What I do (5-minute triage)

1. **Compare curve deltas**: yesterday vs today instantaneous forwards (f(t)) in 0–2Y
2. **Check constraint activity report**: which bounds are binding today vs yesterday
3. **Run one bump-rebuild** on the most influential front-end quote (ON, 1W OIS, meeting-month future) and compare node move vectors:
$$
   \Delta x = x(q+\epsilon)-x(q)
$$
   If (|\Delta x|) changed a lot for same (\epsilon), your mapping (\partial x/\partial q) changed.

### What I tell the trader (plain English)

“PV didn’t move much, but the **curve calibration mapping** changed. When constraints bind/unbind or a meeting/turn boundary is treated differently, the system routes the same sensitivity through different nodes, so DV01 shifts buckets.”

### Deliverable

* A one-page “**why DV01 moved**” note: constraint changes + top 3 quotes driving the shift + before/after forward plot.

---

## Q140) Trader: “My **meeting-month** pricing looks wrong. The OIS curve doesn’t reflect the meeting jump.”

### My first response

If the curve doesn’t show a jump (or shows it in the wrong place), the usual causes are:

* meeting **effective** date knot missing/misplaced
* month averaging window for futures wrong
* meeting coincides with a **turn** and you modeled only one

### Step-by-step fix

1. Insert knot at meeting effective boundary (T_m) into ON forward segmentation
2. Recompute meeting-month future model average:
$$
   \bar r = \frac{n_{pre} r_{pre} + n_{post} r_{post}}{n_{pre}+n_{post}}
$$
3. Relax smoothing across the meeting boundary (don’t flatten it)

### What I show them

* Pre/post implied rates (the decomposition)
* Repricing of: meeting-month future + 1W OIS straddle + 1M OIS

### Desk warning

If you don’t also model turns, the “meeting jump” will absorb turn premium and look too big.

---

## Q141) Trader: “The **IMM strip fits perfectly**, but swaps are off and my switch trades look weird.”

### Translation

You overconstrained the projection curve to match IMM futures, and the system is paying for it in swap space (or basis space).

### What I do

1. Residual migration check: strip vs swaps vs basis
2. Check convexity and IMM date mapping (a perfect fit can still be “perfectly wrong”)
3. Reduce strip weight / widen strip tolerance in belly; enforce smoothness and keep swaps within bid/ask
4. Ensure stub/IMM alignment for swap schedules (stub bugs often show up as “swap vs strip disagreement”)

### What I tell them

“Perfect strip fit is not a virtue if it forces curvature that makes swaps inconsistent. Swaps are the cashflow-weighted truth for longer windows.”

---

## Q142) Trader: “Can you give me a **carry/roll** view that’s consistent with the curve build, not some toy Bloomberg hack?”

### My answer

Yes, but it depends on using the **same curve objects** (discount + projection + basis) and the **same schedule logic** as pricing.

### How I compute (desk-real)

For a trade PV:
$$
PV(t) = \sum_k D(t_k),CF_k(F(\cdot))
$$
* **Carry**: expected PnL from accrual/known fixings and expected forwards, holding curves fixed
* **Roll-down**: change in PV if time advances (\Delta t) and you re-express the trade on the same curve (or on a rolled curve assumption)

### Deliverable

* Carry by component: funding (discount), index accrual (projection), basis
* Roll by key tenors (1M, 3M, 6M, 1Y…)
* Explain where “carry” is actually coming from (front-end meeting path vs curve slope)

---

## Q143) Trader: “Why did **XCCY basis** kink at the front end after the USD curve update? Fix it.”

### What I explain

Front-end FX forwards and XCCY PVs are hypersensitive to **DF ratios**. If USD front end changed due to meeting/turn segmentation, the implied FX forward consistency shifts, and basis becomes the garbage collector.

### Debug path

1. Reprice FX forwards with old vs new USD curve (EUR curve fixed)
2. Identify if the kink is coming from:

* FX forward constraint tightness
* XCCY basis weight too high
* missing USD turn node (classic)

### Fix options (in order)

* Correct USD meeting/turn segmentation (best)
* Rebalance FX forward vs XCCY weights (pragmatic)
* Add smoothness prior to basis front end so it doesn’t spike (defensive)

### What I give trader

* “cause-of-kink” report: which constraint forced which node move
* Proposed weight/tolerance adjustment with impact on PV/risk

---

## Q144) Trader: “We got a weird **quarter-end**. The curve flattened it. My funding trades look wrong.”

### Translation

Your smoothing flattened a real turn premium.

### What I do

1. Ensure dedicated **turn nodes** exist around QE/YE window
2. Lower smoothing penalty locally so the turn survives
3. Confirm that turn effect is localized (doesn’t leak)

### What I show

* ON forwards showing a localized turn spike
* Impact on:

  * short OIS
  * cash instruments
  * any turn-sensitive swaps

### Trader-facing explanation

“Turn is a funding/calendar effect. If we don’t model it explicitly, the solver spreads it out and your funding trades lose the very thing you’re trading.”

---

## Q145) Trader: “A client asked for a **broken-date swap** quote and our price is off vs street. Where’s the bug?”

### My first suspicion

Broken-date swaps expose:

* window integration errors
* schedule/roll bugs
* interpolation overshoot on irregular windows

### Debug checklist

1. Print full cashflow schedule and compare to reference
2. Verify accrual factors and date adjustments
3. Freeze curve and recompute PV using exact window integration across segments
4. Compare to a standard swap nearby: if standard matches but broken doesn’t, it’s almost always schedule/integration

### Fix

* Valuation must integrate (f(t)) over the real accrual window, not approximate with nearest node forward.

---

## Q146) Trader: “Can we **override one quote** (say 5Y swap) but keep the curve move local? I don’t want the whole belly to rotate.”

### Honest answer

By default, the optimizer prefers smooth global moves. If you want locality, you have to **enable locality**.

### How I do it

1. Add a knot near 5Y (or refine knot density in that region)
2. Reduce curvature penalty locally so a local adjustment is “cheap”
3. Add a locality prior: penalize movement outside a neighborhood:
$$
   \eta \sum_{k \notin \mathcal{N}(5Y)} (x_k-x_k^{prev})^2
$$
### What I warn them

Locality and smoothness fight each other. You can have:

* local override + stable risk, **or**
* global smoothness + clean curve,
  but not unlimited amounts of both.

---

## Q147) Trader: “Our **basis swap** quotes disagree across sources. Which one is ‘truth’ for the curve?”

### My answer (the only honest one)

None of them is “truth.” The market is inconsistent. So we encode **liquidity hierarchy** and **robust fitting**.

### What I do

1. Build core curve from the most tradable/tight set (usually one source)
2. Use other sources as checks
3. If we must combine: use bid/ask-aware weighting and robust loss (Huber) so one source can’t bully the curve

### Trader-facing output

* residuals by source
* which quotes were downweighted and why (stale, wide, inconsistent)

---

## Q148) Trader: “My book’s risk is reported off the **risk curve**, but I trade off the **live curve**. Why do they disagree and what do I hedge?”

### Reality

They disagree because they optimize different objectives. Live curves often include:

* flies/switches
* time priors / streaming filters
* extra shape constraints
  Risk curves often prioritize:
* stability
* clean mapping to primary instruments
* governance

### What I advise (desk-grade)

* Hedge **PnL** on the curve that drives your price and marks (often live).
* Hedge **reported risk** using the risk curve sensitivities if that’s what management tracks.
* But reconcile: show the delta between curve sets and where it comes from.

### Deliverable

* “curve-set difference” decomposition: discount vs projection vs basis contributions
* hedge translation: how a live hedge maps into risk-curve buckets

---

## Q149) Trader: “A quote came in late (stale) and now the curve looks unstable. Can you make the system not overreact intraday?”

### My response

Yes. Intraday should be robust to microstructure noise.

### Production tactics

1. **Hysteresis / persistence filters**: don’t accept a move until it persists
2. **Time prior**:
$$
   \eta|x-x_{prev}|^2
$$
3. **Robust loss** so one bad print doesn’t dominate
4. **Publish gating**: if diagnostics fail (kink metrics, residual explosion), do not publish

### Trader benefit

Stable intraday marks and hedges, fewer “why did the curve flip” episodes.

---

## Q150) Trader: “The curve ‘looks fine’ but my **hedge ratios** are insane. Prove the curve isn’t overfitting.”

### What I do

Overfitting shows up in (\partial x/\partial q), not just residuals.

### Proof pack

1. Bump key quotes by 1bp, rebuild, measure node move norms
2. Kink metrics on forwards: (\max|\Delta f|), (\max|\Delta^2 f|)
3. Out-of-sample repricing: remove one instrument, rebuild, see if curve stays sane
4. Check conditioning proxy: warning if system is near singular

### Fix if it is overfit

* reduce knots (or earn knots by liquidity)
* strengthen regularization
* add event nodes instead of allowing fake wiggles
* robust loss and sensible weights

---

# Summary tables

## Trader request → What I do first

|  Q# | Trader says                | First diagnostic I run             | Most common root cause          |
| --: | -------------------------- | ---------------------------------- | ------------------------------- |
| 139 | DV01 moved buckets         | constraint activity + bump/rebuild | bounds binding/unbinding        |
| 140 | meeting jump wrong         | meeting knot + straddle test       | effective date misplacement     |
| 141 | strip perfect, swaps off   | residual migration                 | overweight strip / date mapping |
| 142 | need carry/roll            | same curve objects + schedules     | inconsistent tooling            |
| 143 | XCCY kink after USD update | FX repricing old vs new USD        | USD segmentation leakage        |
| 144 | turn flattened             | turn nodes + local smoothing       | missing turn modeling           |
| 145 | broken-date off            | schedule + exact integration       | date/integration bug            |
| 146 | local override needed      | add knot + local penalty           | optimizer prefers global move   |
| 147 | basis sources disagree     | robust fit + hierarchy             | inconsistent market prints      |
| 148 | live vs risk mismatch      | objective diff + mapping           | different constraints/priors    |
| 149 | intraday unstable          | time prior + robust loss           | microstructure noise            |
| 150 | hedges insane              | bump/rebuild norms                 | overfitting / ill-conditioning  |

## The “don’t waste time” rules

| Rule                                                                        | Why it saves your life               |
| --------------------------------------------------------------------------- | ------------------------------------ |
| If residuals show a pattern by tenor, suspect **dates/windows** before math | markets are noisy, code is worse     |
| If PV fit is good but hedges are wild, suspect **overfit or constraints**   | risk stability is the real objective |
| If XCCY breaks after USD front-end change, suspect **DF ratio propagation** | cross-currency is a coupling machine |
| If only a few tenors fail repeatedly, suspect **schedule/EOM/stub**         | localized date bugs are common       |

If you want the next batch, I can do the same but for **“trader asks for a new feature”** (custom curve views, stress ladders, scenario shocks, hedging decomposition), not just debugging.

---
Sure. More “**trader pings you with a problem**” scenarios, with **detailed answers**, continuing numbering from **Q150**. Because nothing says “career choice” like debugging a curve minutes before close.

---

## Q151) Trader: “Why are there **negative forwards** in the belly today? Fix it without wrecking fit.”

### What I say first

Negative forwards are not automatically wrong (depends on regime), but **sudden, localized negativity** is usually an artifact: interpolation overshoot, missing structural nodes, or one quote bullying the solve.

### My triage

1. **Locate it precisely**

* Plot instantaneous forwards (f(t)) and second differences (\Delta^2 f) around the negative region.
* Identify the neighboring knots and which instruments load there (annuity weights / sensitivity buckets).

2. **Check if it’s interpolation**

* If you’re using cubic on DFs or zero rates, overshoot is common.
* Quick kill test: switch temporarily to monotone interpolation (or monotone forward interpolation) and rebuild.

3. **Check if it’s a missing structural node**

* Meeting/turn in that region? If yes, add an event node and relax smoothing across it.

4. **Check outliers**

* Which instrument residual is “dragging” the curve into a dip? Often a stale swap/basis point.

### Fix options (in order)

* Move to **positivity-preserving representation** (monotone forward interpolation, log-DF parametrization).
* Add event nodes (meeting/turn) instead of smoothing everything.
* Use robust loss / downweight the bully quote.
* Only then consider hard floors (hard floors create non-smooth risk).

### What I deliver

* Before/after forward plot
* List of instruments whose residuals improved/worsened
* Change in bump/rebuild stability norms (prove we didn’t trade fit for instability)

---

## Q152) Trader: “My **5s10s switch** is mispricing vs broker. Par swaps fit fine. How is that possible?”

### Explanation (desk-real)

A switch is *relative value*:
$$
S(10Y)-S(5Y)
$$
Par swaps can fit within tolerance while the *relative curvature* is off because:

* regularization choices prefer smoothness,
* flies/switches weren’t in calibration,
* knot density around 5–10Y is too sparse or the join is kinked.

### Debug steps

1. Reprice 5Y and 10Y par instruments (confirm they’re within tolerance).
2. Compute the implied **curvature metric** around 5–10Y (e.g., second difference on par rates/forwards).
3. Check whether the curve join (front grid → long grid) sits in that region.

### Fix

* Add the switch (soft) as a live-curve constraint if trading uses it.
* Add a bridge knot near 7Y (often solves curvature without overfitting).
* Tune local smoothing so curvature can exist without creating kinks.

### Output

* “Switch explain”: which nodes moved to correct (S(10)-S(5)), and what it did to DV01 buckets.

---

## Q153) Trader: “The **fly (2s5s10s)** is way off today. I need a curve that respects flies.”

### What I answer

Flies are essentially curvature targets. Forcing them hard is a great way to create kinks and unstable sensitivities if you’re not careful.

### What I do

1. Verify whether the fly is:

* a traded market quote we should respect, or
* a derived/indicative level that shouldn’t dominate.

2. Add fly as a **soft residual**:
$$
   r_{\text{fly}} = \Big(S(5)-\tfrac12(S(2)+S(10))\Big) - \text{Fly}^{mkt}
$$
   with moderate weight and robust loss.

3. Monitor:

* kink metrics (\max|\Delta^2 f|)
* bump/rebuild response norms around 2/5/10Y

### Fix strategy

* If fly is important: include it in live curve with controlled weight.
* If it destabilizes risk: widen tolerance and rely on par fits + smoother curvature penalties.

### Deliverable

* “Fly fit vs stability” report: how much fly improves vs how much risk worsens.

---

## Q154) Trader: “I’m seeing **PnL on basis swaps** but risk says it’s all in the projection curve. Why is basis risk ‘leaking’?”

### Root cause

You’re not cleanly separating:

* 3M curve
* 6M curve
* explicit basis curve/function

If basis is embedded implicitly (or basis is over-smoothed), the solver will fit basis moves by reshaping one projection curve, so risk appears as “projection DV01” rather than basis DV01.

### Debug path

1. Check curve architecture: do we have an explicit basis object or not?
2. Freeze discount + freeze one projection curve, rebuild the other:

* see if basis residuals can be explained without distorting projections.

### Fix

* Introduce explicit basis curve (preferred) or at least constrain basis difference to be smooth and local:
$$
  b(t)=f^{6M}(t)-f^{3M}(t)
$$
  penalize curvature of (b(t)) rather than forcing one curve to absorb it.

### Deliverable

* Risk decomposition before/after: projection DV01 vs basis DV01.

---

## Q155) Trader: “Our **FRAs** and **IMM futures** disagree today. Which one do we trust in the projection curve?”

### The only correct answer

Neither universally. Trust depends on **liquidity, cut, and instrument definition**. Also FRAs and futures don’t always map 1:1 if conventions/convexity differ.

### Triage

1. Ensure IMM contract date mapping is correct (comb-pattern check).
2. Confirm convexity/settlement assumptions for futures.
3. Compare bid/ask widths and tradeability today.

### Practical policy

* Use the **most tradable** as primary (often futures in some markets, FRAs in others).
* Use the other as **checks** unless they’re both liquid.
* If both liquid but disagree: widen tolerances and let the curve fit in a least-squares sense.

### Output

* A “who paid” residual migration table: FRA residuals vs futures residuals after the solve.

---

## Q156) Trader: “The **1Y point** looks ‘sticky’ and the curve makes a weird kink right after it.”

### Root cause shortlist

* 1Y instrument overweighted or tight tolerance
* knot density jump at 1Y (front-end grid ends there)
* smoothing penalty not maturity-aware
* missing bridge knot at 18M/2Y

### Fix

* Add a 18M/2Y bridge knot.
* Make regularization maturity-dependent so the join doesn’t dump curvature at the boundary.
* Keep meeting/turn relaxations localized so they don’t spill into the join.

### What I show

* forward curve around 6M–3Y
* second differences around the join
* which instruments constrain 1Y vs 2Y

---

## Q157) Trader: “Client wants a quote on a **forward-starting swap** (e.g., 1Yx5Y). Our price is off vs street.”

### Likely causes

Forward-starting swaps are sensitive to **forward curve shape** and schedule alignment:

* start date roll conventions
* spot lag
* daycount and payment frequency
* curve integration across the start boundary

### Debug steps

1. Validate schedule: exact start date, end date, coupon dates, stubs.
2. Reprice using frozen curves and confirm cashflow timing.
3. Compare the implied forward swap rate to street by decomposing:
$$
   S(1Yx5Y) \approx \frac{P(1Y)-P(6Y)}{A(1Y,6Y)}
$$
   (With proper discounting and projection mechanics, not a toy formula.)

### Fix

* If schedule mismatch: correct it.
* If shape mismatch: check whether your curve uses fly/switch constraints that street doesn’t, or whether your smoothing is flattening mid-curve.

### Deliverable

* Explain decomposition: which part is discounting vs projection vs basis.

---

## Q158) Trader: “I overrode a single quote (say 2Y swap) and the whole curve rotated. I only wanted a local move.”

### Explanation

Your optimizer is choosing the cheapest move under regularization. If curvature is expensive, it prefers a smooth global rotation.

### How to make it local (without chaos)

* Add local knot density around 2Y.
* Add a locality penalty outside the neighborhood:
$$
  \eta\sum_{k\notin \mathcal{N}(2Y)} (x_k-x_k^{prev})^2
$$
* Keep global smoothness but allow local flexibility.

### Output

* “override impact” report: node move vector before/after locality controls + DV01 changes.

---

## Q159) Trader: “We switched a trade’s CSA from **EUR collateral to USD collateral** and the PV moved more than expected. Is our curve set wrong?”

### What I say

Changing CSA changes discounting and cross-currency consistency, so PV and risk can move *meaningfully*. The question is whether it moved for the **right** reasons.

### Debug steps

1. Verify trade’s curve set selection (CSA id → discount curve object).
2. Ensure EUR|USD-collateral curve exists (or correct construction path via XCCY/FX).
3. Decompose PV delta:

* pure discounting change
* FX forward / basis consistency change
* projection curve unchanged (unless system couples it)

### Proof

Reprice with:

* same projection curves
* only discount curve switched
  If most delta comes from discounting and cross-currency linkage, it’s expected.

---

## Q160) Trader: “Our **cleared vs bilateral** pricing differs. The market doesn’t. Why do we?”

### Root cause

Cleared vs bilateral can imply different:

* collateral rate / CSA discounting curve
* payment lags and conventions
* curve set (multi-curve under different discounting)

### Debug flow

1. Confirm discount curve selection: clearing house discount vs bilateral CSA.
2. Check payment lag and compounding conventions (cleared OIS often has strict standardization).
3. Compare curve inputs: are they built off the same instrument universe and cut?

### Deliverable

* “clearing delta explain”: PV and DV01 difference decomposed into discounting vs conventions vs curve-fit differences.

---

## Q161) Trader: “The **front-end OIS** looks off only around a long weekend. Everything else is fine.”

### Translation

Weekend/holiday accrual handling bug or calendar mismatch.

### Fix path

1. Validate weekend rate application (Friday fixing spanning weekend).
2. Confirm holiday calendar matches the index and venue.
3. Check Modified Following behavior around month-end if weekend coincides.

### Quick kill test

Freeze the curve and recompute OIS coupon accrual day-by-day. If still off, it’s schedule/compounding logic, not the curve.

---

## Q162) Trader: “Why did adding a single **9M OIS** quote make my **IMM strip** fit worse?”

### Explanation

You reduced a degree of freedom the optimizer was using as a buffer. 9M OIS inserts an additional PV constraint on the same ON forward curve that IMM strip windows depend on.

### What I do

* Check whether IMM mapping is correct (don’t blame the new quote if you have a date bug).
* Rebalance weights so 9M doesn’t dominate.
* Consider adding a knot in that region if the curve needs the DOF legitimately.

### Deliverable

* Residual migration table: what worsened to improve 9M.

---

## Q163) Trader: “My **hedge ratios** changed after we swapped interpolation scheme. PV didn’t. That’s unacceptable.”

### Reality

PV equality at calibration points doesn’t guarantee the same sensitivity mapping. Interpolation changes (\partial D(t)/\partial x_k) and how node moves distribute.

### What I do

1. Run quote-bump rebuild comparisons pre/post interpolation change.
2. Compare node move vectors and DV01 ladders.
3. Decide whether the new scheme is acceptable for risk governance.

### Fix

* For risk curves, prefer monotone/shape-preserving methods that don’t create overshoot.
* If traders need consistent hedges, constrain the mapping (stronger regularization, stable parametrization).

---

## Q164) Trader: “I think your curve is overfitting. Show me in one slide.”

### One-slide proof

* Residuals: within bid/ask is necessary but not sufficient.
* Stability metric: max node move from 1bp quote bump.
* Kink metrics: (\max|\Delta f|), (\max|\Delta^2 f|).
* Out-of-sample repricing: remove one instrument, rebuild, see if curve stays sane.

If those look ugly, you’re overfitting even if PV residuals are pretty.

---

## Q165) Trader: “Can you stress the curve for a **surprise central bank hike** and tell me what breaks across instruments?”

### Proper stress (not a toy parallel shift)

* Insert a discrete jump at the meeting effective date:

  * increase post-meeting ON forward segment(s)
* Rebuild coupled curves:

  * discounting (OIS)
  * projection curves
  * basis curves
  * XCCY/FX (if multi-currency)

### What breaks/changes first

* Meeting-month futures repricing
* Short OIS straddling meeting
* FX forwards / XCCY basis front end (if USD is anchor)

### Deliverable

* PV and DV01 impact by instrument type bucket (cash, OIS, futures, swaps, basis, XCCY)
* Identify where the system is absorbing inconsistency (basis kinks vs FX misfit vs local curve distortion)

---

# Summary tables

## Q151–Q165 quick map

|  Q# | Trader issue              | Fastest real root cause                  | First diagnostic             |
| --: | ------------------------- | ---------------------------------------- | ---------------------------- |
| 151 | negative forwards         | interpolation/outlier/event node missing | monotone interp kill-test    |
| 152 | switch misprices          | curvature/knot density                   | curvature + join check       |
| 153 | fly off                   | fly as hard target causes kinks          | kink + bump/rebuild norms    |
| 154 | basis risk leaks          | basis not explicit / over-smoothed       | freeze curves, rebuild basis |
| 155 | FRA vs IMM disagree       | date mapping/convexity/cut               | comb-pattern + mapping       |
| 156 | kink after 1Y             | join/knot density mismatch               | forward plot + (\Delta^2)    |
| 157 | fwd-start swap off        | schedule/integration/shape               | schedule + decomposition     |
| 158 | override rotates curve    | regularization prefers global moves      | node move vector             |
| 159 | CSA switch PV big         | correct collateral curve set             | PV decomposition             |
| 160 | cleared vs bilateral diff | discounting + conventions                | curve-set selection check    |
| 161 | long weekend OIS off      | weekend accrual logic                    | day-by-day accrual repricer  |
| 162 | 9M OIS worsened IMM       | constraint redistribution                | residual migration           |
| 163 | interp changed hedges     | mapping (\partial x/\partial q) changed  | bump/rebuild compare         |
| 164 | prove overfit             | unstable mapping                         | node move norms + checks     |
| 165 | surprise hike stress      | meeting jump propagation                 | bucket PV/DV01 by type       |

## “Trader-safe” default responses

| Situation                      | Safe default action                                  |
| ------------------------------ | ---------------------------------------------------- |
| Weird localized misfit         | suspect dates/schedules before tuning weights        |
| PV ok, risk insane             | run bump/rebuild stability and kink metrics          |
| Cross-currency front-end weird | check USD segmentation + FX spot lag + cut alignment |
| Basis behaving badly           | check stale quote overweight + basis architecture    |
| Holidays/weekends involved     | day-by-day accrual repricing is mandatory            |

If you want the next batch, I can do “trader asks for **new analytics**” (custom curve views, hedge decomposition by curve component, real-time residual attribution, and what metrics make traders actually trust the curve).

---
Fine. Trader wants **new analytics**. You want to sound like a VP strat who can ship something traders will actually use (instead of another dashboard they ignore). Continuing numbering from **Q165**.

---

## Q166) Trader: “Give me a **custom curve view** that matches how I think: meetings, turns, and the strip. Not your generic zero curve.”

### What I build (the view, not the math lecture)

A “front-end narrative” panel with 4 layers:

1. **Segmented ON forward path** (r(t)) with explicit markers:

* meeting effective dates
* turn windows (EOM/QE/YE)
* holiday clusters (long weekends)

2. **Window-averaged overlays**:

* meeting-month FF average bands (pre/post decomposition)
* IMM quarter forward averages (for the STIR strip)

3. **Local sensitivity heat**:

* show where today’s curve is actually “supported” by market quotes (a density map)

4. **Diff vs yesterday**:

* forward delta plot (not just DF delta) so traders see *where* it moved.

### Why traders like it

It maps directly to their mental model:

* “what’s priced for the next meeting”
* “where’s the turn premium”
* “what does the strip imply”

### Minimum outputs

* (r_{pre}), (r_{post}) for next meeting
* implied average for each futures month/IMM quarter
* turn premium magnitude and window

---

## Q167) Trader: “I need **hedge decomposition**: how much of my DV01 is discounting vs forwarding vs basis.”

### The decomposition (what it means)

For a trade PV:
$$
PV = PV(D,\ F^{(idx)},\ b,\ FX,\ldots)
$$
You want to split sensitivity into components:

* **Discounting DV01**: effect of bumping discount curve quotes and rebuilding
* **Projection DV01**: effect of bumping projection curve quotes and rebuilding
* **Basis DV01**: effect of bumping basis instruments (3s6s, OIS–IBOR, XCCY basis) and rebuilding
* (Optional) **FX forward / XCCY coupling risk** for multi-currency books

### How I compute it (desk-correct)

You cannot just “bump nodes.” You bump **market quotes** by bucket:

* Discount bucket: OIS, discount swaps, cash anchors
* Projection bucket: FRAs, IMM futures, index swaps
* Basis bucket: basis swaps (3s6s, OIS–IBOR), XCCY basis
* FX bucket: FX swaps/forwards

Then:

1. bump quote (q_i \to q_i+\epsilon)
2. rebuild full coupled curves
3. reprice trade
4. accumulate sensitivities

### What traders see

A table like:

* DV01 total
* DV01 split by curve component
* top contributing quotes by absolute sensitivity

### Why it builds trust

They stop hedging the wrong “curve thing.” They can say:
“I’m long discounting DV01, short 3M forwarding DV01, and I’m exposed to 3s6s basis around 2–5Y.”

---

## Q168) Trader: “Show me **real-time residual attribution**. When something misfits, tell me what moved and who is to blame.”

### What “residual attribution” means

Residual for instrument (i):
$$
r_i = \text{ModelPrice}_i(x) - \text{MarketPrice}_i
$$
Traders want to know:

* which instruments are currently failing,
* and whether it’s because of:

  * quote change
  * curve change
  * solver/constraint behavior

### What I implement

A streaming panel with:

1. **Residual by instrument** (normalized by bid/ask):
$$
   z_i = \frac{r_i}{\text{bidask}_i}
$$
2. **Attribution split**

* “Quote moved” vs “Curve moved” vs “Constraint changed”

3. **Top offenders list**

* show 10 largest (|z_i|) grouped by bucket

### The key trick

Track the last published curve (x_{t-1}) and current curve (x_t). Then for an instrument (i):

* price using old curve + new quote
* price using new curve + old quote
  This isolates whether the residual is driven by curve update or quote update.

### Why traders trust it

It turns “curve is wrong” into:
“this single stale basis quote is 3 bid/ask wide” or “constraints started binding after that print.”

---

## Q169) Trader: “I want a **who-moved-my-curve** report. One page. Every time the curve updates.”

### Deliverable

A deterministic “curve change digest” with:

1. **Top curve moves**

* max (\Delta f(t)) and where
* max (\Delta^2 f(t)) (kink metric) and where

2. **Top quote drivers**

* which 5 quotes contributed most to node movements (via linearized attribution or bump test)

3. **Constraint activity**

* which constraints became active/inactive

4. **Fit status**

* count of instruments outside tolerance (by type)

### Why this matters

It prevents gaslighting. The trader sees a causal chain:
“Curve moved because meeting-month future jumped and the meeting node rebalanced.”

---

## Q170) Trader: “Give me a **liquidity-aware curve confidence score**. I want to know when to distrust the curve.”

### The score is not magic

It’s a composite of diagnostics traders actually care about:

1. **Fit quality** (bid/ask normalized)

* percent of instruments within 1x bid/ask
* worst offenders

2. **Stability**

* average node move norm per 1bp quote bump (or proxy)
* constraint thrash indicator

3. **Smoothness sanity**

* kink metrics vs historical percentiles

4. **Data health**

* stale quote count
* cross-source disagreement

### Output

A single number (0–100) plus “why it’s low today” bullet points.

### Why traders trust it

Because it’s honest and tied to things they see:

* stale markets
* holiday weirdness
* constraint binding
* big outliers

---

## Q171) Trader: “Can you give me **scenario ladders**: meeting surprise, turn stress, and strip shift, with PV and risk impacts?”

### What I build

A ladder set aligned to desk realities:

1. **Meeting surprise**

* add a discrete jump to post-meeting ON forwards
* optionally ramp expectation over a week for gradual repricing

2. **Turn stress**

* widen turn premium window and magnitude (QE/YE)

3. **Strip shift**

* shift IMM futures in a coherent pattern (front contracts more, belly less, etc.)

Then rebuild and output:

* PV change
* DV01 bucket shifts
* basis/XCCY propagation

### Key feature

“Where does the inconsistency land?”
Show whether basis kinks, FX misfits, or local curve distortions absorb the shock.

---

## Q172) Trader: “I need a **hedge suggestion engine**: given my DV01 split, tell me what to trade.”

### What I will and won’t do

I won’t pretend there’s one best hedge. I will produce a ranked shortlist aligned to your curve components.

### Approach

1. Compute exposure vector by market quotes:
$$
   \frac{\partial PV}{\partial q_i}
$$
2. Map to hedge instruments with liquidity constraints:

* if you need discounting DV01: OIS swaps / liquid OIS tenors
* if you need projection DV01: FRAs/IMM futures / index swaps
* if you need basis DV01: basis swaps in matching maturities
* if XCCY: FX swaps / XCCY basis tenors

3. Provide top 5 hedges with:

* expected PV neutralization fraction
* residual exposures left
* sensitivity overlap warnings (don’t “hedge basis” with a swap)

### Trader output

A “hedge menu,” not one commandment.

---

## Q173) Trader: “Why do I trust **your curve** more than Bloomberg’s? Give me the arguments.”

### What traders actually accept

1. **Governance + determinism**

* same inputs → same curve
* logged overrides, clear precedence rules

2. **Risk stability**

* bump/rebuild is well-behaved
* no violent bucket migration

3. **Correct microstructure**

* meetings modeled as discrete events
* turns modeled explicitly
* holiday/weekend accrual handled correctly

4. **Cross-asset consistency**

* FX forwards and XCCY basis fit within tolerances *without* basis becoming a garbage collector

5. **Transparent diagnostics**

* residual attribution and confidence score

### What I won’t claim

“I’m right because I’m internal.” Traders hate that. They trust transparency, not branding.

---

## Q174) Trader: “Give me a **real-time ‘curve health’ dashboard** that I can glance at and decide if I can trade off it.”

### The dashboard must be glanceable

Four tiles, no fluff:

1. **Health score (0–100)** + top 3 reasons it moved
2. **Worst residuals** (bid/ask normalized), grouped by type
3. **Stability**: bump/rebuild norm + constraint thrash indicator
4. **Front-end structure**: next meeting jump + next turn premium

### Why this works

It answers the only trader question that matters:
“Can I rely on this curve right now?”

---

# Summary tables

## Analytics menu

|  Q# | Analytics request              | What I deliver             | Why traders care       |
| --: | ------------------------------ | -------------------------- | ---------------------- |
| 166 | custom curve view              | meeting/turn/strip overlay | matches mental model   |
| 167 | hedge decomposition            | DV01 split by component    | hedge the right thing  |
| 168 | real-time residual attribution | z-scores + cause split     | blame the right input  |
| 169 | who-moved-my-curve             | top moves + top drivers    | stops noise arguments  |
| 170 | confidence score               | 0–100 + reasons            | tells when to distrust |
| 171 | scenario ladders               | meeting/turn/strip shocks  | tradable stress views  |
| 172 | hedge suggestion engine        | ranked hedge list          | speeds hedge workflow  |
| 173 | why trust internal curve       | governance + stability     | credibility            |
| 174 | curve health dashboard         | 4-tile glance view         | practical usage        |

## Metrics that make traders actually trust the curve

| Metric                         | What it proves                 |
| ------------------------------ | ------------------------------ |
| Bid/ask-normalized residuals   | you’re not “overfitting” noise |
| Stability (bump/rebuild norms) | hedges won’t jump buckets      |
| Constraint activity report     | explains non-smooth risk       |
| Kink metrics vs history        | no fake curvature              |
| Out-of-sample repricing checks | not just fitting targets       |
| Cross-currency consistency     | USD doesn’t poison everything  |
| Deterministic rebuild          | reproducible marks             |

If you want to go even more desk-real, I can translate these into **exact data schemas**: what you store per instrument, what you publish per curve update, and how you compute “top quote drivers” efficiently (Jacobians vs finite bumps) without melting production.

---
Here you go: **currency-by-currency VP-style grilling**. Each market has its own “cute” conventions, liquidity pockets, and ways to embarrass you in front of traders.

I’ll continue numbering from your last set.

---

## USD (SOFR / legacy Fed Funds)

### Q175) “Walk me through the **USD discount curve** build today. What’s liquid, what’s legacy, and where do meeting dates bite?”

**Answer**

* **Discounting anchor** in USD is **SOFR OIS** (SOFR is the NY Fed’s overnight secured repo-based rate). ([newyorkfed.org][1])
* **Liquid short-end**: ON/TN/SN cash anchors + **SOFR futures** / OIS; **Fed Funds exposure still exists** (legacy products and some basis risk), but USD is basically a “single-rate SOFR world” for core discounting. ([CME Group][2])
* **Meeting dates (FOMC)** matter because the short-end curve is *not* smooth: the market prices discrete jumps, and you either:

  * put explicit meeting-date knots/steps, or
  * you smear the jump into fake curvature and your Jacobian becomes garbage.
* Practical desk nuance: the “right” representation is **piecewise forwards over event windows** (meeting, turn, quarter-end balance sheet effects), then smooth **inside** windows, not across them.

---

### Q176) “When would you use **Term SOFR** in curve-land, and what’s the trap if you mix it with OIS discounting?”

**Answer**

* **Term SOFR** is forward-looking, derived from SOFR derivatives markets, and is typically used for **cash products** (loans) under restricted conventions, not as your core discounting engine. ([CME Group][3])
* Desk trap: if you treat Term SOFR like a discount curve input (or you bootstrap discounting off it), you’re effectively mixing:

  * an **OIS/RFR compounded-in-arrears** world
  * with a **forward-looking term** construct (and potentially different credit/liquidity embedded assumptions depending on product).
* Correct approach: keep **SOFR OIS** as discounting; Term SOFR is a **separate projection/forecast curve** only when the product explicitly references it.

---

## EUR (EuroSTR / Euribor multi-rate)

### Q177) “Explain why EUR is still a **multi-rate** market and what that implies for curve architecture.”

**Answer**

* EUR commonly has **€STR OIS** for discounting and **EURIBOR** (3M/6M) for projection, meaning **basis curves are structural**, not optional. ([CME Group][4])
* So you often solve **at least**:

  * €STR discount curve
  * Euribor 3M projection curve
  * Euribor 6M projection curve
  * 3s6s basis and/or €STR–Euribor basis constraints (depending on your setup and instrument set)
* Liquidity nuance: you can trade and hedge **€STR futures** and **Euribor** and even **basis spread futures** in listed space, which affects what you treat as “hard” intraday. ([CME Group][4])

---

### Q178) “How do you stop the EUR system from ‘hiding’ Euribor/€STR inconsistency by bending the wrong curve?”

**Answer**

* The failure mode: you over-tighten fit on one bucket (say Euribor FRAs/futures) and the solver compensates by distorting either:

  * €STR discounting (bad), or
  * 3s6s basis shape (sometimes OK), or
  * the long-end swap curvature (usually terrible).
* Fix is architectural + numerical:

  1. **Explicit basis object** (don’t let one projection curve absorb basis)
  2. **Bid/ask-normalized residuals** and robust loss, so a single stale Euribor print can’t bully the whole system
  3. **Local smoothness** on the basis curve (so it can move, but not kink to “solve politics”)

---

## GBP (SONIA, heavy calendar pain)

### Q179) “What makes GBP curve-building uniquely annoying even in a ‘single-rate SONIA world’?”

**Answer**

* SONIA is an overnight unsecured benchmark published by the BoE; GBP short-end trades heavily in **SONIA futures** and OIS. ([Banque d'Angleterre][5])
* GBP’s special brand of pain: **calendar + compounding** around UK bank holidays (and holiday clusters).
* Classic bug: mis-handling weekend/holiday accrual application in compounded RFR OIS leads to:

  * weekly OIS misfit
  * fake “turn” effects
  * DV01 bucket snapping around holiday weeks
* What a VP strat answer should include: “I always validate day-by-day accrual around Easter/Christmas weeks and compare to a reference schedule.”

---

## CHF (SARON, turns and repo flavor)

### Q180) “How does CHF differ structurally from GBP/USD in the short end, and what instruments do you expect to be liquid?”

**Answer**

* CHF’s RFR is **SARON**, based on the Swiss repo market (secured). ([SIX][6])
* CHF listed liquidity exists: **3M SARON futures** on Eurex / SARON STIR products on major venues. ([Deutsche Börse Group][7])
* CHF nuance: **turns (especially year-end)** can be brutal and very localized. If you don’t model turn windows explicitly, the solver will shove the effect into:

  * SNB-meeting-shaped jumps, or
  * random curvature that detonates sensitivities.

---

### Q181) “What’s the operational nuance with SARON publication and what can it break?”

**Answer**

* SARON is published end-of-day; futures contracts reference average SARON over a period and settle cash. ([ice.com][8])
* If your curve build cutoffs / timestamps aren’t aligned (cash, OIS quotes, futures settlement assumptions), you’ll get:

  * persistent short-end residuals
  * apparent “model drift” intraday that is really just data cut mismatch

---

## JPY (TONA, long holiday land)

### Q182) “What are the two JPY-specific curve killers: **holiday structure** and **instrument ecosystem**?”

**Answer**

* JPY RFR is **TONA** (TONAR/TONA), published by BoJ; JPY OIS reference it. ([Cbonds][9])
* JPY has listed **3M TONA futures** (JPX/TFX), with reference quarters tied to IMM-style dates. ([JPX][10])
* Killer #1: **Golden Week / long holiday clusters**. If you mis-handle multi-day accrual windows, weekly/monthly instruments disagree.
* Killer #2: legacy/parallel indices can exist (TIBOR, legacy fallbacks). If you don’t isolate index conventions properly, you’ll “discover” fake basis.

---

### Q183) “How do you incorporate a BoJ meeting expectation if the short-end is constrained by futures averaging?”

**Answer**

* Use event segmentation: represent the ON forward as piecewise over:

  * pre-meeting window
  * post-meeting window
* Map futures to **averages of those segments** over the contract window.
* Without segmentation, you’ll smear the BoJ jump across time and blow up local risk.

---

## CAD (CORRA, post-CDOR reality)

### Q184) “What changed in CAD after **CDOR cessation**, and what does a modern CAD curve stack look like?”

**Answer**

* CDOR stopped being offered after **June 28, 2024**, with transition to **CORRA** (and term variants for some products). ([National Bank Capital Markets][11])
* CAD modern stack commonly centers on:

  * **CORRA OIS** for discounting
  * legacy handling for CDOR-referencing trades (fallback/converted trades, basis risk)
* A VP-level point: “CAD is not just a curve build. It’s a **transition governance** problem: you need deterministic mappings for legacy cashflows and hedges during/after conversion.”

---

## AUD (AONIA + BBSW duality)

### Q185) “Explain AUD: why do desks still care about **BBSW** if AONIA exists?”

**Answer**

* **AONIA** is an overnight cash benchmark; **BBSW** is a credit-based term benchmark (1M–6M) widely used in AUD derivatives and loans. ([TraditionData][12])
* So you often have:

  * AONIA OIS discount curve (collateral/funding)
  * BBSW 3M/6M projection curves
  * basis between OIS and BBSW (and between BBSW tenors)
* The “trick”: BBSW embeds a bank credit component; if you treat it like a pure RFR, your basis curve becomes your trash can.

---

## SEK (SWESTR vs STIBOR)

### Q186) “What’s the SEK nuance: why do you have to think about **SWESTR and STIBOR** together?”

**Answer**

* **SWESTR** is the Swedish transaction-based overnight reference rate published by the Riksbank. ([riksbank.se][13])
* **STIBOR** still exists as an interbank offered rate benchmark. ([swfbf.se][14])
* Desk implication: you may have a multi-rate setup depending on product set, with basis/transition logic similar in spirit to EUR.
* Clearing/liquidity nuance: OIS eligibility and liquidity pockets can be CCP-dependent. ([LSEG][15])

---

## NOK (NOWA + NIBOR, liquidity quirks)

### Q187) “NOK has NOWA and NIBOR. What is NOWA and why does NOK liquidity force you to be careful with regularization?”

**Answer**

* **NOWA** is the recommended alternative reference rate administered by Norges Bank (overnight). ([norges-bank.no][16])
* **NIBOR** remains important (unsecured term-like benchmark) and has its own conventions. ([nore-benchmarks.com][17])
* NOK liquidity can be patchy. That means:

  * you can’t blindly “force fit”
  * you need robust weighting, priors, and **stability-first** regularization
* A strat answer should explicitly mention: “In thin markets, the curve is partly a *statistical estimate* constrained by tradables.”

---

## SGD (SORA, post-transition)

### Q188) “What’s the SGD curve world now and what legacy does a desk still see?”

**Answer**

* **SORA** is Singapore’s overnight benchmark (MAS-published definition). ([mas.gov.sg][18])
* SIBOR has been replaced in transition programs (legacy still exists in back books / fallbacks). ([Investopedia][19])
* Desk curve stack:

  * SORA OIS discounting
  * legacy handling for SIBOR/SOR exposures and basis during transition tail
* Key nuance: ensure coherent treatment of legacy indices so you don’t get phantom basis risk.

---

## HKD (HIBOR + HONIA, peg-driven weirdness)

### Q189) “HKD is explicitly **multi-rate**. Explain the setup and what can go wrong.”

**Answer**

* HKMA says HIBOR continues; **HONIA** is an alternative with no plan to discontinue HIBOR. ([hkma.gov.hk][20])
* HKD derivatives can reference both HIBOR and HONIA (and product eligibility depends on venue/clearing). ([hkex.com.hk][21])
* Peg nuance: HKD liquidity swings via HKMA interventions can drive sudden HIBOR moves that look ‘non-economic’ if you forget the peg mechanics. (So your curve QC must be sensitive to liquidity regime shifts, not just smoothness.)

---

# Summary tables

## Per-currency cheat sheet (what you’re expected to know in an interview)

| Currency | RFR (discount anchor)                           | Common projection index           | Liquid building blocks you’re expected to cite             | “Nuance/trick” they test                                    |
| -------- | ----------------------------------------------- | --------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------- |
| USD      | SOFR ([newyorkfed.org][1])                      | SOFR / legacy Fed Funds           | SOFR OIS + SOFR futures ([CME Group][2])                   | FOMC segmentation + turns + mapping stability               |
| EUR      | €STR ([CME Group][4])                           | Euribor 3M/6M                     | €STR futures / Euribor + basis ([CME Group][4])            | Structural basis (multi-rate) and who absorbs inconsistency |
| GBP      | SONIA ([Banque d'Angleterre][5])                | SONIA                             | SONIA futures + OIS ([ice.com][22])                        | Calendar/holiday compounding and front-end stability        |
| CHF      | SARON ([SIX][6])                                | SARON                             | SARON futures (Eurex/ICE) ([Deutsche Börse Group][7])      | Year-end turn handling + repo-flavor data cuts              |
| JPY      | TONA ([Cbonds][9])                              | TONA (plus legacy)                | 3M TONA futures ([JPX][10])                                | Long holidays + IMM-style windows + meeting segmentation    |
| CAD      | CORRA ([bankofcanada.ca][23])                   | CORRA + legacy CDOR               | Post-CDOR governance ([National Bank Capital Markets][11]) | Transition architecture + legacy curve hygiene              |
| AUD      | AONIA ([TraditionData][12])                     | BBSW ([rba.gov.au][24])           | OIS + BBSW swaps/basis                                     | Dual world (RFR vs credit term) and basis meaning           |
| SEK      | SWESTR ([riksbank.se][13])                      | STIBOR ([swfbf.se][14])           | SWESTR OIS eligibility ([LSEG][15])                        | Multi-rate coexistence and liquidity pockets                |
| NOK      | NOWA ([norges-bank.no][16])                     | NIBOR ([nore-benchmarks.com][17]) | NOWA admin + NIBOR conventions ([norges-bank.no][25])      | Thin market: robust weighting + stability-first             |
| SGD      | SORA ([mas.gov.sg][18])                         | SORA + legacy SIBOR               | Transition completion context ([abs.org.sg][26])           | Legacy index isolation and basis clean-up                   |
| HKD      | HONIA alt / HIBOR continues ([hkma.gov.hk][20]) | HIBOR/HONIA ([hkex.com.hk][21])   | Venue/clearing eligibility                                 | Peg/liquidity regime shifts and multi-rate sanity           |

## Interview “tell” checklist (what makes your answer sound real)

| If you mention…                                | It signals you’ve actually built curves                   |
| ---------------------------------------------- | --------------------------------------------------------- |
| event segmentation (meeting/turn windows)      | you understand discrete jumps vs fake curvature           |
| bid/ask-normalized residuals + robust loss     | you know markets are inconsistent                         |
| schedule/holiday day-by-day accrual checks     | you’ve debugged real OIS pain                             |
| legacy transition governance (CAD/SGD/etc.)    | you’ve lived through benchmark reform reality             |
| cross-currency collateral leakage (USD anchor) | you understand multi-curve *systems*, not isolated curves |

If you want, I can turn this into an interview drill: I ask **one currency at a time**, you answer, and I push back with the most common desk-head objections until it’s bulletproof.



