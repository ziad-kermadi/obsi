When a Credit Default Swap (CDS) is sitting on a professional trading desk, the "Risk Duration" we discussed is just the tip of the iceberg. To manage a book, a trader decomposes the contract into a series of sensitivities (the "Greeks" of credit) to understand how every possible market wiggle—from a tiny interest rate shift to a total corporate collapse—will hit their PnL.

Here is the granular breakdown of every primary risk sensitivity, how they are calculated, and the intuition behind them.

---

## 1. CS01 (Credit Spread 01)

This is the most important sensitivity. It measures the change in the Mark-to-Market (MTM) value for a **1 basis point (0.01%)** parallel shift in the credit spread curve.

- **Calculation:**
    
    $$\text{CS01} = \text{MTM}(s + 1\text{bp}) - \text{MTM}(s)$$
    
    _Where $s$ is the current market spread._
    
- **The Intuition:** It tells the trader, "If the market gets 1bp more worried about this company, how many dollars do I lose?" For a standard 5Y CDS with a $10M notional and a Risky Duration of 4.5, the CS01 is approximately **$4,500**.
    
- **Usage:** Used to size the position and determine how many offsetting contracts are needed to hedge against general market moves.
    

---

## 2. JTD (Jump-to-Default)

While CS01 handles small "wiggles," JTD handles the "cliff." It measures the immediate PnL impact if the reference entity files for bankruptcy **instantly**.

- **Calculation:**
    
    $$\text{JTD} = \text{Notional} \times (1 - \text{Recovery Rate}) - \text{MTM}$$
    
    _The MTM is subtracted because if you already have a large unrealized loss, the "jump" to the final recovery value is smaller._
    
- **The Intuition:** If you sold protection, you are "Long Risk." If the company defaults, you pay out the par value and receive the recovery (e.g., 40%). If your bond was worth $9M yesterday and is worth $4M today after default, your JTD loss is $5M.
    
- **Usage:** Regulators and risk managers set "JTD Limits" to ensure a single bankruptcy doesn't blow up the entire desk.
    

---

## 3. IR01 (Interest Rate 01 / DV01)

Even though a CDS is a credit instrument, it has interest rate risk. Because the premiums are paid in the future, their Present Value (PV) changes when LIBOR/SOFR rates move.

- **Calculation:**
    
    $$\text{IR01} = \text{MTM}(r + 1\text{bp}) - \text{MTM}(r)$$
    
    _Where $r$ is the risk-free discount curve._
    
- **The Intuition:** If interest rates rise, the "Present Value" of the future premiums you expect to receive drops. Therefore, if you sold protection (and are receiving premiums), a rise in rates hurts your MTM.
    
- **Usage:** Traders "hedge out" this risk by trading interest rate swaps or Treasury futures so they are only betting on the credit, not the Fed.
    

---

## 4. Credit Gamma (Convexity)

This measures how much the **CS01 changes** as the spread moves. As we discussed, credit has structural **negative gamma**.

- **Calculation:**
    
    $$\text{Credit Gamma} = \frac{\partial(\text{CS01})}{\partial s} \approx \frac{\text{MTM}(s+1) - 2\text{MTM}(s) + \text{MTM}(s-1)}{1^2}$$
    
- **The Intuition:** If you are "short gamma," as spreads widen, your CS01 (your sensitivity) actually increases. You get "longer" the risk as the price is falling. This is why credit sell-offs often accelerate; traders are forced to sell more to maintain their risk limits.
    
- **Usage:** Used to understand the "acceleration" of losses during a crisis.
    

---

## 5. Curve Risk (Tenor-Specific CS01)

A company doesn't have one "spread." It has a curve (1Y, 3Y, 5Y, 10Y). Curve risk measures the sensitivity to a move in one specific point on that curve while others stay still.

- **Calculation:**
    
    $$\text{5Y Curve Risk} = \text{MTM}(\text{5Y Spread} + 1\text{bp}) - \text{MTM}(\text{All other tenors constant})$$
    
- **The Intuition:** The 5Y spread might widen while the 1Y spread tightens (a "steepening"). If you only look at a parallel CS01, you miss this.
    
- **Usage:** Traders use this to execute "Curve Trades" (e.g., Buy 3Y protection, Sell 5Y protection) to bet on the shape of the company's future, not just its current health.
    

---

## 6. Recovery Sensitivity (Rec01)

The MTM of a CDS assumes a standard recovery rate (usually 40%). But what if the market decides the company’s assets are actually only worth 20%?

- **Calculation:**
    
    $$\text{Rec01} = \text{MTM}(\text{Recovery} + 1\%) - \text{MTM}(\text{Recovery})$$
    
- **The Intuition:** This matters most for **distressed** names. If a company is healthy, a change in recovery doesn't change the price much. If a company is 1 day from default, the recovery rate is the _only_ thing that matters.
    
- **Usage:** Essential for "Distressed Debt" desks who are playing for the eventual liquidation value.
    

---

## Synthetic Summary Table: The Desk Dashboard

|**Risk Name**|**Metric**|**What it tells the Trader**|**Hedge Tool**|
|---|---|---|---|
|**CS01**|Spread Delta|"Market is getting worried."|CDS Index (CDX/iTraxx)|
|**JTD**|Default Risk|"The company is gone."|None (This is the core risk)|
|**IR01**|Rate Delta|"The Fed is hiking."|Eurodollar / SOFR Futures|
|**Gamma**|Convexity|"My losses are accelerating."|Options / Dynamic Hedging|
|**Rec01**|Recovery Delta|"The assets are worth less."|Physical Bonds|

---

## Worked Example: A $50M Position

Let's look at a trader who sold protection on **$50,000,000** of Company Y.

1. **Current Spread:** 100 bps. **Duration:** 4.6.
    
2. **CS01:** $50\text{M} \times 4.6 \times 0.0001 = \mathbf{\$23,000}$. (If spreads move to 101, they lose $23k).
    
3. **JTD:** $50\text{M} \times (1 - 0.40) - \text{MTM} \approx \mathbf{\$30,000,000}$. (The "Max Pain" scenario).
    
4. **The Move:** Spreads gap to 200 bps (+100 bps).
    
    - **Initial Loss:** $100 \times \$23,000 = \mathbf{\$2,300,000}$.
        
    - **Gamma Impact:** Because of negative gamma, their new duration is now 4.2. Their _new_ CS01 is only $21,000. They are "losing" sensitivity as they approach default.
        

---

## Further Generalizations

- **Theta (Time Decay):** We already discussed this as **Carry**. It is the daily "income" earned for holding the risk.
    
- **Cross-Gamma:** On a large desk, traders also watch how the credit spread of one company moves when _another_ company defaults (Contagion).
    
