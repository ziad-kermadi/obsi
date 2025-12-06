



----
----
----

# **Understanding CDS Conventions: C17 vs. M17 and Other Rolling Conventions (Step-by-Step with Examples)**  

The **C17** and **M17** conventions in **Credit Default Swaps (CDS)** relate to how contracts **roll**, meaning how new CDS contracts are issued and how their maturities align over time. These conventions affect the **tenors (e.g., 5Y, 10Y) and contract standardization**, which impacts liquidity, pricing, and trading strategies.

---

## **1. What Are C17 and M17 Conventions?**
- **C17 (Quarterly Rolling Convention)**  
  - CDS contracts **roll every 3 months**.  
  - Maturity dates align with **IMM (International Monetary Market) dates**:  
    - **March 20**, **June 20**, **September 20**, and **December 20**.  
  - A **5Y CDS issued in March 2022** will **mature in March 2027**.
  
- **M17 (Semi-Annual Rolling Convention)**  
  - CDS contracts **roll every 6 months**.  
  - Maturity dates align with **March 20** and **September 20** only.  
  - A **5Y CDS issued in March 2022** will **mature in March 2027**, and a contract issued in **September 2022** will mature in **September 2027**.

### **Key Difference:**
- **C17 rolls every quarter** → New CDS contracts are issued more frequently.  
- **M17 rolls every 6 months** → CDS contracts align only with March & September.  

---

## **2. Step-by-Step Example to Explain the Confusion**
You mentioned that **both conventions align in month 10 (December 29)**, but after that:
- **C17 rolls again in March**, meaning the 5Y CDS moves from **Dec 29 → March 30**.
- **M17 keeps it at December 29** (since it aligns only with March & September).

Let's break this down with an example.

### **Step 1: Initial Alignment in December 2023**
Imagine today is **December 2023**, and you want to trade a **5-year CDS**.

| **Convention** | **Start Date** | **End Date (5Y Maturity)** |
|--------------|--------------|-------------------|
| **C17 (Quarterly)** | December 29, 2023 | December 29, 2028 |
| **M17 (Semi-Annual)** | December 29, 2023 | December 29, 2028 |

At this point, both conventions align:  
**C17 and M17 both have a 5Y CDS expiring on December 29, 2028.** ✅

---

### **Step 2: The First Roll in March 2024**
- **C17 rolls in March 2024**, issuing a new **on-the-run 5Y CDS**.
- **M17 does NOT roll in March** (it only rolls in September).

Now, the new 5Y CDS under **C17** will **expire on March 30, 2029**, while **M17 still has the December 29, 2028 maturity**.

| **Convention** | **Start Date** | **End Date (5Y Maturity)** |
|--------------|--------------|-------------------|
| **C17 (Quarterly)** | March 30, 2024 | March 30, 2029 |
| **M17 (Semi-Annual)** | December 29, 2023 | December 29, 2028 |

**Now the two conventions are out of sync!** 🚨  
- **C17 now has a new on-the-run 5Y contract expiring in March 2029**.  
- **M17 keeps the previous December 2028 maturity until the next semi-annual roll in September 2024**.  

---

### **Step 3: The September 2024 Roll**
- **M17 finally rolls in September 2024**, issuing a new **5Y CDS expiring in September 2029**.
- **C17 has already rolled in June 2024 and will roll again in December 2024**.

Now we have:

| **Convention** | **Start Date** | **End Date (5Y Maturity)** |
|--------------|--------------|-------------------|
| **C17 (Quarterly)** | September 30, 2024 | September 30, 2029 |
| **M17 (Semi-Annual)** | September 30, 2024 | September 30, 2029 |

At this point, they **align again temporarily**.

---

### **3. Why Does This Happen?**
- **C17 rolls every 3 months**, so maturities are always shifting.  
- **M17 rolls every 6 months**, so its maturities move less frequently.  
- **They align in December 2023, but C17 rolls ahead in March 2024**.
- **They re-align in September 2024 when M17 finally rolls**.

This cycle **keeps repeating**, where they occasionally align but then go out of sync again.

---

## **4. Other CDS Rolling Conventions**
Besides **C17 and M17**, there are other rolling conventions used in the market:

| **Convention** | **Rolling Frequency** | **Maturity Dates** | **Used By** |
|--------------|-----------------|-------------------|------------|
| **C17 (Quarterly)** | Every 3 months | March, June, September, December | Market standard for CDS indices |
| **M17 (Semi-Annual)** | Every 6 months | March, September | Some single-name CDS |
| **SNR (Single-Name Rolling)** | Custom | Fixed maturity for each corporate CDS | Banks & customized hedging |
| **IMM Rolling** | Every 3 months | Follows IMM dates (March 20, June 20, Sept 20, Dec 20) | Some sovereign & corporate CDS |
| **CDX/iTraxx Rolls** | Every 6 months | March, September | Standard for index CDS |

### **Key Takeaways**
- **C17 = Quarterly rolling, new contracts every 3 months**.
- **M17 = Semi-annual rolling, new contracts every 6 months**.
- **They align at some points but then go out of sync**.
- **Market participants choose different conventions based on liquidity and trading needs**.

---

## **5. Final Takeaways**
1. **C17 rolls every 3 months, while M17 rolls every 6 months.**
2. **They align temporarily (e.g., December 29, 2023), but C17 rolls first (e.g., March 30, 2024), making the 5Y CDS maturities different.**
3. **M17 eventually catches up, typically in September, creating a temporary alignment before C17 rolls again.**
4. **Market participants trade the most liquid contract, so traders rolling positions must adjust between conventions.**
5. **These conventions exist to standardize CDS trading and ensure smooth hedging and liquidity.**

🚀 **Now, you have a clear understanding of C17 vs. M17 conventions and how rolling impacts CDS maturities! Let me know if you need even more examples!**



----
----
-----
### **How Can a 5-Year CDS Be Off-the-Run If It Will Become a 4-Year CDS Next Year?**

This is a **great question** because it highlights the subtle **differences between "tenor" and "on-the-run/off-the-run" status**. Let’s break it down **step by step** with examples and explanations.

---

## **1. What Does "On-the-Run" vs. "Off-the-Run" Mean?**
- **On-the-Run (OTR) CDS** → The **most recently issued CDS contract for a given tenor**. This means it is the **latest standard contract** that traders prefer to trade due to **high liquidity**.
- **Off-the-Run (OTR) CDS** → An **older CDS contract** that used to be on-the-run but has since been **replaced by a newer contract**. These contracts still exist and trade, but they are **less liquid**.

📌 **Key Point:**  
Being **off-the-run has nothing to do with how many years are left on a CDS contract**. It only means that **a newer CDS contract has replaced it as the preferred trading instrument**.

---

## **2. How a 5-Year CDS Becomes Off-the-Run?**
A 5-year CDS contract **becomes off-the-run** when a **newer 5-year CDS contract is issued** in a later roll cycle.

### **Example: The Lifecycle of a 5-Year CDS**
| **Date** | **Most Recent 5Y CDS (On-the-Run)** | **Older 5Y CDS (Off-the-Run)** |
|---------|------------------|------------------|
| **March 2023 Roll** | 5Y CDS expiring in March 2028 (on-the-run) | Older 5Y CDS expiring in Sept 2027 (off-the-run) |
| **September 2023 Roll** | 5Y CDS expiring in Sept 2028 (on-the-run) | 5Y CDS expiring in March 2028 (off-the-run) |
| **March 2024 Roll** | 5Y CDS expiring in March 2029 (on-the-run) | 5Y CDS expiring in Sept 2028 (off-the-run) |
| **March 2025** | The **March 2023 contract now has 4 years left** | Still off-the-run |

📌 **Key Observation:**  
- The **March 2023 5Y CDS contract (expiring in March 2028) was on-the-run when it was first issued**.
- When the **September 2023 5Y CDS contract (expiring in September 2028) was issued**, the **March 2023 contract became off-the-run**.
- Even though the **March 2023 CDS still has 5 years to maturity at the time**, it is **no longer the most recently issued contract**, so it **is off-the-run**.

---

## **3. What Happens Next Year When the 5Y CDS Becomes a 4Y CDS?**
When a **5-year CDS ages into a 4-year CDS**, it remains **off-the-run** unless **a new on-the-run 4-year CDS is not issued**.

🔹 **Example: March 2023 5Y CDS Turning Into a 4Y CDS**
- **March 2023**: A trader buys a **5Y CDS (expiring in March 2028)**.  
- **March 2024**: This contract is now a **4Y CDS**.  
- **March 2024 Roll Occurs**: A new **4Y CDS expiring in March 2028 is issued**.
- The **March 2023 4Y contract remains off-the-run** because the newly issued **March 2024 4Y CDS is now the preferred trading instrument**.

📌 **Key Takeaway**:  
- The **March 2023 5Y CDS (now a 4Y CDS in 2024) is still off-the-run** because the **March 2024 roll created a new 4Y CDS contract** that is **on-the-run**.

---

## **4. Why Is the New 5-Year CDS Always the On-the-Run Contract?**
- **The CDS market uses the 5-year contract as the primary benchmark**.
- The new 5-year CDS contract is **always the most liquid and most traded instrument**.
- Traders prefer to **roll their positions into the newest 5Y CDS** rather than trade older contracts.

📌 **Important Rule:**  
Whenever a **new 5-year CDS contract is issued**, it **automatically becomes the on-the-run contract**, and the **previous 5Y contract becomes off-the-run**, even if it **still has 5 years to maturity**.

---

## **5. Why Does Liquidity Move Away From Off-the-Run CDS?**
- The market prefers to trade the **latest standardized contract**.
- **Bid-ask spreads are tighter** on on-the-run contracts.
- **Rolling CDS positions into the new 5Y contract ensures liquidity**.

📌 **Real-World Example**:  
Imagine you have **a government bond issued in 2023 with a 10-year maturity**.  
- In **2023**, it is the **on-the-run 10Y bond**.  
- In **2024**, a new **10Y bond is issued**.  
- The **old bond still has 9 years left, but it is now off-the-run** because a **new 10Y bond is the preferred benchmark**.

The same logic applies to CDS contracts!

---

## **6. Summary Table: How a 5Y CDS Becomes Off-the-Run**
| **Date** | **CDS Contract** | **Years to Maturity** | **Status** |
|---------|------------------|----------------------|------------|
| **March 2023** | 5Y CDS (exp. March 2028) | 5 Years | **On-the-Run** |
| **September 2023** | 5Y CDS (exp. Sept 2028) | 5 Years | **On-the-Run** (New) |
| **March 2024** | 5Y CDS (exp. March 2029) | 5 Years | **On-the-Run** (New) |
| **March 2024** | 4Y CDS (exp. March 2028) | 4 Years | **Off-the-Run** |
| **March 2025** | 3Y CDS (exp. March 2028) | 3 Years | **Off-the-Run** |

---

## **7. Final Takeaways**
- A **5Y CDS can become off-the-run before it even ages into a 4Y or 3Y CDS**.
- Being **off-the-run means a newer contract has replaced it as the preferred instrument**, **not** that it is no longer a 5Y CDS.
- Even if a **5Y CDS still has 5 years left, it can be off-the-run** if a **newer 5Y contract has been issued**.
- **Rolling CDS positions** into the latest on-the-run contract ensures liquidity and minimizes transaction costs.
- The **CDS market operates similarly to the government bond market**, where new issues replace older ones as benchmarks.

🚀 **Now, you fully understand how a 5Y CDS can be off-the-run and what happens when it becomes a 4Y or 3Y CDS! Let me know if you need more clarity!**










---
---
---
## **. What Does "Rolling Into a New Contract" Mean?**
### **Definition of Rolling a CDS**
- **Rolling a CDS position** means **closing an existing contract and replacing it with a newly issued CDS contract of the same tenor**.
- This is done because **on-the-run contracts are more liquid, easier to trade, and have tighter bid-ask spreads**.

🔹 **Example 4: Rolling a 5Y CDS**
- Suppose an investor holds a **5-year CDS entered in March 2023 (expiring in March 2028)**.
- In **March 2024**, the investor wants to maintain a **5-year CDS exposure**.
- However, the contract they hold is now a **4-year CDS (expiring in March 2028)**.
- Instead of keeping the older, **less liquid** 4-year contract, the trader **sells it** and enters into a **new 5-year CDS contract (expiring in March 2029)**.
- This new **March 2024 5Y CDS contract is now the on-the-run 5Y contract**.

📌 **Key Takeaways from Rolling CDS:**
1. **Rolling ensures liquidity** – The trader moves from a **less liquid off-the-run contract to a more liquid on-the-run contract**.
2. **The process repeats every 6 months** – CDS contracts roll in **March and September**.
3. **Hedging becomes easier** – Standardized liquidity makes hedging smoother.







---------------
-------------
----------------

# **Can a 3-Year or 4-Year CDS Be On-the-Run? A Deep Dive with All Cases & Examples**  

### **Short Answer:**  
Yes, a **3-year or 4-year CDS can be on-the-run** if it is the **most recently issued contract for that tenor**. However, the **5-year CDS is the most liquid and standard tenor**, making it the primary on-the-run contract for most credit names.  

Additionally, a **CDS that started as a 5-year contract will naturally become a 4-year or 3-year contract over time**, but it does **not remain on-the-run unless it was rolled into a new contract**.

---

## **1. Understanding On-the-Run vs. Off-the-Run in CDS**
### **What Does On-the-Run Mean?**
- **On-the-Run (OTR) CDS** refers to the **most actively traded and liquid CDS contract for a given tenor**.
- These contracts are **standardized** and roll every **6 months** (March & September).
- The **5-year CDS** is typically the **benchmark contract** because it has the highest liquidity and is used for pricing and hedging in the market.

### **What Does Off-the-Run Mean?**
- **Off-the-Run (OTR) CDS** are **older series of CDS contracts that have been replaced by a newer on-the-run contract**.
- These contracts **still trade but are less liquid**.
- Their pricing is often derived using **interpolation from the on-the-run CDS curve**.

---

## **2. Are 3-Year and 4-Year CDS Ever On-the-Run?**
### **YES – But Not Always**
- The **on-the-run CDS** is **not limited to the 5-year tenor**.
- If the **most recent 3-year or 4-year contract is actively traded**, it can be considered **on-the-run**.
- However, **the 5-year CDS always remains the benchmark**, so 3Y and 4Y CDS **do not have the same liquidity**.

### **Example 1: A 3Y CDS Being On-the-Run**  
- Suppose today is **March 2024** and a CDS roll just happened.
- The **March 2024 3-year CDS (expiring in March 2027)** is issued and becomes the **most liquid 3-year CDS**.
- For someone looking to trade a **3Y CDS**, the **March 2024 3Y contract** is **on-the-run**.
- The **December 2023 3Y contract** (which was previously liquid) is now **off-the-run**.

### **Example 2: A 4Y CDS Being On-the-Run**
- Suppose a trader wants a **4-year CDS**.
- The **March 2024 4-year CDS (expiring in March 2028)** is now the **on-the-run** for that tenor.
- The **December 2023 4-year CDS** becomes **off-the-run**.

---

## **3. What Happens When a 5-Year CDS Becomes a 4-Year or 3-Year CDS?**
### **Does It Remain On-the-Run?**
- A **CDS that starts as a 5-year contract will naturally become a 4-year, 3-year, and eventually a 1-year CDS** as time passes.
- However, **it does not remain on-the-run unless it was rolled into a new contract**.
- The new on-the-run contracts are **always issued fresh every 6 months**, meaning that the original 5-year CDS will **become off-the-run** once a new contract replaces it.

🔹 **Example 3: A 5Y CDS Becoming a 4Y CDS**
- In **March 2023**, a trader enters into a **5-year CDS contract** (expiring in **March 2028**).
- By **March 2024**, this contract has **only 4 years left** until expiration.
- However, in March 2024, a **new 4-year CDS contract (expiring in March 2028) is issued**.
- The **old 5Y contract (now a 4Y contract) is no longer the most liquid contract for the 4Y tenor**.
- The newly issued **March 2024 4Y CDS** becomes the **on-the-run** 4Y CDS.
- The **old 4Y CDS (which started as a 5Y contract in March 2023) becomes off-the-run**.

---


---

## **5. How Market Participants Use Different Tenors**
### **5.1 Why Is the 5-Year CDS the Most Liquid?**
- The 5-year tenor is the **standard** because:
  - **CDS indices (CDX, iTraxx) use 5Y as the benchmark**.
  - Most **credit curves are priced off the 5Y point**.
  - **Regulators and market participants prefer a standardized reference**.

### **5.2 When Would a 3Y or 4Y CDS Be More Useful?**
- **Shorter Horizon Hedging**:  
  - If an investor needs protection for **only 3 years**, buying a 3Y CDS is more efficient than buying a 5Y CDS and closing it early.
- **Pricing Non-Standard Tenors**:  
  - Banks sometimes issue **3-year or 4-year bonds**.  
  - A **3Y CDS aligns with a 3Y bond maturity for hedging**.
- **Trading Curve Positions**:  
  - A trader might buy a **3Y CDS** and sell a **5Y CDS** to **express a view on credit curve steepening**.

---

## **6. Summary Table of Key Cases**
| **Case** | **Can It Be On-the-Run?** | **Why?** |
|---------|------------------|------|
| **New 5Y CDS issued in March 2024** | ✅ Yes | Most liquid and benchmark tenor |
| **New 3Y CDS issued in March 2024** | ✅ Yes | Most liquid 3Y CDS |
| **New 4Y CDS issued in March 2024** | ✅ Yes | Most liquid 4Y CDS |
| **A CDS that started as a 5Y in March 2023 (now 4Y in March 2024)** | ❌ No | A newer on-the-run 4Y CDS was issued |
| **A CDS that started as a 5Y in March 2022 (now 3Y in March 2024)** | ❌ No | A newer on-the-run 3Y CDS was issued |

---

## **7. Final Takeaways**
- **A 3Y or 4Y CDS can be on-the-run, but only if it is the latest issued contract for that tenor**.
- **A 5Y CDS naturally becomes a 4Y or 3Y CDS over time, but it will be off-the-run unless rolled into a new contract**.
- **Rolling means closing an old contract and opening a new one to maintain liquidity**.
- **Traders roll CDS contracts every 6 months in March & September to ensure they hold the most liquid version**.

🚀 **Now you have a deep understanding of CDS rolls, on-the-run vs. off-the-run dynamics, and how different tenors are treated in the market! Let me know if you need more details!**











-----
----
----







# **Understanding CDS Quotation Methods: Upfront, Par Spread, Running Spread, and Flat Spread**

Credit Default Swaps (CDS) are financial derivatives that function as insurance against the default of a reference entity, such as a corporation or government. The cost of this protection can be quoted using various methods, each reflecting the credit risk and market conventions differently. Below, we delve into the primary quotation methods—**Upfront Payments**, **Par Spreads**, **Running Spreads**, and **Flat Spreads**—and explore how to convert between them. Additionally, we discuss the market's shift towards upfront payments and clarify the distinctions between quoted and running spreads.

---

## **1. Quotation Methods for CDS**

### **1.1 Par Spread**

The **Par Spread** is the fixed annual premium, expressed in basis points (bps), that the protection buyer pays periodically (typically quarterly) over the life of the CDS contract. This spread is set so that the **present value (PV) of the premium leg equals the PV of the protection leg**, resulting in a CDS contract with **zero net present value at inception**. In other words, neither party makes an upfront payment; the contract is at par.

**Example:**
- **Reference Entity:** Company XYZ  
- **Notional Amount:** $10 million  
- **Maturity:** 5 years  
- **Par Spread:** 150 bps (1.50%)  

The protection buyer pays 1.50% of $10 million annually, equating to $150,000 per year, typically divided into quarterly payments of $37,500 each.

### **1.2 Running Spread**

The **Running Spread** is synonymous with the par spread in traditional CDS contracts. It represents the periodic premium payments made by the protection buyer without any upfront payment. The term "running" emphasizes the ongoing nature of these payments throughout the contract's life.

**Example:**
- **Reference Entity:** Company ABC  
- **Notional Amount:** $5 million  
- **Maturity:** 3 years  
- **Running Spread:** 200 bps (2.00%)  

The protection buyer pays 2.00% of $5 million annually, totaling $100,000 per year, divided into four quarterly payments of $25,000 each.

### **1.3 Upfront Payment (Standardized CDS Quoting Method)**

In **Upfront Quoting**, the buyer of protection pays a **single upfront premium**, with a fixed running spread that is standardized (typically 100 or 500 bps). The upfront amount is the **difference between the fair value of the CDS and the standardized spread present value**.

**Why Upfront Pricing Became Common**:
- Before standardization, CDS contracts had **non-standard coupons** based on credit risk, creating **disparities** in market conventions.
- The introduction of **fixed spreads (100 or 500 bps) and an upfront payment** ensured that **all CDS contracts were standardized**, making it easier to compare and trade.
- This move made it easier for traders to price, hedge, and manage CDS portfolios by eliminating inconsistencies in contract structures.

**Formula to Convert Between Par Spread and Upfront Payment:**
$$
\text{Upfront} = \left( S_{\text{par}} - S_{\text{fixed}} \right) \times \text{PV01}
$$
where:
- $S_{\text{par}}$ = Fair market spread  
- $S_{\text{fixed}}$ = Standardized running spread (100 or 500 bps)  
- **PV01** = Present value of a 1 bp annuity of the CDS premium payments  

**Example:**
- If a 5-year CDS has a **fair value spread of 300 bps**, but the standardized contract uses **100 bps fixed running spread**, the upfront payment is:
$$
(300 - 100) \times 4.2 = 840 \text{ bps} \text{ (8.4% of notional)}
$$

For a **$10 million** notional CDS, the upfront payment would be **$840,000**.

### **1.4 Flat Spread**

The **Flat Spread** is the implied running spread when a CDS is quoted with an upfront payment. It **adjusts the market spread to a hypothetical contract with no upfront**, making it easier to compare across different structures.

**Conversion Formula (Flat Spread Approximation):**
$$
S_{\text{flat}} \approx S_{\text{fixed}} + \frac{\text{Upfront Payment}}{\text{PV01}}
$$

If a CDS has an **upfront cost of 500 bps** and the **PV01 is 4.2**, the flat spread is:
$$
100 + \frac{500}{4.2} = 219 \text{ bps}
$$

This means that **if the contract had no upfront, the equivalent running spread would be 219 bps**.

---

## **2. Difference Between Running Spread and Quoted Spread**
### **2.1 Running Spread (Legacy Method)**
- The periodic coupon paid by the CDS protection buyer.
- Can be any value (e.g., 150, 300, or 500 bps).
- Used before standardization.

### **2.2 Quoted Spread (Modern Method)**
- Used for standardized CDS contracts where spreads are fixed at **100 or 500 bps**.
- The quoted spread is the equivalent fair market spread if the CDS had no upfront payment.
- Market participants compare CDS values using quoted spreads to determine relative risk levels.

**Example:**
- A 5Y CDS contract has a running spread of **350 bps**.
- The standardized contract uses **100 bps running spread** with an **upfront payment of 1,000 bps**.
- The quoted spread helps traders quickly assess if they are paying a fair price relative to the market.

---

## **3. Why Did the Market Shift to Upfront Pricing?**
### **3.1 Problem with Non-Standard Running Spreads**
Before 2009, CDS contracts had **customized running spreads**. This created **several problems**:
1. **Difficult Comparisons**: A 200 bps CDS and a 500 bps CDS couldn’t be easily compared.
2. **Complicated Risk Management**: Portfolio hedging was harder since contracts had different conventions.
3. **Settlement Disputes**: Bond recovery values vs. CDS payouts weren’t always aligned.

### **3.2 Solution: Standardization with Fixed Running Spreads**
- The market shifted to **fixed running spreads (100 or 500 bps) with upfront adjustments**.
- This allowed traders to price, hedge, and compare contracts easily.
- Upfront payments ensured that contracts reflected **actual credit risk** without arbitrary coupon differences.

---

## **4. Converting Between Different CDS Pricing Methods**
### **4.1 From Par Spread to Upfront Payment**
$$
\text{Upfront} = (S_{\text{par}} - S_{\text{fixed}}) \times \text{PV01}
$$

### **4.2 From Upfront Payment to Running Spread**
$$
S_{\text{flat}} = S_{\text{fixed}} + \frac{\text{Upfront Payment}}{\text{PV01}}
$$

### **4.3 From Running Spread to Quoted Spread**
- If a CDS has a **fixed running spread of 100 bps** and an **upfront payment**, the quoted spread is:
$$
\text{Quoted Spread} = \text{Flat Spread}
$$

- This helps normalize different contracts for direct comparison.

---

## **5. Summary Table of CDS Quotation Methods**
| **Method** | **Definition** | **Example** | **Why It's Used?** |
|------------|--------------|------------|-------------------|
| **Par Spread** | Fixed annual premium that makes CDS fair value at inception | 150 bps for 5Y CDS | Used in old-style CDS contracts |
| **Running Spread** | Regular periodic payments without upfront costs | 200 bps for a 3Y CDS | Used before standardization |
| **Upfront Payment** | Lump-sum payment to adjust for fixed 100 or 500 bps spread | $840,000 upfront for $10M CDS | Standardized market pricing |
| **Flat Spread** | Implied running spread when upfront is applied | 219 bps if upfront is 500 bps | Allows fair comparison |

---

## **6. Final Takeaways**
- The market shifted to **upfront payments and fixed spreads (100/500 bps)** for **easier comparison and risk management**.
- **Quoted spread vs. running spread**: Running spread is **what is paid**, quoted spread **normalizes** CDS contracts for comparison.
- **Converting between pricing methods** is done using **PV01 calculations**.

This guide should now make CDS pricing methods **crystal clear**! 🚀 Let me know if you need deeper examples!

---------
----------
-------------

# 📌 **CDS Key Dates, On-the-Run vs. Off-the-Run, and Rolling Concepts – In-Depth & Easy-to-Understand Guide**  

This guide provides a **detailed breakdown** of **Credit Default Swaps (CDS)**, covering **key dates, on-the-run vs. off-the-run pricing, CDS rolls, and accrued spread adjustments**. The goal is to make these complex concepts easy to understand with **step-by-step explanations and thorough examples**.

---

## **1. Understanding Key Dates in a CDS Contract**
A **Credit Default Swap (CDS)** is a derivative contract where one party pays a fixed spread (the CDS premium) in exchange for protection if a reference entity (corporate or sovereign) **defaults**.  

To fully grasp CDS mechanics, it’s essential to understand the **important dates** associated with a contract.

### **1.1 CDS Timeline and Key Dates**
| **Date Type** | **Meaning** |
|--------------|------------|
| **Trade Date** | The day when two parties agree to enter into the CDS contract. |
| **Effective Date** | The date when CDS protection officially starts (T+1 for single-name CDS). |
| **Fixed Coupon Payment Dates** | Payments occur on **March 20, June 20, September 20, December 20** (standardized for liquidity). |
| **Accrual Start Date** | The last coupon payment date before the trade date. Determines how much premium the buyer owes since the last payment. |
| **Maturity Date** | The date when the CDS contract expires (e.g., 5 years from the effective date). |
| **Default Settlement Date** | If the reference entity defaults, the seller of protection makes a payment on this date. |

---

### **1.2 Accrual Start Date & Accrued Premium**
📌 **What is the Accrual Start Date?**
- The **accrual start date** is the last **fixed coupon payment date** before the trade.
- If a CDS trade happens **between two payment dates**, the buyer of protection **must compensate the seller** for the accrued spread.

📌 **Why Does the Buyer Owe Accrued Premium?**
- CDS spreads are **paid quarterly**.
- However, **if a trade happens mid-cycle**, the buyer has received protection for some time **before the next coupon payment**.
- Since protection has already been provided, the buyer **must pay the seller for that period**.

📌 **Example of Accrued Premium Calculation**
- Suppose today's date is **May 10**, and you enter a CDS contract.
- The **last payment date was March 20**, and the next one is **June 20**.
- The CDS contract charges **100 bps per year**.
- Since **May 10 is 50 days after March 20**, the buyer must pay for those 50 days of protection.
$$
\text{Accrued Spread} = \left( \frac{50}{90} \right) \times 100 \text{ bps} = 55.56 \text{ bps}
$$

- The **full premium payment** on **June 20** will be **100 bps** per year, but since the buyer already paid for 50 days, they will only owe the remaining premium from **May 10 to June 20**.

🔹 **Key Takeaway**: The **accrual start date matters because CDS spreads accumulate daily**, and mid-cycle trades require **payment adjustments**.

---

## **2. On-the-Run vs. Off-the-Run CDS**
📌 **What Do "On-the-Run" and "Off-the-Run" Mean?**  
- **On-the-Run (OTR) CDS**: The **most recently issued CDS contract** for a given tenor (e.g., 5-year CDS of an entity in the latest quarterly roll).
- **Off-the-Run (OTR) CDS**: An older CDS contract that used to be on-the-run but is now **less liquid** after the latest roll.

### **2.1 Why Does On-the-Run Matter?**
- On-the-run CDS contracts are the **most actively traded and have the tightest bid-ask spreads**.
- Traders and investors use on-the-run contracts as **benchmark CDS spreads**.

### **2.2 Example of On-the-Run vs. Off-the-Run**
- **March 2024 Roll**: A new **5-year CDS contract** is issued.
- **June 2024 Roll**: A new **on-the-run 5Y CDS** is introduced, and the **March 2024 contract becomes off-the-run**.

📌 **Key Intuition**
- Imagine **government bonds**: the **most recent U.S. 10-year Treasury bond** is the **on-the-run** bond, and older issues become **off-the-run**.
- CDS works the same way, but it rolls **every 3 months** instead of annually.

---

## **3. How On-the-Run CDS Is Used to Price Off-the-Run CDS**
Since off-the-run CDS contracts trade less frequently, we **infer their prices** using on-the-run CDS curves.

### **3.1 Interpolating Off-the-Run CDS Prices**
If we need a **4.7-year CDS price**, but we only have the **on-the-run 4Y and 5Y CDS**, we use **linear interpolation**:
$$
S_{\text{4.7Y}} = S_{\text{4Y}} + \frac{(4.7 - 4)}{(5 - 4)} \times (S_{\text{5Y}} - S_{\text{4Y}})
$$

where:
- $S_{\text{4Y}}$ = 4-year CDS spread.
- $S_{\text{5Y}}$ = 5-year CDS spread.

---

## **4. CDS Rolling Concept**
### **4.1 Why Do CDS Contracts Roll?**
- CDS contracts **standardize their dates** to ensure high **liquidity and transparency**.
- Instead of having contracts that start and end on **random days**, the market **rolls CDS contracts every quarter** (March 20, June 20, September 20, December 20).

### **4.2 What Happens During a CDS Roll?**
Every quarter:
1. **A new 5-year CDS contract becomes the new on-the-run.**
2. The previous on-the-run contract **becomes off-the-run**.
3. Traders shift their focus to the newest contract for **higher liquidity**.

📌 **Example of a CDS Roll for a 5Y CDS**
| **Date** | **On-the-Run 5Y CDS** | **Off-the-Run** |
|---------|-----------------|----------------|
| **March 2024 Roll** | 5Y CDS from March 2024 to March 2029 | 5Y CDS from Dec 2023 to Dec 2028 |
| **June 2024 Roll** | 5Y CDS from June 2024 to June 2029 | 5Y CDS from March 2024 to March 2029 |

🔹 **Key Takeaway**: A **trader who wants to maintain a 5Y CDS position** must **roll from the old 5Y contract to the new one** every 3 months.

---

## **5. How Traders Use On-the-Run and Off-the-Run CDS**
| **Trading Strategy** | **How It Works?** |
|-----------------|------------------|
| **Hedging Credit Exposure** | Investors buy CDS to hedge against bond defaults. |
| **Curve Trading (On-the-Run vs. Off-the-Run)** | Trade the spread between on-the-run and off-the-run CDS prices (liquidity premium arbitrage). |
| **Roll Arbitrage** | Exploit price discrepancies between different CDS series during a roll. |
| **Basis Trading (CDS vs. Bonds)** | Trade the spread between CDS spreads and bond yields. |

---

## **6. Summary Table: Key CDS Market Concepts**
| **Concept** | **Definition** | **Why It Matters?** |
|------------|--------------|--------------------|
| **On-the-Run CDS** | Most liquid, recently issued CDS contract | Used as a benchmark for pricing |
| **Off-the-Run CDS** | Older, less liquid CDS contract | Priced using interpolation from on-the-run CDS |
| **CDS Roll** | Quarterly adjustment where a new on-the-run contract is issued | Ensures liquidity and standardization |
| **CDS Accrual** | Payment accumulated since the last coupon | Required for clean vs. dirty pricing |

---

## **7. Final Takeaways**
- **On-the-Run CDS is the most liquid and used for pricing.**
- **Off-the-Run CDS is interpolated using the CDS curve.**
- **CDS contracts roll every 3 months** to maintain liquidity.
- **Traders exploit CDS rolls through curve trading and roll arbitrage.**




