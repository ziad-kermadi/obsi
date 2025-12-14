




------
----
---


# **Monte Carlo Pricing Cheat Sheet for Quantitative Finance**

Monte Carlo methods are **crucial for pricing derivatives**, especially in **complex, path-dependent options and stochastic models**. This cheat sheet provides a **step-by-step guide** on **simulation, discretization, variance reduction, and advanced methods like Longstaff-Schwartz for American options**.

---

## **1. What is Monte Carlo Pricing?**
Monte Carlo (MC) methods **simulate random paths of an asset price** and compute the expected payoff under **risk-neutral measure**.
$$
V_0 = e^{-rT} \mathbb{E}^{\mathbb{Q}}[\text{Payoff}]
$$

where:
- $r$ = risk-free rate
- $T$ = time to maturity
- $\mathbb{E}^{\mathbb{Q}}[\cdot]$ = expectation under the **risk-neutral measure**
- **Simulations approximate the expectation**

### **When to Use Monte Carlo?**
| **Use Case**                                    | **Why Monte Carlo?**                     |
| ----------------------------------------------- | ---------------------------------------- |
| **Path-Dependent Options** (Asian, Barrier)     | Can't solve analytically                 |
| **Multi-Asset Options** (Basket, Spread)        | Multi-dimensional nature                 |
| **Stochastic Volatility Models** (Heston, SABR) | Non-closed form solutions                |
| **American Options**                            | Need early exercise (Longstaff-Schwartz) |

---

## **2. Step-by-Step Monte Carlo Pricing**
### **Step 1: Simulating Asset Prices Using Discretization**
Asset price follows a **stochastic differential equation (SDE)** under **Geometric Brownian Motion (GBM)**:
$$
dS_t = \mu S_t dt + \sigma S_t dW_t
$$

Under the **risk-neutral measure** ($\mu = r$):
$$
dS_t = r S_t dt + \sigma S_t dW_t
$$

### **Euler-Maruyama Discretization**
$$
S_{t+\Delta t} = S_t e^{(r - \frac{1}{2} \sigma^2) \Delta t + \sigma \sqrt{\Delta t} Z}
$$

where:
- $Z \sim N(0,1)$ (standard normal variable)
- $\Delta t = T/N$ (time step)

#### **Python Implementation**
```python
import numpy as np

def monte_carlo_gbm(S0, r, sigma, T, N, M):
    dt = T / N  # Time step
    paths = np.zeros((M, N+1))
    paths[:, 0] = S0

    for t in range(1, N+1):
        Z = np.random.randn(M)  # Normal random numbers
        paths[:, t] = paths[:, t-1] * np.exp((r - 0.5 * sigma**2) * dt + sigma * np.sqrt(dt) * Z)

    return paths

# Parameters
S0, r, sigma, T, N, M = 100, 0.05, 0.2, 1, 252, 10000
paths = monte_carlo_gbm(S0, r, sigma, T, N, M)
```

✅ **This simulates $M$ paths over $N$ time steps using risk-neutral GBM.**

---

### **Step 2: Compute Option Payoffs**
For a **European call option** with strike $K$:
$$
\text{Payoff} = \max(S_T - K, 0)
$$

Estimate the price using:
$$
V_0 = e^{-rT} \frac{1}{M} \sum_{i=1}^{M} \text{Payoff}_i
$$

#### **Python Implementation**
```python
K = 100  # Strike price
payoffs = np.maximum(paths[:, -1] - K, 0)
option_price = np.exp(-r * T) * np.mean(payoffs)
print("Monte Carlo Option Price:", option_price)
```

✅ **Averaging the discounted payoffs gives the option price.**

---

## **3. Variance Reduction Techniques**
Monte Carlo has **slow convergence** ($O(\frac{1}{\sqrt{M}})$). Use **variance reduction** to improve efficiency.

### **(i) Antithetic Variates**
Use **negatively correlated paths**:
$$
S_t^+ = S_0 e^{(r - \frac{1}{2} \sigma^2) t + \sigma W_t}
$$
$$
S_t^- = S_0 e^{(r - \frac{1}{2} \sigma^2) t - \sigma W_t}
$$

Average both paths to **reduce variance**.

#### **Python Implementation**
```python
Z = np.random.randn(M // 2)
paths1 = S0 * np.exp((r - 0.5 * sigma**2) * T + sigma * np.sqrt(T) * Z)
paths2 = S0 * np.exp((r - 0.5 * sigma**2) * T - sigma * np.sqrt(T) * Z)
payoffs = np.maximum(paths1 - K, 0) + np.maximum(paths2 - K, 0)
option_price = np.exp(-r * T) * np.mean(payoffs) / 2
print("Antithetic Variates Price:", option_price)
```

---

### **(ii) Control Variates**
Instead of pricing **just the option**, use a related instrument with a **known analytical price** (e.g., GBM expectation):
$$
\mathbb{E}[S_T] = S_0 e^{rT}
$$

Adjust the estimator:
$$
\tilde{V} = V - b(S_T^{MC} - \mathbb{E}[S_T])
$$

#### **Python Implementation**
```python
ST_mc = paths[:, -1]
b = np.cov(ST_mc, payoffs)[0,1] / np.var(ST_mc)
adjusted_payoffs = payoffs - b * (ST_mc - S0 * np.exp(r * T))
option_price = np.exp(-r * T) * np.mean(adjusted_payoffs)
print("Control Variates Price:", option_price)
```

✅ **Drastically improves convergence.**

---

## **4. Pricing American Options - Longstaff-Schwartz Algorithm**
American options allow **early exercise**, making Monte Carlo tricky.

### **Step 1: Backward Induction via Least-Squares Regression**
- Simulate multiple **Monte Carlo paths**.
- At each time step, fit a **regression model** to approximate the **continuation value**.
- Compare **exercise value vs. continuation value**.
- If **exercise is optimal**, stop and use that value.

#### **Python Implementation**
```python
from sklearn.linear_model import LinearRegression

def longstaff_schwartz(S0, r, sigma, K, T, N, M):
    dt = T / N
    paths = monte_carlo_gbm(S0, r, sigma, T, N, M)
    payoffs = np.maximum(K - paths[:, -1], 0)
    
    for t in range(N-1, 0, -1):
        in_the_money = paths[:, t] < K
        X = paths[in_the_money, t].reshape(-1, 1)
        Y = payoffs[in_the_money] * np.exp(-r * dt)
        
        model = LinearRegression().fit(X, Y)
        continuation_value = model.predict(X)
        
        exercise_value = K - X.flatten()
        exercise_now = exercise_value > continuation_value
        payoffs[in_the_money] = np.where(exercise_now, exercise_value, payoffs[in_the_money])
    
    return np.exp(-r * dt) * np.mean(payoffs)

print("American Put Price:", longstaff_schwartz(S0, r, sigma, K, T, N, M))
```

✅ **Backward induction efficiently prices American options.**

---

## **5. Summary Table**
| **Step** | **Formula** | **Purpose** |
|---------|------------|------------|
| **Simulating GBM** | $S_t = S_0 e^{(r - \frac{1}{2} \sigma^2) t + \sigma W_t}$ | Generate asset price paths |
| **Estimating Option Price** | $V_0 = e^{-rT} \mathbb{E}[\text{Payoff}]$ | Compute fair value |
| **Antithetic Variates** | $S_t^+$ and $S_t^-$ | Reduces variance |
| **Control Variates** | $\tilde{V} = V - b(S_T^{MC} - \mathbb{E}[S_T])$ | Improves convergence |
| **Longstaff-Schwartz** | Regression on continuation value | Price American options |

---

🚀 **Monte Carlo is crucial for pricing complex derivatives.** Mastering **variance reduction** and **Longstaff-Schwartz** makes a huge difference in Quantitative Finance! 🚀



-------
---------
--------------

# **Numerical Methods in Quantitative Finance – In-Depth Cheat Sheet**  
This cheat sheet dives deep into **numerical methods** used for pricing, risk management, and model calibration in **banks and hedge funds**. Special focus is given to **Monte Carlo variance reduction techniques** such as **Antithetic Variates and Importance Sampling**.  

---

# **1. Monte Carlo Methods**  
Monte Carlo methods are widely used in banks for:  
✅ **Derivative Pricing** (especially exotic options).  
✅ **Risk Management** (VaR, Expected Shortfall).  
✅ **Portfolio Optimization** (estimating risk-adjusted returns).  

---

## **1.1 Euler-Maruyama Method for Simulating Stochastic Processes**  
Banks need to **simulate financial assets** modeled as **stochastic differential equations (SDEs)**:  
$$
dS_t = \mu S_t dt + \sigma S_t dW_t
$$

**Euler-Maruyama Approximation:**  
For small time step $\Delta t$, the process can be discretized as:
$$
S_{t+\Delta t} = S_t + \mu S_t \Delta t + \sigma S_t \sqrt{\Delta t} Z_t
$$

where $Z_t \sim \mathcal{N}(0,1)$.  

✅ Used for **path-dependent options** (e.g., Asian, Barrier).  
✅ Used for **Monte Carlo Greeks estimation** (derivatives of option prices).  

---

## **1.2 Monte Carlo Option Pricing**  
We estimate the option price as:
$$
V = e^{-rT} \mathbb{E}[f(S_T)]
$$

#### **Steps**:
1. **Simulate paths** of $S_T$ using GBM.
2. **Compute payoffs** $f(S_T)$ for each path.
3. **Estimate price as the discounted mean**:
$$
V \approx e^{-rT} \frac{1}{N} \sum_{i=1}^{N} f(S_T^{(i)})
$$

4. **Estimate standard error**:
$$
\text{SE} = \frac{\sigma}{\sqrt{N}}
$$

---

## **1.3 Variance Reduction Techniques in Monte Carlo**
Monte Carlo simulations are computationally expensive. To **increase efficiency**, banks use **variance reduction techniques**:

### **A. Antithetic Variates**  
- **Idea**: If $X$ is an estimator, construct a negatively correlated estimator $X'$ so that their average has lower variance.  
- **Implementation**:
  1. Generate $Z_1 \sim \mathcal{N}(0,1)$ and its **negative counterpart** $Z_2 = -Z_1$.
  2. Compute two correlated paths:
$$
     S_T^{(1)} = S_0 e^{(r - \frac{1}{2} \sigma^2)T + \sigma W_T}
$$
$$
     S_T^{(2)} = S_0 e^{(r - \frac{1}{2} \sigma^2)T - \sigma W_T}
$$
  3. Average the payoffs:
$$
     V = \frac{1}{2} (f(S_T^{(1)}) + f(S_T^{(2)}))
$$

✅ **Why it Works?**  
Since $S_T^{(1)}$ and $S_T^{(2)}$ are negatively correlated, their variance is **reduced**.

✅ **Where Banks Use It?**  
- **Pricing European options** (where symmetry in distributions helps).  
- **Path-dependent options** (e.g., Lookback, Asian options).  

---

### **B. Importance Sampling**  
- **Idea**: If the rare events contribute the most to expected value, focus on those!  
- **Used For**:  
  - **Pricing deep out-of-the-money options** (where rare events dominate).  
  - **Estimating tail risk (VaR, Expected Shortfall)**.  

#### **Mathematical Formulation**  
In Monte Carlo, we estimate:
$$
I = \mathbb{E}[h(X)] = \int h(x) f(x) dx
$$

where $f(x)$ is the original density. Instead of sampling from $f(x)$, we use a new density $g(x)$, rewriting:
$$
I = \int h(x) \frac{f(x)}{g(x)} g(x) dx
$$

We now sample from $g(x)$, so that:
$$
I \approx \frac{1}{N} \sum_{i=1}^{N} h(X_i) \frac{f(X_i)}{g(X_i)}
$$

✅ **How Banks Use It?**  
- **Deep OTM options**: Sampling more from extreme price movements improves accuracy.  
- **Stress testing**: Banks use importance sampling to model rare crisis events.  

---

# **2. Finite Difference Methods (FDM)**
Finite difference methods solve PDEs numerically. The **Black-Scholes PDE**:
$$
\frac{\partial V}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + r S \frac{\partial V}{\partial S} - r V = 0
$$

### **Explicit vs. Implicit Schemes**
| Method | Pros | Cons |
|---|---|---|
| **Explicit** $V_i^{n+1} = a V_{i-1}^n + b V_i^n + c V_{i+1}^n$ | Simple | Unstable unless $\Delta t$ is small |
| **Implicit** $A V^{n+1} = V^n$ | Unconditionally stable | Requires solving a linear system |
| **Crank-Nicolson** $A V^{n+1} = B V^n$ | Second-order accurate | More computation needed |

✅ **Banks Use FDM for:**  
- **Pricing vanilla options (calls, puts).**  
- **Risk sensitivities (Greeks).**  

---

# **3. Binomial Tree Method**
Used in **pricing American options** where **early exercise** is possible.

### **Steps**
1. Construct a **binomial tree**:  
   - $S_u = S e^{\sigma \sqrt{\Delta t}}, \quad S_d = S e^{-\sigma \sqrt{\Delta t}}$
   - Risk-neutral probability:
$$
     p = \frac{e^{r \Delta t} - d}{u - d}
$$
2. Compute payoffs at maturity.
3. Work **backwards** computing early exercise values.

✅ **Used in:**  
- **American options pricing**  
- **Convertible bonds pricing**  

---

# **4. Optimization Techniques in Banks**
Used in **portfolio management**, **hedging**, and **model calibration**.

### **Gradient Descent (GD)**
$$
x_{n+1} = x_n - \eta \nabla f(x_n)
$$

✅ Used in **portfolio optimization** (adjusting asset weights).  

### **Newton’s Method**
$$
x_{n+1} = x_n - H^{-1} \nabla f(x_n)
$$

✅ Used in **calibrating volatility models** (e.g., Heston model).  

---

### **Final Summary**
| Method | Use in Banks |
|---|---|
| **Monte Carlo** | Pricing exotic derivatives |
| **Antithetic Variates** | Variance reduction in option pricing |
| **Importance Sampling** | Rare event modeling (OTM options, VaR) |
| **Finite Difference** | Solving Black-Scholes PDE for vanilla options |
| **Binomial Trees** | American options, Convertible bonds |
| **Optimization** | Portfolio management, Model calibration |

------------------
--------------------
-----------------------
# **Comparison of Newton-Raphson, Bisection, and Brent's Method**

Root-finding algorithms are essential for solving nonlinear equations, widely used in **quantitative finance** for applications like **implied volatility calculation, bond pricing, and model calibration**. Three commonly used methods are:  

1. **Newton-Raphson Method** – Fast and efficient but requires a derivative.
2. **Bisection Method** – Robust but slow.
3. **Brent’s Method** – Hybrid approach that combines the best of both.

### **1. Newton-Raphson Method**
✅ **Fast (quadratic convergence) but requires derivatives.**  

#### **Algorithm**
$$
x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}
$$
- Uses **tangent line approximation** to find the root iteratively.
- Converges **quadratically** (error reduces as $\epsilon_{n+1} \approx C (\epsilon_n)^2$).
- Requires the **first derivative** $f'(x)$, which can be expensive to compute.

#### **Pros:**
✅ **Fast convergence** (if initial guess is close).  
✅ **Used in implied volatility & risk-neutral pricing**.  
✅ **Good for well-behaved smooth functions**.  

#### **Cons:**
❌ **Fails if $f'(x)$ is zero or near-zero** (causes division errors).  
❌ **Can diverge if initial guess is poor**.  
❌ **Not guaranteed to work for non-smooth or discontinuous functions**.  

#### **Use Cases in Finance:**
- **Finding implied volatility** in Black-Scholes.
- **Calibrating stochastic models** (e.g., Heston model).
- **Solving for bond yield to maturity (YTM)**.

---

### **2. Bisection Method**
✅ **Slow but guaranteed to converge.**  

#### **Algorithm**
1. **Choose two points** $a$ and $b$ such that $f(a) \cdot f(b) < 0$ (i.e., the root is between them).
2. Compute the **midpoint**:
$$
   c = \frac{a + b}{2}
$$
3. If $f(c) = 0$, return $c$. Otherwise, update $a$ or $b$ so that the root remains bracketed.
4. Repeat until convergence.

#### **Pros:**
✅ **Always converges** if $f(x)$ is continuous.  
✅ **Does not require derivatives**.  
✅ **Useful for root bracketing problems**.  

#### **Cons:**
❌ **Converges linearly** (slow compared to Newton-Raphson).  
❌ **Inefficient for high-precision results**.  

#### **Use Cases in Finance:**
- **Solving for bond YTM** (where a rate must be found that satisfies a bond price equation).
- **Finding break-even points in risk-neutral pricing**.

---

### **3. Brent’s Method**
✅ **Best of both worlds – robust like Bisection but fast like Newton.**  

Brent’s method is a **hybrid approach** combining:
- **Bisection Method** (guarantees convergence).
- **Secant Method** (uses past two estimates for better speed).
- **Inverse Quadratic Interpolation** (accelerates convergence when function is smooth).

#### **Algorithm Steps**
1. Start with two points $a, b$ where $f(a) \cdot f(b) < 0$.
2. Use **Secant Method** to get a new estimate:
$$
   x = b - f(b) \frac{b - a}{f(b) - f(a)}
$$
3. If Secant is ineffective, use **Inverse Quadratic Interpolation** for a better estimate.
4. If the method isn’t converging fast, **fall back to Bisection** to guarantee convergence.
5. Repeat until $|f(x)|$ is within tolerance.

#### **Pros:**
✅ **Fast (almost as fast as Newton-Raphson)**.  
✅ **Always converges (unlike Newton-Raphson)**.  
✅ **Does not require derivatives**.  

#### **Cons:**
❌ **More complex than Bisection or Newton-Raphson**.  
❌ **May perform slightly worse if function is well-behaved** (compared to Newton).  

#### **Use Cases in Finance:**
- **Solving nonlinear equations when derivatives are unavailable.**  
- **Finding implied volatility (alternative to Newton-Raphson).**  
- **Used in numerical libraries like SciPy for robust root-finding.**  

---

## **4. Comparing Newton-Raphson, Bisection, and Brent’s Method**
| Method | Speed | Requires Derivative? | Always Converges? | Best Use Case |
|---|---|---|---|---|
| **Newton-Raphson** | **Fast (Quadratic convergence)** | ✅ Yes | ❌ No | **Implied volatility, risk-neutral pricing, bond YTM** |
| **Bisection** | **Slow (Linear convergence)** | ❌ No | ✅ Yes | **Robust root finding, bond pricing, risk-free rate matching** |
| **Brent’s Method** | **Fast (Super-linear convergence)** | ❌ No | ✅ Yes | **Hybrid approach, used in SciPy, robust financial calculations** |

---

### **5. Final Thoughts**
- **Use Newton-Raphson when derivatives are available and function is smooth.**
- **Use Bisection when you need guaranteed convergence but speed is not a priority.**
- **Use Brent’s Method when you need a robust method that works in all cases.**

-------------
-------------
-----------------
# **Finite Difference Methods – In-Depth Step-by-Step Explanation with Intuition**

## **1. Introduction to Finite Difference Methods**
Finite Difference Methods (FDM) are **numerical techniques used to solve differential equations**. In **quantitative finance**, they are used to solve **Partial Differential Equations (PDEs)**, particularly for **option pricing models like Black-Scholes**.

✅ **Common applications in finance:**  
- Pricing **European and American options**.  
- Solving the **Black-Scholes PDE**.  
- Computing **option Greeks** (sensitivities).  
- Risk assessment and **hedging strategy simulations**.  

---

## **2. Understanding the Black-Scholes PDE**
The **Black-Scholes equation** is a fundamental PDE in finance:
$$
\frac{\partial V}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + r S \frac{\partial V}{\partial S} - r V = 0
$$

where:
- $V(S,t)$ is the option price at asset price $S$ and time $t$.
- $\sigma$ is the volatility.
- $r$ is the risk-free interest rate.
- $S$ is the underlying asset price.

💡 **Key challenge**: There is no closed-form solution for **American options** or complex derivatives → **we must use numerical methods like Finite Differences!**

---

## **3. Finite Difference Approximation – The Core Idea**
### **How do we approximate derivatives numerically?**
The **finite difference approach** approximates derivatives using **Taylor Series Expansions**.

### **3.1 First-Order Derivatives**
Consider a function $V(S,t)$. The **first derivative** can be approximated in three ways:

#### **(1) Forward Difference Approximation**
$$
\frac{\partial V}{\partial S} \approx \frac{V(S + \Delta S, t) - V(S, t)}{\Delta S}
$$
✅ **Easy to compute**  
❌ **Less accurate (only first-order accurate)**  
❌ **Can be unstable**  

#### **(2) Backward Difference Approximation**
$$
\frac{\partial V}{\partial S} \approx \frac{V(S, t) - V(S - \Delta S, t)}{\Delta S}
$$
✅ **Better stability than forward difference**  
❌ **Still first-order accurate**  

#### **(3) Central Difference Approximation**
$$
\frac{\partial V}{\partial S} \approx \frac{V(S + \Delta S, t) - V(S - \Delta S, t)}{2 \Delta S}
$$
✅ **More accurate (second-order accurate)**  
✅ **Used in most FDM schemes**  

### **3.2 Second-Order Derivatives**
We approximate the **second derivative** using:
$$
\frac{\partial^2 V}{\partial S^2} \approx \frac{V(S + \Delta S, t) - 2V(S, t) + V(S - \Delta S, t)}{\Delta S^2}
$$

✅ **Second-order accurate**  
✅ **Essential for solving PDEs like Black-Scholes**  

---

## **4. Discretizing the Black-Scholes PDE**
We replace derivatives with finite differences to **convert the PDE into algebraic equations**.

### **Discretization Grid**
We define a **discrete grid** in time and asset price:
$$
S_i = i \Delta S, \quad t_n = n \Delta t
$$

where:
- $\Delta S$ is the **step size for stock prices**.
- $\Delta t$ is the **step size for time**.

**Key idea:** We approximate $V(S,t)$ using a **grid** of values $V_{i}^{n}$, where:
- $i$ indexes **stock price steps**.
- $n$ indexes **time steps**.

---

## **5. Types of Finite Difference Methods**
There are **three main FDM approaches** for solving PDEs:

| Method | Stability | Accuracy | Computational Cost |
|--------|----------|----------|-------------------|
| **Explicit** | Low | First-Order | Low |
| **Implicit** | High | First-Order | High |
| **Crank-Nicolson** | High | Second-Order | Medium |

---

## **5.3 Crank-Nicolson Method (Best of Both Worlds)**
Crank-Nicolson is a **semi-implicit scheme** that combines the best of the **explicit and implicit methods**.

### **Why Crank-Nicolson?**
✅ **Higher accuracy** (second-order accurate in time).  
✅ **Unconditionally stable** (like implicit FDM).  
✅ **Used in industry for pricing vanilla options**.  

---

### **Step-by-Step Derivation**
We take the **average of explicit and implicit schemes**:
$$
\frac{V_i^{n+1} - V_i^n}{\Delta t} = \frac{1}{2} \left( L V^n + L V^{n+1} \right)
$$

where $L$ is the **Black-Scholes differential operator**:
$$
L V = \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + r S \frac{\partial V}{\partial S} - r V
$$

Rewriting in discrete form:
$$
\frac{V_i^{n+1} - V_i^n}{\Delta t} = \frac{1}{2} \left( L_i^n V^n + L_i^{n+1} V^{n+1} \right)
$$

This leads to a **tridiagonal system**:
$$
A V^{n+1} = B V^n
$$

where:
$$
A = \left( 1 + \frac{r \Delta t}{2} \right) I - \frac{\sigma^2 S^2 \Delta t}{4 \Delta S^2} D^2 - \frac{r S \Delta t}{4 \Delta S} D^1
$$
$$
B = \left( 1 - \frac{r \Delta t}{2} \right) I + \frac{\sigma^2 S^2 \Delta t}{4 \Delta S^2} D^2 + \frac{r S \Delta t}{4 \Delta S} D^1
$$

where:
- $D^2$ is the second difference matrix (for $\frac{\partial^2 V}{\partial S^2}$).
- $D^1$ is the first difference matrix (for $\frac{\partial V}{\partial S}$).

### **Solving the System**
1. **Construct matrices $A$ and $B$.**
2. **Solve the tridiagonal system $A V^{n+1} = B V^n$ using the Thomas Algorithm** (fast solver for tridiagonal matrices).
3. **Iterate backward in time** until $V_0^0$ (initial option price) is obtained.

---

### **Advantages of Crank-Nicolson**
✅ **Second-order accuracy** in both space and time.  
✅ **Unconditionally stable**, meaning **larger $\Delta t$ can be used** without instability.  
✅ **Works well for European options**.  
✅ **Used in real-world financial software**.

---

### **Limitations**
❌ **Oscillations for American options** → Must be combined with **Projective Successive Over-Relaxation (PSOR)**.  
❌ **More computationally expensive** than explicit FDM.  

---

### **Final Thoughts**
| Method | Best Use Case |
|--------|--------------|
| **Explicit** | Fast option pricing but unstable. |
| **Implicit** | Robust but computationally expensive. |
| **Crank-Nicolson** | Industry standard for pricing vanilla options. |
