





----------
-----
-----
Here’s a **detailed Mathematical Finance Cheat Sheet** with **expanded explanations, more intuition, and additional coverage of continuous probability**. This should provide **both conceptual understanding and practical formulas** to prepare you thoroughly for a **Quant Strat interview**.

---

# **Mathematical Finance Cheat Sheet for Quant Interviews**

## **1. Probability & Statistics**

### **Measure-Theoretic Probability**
Probability is formalized using a **probability space** $(\Omega, \mathcal{F}, \mathbb{P})$, where:
- $\Omega$ is the sample space (all possible outcomes).
- $\mathcal{F}$ is the sigma-algebra (events we can assign probabilities to).
- $\mathbb{P}$ is the probability measure.

#### **Expectation as an Integral**
For a **random variable** $X$, the expectation is:
$$
\mathbb{E}[X] = \int_{\Omega} X(\omega) d\mathbb{P}(\omega)
$$
where $d\mathbb{P}$ denotes integration w.r.t. the probability measure.

#### **Conditional Expectation**
Given a sub-sigma algebra $\mathcal{G}$, the conditional expectation of $X$ given $\mathcal{G}$, denoted $\mathbb{E}[X | \mathcal{G}]$, is the **best $L^2$ estimator** satisfying:
$$
\int_A \mathbb{E}[X | \mathcal{G}] d\mathbb{P} = \int_A X d\mathbb{P}, \quad \forall A \in \mathcal{G}
$$

### **Continuous Distributions**
#### **Normal Distribution $\mathcal{N}(\mu, \sigma^2)$**
- **PDF**: 
$$
  f(x) = \frac{1}{\sqrt{2\pi \sigma^2}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$
- **Expectation**: $\mathbb{E}[X] = \mu$
- **Variance**: $\mathbb{V}ar(X) = \sigma^2$

#### **Exponential Distribution $\text{Exp}(\lambda)$**
- **PDF**: $f(x) = \lambda e^{-\lambda x}, \quad x > 0$
- **Expectation**: $\mathbb{E}[X] = \frac{1}{\lambda}$
- **Memoryless Property**: $P(X > s+t | X > s) = P(X > t)$

#### **Poisson Process $N(t) \sim \text{Poisson}(\lambda t)$**
- $P(N(t) = k) = \frac{(\lambda t)^k e^{-\lambda t}}{k!}$
- **Interarrival times**: $X_i \sim \text{Exp}(\lambda)$

### **Martingales**
A stochastic process $M_t$ is a **martingale** if:
$$
\mathbb{E}[M_t | \mathcal{F}_s] = M_s, \quad s \leq t
$$
i.e., the best estimate of future values is the current value.

---

## **2. Stochastic Calculus**

### **Brownian Motion $W_t$**
A **Wiener process** satisfies:
1. $W_0 = 0$
2. $W_t \sim \mathcal{N}(0, t)$ (Normally distributed increments)
3. **Independent** increments
4. **Continuous paths** (but nowhere differentiable)

### **Itô’s Lemma**
For a function $f(X_t, t)$, where $dX_t = \mu dt + \sigma dW_t$, Itô’s Lemma states:
$$
df = \left( \frac{\partial f}{\partial t} + \mu \frac{\partial f}{\partial x} + \frac{1}{2} \sigma^2 \frac{\partial^2 f}{\partial x^2} \right) dt + \sigma \frac{\partial f}{\partial x} dW_t
$$

### **Stochastic Differential Equations (SDEs)**
- **Geometric Brownian Motion (GBM)**:
$$
  dS_t = \mu S_t dt + \sigma S_t dW_t
$$
  - Solution: $S_t = S_0 e^{(\mu - \frac{1}{2} \sigma^2)t + \sigma W_t}$

---

## **3. Black-Scholes Model**
- **PDE**:
$$
  \frac{\partial V}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + r S \frac{\partial V}{\partial S} - r V = 0
$$
- **Solution (Call Option)**:
$$
  C(S, t) = S N(d_1) - e^{-r(T-t)} K N(d_2)
$$
  where
$$
  d_1 = \frac{\ln(S/K) + (r + \frac{1}{2} \sigma^2)(T-t)}{\sigma \sqrt{T-t}}, \quad d_2 = d_1 - \sigma \sqrt{T-t}
$$

---

## **4. Greeks**
| Greek | Definition |
|---|---|
| **Delta** $\frac{\partial C}{\partial S}$ | Sensitivity to spot price changes |
| **Gamma** $\frac{\partial^2 C}{\partial S^2}$ | Convexity of option value |
| **Theta** $\frac{\partial C}{\partial t}$ | Time decay |
| **Vega** $\frac{\partial C}{\partial \sigma}$ | Sensitivity to volatility |
| **Rho** $\frac{\partial C}{\partial r}$ | Sensitivity to interest rate |

---

## **5. Numerical Methods**
### **Monte Carlo Simulation**
Estimate $\mathbb{E}[f(S_T)]$ via:
1. Simulate $N$ paths of $S_T$.
2. Compute $f(S_T)$ for each path.
3. Average and discount back.

### **Finite Difference Methods**
- Explicit: $V_{i}^{n+1} = a V_{i-1}^{n} + b V_{i}^{n} + c V_{i+1}^{n}$
- Implicit: Solve a linear system.

---

## **6. Portfolio Optimization (MPT)**
- **Mean-Variance Optimization**:
$$
  \min_{\omega} \frac{1}{2} \omega^T \Sigma \omega \quad \text{s.t.} \quad \mu^T \omega = r_f
$$
- **Sharpe Ratio**:
$$
  SR = \frac{\mathbb{E}[r_p] - r_f}{\sigma_p}
$$

---

## **7. Market Models**
### **Heston Model (Stochastic Volatility)**
$$
dS_t = \mu S_t dt + \sqrt{V_t} S_t dW_t^S
$$
$$
dV_t = \kappa (\theta - V_t) dt + \sigma \sqrt{V_t} dW_t^V
$$

### **SABR Model**
$$
dS_t = \sigma_t S_t^\beta dW_t, \quad d\sigma_t = \nu \sigma_t dZ_t
$$

---

This **detailed cheat sheet** expands on the key topics, providing **intuition and precise formulas** for a **Quant Strat Interview**. Let me know if you want **even deeper derivations, examples, or further refinements! 🚀**