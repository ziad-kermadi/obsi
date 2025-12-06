Got it! I'll ensure **all rows are complete** while keeping the **same structured format**. Here's a **refined and complete** **Linear Algebra Cheat Sheet** with **no missing rows** and **perfectly formatted tables**.

---

# **Linear Algebra Cheat Sheet for Quant Strat Interviews**

Linear algebra is critical for **Quantitative Strategies (Quant Strat) roles**, particularly in **portfolio optimization, derivatives pricing, statistical arbitrage, and machine learning**. This cheat sheet provides **concise definitions, formulas, and examples** relevant to a **Quantitative Finance interview**.

---

## **1. Vectors & Vector Operations**
A **vector** is an ordered **n-tuple** of numbers representing a point in **n-dimensional space**.

### **Basic Operations on Vectors**
| **Operation** | **Formula** | **Example** | **Application in Finance** |
|--------------|------------|------------|---------------------------|
| **Vector Addition** | $\mathbf{a} + \mathbf{b} = (a_1 + b_1, a_2 + b_2, \dots, a_n + b_n)$ | $(1,2) + (3,4) = (4,6)$ | Combining portfolio returns |
| **Scalar Multiplication** | $c\mathbf{a} = (c a_1, c a_2, \dots, c a_n)$ | $2(1,3) = (2,6)$ | Scaling asset weights |
| **Dot Product (Inner Product)** | $\mathbf{a} \cdot \mathbf{b} = \sum_{i=1}^{n} a_i b_i$ | $(1,2) \cdot (3,4) = 1(3) + 2(4) = 11$ | Risk factor exposure |
| **Norm (Magnitude)** | $||\mathbf{a}|| = \sqrt{a_1^2 + a_2^2 + \dots + a_n^2}$ | $||(3,4)|| = \sqrt{3^2 + 4^2} = 5$ | Volatility estimation |
| **Angle Between Vectors** | $\cos \theta = \frac{\mathbf{a} \cdot \mathbf{b}}{||\mathbf{a}|| \cdot ||\mathbf{b}||}$ | | Correlation between assets |
| **Projection of $\mathbf{a}$ onto $\mathbf{b}$** | $\text{proj}_{\mathbf{b}} \mathbf{a} = \frac{\mathbf{a} \cdot \mathbf{b}}{\mathbf{b} \cdot \mathbf{b}} \mathbf{b}$ | | Principal Component Analysis (PCA) |

---

## **2. Matrices & Matrix Operations**
A **matrix** is a rectangular array of numbers representing **linear transformations**.

### **Matrix Operations**
| **Operation** | **Formula** | **Example** | **Application in Finance** |
|--------------|------------|------------|---------------------------|
| **Matrix Addition** | $A + B = [a_{ij} + b_{ij}]$ | $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} + \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix} = \begin{bmatrix} 6 & 8 \\ 10 & 12 \end{bmatrix}$ | Portfolio returns aggregation |
| **Matrix Multiplication** | $(AB)_{ij} = \sum_k A_{ik} B_{kj}$ | $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} \times \begin{bmatrix} 2 & 0 \\ 1 & 3 \end{bmatrix} = \begin{bmatrix} 4 & 6 \\ 10 & 12 \end{bmatrix}$ | Transition matrices in Markov models |
| **Transpose ($A^T$)** | $A^T_{ij} = A_{ji}$ | $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}^T = \begin{bmatrix} 1 & 3 \\ 2 & 4 \end{bmatrix}$ | Covariance matrix computation |
| **Determinant ($\det(A)$)** | $\det(A) = a_{11}a_{22} - a_{12}a_{21}$ | $\det \begin{bmatrix} 3 & 2 \\ 1 & 4 \end{bmatrix} = 3(4) - 2(1) = 10$ | Matrix invertibility check |
| **Inverse ($A^{-1}$)** | $A^{-1} = \frac{1}{\det(A)} \begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$ | $A^{-1} = \frac{1}{10} \begin{bmatrix} 4 & -2 \\ -1 & 3 \end{bmatrix}$ | Risk-return optimizations |

---

## **3. Eigenvalues & Eigenvectors**
### **Definitions**
- **Eigenvalue equation**: $Ax = \lambda x$
- **Eigenvalues** ($\lambda$) represent **scaling factors**.
- **Eigenvectors** ($x$) remain in the **same direction** after transformation.

### **Applications in Quant Finance**
| **Concept** | **Formula** | **Example** | **Use Case** |
|------------|------------|------------|--------------|
| **Eigenvalue Computation** | $\det(A - \lambda I) = 0$ | $\det \begin{bmatrix} 3-\lambda & 2 \\ 1 & 4-\lambda \end{bmatrix} = 0$ | Principal Component Analysis (PCA) |
| **Spectral Decomposition** | $A = Q \Lambda Q^{-1}$ | Expressing matrices via eigenvalues | Portfolio factor analysis |
| **Singular Value Decomposition (SVD)** | $A = U \Sigma V^T$ | Factorizing a matrix | Risk factor models |

---

## **4. Systems of Linear Equations**
A system of linear equations is written in **matrix form** as:
$$
Ax = b
$$

where:
- $A$ is a coefficient matrix,
- $x$ is a column vector of unknowns,
- $b$ is a column vector of constants.

### **Solving Systems of Equations**
| **Method** | **Description** | **Example** | **Application in Finance** |
|-----------|----------------|------------|----------------------------|
| **Gaussian Elimination** | Converts to upper-triangular form | Solving linear regression models | Portfolio optimization |
| **LU Decomposition** | Factorizes $A = LU$ (Lower & Upper) | Fast matrix inversions | Interest rate models |
| **QR Decomposition** | $A = QR$ where $Q$ is orthogonal | Eigenvalue problems | Least-squares regression |

---

## **5. Applications in Quantitative Finance**
| **Concept** | **Linear Algebra Tool** | **Financial Application** |
|------------|----------------------|--------------------------|
| **Portfolio Optimization** | Matrix operations, eigenvectors | Markowitz mean-variance optimization |
| **Risk Modeling** | Covariance matrix, SVD | Principal Component Analysis (PCA) |
| **Factor Models** | Matrix factorization (SVD, Eigenvectors) | Fama-French, Arbitrage Pricing Theory (APT) |
| **Derivative Pricing** | Matrix exponentiation | Black-Scholes PDE solutions |
| **Interest Rate Models** | Solving large systems of equations | Yield curve calibration |

---

## **6. Final Summary Table**
| **Concept**               | **Formula**                                  | **Key Application**      |
| ------------------------- | -------------------------------------------- | ------------------------ |
| **Dot Product**           | $\mathbf{a} \cdot \mathbf{b} = \sum a_i b_i$ | Correlation & regression |
| **Matrix Multiplication** | $C = AB$                                     | Markov models            |
| **Eigenvalues**           | $Ax = \lambda x$                             | PCA & risk modeling      |
| **Inverse Matrix**        | $A^{-1} A = I$                               | Risk-return analysis     |
| **Covariance Matrix**     | $\Sigma = \mathbb{E}[X X^T]$                 | Volatility modeling      |

---
-----
----




-------
----------
-------------
# **Jacobian & Hessian Matrices in Quantitative Finance - Cheat Sheet**  

Jacobian and Hessian matrices are widely used in **Quantitative Finance**, particularly in **derivatives pricing, risk modeling, and optimization**. This cheat sheet provides a **step-by-step breakdown** of their **definitions, formulas, calculations, and applications in pricing and risk management**.

---

## **1. What is the Jacobian Matrix?**
The **Jacobian matrix** $J$ is a matrix of **first-order partial derivatives** of a **vector-valued function**.

If we have a function:
$$
\mathbf{F}(\mathbf{x}) =
\begin{bmatrix}
f_1(x_1, x_2, \dots, x_n) \\
f_2(x_1, x_2, \dots, x_n) \\
\vdots \\
f_m(x_1, x_2, \dots, x_n)
\end{bmatrix}
$$

Then, the **Jacobian matrix** is:
$$
J =
\begin{bmatrix}
\frac{\partial f_1}{\partial x_1} & \frac{\partial f_1}{\partial x_2} & \dots & \frac{\partial f_1}{\partial x_n} \\
\frac{\partial f_2}{\partial x_1} & \frac{\partial f_2}{\partial x_2} & \dots & \frac{\partial f_2}{\partial x_n} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial f_m}{\partial x_1} & \frac{\partial f_m}{\partial x_2} & \dots & \frac{\partial f_m}{\partial x_n}
\end{bmatrix}
$$

### **Uses in Quantitative Finance**
| **Application** | **How the Jacobian is Used** |
|---------------|--------------------------|
| **Risk Sensitivities (Greeks Calculation)** | Computes **Delta, Gamma, Vega** for derivatives pricing |
| **Multi-Factor Models** | Measures **rates of change** of multiple asset prices w.r.t. model parameters |
| **Interest Rate Curve Construction** | Computes **forward rate sensitivities** w.r.t. yield curve shifts |
| **Machine Learning (Neural Networks in Finance)** | Represents the gradient flow of loss functions |

---

## **2. What is the Hessian Matrix?**
The **Hessian matrix** $H$ is a **square matrix of second-order partial derivatives** of a **scalar-valued function**.

For a function:
$$
f(x_1, x_2, \dots, x_n)
$$

The **Hessian matrix** is:
$$
H =
\begin{bmatrix}
\frac{\partial^2 f}{\partial x_1^2} & \frac{\partial^2 f}{\partial x_1 \partial x_2} & \dots & \frac{\partial^2 f}{\partial x_1 \partial x_n} \\
\frac{\partial^2 f}{\partial x_2 \partial x_1} & \frac{\partial^2 f}{\partial x_2^2} & \dots & \frac{\partial^2 f}{\partial x_2 \partial x_n} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial^2 f}{\partial x_n \partial x_1} & \frac{\partial^2 f}{\partial x_n \partial x_2} & \dots & \frac{\partial^2 f}{\partial x_n^2}
\end{bmatrix}
$$

### **Uses in Quantitative Finance**
| **Application** | **How the Hessian is Used** |
|---------------|--------------------------|
| **Options Pricing (Gamma & Vega Sensitivity)** | Hessian measures **Gamma (Δ²Price/ΔS²)** and **Vega (Δ²Price/Δσ²)** |
| **Portfolio Risk Optimization** | Hessian used in **Markowitz Portfolio Optimization (Quadratic Programming)** |
| **Machine Learning & AI in Trading** | Hessian is used for **second-order optimization (Newton’s Method, L-BFGS)** |
| **Hedging Strategies** | Hessian helps compute **higher-order derivatives of risk factors** |

---

## **3. Computing Jacobian & Hessian Matrices**
### **Example 1: Jacobian for Multi-Asset Option Pricing**
Consider a function representing **a basket option price**:
$$
F(S_1, S_2) = \ln(S_1 S_2)
$$

Compute the **Jacobian**:
$$
J =
\begin{bmatrix}
\frac{\partial}{\partial S_1} \ln(S_1 S_2) & \frac{\partial}{\partial S_2} \ln(S_1 S_2)
\end{bmatrix}
=
\begin{bmatrix}
\frac{1}{S_1} & \frac{1}{S_2}
\end{bmatrix}
$$

#### **Financial Interpretation**
- The **Jacobian** gives the **instantaneous rate of change of the option price** w.r.t. the underlying assets.
- The **Greek equivalent**: **Delta** $(\Delta_1, \Delta_2)$ measures how **option price changes w.r.t. changes in asset prices**.

---

### **Example 2: Hessian for Option Vega & Gamma**
Consider a **Black-Scholes option pricing function**:
$$
f(S, \sigma) = S e^{-rT} N(d_1)
$$

where:
$$
d_1 = \frac{\ln(S/K) + (r + \frac{1}{2} \sigma^2)T}{\sigma \sqrt{T}}
$$

Compute the **Hessian Matrix**:
$$
H =
\begin{bmatrix}
\frac{\partial^2 f}{\partial S^2} & \frac{\partial^2 f}{\partial S \partial \sigma} \\
\frac{\partial^2 f}{\partial \sigma \partial S} & \frac{\partial^2 f}{\partial \sigma^2}
\end{bmatrix}
$$

where:
- **$\frac{\partial^2 f}{\partial S^2}$ (Gamma)** measures **convexity of option price** w.r.t. stock price.
- **$\frac{\partial^2 f}{\partial \sigma^2}$ (Vega Convexity)** measures **option price convexity w.r.t. volatility changes**.

---

## **4. Python Implementation**
### **Compute the Jacobian**
```python
import numpy as np
from sympy import symbols, Matrix, diff

# Define variables
S1, S2 = symbols('S1 S2')
F = np.log(S1 * S2)

# Compute Jacobian
J = Matrix([F]).jacobian([S1, S2])
print("Jacobian Matrix:\n", J)
```

### **Compute the Hessian**
```python
from sympy import hessian

# Define function
f = S1**2 + 3*S1*S2 + S2**2

# Compute Hessian
H = hessian(f, (S1, S2))
print("Hessian Matrix:\n", H)
```

---

## **5. Summary Table**
| **Concept** | **Formula** | **Use Case in Quant Finance** |
|------------|------------|---------------------------|
| **Jacobian Matrix** | $J_{ij} = \frac{\partial f_i}{\partial x_j}$ | Sensitivities (Delta, Rho), Risk Management |
| **Hessian Matrix** | $H_{ij} = \frac{\partial^2 f}{\partial x_i \partial x_j}$ | Gamma, Vega, Portfolio Optimization |
| **Greeks from Hessian** | $\Gamma = \frac{\partial^2 Price}{\partial S^2}$ | Second-order risk exposure |
| **Optimization Using Hessian** | Newton’s Method $x = x - H^{-1} \nabla f(x)$ | Machine Learning in Trading, Risk Hedging |

---

## **6. Final Takeaways**
- ✅ **Jacobian** is the matrix of **first derivatives** → Used for **risk sensitivities, Greeks, and multi-factor models**.
- ✅ **Hessian** is the matrix of **second derivatives** → Used in **convexity risk, Vega/Gamma, and optimization**.
- ✅ **Portfolio Optimization & Machine Learning** use the **Hessian in Newton's Method** for better convergence.
- ✅ **Monte Carlo & Greeks Calculation** rely on **Jacobian for risk exposure analysis**.

🚀 **Mastering these matrices is crucial for Quantitative Pricing, Derivatives Modeling, and Machine Learning in Trading!** 🚀

---------
---------
-----------



# **Cholesky Decomposition in Quantitative Finance - Cheat Sheet**

## **1. What is Cholesky Decomposition?**
Cholesky decomposition is used to **factorize a positive-definite matrix** $\Sigma$ into a **lower triangular matrix** $L$:
$$
\Sigma = L L^T
$$

where:
- $\Sigma$ is a **covariance matrix** (must be **symmetric and positive-definite**).
- $L$ is a **lower triangular matrix** such that $L L^T = \Sigma$.

### **Key Use Cases in Quantitative Finance**
| **Application** | **How Cholesky is Used** |
|---------------|--------------------------|
| **Monte Carlo Simulation** | Generates correlated random variables for pricing models (Black-Scholes, Heston, LIBOR Market Model) |
| **Risk Management** | Computes correlated asset returns for VaR (Value at Risk) and stress testing |
| **Factor Models (PCA, APT)** | Used to decompose covariance matrices in portfolio risk analysis |
| **Mean-Variance Optimization** | Helps construct portfolios with correlated assets |

---

## **2. Cholesky Decomposition - Step by Step**
### **Step 1: Check if the Matrix is Positive Definite**
For Cholesky to work, $\Sigma$ must be:
- **Symmetric**: $\Sigma^T = \Sigma$
- **Positive Definite**: All eigenvalues must be **positive**.

If not, use **Near-PSD (Positive Semi-Definite) Fixes** (explained below).

---

### **Step 2: Compute Cholesky Factorization**
For a given covariance matrix:
$$
\Sigma =
\begin{bmatrix}
4 & 2 \\
2 & 3
\end{bmatrix}
$$

We find **$L$** such that $\Sigma = L L^T$.

#### **Manual Calculation**
1. Compute $L_{11}$:
$$
   L_{11} = \sqrt{\Sigma_{11}} = \sqrt{4} = 2
$$
2. Compute $L_{21}$:
$$
   L_{21} = \frac{\Sigma_{21}}{L_{11}} = \frac{2}{2} = 1
$$
3. Compute $L_{22}$:
$$
   L_{22} = \sqrt{\Sigma_{22} - L_{21}^2} = \sqrt{3 - 1^2} = \sqrt{2}
$$

Thus, the Cholesky factor is:
$$
L =
\begin{bmatrix}
2 & 0 \\
1 & \sqrt{2}
\end{bmatrix}
$$

---

## **3. Python Implementation**
```python
import numpy as np
from scipy.linalg import cholesky

# Define covariance matrix (must be symmetric and positive definite)
Sigma = np.array([
    [4, 2],
    [2, 3]
])

# Compute Cholesky decomposition
L = cholesky(Sigma, lower=True)
print("Lower Triangular Matrix L:\n", L)
```

Output:
```
Lower Triangular Matrix L:
[[2.         0.        ]
 [1.         1.41421356]]
```

✅ **Cholesky is faster and more stable than Eigen or LU decomposition**.

---

## **4. Monte Carlo Simulation with Cholesky**
Cholesky is crucial for **simulating correlated asset returns** in **Monte Carlo pricing**.

### **Step-by-Step: Generating Correlated Asset Returns**
1. **Define the covariance matrix** $\Sigma$.
2. **Compute Cholesky decomposition**: $\Sigma = L L^T$.
3. **Generate independent normal random variables** $Z$.
4. **Transform them using $L$** to introduce correlation:
$$
R = L Z + \mu
$$

where:
- $Z \sim N(0, I)$ is a standard normal vector.
- $R$ represents correlated asset returns.

### **Python Code**
```python
import numpy as np
from scipy.linalg import cholesky

# Step 1: Define Covariance Matrix
Sigma = np.array([
    [0.04, 0.02],
    [0.02, 0.03]
])

# Step 2: Cholesky Decomposition
L = cholesky(Sigma, lower=True)

# Step 3: Generate Standard Normal Random Variables
n_assets = 2
n_scenarios = 10000  # Monte Carlo paths
Z = np.random.randn(n_assets, n_scenarios)

# Step 4: Generate Correlated Returns
correlated_returns = L @ Z

# Step 5: Compute Means and Covariance of Simulated Data
print("Mean of simulated returns:\n", np.mean(correlated_returns, axis=1))
print("Covariance of simulated returns:\n", np.cov(correlated_returns))
```

✅ **Used in Monte Carlo option pricing, risk modeling, and scenario analysis.**

---

## **5. Near-PSD Fix (If Matrix is Not Positive Definite)**
Sometimes, empirical covariance matrices are **not positive definite** due to:
- **Insufficient data**.
- **Multicollinearity**.
- **Errors in estimation**.

### **Fix: Add Small Positive Value (Jitter)**
```python
# Ensure matrix is positive definite
Sigma_fixed = Sigma + np.eye(Sigma.shape[0]) * 1e-6
L = cholesky(Sigma_fixed, lower=True)
```
✅ **Ensures stability when dealing with real-world data.**

---

## **6. Summary Table**
| **Step** | **Formula / Concept** | **Use Case** |
|---------|-----------------|------------------|
| **Check Positive Definite** | $\det(\Sigma) > 0$ and all eigenvalues $> 0$ | Required for Cholesky |
| **Compute Cholesky Decomposition** | $\Sigma = L L^T$ | Factorizes covariance matrices |
| **Generate Random Variables** | $Z \sim N(0, I)$ | Monte Carlo simulations |
| **Transform to Correlated Data** | $R = L Z + \mu$ | Simulate correlated asset returns |
| **Near-PSD Fix** | $\Sigma + \epsilon I$ | Fixes numerical issues |

---

## **7. Real-World Quantitative Finance Applications**
| **Application** | **Cholesky’s Role** |
|---------------|--------------------|
| **Monte Carlo Option Pricing** | Generates correlated asset paths for derivatives pricing (e.g., basket options) |
| **Portfolio Risk Analysis (VaR, CVaR)** | Computes correlated asset return distributions |
| **LIBOR Market Model (LMM)** | Simulates interest rate term structures |
| **Factor Models & PCA** | Decomposes covariance matrices for statistical arbitrage |
| **Multivariate GARCH Models** | Models volatility clustering in asset returns |

---

🚀 **Mastering Cholesky Decomposition is key for Quantitative Finance!** It allows efficient Monte Carlo simulations, portfolio risk estimation, and derivatives pricing. Let me know if you need **more in-depth examples!** 🚀





----
----
-----


# **Covariance Matrix Cheat Sheet for Quant & Pricing Roles**

## **1. What is a Covariance Matrix?**
A **covariance matrix** measures the relationships (dependencies) between multiple financial assets (stocks, bonds, options, etc.). It helps in:
- **Portfolio Optimization (Markowitz Mean-Variance Framework)**
- **Risk Management & Hedging**
- **Factor Models & PCA for Risk Decomposition**
- **Options Pricing via Monte Carlo Simulations**

---

## **2. Covariance Formula for Two Variables**
The **covariance** between two random variables $X$ and $Y$ is:
$$
\text{Cov}(X, Y) = \mathbb{E}[(X - \mathbb{E}[X]) (Y - \mathbb{E}[Y])]
$$

where:
- $\mathbb{E}[X]$ = Expected value (mean) of $X$
- $\mathbb{E}[Y]$ = Expected value (mean) of $Y$

### **Interpretation:**
- $\text{Cov}(X, Y) > 0$ → $X$ and $Y$ move **together** (positive correlation).
- $\text{Cov}(X, Y) < 0$ → $X$ and $Y$ move **opposite** (negative correlation).
- $\text{Cov}(X, Y) = 0$ → $X$ and $Y$ are **uncorrelated**.

---

## **3. Step-by-Step: Building a Covariance Matrix**
### **Step 1: Collect Asset Returns Data**
- Assume you have $N$ assets and $T$ time periods of historical returns.
- **Returns Matrix $R$ (T x N):** Each row is a time period, each column is an asset.
$$
R =
\begin{bmatrix}
r_{1,1} & r_{1,2} & \dots & r_{1,N} \\
r_{2,1} & r_{2,2} & \dots & r_{2,N} \\
\vdots  & \vdots  & \ddots & \vdots  \\
r_{T,1} & r_{T,2} & \dots & r_{T,N}
\end{bmatrix}
$$

where:
- $r_{t,i}$ is the **return of asset $i$ at time $t$**.

---

### **Step 2: Compute the Mean Returns**
For each asset $i$, compute the **mean return**:
$$
\mu_i = \frac{1}{T} \sum_{t=1}^{T} r_{t,i}
$$

or in **matrix form**:
$$
\boldsymbol{\mu} = \frac{1}{T} \sum_{t=1}^{T} R_{t,:}
$$

---

### **Step 3: Compute the Covariance Matrix**
$$
\Sigma = \frac{1}{T-1} (R - \boldsymbol{\mu})^T (R - \boldsymbol{\mu})
$$

where:
- $(R - \boldsymbol{\mu})$ is the **demeaned returns matrix**.

This formula ensures an unbiased estimate of covariance by using $\frac{1}{T-1}$ instead of $\frac{1}{T}$.

---

## **4. Covariance Matrix Example**
Given the **returns matrix $R$ for 3 assets** over 3 time periods:
$$
R =
\begin{bmatrix}
0.02 & 0.03 & 0.01 \\
0.01 & 0.05 & -0.02 \\
0.03 & 0.02 & 0.00
\end{bmatrix}
$$

1. **Compute Mean Returns**:
$$
   \mu = \begin{bmatrix} 0.02 & 0.0333 & -0.0033 \end{bmatrix}
$$

2. **Demean Returns**:
$$
   R - \mu =
   \begin{bmatrix}
   0.0000 & -0.0033 & 0.0133 \\
   -0.0100 & 0.0167 & -0.0167 \\
   0.0100 & -0.0133 & 0.0033
   \end{bmatrix}
$$

3. **Compute Covariance Matrix**:
$$
   \Sigma =
   \begin{bmatrix}
   0.0001 & 0.00005 & -0.00002 \\
   0.00005 & 0.00015 & -0.00005 \\
   -0.00002 & -0.00005 & 0.00008
   \end{bmatrix}
$$

---

## **5. Covariance Matrix in Python (Numpy)**
```python
import numpy as np

# Simulated asset returns (T=3, N=3)
returns = np.array([
    [0.02, 0.03, 0.01],
    [0.01, 0.05, -0.02],
    [0.03, 0.02, 0.00]
])

# Compute covariance matrix
cov_matrix = np.cov(returns, rowvar=False)
print(cov_matrix)
```

✅ **Key Notes**:
- `rowvar=False`: Treats columns as variables (assets).
- Automatically applies **$\frac{1}{T-1}$ normalization**.

---

## **6. Using Covariance Matrix in Finance**
| **Application** | **Formula** | **Use Case** |
|---------------|------------|-------------|
| **Portfolio Variance** | $\sigma_p^2 = w^T \Sigma w$ | **Risk estimation for portfolios** |
| **Optimal Portfolio Weights (Markowitz)** | $w^* = \Sigma^{-1} (\mu - r_f)$ | **Mean-variance optimization** |
| **Correlation Matrix** | $C_{ij} = \frac{\Sigma_{ij}}{\sigma_i \sigma_j}$ | **Finds relationships between assets** |
| **Factor Models (PCA, APT)** | $R = F \beta + \epsilon$ | **Risk decomposition** |
| **Monte Carlo Simulation** | Cholesky Decomposition $\Sigma = LL^T$ | **Generating correlated asset paths** |

---

## **7. Converting Covariance to Correlation Matrix**
The **correlation matrix** normalizes the covariance matrix:
$$
C_{ij} = \frac{\Sigma_{ij}}{\sigma_i \sigma_j}
$$

### **In Python (Numpy)**
```python
std_devs = np.sqrt(np.diag(cov_matrix))
correlation_matrix = cov_matrix / np.outer(std_devs, std_devs)
print(correlation_matrix)
```
✅ **Ensures values are between [-1,1]**.

---

## **8. Monte Carlo Simulation Using Covariance Matrix**
To generate correlated asset paths, use **Cholesky decomposition**:
$$
\Sigma = L L^T
$$

1. **Generate standard normal shocks $Z$**.
2. **Transform them using $L$**:
$$
R_t = L Z_t + \mu
$$

### **In Python**
```python
from scipy.linalg import cholesky

# Cholesky decomposition
L = cholesky(cov_matrix, lower=True)

# Generate standard normal random numbers
Z = np.random.randn(3, 10000)  # 3 assets, 10,000 simulations

# Generate correlated returns
correlated_returns = L @ Z
```
✅ **Used in derivative pricing models**.

---

## **9. Summary Table**
| **Step** | **Formula** | **Explanation** |
|----------|------------|----------------|
| **Compute Mean Returns** | $\mu_i = \frac{1}{T} \sum_{t=1}^{T} r_{t,i}$ | Average return per asset |
| **Demean Returns** | $R - \mu$ | Centered returns matrix |
| **Covariance Matrix** | $\Sigma = \frac{1}{T-1} (R - \mu)^T (R - \mu)$ | Measures asset return co-movements |
| **Portfolio Variance** | $\sigma_p^2 = w^T \Sigma w$ | Portfolio risk measure |
| **Correlation Matrix** | $C = D^{-1} \Sigma D^{-1}$ | Standardized covariance |
| **Monte Carlo Cholesky** | $R_t = L Z_t + \mu$ | Simulates correlated asset returns |

---

🚀 **Mastering covariance matrices is crucial for portfolio optimization, risk management, and Monte Carlo simulations in Quantitative Finance!** Let me know if you need **deeper examples or real-world applications!** 🚀
----
-----
----

