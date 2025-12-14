# **Pricing Formulas Cheat Sheet for Interest Rate Derivatives**  

This table summarizes the **pricing formulas** for **interest rate derivatives**, from **vanilla products (swaps, caps, floors, swaptions)** to **exotic instruments (CMS, Bermudan swaptions, structured products)**.

---

## **1. Vanilla Interest Rate Derivatives Pricing**
| **Instrument**                   | **Payoff Formula**                | **Pricing Formula**                                         | **Model Used**              |
| -------------------------------- | --------------------------------- | ----------------------------------------------------------- | --------------------------- |
| **Zero-Coupon Bond**             | $P(0,T) = e^{-rT}$                | $P(0,T) = e^{-\int_0^T f(0,t) dt}$                          | Yield Curve (Bootstrapping) |
| **Forward Rate Agreement (FRA)** | $(L(T) - K) \times \Delta$        | $P(0,T) \Delta \mathbb{E}^{\mathbb{Q}}[L(T) - K]$           | Black’s Model               |
| **Interest Rate Swap (IRS)**     | $\sum P(0,T_i) (L(T_i) - K)$      | $\sum P(0,T_i) (F(0,T_i) - K) \Delta$                       | Discounted Cash Flows       |
| **Caplet/Floorlet**              | $\max(L(T) - K, 0) \times \Delta$ | Black’s Formula: $V = P(0,T) \times BS(L(T), K, \sigma, T)$ | Black’s Model               |
| **Cap/Floor**                    | $\sum \text{Caplet}$              | $\sum P(0,T_i) \text{Black’s Formula}$                      | Black’s Model               |

✅ **Caps and Floors are portfolios of Caplets and Floorlets.**

---

## **2. Swaption Pricing (European)**
| **Instrument** | **Payoff** | **Pricing Formula** | **Model Used** |
|--------------|------------------|-----------------|----------------|
| **Payer Swaption** | $\max(A(T) - K, 0)$ | Black’s Formula: $V = P(0,T) \times BS(A(T), K, \sigma, T)$ | Black’s Model |
| **Receiver Swaption** | $\max(K - A(T), 0)$ | Black’s Formula: $V = P(0,T) \times BS(K, A(T), \sigma, T)$ | Black’s Model |
| **CMS Swaption** | Payer/Receiver on CMS Rate | Requires **Convexity Adjustments** | HJM / Libor Market Model |

✅ **Swaption pricing is often done in Black's Model using **lognormal swap rate assumptions**.

---

## **3. Callable & Bermudan Swaptions**
| **Instrument** | **Payoff** | **Pricing Approach** | **Model Used** |
|--------------|------------------|-----------------|----------------|
| **Callable Swap** | Right to terminate IRS | Backward Induction | Hull-White |
| **Bermudan Swaption** | Right to enter swap on multiple dates | Least-Squares Monte Carlo (Longstaff-Schwartz) | Hull-White, LMM |

✅ **Bermudan Swaptions require backward induction methods or Monte Carlo.**

---

## **4. Constant Maturity Swap (CMS) & Structured Products**
| **Instrument** | **Payoff** | **Pricing Formula** | **Model Used** |
|--------------|------------------|-----------------|----------------|
| **CMS Swap** | Floating Leg based on CMS Rate | $\mathbb{E}^{\mathbb{T}}[S_T]$ | Convexity Adjusted Models |
| **CMS Cap/Floor** | Cap/Floor on CMS Rate | Requires Convexity Correction | HJM, LMM |
| **Callable Range Accrual** | Pays coupon if **rate in range** | Monte Carlo or PDE | Hull-White, LMM |
| **Snowball Swap** | Coupon linked to past fixings | Backward Induction | Hull-White |

✅ **CMS instruments require convexity adjustments due to the distribution of the swap rate.**

---

## **5. Summary Table: Interest Rate Derivatives Pricing**
| **Product** | **Payoff** | **Pricing Formula** | **Common Models** |
|------------|------------|----------------|----------------|
| **Zero-Coupon Bond** | $P(0,T) = e^{-rT}$ | $P(0,T) = e^{-\int_0^T f(0,t) dt}$ | Bootstrapping, Yield Curve |
| **FRA** | $(L(T) - K) \times \Delta$ | $P(0,T) \Delta \mathbb{E}^{\mathbb{Q}}[L(T) - K]$ | Black’s Model |
| **IRS** | $\sum P(0,T_i) (L(T_i) - K)$ | $\sum P(0,T_i) (F(0,T_i) - K) \Delta$ | Discounted Cash Flows |
| **Cap/Floor** | $\max(L(T) - K, 0) \times \Delta$ | Black’s Formula | Black’s Model |
| **Swaption** | $\max(A(T) - K, 0)$ | Black’s Formula | Black’s Model |
| **Bermudan Swaption** | Multiple exercise rights | Monte Carlo, PDE | Hull-White, LMM |
| **CMS Swap** | Floating rate based on swap rate | Convexity Adjustments | HJM, LMM |
| **Callable Range Accrual** | Pays coupon if rate in range | Monte Carlo | Hull-White |

🚀 **This cheat sheet covers everything from vanilla to exotic interest rate derivatives! Let me know if you need more details on any specific model.** 🚀