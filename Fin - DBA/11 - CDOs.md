



----
----
----
### **Step-by-Step Breakdown of the Gaussian Copula Model for CDO Pricing**

The **Gaussian Copula Model** is used to price **CDO tranches** by simulating **correlated defaults** in a portfolio of credit assets. The key challenge is modeling **how defaults are related**—this is where the **copula** function comes in.

---

## **1. Objective of the Model**
We want to determine **expected tranche losses**, given:
1. **Default probabilities ($P_{\text{default}}$)** for each asset.
2. **Correlations ($\rho$)** between defaults.
3. **Loss Given Default (LGD)** and recovery rates.
4. **Discounted payoffs** for tranches.

---

## **2. Step-by-Step Walkthrough**

### **Step 1: Define Individual Default Probabilities**
For each credit asset $i$, we assume:
- Default follows a **Bernoulli process** (asset either defaults or survives).
- Probability of default over a given horizon is **$P_{\text{default}, i}$**.
- Example: If **$P_{\text{default}, i} = 5\%$**, then in a risk-neutral world, this means there's a **5% chance that asset $i$ will default** in the given time frame.
$$
P_{\text{default}, i} = P(X_i < x_{\text{default}})
$$

where:
- $X_i \sim \mathcal{N}(0,1)$ (standard normal variable for asset $i$)
- $x_{\text{default}} = \Phi^{-1}(P_{\text{default}, i})$ (inverse CDF of normal distribution)

**Intuition**:  
- Each asset’s default probability corresponds to a threshold $x_{\text{default}}$ in a **standard normal space**.
- If the random variable $X_i$ falls below $x_{\text{default}}$, the asset **defaults**.

---

### **Step 2: Introduce Correlation with a Gaussian Copula**
Now, we introduce **systematic risk** using a **Gaussian copula**, which captures the correlation in defaults.

Each asset’s credit quality is linked to a **systematic risk factor $Z_{\text{market}}$** and an **idiosyncratic risk factor $\epsilon_i$**:
$$
X_i = \rho Z_{\text{market}} + \sqrt{1 - \rho^2} \epsilon_i
$$

where:
- $Z_{\text{market}} \sim \mathcal{N}(0,1)$ (systematic risk affecting all assets)
- $\epsilon_i \sim \mathcal{N}(0,1)$ (idiosyncratic risk specific to asset $i$)
- $\rho$ = Correlation between asset $i$’s default and market-wide risk.

🔹 **Key Idea**:
- If $Z_{\text{market}}$ drops significantly (negative shock), multiple assets can default together.
- Higher $\rho$ → Defaults **cluster** (correlated defaults).
- Lower $\rho$ → Defaults occur more independently.

---

### **Step 3: Simulating Default Scenarios**
1. **Simulate $Z_{\text{market}}$** from $\mathcal{N}(0,1)$.
2. **For each asset $i$, compute $X_i$**:
$$
   X_i = \rho Z_{\text{market}} + \sqrt{1 - \rho^2} \epsilon_i
$$
   where $\epsilon_i \sim \mathcal{N}(0,1)$.
3. **Check if $X_i$ falls below $x_{\text{default}, i}$**:
   - If **$X_i < x_{\text{default}, i}$** → **Default occurs**.
   - Otherwise → No default.

**Example**:
- Assume $P_{\text{default}, i} = 5\%$, so $x_{\text{default}, i} = \Phi^{-1}(0.05) = -1.645$.
- If simulated $X_i = -2.0$, the asset defaults.

---

### **Step 4: Compute Portfolio Default Loss**
Once we have **which assets default**, we calculate **portfolio loss**:
$$
\text{Portfolio Loss} = \sum_{i \in \text{defaults}} \text{LGD}_i \times \text{Notional}_i
$$

where:
- $\text{LGD}_i$ = Loss given default (e.g., 60% if recovery is 40%).
- $\text{Notional}_i$ = Size of credit exposure.

---

### **Step 5: Allocate Losses to CDO Tranches**
Each **CDO tranche absorbs losses sequentially**.

#### **Tranche Loss Formula**
$$
\text{Tranche Loss} = \min( \max( \text{Total Portfolio Loss} - \text{Subordination}, 0), \text{Tranche Thickness})
$$

where:
- **Subordination** = How much loss is absorbed **before** the tranche.
- **Tranche Thickness** = Total notional of the tranche.

**Example**:
- A mezzanine tranche covers **5%–15% of portfolio losses**.
- If **total portfolio loss = 10%**, then:
  - **Equity tranche (0%–5%) fully wiped out**.
  - **Mezzanine tranche loses 5%**.
  - **Senior tranche (15%–100%) remains untouched**.

---

### **Step 6: Compute Expected Loss for Each Tranche**
$$
\text{Expected Loss} = \int_0^1 \text{Tranche Loss} \times P_{\text{default}}(x) dx
$$

where $P_{\text{default}}(x)$ is obtained from the **simulated Gaussian Copula model**.

**Interpretation**:
- Higher **correlation** $\rho$ → Tranche losses more concentrated (affecting **senior tranches more**).
- Lower **correlation** $\rho$ → Tranche losses spread more evenly.

---

### **Step 7: Compute Present Value of Expected Payoff**
$$
\text{Tranche Price} = \sum \frac{\text{Expected Payoff}_t}{(1+r)^t}
$$

where:
- **Expected Payoff** = Initial tranche notional minus expected loss.
- **Discount Factor** $(1+r)^t$ accounts for time value of money.

---

## **Summary of Steps**
| **Step** | **Process** |
|---------|------------|
| **Step 1** | Define **individual default probabilities** $P_{\text{default}}$. |
| **Step 2** | Introduce **correlation structure** using Gaussian Copula. |
| **Step 3** | Simulate **default scenarios** based on $Z_{\text{market}}$. |
| **Step 4** | Compute **portfolio loss** based on defaults. |
| **Step 5** | Allocate losses **to tranches** based on subordination. |
| **Step 6** | Compute **expected loss for each tranche**. |
| **Step 7** | Compute **present value of expected tranche payoffs**. |

---

## **Key Takeaways**
1. **Gaussian Copula links individual defaults** to a **systematic risk factor**.
2. **Default correlation ($\rho$) matters**:
   - **High correlation** → More senior tranche losses.
   - **Low correlation** → More spread-out losses.
3. **Monte Carlo simulations are used** to estimate expected losses.
4. **Tranches absorb losses in order**: Equity → Mezzanine → Senior.

---

