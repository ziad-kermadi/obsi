


# **Continuous-Time Probability Cheat Sheet for Quantitative Finance**  

Continuous-time probability is crucial in **stochastic processes, derivative pricing, risk modeling, and high-frequency trading**. This cheat sheet provides a **comprehensive guide** to **PDFs, CDFs, expectations, variances, characteristic functions, moments, joint distributions, and key probability distributions in continuous-time finance**.

---

## **1. Probability Density Function (PDF) & Cumulative Distribution Function (CDF)**
| **Concept** | **Definition** | **Formula** | **Interpretation** |
|------------|--------------|------------|------------------|
| **Probability Density Function (PDF)** | Likelihood of a continuous random variable taking a specific value | $f(x)$, where $P(a \leq X \leq b) = \int_a^b f(x) dx$ | The height of the curve represents the probability density |
| **Cumulative Distribution Function (CDF)** | Probability that $X$ takes a value **less than or equal to** $x$ | $F(x) = P(X \leq x) = \int_{-\infty}^{x} f(t) dt$ | The area under the PDF up to $x$ |

### **Key Relationship Between PDF and CDF**
$$
F(x) = \int_{-\infty}^{x} f(t) dt, \quad f(x) = \frac{d}{dx} F(x)
$$

✅ **PDF is the derivative of the CDF.**  
✅ **CDF is the integral of the PDF.**  

### **Example: Normal Distribution**
- **PDF**: $f(x) = \frac{1}{\sqrt{2\pi \sigma^2}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$
- **CDF**: No closed-form solution, computed numerically.

---

## **2. Expectation & Variance in Continuous Time**
| **Concept** | **Definition** | **Formula** | **Interpretation** |
|------------|--------------|------------|------------------|
| **Expectation $\mathbb{E}[X]$** | The mean (expected value) of $X$ | $\mathbb{E}[X] = \int x f(x) dx$ | Center of mass of $X$ |
| **Variance $\text{Var}(X)$** | Spread of $X$ around its mean | $\text{Var}(X) = \mathbb{E}[(X - \mathbb{E}[X])^2]$ | Measures risk/uncertainty |
| **Standard Deviation $\sigma$** | Square root of variance | $\sigma = \sqrt{\text{Var}(X)}$ | Used in **volatility models** |

### **Key Properties**
- **Linearity of Expectation**:  
$$
  \mathbb{E}[aX + b] = a\mathbb{E}[X] + b
$$
- **Variance of Linear Transformations**:  
$$
  \text{Var}(aX + b) = a^2 \text{Var}(X)
$$

✅ **Expectation gives fair price; variance measures risk.**

---

## **3. Moments & Characteristic Functions**
### **Moments of a Distribution**
| **Moment** | **Formula** | **Interpretation** |
|-----------|------------|------------------|
| **n-th Moment** | $\mathbb{E}[X^n]$ | Measures the "shape" of a distribution |
| **First Moment** | $\mathbb{E}[X]$ | Mean (central location) |
| **Second Moment** | $\mathbb{E}[X^2]$ | Related to variance |
| **Central Moments** | $\mathbb{E}[(X - \mathbb{E}[X])^n]$ | Measures deviations from the mean |
| **Skewness ($\gamma_1$)** | $\frac{\mathbb{E}[(X - \mathbb{E}[X])^3]}{\sigma^3}$ | Asymmetry of the distribution |
| **Kurtosis ($\gamma_2$)** | $\frac{\mathbb{E}[(X - \mathbb{E}[X])^4]}{\sigma^4}$ | Measures "fat tails" in return distributions |

### **Characteristic Function**
The **characteristic function** $\phi_X(t)$ of a random variable $X$ is:
$$
\phi_X(t) = \mathbb{E}[e^{itX}]
$$

**Properties:**
- $\phi_X(0) = 1$.
- The **Fourier transform of the PDF**.
- The **n-th derivative at 0** gives the **n-th moment**:
$$
\mathbb{E}[X^n] = (-i)^n \frac{d^n \phi_X(t)}{dt^n} \Big|_{t=0}
$$

✅ **Characteristic functions are useful in computing option prices (e.g., FFT methods).**

---

## **4. Joint, Marginal, and Conditional Distributions**
| **Concept** | **Definition** | **Formula** | **Interpretation** |
|------------|--------------|------------|------------------|
| **Joint Distribution $f_{X,Y}(x,y)$** | Probability of two variables occurring together | $P(a \leq X \leq b, c \leq Y \leq d) = \int_a^b \int_c^d f_{X,Y}(x,y) dy dx$ | Used in **multi-asset pricing models** |
| **Marginal Distribution** | Distribution of one variable, integrating out the other | $f_X(x) = \int f_{X,Y}(x,y) dy$ | Summarizes individual asset risks |
| **Conditional Distribution** | The probability of $X$ given $Y$ | $f_{X|Y}(x|y) = \frac{f_{X,Y}(x,y)}{f_Y(y)}$ | Used in **Bayesian inference** |

✅ **Joint distributions model multi-asset returns; marginal helps find individual risks; conditional is key in filtering models.**

---

## **5. Continuous-Time Probability Distributions in Quant Finance**
| **Distribution** | **PDF** | **Applications in Quant Finance** |
|----------------|------------------|--------------------------|
| **Normal $N(\mu, \sigma^2)$** | $f(x) = \frac{1}{\sqrt{2\pi\sigma^2}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$ | Black-Scholes, Gaussian models |
| **Lognormal** | $S_T = S_0 e^{(r - \frac{1}{2} \sigma^2)T + \sigma W_T}$ | Asset price modeling (GBM) |
| **Exponential** | $f(x) = \lambda e^{-\lambda x}$ | Time between jumps (Poisson processes) |
| **Poisson** | $P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}$ | Jump processes in Merton models |
| **Gamma** | $f(x) = \frac{x^{\alpha-1} e^{-x/\beta}}{\beta^\alpha \Gamma(\alpha)}$ | Modeling time between events |
| **Chi-Square** | $f(x) = \frac{x^{(k/2)-1} e^{-x/2}}{2^{k/2} \Gamma(k/2)}$ | Hypothesis testing in finance |

✅ **These distributions are used in risk modeling, option pricing, and Monte Carlo simulations.**

---

## **6. Summary Table**
| **Concept** | **Formula** | **Use Case in Quant Finance** |
|------------|------------|------------------------------|
| **PDF** | $f(x)$ | Likelihood of price movements |
| **CDF** | $F(x) = \int f(x) dx$ | Probability an event occurs by time $t$ |
| **Expectation** | $\mathbb{E}[X] = \int x f(x) dx$ | Option pricing, mean returns |
| **Variance** | $\mathbb{E}[(X - \mathbb{E}[X])^2]$ | Measures risk |
| **Characteristic Function** | $\phi_X(t) = \mathbb{E}[e^{itX}]$ | Fast Fourier Transform (FFT) methods in pricing |
| **Joint Distribution** | $f_{X,Y}(x,y)$ | Multi-asset pricing models |
| **Conditional Probability** | $P(A | B) = \frac{P(B | A) P(A)}{P(B)}$ | Bayesian risk assessment |

---

🚀 **Mastering continuous-time probability is essential for stochastic calculus, risk-neutral pricing, and high-frequency trading!** 🚀 Let me know if you need **specific applications to models like Heston, Variance Gamma, or Jump-Diffusions!**


----
---
---

# **Continuous-Time Probability Cheat Sheet for Quantitative Finance**

Continuous-time probability plays a **crucial role in Quantitative Finance**, especially in **derivatives pricing, stochastic calculus, risk modeling, and high-frequency trading**. This cheat sheet covers **stochastic processes, Ito calculus, measure theory, and common probability distributions used in pricing models**.

---

## **1. Key Concepts in Continuous-Time Probability**
| **Concept** | **Definition** | **Usage in Quantitative Finance** |
|------------|--------------|-------------------------------|
| **Stochastic Process** | A collection of random variables $X_t$ indexed by time $t$ | Models asset prices, interest rates, volatility |
| **Filtration $\mathcal{F}_t$** | A growing set of information available up to time $t$ | Defines what is "known" at each time step |
| **Martingale** | A stochastic process $X_t$ such that $\mathbb{E}[X_t | \mathcal{F}_s] = X_s$ for $s \leq t$ | No arbitrage pricing, risk-neutral measure |
| **Brownian Motion $W_t$** | A continuous-time stochastic process with independent, normally distributed increments | Drives randomness in Black-Scholes, Heston models |
| **Ito Process** | A generalization of Brownian motion with drift and volatility | Models stock price evolution |
| **Risk-Neutral Measure $\mathbb{Q}$** | A probability measure where discounted asset prices are martingales | Used for pricing derivatives |

---

## **2. Brownian Motion ($W_t$)**
### **Definition**
A **Brownian motion** (or **Wiener process**) $W_t$ satisfies:
1. $W_0 = 0$.
2. $W_t \sim N(0, t)$ (normally distributed increments).
3. Independent increments: $W_{t+s} - W_t \sim N(0, s)$.
4. Continuous paths.

### **Properties**
| **Property** | **Formula** | **Use in Finance** |
|------------|------------|------------------|
| **Mean** | $\mathbb{E}[W_t] = 0$ | Asset return expectations |
| **Variance** | $\text{Var}(W_t) = t$ | Models market randomness |
| **Scaling Property** | $a W_t \sim W_{a^2 t}$ | Scaling of time in volatility models |
| **Independent Increments** | $W_{t+s} - W_t \sim N(0, s)$ | No memory in price movements |

---

## **3. Ito Process (Generalized Brownian Motion)**
An **Ito Process** extends Brownian motion by adding **drift ($\mu$)** and **volatility ($\sigma$)**:
$$
dX_t = \mu(X_t, t) dt + \sigma(X_t, t) dW_t
$$

### **Geometric Brownian Motion (GBM)**
Used in **Black-Scholes pricing**:
$$
dS_t = r S_t dt + \sigma S_t dW_t
$$

where:
- $r$ = risk-free rate
- $\sigma$ = volatility
- $dW_t$ = Brownian motion increment

#### **Python Simulation**
```python
import numpy as np

def simulate_gbm(S0, r, sigma, T, N):
    dt = T / N
    W = np.random.randn(N) * np.sqrt(dt)
    return S0 * np.exp(np.cumsum((r - 0.5 * sigma**2) * dt + sigma * W))

S0, r, sigma, T, N = 100, 0.05, 0.2, 1, 252
simulated_path = simulate_gbm(S0, r, sigma, T, N)
```
✅ **Models asset price evolution under risk-neutral measure.**

---

## **4. Ito's Lemma (Key Tool for Derivative Pricing)**
Ito’s Lemma is the **continuous-time equivalent of the chain rule** for stochastic processes.

For $X_t$ following:
$$
dX_t = \mu dt + \sigma dW_t
$$

and function $f(X_t, t)$, Ito’s Lemma states:
$$
df = \left(\frac{\partial f}{\partial t} + \mu \frac{\partial f}{\partial X} + \frac{1}{2} \sigma^2 \frac{\partial^2 f}{\partial X^2} \right) dt + \sigma \frac{\partial f}{\partial X} dW_t
$$

### **Example: Applying Ito’s Lemma to $S_t = e^X$**
Let $X_t = \ln S_t$ and apply Ito’s Lemma:
$$
dS_t = S_t \left( \mu dt + \sigma dW_t \right)
$$

✅ **Derives the Black-Scholes formula.**

---

## **5. Stochastic Integration & Expectation**
| **Concept** | **Formula** | **Application in Finance** |
|------------|------------|----------------------------|
| **Expectation of Ito Integral** | $\mathbb{E} \left[ \int_0^T \sigma dW_t \right] = 0$ | Expected future shocks are zero |
| **Variance of Ito Integral** | $\mathbb{E} \left[ \left(\int_0^T \sigma dW_t\right)^2 \right] = \sigma^2 T$ | Used in volatility modeling |
| **Martingale Property** | $\mathbb{E}[X_t | \mathcal{F}_s] = X_s$ | No arbitrage pricing |

---

## **6. Change of Measure: Girsanov’s Theorem**
**Girsanov’s Theorem** allows switching from the **real-world measure $\mathbb{P}$** to the **risk-neutral measure $\mathbb{Q}$**.

Under $\mathbb{Q}$:
$$
dS_t = r S_t dt + \sigma S_t dW_t^{\mathbb{Q}}
$$

where:
$$
dW_t^{\mathbb{Q}} = dW_t + \theta dt
$$

and $\theta$ is the **market price of risk**:
$$
\theta = \frac{\mu - r}{\sigma}
$$

✅ **Girsanov’s theorem is fundamental for derivative pricing under risk-neutral measure.**

---

## **7. Common Probability Distributions in Finance**
| **Distribution** | **PDF / Characterization** | **Use in Quant Finance** |
|----------------|------------------|--------------------------|
| **Normal $N(\mu, \sigma^2)$** | $f(x) = \frac{1}{\sqrt{2\pi\sigma^2}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$ | Stock returns, Brownian motion |
| **Lognormal** | $S_T = S_0 e^{(r - \frac{1}{2} \sigma^2)T + \sigma W_T}$ | Asset price modeling (GBM) |
| **Poisson** | $P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}$ | Jump processes in Merton models |
| **Gamma** | $f(x) = \frac{x^{\alpha-1} e^{-x/\beta}}{\beta^\alpha \Gamma(\alpha)}$ | Modeling time between trades |
| **Exponential** | $f(x) = \lambda e^{-\lambda x}$ | Arrival times in Poisson processes |

---

## **8. Summary Table**
| **Concept** | **Formula** | **Use Case in Quant Finance** |
|------------|------------|------------------------------|
| **Brownian Motion** | $dW_t \sim N(0, dt)$ | Models randomness in asset prices |
| **Ito Process** | $dX_t = \mu dt + \sigma dW_t$ | General stochastic process |
| **Ito’s Lemma** | $df = \text{drift} + \text{volatility} dW_t$ | Deriving Black-Scholes PDE |
| **Girsanov’s Theorem** | $dW_t^{\mathbb{Q}} = dW_t + \theta dt$ | Risk-neutral measure conversion |
| **GBM Process** | $dS_t = r S_t dt + \sigma S_t dW_t$ | Pricing derivatives |

---



