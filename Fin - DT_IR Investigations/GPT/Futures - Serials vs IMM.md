You sit on a rates desk and somehow the market still expects humans to remember naming conventions invented decades ago by committees. Fine. Let’s clean this up properly, beginner-friendly, desk-real, and without fairy tales.

I’ll go **what they are → why they exist → how they are actually used in curve building**.

---

## 1. Big picture first (before we drown in acronyms)

When you build a **money-market / short-end yield curve**, you need **liquid, observable rates** at known future dates.

Futures give you:

* Standardized contracts
* High liquidity
* Implied forward rates

But futures come in **different date structures**, which is where **IMM vs serial** shows up.

---

## 2. What a “rates future” really represents

Ignore the contract noise for a second.

A short-term rates future is basically:

> “The market’s expectation of the **average short rate** over a future period.”

Mathematically (simplified):
$$
\text{Futures price} = 100 - \text{Implied rate}
$$
So:

* Price = 95.00 → Implied rate ≈ 5.00%

The **only thing that differs** across futures types is:

* **When the averaging period starts**
* **When it ends**

That’s it. Everything else is branding.

---

## 3. IMM futures (the main characters)

### What “IMM” means

IMM dates are **standard quarterly dates**:

* March
* June
* September
* December

Specifically:

* **3rd Wednesday** of those months

So:

* IMM Mar
* IMM Jun
* IMM Sep
* IMM Dec

These dates are globally synchronized across:

* Futures
* Swaps
* Options
* Clearing

Which is why desks love them.

---

### Examples of IMM futures

Depending on currency:

| Currency | Contract      |
| -------- | ------------- |
| USD      | SOFR futures  |
| EUR      | €STR futures  |
| GBP      | SONIA futures |
| JPY      | TONA futures  |

Each contract typically covers **3 months** starting on an IMM date.

---

### Why IMM futures dominate curve building

* Massive liquidity
* Clean quarterly grid
* Align perfectly with:

  * Swap payment schedules
  * Option expiries
  * Risk buckets (3M, 6M, 9M…)

**Translation**: your DV01 doesn’t do weird things.

---

### How IMM futures map to the curve

Example:

| Contract | Period covered |
| -------- | -------------- |
| Mar IMM  | Mar → Jun      |
| Jun IMM  | Jun → Sep      |
| Sep IMM  | Sep → Dec      |

So they directly give you:

* Forward rates over **standard 3M intervals**

Which is perfect for bootstrapping.

---

## 4. Serial futures (the supporting cast)

### What “serial” means

Serial futures are:

* **Non-IMM start dates**
* Typically **monthly**, not quarterly

Think:

* Jan
* Feb
* Apr
* May
* Jul
* Aug
* Oct
* Nov

Basically: all the months IMM skipped.

---

### Why serial futures exist at all

Because reality is annoying:

* Central bank meetings don’t respect IMM dates
* Front-end risk is very sensitive month-to-month
* Traders want finer resolution near today

Serials allow you to:

* Fill the **gaps between IMM contracts**
* Get **better shape** in the very front end (0–1Y)

---

### Liquidity reality check (important)

* IMM futures: **very liquid**
* Serial futures:

  * Liquid in the first few months
  * Rapidly become garbage further out

So:

* You **trust** serials close to spot
* You **fade** them further out

No religion here, just survival.

---

## 5. How curve builders actually use them (this is the part nobody says out loud)

### Typical desk setup

1. **Very front end**

   * O/N fixings
   * Short deposits
   * Possibly 1–2 serial futures

2. **Front to belly**

   * Mix of:

     * Serial futures (if liquid)
     * IMM futures (dominant)

3. **Beyond ~1–2Y**

   * Swaps take over
   * Futures become less relevant

---

### Practical example (USD SOFR curve)

| Maturity | Instrument          |
| -------- | ------------------- |
| O/N – 1M | Fixings / OIS       |
| 1M – 3M  | Serial SOFR futures |
| 3M – 2Y  | IMM SOFR futures    |
| 2Y+      | OIS swaps           |

Serials smooth the curve **before** the IMM grid locks in.

---

## 6. Why quants and traders argue about serials

Because serials:

* Can distort the curve if over-weighted
* Sometimes reflect **positioning**, not economics
* Have worse convexity effects

So desks often:

* Down-weight serials
* Use them only as **shape guidance**
* Let IMM futures anchor the level

If someone tells you “we fully trust serials out to 18 months”, they are lying or new.

---

## 7. IMM vs Serial: clean comparison table

| Feature          | IMM futures                 | Serial futures            |
| ---------------- | --------------------------- | ------------------------- |
| Start dates      | Quarterly (Mar/Jun/Sep/Dec) | Monthly (non-IMM months)  |
| Liquidity        | Extremely high              | Good only very near term  |
| Tenor            | Usually 3M                  | Usually 1M or short stubs |
| Curve role       | Backbone of short-end curve | Front-end refinement      |
| Risk alignment   | Excellent                   | Messy                     |
| Desk trust level | High                        | Conditional               |

---

## 8. Mental model you should keep

* **IMM futures = skeleton**
* **Serial futures = cartilage**
* **Swaps = muscles**
* **Fixings = blood**

Remove any one of them and the curve starts walking funny.

---

## 9. One-line desk summary (the kind traders actually use)

> “Serials help shape the front end, IMMs anchor the curve.”

That sentence alone explains 80% of curve-building arguments.

---

## 10. Ultra-compact recap table

| Question                 | Answer                               |
| ------------------------ | ------------------------------------ |
| What is IMM?             | Quarterly standardized futures dates |
| What is serial?          | Monthly non-IMM futures              |
| Why both?                | Liquidity vs granularity             |
| Which do we trust more?  | IMM                                  |
| Where do serials matter? | Very front end                       |
| Why arguments?           | Weighting and distortion risk        |

If you want next:

* IMM vs **stub risk**
* Convexity adjustment differences
* How futures translate into forwards step-by-step
* Why futures-based curves differ from deposit-based ones

I’ll complain, but I’ll do it.
