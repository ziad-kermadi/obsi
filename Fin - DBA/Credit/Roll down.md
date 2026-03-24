
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
    

**Would you like to explore how to hedge against the curve shifting or flattening, so you can isolate and protect that roll-down gain?**