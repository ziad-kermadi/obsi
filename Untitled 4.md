Here are 20 questions (mix of derivation/proof and short-answer) for **Chapter 1 (Introduction)** of Bergomi’s *Stochastic Volatility Modeling*, each followed by its answer including the key formula and an intuition note. At the end you’ll find a summary table.

---

1. **Derivation:** Show how specifying a break-even condition for the second-order term in the delta-hedged P\&L leads to the Black–Scholes PDE.
   **Answer:**

   2. Start from the P\&L of a short delta-hedged option over a small δt:

      $$
      \text{P\&L} = -\tfrac12\,\Gamma\,S^2(\delta W)^2 + \Theta\,\delta t + o(\delta t)\quad{\text{with }}\Gamma=\tfrac{\partial^2P}{\partial S^2},\;\Theta=\tfrac{\partial P}{\partial t}.
      $$
   3. Impose “break-even” by cancelling the $O(\delta t)$ term on average:
      $\mathbb{E}[(\delta W)^2]=\delta t$, so
      $-\tfrac12\,\Gamma\,S^2 + \Theta = 0$.
   4. Rearranged, this is

      $$
      \tfrac{\partial P}{\partial t} + \tfrac12\,\sigma^2 S^2\!\tfrac{\partial^2 P}{\partial S^2}=0,
      $$

      which with drift terms gives the full Black–Scholes PDE .
      **Intuition:** Ensuring no systematic P\&L drift at second order enforces the PDE—modeling emerges from hedging consistency, not assumed asset dynamics.

5. **Short-Answer:** What is the key probabilistic interpretation of the Black–Scholes PDE solution?
   **Answer:**
   The solution is the risk-neutral expectation of the payoff under the diffusion
   $\!dS_t=(r-q)S_t\,dt+\sigma S_t\,dW_t$.
   **Intuition:** Pricing arises from averaging under an arbitrage-free measure implied by hedging.

6. **Short-Answer:** Define a “usable model” in the multi-instrument hedging context.
   **Answer:**
   A model is usable if there exists, for all $t,S$, a positive break-even covariance matrix $C_{ij}$ such that

   $$
   \Theta = -\tfrac12\sum_{i,j}\Gamma_{ij}\,C_{ij},
   $$

   ensuring no systematic carry P\&L .
   **Intuition:** All gamma/theta exposures can be matched by some positive covariance of hedges.

7. **Derivation:** Starting from multiple hedge instruments $S_i$, derive the P\&L form
   $\text{P\&L}=-\tfrac12\sum_{ij}\phi_{ij}\bigl(\tfrac{\delta S_i}{S_i}\tfrac{\delta S_j}{S_j}-C_{ij}\delta t\bigr)$.
   **Answer:**

   Write carry P\&L in basis of instruments $T$ with weights $\omega_k$.
   Show $A=-\tfrac12\sum_k\phi_k\omega_k$ and re-express in matrix form using $C=T\omega T^\top$.
   Yields the stated P\&L .
      **Intuition:** Multivariate gamma exposures translate into an implied covariance matrix.

8. **Short-Answer:** Why is delta-hedging alone inadequate to control P\&L dispersion?
   **Answer:**
   Because residual P\&L variance is dominated by realized-volatility volatility and its autocorrelation, not just spot moves .
   **Intuition:** Volatility itself fluctuates, creating second-order risk beyond delta.

9. **Short-Answer:** What question motivates the move to stochastic volatility models?
   **Answer:**
   How to model the joint dynamics of spot and the whole implied-volatility surface, since delta-hedging with vanilla options exposes one to volatility-of-volatility risk .
   **Intuition:** Vanilla options used as hedges have their own future-smile risk.

10. **Short-Answer:** In section 1.2 the “real case” of delta-hedging refers to what?
   **Answer:**
   Actual asset returns exhibit fat tails and stochastic volatility, so delta-hedged P\&L has non-Gaussian, longer-tailed dispersion .
   **Intuition:** Reality departs from the lognormal assumption, so hedging leaves larger P\&L swings.

11. **Derivation:** Show that under Black–Scholes assumptions, delta-hedging reduces P\&L to pure theta/gamma.
   **Answer:**

   12. Use $\Delta=\partial P/\partial S$ to cancel the linear $\delta S$ term in the Taylor expansion of $P(t+\delta t,S+\delta S)$.
   13. Remaining P\&L is $-\tfrac12\,\Gamma S^2(\delta W)^2+\Theta\,\delta t$.
   14. Imposing break-even yields the PDE as above .
      **Intuition:** Delta choice neutralizes first-order spot risk, leaving curvature exposure.

15. **Short-Answer:** What are the two illustrative exotic examples in § 1.3?
   **Answer:**

   16. Barrier option (Example 1).
   17. Forward-start option (Example 2) .
      **Intuition:** Barrier options isolate future skew risk; forward-starts isolate forward-volatility risk.

18. **Short-Answer:** What does the “forward-start option” teach about volatility modeling?
    **Answer:**
    It shows that at the forward date, one needs a model for the future implied skew, not just realized volatility, motivating stochastic-volatility dynamics .
    **Intuition:** The option’s payoff depends on future smile shape.

19. **Short-Answer:** Why do practitioners still use Black–Scholes despite its model error?
    **Answer:**
    It offers simple hedging rules, tractability, and a benchmark PDE consistent with break-even arguments, even if spot dynamics are misspecified .
    **Intuition:** It’s a minimal “floater” that avoids making stronger assumptions.

20. **Short-Answer:** State the convex-order no-arbitrage condition mentioned in the digest.
    **Answer:**
    If $g(S)\ge f(S)$ ∀ $S$, then $g$ should price higher than $f$; for linear pricing this implies convex-order on implied densities .
    **Intuition:** Prices preserve payoff ordering under risk-neutral expectations.

21. **Derivation:** Outline why modeling doesn’t start from assuming $dS_t$ is lognormal.
    **Answer:**
    Hedging-based break-even yields the PDE; the diffusion form for $S_t$ is its probabilistic representation, not the modeling premise .
    **Intuition:** Hedge consistency drives model form, not vice versa.

22. **Short-Answer:** What is the main source of P\&L dispersion for longer-dated options?
    **Answer:**
    Volatility-of-volatility and its long-range correlations dominate dispersion for maturities beyond very short ones .
    **Intuition:** Vol surface dynamics matter more as time horizon lengthens.

23. **Short-Answer:** What does the chapter conclude about delta vs. vega hedging?
    **Answer:**
    Delta-hedging is often done daily, but vega-hedging less frequently; trends between re-hedges materialize half of cross-gamma P\&L .
    **Intuition:** Less-frequent vega rebalancing leaves residual volatility risk.

24. **Derivation:** Explain how the break-even condition at order 2 gives rise to a parabolic equation.
    **Answer:**
    Order-2 term in Taylor expansion yields $-\tfrac12\Gamma S^2\sigma^2\delta t + \Theta\delta t=0$; dividing by $\delta t$ gives a time-dependent second-derivative PDE, a parabolic equation .
    **Intuition:** Cancellation at second order enforces diffusion-like dynamics.

25. **Short-Answer:** What is the practical implication of section 1.3’s conclusion?
    **Answer:**
    Exotic option P\&L depends intricately on implied-volatility dynamics, so one needs stochastic-volatility models to price and hedge them accurately .
    **Intuition:** Static calibration isn’t enough; the smile’s future path matters.

26. **Short-Answer:** Describe the role of vanilla options in gamma-hedging.
    **Answer:**
    Vanilla options act as hedging instruments for exotic books; their own gamma exposures create cross-gamma P\&L that must be modeled .
    **Intuition:** You hedge options with options—but then must model option dynamics.

27. **Short-Answer:** Summarize the main points of the chapter’s digest.
    **Answer:**

    * Delta-hedging removes first-order spot risk.
    * Break-even at second order yields BS PDE.
    * Multi-instrument usability requires break-even covariance matrix.
    * Real P\&L dispersion is dominated by vol-of-vol risk.
    * Stochastic-vol models needed for implied-vol dynamics and exotics .
      **Intuition:** Each bullet tracks one layer of modeling complexity.

28. **Short-Answer:** Why is modeling implied-volatility dynamics more relevant than realized-volatility dynamics?
    **Answer:**
    Because hedged positions’ P\&L is marked to market using implied vols, and dynamic trading in options exposes one to uncertainty in future implied vols .
    **Intuition:** You trade at market prices (implied vols), not realized vols.

---

### Summary Table

| Q No. | Type         | Concept                               | Formula/Proof                  | Intuition Focus                 |
| :---: | ------------ | ------------------------------------- | ------------------------------ | ------------------------------- |
|   1   | Derivation   | BS PDE from hedging                   | P\&L → PDE                     | Hedge consistency               |
|   2   | Short-Answer | Probabilistic interpretation          | Risk-neutral expectation       | Diffusion representation        |
|   3   | Short-Answer | Usable model (multi-instrument)       | $\Theta=-\tfrac12\sum\Gamma C$ | Covariance break-even           |
|   4   | Derivation   | Multi-asset P\&L form                 | Eq. (1.8)                      | Implied covariance matrix       |
|   5   | Short-Answer | Delta-hedging inadequacy              | –                              | Vol-of-vol dominates            |
|   6   | Short-Answer | Motivation for stochastic vols        | –                              | Joint spot-vol surface dynamics |
|   7   | Short-Answer | “Real case” vs BS                     | –                              | Fat tails & stochastic vols     |
|   8   | Derivation   | Cancel first-order term with Δ choice | Use Δ=∂P/∂S                    | Neutralize linear spot risk     |
|   9   | Short-Answer | Exotic examples                       | –                              | Barrier & forward-start         |
|   10  | Short-Answer | Forward-start insight                 | –                              | Future skew modeling            |
|   11  | Short-Answer | Why BS persists                       | –                              | Tractability & benchmark        |
|   12  | Short-Answer | Convex-order no-arb                   | –                              | Payoff ordering                 |
|   13  | Derivation   | Modeling vs diffusion assumption      | –                              | Hedge drives model              |
|   14  | Short-Answer | Long-dated P\&L source                | –                              | Vol-of-vol correlations         |
|   15  | Short-Answer | Delta vs vega hedging frequency       | –                              | Residual vega risk              |
|   16  | Derivation   | Parabolic PDE from order-2            | –                              | Diffusion characteristic        |
|   17  | Short-Answer | Practical exotic implication          | –                              | Future smile matters            |
|   18  | Short-Answer | Vanilla as hedge                      | –                              | Options hedged by options       |
|   19  | Short-Answer | Chapter digest                        | –                              | Layered modeling insights       |
|   20  | Short-Answer | Implied vs realized volatility        | –                              | Market-based marking            |

Let me know if you’d like any adjustments before moving on to Chapter 2!
