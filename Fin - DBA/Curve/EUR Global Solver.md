![[IMG-20250711180631248.jpg|1050]]
Great — let's go through the content in the screenshot **slowly and thoroughly**, breaking it down **line by line**, clarifying all terminology, roles of components, and why it's structured that way.

---

# 🔍 Overview: What’s This Document About?

It’s about a **curve calibration framework enhancement** called **Global Solver (GS)** used for EUR swaps. It solves issues that arise when pricing EUR instruments, particularly in the context of **OIS-based curves** being constructed using **6M Euribor-driven market data**.

---

## ✅ Section-by-Section Explanation

---

## 🔧 **What problems does it solve?**

### 📌 **Problem 1: EUR OIS Curve is Based on Euribor Market Liquidity**

> “EUR curve is an OIS backbone curve but most of the EUR liquidity is within 6M Euribor swaps”

Let’s decode this:

* A **curve** like the EUR discounting or projection curve can be built using different swap instruments.
* Ideally, for an **OIS curve**, we’d use OIS (EONIA/ESTR) instruments directly.
* However, the EUR market is **more liquid in 6M Euribor swaps**, not OIS swaps.

So, desks often **derive OIS curves using Euribor swap quotes**, and convert between them using **basis spreads** and **conversion factors**.

> “they are driving the OIS swaps using feeds from Euribor swaps with arithmetic conversion factors between bases”

This means:

* You observe Euribor swaps (liquid),
* Apply basis adjustments (e.g., Euribor-ESTR basis),
* Infer the OIS curve from them.

🧠 **Issue**: This dependency creates **fragile setups** when basis spreads shift — hard to maintain consistently.

---

### 📌 **Problem 2: PV Discrepancies**

> “Pricing PV does not match risking PV, particularly in the US curve”

This means:

* Sometimes the **present value (PV)** computed during pricing differs from the **PV used for risk or sensitivity calculations (bump-and-revalue)**.
* This mismatch can happen when curves used in pricing and risk are **not aligned or constructed differently**.

🔧 Global Solver is introduced to solve both issues.

---

## ⚙️ **What is Global Solver (GS)?**

> “GS can be configured to solve these problems”

It's a framework that:

* Separates curve building into **template vs. target**,
* And applies **centralized curve calibration logic** to ensure consistency and robustness.

---

## 🔌 **GS enabled in config key**:

> `GS enabled in CNF#DT_IR@EUR#LiveModel=GS_Price`

This means:

* In your system config, the **LiveModel** for EUR in that particular setup is set to use `GS_Price`.

So when curves are built in that environment, **Global Solver logic is triggered**.

---

## 🏗️ **We build 2 curves**:

### 1. **TemplateModel** (Skeleton)

* Think of this as a **placeholder curve**:

  * It defines which instruments to use.
  * But not the actual market data or levels.
* It's the blueprint.

### 2. **TargetModel** (Input)

* This contains:

  * The actual **market instruments**, quotes, and settings you want to calibrate the final curve from.
* It's the concrete data.

---

## 🔁 **GS Flow – How It Works Step-by-Step**

1. You specify:

```
GS_Price.TemplateModel=Dummy&DS2
GS_Price.TargetModel=Target&DS2
```

2. The solver:

   * Uses the instruments in `TargetModel`.
   * Fits those instruments **onto the skeleton structure defined by `TemplateModel`**.

3. It then calls:

   * A function (probably in C++ or Python) like `DBA_calibrateCurve(...)`
   * This is a system function that tries to **find the set of curve node values** that match market inputs.

> “If there are conflicting instruments, it will still fail to build”

So if you have inconsistent instruments in your TargetModel (e.g. bad inputs, stale quotes, etc), calibration will still break — but the GS flow is more robust in organizing and debugging the process.

---

## 🔄 Clarification on TemplateCurve vs Final Curve

> “TemplateCurve is not treated as a curve itself but is built as a curve”

Translation:

* The **TemplateCurve** is not used for pricing.
* But it's constructed to **define how the real curve should look**, in terms of structure and interpolation.
* It's like a **blueprint of the curve's node layout**.

---

## 🧠 Why Use This Structure?

### Without Global Solver:

* Curves are built in a more ad-hoc way, where:

  * Curve structure
  * Market instruments
  * Solver
  * and Risk setup
    are **not separated**.

This creates inconsistencies:

* If you swap instruments,
* Or if the pricing desk builds a curve differently from the risk engine,
* You get **PV mismatches** and maintenance nightmares.

---

### With Global Solver:

* Clear **separation of roles**:

  | Component      | Role                                 |
  | -------------- | ------------------------------------ |
  | TemplateModel  | Blueprint (what structure to follow) |
  | TargetModel    | Market instruments and data          |
  | CalibrateCurve | Solver (finds node levels)           |
  | DBA            | Executes calibration and validation  |

This ensures consistency across pricing, risk, and models.

---

## 📊 Summary Table

| Concept             | Meaning                                                                |
| ------------------- | ---------------------------------------------------------------------- |
| EUR OIS curve issue | Built using 6M Euribor swaps, with complex basis conversions           |
| PV mismatch issue   | Risk PVs don’t match pricing PVs due to inconsistent curve structures  |
| Global Solver       | Framework to fix curve consistency and robustness                      |
| TemplateModel       | Blueprint curve structure (skeleton)                                   |
| TargetModel         | Actual market instruments and data used in calibration                 |
| GS\_Price config    | Sets curve to use GS logic for calibration                             |
| DBA calibration     | Final step that solves curve using instruments in the structured setup |

---

![[IMG-20250711180636967.jpg|1075]]
![[IMG-20250711180642585.jpg|1075]]
![[IMG-20250711180649396.jpg]]
![[IMG-20250711180655246.jpg]]
![[IMG-20250711180700484.jpg]]
