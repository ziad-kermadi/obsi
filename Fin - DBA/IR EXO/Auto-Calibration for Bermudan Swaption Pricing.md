![[IMG-20260128210256503.jpeg]]
![[IMG-20260128210424715.jpeg]]
![[IMG-20260128210435008.jpeg]]
![[IMG-20260128210447624.jpeg]]]![[IMG-20260128210457126.jpeg]]
![[IMG-20260128210506133.jpeg]]
![[IMG-20260128210515369.jpeg]]
![[IMG-20260128210524002.jpeg]]
![[IMG-20260128210534623.jpeg]]
![[IMG-20260128210543728.jpeg]]
![[IMG-20260128210553231.jpeg]]
c![[IMG-20260128210620328.jpeg]]
![[IMG-20260128210627697.jpeg]]
![[IMG-20260128210635829.jpeg]]
![[IMG-20260128210642715.jpeg]]
![[IMG-20260128210651064.jpeg]]


# How Bermudian Swaptions Are Priced: The DBAnalytics / Cheyette Approach

These screenshots are from an internal Confluence training page describing a specific **auto-calibration framework** used in practice. Here's what they show end-to-end:

---

## 1. Trade Setup: The SECURITY Object

The first step is configuring the trade itself. The desk uses two possible SECURITY types:

**ASWAPTION** (simpler, standard Bermudan par swaptions):

|Field|What it does|
|---|---|
|`ExerciseStart / ExerciseEnd / ExerciseInterval`|Defines the discrete exercise date schedule|
|`OptionType`|CALL (receiver), PUT (payer), CANPAYF (right to switch existing swap)|
|`NotificationDays`|How far ahead the holder must notify|
|`FixedRate / FixedFrequency / FloatFrequency`|Defines the underlying swap economics|
|`FixedBasis / FloatBasis`|Day count conventions|

**CMSOPT** (more flexible — handles complex cashflows, caps/floors, compound options):

- Has two columns for each field (one per leg), with explicit `ExerciseStart`, `ExerciseEnd`, `ExerciseInterval` within the cashflow schedule
- `OptionPosition`: LONG = desk holds the option, SHORT = counterparty holds it
- `OptionType`: ENTER (exercise enters the swap) or CANCEL (exercise cancels it)
- `AbsoluteCap / AbsoluteFloor` must be **disabled** (set to ±9.99) for auto-calibration to work

---

## 2. The Model: Cheyette (not Hull-White)

This desk uses the **Cheyette model** rather than plain Hull-White. The distinction matters:

- SABR is fine for **European swaptions** (terminal distribution only), but it is not a term-structure model — it cannot describe how rates evolve dynamically over time across multiple exercise dates
- The Cheyette model is a **one-factor HJM framework** with **local volatility**, described by two state variables x and y:

```
dx(t) = [y(t) - κ(t)x(t)] dt + σ(t, x, y) dW(t)
dy(t) = [σ(t, x, y)² - 2κ(t)y(t)] dt
```

with `x(0) = 0, y(0) = 0`

- The short rate is: `r(t) = f(0,t) + x(t)` — i.e., it fits the initial forward curve exactly by construction
- The conditional zero-coupon bond price has a **closed-form expression**, which is what makes backward induction tractable:

```
P(t,T) = P(0,T)/P(0,t) · exp(-G(t,T)·x(t) - ½·G(t,T)²·y(t))
```

- `κ` is the **mean reversion** — a key parameter that controls the term structure of volatility
- `σ(t, x, y)` is the **local vol function** — this is what gets calibrated

The Cheyette model is Markovian (state at time t depends only on x(t), y(t)), which means you can price on a **PDE grid** (DBL) rather than full Monte Carlo.

---

## 3. Market Data Objects: DBMVOL and DBOPTVOL

The calibration requires two distinct vol objects:

**DBMVOL** (the Cheyette term-structure vol to be calibrated):

- `VolName = DBMVOL`, `VolMode = MR`, `DiffusionType = YIELD`
- `ModelType = CHEYETTE`
- Contains a `MeanRev` subtable with the κ value (e.g., 0.008952)
- This is the object that gets **written to** by the auto-calibration procedure

**DBOPTVOL** (the SABR vanilla swaption vol surface — market input):

- Contains the `CMSMAT` matrix of ATM swaption implied vols
- Contains `SABRSTOCHVOL` matrix for the smile
- `ModelType = CEV` (or AFSABR — arbitrage-free SABR is the default)
- This is the **market data source** — it stays fixed during risk calculations, ensuring Greeks are consistent with the vanilla book

---

## 4. The Auto-Calibration Procedure (3 Steps)

This is the heart of the framework. Instead of calibrating the model offline and storing parameters, this is done **on-the-fly at pricing time**:

**Step 1 — Identify hedging European swaptions** Either extracted automatically from the Bermudan trade structure (co-terminal swaptions), or specified manually via an `EXTRAINFO` object with a `EuropeanSwaptionGrid` subtable.

**Step 2 — Calibrate the Cheyette local vol to those targets** The local vol function `σ(t, x, y)` is parameterised as **SKEWPARAMETRIC-7**:

```
σ(t,r) = [(β-γ)/2 - (β+γ)/2 · tanh(u)]² + α²·(1 - tanh(u)²)
```

where `u = (r - f(0,t)) / (scale × √t)`

The three time-dependent parameters α(t), β(t), γ(t) are calibrated to match the SABR vol surface at three strikes per exercise date. Mean reversion κ is calibrated separately by matching the ratio of vols of co-terminal swaptions.

Key settings in the `EXTRAINFO` object:

- `ResultType = AutoCalibration`
- `VolType = SKEWPARAMETRIC-7`
- `DeltaType = ATMDELTA`
- `MRCalibMethod`: controls how mean reversion is determined

**Step 3 — Price the Bermudan on the PDE grid (DBL)** With calibrated parameters, DBL (a numerical PDE engine supporting up to 3 factors) runs backward induction through the Cheyette state space, applying the optimal exercise condition at each exercise date:

```
V(t_i, x) = max [ ContinuationValue(t_i, x) , SwapNPV(t_i, x) ]
```

The swap NPV at each node is computed analytically using the closed-form bond price formula above.

---

## 5. Why Auto-Calibration Solves a Key Risk Problem

A critical insight from the screenshots: when risk (Greeks) are calculated, **the vanilla vol surface is held fixed**. This means:

- **Vega** from the Bermudan flows back into the same European swaption instruments used for calibration
- The desk can **offset Bermudan vega directly against European swaption vega** — they live in the same space
- If you used a pre-calibrated static DBMVOL, the Greeks would be in "term structure space" and you'd need an expensive mode-hedge projection step to map back to vanilla swaption space

---

## 6. The Bermudan Tax

The screenshots also cover a practical market reality: **Bermudan swaptions trade cheaper in the market than the model price** obtained by calibrating to European swaptions. This persistent difference cannot be explained by model improvements.

Two methods to handle it:

**BermTax EXTRAINFO** (direct price adjustment):

- `BermTaxMethod = 1` (Recursive Tax): rescales the continuation premium at each exercise date — more control over strike dependence, but not arbitrage-free
- `BermTaxMethod = 2` (Flat Tax): applies the rescaling once, not recursively — simpler and arbitrage-free

**MR BermTax** (model-level adjustment):

- More elegant: instead of adjusting the price, **shift the mean reversion κ** by values in a `BermMeanRevOffsetMatrix` (indexed by no-call period vs. exercise frequency)
- A negative κ shift cheapens the Bermudan while keeping European prices unchanged — arbitrage-free by construction
- The matrix shown has shifts of −0.05 across most tenors

---

## Summary of the Full Stack

```
Trade (ASWAPTION or CMSOPT SECURITY)
         ↓
EXTRAINFO (AutoCalibration settings: VolType, MRCalibMethod, BermTax)
         ↓
DBOPTVOL (SABR market vol surface — fixed, never touched by calibration)
         ↓
Auto-calibration: fit DBMVOL (Cheyette local vol + mean reversion)
    to co-terminal European swaption targets from DBOPTVOL
         ↓
DBL PDE engine: backward induction on Cheyette state grid
    → optimal exercise at each date
    → Bermudan price
         ↓
Greeks: bump-and-reprice with vol surface held fixed
    → Vegas land in European swaption space → directly hedgeable
         ↓
Bermudan Tax adjustment (if applicable)
```

The key insight of this whole framework is that by doing calibration **inside** the pricing call rather than outside it, the desk gets Greeks that are automatically consistent with how they hedge — which is the thing that actually matters on a trading desk.