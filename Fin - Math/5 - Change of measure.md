---
title: "5 - Change of measure"
tags: [math]
type: note
status: draft
created: 2025-03-19
updated: 2026-10-05
---
# **Change of Measure in Quantitative Finance – Memory Guide & Cheat Sheet**  

Change of measure is **critical in derivatives pricing, risk-neutral valuation, and stochastic calculus**. Many people get confused about **which measure is the starting vs. target measure in the Radon-Nikodym derivative**. This guide provides **a foolproof way to remember it**, along with the usual cheat sheet content.

---

## **1. Key Idea: What Does Change of Measure Do?**
We often want to **switch from the real-world measure $\mathbb{P}$ to the risk-neutral measure $\mathbb{Q}$** because:
- Under $\mathbb{P}$, assets grow at their **expected return $\mu$**.
- Under $\mathbb{Q}$, assets grow at the **risk-free rate $r$**, simplifying derivative pricing.

### **Analogy to Currency Exchange**
Think of changing measure like **converting between currencies**:
- **Radon-Nikodym derivative** acts like an **exchange rate**.
- If you’re moving from USD to EUR, you need **EUR/USD**.
- If you’re moving from $\mathbb{P}$ to $\mathbb{Q}$, you need **$d\mathbb{Q}/d\mathbb{P}$**.

---

## **2. Remembering the Radon-Nikodym Derivative Form**
The **Radon-Nikodym derivative (likelihood ratio process)** tells us how to adjust probabilities when changing measure.
$$
\frac{d\mathbb{Q}}{d\mathbb{P}} = Z_T = e^{-\theta W_T^{\mathbb{P}} - \frac{1}{2} \theta^2 T}
$$

where:
- **Numerator = Target Measure** (where you are going)
- **Denominator = Starting Measure** (where you are coming from)
- **$\theta = \frac{\mu - r}{\sigma}$** is the **market price of risk**.

### **Memory Trick: “WHERE YOU GO ON TOP”**
$$
\frac{\text{Target Measure}}{\text{Starting Measure}}
$$

✅ If you’re going **from $\mathbb{P}$ to $\mathbb{Q}$**, write:
$$
\frac{d\mathbb{Q}}{d\mathbb{P}}
$$

✅ If you’re going **from $\mathbb{Q}$ to $\mathbb{P}$**, write:
$$
\frac{d\mathbb{P}}{d\mathbb{Q}}
$$

---

## **3. Girsanov’s Theorem (How We Change the Brownian Motion)**
The key tool for changing measure is **Girsanov’s Theorem**, which modifies the **drift** of a stochastic process.

For a **stock price** under the **real-world measure $\mathbb{P}$**:
$$
dS_t = \mu S_t dt + \sigma S_t dW_t^{\mathbb{P}}
$$

We define the **new Brownian motion under $\mathbb{Q}$**:
$$
dW_t^{\mathbb{Q}} = dW_t^{\mathbb{P}} + \theta dt
$$

where:
$$
\theta = \frac{\mu - r}{\sigma}
$$

Thus, under $\mathbb{Q}$, the stock price follows:
$$
dS_t = r S_t dt + \sigma S_t dW_t^{\mathbb{Q}}
$$

✅ **This removes the risk premium $\mu - r$ and replaces it with the risk-free rate $r$, making pricing easier.**

---

## **4. How to Use the Radon-Nikodym Derivative**
The Radon-Nikodym derivative **relates expectations under different measures**:
$$
\mathbb{E}^{\mathbb{Q}}[X] = \mathbb{E}^{\mathbb{P}}[X Z_T]
$$

- If you **know expectations under $\mathbb{P}$** but want to convert to $\mathbb{Q}$, multiply by $Z_T$.
- If you **know expectations under $\mathbb{Q}$** and want to go back to $\mathbb{P}$, multiply by $1/Z_T$.

### **Example: Expected Stock Price at $T$**
1. Under $\mathbb{P}$:
$$
   \mathbb{E}^{\mathbb{P}}[S_T] = S_0 e^{\mu T}
$$

2. Convert to $\mathbb{Q}$:
$$
   \mathbb{E}^{\mathbb{Q}}[S_T] = S_0 e^{rT}
$$

✅ **Under $\mathbb{Q}$, expected growth is risk-free.**

---

## **5. Applications of Change of Measure in Quant Finance**
### **(i) Black-Scholes Derivative Pricing**
- Stock price follows **Geometric Brownian Motion (GBM)** under $\mathbb{P}$:
$$
  dS_t = \mu S_t dt + \sigma S_t dW_t^{\mathbb{P}}
$$

- Convert to $\mathbb{Q}$:
$$
  dS_t = r S_t dt + \sigma S_t dW_t^{\mathbb{Q}}
$$

- Compute option price as an expectation under $\mathbb{Q}$:
$$
  C_0 = e^{-rT} \mathbb{E}^{\mathbb{Q}}[\max(S_T - K, 0)]
$$

✅ **Risk-neutral measure makes pricing easier!**

---

### **(ii) Interest Rate Models (Forward Measure)**
To price bonds and interest rate derivatives, we change from $\mathbb{Q}$ (risk-neutral) to **forward measure $\mathbb{Q}^T$**.

| **Measure** | **Numeraire** | **Used For** |
|------------|-------------|-------------|
| **Risk-Neutral ($\mathbb{Q}$)** | Bank account $B_t = e^{rt}$ | Equity and option pricing |
| **Forward Measure ($\mathbb{Q}^T$)** | Bond price $P(0,T)$ | Interest rate derivatives |

✅ **Each measure simplifies pricing for different asset classes.**

---

## **6. Summary Table**
| **Concept** | **Formula** | **Memory Trick** |
|------------|------------|----------------|
| **Radon-Nikodym Derivative** | $\frac{d\mathbb{Q}}{d\mathbb{P}} = e^{-\theta W_T^{\mathbb{P}} - \frac{1}{2} \theta^2 T}$ | **"Where you go on top"** |
| **Girsanov’s Theorem** | $dW_t^{\mathbb{Q}} = dW_t^{\mathbb{P}} + \theta dt$ | **Changes drift of Brownian motion** |
| **Risk-Neutral Expectation** | $\mathbb{E}^{\mathbb{Q}}[X] = \mathbb{E}^{\mathbb{P}}[X Z_T]$ | **Multiply by $Z_T$ to convert** |
| **Stock Price Under $\mathbb{P}$** | $dS_t = \mu S_t dt + \sigma S_t dW_t^{\mathbb{P}}$ | **Real-world dynamics** |
| **Stock Price Under $\mathbb{Q}$** | $dS_t = r S_t dt + \sigma S_t dW_t^{\mathbb{Q}}$ | **Risk-neutral dynamics** |

---

## **7. Final Takeaways**
✅ **Always put the target measure on top in $\frac{d\mathbb{Q}}{d\mathbb{P}}$** (Use the "Where You Go On Top" trick).  
✅ **Girsanov’s Theorem shifts the drift** to make assets grow at the risk-free rate under $\mathbb{Q}$.  
✅ **Monte Carlo simulations use risk-neutral measure** because it simplifies expected payoffs.  
✅ **Different measures are used for different asset classes** (equities = $\mathbb{Q}$, bonds = $\mathbb{Q}^T$).  

🚀 **Understanding change of measure is key for option pricing, risk management, and interest rate modeling! Let me know if you need deeper examples!** 🚀é&"