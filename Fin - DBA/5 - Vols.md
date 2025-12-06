






# **Choosing Between Local Volatility vs. Stochastic Volatility Models for Pricing Exotic Derivatives**

When pricing exotic derivatives, the choice between a **Local Volatility (LV) model** and a **Stochastic Volatility (SV) model** depends on the product's sensitivity to **volatility dynamics** and the characteristics of the underlying asset. Below is a structured approach to deciding when to use each model.

---

## **1. Key Differences Between Local and Stochastic Volatility Models**

| Feature                  | Local Volatility (LV) | Stochastic Volatility (SV) |
|--------------------------|----------------------|---------------------------|
| **Volatility Behavior**  | Deterministic function of spot price and time: $\sigma_{\text{local}}(S,t)$. | Volatility follows a stochastic process (e.g., Heston, SABR). |
| **Captures Smile Dynamics?** | Yes, but only at one point in time (calibrated to today’s smile). | Yes, evolves dynamically over time, better for term structure. |
| **Path Dependency?** | No memory; future volatility depends only on $S(t)$. | Has memory; volatility follows its own stochastic path. |
| **Skew Behavior** | Imposes **static** volatility skew. | Allows **dynamics** in skew evolution. |
| **Handles Smile Evolution?** | No, does not model forward smile. | Yes, models how implied vol smile shifts over time. |
| **Repricing Vanilla Options?** | Perfectly fits vanilla options today. | Approximate fit to vanillas but captures future movements better. |

---

## **2. When to Use Local Volatility**
### ✅ **Best for Path-Independent Exotic Options**
Local Volatility is useful when:
- The exotic derivative’s price depends primarily on the current **implied volatility surface**, and **not on future volatility movements**.
- The exotic option **does not have strong path dependence**.

#### **Examples Where Local Volatility Works Well:**
- **Barrier options (e.g., knock-in/out, single-touch options)**
  - LV is good because it captures the current implied volatility surface.
  - But if barrier risk depends on volatility clustering, SV may be better.
- **Lookback options**
  - Since pricing depends only on the max/min price reached, local vol works fine.
- **Simple Asian options**
  - If averaging is short-term, LV suffices.

#### **Why LV Works Here**
- It ensures **exact replication of vanilla options** (since it is directly calibrated to today’s market-implied volatility surface).
- It gives **static arbitrage-free smiles** but **does not model volatility evolution over time**.

### ❌ **When Local Volatility Fails**
- **Exotic products that depend on future volatility paths**, such as forward-starting options.
- **Products that need stochastic skew behavior**, such as long-dated cliquets.

---

## **3. When to Use Stochastic Volatility**
### ✅ **Best for Path-Dependent Exotic Options**
Stochastic Volatility (SV) models are necessary when:
- The exotic option’s value **depends on the evolution of volatility over time**.
- **Forward-starting volatility or skew risk is important**.
- Market data shows that volatility **clusters and mean-reverts**, making a stochastic component necessary.

#### **Examples Where Stochastic Volatility is Needed:**
- **Long-dated exotics (e.g., forward-starting options, cliquets)**
  - Local volatility **cannot predict future smile movements**.
  - Stochastic vol can model how implied vol changes over time.
- **Callable derivatives and Bermudan swaptions**
  - These depend on how volatility evolves during their lifetime.
- **Quanto options**
  - Require proper modeling of stochastic correlation between FX and vol.
- **Variance and volatility swaps**
  - Directly depend on the behavior of realized volatility.

#### **Why SV Works Here**
- Captures **volatility clustering** (volatility tends to persist once high).
- Handles **forward-starting volatility effects**.
- Models **dynamics of the volatility smile** over time.

### ❌ **When Stochastic Volatility Fails**
- If the exotic derivative **does not depend on future volatility evolution**, an SV model might be overkill.
- More complex calibration; **SV models do not perfectly match today’s implied volatility surface**.

---

## **4. Rule of Thumb for Choosing the Model**
| Exotic Derivative Type  | Local Volatility | Stochastic Volatility |
|-------------------------|-----------------|----------------------|
| Barrier Options        | ✅ Works well | ❌ Not needed unless long-dated |
| Lookback Options       | ✅ Good enough | ❌ Overkill |
| Asian Options (short-term) | ✅ Good enough | ❌ Unnecessary |
| Forward-Starting Options | ❌ Fails (no forward skew) | ✅ Needed |
| Cliquets (Ratchet Options) | ❌ Fails | ✅ Needed |
| Bermudan Swaptions | ❌ Too simplistic | ✅ Needed |
| Quanto Options | ❌ Misses correlation risk | ✅ Needed |
| Variance Swaps | ❌ Cannot model realized vol clustering | ✅ Needed |

---

## **5. Summary**
- **Local Volatility is good** when pricing depends only on **current** volatility (e.g., barrier options).
- **Stochastic Volatility is needed** when future volatility movement impacts pricing (e.g., forward-starting options, Bermudans).
- **For complex exotics**, SV models are usually preferred because they incorporate **volatility clustering, forward skew, and mean reversion**.

🚀 **Final Rule:**  
👉 **If the exotic option’s value is mainly influenced by today’s implied volatility surface, use Local Volatility.**  
👉 **If the exotic option’s price depends on the evolution of volatility over time, use Stochastic Volatility.**  

Let me know if you want a specific model derivation! 🚀
