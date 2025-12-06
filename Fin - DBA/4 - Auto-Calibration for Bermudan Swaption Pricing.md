


DBA pics to be done


### **Understanding Risk Aggregation Using the Vanilla European Swaption Market Vol Surface**  

When we say that the **vanilla European swaption market volatility surface** is used as an input, it means that the **volatility risk of non-standard options (e.g., Bermudan swaptions or callable swaps) is mapped to a set of vanilla European swaptions**. This enables:

1. **Risk to be expressed in terms of vanilla option volatilities** rather than bespoke, complex instruments.
2. **Risk aggregation across the vanilla portfolio**, meaning that exposures from exotic instruments can be combined and managed alongside standard swaptions.

---

### **Breaking Down the Concept:**
#### **1. Vanilla European Swaption Market Vol Surface**
- The **volatility surface** provides **implied volatilities** for swaptions of different **tenors** and **maturities**.
- It is the most liquid and observable source of market-implied vol information for interest rate options.
- Since European swaptions have well-defined payoff structures and are actively traded, they serve as a **benchmark** for pricing and risk management.

#### **2. Mapping Exotic Option Risks to Vanilla Swaptions**
- Many **interest rate exotics** (e.g., **Bermudan swaptions, callable swaps**) are **path-dependent**, making direct risk management complex.
- Instead of managing risk in terms of exotic Greeks (which are hard to hedge), risk exposure is **mapped** to equivalent vanilla **European swaptions** that have the **closest sensitivity to market movements**.
- This process is often done through **"Vega bucketing"**—breaking down an exotic option’s vega exposure into exposures across the vanilla swaption surface.

#### **3. Why This Allows Risk Aggregation**
- Once the risk is expressed in **terms of vanilla swaptions**, it can be **aggregated** with existing vanilla options in the portfolio.
- This simplifies risk management because:
  - **Hedging exotic options becomes easier** (hedges can be done using liquid European swaptions).
  - **Portfolio-wide risk can be computed in terms of standard volatilities**.
  - **Regulatory and internal risk reports** can use a common benchmark (European swaption volatilities).

---



### **Mathematical Justification for Mapping Exotic Swaptions to Vanilla European Swaptions**

Yes, the **mapping of exotic swaption risk to a vanilla European swaption volatility surface** is **rigorously justified** through **perturbation theory, local vol approximations, and Dupire’s volatility framework**. The key principle is that we approximate the exposure of complex instruments using **a portfolio of liquid vanilla instruments**.

---

## **1. Theoretical Basis: Vega Sensitivity Approximation**
The mapping relies on the fact that an exotic swaption’s price can be approximated by a **weighted sum of vanilla European swaption prices**, using a Taylor expansion around the European swaption volatilities.

Using **first-order Vega approximation**, the price $V_{\text{exotic}}$ of an exotic swaption can be expanded as:
$$
V_{\text{exotic}} \approx \sum_{i} w_i V_{\text{european}, i}
$$

where:
- $V_{\text{european}, i}$ are the prices of vanilla European swaptions with similar characteristics.
- $w_i$ are **vega weights**, determined by the sensitivity of the exotic instrument to changes in vanilla implied volatilities.

### **Vega Risk Approximation**
By differentiating the exotic swaption price w.r.t. the volatility of vanilla swaptions:
$$
\frac{\partial V_{\text{exotic}}}{\partial \sigma_{\text{european}, i}} = w_i
$$

This equation shows that the risk of an exotic swaption can be **expressed as a sum of vega risks in vanilla swaptions**, which justifies the mapping.

---

## **2. Formal Derivation Using Local Volatility (Dupire Framework)**
From **Dupire’s local volatility** framework, we express the price of an exotic option $V_{\text{exotic}}$ using an **expectation of vanilla option payoffs**:
$$
V_{\text{exotic}} = \mathbb{E}^{\mathbb{Q}} \left[ e^{-r(T-t)} \varphi(S_T) \right]
$$

where $\varphi(S_T)$ represents the payoff of the exotic option. If the exotic swaption has multiple exercise points (e.g., Bermudan swaptions), its price is computed using **backward induction** via a dynamic programming approach.

By **perturbing** the implied volatility surface $\sigma(K,T)$ of European swaptions, we obtain:
$$
\frac{\partial V_{\text{exotic}}}{\partial \sigma(K,T)} \approx \sum_{i} w_i \frac{\partial V_{\text{european}, i}}{\partial \sigma(K,T)}
$$

This confirms that the exotic instrument’s sensitivity to market movements **can be decomposed into a weighted sum of vanilla European swaption volatilities**.

---

## **3. Practical Implementation: Vega Bucketing & Replication Portfolio**
The mapping is implemented in practice via **vega bucketing**:

1. **Compute the exotic swaption’s vega sensitivity** w.r.t. shifts in the vanilla swaption vol surface.
$$
   \text{Vega}_{\text{exotic}, i} = \frac{\partial V_{\text{exotic}}}{\partial \sigma_{\text{european}, i}}
$$

2. **Construct a portfolio of vanilla swaptions** whose vega exposure replicates the exotic swaption’s vega profile:
$$
   V_{\text{exotic}} \approx \sum_{i} w_i V_{\text{european}, i}
$$

3. **Use market prices of vanilla swaptions** to hedge and aggregate risk.

---

### **Conclusion**
Yes, there is a rigorous mathematical basis for mapping exotic swaption risk to a vanilla swaption volatility surface. The key justifications are:

1. **First-order Taylor expansion of vega sensitivities**: Exotic swaption vega can be expressed as a linear combination of vanilla swaption vegas.
2. **Dupire’s local volatility framework**: Exotic payoffs can be rewritten as expectations over vanilla swaptions.
3. **Replication arguments**: A portfolio of vanilla swaptions can be constructed to hedge exotic swaption exposures.

This methodology allows **risk aggregation** across portfolios and makes risk management more tractable. 🚀

