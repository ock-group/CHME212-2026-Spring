# Lab 09 Lecture Script: Monte Carlo Simulation

---

## Overview

Today's lab is about **Monte Carlo simulation** — a technique for propagating uncertainty through a model by repeated random sampling. The core workflow is always the same:

1. Identify which inputs are uncertain and characterize their distributions
2. Draw a large number of random samples from those distributions
3. Run each sample through your model
4. Analyze the output distribution to answer engineering questions (failure rates, percentiles, etc.)

We'll go through three instructor examples, then you'll work through problems on your own.

---

## Example 1: Estimating π with Monte Carlo

This is the classic "dart--throwing" demonstration. The idea: randomly throw darts at a unit square. The fraction that land inside the quarter-circle of radius 1 is approximately π/4.

**The geometry:**
- Square: x ∈ [0, 1], y ∈ [0, 1]
- Quarter-circle condition: x² + y² ≤ 1

So if we throw N darts and count the ones inside the circle:

```
π ≈ 4 × (number inside) / N
```

**Walk through the code:**

```python
rng = np.random.default_rng(seed=0)
N   = 10_000

x = rng.random(N)
y = rng.random(N)
inside = (x**2 + y**2) <= 1.0

pi_est = 4 * np.sum(inside) / N
```

- `rng.random(N)` gives N uniform samples in [0, 1)
- `inside` is a boolean array — True where the dart landed in the circle
- `np.sum(inside)` counts the True values

With N = 10,000, you get about 3–4 decimal places of accuracy. The error shrinks as 1/√N — we'll see this convergence idea again in Example 3.

**Key takeaway:** Monte Carlo is flexible. Geometry that would be hard to integrate analytically becomes trivial to simulate.

---

## Example 2: Process Reliability — Propagating Input Uncertainty

Now we apply Monte Carlo to a real engineering scenario. A reactor must produce product with **purity ≥ 97.5%**, but process conditions vary batch to batch:

- Residence time: τ ~ N(120, 12²) s
- Temperature: T ~ N(400, 8²) K

The purity model is:

```
Purity = 100 × (1 − 0.2 × exp(−k(T) × τ))
k(T) = exp(−1550 / T)
```

**The simulation:**

```python
tau_mc = rng_mc.normal(tau_mean, tau_std, N_mc)   # sample τ
T_mc   = rng_mc.normal(T_mean,   T_std,   N_mc)   # sample T
pur_mc = purity_model(tau_mc, T_mc)                # vectorized model call
```

Notice: we call `purity_model` **once** on the entire array. NumPy handles all 50,000 evaluations simultaneously — no explicit loop needed.

**Reading the results:**
- Mean purity tells you where the distribution is centered
- Std tells you how spread out it is
- Failure rate = fraction of batches below the spec limit

**The histogram** shows the full output distribution. The spec line (red dashed) lets you visually see how much of the distribution falls below the threshold.

**Key takeaway:** Monte Carlo turns "how often does my process fail?" into a counting problem. Sample → evaluate → count.

---

## Example 3: Convergence — How Many Samples Are Enough?

A natural question: how large does N need to be? The answer depends on what you're estimating.

- For the **mean**, you converge quickly (a few thousand samples is often enough)
- For **tail probabilities** (rare failures), you need many more — if the true failure rate is 0.1%, you need at least ~10,000 samples just to see a handful of failures

**The running failure rate plot** shows this directly. Track the cumulative failure estimate as more batches are added:

```python
running_fail = np.cumsum(pur_cv < spec) / np.arange(1, N_total + 1)
```

- `(pur_cv < spec)` → boolean array of failures
- `np.cumsum(...)` → cumulative count of failures at each step
- Dividing by the number of samples so far gives the running fraction

We plot this on a **log scale** for the x-axis (`ax.semilogx`) so you can see the early noisy behavior and the later convergence in the same figure.

**Rule of thumb:** To estimate a failure rate of p% with reasonable precision, you want at least 10/p samples. For a 1% failure rate, that's at least ~1,000. For 0.01%, you need ~100,000.

---

## Warm-Up Exercises

Before the main problems, four quick exercises to build familiarity with the building blocks.

### Exercise 1 — Sample and characterize

Draw 1,000 samples from N(500, 20²) and compute the sample mean, std, and fraction exceeding 530.

The sample mean (~498.6) and std (~19.9) won't match the true parameters exactly — that's sampling variability. With larger N they'd get closer.

### Exercise 2 — Single uncertain input through a model

C₀ ~ N(2.0, 0.1²) mol/L goes through a first-order reactor:

```
C_out = C0 × exp(−k × t)
```

With k = 0.5 min⁻¹ and t = 3 min fixed. Notice that the mean of C_out is close to C₀_mean × exp(−k×t) = 2.0 × e^(−1.5) ≈ 0.446 mol/L. The small std (0.022) reflects the ×exp(−1.5) scaling of the C₀ uncertainty.

### Exercise 3 — Multiple uncertain inputs

Now k and t are also uncertain. When you add uncertainty in both k and t:
- The mean barely changes (0.453 vs 0.447)
- The **std increases significantly** (0.082 vs 0.022) — more uncertainty sources → wider output distribution
- A small but nonzero fraction falls below 0.25 mol/L

This illustrates why it matters to identify **all** major uncertainty sources.

### Exercise 4 — Percentiles and visualization

From the Exercise 3 distribution, compute the 5th and 95th percentiles:
```python
p5  = np.percentile(C_out2, 5)
p95 = np.percentile(C_out2, 95)
```

The 5th percentile is the value below which 5% of batches fall — a useful way to characterize worst-case scenarios without focusing on the extreme tail.

---

## Practice Problems

### Problem 1: Heat Exchanger — Outlet Temperature Reliability

The model is the "effectiveness" formulation:

```
T_out = T_in + η × (T_hot − T_in)
```

Three uncertain inputs: T_in ~ N(25, 3²), T_hot ~ N(120, 5²), η ~ N(0.75, 0.04²). Spec: T_out ≥ 80°C.

**(a)** Run 50,000 samples and compute statistics.

Results: Mean ≈ 96.3°C, Std ≈ 5.4°C, Failure rate ≈ 0.08%. The process is well above spec on average — but not zero failures.

**(b)** Histogram with spec line.

Look at where the spec line sits relative to the distribution bulk. Even a small tail can represent thousands of failed batches per year if you run many batches.

**(c) Sensitivity analysis — one-at-a-time (OAT)**

To find the most influential input, hold two at their mean values and vary only one:

```python
T_out_Tin  = T_out_model(T_in_s,  120,     0.75)   # only T_in varies
T_out_Thot = T_out_model(25,      T_hot_s, 0.75)   # only T_hot varies
T_out_eta  = T_out_model(25,      120,     eta_s)   # only eta varies
```

Results:
- σ(T_out | only T_in varies)  ≈ 0.75°C
- σ(T_out | only T_hot varies) ≈ 3.73°C
- σ(T_out | only η varies)     ≈ 3.81°C

**Conclusion:** η (effectiveness) is the most influential input. If you want to reduce output variability, improving consistency of the heat exchanger effectiveness will have the most impact — not tighter control of inlet temperatures.

---

### Problem 2: Batch Reactor Yield — Operating Condition Optimization

The Arrhenius-based model:

```
k(T) = A × exp(−Ea/R / T)
Y    = 100 × (1 − exp(−k × τ))
```

With A = 2×10⁶ min⁻¹, Ea/R = 6000 K. Baseline: τ ~ N(30, 3²) min, T ~ N(370, 10²) K. Spec: Y ≥ 85%.

**(a)** Baseline simulation.

Mean yield ≈ 98.2%, Std ≈ 3.5%, Failure rate ≈ 1.56%. Despite a high mean, the wide temperature distribution causes a non-trivial failure rate.

**(b)** Raise mean temperature to 380 K.

Failure rate drops to 0.09% — a reduction of 1.47 percentage points. Higher temperature → faster reaction → higher yield on average.

**(c)** Tighten temperature control: σ_T = 5 K (mean stays at 370 K).

Failure rate drops to 0.00% (with 50,000 samples, no failures observed). This is **more effective** than raising the mean temperature.

**Why?** The yield curve is nonlinear and the distribution is already centered well above the spec. The problem isn't the mean — it's the spread. Tightening σ_T cuts off the low-yield tail without needing to push the average higher.

This is a general lesson: **sometimes reducing variability is more effective than shifting the mean**.

**(d)** Overlaid histograms.

Plot all three scenarios together with the spec line. You can visually confirm:
- Baseline: leftmost, most spread out, visibly overlapping with the spec
- Higher T: shifted right but same width
- Tighter control: narrower, even if centered the same as baseline

---

## Key Takeaways

1. **The MC workflow is always the same:** sample inputs → evaluate model → analyze output distribution
2. **Vectorize:** pass the full sample array through your model at once — no loops
3. **More samples = more accurate estimates**, especially for rare events. Error scales as 1/√N
4. **Percentiles and failure rates** translate the output distribution into engineering decisions
5. **Sensitivity analysis** (OAT) identifies which inputs drive output variability — guide where to focus improvement effort
6. **Reducing variance can be more effective than shifting the mean** when your process is already well-centered above spec
