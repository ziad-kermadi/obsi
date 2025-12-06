

### **Forward Swap Rate: Definition, Intuition, and Calculation**  

The **forward swap rate** is the fixed rate on a **swap that starts at a future date**. It is the rate that makes the present value of future floating payments equal to the present value of future fixed payments.

---

## **1. Why Do We Need Forward Swap Rates?**
- In interest rate derivatives markets, swaps are often structured **to start in the future** (e.g., a **1y5y swap**, meaning a swap that starts in 1 year and has a 5-year tenor).
- The forward swap rate helps determine the **fair fixed rate** for such delayed swaps.
- It is commonly used in pricing **swaptions** and **constant maturity swaps (CMS)**.

---

## **2. Definition of Forward Swap Rate**
The **forward swap rate** $S(T_1, T_2)$ is the fixed rate that makes the **present value (PV) of future fixed leg payments equal to the PV of the future floating leg payments**.
$$
\sum_{i=1}^{N} P(T_0, T_i) \Delta_i S(T_1, T_2) = 1 - P(T_0, T_N)
$$

where:
- $T_1$ = Start date of the forward swap.
- $T_2$ = End date of the forward swap.
- $P(T_0, T_i)$ = Discount factor from today ($T_0$) to payment date $T_i$.
- $\Delta_i$ = Year fraction of the fixed payment intervals.
- $S(T_1, T_2)$ = Forward swap rate.

Solving for $S(T_1, T_2)$:
$$
S(T_1, T_2) = \frac{1 - P(T_0, T_N)}{\sum_{i=1}^{N} P(T_0, T_i) \Delta_i}
$$

---

## **3. Intuition Behind Forward Swap Rate**
- **Concept:** The fixed rate should ensure the swap has **zero value at inception**.
- **Floating leg intuition:** The floating leg resets periodically, meaning its **value equals 1 at par**.
- **Fixed leg intuition:** The fixed rate must be set so that the fixed cash flows are equal in value to this floating leg.

---

## **4. Example Calculation**
Suppose:
- A **1y5y swap** (a swap that starts in **1 year** and lasts **5 years**).
- Discount factors:  
  - $P(0,1) = 0.98$,  
  - $P(0,2) = 0.95$,  
  - $P(0,3) = 0.91$,  
  - $P(0,4) = 0.87$,  
  - $P(0,5) = 0.83$,  
  - $P(0,6) = 0.78$.
- Fixed payments are **annual** ($\Delta = 1$).

**Step 1: Compute the PV of the Fixed Leg Denominator**
$$
\sum_{i=2}^{6} P(0, i) = 0.95 + 0.91 + 0.87 + 0.83 + 0.78 = 4.34
$$

**Step 2: Compute the PV of the Floating Leg**
$$
1 - P(0,6) = 1 - 0.78 = 0.22
$$

**Step 3: Compute the Forward Swap Rate**
$$
S(1,6) = \frac{0.22}{4.34} = 5.07\%
$$

Thus, the **fair fixed rate for this 1y5y swap** is **5.07%**.

---

## **5. Where Are Forward Swap Rates Used?**
1. **Swaptions:**  
   - A **swaption’s intrinsic value** depends on the forward swap rate.  
   - If the forward swap rate is higher than the strike, the swaption is in the money.  

2. **Constant Maturity Swaps (CMS):**  
   - CMS products reference forward swap rates.  

3. **Convexity Adjustments for CMS Rates:**  
   - CMS rates require convexity adjustments due to **measure changes** (from swap measure to discounting measure).  

---

## **6. Final Takeaway**
✅ **A forward swap rate is the fixed rate on a future-starting swap that makes its value zero today.**  
✅ **It is computed using discount factors to equate the PV of fixed and floating legs.**  
✅ **It is widely used in swaptions, CMS, and risk management.**  




------
---------------
---------------------
### **Introduction: Why Moneyness is Useful and Why Strike Relative to Forward Makes Sense**  

In derivatives pricing, **moneyness** is a crucial concept because it determines how far an option is from being in-the-money (ITM) or out-of-the-money (OTM). Instead of looking at the **strike price alone**, we typically **normalize it relative to the forward price**.  

### **Why Use Moneyness Instead of Strike?**  
If we only consider the **strike price** $K$, we run into several issues:  
✅ **Strike price alone is meaningless across different maturities**. A strike of 100 for a 1-month option vs. a 5-year option is not comparable.  
✅ **Moneyness accounts for the expected movement of the underlying**. If the forward price is 120, a strike of 100 is much deeper ITM than if the forward is 105.  
✅ **Normalization makes volatility surfaces more comparable across tenors**.  

### **Why Take Strike Relative to the Forward Price?**  
- The forward price $F_t$ represents the market’s best estimate of where the underlying **should be** at expiration.  
- Defining moneyness as **$K / F_t$ rather than $K / S_t$ (spot price)** removes dependence on the current price and accounts for **the risk-free drift** in the underlying.  
- This allows us to construct **more stable and meaningful volatility surfaces** across maturities.  

Now, let's dive into **normalized moneyness** and how it helps in structuring volatility surfaces.  

---

## **1. Why Use Normalized Moneyness?**
- **Volatility smiles and skews** often change shape across maturities. Using absolute strike levels makes interpolation difficult.
- **Normalized moneyness adjusts for different forward levels** across maturities, making the volatility surface more comparable.
- **Ensures consistency across different tenors** so that volatility across different expiries can be compared on the same scale.

---

## **2. Definition of Normalized Moneyness**
### **A. Standard Moneyness vs. Normalized Moneyness**
1. **Standard Moneyness** (Strike relative to Forward):
$$
   M = \frac{K}{F_t}
$$
   - $K$ = Strike price
   - $F_t$ = Forward price at time $t$
   - **Issue**: This does not account for the passage of time or volatility.

2. **Normalized Moneyness (Log Moneyness Adjusted for Time & Volatility)**
$$
   \tilde{m} = \frac{\ln(K/F_t)}{\sigma_{\text{ATM}} \sqrt{T}}
$$
   where:
   - $\sigma_{\text{ATM}}$ = ATM volatility
   - $T$ = Time to maturity
   - **Benefit**: This rescales the strike into a dimensionless quantity, allowing **better comparison across expiries**.

### **B. Alternative Normalized Moneyness Definitions**
- **Black-Scholes Delta as Moneyness Proxy**  
  - Instead of strikes, we can use **option delta** as a proxy:
$$
    \tilde{m} = \Phi^{-1}(\Delta)
$$
  - This ensures smooth volatility interpolation across strikes.

---

## **3. How Normalized Moneyness is Used in Volatility Surfaces**
### **A. Expressing the Volatility Surface in Terms of Normalized Moneyness**
Instead of defining the volatility as:
$$
   \sigma(K, T)
$$
we define it as:
$$
   \sigma(\tilde{m}, T)
$$
where $\tilde{m}$ is the **normalized moneyness**.

### **B. Benefits**
✅ **Smoother interpolation**: Volatility behaves more predictably in terms of $\tilde{m}$ than in terms of raw strike price.  
✅ **Better extrapolation**: When fitting a volatility model, normalized moneyness provides a more **stable and universal** structure across different tenors.  
✅ **Allows for better stochastic volatility modeling**: Works well with models like **SABR** where vol dynamics are often specified in terms of log moneyness.

---

## **4. Example: Normalized Moneyness in Practice**
Let’s say we have:
- **Forward price** $F = 100$
- **Strike price** $K = 110$
- **ATM volatility** $\sigma_{\text{ATM}} = 20\%$
- **Time to expiry** $T = 1$ year

The **normalized moneyness** is:
$$
\tilde{m} = \frac{\ln(110/100)}{0.2 \times \sqrt{1}} = \frac{0.0953}{0.2} = 0.4765
$$

Now, instead of plotting vol as a function of $K$, we plot it as a function of $\tilde{m}$, creating a **better-structured surface**.

---

## **5. Integration into Volatility Models**
### **A. SABR Model (Uses Normalized Moneyness)**
- The **SABR stochastic volatility model** often expresses vol dynamics as a function of log-moneyness:
$$
  \sigma(K, T) = \sigma_{\text{ATM}} \left( 1 + \beta \tilde{m} + \gamma \tilde{m}^2 + \dots \right)
$$
- This ensures that vol moves **smoothly** across strikes and maturities.

### **B. Local Volatility Models**
- The **Dupire local volatility model** can be rewritten in terms of normalized moneyness for smoother estimation:
$$
  \sigma_{\text{local}}(S, T) = \sigma(\tilde{m}, T) \sqrt{\frac{2}{1 + \tilde{m}^2}}
$$

---

## **6. Final Takeaway**
✅ **Normalized moneyness rescales strikes to improve volatility interpolation**.  
✅ **It accounts for the passage of time and volatility, making surfaces more stable**.  
✅ **Widely used in stochastic models like SABR and Dupire local vol**.  
✅ **Ensures smooth term structure evolution, making pricing and hedging more reliable**.  


--------
-------
------------
### **What Does a Roll Date for a Derivative Product Mean?**  

A **roll date** in derivatives refers to the **specific date on which a position in a contract is rolled forward** to a new contract with a later maturity. This process is commonly used in **futures, swaps, and structured products** to maintain exposure beyond the expiration of the current contract.

---

## **1. Why Do We Have Roll Dates?**
Many derivative contracts have **finite maturities**. If a trader or portfolio manager wants to maintain exposure **beyond expiration**, they must **"roll"** the position into a new contract.

A **roll date** is the date when:
1. The existing contract is **closed or expired**.
2. A new contract with a later maturity is **entered into**.

This is common in:
- **Futures contracts** (e.g., rolling from a March contract to a June contract).
- **Interest rate swaps** (e.g., rolling quarterly fixings in CMS swaps).
- **Options positions** (e.g., rolling short-dated options into longer-dated ones).
- **Total return swaps (TRS)** where the funding leg is reset periodically.

---

## **2. Examples of Roll Dates in Different Products**
| **Product** | **What Happens on the Roll Date?** |
|------------|--------------------------------|
| **Futures** | The expiring contract is closed, and a new contract with a later expiry is entered. |
| **Swaps (e.g., CMS, OIS, TRS)** | The floating leg reference rate is reset for the next period. |
| **FX Forwards** | The near-leg contract settles, and a new forward contract is entered for the next period. |
| **Options** | The current position is closed or adjusted, and a new position is opened with a later expiry. |

---

## **3. How Roll Dates Work in Practice**
### **A. Rolling a Futures Contract**
- Suppose a trader holds a **March crude oil futures contract** but wants exposure beyond March.
- **Roll date:** A few days before March expiration.
- **Action:** Close the March contract and enter a new **June futures contract**.

### **B. Rolling an Interest Rate Swap (IRS)**
- In a **quarterly reset swap**, the floating leg references **LIBOR, SOFR, or EURIBOR**.
- **Roll date:** The next fixing date (e.g., every 3 months).
- **Action:** The floating leg is reset based on the new reference rate.

### **C. Rolling a Total Return Swap (TRS)**
- TRS positions are typically rolled **monthly or quarterly**.
- **Roll date:** The next payment/reset date.
- **Action:** The TRS counterparty resets the floating leg and continues exposure to the underlying.

---

## **4. Key Considerations for Roll Dates**
- **Liquidity Risks:**  
  - Rolling large positions can **impact the market** and widen bid-ask spreads.
- **Basis Risk:**  
  - The old and new contracts may not be perfectly aligned, leading to a price mismatch.
- **Funding Costs:**  
  - The cost of rolling positions includes transaction costs and potential slippage.

---

## **5. Final Takeaway**
✅ **A roll date is when a derivative contract is replaced with a new one to maintain exposure.**  
✅ **Common in futures, swaps, FX, and options.**  
✅ **Key concerns include liquidity, basis risk, and transaction costs.**  
