Sure! Let’s step through a detailed proof that the determinant of an upper triangular matrix is the product of its diagonal entries. We’ll start by recalling the definition of the determinant and then demonstrate the result using properties of row expansion.

---

### **Step 1: Definition of the determinant**

For an n×nn \times n matrix AA, the determinant is defined as:

det⁡(A)=∑σ∈Snsgn(σ)∏i=1nAi,σ(i)\det(A) = \sum_{\sigma \in S_n} \text{sgn}(\sigma) \prod_{i=1}^n A_{i, \sigma(i)}

where:

- SnS_n is the set of all permutations of {1,2,…,n}\{1, 2, \ldots, n\},
    
- sgn(σ)\text{sgn}(\sigma) is the sign of the permutation σ\sigma,
    
- The product ∏i=1nAi,σ(i)\prod_{i=1}^n A_{i, \sigma(i)} corresponds to one term in the determinant’s expansion.
    

---

### **Step 2: Structure of an upper triangular matrix**

If AA is upper triangular, then:

A=[a11a12a13⋯a1n0a22a23⋯a2n00a33⋯a3n⋮⋮⋮⋱⋮000⋯ann]A = \begin{bmatrix} a_{11} & a_{12} & a_{13} & \cdots & a_{1n} \\ 0 & a_{22} & a_{23} & \cdots & a_{2n} \\ 0 & 0 & a_{33} & \cdots & a_{3n} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & 0 & \cdots & a_{nn} \end{bmatrix}

Notice that all entries below the diagonal are zero:

Aij=0for i>j.A_{ij} = 0 \quad \text{for } i > j.

---

### **Step 3: Contribution from each permutation**

The determinant is a sum over permutations, but only certain terms contribute. In particular:

1. Each term in the determinant is associated with a permutation σ\sigma, and each σ\sigma defines a unique pattern of row-column assignments.
    
2. To have a nonzero product ∏i=1nAi,σ(i)\prod_{i=1}^n A_{i, \sigma(i)}, **all chosen entries Ai,σ(i)A_{i, \sigma(i)} must be nonzero**.
    
3. In an upper triangular matrix, the only nonzero entries are those where i≤σ(i)i \leq \sigma(i). To pick nn nonzero entries, σ(i)=i\sigma(i) = i must hold for all ii.
    

Thus, the **only permutation that contributes is the identity permutation σ=(1,2,…,n)\sigma = (1, 2, \ldots, n)**.

---

### **Step 4: Determinant reduces to diagonal product**

For the identity permutation σ(i)=i\sigma(i) = i, the term becomes:

∏i=1nAi,σ(i)=∏i=1nAii.\prod_{i=1}^n A_{i, \sigma(i)} = \prod_{i=1}^n A_{ii}.

Because no other permutation contributes (all other permutations will select at least one zero below the diagonal), the determinant simplifies to:

det⁡(A)=∏i=1nAii.\det(A) = \prod_{i=1}^n A_{ii}.

---

### **Step 5: Generalization to any size**

This logic holds for any nn:

- Only the diagonal entries contribute because every other permutation leads to a product containing zero.
    
- The determinant is simply the product of the diagonal entries.
    

---

### **Conclusion**

For an upper triangular matrix, the determinant is equal to the product of its diagonal entries:

det⁡(A)=a11⋅a22⋅⋯⋅ann.\det(A) = a_{11} \cdot a_{22} \cdot \cdots \cdot a_{nn}.


