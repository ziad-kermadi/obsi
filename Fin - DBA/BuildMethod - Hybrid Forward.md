
The **Hybrid Forward Curve Building** method is a technique used to construct a piecewise-continuous curve for forward rates. It combines elements of **Quadratic Forward**, **Constant Forward**, and **Linear Zero** approaches to ensure smoothness while keeping the risk characteristics of the curve appropriate for pricing and risk management.

Below is a **step-by-step breakdown** of how this curve is built mathematically:

---

### **Step 1: Initial Setup & Notation**
We define:
- $df(t)$ as the **discount function**, which relates to the yield curve.
- $f(t)$ as the **continuously-compounded forward rate**.
- $t_i$ as the **time to maturity** for the build instruments.
- $n_i$ as the **number of cash flows** for each instrument.
- $\tau_{i,j}$ as the accrual time for the $j^{th}$ coupon.
- $r_i$ as the fixed interest rate for the instrument.

The fundamental equation for repricing an instrument is:
$$
\epsilon_i = df_{i,0} - r_i \sum_{j=1}^{n_i -1} \tau_{i,j} df_{i,j} - (1 + r_i \tau_{i, n_i}) df_{i, n_i}
$$

where:
$$
df_{i,j} = df(t_{i,j})
$$

This equation ensures that the present value of cash flows correctly matches the market price.

---

### **Step 2: Defining the Time Grid**
The **time vector** $t$ contains the maturities where the bootstrap equations will be solved:
$$
\bar{t} = (t_i) \quad \text{where} \quad t_i = t_i n_i
$$

For each instrument, we choose an index for the first cash flow strictly after the previous bootstrap node. This ensures that previous flows do not need to be revalued as the bootstrap progresses.
$$
\bar{n}_i \text{ is chosen such that } t_{i-1} \in [t_{i,n_{i-1}}, t_{i,\bar{n}_i}]
$$

This selection minimizes recalculations in the iterative process.

---

### **Step 3: Discount Factor Calculation**
The discount factors are calculated using the forward rate:
$$
df(t) = df_{i-1} \cdot \exp \left(- \int_{t_{i-1}}^t f_i(s; \vec{x}) ds \right)
$$

where:

- $f_i(s; \vec{x})$ is the forward rate function.
- $\vec{x}$ is a vector of parameters defining the forward rate.

A convenient notation is:
$$
F_i(t; \vec{x}) = \int_{t_{i-1}}^t f_i(s; \vec{x}) ds
$$

which simplifies expressions involving the forward rate.

---

### **Step 4: Bootstrap Procedure Using Newton-Raphson**
We solve for the forward rate parameters iteratively using **Newton-Raphson**. The system of equations for iteration $j$ is:
$$
\epsilon_{i,j} = \sum_{k=0}^{n_i -1} p_{i,k} df_{i,k} + df_{i-1} - \sum_{k=\bar{n}_i}^{n_i} p_{i,k} df_{i,k} \cdot \exp(-F_i(t_i, k; x_{i,j}))
$$

The next estimate for $x_{i,j}$ is:
$$
\Delta_{i,j} = df_{i-1} - \sum_{k=\bar{n}_i}^{n_i} p_{i,k} \frac{\delta F_i}{\delta x} (t_i, k; x_{i,j}) \cdot \exp(-F_i(t_i, k; x_{i,j}))
$$
$$
x_{i,j+1} = x_{i,j} + \frac{\epsilon_{i,j}}{\Delta_{i,j}}
$$

This iteratively finds the forward rate that ensures repricing accuracy.

---

### **Step 5: Functional Forms of the Forward Spline**
Three types of spline interpolations are used:

1. **Constant Forward Spline**:
$$
   f(t) = f_{i-1}, \quad t \in [t_{i-1}, t_i]
$$
$$
   F(t) = f_{i-1} (t - t_{i-1})
$$

2. **Linear Forward Spline**:
$$
   f(t) = f_{i-1} + \frac{f_i - f_{i-1}}{t_i - t_{i-1}} (t - t_{i-1})
$$
$$
   F(t) = f_{i-1} (t - t_{i-1}) + \frac{1}{2} \frac{f_i - f_{i-1}}{t_i - t_{i-1}} (t - t_{i-1})^2
$$

3. **Quadratic Forward Spline**:
$$
   f(t) = f_{i-1} + \frac{f_i - f_{i-1}}{t_i - t_{i-1}} (t - t_{i-1}) + c_i (t - t_{i-1})^2
$$
$$
   F(t) = f_{i-1} (t - t_{i-1}) + \frac{1}{2} \frac{f_i - f_{i-1}}{t_i - t_{i-1}} (t - t_{i-1})^2 + \frac{c_i}{3} (t - t_{i-1})^3
$$

where $c_i$ is a curvature coefficient.

---

### **Step 6: The Hybrid Forward Build Procedure**
The curve-building process follows these steps:

1. **Bootstrap Step**: A constant forward spline is used for initial rates.
2. **Midpoint Calculation**: The forward curve is built from **midpoints** of rates.
3. **Quadratic Adjustment**: The final segment is adjusted using a quadratic spline to ensure smoothness.

For the first few nodes, the **constant forward rate bootstrap** is used:
$$
f_i = \tilde{f}_{i-1} + \frac{t_i - t_{i-2}}{t_i + t_{i-2}} (\tilde{f}_i - \tilde{f}_{i-1})
$$

For the final node, the last forward rate $f_n$ is computed via a linear spline, with $c_n = 0$ ensuring smooth termination.

---

### **Conclusion**
This method builds a **smooth forward curve** that transitions seamlessly across different segments:
- **Constant forward segments** for initial estimates.
- **Linear interpolation** to match midpoints.
- **Quadratic smoothing** for a final curve that ensures risk-sensitive pricing.

This approach balances **mathematical flexibility** (via splines) with **practical stability** (ensuring that swaps and discount factors behave correctly for pricing and risk management).


![[Pasted image 20250318181133.png]]

----------------------------------------------------------------------------------------------------------------------------------------------------
### **Concept of Splines**
Splines are **piecewise polynomial functions** used to interpolate or approximate data while ensuring smooth transitions between points. They are commonly used in numerical methods, finance, and engineering to model curves.

A **spline function** $S(x)$ is defined piecewise over different intervals $[x_i, x_{i+1}]$, ensuring **continuity** and sometimes **smoothness** (continuous derivatives) across segments.

---

## **Types of Splines and Their Conditions**
Each type of spline has different degrees of smoothness and constraints.

### **1. Linear Spline (Piecewise Linear Interpolation)**
- Uses straight-line segments to connect points.
- **Formula** for $x \in [x_i, x_{i+1}]$:
$$
  S_i(x) = a_i + b_i (x - x_i)
$$
  where:
  - $a_i = y_i$ (ensures it passes through data points).
  - $b_i = \frac{y_{i+1} - y_i}{x_{i+1} - x_i}$ (ensures slope continuity).

#### **Conditions**
- **Interpolation Condition**:
$$
  S_i(x_i) = y_i, \quad S_i(x_{i+1}) = y_{i+1}
$$
- No continuity of the first derivative; sharp changes in slope are possible.

---

### **2. Quadratic Spline (Piecewise Quadratic Interpolation)**
- Uses a quadratic polynomial in each segment.
- **Formula** for $x \in [x_i, x_{i+1}]$:
$$
  S_i(x) = a_i + b_i (x - x_i) + c_i (x - x_i)^2
$$

#### **Conditions**
- **Interpolation Condition**:
$$
  S_i(x_i) = y_i, \quad S_i(x_{i+1}) = y_{i+1}
$$
- **Continuity of First Derivative**:
$$
  S_i'(x_{i+1}) = S_{i+1}'(x_{i+1})
$$
- Often an additional condition (e.g., natural or clamped boundary) is needed to fully determine the system.

---

### **3. Cubic Spline (Most Common)**
- Uses a cubic polynomial per interval to ensure **continuity of both first and second derivatives**.
- **Formula** for $x \in [x_i, x_{i+1}]$:
$$
  S_i(x) = a_i + b_i (x - x_i) + c_i (x - x_i)^2 + d_i (x - x_i)^3
$$

#### **Conditions**
- **Interpolation Condition**:
$$
  S_i(x_i) = y_i, \quad S_i(x_{i+1}) = y_{i+1}
$$
- **Continuity of First Derivative**:
$$
  S_i'(x_{i+1}) = S_{i+1}'(x_{i+1})
$$
- **Continuity of Second Derivative**:
$$
  S_i''(x_{i+1}) = S_{i+1}''(x_{i+1})
$$
- **Boundary Conditions** (to close the system):
  - **Natural Spline**: Second derivative at endpoints is zero.
$$
    S''(x_0) = 0, \quad S''(x_n) = 0
$$
  - **Clamped Spline**: First derivatives at endpoints are specified.
$$
    S'(x_0) = f'(x_0), \quad S'(x_n) = f'(x_n)
$$
  - **Not-a-Knot Spline**: Third derivatives are continuous at second and penultimate points.

---

## **Solving for Spline Coefficients**
To find the unknown coefficients ($a_i, b_i, c_i, d_i$), we solve a system of equations based on the above conditions.

For **cubic splines**, this results in a **tridiagonal system**, which is efficiently solved using numerical methods like:
- Gaussian elimination for tridiagonal matrices.
- Thomas Algorithm (a specialized efficient solver).

---

### **Key Takeaways**
- **Linear splines** are simple but not smooth.
- **Quadratic splines** ensure continuity of the first derivative.
- **Cubic splines** ensure full smoothness (most widely used in finance and engineering).
- **Hybrid Forward Curves** (from your previous request) use a combination of constant, linear, and quadratic splines to build smooth forward rate curves.



--------------------------------------------------------------------------



![[Hybrid_forward_splines.pdf]]

![[Pasted image 20250317113709.png]]

![[Pasted image 20250317113725.png]]

![[Pasted image 20250317113735.png]]

