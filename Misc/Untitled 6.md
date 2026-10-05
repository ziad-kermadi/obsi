---
title: "Untitled 6"
tags: [inbox]
type: note
status: draft
created: 2025-06-10
updated: 2026-10-05
---
Below are four “next-level” alpha ideas—each digs deeper into market microstructure, cross-asset relationships, or latent regimes. I’ll give you intuition, the core formula, and a sketch of how you’d implement it in Python. At the end is a comparative table so you can decide which to explore first.

---

## 1. **Order-Flow Imbalance (OFI) Alpha**

**Intuition:**  
Large buyer-initiated trades push price up; large seller-initiated trades push it down. The **net imbalance** over a short window often leads the immediate price move.

**Core Formula:**

OFIt=∑i∈B(t)Qi  −  ∑j∈S(t)Qj\text{OFI}_t = \sum_{i \in \mathcal{B}(t)} Q_i \;-\;\sum_{j \in \mathcal{S}(t)} Q_j

where B(t)\mathcal{B}(t) are buyer-initiated trades in window tt (price > mid), S(t)\mathcal{S}(t) seller-initiated (price < mid), and QQ is volume.

**Signal:**

signalt={+1if OFIt>θofi−1if OFIt<− θofi0otherwise\text{signal}_t = \begin{cases} +1 & \text{if }\text{OFI}_t > \theta_{\mathrm{ofi}}\\ -1 & \text{if }\text{OFI}_t < -\,\theta_{\mathrm{ofi}}\\ 0 & \text{otherwise} \end{cases}

**Implementation Sketch:**

```python
# assume `trades` DataFrame has columns ['datetime','price','qty','mid']
trades['side'] = np.where(trades.price >= trades.mid, +1, -1)
trades['ofi_contrib'] = trades.side * trades.qty

# resample to 1-min windows
ofi = trades.ofi_contrib.resample('1min').sum()

# z-score normalize over rolling window
ofi_z = (ofi - ofi.rolling(30).mean()) / ofi.rolling(30).std()

# entry/exit thresholds
signal = np.sign(ofi_z).where(ofi_z.abs() > 1.5, 0)
```

---

## 2. **VWAP Deviation (“Anchored Momentum”)**

**Intuition:**  
Price deviations from the Volume-Weighted Average Price over the past TT minutes often mean-revert intraday—because institutional executions target VWAP.

**Core Formula:**

VWAPt=∑i=1TPt−i Qt−i∑i=1TQt−i,Δt=Pt−VWAPtVWAPt\text{VWAP}_t = \frac{\sum_{i=1}^T P_{t-i}\,Q_{t-i}}{\sum_{i=1}^T Q_{t-i}} \quad,\quad \Delta_t = \frac{P_t - \text{VWAP}_t}{\text{VWAP}_t}

**Signal:**

signalt={+1Δt<−δ−1Δt>+δ0∣Δt∣≤δ\text{signal}_t = \begin{cases} +1 & \Delta_t < -\delta\\ -1 & \Delta_t > +\delta\\ 0 & |\Delta_t| \le \delta \end{cases}

**Implementation Sketch:**

```python
# trades as before
trades['vwap_num'] = trades.price * trades.qty
res = trades.resample('1min').agg({
    'vwap_num':'sum','qty':'sum','price':'last'
})
res['vwap'] = res.vwap_num / res.qty
res['delta'] = (res.price - res.vwap) / res.vwap

# thresholds
delta = 0.001  # 0.1%
res['signal'] = 0
res.loc[res.delta < -delta, 'signal'] = 1
res.loc[res.delta >  delta, 'signal'] = -1
```

---

## 3. **Cross-Market Residual Momentum**

**Intuition:**  
XLKQ should follow synthetic = XLK × FX. After regressing one on the other, the **residual** often mean-reverts or trends short-term.

**Core Steps & Formula:**

1. Build synthetic:  
    St=PtXLK×FXt×100\displaystyle S_t = P^{XLK}_t \times \mathrm{FX}_{t}\times100
    
2. Fit rolling regression (window ww):  
    PtXLKQ=αt+βt St+εt\displaystyle P^{XLKQ}_t = \alpha_t + \beta_t\,S_t + \varepsilon_t
    
3. Residual z-score:  
    Zt=εt−εˉstσε,st\displaystyle Z_t = \frac{\varepsilon_t - \bar\varepsilon_{st}}{\sigma_{\varepsilon,st}}
    

**Signal:**  
Same as spread-z in your Hybrid: go long when Zt<−ζZ_t<-\zeta, short when Zt>ζZ_t>\zeta.

**Implementation Sketch:**

```python
df = pd.DataFrame({'x': syn, 'y': real})
roll = df.rolling(60)
beta = roll.cov().y.div(roll.var().x)
df['resid'] = df.y - beta*df.x
df['z'] = (df.resid - df.resid.rolling(60).mean()) / df.resid.rolling(60).std()
signal = np.sign(df.z).where(df.z.abs()>2, 0)
```

---

## 4. **Regime-Switching (HMM) Signal**

**Intuition:**  
Markets switch between “trending” and “mean-reverting” regimes. A 2-state Hidden Markov Model on returns & vol can identify the regime—and you trade differently in each.

**Core Steps:**

1. Fit a 2-state HMM on  
    {rt,∣rt∣}\{r_t, |r_t|\}
    
2. Decode most likely state s^t\hat s_t.
    
3. **In “mean-reverting”** (lower mean, higher vol) → use OFI or VWAP alpha.  
    **In “trending”** → use momentum crossover or VWAP-momentum.
    

**Implementation Sketch:**

```python
from hmmlearn import GaussianHMM

X = np.column_stack([df.ret.values, np.abs(df.ret.values)])
model = GaussianHMM(n_components=2).fit(X)
states = model.predict(X)
df['state'] = states

# Combine with OFI
df['base_signal'] = ...  # from OFI or VWAP
df['signal'] = df.apply(
    lambda r: r.base_signal if r.state==0 else np.sign(r.ret),
    axis=1
)
```

---

### 🔢 Comparative Table

|Alpha|Data Required|Window|Turnover|Pros|Cons|
|---|---|---|---|---|---|
|**Order-Flow Imbalance**|Tick trades (price, qty)|5–30 min|Med–High|Direct microstructure edge|Noisy, requires clean timestamps|
|**VWAP Deviation**|Tick trades|5–60 min|Med|Targets institutional patterns|Needs volume data, whipsaw early|
|**Residual Momentum**|US ETF + FX + UK ETF bars|60 min (roll)|Low–Med|Exploits cross-market inefficiency|Depends on FX quality, stale data|
|**HMM Regime Mix**|Returns + absolute returns|Entire history|Adaptive|Hybrid alpha switching|Complex, needs model calibration|

---

**Next Steps:** pick one or two, backtest them (you already have the framework), compare annualized Sharpe/turnover, and then refine your thresholds. Let me know which you’d like code fleshed out further!