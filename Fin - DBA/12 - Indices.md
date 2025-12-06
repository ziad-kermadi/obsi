


---------
--------------
---------------
# 📌 **Index Credit Derivatives – Cheat Sheet (CDX, iTraxx, & More)**  

This cheat sheet covers **index credit derivatives**, their **pricing**, and the **differences between major indices (CDX, iTraxx, etc.)**. It also explains **index series, rolls, and qualifiers**.

---

## **1. What Are Index Credit Derivatives?**
Index credit derivatives are standardized **credit default swap (CDS) indices** that allow investors to **gain or hedge exposure to a basket of corporate credit risk**.

**Common Indices:**
- **CDX (North America)**
- **CDX High Yield (CDX HY)**
- **iTraxx (Europe, Asia, Crossover, Senior Financials)**
- **Emerging Market Credit Indices (e.g., CDX.EM, iTraxx Asia)**

### **Key Features**
- **Standardized CDS contracts** referencing a basket of corporate bonds.
- **Traded in series** (new series launched every **6 months** in March & September).
- **Liquidity & Transparency**: More liquid than single-name CDS.
- **Tranching Possible**: Can be split into different risk levels.

---

## **2. Characteristics of Major Credit Indices**
| **Index** | **Region** | **Type** | **Number of Names** | **Credit Quality** | **Maturity** |
|----------|-----------|----------|---------------------|-------------------|--------------|
| **CDX IG** | North America | Investment Grade | 125 | IG-rated | 5Y |
| **CDX HY** | North America | High Yield | 100 | Sub-IG (junk bonds) | 5Y |
| **CDX EM** | Emerging Markets | Sovereign & Corporate | 50-75 | Mixed | 5Y |
| **iTraxx Europe** | Europe | Investment Grade | 125 | IG-rated | 5Y |
| **iTraxx Crossover** | Europe | High Yield | 75 | BB & lower | 5Y |
| **iTraxx Senior Financials** | Europe | Financial Institutions | 30 | IG banks | 5Y |
| **iTraxx Asia** | Asia | Corporate Bonds | 50 | Mixed | 5Y |

### **What Do Index Qualifiers Mean?**
| **Qualifier** | **Meaning** |
|--------------|------------|
| **CDX IG** | Investment Grade (North America) |
| **CDX HY** | High Yield (North America) |
| **CDX EM** | Emerging Markets |
| **iTraxx Europe** | European Investment Grade |
| **iTraxx Crossover** | European High Yield |
| **iTraxx Senior Financials** | European Banks (IG) |
| **Series 39, 40, etc.** | Index version (rolls every 6 months) |

---

## **3. What Is an Index Series?**
- **Each index series lasts for 6 months**.
- New series are launched in **March & September**.
- **Series numbers increase** (e.g., **CDX IG 39 → CDX IG 40**).
- **Updated constituents**: If a company **defaults or is upgraded/downgraded**, it may be replaced.

**Example of a Roll:**
- CDX IG 39 (March 2023)
- CDX IG 40 (September 2023)

---

## **4. Pricing of Index Credit Derivatives**
### **4.1 Pricing CDX/iTraxx Index (Weighted CDS Approach)**
A credit index is priced as the **weighted sum of single-name CDS spreads**.

#### **Step 1: Compute Index Spread**
$$
S_{\text{index}} = \frac{\sum_{i=1}^{N} w_i S_i}{\sum_{i=1}^{N} w_i}
$$
where:
- $S_i$ = CDS spread of each name.
- $w_i$ = Weight of each name.

#### **Step 2: Compute Default Leg (Protection Payment)**
$$
V_{\text{prot}} = \sum_{i=1}^{N} P(0,t_i) \cdot \text{LGD} \cdot \text{PD}(t_i)
$$

#### **Step 3: Compute Premium Leg (CDX Spread Payments)**
$$
V_{\text{prem}} = S_{\text{index}} \sum_{i=1}^{N} P(0,t_i) \cdot \delta_i
$$

#### **Step 4: Solve for Equilibrium Spread**
$$
S_{\text{index}} = \frac{V_{\text{prot}}}{\sum P(0,t_i) \cdot \delta_i}
$$

---

### **4.2 Pricing CDX Tranches (Gaussian Copula Model)**
CDX tranches require **correlation modeling** to determine **how defaults cluster**.

#### **Step 1: Use Gaussian Copula for Default Correlation**
$$
X_i = \rho Z_{\text{market}} + \sqrt{1 - \rho^2} \epsilon_i
$$
where:
- $Z_{\text{market}} \sim \mathcal{N}(0,1)$ (systematic factor).
- $\epsilon_i \sim \mathcal{N}(0,1)$ (idiosyncratic risk).

#### **Step 2: Compute Expected Loss for Each Tranche**
$$
\text{Tranche Loss} = \min(\max(\text{Total Portfolio Loss} - \text{Subordination}, 0), \text{Tranche Thickness})
$$

#### **Step 3: Compute Present Value of Expected Payoffs**
$$
V_{\text{tranche}} = \sum \frac{\text{Expected Payoff}_t}{(1+r)^t}
$$

#### **Step 4: Solve for Base Correlation**
The **base correlation** is calibrated to match **market prices** of different tranches.

---

### **4.3 Pricing First-to-Default Baskets**
The probability that **one or more names default** is:
$$
P_{\text{FTD}} = 1 - \prod_{i=1}^{N} (1 - P_{\text{default}, i})
$$

---

## **5. Key Differences in Pricing**
| Feature  | Single-Name CDS  | CDX Index | CDX Tranche |
|---------|----------------|----------------|----------------|
| **Pricing Complexity** | Simple | Weighted sum of CDS spreads | Requires correlation modeling |
| **Premium Leg** | Paid until default | Paid until default in index | Paid until tranche is hit |
| **Default Risk** | 1 entity | Many entities | Correlated defaults matter |
| **Market Liquidity** | Medium | High | Medium |
| **Hedging Use** | Single-name risk | Broad credit exposure | Tailored risk exposure |

---

## **6. Market Uses of Index Credit Derivatives**
| **Product** | **Use Case** |
|------------|-------------|
| **CDX/iTraxx** | Hedging portfolio-wide credit risk |
| **CDX Tranches** | Expressing views on correlation & default clustering |
| **Credit Spread Options** | Trading volatility of index CDS spreads |
| **First-to-Default Baskets** | Expressing views on single credit events |

---

## **7. Summary Table of Index Credit Derivatives**
| **Product** | **Definition** | **Pricing Method** |
|------------|---------------|--------------------|
| **CDX/iTraxx** | CDS referencing a basket of credits | Weighted sum of single-name CDS |
| **CDX Tranche** | Exposure to specific part of CDX loss distribution | Gaussian Copula Model |
| **Credit Spread Option** | Option on an index CDS spread | Black-Scholes variant |
| **First-to-Default Basket** | Pays when the first name in a basket defaults | Survival probability model |

---

## **8. Final Takeaways**
1. **Index credit derivatives aggregate multiple CDS** into a single tradeable instrument.
2. **CDX/iTraxx indices roll every 6 months** to keep the constituents updated.
3. **Pricing of CDX & iTraxx** follows **weighted average CDS pricing**, while **tranches require correlation modeling**.
4. **CDX Tranches & First-to-Default baskets** allow investors to **express views on correlation & systemic risk**.

---

This is a **full guide to Index Credit Derivatives, Pricing, and Market Applications**. 🚀 Let me know if you need **further deep dives into specific areas!**