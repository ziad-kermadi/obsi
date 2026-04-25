- **Par Spread** = Coupon for which the CDS has NPV=0, assuming a piece-wise constant hazard curve (considered in conjunction with all other par spreads); _also called Running Spread_
- **Quoted Spread** = Coupon for which the CDS has NPV=0, assuming a flat hazard curve (considered in isolation to all other quoted spreads); _also called Conventional Spread_
- **Quoted Upfront** = Value that matches the NPV of a CDS with a fixed coupon (500p in this example), assuming a flat hazard curve (considered in isolation to all other spreads)

### 1. The "Big Bang" of 2009

Before 2009, CDS were like car insurance: every company had a unique "Par" price. If IBM was 72bps and Apple was 45bps, you signed a contract for exactly 72 or 45. This was a nightmare for clearing and offsetting trades because every contract was "special."

The industry changed the rules (The Big Bang) to make CDS look more like Bonds or Options. They decided:

- **Fixed Coupons:** Almost all CDS now only pay **100bps** (for safe companies) or **500bps** (for risky ones).
    
- **Upfront Cash:** Since the "real" risk isn't exactly 100 or 500, the difference is paid in a lump sum of cash on day one.
    

---

### 2. Why Par is a "Dummy" Value

The **Par Spread** is the coupon that _would_ make the upfront payment zero. But since you are **forced** to use a 100 or 500 coupon, the Par Spread is a purely theoretical price.

On the desk, it’s considered a "dummy" or "informational" value because:

1. **Non-Tradable:** You can't call a dealer and say "I want to hit your Par Spread." They will tell you the Quoted Spread and the Upfront.
    
2. **No Cash Flow:** No one actually pays the Par Spread. The actual money moving is the Fixed Coupon + the Upfront.
    
3. **Model Dependent:** To see a Par Spread, you have to "back-calculate" it using a model. Depending on which "hidden" data points (like that 3-month point we discussed) your model uses, your Par Spread might be different from the guy sitting next to you.
    

### 3. The Intuition: The MSRP vs. The Sale Price

Think of the Par Spread like the **MSRP (Manufacturer's Suggested Retail Price)** on a car window.

- **Par Spread (MSRP):** The "theoretical" fair price of the car ($30,000).
    
- **Fixed Coupon (Down Payment):** The standard monthly payment the dealership demands ($500/month).
    
- **Upfront (The Cash Adjustment):** Because the car's actual value isn't exactly $30,000 today, you either pay extra cash at the start or get a rebate to make the math work.
    

The traders only care about the **Quoted Spread** (how they communicate the "vibe" of the price) and the **Upfront** (the actual cash they have to wire). The Par Spread is just a helpful reference to see where the "break-even" point used to be.

---

### Comparison of Utility

|**Value**|**Status on the Desk**|**Why?**|
|---|---|---|
|**Quoted Spread**|**The Language**|It’s how traders talk. "IBM is trading at 85."|
|**Upfront Cash**|**The Reality**|This is the actual money that leaves the bank account.|
|**Par Spread**|**The Ghost**|A theoretical calculation of what the price _would_ be if coupons weren't standardized.|

### Worked Example: The "Dummy" Math

Imagine a company's "True Risk" (Par) is **120bps**.

- The market standard coupon is **100bps**.
    
- Because 100 < 120, the buyer is "underpaying" every year.
    
- To fix this, the buyer pays an **Upfront** amount (e.g., 1.5% of the total deal) at the start.
    

If you change your model slightly, the **Par** might move to **122bps**, but the **Quoted Spread** and **Upfront** are what the brokers are actually showing on their screens. You look at the Par just to get a sense of "Is this company getting riskier?" but you don't trade it.


To understand the math behind switching between **Quoted Spread**, **Par Spread**, and **Upfront**, we have to look at the **Risky PV01 (RPV01)**. This is the "bridge" that connects a spread (an annual rate) to a cash value (an upfront payment).

### 1. The Core Intuition: The "Present Value" Bridge

In a CDS, there are two "legs":

1. **Premium Leg:** The stream of payments the buyer makes.
    
2. **Protection Leg:** The one-time payment the seller makes if the company defaults.
    

The **RPV01** is the present value of a **1 basis point (0.01%)** payment made every year until the contract ends or the company defaults. Think of it as the "multiplier" that turns an annual rate into a lump sum.

---

### 2. The Conversion Math

To move between these three values, we use the following relationships. Let $S_{par}$ be the Par Spread, $S_{fixed}$ be the standard coupon (usually 100 or 500 bps), and $S_{quoted}$ be the conventional quote.

#### **A. Quoted Spread $\rightarrow$ Upfront**

This is the most common "on-the-desk" calculation. The Upfront is the difference between what the market says the risk is ($S_{quoted}$) and what you are actually paying ($S_{fixed}$), multiplied by the RPV01.

$$Upfront = (S_{quoted} - S_{fixed}) \times RPV01(S_{quoted})$$

> **Important Note:** In this specific conversion, the $RPV01$ is calculated using the **ISDA Standard Model**, which assumes a **flat hazard rate** derived from $S_{quoted}$ itself.

#### **B. Upfront $\rightarrow$ Quoted Spread**

This is just the algebra of the above. If you know the cash that changed hands, you solve for the $S_{quoted}$ that makes the equation balance. Because $S_{quoted}$ is hidden inside the $RPV01$ calculation (which involves an integral/summation), this is usually solved using an iterative numerical method (like Newton-Raphson).

#### **C. Par Spread $\leftrightarrow$ Upfront**

The **Par Spread** ($S_{par}$) is the spread where the Upfront would be zero. Therefore:

$$S_{par} \times RPV01_{curve} = Protection\_Leg_{curve}$$

To find the Upfront using the Par Spread, you compare the fixed coupon to the Par Spread:

$$Upfront = (S_{par} - S_{fixed}) \times RPV01_{curve}$$

---

### 3. Worked Example: 5Y CDS on "Company X"

Let's say Company X has a **Quoted Spread of 250 bps**. The standard coupon is **100 bps**. The RPV01 for a 5-year contract at this risk level is roughly **4.2**.

1. **Identify the Gap:** The market says the risk is 250, but you only pay 100 every year. The "gap" is **150 bps** ($250 - 100$).
    
2. **Calculate Upfront:**
    
    $$Upfront = (0.0250 - 0.0100) \times 4.2 = 0.015 \times 4.2 = 0.063$$
    
    This means you pay **6.3%** of the contract amount upfront.
    
3. **The "Dummy" Par:** If you build a full curve with 1Y, 3Y, and 5Y points and find that the 5Y point needs to be **254 bps** to make the whole curve stay smooth, then **254** is your **Par Spread**. It’s close to 250, but it accounts for the "shape" of the risk over time.
    

---

### 4. Synthetic Comparison Table

|**Metric**|**Calculation Basis**|**Why they differ**|
|---|---|---|
|**Quoted Spread**|**ISDA Standard Model**|Uses a **flat** hazard rate. Ignores other maturities.|
|**Par Spread**|**Full Bootstrap**|Uses a **piece-wise** hazard curve. Influenced by 1Y, 3Y, etc.|
|**Upfront**|**Cash Value**|The actual "Delta" between market risk and the fixed coupon.|

### Why they look different in the Stack Exchange post

The "Spread Diff" the user saw was essentially:

$$\Delta = S_{par} - S_{quoted}$$

This difference exists because the **Par Spread** "sees" the 3-month risk bump, while the **Quoted Spread** model closes its eyes and pretends the 6-month risk is a perfectly straight line from day zero.

