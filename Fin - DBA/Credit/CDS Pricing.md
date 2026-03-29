# CDS Pricing: A Trading Desk Walkthrough

Credit Default Swaps are among the most analytically rich instruments on a trading desk. Let me walk through pricing from first principles — exactly as a quant or structurer would approach it.

---

## 1. What you're actually pricing

A CDS is a bilateral contract where the **protection buyer** pays a fixed periodic spread (the "coupon") in exchange for a contingent payment if a reference entity defaults. Pricing means finding the **par spread** — the coupon that makes the contract worth zero at inception — or marking an existing trade with a running coupon.

There are **two legs** to value separately:---
![[IMG-20260329144833654.png]]

## 2. The inputs — what the desk actually uses

Before any calculation, the desk assembles these market inputs:

**Market observables:**

- **CDS spread curve** — par spreads at 1Y, 2Y, 3Y, 5Y, 7Y, 10Y for the reference entity (pulled from Bloomberg CDSW or an internal pricing service)
- **Risk-free discount curve** — OIS (SOFR, €STR) swapped curve, stripped from swap quotes. On a trading desk this is almost always SOFR-discounted post-UMR
- **Recovery rate (R)** — conventionally assumed **40%** for senior unsecured corporate, 25% for subordinated, varies by sector. Sovereigns often use 25%. This is a _fixed input_, not solved for

**Trade economics:**

- Notional
- Trade date, effective date, maturity
- Running coupon — standardised at **100bps** or **500bps** since the 2009 Big Bang (SNAC convention)
- Day count: ACT/360 for USD, ACT/365 for GBP
- Payment frequency: quarterly in arrears (IMM dates: 20 Mar / 20 Jun / 20 Sep / 20 Dec)

---

## 3. Building the survival curve — the heart of CDS pricing

The **survival probability curve** Q(t) is the implied probability the reference entity has _not_ defaulted by time t. You bootstrap it from quoted par spreads.

The key relationship linking a par spread S(T) to the hazard rate h(t):

$$\text{Fee leg PV} = \text{Protection leg PV}$$

Under the simplest piecewise-constant hazard rate model:

- **Survival probability:** Q(t) = exp(−∫₀ᵗ h(u) du)
- **Default probability density:** q(t) = −dQ/dt = h(t) · Q(t)
- Between two pillar dates tᵢ₋₁ and tᵢ: h is constant at hᵢ, so Q(tᵢ) = Q(tᵢ₋₁) · exp(−hᵢ · Δtᵢ)

**Bootstrap procedure (done numerically, pillar by pillar):**

For each tenor node T₁ < T₂ < ... < Tₙ, solve for hₙ such that the CDS struck at S(Tₙ) is worth zero — given survival probabilities already solved at all prior nodes. This is a one-dimensional root-find (Newton or bisection) at each step.---

![[survival_curve_bootstrap.html]]
## 4. Valuing the fee leg (premium leg)

The protection buyer pays the running coupon S on the notional, accruing over each period, _conditional on no prior default_. The present value of the fee leg is:

**PV(Fee Leg) = S · Σᵢ Δtᵢ · DF(tᵢ) · Q(tᵢ)**

Where:

- Δtᵢ = day-count fraction for period i (ACT/360)
- DF(tᵢ) = risk-free discount factor at payment date tᵢ
- Q(tᵢ) = survival probability to tᵢ

**Critical detail — accrued on default.** If the reference entity defaults _mid-period_, the protection buyer owes the accrued premium from the last payment date to the default date. The desk adds an **accrued fee correction** term:

**PV(Accrued) = S · Σᵢ ∫ₜᵢ₋₁ᵗⁱ (t − tᵢ₋₁) · DF(t) · q(t) dt**

This integral is approximated in closed form by mid-period approximation: ≈ S · Δtᵢ/2 · DF(midᵢ) · [Q(tᵢ₋₁) − Q(tᵢ)]

---

## 5. Valuing the protection leg (contingent leg)

The protection seller pays (1 − R) · Notional at the moment of default, if default occurs before maturity. Its PV is the expectation of this discounted payment:

**PV(Protection Leg) = (1 − R) · ∫₀ᵀ DF(t) · q(t) dt**

Where q(t) = −dQ/dt is the risk-neutral default probability density.

Numerically, this integral is evaluated as a sum over small time steps:

**PV(Protection Leg) ≈ (1 − R) · Σⱼ DF(tⱼ) · [Q(tⱼ₋₁) − Q(tⱼ)]**

The convention on the desk is to integrate over **daily steps** or at minimum over monthly steps between coupon dates.

---

## 6. Putting it together — the full pricing engine---

![[cds_full_pricer (1).html]]

## 7. From par spread to upfront — the SNAC convention

Post-2009 Big Bang, CDS trade at **fixed coupons** (100bps for IG, 500bps for HY) with an **upfront payment** to compensate for the difference between the coupon and the market par spread. This is how DTCC clears everything.

**Upfront = (Par spread − Coupon) × RPV01 × Notional**

Where **RPV01** (also called the Risky Annuity or Risky Duration) is the fee leg PV per unit of spread:

**RPV01 = Σᵢ Δtᵢ · DF(tᵢ) · Q(tᵢ)**

- If market spread > coupon → protection buyer **receives** upfront (the contract is "in the money" for the buyer — they're paying less coupon than the market rate implies, so they compensate the seller at trade inception)
- If market spread < coupon → protection buyer **pays** upfront

---

## 8. Risk metrics the desk monitors daily

These are the Greeks that populate the risk ladder:

**CS01 (Credit Spread 01):** P&L impact of a 1bp parallel shift in the CDS spread curve. CS01 ≈ −Notional × RPV01 × 0.0001

For a $10M 5Y HY position, CS01 is roughly $4,000–$4,500 per bp.

**IR DV01:** Sensitivity to a 1bp parallel shift in the risk-free discount curve. Much smaller than CS01; CDS pricing is driven by credit risk, not rates. Typically 5–10% of CS01 in magnitude.

**Rec01:** Sensitivity to a 1% change in recovery rate. If R rises by 1%, protection leg PV falls (less is paid out on default). Rec01 = −∂MTM/∂R = ∫₀ᵀ DF(t) q(t) dt

**Jump-to-default (JTD):** The P&L if the reference entity defaults _right now_:

- Protection buyer receives: (1 − R) × Notional
- Protection buyer loses: accrued coupon since last payment
- Net JTD ≈ +(1 − R) × N − accrued

---

## 9. What actually happens at the desk — workflow---

![[IMG-20260329145005351.png]]

## 10. Subtleties that separate desk practice from textbook

**Convexity and duration.** The RPV01 falls as spreads widen because higher hazard rates reduce survival probabilities faster. This means CDS have **negative convexity for the protection buyer** — gains from spread widening come at a decelerating rate. Desks hedge this when running large directional books.

**Curve trades.** Flat hazard rate is unrealistic. Upward-sloping spread curves (front short, back long) imply near-term stability with long-run uncertainty. The desk runs **steepener/flattener** trades (e.g. buy 5Y protection, sell 10Y protection) to express views on curve shape without taking a directional credit bet.

**Full upfront vs running spread — ISDA SNAC.** Since 2009, all investment grade trades settle with a 100bps coupon, HY at 500bps. The upfront is literally wired at T+1. This is crucial for clearing through ICE or LCH, which require a standard coupon.

**Accrual on default.** The final coupon period accrues even on default event. This causes a non-trivial "stub" payment that affects the MTM, particularly for long-dated trades near coupon payment dates.

**IMM date roll.** CDS mature on the next quarterly IMM date after the stated tenor. A "5Y" trade done on 15 March 2025 matures on 20 March 2030, not on 15 March 2030. This changes the actual coverage period.

**Counterparty risk and XVA.** A sold protection position on a financial entity creates wrong-way risk — the counterparty's credit may be correlated with the reference entity. The desk's XVA desk adds a CVA overlay, which is often non-trivial for CDS on sovereigns or banks.

**Index vs single-name.** CDX.NA.IG (125 names) and iTraxx Europe (125 names) are traded at tighter bid-offer spreads and higher liquidity than single-name. The intrinsic vs index basis is actively traded. The index spread ≈ average of constituent spreads but deviates in stress — this basis is the "skew."

---

This is the complete analytical framework that sits behind every CDS trade: bootstrap the hazard curve from market spreads, compute the two legs by integrating over survival probabilities and discount factors, convert to upfront using the RPV01, and continuously re-mark as spreads move. Everything else — index rolls, structured credit tranches, CDS options — is built on this foundation.