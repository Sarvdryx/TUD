# Comprehensive Roadmap: Markov Chain Modeling & Forecasting for VIC Stock

**Course:** Applied Mathematics & Statistics  
**Target:** 10.0/10.0 Base Score + 1.0 Bonus Point  
**Asset Choice:** Vingroup Joint Stock Company (Ticker: `VIC`)  

---

## 1. Executive Summary & Project Objectives

The primary objective of this project is to apply **Discrete-Time Markov Chains (DTMC)** to model, analyze, and forecast market regime transitions for **VIC stock** using a minimum of 5 years of daily financial time-series data. 

### Key Academic & Methodological Rules
- **Pure Implementation Constraint:** External libraries (e.g., `pandas`, `numpy`) are strictly restricted to data loading, basic array operations, and result validation. **Core mathematical computations** (log-returns, threshold segmentation, transition counting, state matrix construction, power iteration/linear system solvers for stationary distribution) **must be implemented from scratch**.
- **Deliverables:** Uncompressed `.ipynb` Jupyter Notebook (formatted as an academic paper with Markdown & LaTeX), exported `.pdf` from the notebook, and raw `.csv` or `.xlsx` dataset file.

---

## 2. Milestone Roadmap & Implementation Steps

```
┌─────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: Data Collection & Threshold Segmentation (2.0 pts)             │
├─────────────────────────────────────────────────────────────────────────┤
│ PHASE 2: Custom Markov Transition Matrix & Directed Graph (2.5 pts)     │
├─────────────────────────────────────────────────────────────────────────┤
│ PHASE 3: Multi-Step Forecasting & Stationary Distribution (2.5 pts)     │
├─────────────────────────────────────────────────────────────────────────┤
│ PHASE 4: Financial Economic Interpretation & Model Evaluation (1.5 pts) │
├─────────────────────────────────────────────────────────────────────────┤
│ PHASE 5: BONUS — Multi-Dimensional State Space (Price + Vol) (+1.0 pt)  │
├─────────────────────────────────────────────────────────────────────────┤
│ PHASE 6: Academic Notebook Formatting & Submission Check (1.5 pts)      │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Data Collection & Preprocessing (2.0 Points)

### 1.1 Data Retrieval
- **Time Window:** Retrieve daily adjusted closing prices ($P_t$) and trading volume ($V_t$) for `VIC` covering at least **5 years** (e.g., January 2019 to Present; ~1,250+ trading days).
- **Sources:** `vnstock` library, Yahoo Finance API, or direct `.csv` exports from CafeF / Vietstock.

### 1.2 Daily Log-Return Calculation
Compute daily logarithmic returns:
$$R_t = \ln\left(\frac{P_t}{P_{t-1}}\right)$$

*Implementation Note:* Implement the natural logarithm ratio explicitly over price series rather than relying on high-level financial functions.

### 1.3 State Space Definition ($S$) & Mathematical Justification
Divide the continuous log-return distribution into 3 discrete market regimes:
- $S = 0$ **(Bear / Sharp Decline):** $R_t < \mu - 0.5\sigma$
- $S = 1$ **(Sideway / Consolidation):** $\mu - 0.5\sigma \le R_t \le \mu + 0.5\sigma$
- $S = 2$ **(Bull / Strong Growth):** $R_t > \mu + 0.5\sigma$

#### Academic Justification Strategy:
1. Calculate sample mean ($\mu$) and standard deviation ($\sigma$) of VIC's daily log-returns.
2. Cite empirical literature in quantitative finance (e.g., standard deviation classification thresholds) demonstrating that a $0.5\sigma$ boundary creates balanced regime frequencies while isolating tail risk events.
3. Present summary statistics: Mean, Standard Deviation, Skewness, Kurtosis, and a histogram overlay with threshold bounds.

---

## Phase 2: Custom Markov Transition Matrix & State Graph (2.5 Points)

### 2.1 Transition Count & Probability Matrix ($P$)
Build a custom algorithm to calculate the $3 \times 3$ Transition Matrix $P$:

1. Iterate through the historical state sequence $\{S_1, S_2, \dots, S_T\}$.
2. Count transitions $n_{ij}$ from state $i$ at day $t$ to state $j$ at day $t+1$.
3. Compute maximum likelihood transition probabilities:
$$P_{ij} = P(S_{t+1} = j \mid S_t = i) = \frac{n_{ij}}{\sum_{k=0}^{2} n_{ik}}$$

### 2.2 Mathematical Validation
Verify row-stochasticity for all rows $i \in \{0, 1, 2\}$:
$$\sum_{j=0}^{2} P_{ij} = 1.0, \quad P_{ij} \ge 0$$

### 2.3 Directed State Transition Graph
- Construct a directed graph using `networkx` or `matplotlib`.
- Nodes represent regimes $\{0: \text{Bear}, 1: \text{Sideway}, 2: \text{Bull}\}$.
- Directed edge weights correspond to non-zero transition probabilities $P_{ij}$.

---

## Phase 3: Short-Term Forecasting & Stationary Distribution (2.5 Points)

### 3.1 State Probability Forecasting ($t = 1, 5, 20$ Trading Days)
Given an initial state vector $\mathbf{v}_0$ representing a strong Bear market scenario:
$$\mathbf{v}_0 = [1.0, \, 0.0, \, 0.0]$$

Compute the probability vector at horizon $t$:
$$\mathbf{v}_t = \mathbf{v}_0 \cdot P^t$$

- Calculate $\mathbf{v}_1$ (1-day horizon), $\mathbf{v}_5$ (1-week horizon), and $\mathbf{v}_{20}$ (1-month horizon).
- Explicitly implement matrix power multiplication ($P^t$).

### 3.2 Stationary Distribution ($\boldsymbol{\pi}$) Computation
Find the stationary distribution vector $\boldsymbol{\pi} = [\pi_0, \pi_1, \pi_2]$ satisfying:
$$\boldsymbol{\pi} P = \boldsymbol{\pi} \quad \text{and} \quad \sum_{i=0}^{2} \pi_i = 1$$

#### Algebraic Formulation:
Rewrite as a linear system $(P^T - I)\boldsymbol{\pi}^T = \mathbf{0}$ subject to $\sum \pi_i = 1$:
$$\begin{bmatrix} P_{00}-1 & P_{10} & P_{20} \\ P_{01} & P_{11}-1 & P_{21} \\ 1 & 1 & 1 \end{bmatrix} \begin{bmatrix} \pi_0 \\ \pi_1 \\ \pi_2 \end{bmatrix} = \begin{bmatrix} 0 \\ 0 \\ 1 \end{bmatrix}$$

Solve using custom Gaussian elimination or `numpy.linalg.solve` for validation.

#### Economic Interpretation:
Interpret $\pi_i$ as the long-term equilibrium probability (unconditional proportion of time) VIC stock spends in regime $i$.

---

## Phase 4: Financial Economic Interpretation & Evaluation (1.5 Points)

### 4.1 Real-World Market Alignment
- Relate transition probabilities to major macroeconomic events affecting Vingroup (e.g., VinFast NASDAQ listing, corporate bond issuance cycles, real estate market fluctuations).
- Compare state persistence ratios $P_{ii}$ (diagonal elements) to determine VIC's regime stickiness.

### 4.2 Critical Evaluation of Markov Property Assumptions
1. **Memorylessness Assessment:** The First-Order Markov assumption states $P(S_{t+1} \mid S_t, S_{t-1}, \dots) = P(S_{t+1} \mid S_t)$. Evaluate whether historical price paths contain long-memory effects.
2. **Key Limitations in Financial Markets:**
   - Volatility Clustering (GARCH effects).
   - Non-stationarity of transition probabilities over time (macro shock shifts).
   - Autocorrelation in log-returns.

---

## Phase 5: BONUS — Multi-Dimensional State Space (+1.0 Point)

To achieve maximum bonus credit, extend the state space from 1D (Price Return) to **2D (Price Return $\times$ Trading Volume)**.

### 5.1 Volume Regime Classification
1. Calculate daily volume growth rate:
$$\Delta V_t = \frac{V_t - V_{t-1}}{V_{t-1}}$$
2. Classify Volume into 2 Regimes (or relative to 20-day Moving Average):
   - $V = 0$ **(Low / Normal Liquidity):** $\Delta V_t \le 0$
   - $V = 1$ **(High / Spiking Liquidity):** $\Delta V_t > 0$

### 5.2 Combined 6-Regime Space ($3 \times 2 = 6$)
Define state indices $K \in \{0, 1, 2, 3, 4, 5\}$:
- State 0: Bear + Low Volume
- State 1: Bear + High Volume (Panic Selling)
- State 2: Sideway + Low Volume (Accumulation)
- State 3: Sideway + High Volume (Consolidation Shift)
- State 4: Bull + Low Volume (Weak Rally)
- State 5: Bull + High Volume (Institutional Breakout)

### 5.3 2D Transition Matrix ($6 \times 6$) & Stationary Distribution
1. Build the $6 \times 6$ transition probability matrix $P_{6\times6}$.
2. Solve for the 6-state stationary distribution $\boldsymbol{\pi}_{6D}$.
3. Analyze key structural insights (e.g., probability of transitioning from "Bear + High Volume" to "Bull + High Volume").

---

## Phase 6: Academic Formatting & Submission Checklist (1.5 Points)

### 6.1 Notebook Quality Standards
- **Zero Execution Errors:** Ensure top-to-bottom clean run (`Kernel -> Restart & Run All`).
- **LaTeX Math Rendering:** All equations formatted in standard LaTeX ($R_t$, $\boldsymbol{\pi}$, $P^t$).
- **Visual Polish:** High-DPI charts with proper titles, legends, color-coded regimes, and clear axis labels.

### 6.2 Naming & Packaging Rules
- Files required in submission folder (no `.zip` or `.rar` archives):
  1. `[MSSV]-[Full Name].ipynb`
  2. `[MSSV]-[Full Name].pdf` (Exported directly from `.ipynb`)
  3. `VIC_historical_data.csv` (Raw data source)

---

## Summary Checklist for Maximum Score

| Rubric Item | Weight | Requirement Strategy |
| :--- | :---: | :--- |
| **Data Preprocessing** | 2.0 pts | >5 yr VIC data, explicit log-returns, statistical mean/std regime thresholds |
| **Markov Model** | 2.5 pts | Custom matrix algorithm, row-stochastic validation, clear directed graph |
| **Forecasting & $\boldsymbol{\pi}$** | 2.5 pts | $t=1, 5, 20$ projections, exact linear system solution for stationary distribution |
| **Economic Analysis** | 1.5 pts | Deep market context alignment, critical critique of memorylessness assumption |
| **Formatting & Skills** | 1.5 pts | Clean code, professional layout, LaTeX equations, crisp visualizations |
| **BONUS Feature** | **+1.0 pt** | **2D Joint State Space (Price Return $\times$ Volume Liquidity)** |
