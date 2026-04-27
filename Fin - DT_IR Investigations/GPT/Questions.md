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


# 3. ![[IMG-20260427181817550.png]]

This table, titled "The PnL Map," is a fundamental framework used by interest rate traders and risk managers to decompose how a portfolio's value changes when the market moves.

In fixed income, we do not look at price changes in isolation. Instead, we use a **Taylor Series Expansion** to approximate the change in Price ($P$) based on changes in Yield ($y$), Volatility ($\sigma$), and other factors.

---

## 1. Level Risk (The Parallel Shift)

**The Concept:** This assumes the entire yield curve moves up or down by the same amount. If the 2-year, 5-year, and 30-year rates all rise by 10 basis points (bps), that is a parallel shift.

**The Math:**

The first-order derivative of Price with respect to Yield is **Duration**. Traders use **DV01** (Dollar Value of an 01), which is the dollar change in value for a 1 bp (0.01%) move in rates.

$$\Delta P \approx -\text{DV01} \times \Delta \text{Yield (in bps)}$$

- **Worked Example:** You hold a bond with a DV01 of $\$1,000$. If the entire curve shifts up by **5 bps**, your PnL is:
    
    $$\Delta P = -(\$1,000) \times 5 = -\$5,000$$
    

---

## 2. Slope Risk (The Curve Twist)

**The Concept:** The curve doesn't move in parallel; the "spread" between different maturities changes. A **steepener** occurs when long-term rates rise faster than short-term rates (or short-term rates fall faster).

**The Math:**

To calculate this, we use **Key-Rate DV01 (KRD)**. This measures sensitivity to a shift in one specific point on the curve while keeping others constant.

$$\Delta \text{PnL}_{\text{Slope}} = \sum_{i=1}^{n} -(\text{KRDV01}_i \times \Delta y_i)$$

- **Worked Example (The DV01-Neutral Steepener):** You want to bet on the 2s10s spread widening without being exposed to the overall "Level" of rates.
    
    1. Long 2Y Bond: $+ \$500$ DV01.
        
    2. Short 10Y Bond: $- \$500$ DV01.
        
    3. **Market Move:** 2Y yield drops 10bps; 10Y yield rises 10bps (A twist).
        
        $$\Delta \text{PnL} = (-500 \times -10) + (500 \times 10) = +5,000 + 5,000 = +\$10,000$$
        
        _Note: If the whole curve moved up 10bps in parallel, your PnL would be zero because the legs offset._
        

---

## 3. Curvature Risk (The Butterfly/Convexity)

**The Concept:** The "belly" of the curve (e.g., 5-year) moves differently than the "wings" (2-year and 10-year). This is often managed via a **Butterfly trade**.

**The Math:**

In the image, "Convexity" is listed. Mathematically, convexity is the second-order derivative ($P''$). It represents the "benefit" of the curve's shape.

$$\Delta \text{PnL} \approx \frac{1}{2} \times \text{Dollar Convexity} \times (\Delta y)^2$$

- **Worked Example:** A "Body" (5Y) vs "Wings" (2Y/10Y) trade. If the 5Y rate drops significantly while the 2Y and 10Y stay still, the curve becomes more "humped." You gain or lose based on the change in the curvature coefficient ($\beta_2$ from your previous equation).
    

---

## 4. Volatility Risk (Vega)

**The Concept:** For products with embedded options (like Swaptions or Mortgage-Backed Securities), the price changes even if interest rates stay still, simply because the _uncertainty_ about future rates changes.

**The Math:**

Vega is the sensitivity of the price to a 1% change in implied volatility.

$$\Delta \text{PnL} = \text{Vega} \times \Delta \sigma$$

- **Worked Example:** You own a Swaption with a Vega of $\$2,000$. If implied volatility increases by **3 points**, your PnL is:
    
    $$\Delta P = \$2,000 \times 3 = +\$6,000$$
    

---

## 5. Correlation & Basis Risk

### Correlation (Cross-Gamma)

This is the risk that two different rates move together or apart.

- **Math:** $\Delta \text{PnL} \approx \frac{\partial^2 P}{\partial y_1 \partial y_2} \times \Delta y_1 \Delta y_2$
    
- **Example:** You are long the 10Y Treasury and short the 10Y Libor Swap. If they stop moving in sync, you have "Cross-Gamma" or correlation risk.
    

### Basis Risk

The risk that two similar but not identical instruments diverge.

- **Math:** $\Delta \text{PnL} = -\text{Basis DV01} \times \Delta(\text{Spread})$
    
- **Example:** The Libor-OIS spread. If you are hedged in one but exposed to the other, a widening of the spread between them hits your PnL.
    

---

## The Intuition: The "Instrument Cluster"

Imagine you are flying a plane.

- **Level** is your altitude.
    
- **Slope** is the pitch (nose up or down).
    
- **Curvature** is the structural integrity or "flex" of the wings.
    

The image’s warning—**"You Can’t Just Hedge Parallel"**—is critical. If you only hedge your "altitude" (Level), you could still crash if the "nose" of the plane pitches down violently (Slope risk). A professional trader must ensure that their "gages" for all six dimensions are near zero, or at least sized to their specific convictions.

---

## Synthetic Risk Summary Table

|**Risk Type**|**Exposure Metric**|**Second-Order Metric**|**Hedging Tool**|
|---|---|---|---|
|**Level**|DV01 / Delta|Convexity / Gamma|Futures, Swaps|
|**Slope**|Key-Rate DV01|-|Curve Spread Trades|
|**Curvature**|Butterfly Weight|-|2s5s10s Butterflies|
|**Volatility**|Vega|Vanna / Volga|Options, Swaptions|
|**Basis**|Basis DV01|-|Basis Swaps|

---

## Further Generalizations

In modern electronic trading, we often use **Principal Component Analysis (PCA)** to refine these. Instead of arbitrarily choosing "Slope" as 2s10s, PCA looks at the last 10 years of data and identifies the "true" slope factor that explains the most variance.

**A Correction to the Image Note:**

The image says: _"A 2s10s steepener has zero parallel DV01."_ **Strictly speaking, this is only true if the trade is intentionally "DV01 Neutralized."** If you just buy $\$10$M of 2Y and sell $\$10$M of 10Y, you actually have a **massive** Level (DV01) risk because the 10Y bond is much more sensitive to rates than the 2Y bond. To achieve "zero parallel DV01," you must sell much less of the 10Y than you bought of the 2Y.

**Related Question:** If you are "Long Convexity," do you want the market to be volatile or stable? (Hint: Think about the $(\Delta y)^2$ term—it's always positive!)



# 4. ![[IMG-20260427182855175.png]]

This image highlights the critical difference between **Total Risk** (Level) and **Component Risk** (Shape). In the world of rates trading, if you only look at your total DV01, you are essentially driving a car while only looking at the altimeter—you know if you're going up or down a mountain, but you have no idea if you're about to drive off a cliff.

---

## 1. DV01: The "Hammer" (Parallel Shift)

**Total DV01** measures your sensitivity to the entire curve moving in unison. It treats the yield curve as a single, rigid board.

### The Math

$$\Delta PnL_{parallel} = DV01_{total} \times \Delta y_{parallel}$$

- **$DV01_{total}$**: The sum of all DV01s across every maturity you own.
    
- **$\Delta y_{parallel}$**: The number of basis points (bps) the whole curve moves up or down.
    

### The Intuition

Imagine a long table. If you lift the whole table up by 1 inch, every point on the table moves up by 1 inch. This is a parallel shift. If you are "Long" the table (positive DV01), you profit when interest rates fall (the table goes down) because bond prices move inversely to yields.

---

## 2. Key-Rate DV01: The "Scalpel" (Surgical Hedge)

**Key-Rate DV01 (KR DV01)** recognizes that the yield curve is not a rigid board; it's a flexible string. One part can move up while another moves down.

### The Math

$$\Delta PnL_{KR} = \sum_{i=1}^{N} (DV01_{i} \times \Delta y_{i})$$

- **$DV01_{i}$**: Your risk at a specific "bucket" or "node" (e.g., 2Y, 5Y, 10Y).
    
- **$\Delta y_{i}$**: The specific yield change at that maturity.
    

### The Intuition

Instead of lifting the whole table, you are now pushing down on one specific spot with your finger. The areas closest to your finger move the most, while the ends of the table might stay still. This allows you to see how a "twist" in the curve affects your money, even if the "average" move is zero.

---

## 3. Breaking Down the Image's Example

The image provides a classic "2s10s Steepener" example. Let's look at the **exact math**, and I will be firm here: the image's example math is a bit shorthand with its signs, so let's clarify the logic.

**The Portfolio:**

- **Long 10Y Bond:** $DV01 = +\$900$ (You make $900 if the 10Y rate falls 1bp).
    
- **Short 2Y Bond:** $DV01 = -\$450$ (You lose $450 if the 2Y rate falls 1bp).
    
- **Net Portfolio DV01:** $+450$ (If the whole curve falls 1bp, you make $450).
    

### Scenario A: A Parallel Shift (+10bp)

If the whole curve moves up by 10bp:

$$\Delta PnL = -(Net DV01 \times \Delta y) = -(450 \times 10) = -\$4,500$$

You lost $4,500. This is your "Level" risk.

### Scenario B: The Curve Twist (2Y -10bp, 10Y +10bp)

This is where the "Parallel Hedge" fails. The 2Y yield _falls_ 10bp and the 10Y yield _rises_ 10bp.

1. **PnL on 10Y:** Since you are Long, and rates rose, you lose money.
    
    $$PnL_{10Y} = -(900 \times 10) = -\$9,000$$
    
2. **PnL on 2Y:** Since you are Short, and rates fell (making the bond more expensive to buy back), you lose money.
    
    $$PnL_{2Y} = -(-450 \times -10) = -\$4,500$$
    
3. **Total PnL:**
    
    $$\text{Total} = -9,000 + (-4,500) = -\$13,500$$
    

**The Takeaway:** In a parallel move, you only lost $4.5k. In a twist, you lost **$13.5k**. This is why the image says "Parallel hedge is just the sum of key-rate hedges"—if you don't look at the individual pieces, you are blind to the "Shape Risk."

---

## 4. Synthetic Comparison Table

|**Feature**|**Total DV01 (Parallel)**|**Key-Rate DV01 (KR)**|
|---|---|---|
|**Philosophy**|The curve moves as one unit.|The curve is a collection of segments.|
|**Visibility**|Shows total exposure to "The Market."|Shows exposure to "The Spread" or "The Shape."|
|**Risk Type**|Level Risk|Slope / Curvature Risk|
|**Analogy**|Lifting a rigid plank of wood.|Plucking a guitar string at a specific fret.|
|**Common Use**|Hedging against Fed rate hikes.|Trading a recession (curve flattening/steepening).|

---

## 5. Further Generalizations & Related Concepts

### The "Sum of the Parts" Rule

The image notes: _"Parallel hedge is just the sum of key-rate hedges."_ This is mathematically true because of the **superposition principle**. If you hedge every single key rate to zero, your total DV01 will automatically be zero. However, the reverse is **not** true. You can have a total DV01 of zero while having massive, offsetting Key-Rate risks. This is called a **Curve Neutral** trade.

### Related Concept: Principal Component Analysis (PCA)

In professional trading, we often group these Key Rates into three "Factors" (as we discussed in the first image):

1. **PC1 (Level):** 90% of curve movement.
    
2. **PC2 (Slope):** 7% of curve movement.
    
3. **PC3 (Curvature):** 2% of curve movement.
    

By managing Key-Rate DV01, you are essentially managing your exposure to PC2 and PC3.

**Further Question:** If you wanted to build a trade that makes money if the 5-year rate "humps" up while the 2-year and 10-year stay flat, how would you distribute your Key-Rate DV01s? (Hint: You would need to be "Short" the 5Y and "Long" the others to be DV01 neutral).