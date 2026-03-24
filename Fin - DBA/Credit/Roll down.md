
Let's dive into the mechanics of **Roll-Down**. This is the hidden engine of fixed-income and credit trading. While the premium is the visible cash flow you collect, roll-down is the "stealth" capital gain you earn simply because time passes.

Here is the step-by-step intuition, the mathematical framework, a minutely commented code example, and the updated carry formula.

---

## 1. The Intuition: Sliding Down the Curve

Imagine a hill that gets steeper the higher you climb. This represents a normal **Credit Spread Curve**.

- Investors demand more compensation (a higher spread) to lock their money away for 5 years compared to 1 year because more things can go wrong over a longer horizon.
    
- Therefore, the 5-year spread is usually higher than the 4-year spread.
    

**The Magic of Time:**

When you sell protection on a 5-Year CDS, you lock in that higher 5-year rate.

Exactly one year later, assuming the market hasn't panicked and the company hasn't defaulted, your contract is no longer a 5-year instrument. It is now a **4-year instrument**.

Because 4-year risk is priced lower (tighter spread) than 5-year risk, your contract has effectively "rolled down" the yield curve to a lower spread.

- As we established earlier, a tightening (dropping) spread creates a **Mark-to-Market (MTM) gain**.
    
- You get to claim a capital gain simply because the contract aged, assuming the overall market curve stayed static.
    

---

## 2. The Math: Updating the Core Formula

Let's redefine the formula from the slide to include this hidden tailwind.

Let:

- $S_5$ = The spread at inception (5-Year).
    
- $S_4$ = The spread of a 4-Year contract in today's market.
    
- $D_4$ = The duration of the contract at the end of Year 1 (roughly 4).
    
- $N$ = Notional amount.
    

**The Roll-Down MTM Gain** is calculated as the difference in spreads multiplied by the duration remaining:

$$\text{Roll-Down Gain} \approx D_4 \times (S_5 - S_4) \times \left( \frac{N}{10,000} \right)$$

Now we add this to your total carry equation:

$$\text{Total 1Y Carry} = \text{Premium Earned} + \text{Roll-Down Gain}$$

_(Note: We assume JTD is $0$ here to isolate the baseline expected return)._

---

## 4. The Interactive Roll-Down Explorer

To really cement this intuition, use this tool to see how steepening or flattening the credit curve impacts your total carry. If you make the 4-year spread _higher_ than the 5-year spread (an inverted curve), you will see roll-down turn into a penalty.

![[IMG-20260324200349490.png]]
![[IMG-20260324200406938.png]]


---

## 5. Comparative Summary: Sources of Fixed Income Return

|**Return Source**|**Driver**|**Realized or Unrealized?**|**Risk Profile / Market Environment**|
|---|---|---|---|
|**Premium / Coupon**|Time passing (contractual rate).|Realized (Cash paid out).|Safe, steady. Exists regardless of market curve shape.|
|**Roll-Down**|Time passing + A steep yield/credit curve.|Unrealized (MTM Capital Gain).|Vulnerable to curve flattening. Disappears if the curve is flat.|
|**General MTM Gain/Loss**|Macro shocks or company upgrades/downgrades.|Unrealized (Price fluctuation).|Highly volatile. This is where the "Jump-to-Default" negative gamma lives.|

---

## Further Generalizations

1. **Curve Inversion:** What happens if the credit curve inverts (e.g., the 1-year spread is $500\text{ bps}$ because default is imminent, but the 5-year is $200\text{ bps}$ because if they survive year 1, they'll be fine)? In this distressed scenario, roll-down becomes severely negative. Time passing actively destroys your MTM.
    
2. **The "Forward" Assumption:** The math above assumes the yield curve today looks exactly the same in exactly one year. This is called the "static curve assumption." In reality, curves shift daily, which is why actual trading requires hedging the curve shape.
    


# Worked example

Let's grab some scratch paper and work through the exact arithmetic manually. While the previous Python snippet handled the final output, seeing the raw numbers interact step-by-step is the absolute best way to make the mechanics stick.

Here is a fully integrated, manual walkthrough of a credit carry trade, calculating both the cash premium and the hidden roll-down gain.

---

### 1. The Intuition: The Gravity of the Curve

When you sell a 5-year Credit Default Swap (CDS), you are locking in a compensation rate for 5 years of risk. Because longer timeframes carry more uncertainty, 5-year risk pays a higher premium than 4-year risk.

As you hold the contract for a year, "gravity" pulls your contract down the curve. You originally priced it for 5 years of danger, but a year later, it only has 4 years of danger left. Assuming the market hasn't panicked, the market re-prices your contract at the safer, lower 4-year rate. That drop in the spread generates an instant capital gain for you.

### 2. The Setup: Market Conditions

Let's assume you execute the following trade today:

- **Action:** Sell Protection on Company X.
    
- **Notional Size:** **$20,000,000**
    
- **5-Year Spread (Today):** **200 bps** (**2.00%**)
    
- **4-Year Spread (Today):** **150 bps** (**1.50%**)
    

**The Core Assumption:** We are assuming a "static curve." This means we are betting that exactly one year from today, the market will look identical to how it looks right now.

---

### 3. Step-by-Step Calculation

#### Step A: Calculate the Premium Carry (The Cash)

This is the straightforward rent you collect for providing insurance. You are earning 200 basis points on a $20 million position over the course of the year.

- **Formula:** $\text{Annual Premium} = \text{Notional} \times \text{Inception Spread}$
    
- **Math:** **$20,000,000** * **0.02**
    
- **Result:** **$400,000** in hard cash collected.
    

#### Step B: Calculate the Roll-Down MTM (The Capital Gain)

Fast forward 12 months. Your 5-year contract is now a 4-year contract. Because we assume the curve remained static, the market now values your specific contract at the current 4-year rate of **150 bps**.

First, find the spread differential:

- **Inception Spread:** **200 bps**
    
- **New Market Spread:** **150 bps**
    
- **Spread Tightening:** **50 bps** (The spread dropped, which is good for the seller of protection).
    

Next, we need to figure out how much a 1 basis point move is worth in dollars for a 4-year contract. This is the **CS01** (Credit Spread 01).

- **Formula:** $\text{CS01} \approx \text{Duration} \times \left( \frac{\text{Notional}}{10,000} \right)$
    
- _Note: A 4-year CDS has a risk-duration of roughly 4.0._
    
- **Math:** 4.0 * (**$20,000,000** / **10,000**) = **$8,000** per basis point.
    

Finally, calculate the Mark-to-Market (MTM) gain:

- **Formula:** $\text{Roll-Down Gain} = \text{Spread Tightening} \times \text{CS01}$
    
- **Math:** **50 bps** * **$8,000/bp**
    
- **Result:** **$400,000** in unrealized capital gains.
    

#### Step C: Total Expected 1-Year Carry

Combine your cash flow with your capital gain.

- **Math:** **$400,000** (Premium) + **$400,000** (Roll-Down)
    
- **Total Return:** **$800,000**
    

In this scenario, exactly 50% of your total profit came purely from the passage of time pushing your contract down a steep yield curve.

---

### 5. Comparative Table: Component Breakdown

|**Component**|**Amount**|**% of Total Return**|**Source of Profit**|**Risk Factor**|
|---|---|---|---|---|
|**Premium Carry**|**$400,000**|50%|Contractual agreement (Spread).|Jump-to-Default (JTD). The company goes bankrupt and you lose principal.|
|**Roll-Down Carry**|**$400,000**|50%|The steepness of the credit curve.|Curve flattening. If the 4Y spread rises to meet the 5Y spread, this profit vanishes.|
|**Total 1Y Return**|**$800,000**|100%|Combined Time + Curve Mechanics.|Macro shocks causing the entire curve to shift upward (MTM wipeout).|

---

### Further Generalizations

1. **Hedging the Curve (Curve Trades):** You might look at the example above and think, _"I love the roll-down, but what if the whole market panics and the 4-year spread jumps to 300 bps? I lose everything."_ To isolate just the roll-down, traders execute **Curve Trades**. They will sell 5-year protection (going long risk) and simultaneously _buy_ 4-year protection (going short risk). This neutralizes their exposure to the market panicking, allowing them to purely harvest the difference in the steepness of the curve.
    
2. **Inverted Curves:** If a company is in severe distress, the curve inverts. The 1-year spread might be **1,000 bps**, while the 5-year is **500 bps** (because if they survive year 1, they are likely saved). In this scenario, roll-down is a massive _penalty_. Time passing pushes you into higher and higher spreads, destroying your Mark-to-Market value daily.
    

Would you like to step through the math of how a Long 5Y / Short 4Y curve hedge is constructed to eliminate market risk?