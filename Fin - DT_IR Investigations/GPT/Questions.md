# 1. **Theoretically, 1m,6m,12m forward rates can be calculated from a 3m LIBOR forecasting curve , but these calculated rates will be different from the real**

This is an excellent question that cuts straight to one of the most fundamental shifts in modern quantitative finance.

You are completely correct in your underlying assumption: calculating rates theoretically versus observing them in the market yields different results. However, to be totally straight up with you from the start—since we are in 2026—LIBOR is officially dead and has been replaced by Risk-Free Rates (RFRs) like SOFR. That said, the exact same principle you're asking about still applies today to any "Term" rates (like Term SOFR).

Let's break down exactly what this sentence means step-by-step, look at the math, and explore why the market behaves this way.

### Step 1: The "Theoretical" Calculation (The Single-Curve Illusion)

In the old days (pre-2008 financial crisis), the market operated on a **Single-Curve Framework**. The assumption was that money is money, and the only thing that mattered was the time value of money.

Theoretically, if you have a curve that accurately forecasts 3-month (3m) rates into the future, you should be able to mathematically construct a 6-month (6m) rate. How? By assuming that locking in a 6-month loan is financially equivalent to taking out a 3-month loan today, and immediately locking in a forward agreement to renew that loan for another 3 months when the first one expires.

The formula for compounding two adjacent periods looks like this:

$$1 + (R_{6m} \times \tau_{6m}) = [1 + (R_{0,3} \times \tau_{0,3})] \times [1 + (F_{3,6} \times \tau_{3,6})]$$

_(Where $R$ is the spot rate, $F$ is the forward rate, and $\tau$ is the time fraction in years, typically Actual/360)._

### Step 2: The Market Reality (The Multi-Curve World)

If you do the math above, you get an _implied_ 6m rate. But if you look at the trading screens, the _actual_ quoted 6m rate in the market is almost always **higher** than your calculated rate. Conversely, the market-quoted 1m rate will be slightly **lower** than what a 3m curve would imply.

Why? Because of **Tenor Basis Spreads**. The theoretical calculation assumes there is no friction or added risk between different time horizons. The market disagrees.


# 2. ![[IMG-20260426182403568.png]]

This text describes the three primary components used to analyze and describe movements in a yield curve (the relationship between interest rates and their time to maturity). In finance, specifically fixed income, we don't just say "interest rates went up"; we describe *how* the shape of the curve changed.

The mathematical foundation here is a **polynomial proxy** for the yield curve. While professionals often use more complex models like Nelson-Siegel, this quadratic equation is the simplest way to visualize the three dimensions of curve movement.

---

## 1. The Mathematical Framework

The equation provided is:
$$y(t) = \beta_0 + \beta_1 t + \beta_2 t^2$$

Where:
* $y(t)$ is the yield for a specific maturity $t$.
* $t$ is the time to maturity (e.g., 2 years, 10 years).
* $\beta_0, \beta_1, \beta_2$ are the coefficients representing the three dimensions.

### The Three Dimensions

| Dimension | Coefficient | Financial Name | Visual Effect |
| :--- | :--- | :--- | :--- |
| **Level** | $\beta_0$ | Parallel Shift | The entire curve moves up or down by the same amount. |
| **Slope** | $\beta_1$ | Twist / Tilt | The curve rotates, changing the spread between short and long rates. |
| **Curvature** | $\beta_2$ | Butterfly / Bend | The middle of the curve (the "belly") moves relative to the ends (the "wings"). |

---

## 2. Step-by-Step Breakdown

### Level ($\beta_0$): The Center
The $\beta_0$ term is a constant. It does not depend on time ($t$).
* **The Math:** If you increase $\beta_0$ by 0.50, every single point on the curve—from the 1-month rate to the 30-year rate—increases by exactly 0.50.
* **Intuition:** This represents the general "height" of interest rates, often driven by central bank policy or long-term inflation expectations.

### Slope ($\beta_1$): The Twist
The $\beta_1 t$ term is linear. Its impact grows as time ($t$) increases.
* **The Math:** If $\beta_1$ is positive, the curve moves upward as $t$ increases (a "normal" upward-sloping curve). If $\beta_1$ increases further, the 10-year rate rises much more than the 2-year rate.
* **Intuition:** This is a "twist." A **Steepener** means the gap between long-term and short-term rates is widening. A **Flattener** means that gap is shrinking.

### Curvature ($\beta_2$): The Bend
The $\beta_2 t^2$ term is quadratic. It creates a parabola.
* **The Math:** Because it is squared, it has a disproportionate effect on the "middle" maturities depending on how you scale $t$. It creates a "hump" or a "trough" in the middle of the curve.
* **Intuition:** A "positive butterfly" move means the belly of the curve becomes more curved (lower rates in the middle relative to the ends).


---

## 3. Worked Example: "A steepener with level down 20bp"

Let's translate the quote from the image into a practical scenario. Imagine the current 2-year yield is **3.0%** and the 10-year yield is **4.0%**.

1.  **"Level down 20bp":**
    * Subtract 20 basis points (0.20%) from everything.
    * New 2Y = 2.80%
    * New 10Y = 3.80%
2.  **"A steepener":**
    * The slope must increase. Let's say the 10Y rate rises relative to the 2Y, or the 2Y falls more than the 10Y.
    * If the curve steeps by another 10bp, we might see:
    * Final 2Y = **2.75%**
    * Final 10Y = **3.85%**

**Result:** The entire market is lower (level down), but long-term debt is now much more expensive relative to short-term debt than it was before (steepener).

---

## 4. The Intuition

Think of the yield curve like a **flexible wooden plank** held up by three hands:

1.  **Hand 1 (Level):** Lifts the whole plank straight up or down.
2.  **Hand 2 (Slope):** Holds the left end still and pushes the right end up or down, tilting it.
3.  **Hand 3 (Curvature):** Pushes down on the middle of the plank while the ends stay put, causing it to bow or bend.

Traders use these dimensions because they are **uncorrelated**. Usually, a change in the "Level" doesn't automatically mean the "Slope" will change. By breaking the curve into these three numbers, a complex movement involving dozens of different bonds can be summarized in a single sentence.

---

## 5. Comparative Summary Table

| Feature | Level | Slope | Curvature |
| :--- | :--- | :--- | :--- |
| **Equation Term** | $\beta_0$ | $\beta_1 t$ | $\beta_2 t^2$ |
| **Market Terminology** | Parallel Shift | Steepening / Flattening | Butterfly / Belly move |
| **Primary Driver** | Inflation / Fed Funds Rate | Economic growth expectations | Supply/Demand in specific maturities |
| **Visual Shape** | Vertical displacement | Change in angle | Change in "hump" |

---

## Further Generalizations & Related Concepts

While the quadratic formula $y(t) = \beta_0 + \beta_1 t + \beta_2 t^2$ is helpful for intuition, it is technically flawed for long-term modeling because a $t^2$ term will eventually go to infinity, which interest rates do not do. 

**The Nelson-Siegel Model**
To fix this, economists use exponential decay:
$$y(t) = \beta_0 + \beta_1 \left( \frac{1 - e^{-\lambda t}}{\lambda t} \right) + \beta_2 \left( \frac{1 - e^{-\lambda t}}{\lambda t} - e^{-\lambda t} \right)$$
* **$\beta_0$** is still the Level (as $t \to \infty$, the other terms disappear).
* **$\beta_1$** is still the Slope (it starts at 1 and decays to 0).
* **$\beta_2$** is still Curvature (it starts at 0, humps up in the middle, and goes back to 0).

**Principal Component Analysis (PCA)**
If you take 30 years of interest rate data and run a statistical analysis (PCA) to see what moves the curve, the math consistently finds that:
1.  **80-90%** of movement is the **Level**.
2.  **5-10%** of movement is the **Slope**.
3.  **1-3%** of movement is the **Curvature**.

**Further Question for Exploration:** If the Fed raises short-term rates but the market expects a recession (lowering long-term rates), which of these three dimensions would you expect to see the largest change in? (Hint: It’s the Slope—specifically a "bear flattener").

