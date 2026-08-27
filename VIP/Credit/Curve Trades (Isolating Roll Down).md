
The core idea is simple:

Do not bet on the absolute level of credit spreads. Bet on the relative shape of the credit curve.

That distinction is extremely important. A naked long-credit position can make money from roll-down, but it is also exposed to the entire credit market moving wider or tighter. A curve trade attempts to remove most of that outright market exposure so that the remaining P&L comes primarily from the relative movement between maturities.

The quoted paragraph is directionally correct, but there is an important refinement: you do not simply buy and sell equal notionals. You hedge the relevant spread risk, typically CS01/DV01-weighted. Otherwise the position is only approximately market-neutral.

  

1. Start with the credit curve

Suppose we have the following CDS curve:

|   |   |
|---|---|
|Maturity|CDS spread|
|1Y|150 bps|
|2Y|170 bps|
|3Y|190 bps|
|4Y|220 bps|
|5Y|250 bps|
|7Y|280 bps|
|10Y|300 bps|

The curve is upward sloping:

$$  
s_{5Y} > s_{4Y}  
$$

Specifically,

$$  
250 - 220 = 30\text{ bps}  
$$

So the 5Y–4Y slope is:

$$  
\boxed{s_{5Y}-s_{4Y}=30\text{ bps}}  
$$

This slope is the quantity we are interested in.

  

2. What is roll-down?

Suppose you sell 5Y protection today.

Selling protection means you receive the CDS premium and take long credit risk.

Ignoring the possibility of default for the moment, your position has the economic exposure:

$$  
\text{Long Credit}  
$$

You initially own a 5Y credit instrument.

Now time passes.

After one year, your CDS has only four years remaining.

So economically you have moved from:

$$  
5Y \longrightarrow 4Y  
$$

This is where the curve matters.

If the credit curve stays unchanged, then after one year your remaining 4Y CDS may be valued using approximately today’s 4Y spread.

Initially:

$$  
s_{5Y}^{0}=250\text{ bps}  
$$

After one year:

$$  
s_{4Y}^{1}\approx s_{4Y}^{0}=220\text{ bps}  
$$

Therefore the spread associated with your position has effectively moved:

$$  
250 \rightarrow 220  
$$

which is a tightening of:

$$  
\Delta s = 220-250=-30\text{ bps}  
$$

For a short-protection position, tightening spreads are profitable.

Therefore:

$$  
\boxed{  
\text{Roll-down P&L}  
\approx  
\text{CS01}\times 30\text{ bps}  
}  
$$

with the precise sign depending on the CS01 convention being used.

  

3. Why is roll-down profitable?

This is the part people often explain badly.

There is no magical free 30 bps being created.

You are exploiting the fact that the credit curve is shaped like:

$$  
250\text{ bps at 5Y}  
$$

versus

$$  
220\text{ bps at 4Y}  
$$

As time passes, your 5Y contract becomes a 4Y contract.

If the entire curve remains unchanged, the contract “slides” down the curve.

Graphically:

Spread

  ^

  |

300|                         *

   |

280|                    *

   |

250|               *    <- Today: 5Y

   |

220|          *         <- Future: remaining 4Y

   |

   +---------------------------------> Maturity

             4Y       5Y

The position starts at the 5Y point.

One year later, it is at the 4Y point.

Therefore:

$$  
250 \rightarrow 220  
$$

That is roll-down.

  

4. But there is a huge problem

Suppose you are correct about the curve.

You expect:

$$  
5Y \rightarrow 4Y  
$$

and therefore expect to earn approximately 30 bps of tightening.

But then the entire credit market suddenly gets much worse.

For example:

$$  
4Y:220\rightarrow300  
$$

and

$$  
5Y:250\rightarrow330  
$$

Now the market has widened massively.

Your long-credit position loses a lot of money.

So even though your original thesis was:

“The 5Y should roll down toward the 4Y.”

you are actually exposed to two different things:

Outright credit risk

$$  
\text{Does the entire credit curve move wider/tighter?}  
$$

Curve risk

$$  
\text{Does 5Y move relative to 4Y?}  
$$

These are different bets.

That distinction is fundamental.

  

5. Decomposing the trade into level + curve

Let’s write the two spreads as:

$$  
s_4  
$$

and

$$  
s_5  
$$

We can think of them approximately as:

$$  
s_4=L-c  
$$

$$  
s_5=L+c  
$$

where:

- $L$ = overall credit-market level
- $c$ = curvature/slope component

This is only a simplified representation, but it gives excellent intuition.

The spread difference is:

$$  
s_5-s_4  
$$

Substituting:

$$  
(L+c)-(L-c)=2c  
$$

The level $L$ cancels.

That is exactly what a curve trade tries to accomplish.

You want:

$$  
\boxed{\text{Exposure to }s_5-s_4}  
$$

rather than:

$$  
\boxed{\text{Exposure to }s_5}  
$$

  

6. Constructing the curve trade

Suppose the curve is:

$$  
s_{4Y}=220  
$$

$$  
s_{5Y}=250  
$$

and you believe the curve will flatten.

You therefore:

Sell 5Y protection

This means:

$$  
\boxed{\text{Long 5Y credit risk}}  
$$

Buy 4Y protection

This means:

$$  
\boxed{\text{Short 4Y credit risk}}  
$$

So the position is:

$$  
\boxed{\text{Long 5Y} - \text{Long 4Y}}  
$$

or, economically:

$$  
\boxed{\text{Sell 5Y protection + Buy 4Y protection}}  
$$

This produces exposure approximately proportional to:

$$  
s_{5Y}-s_{4Y}  
$$

  

7. The intuition behind the hedge

Imagine the market suddenly panics.

Everything widens by 50 bps.

For example:

$$  
220\rightarrow270  
$$

and

$$  
250\rightarrow300  
$$

The curve hasn’t changed.

The 5Y–4Y spread is still:

$$  
300-270=30\text{ bps}  
$$

It was:

$$  
250-220=30\text{ bps}  
$$

So the level moved:

$$  
+50\text{ bps}  
$$

but the curve did not move.

Your curve trade should therefore have approximately zero P&L from this parallel move.

That is the entire point.

  

8. Numerical example

Suppose, for simplicity, that the two CDS positions have the same CS01.

Let:

$$  
CS01 = £50,000/\text{bp}  
$$

per unit of notional.

You:

- sell 5Y protection
- buy 4Y protection

Assume a market shock occurs.

Before shock

$$  
s_4=220  
$$

$$  
s_5=250  
$$

After shock

$$  
s_4=270  
$$

$$  
s_5=300  
$$

Both spreads widened by:

$$  
+50\text{ bps}  
$$

  

8.1 P&L of 5Y position

You are long credit through selling protection.

Widening spreads hurt you:

$$  
P&L_{5Y}  
\approx  
-CS01\times\Delta s_5  
$$

Therefore:

$$

  

P&L_{5Y}

-50,000\times50  
$$

$$  
=-£2.5\text{m}  
$$

  

8.2 P&L of 4Y hedge

You bought protection.

Therefore widening spreads help you:

$$  
P&L_{4Y}  
\approx  
+CS01\times\Delta s_4  
$$

Thus:

$$

  

P&L_{4Y}

50,000\times50  
$$

$$  
=+£2.5\text{m}  
$$

  

8.3 Net P&L

$$

  

P&L_{\text{net}}

-2.5+2.5  
$$

$$  
\boxed{P&L_{\text{net}}\approx0}  
$$

So the entire 50 bp credit-market shock has approximately disappeared.

That is what “isolating the curve” means.

  

9. Now make the curve actually move

Suppose instead that the market moves:

$$  
4Y:220\rightarrow270  
$$

but:

$$  
5Y:250\rightarrow290  
$$

So:

$$  
\Delta s_4=+50  
$$

$$  
\Delta s_5=+40  
$$

The original curve slope was:

$$  
250-220=30  
$$

The new slope is:

$$  
290-270=20  
$$

Therefore:

$$

  

\Delta(s_5-s_4)

20-30

-10\text{ bps}  
$$

The curve flattened by:

$$  
10\text{ bps}  
$$

Your position benefits because you are:

$$  
\text{Long 5Y} - \text{Long 4Y}  
$$

So you want:

$$  
s_5-s_4  
$$

to decrease.

This is exactly a flattening trade.

  

10. P&L of the curve move

With equal CS01:

$$  
P&L  
\approx  
-CS01\Delta s_5  
+  
CS01\Delta s_4  
$$

Factor out CS01:

$$  
P&L  
\approx  
CS01(\Delta s_4-\Delta s_5)  
$$

Substitute:

$$

  

P&L

50,000(50-40)  
$$

$$  
=£500,000  
$$

So:

$$  
\boxed{P&L=+£500k}  
$$

The market widened.

Yet you made money.

Why?

Because you weren’t really betting on credit spreads tightening.

You were betting on:

$$  
\boxed{\text{4Y widening more than 5Y}}  
$$

That is a curve view.

  

11. Why this is closely related to roll-down

Now we can understand the original statement properly.

Suppose initially:

$$  
s_5=250  
$$

$$  
s_4=220  
$$

You sell 5Y protection.

You expect that over time:

$$  
5Y\rightarrow4Y  
$$

and therefore:

$$  
250\rightarrow220  
$$

The roll-down is approximately:

$$  
250-220=30\text{ bps}  
$$

But you do not want to take the full outright exposure associated with being long 5Y credit.

So you hedge it using the 4Y.

The resulting position is effectively a bet that:

$$  
s_5-s_4  
$$

will behave favorably.

Hence:

$$  
\boxed{\text{Curve trade} \approx \text{isolated roll-down trade}}  
$$

with the caveat that real-world implementation requires proper risk matching.

  

12. The mathematical P&L formulation

For small spread changes, CDS P&L can be approximated linearly.

For instrument $i$:

$$  
dP_i\approx -CS01_i,ds_i  
$$

for a long-credit position.

For a protection buyer, the sign reverses.

Suppose:

- 5Y position is long credit
- 4Y position is short credit

Then:

$$

  

dP

-CS01_5,ds_5  
+  
CS01_4,ds_4  
$$

Therefore:

$$

  

\boxed{

  

dP

CS01_4,ds_4-CS01_5,ds_5  
}  
$$

If:

$$  
CS01_4=CS01_5=CS01  
$$

then:

$$

  

dP

CS01(ds_4-ds_5)  
$$

or:

$$  
\boxed{  
dP=-CS01,d(s_5-s_4)  
}  
$$

This is beautiful because it tells us exactly what the trade is doing.

The P&L is approximately proportional to the change in the 5Y–4Y spread.

  

13. Why equal notionals are usually wrong

This is one of the most important practical details missing from the original paragraph.

You generally don’t say:

“I’ll sell £10m 5Y protection and buy £10m 4Y protection.”

and assume the trade is hedged.

The reason is that:

$$  
CS01_{5Y}\neq CS01_{4Y}  
$$

A 5Y CDS generally has a different spread sensitivity from a 4Y CDS.

Therefore, if you want to hedge level exposure, you typically choose notionals so that:

$$  
N_5CS01_5  
\approx  
N_4CS01_4  
$$

Therefore:

$$

  

\boxed{

  

N_4

N_5\frac{CS01_5}{CS01_4}  
}  
$$

This creates a CS01-neutral curve trade.

  

14. Example with different CS01s

Suppose:

$$  
CS01_{5Y}=£60,000/\text{bp}  
$$

and:

$$  
CS01_{4Y}=£50,000/\text{bp}  
$$

You sell:

$$  
N_5=£10m  
$$

5Y protection.

Your 5Y spread risk is:

$$  
10m\times60,000  
$$

The exact scaling depends on how CS01 is quoted, but conceptually we have:

$$  
Risk_5=600  
$$

risk units.

To offset it, you need approximately:

$$  
N_4\times50,000=600  
$$

so:

$$  
N_4=12  
$$

Thus the hedge might be:

$$  
\boxed{  
\text{Sell £10m 5Y}  
+  
\text{Buy £12m 4Y}  
}  
$$

rather than equal notionals.

This is why professional curve trades are generally described in terms of risk-neutral quantities, not merely notionals.

  

15. Level risk versus curve risk

This is probably the most useful mental model.

Think of the spread vector:

$$

  

\mathbf{s}

\begin{pmatrix}  
s_{4Y}\  
s_{5Y}  
\end{pmatrix}  
$$

There are two basic directions in this 2D space.

Parallel shift

$$  
\begin{pmatrix}  
1\  
1  
\end{pmatrix}  
$$

Both maturities move together.

Example:

$$  
\begin{pmatrix}  
220\  
250  
\end{pmatrix}  
\rightarrow  
\begin{pmatrix}  
270\  
300  
\end{pmatrix}  
$$

This is a level move.

  

Curve move

A relative move such as:

$$  
\begin{pmatrix}  
1\  
-1  
\end{pmatrix}  
$$

One maturity moves relative to the other.

For example:

$$  
\begin{pmatrix}  
220\  
250  
\end{pmatrix}  
\rightarrow  
\begin{pmatrix}  
270\  
290  
\end{pmatrix}  
$$

The whole market widened, but the curve flattened.

  

16. The linear algebra interpretation

This is actually a very useful way to think about curve trading.

Suppose your trade has sensitivities:

$$

  

\boldsymbol{\Delta}

\begin{pmatrix}  
\Delta_4\  
\Delta_5  
\end{pmatrix}  
$$

A parallel market move is:

$$

  

d\mathbf{s}

\begin{pmatrix}  
1\  
1  
\end{pmatrix}dL  
$$

Your P&L is:

$$  
dP=  
\boldsymbol{\Delta}^{T}d\mathbf{s}  
$$

Therefore:

$$

  

dP

(\Delta_4+\Delta_5)dL  
$$

To make the portfolio insensitive to parallel shifts, you want:

$$  
\boxed{\Delta_4+\Delta_5=0}  
$$

That is precisely why we construct:

$$  
\text{Long 5Y}  
+  
\text{Short 4Y}  
$$

with appropriate risk weights.

You are effectively forcing the portfolio sensitivity to the level factor to be approximately zero.

What remains is exposure to the slope factor.

This is why curve trading is essentially a form of factor hedging.

  

17. A cleaner factor model

A useful simplified model is:

$$  
s(T)=L+\beta(T)C  
$$

where:

- $L$ = level factor
- $C$ = curve/slope factor
- $\beta(T)$ = maturity loading

Then:

$$  
ds(T)=dL+\beta(T)dC  
$$

For 4Y:

$$  
ds_4=dL+\beta_4dC  
$$

For 5Y:

$$  
ds_5=dL+\beta_5dC  
$$

Construct a portfolio such that the level exposure cancels.

The remaining P&L becomes predominantly:

$$  
\propto  
(\beta_5-\beta_4)dC  
$$

So the curve trade is effectively:

$$  
\boxed{\text{neutral to }L,\quad\text{long/short }C}  
$$

This is exactly the same conceptual framework used in:

- yield-curve trades
- swap spread trades
- CDS curve trades
- bond curve trades
- volatility term-structure trades

  

18. What does “4-year spread jumps to 300 bps” really mean?

The original wording says:

“What if the whole market panics and the 4-year spread jumps to 300 bps?”

This by itself is slightly misleading.

A curve trade doesn’t automatically neutralize the position merely because a 4Y hedge exists.

What matters is how the 4Y and 5Y move relative to their risk weights.

Consider three scenarios.

Scenario A: Parallel widening

$$  
4Y:+50  
$$

$$  
5Y:+50  
$$

Curve:

$$  
\text{unchanged}  
$$

Approximately:

$$  
P&L\approx0  
$$

Good.

  

Scenario B: 4Y widens more

$$  
4Y:+70  
$$

$$  
5Y:+50  
$$

Then:

$$  
s_5-s_4  
$$

decreases.

The curve flattens.

For the long-5Y/short-4Y position:

$$  
\boxed{\text{Profit}}  
$$

  

Scenario C: 5Y widens more

$$  
4Y:+40  
$$

$$  
5Y:+70  
$$

Then:

$$  
s_5-s_4  
$$

increases.

The curve steepens.

For the long-5Y/short-4Y position:

$$  
\boxed{\text{Loss}}  
$$

So the actual statement is:

The hedge protects you from a common market move, not from every possible market move.

  

19. Roll-down versus carry

These concepts are related but not identical.

Carry

Carry is the economics of simply holding the position over time, assuming the relevant market variables do not move.

For CDS, the protection seller receives premium.

Very loosely:

$$  
\text{Carry}  
\approx  
\text{premium received}  
-\text{funding/default-related costs}  
$$

depending on the exact setup.

  

Roll-down

Roll-down is the gain/loss from the position moving to a shorter maturity point on an unchanged curve.

For a 5Y position rolling to 4Y:

$$  
\boxed{  
\text{Roll-down}  
\approx  
CS01\times(s_{5Y}-s_{4Y})  
}  
$$

for the appropriate long-credit direction.

Thus a position can have:

$$  
\text{Positive carry}  
$$

and

$$  
\text{Positive roll-down}  
$$

simultaneously.

That is often the attraction of selling longer-dated protection in an upward-sloping credit curve.

  

20. Why the curve trade is a better expression of the thesis

Suppose your actual belief is:

“The 5Y CDS is expensive relative to the 4Y CDS.”

You could simply sell 5Y protection.

But doing this creates two exposures:

$$  
\text{5Y outright credit risk}  
$$

and

$$  
\text{5Y vs 4Y curve risk}  
$$

If instead you:

$$  
\boxed{  
\text{Sell 5Y protection}  
+  
\text{Buy 4Y protection}  
}  
$$

you are saying:

“I don’t really care where the entire credit market trades. I care that 5Y is expensive relative to 4Y.”

This is much closer to the actual thesis.

  

21. A complete worked example

Let’s construct a realistic simplified trade.

Suppose:

$$  
s_{4Y}=220\text{ bps}  
$$

$$  
s_{5Y}=250\text{ bps}  
$$

Therefore:

$$  
\text{5Y-4Y}=30\text{ bps}  
$$

Suppose:

$$  
CS01_{5Y}=60,000/\text{bp}  
$$

$$  
CS01_{4Y}=50,000/\text{bp}  
$$

You sell:

$$  
10m  
$$

of 5Y protection.

Your 5Y CS01 exposure is:

$$  
600,000/\text{bp}  
$$

To neutralize that, buy enough 4Y protection so:

$$  
N_4\times50,000=600,000  
$$

Therefore:

$$  
N_4=12m  
$$

So your trade is:

$$  
\boxed{  
\text{Sell }£10m\ 5Y  
+  
\text{Buy }£12m\ 4Y  
}  
$$

  

Market shock

Now suppose:

$$  
4Y:220\rightarrow330  
$$

and:

$$  
5Y:250\rightarrow330  
$$

Both widen by different amounts:

$$  
\Delta s_4=+110  
$$

$$  
\Delta s_5=+80  
$$

The 5Y leg loses:

$$

  

600,000\times80

-£48m  
$$

The 4Y leg gains:

$$

  

600,000\times110

+£66m  
$$

Therefore:

$$

  

P&L_{\text{net}}

-48+66  
$$

$$  
\boxed{P&L_{\text{net}}=+£18m}  
$$

Despite an enormous credit-market shock.

The reason is that the 4Y widened by more than the 5Y, so the curve flattened significantly.

  

22. The roll-down interpretation of that trade

Suppose nothing happens to the market except time passing.

Your 5Y protection position becomes effectively a 4Y position.

If the curve is unchanged:

$$  
250\rightarrow220  
$$

Meanwhile the 4Y hedge is already sitting at:

$$  
220  
$$

This means the portfolio is designed around the spread:

$$  
250-220  
$$

The initial curve steepness is:

$$  
30\text{ bps}  
$$

If your position captures that 30 bp transition with little level exposure, then the primary source of expected P&L is the curve’s shape.

That is what traders mean when they say they are “isolating the roll.”

  

23. Why “isolating roll-down” is never perfect

In real markets, several things prevent the trade from being perfectly isolated.

1. CS01 changes through time

As maturity changes:

$$  
CS01_{5Y}\rightarrow CS01_{4Y}  
$$

So your hedge ratio evolves.

You may need to rebalance.

  

2. The curve doesn’t simply shift

Real CDS curves are not just:

$$  
\text{parallel level}+\text{slope}  
$$

There can be:

- level moves
- slope moves
- curvature moves
- basis movements
- liquidity effects
- jump/default risk
- recovery assumptions
- upfront changes
- index/component basis

So a 4Y/5Y trade can still have residual exposure to other factors.

  

3. Default risk is not linear spread risk

The approximation:

$$  
dP\approx CS01,ds  
$$

works for relatively small spread moves.

During a major credit event, the relationship can become highly nonlinear.

Then:

$$  
\boxed{\text{CS01 hedging is not enough}}  
$$

  

4. Roll-down depends on the entire curve

The simplified calculation:

$$  
s_{5Y}-s_{4Y}  
$$

is intuitive, but actual CDS valuation depends on:

- discount factors
- survival probabilities
- hazard rates
- recovery
- premium-leg cash flows
- default-leg cash flows
- accrued premium
- payment dates
- curve interpolation

So the actual roll-down should ideally be calculated by re-pricing the trade under a one-year passage of time while freezing the market curves.

  

24. The rigorous definition of roll-down P&L

Suppose today’s portfolio value is:

$$  
V(t,\mathbf{s}(t))  
$$

One year later, for a pure roll-down calculation, we hold the market environment conceptually fixed and let only time pass:

$$  
V(t+\Delta t,\mathbf{s}_{\text{frozen}})  
$$

Then:

$$

  

\boxed{

  

P&L_{\text{roll}}

V(t+\Delta t,\mathbf{s}_{\text{frozen}})

V(t,\mathbf{s}_{\text{current}})  
}  
$$

This is the rigorous definition.

The simple:

$$  
CS01\times\Delta spread  
$$

formula is just a first-order approximation.

  

25. A more complete P&L decomposition

For a curve trade, you can think of total P&L approximately as:

$$

  

dP

\underbrace{\text{Carry}}{\text{premium/funding}}  
+  
\underbrace{\text{Roll-down}}{\text{aging along curve}}  
+  
\underbrace{\text{Curve move}}{\text{relative maturity move}}  
+  
\underbrace{\text{Level move}}{\text{market-wide credit move}}  
+  
\underbrace{\text{Convexity}}{\text{nonlinear effects}}  
+  
\underbrace{\text{Basis/liquidity}}{\text{miscellaneous}}  
$$

The goal of a curve trade is to make:

$$  
\boxed{\text{Level move}\approx0}  
$$

while retaining:

$$  
\boxed{\text{Curve/roll exposure}}  
$$

That is the fundamental objective.

  

26. Steepener versus flattener

This terminology is worth getting exactly right.

Suppose:

$$  
\text{Slope}=s_{5Y}-s_{4Y}  
$$

If you expect:

$$  
s_{5Y}-s_{4Y}\uparrow  
$$

the curve steepens.

You want to profit from:

$$  
\boxed{\text{5Y widening relative to 4Y}}  
$$

or equivalently:

$$  
\boxed{\text{5Y tightening less than 4Y}}  
$$

  

If you expect:

$$  
s_{5Y}-s_{4Y}\downarrow  
$$

the curve flattens.

You want to profit from:

$$  
\boxed{\text{4Y widening relative to 5Y}}  
$$

or:

$$  
\boxed{\text{5Y tightening relative to 4Y}}  
$$

For the example we’ve been discussing:

$$  
\boxed{  
\text{Sell 5Y protection}  
+  
\text{Buy 4Y protection}  
}  
$$

is a position that benefits from the 5Y–4Y spread tightening/flattening, after proper risk weighting.

  

27. The most important intuition

Forget CDS for a moment.

Imagine two bonds:

- Bond A: 4 years, yield spread = 220 bps
- Bond B: 5 years, yield spread = 250 bps

You believe the 5Y is rich in spread terms relative to the 4Y.

Buying/selling them outright exposes you to the whole credit market.

Instead, you construct:

$$  
\text{Long one}  
+  
\text{Short the other}  
$$

so that a common market movement cancels.

Then you are effectively trading:

$$  
\boxed{\text{Relative richness}}  
$$

rather than:

$$  
\boxed{\text{Absolute credit direction}}  
$$

That is exactly what a curve trade is.

  

28. Connecting this directly to “sell 5Y protection, buy 4Y protection”

Let’s translate every word.

“Sell 5Y protection”

You receive the premium.

Economically:

$$  
\boxed{\text{Long 5Y credit}}  
$$

You profit if 5Y spreads tighten.

  

“Buy 4Y protection”

You pay premium.

Economically:

$$  
\boxed{\text{Short 4Y credit}}  
$$

You profit if 4Y spreads widen.

  

Combined

You are:

$$

  

\boxed{

  

\text{Long 5Y}

\text{Long 4Y}  
}  
$$

Therefore:

- 5Y tightening relative to 4Y → profit
- 4Y widening relative to 5Y → profit
- parallel widening → approximately hedged
- parallel tightening → approximately hedged

So the position has become predominantly a relative-value trade.

  

29. One subtle but important distinction

There are actually two different concepts people sometimes lazily call “roll-down.”

Expected roll-down

The gain predicted from the current curve shape under the assumption that the curve remains unchanged as time passes.

Realized curve P&L

What actually happens because the curve moves.

These are not the same.

Suppose you expect:

$$  
30\text{ bps}  
$$

of roll-down.

But after one year the curve changes dramatically.

You might realize:

$$  
-10\text{ bps}  
$$

even though the initial curve suggested:

$$  
+30\text{ bps}  
$$

This is why saying:

“The curve is steep, therefore roll-down is free money”

is nonsense.

The roll-down is an expected mechanical effect under a particular market assumption. The market is not contractually obligated to cooperate.

  

30. Practical trader formulation

A trader might therefore think:

View: 5Y CDS is rich versus 4Y.

Expression: Sell 5Y protection, buy risk-weighted 4Y protection.

Hedge objective: Neutralize aggregate credit CS01.

Expected return: Carry + roll-down + expected curve convergence.

Main risk: 5Y–4Y curve steepens instead.

Secondary risks: Curvature, basis, jump-to-default, liquidity, hedge slippage, changing CS01.

This is much more precise than saying simply:

“I’m long credit because I like roll-down.”

  

31. Compact mathematical summary

Let:

$$  
S_T=\text{CDS spread at maturity }T  
$$

and:

$$  
CS01_T=\frac{\partial P}{\partial S_T}  
$$

approximately.

A long-credit 5Y position has:

$$  
dP_5\approx-CS01_5,dS_5  
$$

A short-credit 4Y hedge has:

$$  
dP_4\approx+CS01_4,dS_4  
$$

Therefore:

$$

  

\boxed{

  

dP

-CS01_5,dS_5  
+  
CS01_4,dS_4  
}  
$$

Choose notionals so:

$$  
\boxed{  
CS01_5\approx CS01_4  
}  
$$

Then:

$$  
dP  
\approx  
CS01(dS_4-dS_5)  
$$

Therefore:

$$  
\boxed{  
dP\approx-CS01,d(S_5-S_4)  
}  
$$

So:

$$  
\boxed{  
\text{P&L} \propto -\Delta(\text{5Y-4Y spread})  
}  
$$

for this particular trade direction.

That single equation captures the entire idea.

  

32. Summary table

|   |   |
|---|---|
|Concept|Meaning|
|Sell 5Y protection|Long 5Y credit risk|
|Buy 4Y protection|Short 4Y credit risk|
|Combined position|Long 5Y / short 4Y|
|Main exposure|5Y–4Y relative spread|
|Parallel widening|Approximately hedged|
|Parallel tightening|Approximately hedged|
|5Y tightens more than 4Y|Profit|
|4Y widens more than 5Y|Profit|
|5Y widens more than 4Y|Loss|
|5Y–4Y spread decreases|Profit|
|5Y–4Y spread increases|Loss|
|Roll-down|P&L from aging along an unchanged curve|
|Carry|Economics of holding the position|
|CS01 hedge|Risk-weighted hedge, not necessarily equal notional|
|Main objective|Remove level exposure and retain curve exposure|

  

33. The mental picture to keep

The cleanest way to remember the whole thing is:

                 CREDIT CURVE

  

Spread

  ^

  |

  |                         ● 5Y

  |                      /

  |                   /

  |                /

  |             ● 4Y

  |

  +----------------------------------> Maturity

  

  

Trade:

    SELL 5Y protection

        +

    BUY 4Y protection

        ↓

    LONG 5Y / SHORT 4Y

        ↓

    Neutralize LEVEL

        ↓

    Keep CURVE exposure

        ↓

    Harvest ROLL-DOWN / curve convergence

And the key equation is:

$$  
\boxed{  
P&L_{\text{curve}}  
\approx  
-CS01,  
\Delta(S_{5Y}-S_{4Y})  
}  
$$

So the phrase “isolating roll-down” really means:

Constructing the position so that a broad credit-market move is largely hedged, leaving the P&L driven primarily by where one maturity sits relative to another as the position rolls through the curve.

That distinction between outright spread risk and relative curve risk is the foundation of practically every serious curve-trading discussion.