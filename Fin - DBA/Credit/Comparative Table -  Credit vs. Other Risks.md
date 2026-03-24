| **Feature**          | **Interest Rate Risk (Duration)** | **Equity Risk (Delta)** | **Credit Risk (Jump/Gamma)**         |
| -------------------- | --------------------------------- | ----------------------- | ------------------------------------ |
| **Primary Driver**   | Yield curve shifts                | Price movements         | Spread widening + Event risk         |
| **Gamma Profile**    | Usually low/neutral               | Can be long or short    | **Structurally Short (Negative)**    |
| **Path Dependency**  | Generally continuous              | Continuous (mostly)     | Discontinuous (The "Jump")           |
| **Upside/Downside**  | Symmetric                         | Symmetric (for spot)    | Asymmetric (capped gain, huge loss)  |
| **Hedge Difficulty** | Easy (Futures/Swaps)              | Moderate                | Hard (Requires constant rebalancing) |

==*But why is gamma structurally negative for credit ?*==

It is structurally negative because of the **asymmetry of the payoff**. In credit, you are essentially "selling a disaster policy." The profile of a credit position (like a bond or a CDS) is mathematically identical to being **short an out-of-the-money (OTM) put option**, and short options always carry negative gamma.

Here is the step-by-step intuition, the formulaic breakdown, and the "why" behind this structural trap.

---

## 1. The Intuition: The "Collector" vs. The "Payer"

To understand why it's structural, look at the two possible states of a credit instrument:

- **State A (The "Normal" State):** The company is healthy. You collect a small, steady coupon (the spread). Your upside is strictly **capped** at the par value plus interest.
    
- **State B (The "Default" State):** The company hits a wall. You lose a massive, discrete chunk of principal (e.g., 60% of the bond's value). Your downside is **uncapped** until you hit the recovery floor.
    

In a normal equity position, if the stock goes up 10%, you gain. If it goes up 100%, you gain more. In credit, if the company’s health improves by 100%, **you still only get your coupon.** > **The Rule:** Because you cannot gain more than your fixed coupon, but you can lose almost everything in a "jump," your delta (sensitivity) must increase as things get worse and decrease as things get better. That change in delta is, by definition, **Negative Gamma.**

---

## 2. The Formula: The Probability Delta

Let $V$ be the value of a credit-sensitive bond, $R$ be the recovery rate, and $P_{def}$ be the probability of default. We can simplify the value of the bond as:

$$V = (1 - P_{def}) \times \text{Risk-Free Value} + P_{def} \times R$$

The **Delta** ($\Delta$) with respect to the probability of default is:

$$\frac{\partial V}{\partial P_{def}} = R - \text{Risk-Free Value}$$

The **Gamma** ($\Gamma$) is the rate of change of that delta. As $P_{def}$ increases (the company gets riskier), the market price $V$ doesn't drop linearly. It accelerates toward $R$.

In a standard "Merton-style" model, credit is viewed as a **short put option** on the company's assets.

- **Asset Value ($A$) > Debt ($D$):** You get paid in full (Flat line).
    
- **Asset Value ($A$) < Debt ($D$):** You own the remaining assets (Sloping line).
    

The "bend" in that payoff curve where the flat line meets the sloping line is where the **Negative Gamma** lives. Since most corporate bonds trade near par, they sit right on the "shoulder" of that curve, meaning any move toward default increases your risk exposure exponentially.

---

## 3. Worked Example: The "Gamma Trap"

Let’s look at a Portfolio Manager (PM) holding $\$100\text{M}$ of a "BB" rated bond.

1. **At Par ($100$):** The spread is $200\text{ bps}$. The "Spread Delta" (sensitivity) is relatively low. The PM feels safe.
    
2. **The Slide ($90$):** Bad earnings. Spread widens to $500\text{ bps}$. The bond is now "distressed."
    
    - **The Shift:** Because the bond is closer to the "Jump" point, every $1\text{ bp}$ move in spread now causes a much larger price drop than it did at par.
        
    - **The Gamma Effect:** The PM's exposure has grown. To maintain the same "risk," they would actually need to _sell_ part of the bond as it drops—but selling into a falling market is exactly what negative gamma forces you to do.
        
3. **The Jump ($40$):** Default occurs. The delta goes from "high" to "zero" instantly as the bond hits the recovery floor.
    

---

## 4. Why is it "Structural"?

It is structural because of the **Legal Priority of Claims**:

1. **Fixed Upside:** Debt holders are promised a fixed return. They do not share in the "moonshot" profits of a company.
    
2. **Binary Outcome:** A company is either "In Default" or "Not In Default." This binary nature creates a "cliff" in the pricing model.
    
3. **Mean Reversion vs. Total Loss:** While interest rates tend to stay within a range (allowing for positive convexity), credit spreads have a "zero bound" on the upside (they can't go much below $0\text{ bps}$) but no bound on the downside (they can go to "Default").
    

---

## Comparative Summary Table

|**Feature**|**Interest Rates (Treasuries)**|**Credit Spreads (Corporate Bonds)**|
|---|---|---|
|**Payoff Shape**|Convex (Curving Up)|Concave (Curving Down)|
|**Gamma Sign**|**Positive** (usually)|**Negative** (structural)|
|**Volatility Impact**|Higher vol can increase bond price|Higher vol always decreases bond price|
|**Exposure Trend**|Risk decreases as price falls|Risk increases as price falls|
|**Metaphor**|A bowl (ball stays in center)|An umbrella (ball rolls off the edge)|

---

## Further Generalizations

- **Negative Convexity in Mortgages:** This is a related concept. In MBS (Mortgage-Backed Securities), you have negative gamma because people refinance when rates drop. You lose your high-coupon asset right when it's most valuable.
    
- **The "Volatility Smile":** Because credit has built-in negative gamma, credit options (SCDS options) usually have a very steep "skew." The market charges a massive premium for protection against that "Jump," knowing that the delta will explode if the spread starts to move.
    