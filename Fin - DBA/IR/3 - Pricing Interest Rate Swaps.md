

DBA pics to be added








### **Understanding the Arrear Interval in Swaps**

The **arrear interval** in a swap refers to the time gap between when the floating rate is **observed (fixed)** and when it is **applied** to the cash flow. This concept is crucial in differentiating **standard (vanilla) swaps** and **in-arrears swaps**.

---

## **1. Standard (Vanilla) Interest Rate Swap**
### **Why Does the Arrear Interval Match the Payment Frequency?**
- In a normal **quarterly swap (3M frequency)**, the floating rate for each period is **fixed at the beginning** of that period.
- The rate remains unchanged throughout that **3-month period** and is applied at the end of the period when the payment is due.
- This results in an **arrear interval equal to the frequency** (i.e., 3M for a quarterly swap).

### **Example (Vanilla 3M LIBOR Swap)**
- **March 18**: The 3M LIBOR rate is observed and fixed.
- **March 20**: The floating rate period starts (2-day forward convention).
- **June 20**: Interest payment is made, based on the **March 18 LIBOR fixing**.

#### **Key Point**
- The floating rate is set in advance and **applied for the entire period**, meaning the arrear interval is **equal to the frequency** (3 months in this case).

---

## **2. In-Arrears Swap**
### **Why is the Arrear Interval 0?**
- In an **in-arrears swap**, the floating rate is **observed at the end of the period**, instead of the beginning.
- This means there is **no forward-starting period**—the rate is fixed at the exact moment it is applied.
- Since the rate is set and applied at the **same time**, the **arrear interval is 0**.

### **Example (In-Arrears 3M LIBOR Swap)**
- **March 20**: The 3-month period starts.
- **June 20**: The floating rate is observed **on the same day as the payment** and applied immediately.

#### **Key Point**
- Unlike standard swaps where the rate is **pre-determined**, in-arrears swaps use the rate that is **only known at the end of the accrual period**.
- Since there is no lead time between **fixing and application**, the arrear interval is **zero**.

---

## **Comparison of Swaps with Different Arrear Intervals**
| Swap Type            | Floating Rate Observed | Applied To | Arrear Interval |
|----------------------|----------------------|------------|----------------|
| **Standard Swap**    | Start of period      | End of period | Matches frequency (e.g., 3M for a quarterly swap) |
| **In-Arrears Swap**  | End of period        | Same time (immediate) | 0 |

---

### **Why Does This Matter?**
- **Risk & Discounting:** In-arrears swaps introduce **rate uncertainty** since payments depend on an unknown rate until maturity.
- **Valuation Complexity:** Discounting must account for the fact that the rate is unknown until the very last moment.
- **Usage:** In-arrears swaps are often used in structured products or bespoke hedging where **exposure to the most recent rate levels is preferred**.

---

### **Conclusion**
- **Standard swaps** have an arrear interval equal to the frequency because the floating rate is fixed in advance.
- **In-arrears swaps** have an arrear interval of **0** because the rate is observed **at the exact time it is applied**.





