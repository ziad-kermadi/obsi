![[Pasted image 20250316141256.png]]

![[Pasted image 20250318182748.png]]
![[Pasted image 20250318182800.png]]












![[Pasted image 20250316141431.png]]

![[Pasted image 20250316141523.png]]

![[Pasted image 20250317094951.png]]

![[Pasted image 20250317095900.png]]



![[Pasted image 20250319100622.png]]


![[Pasted image 20250317104041.png]]
***LIBOR CURVE**

![[Pasted image 20250317105919.png]]

![[Pasted image 20250317110004.png]]

![[Pasted image 20250317110042.png]]

![[Pasted image 20250317110133.png]]

![[Pasted image 20250317110205.png]]

![[Pasted image 20250317110237.png]]

![[Pasted image 20250317110315.png]]

![[Pasted image 20250317110535.png]]

![[Pasted image 20250317110716.png]]
![[Pasted image 20250317110743.png]]

![[Pasted image 20250317111109.png]]


## **Ensuring Smoothness in Turn Adjustments: The Parallel_Zero Method**
To ensure smoothness when adjusting for **turn effects** in interest rate curve construction, we need to properly constrain how rates behave across an **enclosing future** and its enclosed **turn future**. This prevents artificial jumps in the term structure, particularly in forward rates.

---

## **1. What Are We Trying to Achieve?**
The goal of the **Parallel_Zero** method is to ensure that:
- The **gradients (slopes) of the zero-coupon yield curve** remain equal before and after the turn future.
- The resulting forward curve is **continuous and smooth**, avoiding artificial jumps in implied forward rates.

This is crucial because an **unconstrained turn adjustment** can cause sharp discontinuities in the curve, leading to mispricing and arbitrage opportunities.

---

## **2. How Do We Ensure the Parallel_Zero Condition?**
We need to impose a condition on the **zero rate curve** $Z(t)$, specifically ensuring that its gradient remains unchanged across the transition.

### **Mathematical Formulation**
Let:
- $Z(t)$ be the continuously compounded **zero rate curve**.
- $P(t,T) = e^{-Z(T) (T-t)}$ be the **discount factor**.
- $f(t,T) = -\frac{d \ln P(t,T)}{dT}$ be the **instantaneous forward rate**.

For **Parallel_Zero**, we impose:
$$
\frac{dZ}{dT} \Big|_{\text{before turn}} = \frac{dZ}{dT} \Big|_{\text{after turn}}
$$

This ensures that the **zero rate curve does not experience a kink** at the turn future.

---

## **3. Practical Implementation of Parallel_Zero**
To implement this numerically when constructing the curve:

### **Step 1: Build the Discount Curve Using the Enclosing Future**
- First, construct the OIS or LIBOR discount curve using the **longer enclosing future**.
- This gives initial values for discount factors $P(t,T)$.

### **Step 2: Insert the Turn Future’s Impact**
- The turn future introduces a special rate shift (e.g., quarter-end or year-end funding pressures).
- Instead of an arbitrary discontinuity, we require that the **zero rate slope remains unchanged**.

### **Step 3: Adjust the Discount Factors to Maintain Smooth Zero Rate Gradients**
- The discount factors before and after the turn period are adjusted so that:
$$
  \frac{Z(T_{\text{before}}) - Z(T_{\text{previous}})}{T_{\text{before}} - T_{\text{previous}}} = \frac{Z(T_{\text{after}}) - Z(T_{\text{before}})}{T_{\text{after}} - T_{\text{before}}}
$$
  This ensures that **Z(T)** transitions smoothly.

### **Step 4: Solve for Forward Rates Consistently**
- Forward rates are computed from discount factors:
$$
  f(t,T) = -\frac{d \ln P(t,T)}{dT}
$$
- The constraint on the zero rate gradient ensures that **forward rates remain continuous**.

---

## **4. Why Parallel_Zero is Preferred**
- **Avoids artificial kinks** in zero rates.
- **Prevents abrupt jumps in forward rates**, which can cause distortions in short-term pricing.
- **Reflects market behavior** more realistically, as traders hedge using continuous term structures.

---

## **5. Alternative Methods and Their Issues**
- **Constant_Cash** forces cash rates to be equal, but this can create forward rate jumps.
- **Constant_Forward** forces forward rates to be equal, but may lead to zero-rate discontinuities.

**Parallel_Zero** is the best choice because it ensures a **smooth, continuous transition in term structure**.

---

### **Final Takeaway**
To **implement Parallel_Zero**, ensure:
1. **The gradient of the zero rate curve remains unchanged across the turn future.**
2. **Discount factors are solved numerically while preserving this smooth transition.**
3. **Forward rates remain continuous and arbitrage-free.**

Would you like a step-by-step numerical example to illustrate how discount factors are adjusted? 🚀

---------------------------------------------------------------------------------------------------------------------------------------------------

**OIS Curve**
![[Pasted image 20250317121046.png]]

![[Pasted image 20250317121118.png]]
![[Pasted image 20250317121143.png]]
![[Pasted image 20250317121213.png]]
![[Pasted image 20250317121236.png]]

### **Understanding the DiscountSpreads Table: Types of Spreads and Market Sources**

The **DiscountSpreads** table provides spreads that reflect adjustments in the discounting curves used for present value calculations in derivatives pricing. These spreads account for different **collateralization agreements**, currencies, and counterparty risk. Let’s break it down step by step.

---

## **1. What Types of Spreads Are in the DiscountSpreads Table?**
The table contains **different discount spreads**, which are adjustments applied to the risk-free discount curve. These spreads exist because different trades use different discounting methodologies based on:
- **Collateral agreements (CSA vs. non-CSA)**
- **Currency (USD, EUR, etc.)**
- **Counterparty risk (whether discounting is bilateral, cleared, or non-collateralized)**

Each **column** represents a different type of discount spread.

---

### **2. Explanation of the Table Columns**
Each column corresponds to a different collateralized or uncollateralized discounting scenario.

#### **Column 0: "csa-2way-usd-cash"**
- **CSA (Credit Support Annex) discounting using USD collateral**.
- **This spread represents the difference between the OIS (Overnight Indexed Swap) discount rate and the base discount curve.**
- Instruments: Derived from **OIS swap rates** and Fed Funds-based discounting curves.

#### **Column 1: "csa-db-post-usd-cash"**
- Similar to Column 0 but specific to **Deutsche Bank’s CSA post-trade discounting**.
- Can differ slightly from generic CSA discounting due to **bank-specific funding assumptions**.
- Instruments: Derived from **bank-specific repo rates and discounting practices**.

#### **Column 2: "csa-2way" (EUR collateral)**
- Reflects discounting when CSA agreements allow **EUR collateral** instead of USD.
- **EUR collateral leads to a different discounting curve due to cross-currency basis risk.**
- Instruments: Derived from **EUR OIS swaps (e.g., €STR swaps)**.

#### **Column 3: "no-csa" (Uncollateralized Discounting)**
- Represents the discount spread for **uncollateralized transactions**.
- Since these are riskier than CSA-backed transactions, the discount rate is typically **higher** to account for credit risk.
- Instruments: Derived from **LIBOR-based swaps and unsecured funding rates**.

#### **Column 4: "csa-2way-eur-cash"**
- Represents the **CSA-based discounting curve when EUR cash is used as collateral**.
- **This can differ from generic EUR CSA discounting due to specific bank repo agreements.**
- Instruments: Derived from **EUR repo rates and €STR-based discounting curves**.

---

## **3. Where Do These Discount Spreads Come From in the Market?**
To construct these spreads, we look at **market-traded derivatives that reveal discounting adjustments**. The key instruments used include:

### **(A) OIS Swaps (For CSA Discounting)**
- **Overnight Index Swaps (OIS) are used to derive the "pure" risk-free discount curve.**
- Fed Funds swaps → **USD OIS discounting**
- €STR swaps → **EUR OIS discounting**
- SONIA swaps → **GBP OIS discounting**

### **(B) Cross-Currency Basis Swaps (For Different Collateral Currencies)**
- When a swap is collateralized in EUR but discounted in USD, a **cross-currency basis spread** must be applied.
- **EUR/USD Cross-currency basis swaps** give the discount adjustment.

### **(C) LIBOR-Based Interest Rate Swaps (For No-CSA Discounting)**
- When a trade is **not collateralized**, counterparties demand a risk premium.
- This is reflected in **LIBOR swap rates vs. OIS rates**.

### **(D) Repo Markets (For Bank-Specific CSA Discounting)**
- Large banks use their **own funding rates (repo, unsecured funding) to determine discounting.**
- These rates impact **CSA-specific discount spreads**.

---

## **4. Why Do These Spreads Matter?**
- **They determine which discount curve should be used when pricing derivatives.**
- **Different collateral agreements require different discounting assumptions.**
- **They directly impact the fair valuation of swaps, bonds, and collateralized trades.**

---

![[Pasted image 20250318165826.png]]
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![[Pasted image 20250317121303.png]]
![[Pasted image 20250317121325.png]]
![[Pasted image 20250317121351.png]]

![[Pasted image 20250318171121.png]]

![[Pasted image 20250318170430.png]]
![[Pasted image 20250317121428.png]]


