# Lab 10 Lecture Script: Numerical Differentiation & Integration

---

## Overview

**Learning objectives for today:**
- Compute derivatives of discrete data with `np.gradient`
- Integrate discrete data with `scipy.integrate.trapezoid`
- Integrate Python functions with `scipy.integrate.quad`
- Apply these tools to real chemical engineering problems (reactor design, thermodynamics)

**Structure:** ~10 min instructor demo → ~10 min warm-up → ~30 min practice problems

---

## Part 1: Instructor Demo

### Framing

> "Today we're connecting calculus to data. In the real world you almost never have a clean analytical function — you have measurements at discrete time points, or tabulated values from a reference. Python gives us three workhorses for these situations."

Write on board or show the summary table:

| Task | Tool | Key syntax |
|------|------|-----------|
| Derivative of discrete data | `np.gradient` | `np.gradient(y, x)` |
| Integral of discrete data | `trapezoid` | `trapezoid(y, x)` |
| Integral of a Python function | `quad` | `result, err = quad(f, a, b)` |

---

### Example 1: Reaction Rate from Concentration Data (`np.gradient`)

**Setup — motivate the problem:**

> "Imagine you're running a batch reactor. You can't measure reaction rate directly — you measure concentration at a few time points. But the rate is just the derivative of concentration with respect to time. So we need numerical differentiation."

Write the governing equation on the board:
$$r_A = -\frac{dC_A}{dt}$$

**Walk through the code:**

```python
t   = np.array([0,    10,   20,   30,   40,   50,   60  ])  # s
C_A = np.array([1.00, 0.78, 0.61, 0.48, 0.37, 0.29, 0.23])  # mol/L

r_A = -np.gradient(C_A, t)
```

**Key points to stress:**

1. **Argument order matters:** `np.gradient(y, x)` — the thing you're differentiating comes first, the independent variable second. A common mistake is writing `np.gradient(t, C_A)` — that gives the inverse.

2. **How does `np.gradient` work?** It uses:
   - *Central differences* at interior points: $\dfrac{f(x+h) - f(x-h)}{2h}$
   - *One-sided differences* at the endpoints (higher-order, not just first-order)
   - This handles **non-uniform spacing** automatically — you don't need equally-spaced data.

3. **The negative sign:** The reaction rate of disappearance is positive, but $dC_A/dt$ is negative (concentration is decreasing). So we negate it.

**Show the printed table and plots.** Point out that the rate decreases over time as concentration falls — this is physically expected for any positive-order reaction.

---

### Example 2: Heat Required from Tabulated Cp Data (`trapezoid`)

**Setup:**

> "Now integration. A common thermodynamics calculation: how much heat does it take to raise a gas from one temperature to another? You need to integrate Cp over the temperature range. If you have tabulated data, you use the trapezoidal rule."

$$\Delta H = \int_{400}^{1000} C_p(T) \, dT$$

**Walk through the code:**

```python
T_data  = np.array([400,  500,  600,  700,  800,  900,  1000])  # K
Cp_data = np.array([34.3, 35.2, 36.3, 37.5, 38.6, 39.7,  40.8])  # J/mol·K

delta_H = trapezoid(Cp_data, T_data)   # J/mol
```

**Key points to stress:**

1. **Always pass `x` as the second argument.** If you write `trapezoid(Cp_data)` without `T_data`, it assumes unit spacing (i.e., spacing of 1) and gives you a number 100× too small. Your x-axis has units — tell the function what they are.

2. **What is the trapezoidal rule?** It approximates the area under the curve by connecting adjacent data points with straight lines and summing the resulting trapezoids. It's exact for linear functions; the error grows when the function curves significantly between data points.

3. **Show the shaded area plot.** The filled region is exactly what `trapezoid` is computing.

> "Notice Cp increases roughly linearly with temperature for steam in this range — so the trapezoidal rule is very accurate here."

---

## Part 2: Warm-Up Exercises (~10 min)

> "Before the main problems, let's make sure everyone is comfortable with the syntax. Three quick exercises — work independently, I'll give you about 3 minutes per exercise, then we'll go over them together."

### Exercise 1 — `np.gradient`: velocity from position data

**Circulate and check:** The blanks are `np.gradient(x, t)`. Watch for students who write `np.gradient(t, x)` — that's the most common error.

**When reviewing:**
> "At `t=0`, np.gradient used a forward difference formula. At `t=3.0`, it used a backward difference. At all interior points, it used central differences. The order of arguments determines what derivative you're computing — y with respect to x."

Expected output: velocity starts fast (~2.8 m/s), slows to zero by the end as the object decelerates to rest.

### Exercise 2 — `trapezoid`: total volume from flow data

**Key teaching moment:** the time points are *not* equally spaced (0, 2, 5, 8, 10, 13, 15 min). This is intentional. Emphasize that `trapezoid(Q, t_pump)` handles this correctly without any extra work from the user.

**When reviewing:**
> "This is why we always pass the x-array. The gaps between measurements are different sizes — 2 min, then 3 min, then 3 min, then 2 min, etc. `trapezoid` accounts for each interval's actual width."

### Exercise 3 — `quad`: Shomate equation for N₂

**Key teaching point:** `quad` integrates a Python *function*, not an array. You define a function `Cp_N2(T)` and pass it to `quad`.

```python
dH, err = quad(Cp_N2, 300, 1200)
```

> "`quad` also returns an error estimate — this is the estimated absolute error in the result. It's almost always tiny (like 1e-9), but it's good practice to check it, especially near singularities."

**The Shomate formula:** $C_p = A + B\tau + C\tau^2 + D\tau^3 + E/\tau^2$ where $\tau = T/1000$.

---

## Part 3: Practice Problems (~30 min)

### Problem 1: Batch Reactor — Rate from Concentration Data

> "This is a more realistic version of the demo. The reaction order is unknown — you'll figure it out from the data."

**Part (a) — `np.gradient`:**

One-liner solution: `r_A = -np.gradient(C_A, t_rxn)`

If students are confused about the negative sign, ask: *"Is the concentration increasing or decreasing? So what sign is dC_A/dt? And we want the rate of reaction, which is positive..."*

**Part (b) — manual finite differences (no `np.gradient`):**

This part builds understanding of what `np.gradient` is doing under the hood.

```python
dCdt = np.zeros(len(t_rxn))

# Forward difference: first point only
dCdt[0] = (C_A[1] - C_A[0]) / (t_rxn[1] - t_rxn[0])

# Central differences: all interior points at once (vectorized)
dCdt[1:-1] = (C_A[2:] - C_A[:-2]) / (t_rxn[2:] - t_rxn[:-2])

# Backward difference: last point only
dCdt[-1] = (C_A[-1] - C_A[-2]) / (t_rxn[-1] - t_rxn[-2])

r_A_manual = -dCdt
```

**Highlight the slicing:** `C_A[2:]` skips the first two elements; `C_A[:-2]` skips the last two. Their difference, divided by the spacing, gives the central difference at every interior point simultaneously — no loop needed.

> "The endpoint values from (a) and (b) differ slightly. `np.gradient` uses a higher-order (more accurate) one-sided formula at the boundaries, while we're using simple first-order formulas here."

**Part (c) — log-log plot for reaction order:**

```python
coeffs = np.polyfit(np.log(C_A), np.log(r_A), 1)
n = coeffs[0]   # slope = reaction order
```

**Explain the math:** If $r_A = k C_A^n$, then $\ln(r_A) = \ln(k) + n \ln(C_A)$. A plot of $\ln(r_A)$ vs. $\ln(C_A)$ is a straight line with slope $n$.

> "From the data, the slope should come out close to 1.5 — so this appears to be a 3/2-order reaction. In practice you'd round to 1 or 2 unless there's strong reason to believe a fractional order."

---

### Problem 2: PFR Design — Levenspiel Integration

> "This is the classic Levenspiel plot from kinetics. The PFR design equation is an integral — and quad is perfect for this."

**The design equation:**
$$V = F_{A0} \int_0^{X_f} \frac{dX}{-r_A(X)}$$

**Part (a) — `quad` integration:**

```python
def levenspiel_1st(X):
    return 1.0 / (k * C_A0 * (1 - X))

integral, err = quad(levenspiel_1st, 0, X_f)
V_quad = F_A0 * integral
```

**Compare to analytical result:**
$$V_\text{exact} = \frac{F_{A0}}{k C_{A0}} \ln\!\left(\frac{1}{1-X_f}\right)$$

> "The relative error from `quad` should be essentially machine precision — around 1e-14. This is the power of adaptive Gaussian quadrature: it chooses its own evaluation points to minimize error."

**Connect to the Levenspiel plot:** The area under the $1/(-r_A)$ vs. $X$ curve equals $V/F_{A0}$. This is a visual and physical interpretation of the integral.

> "Notice the curve shoots up near X=1 — the rate approaches zero as conversion approaches 100%, so you need infinite volume to achieve complete conversion. This is why we design for 90%, not 100%."

---

## Wrap-Up (~3 min)

**Key takeaways — write on board:**

| Situation | Use |
|-----------|-----|
| Derivative from data points | `np.gradient(y, x)` |
| Integral from data points | `trapezoid(y, x)` |
| Integral of a function | `quad(f, a, b)` |

**Common mistakes to remember:**
- `np.gradient(y, x)` not `np.gradient(x, y)`
- `trapezoid(y, x)` not `trapezoid(y)` — always include the x-array
- `quad` returns `(result, error)` — unpack both

**Looking ahead:**
> "Next chapter we move into solving differential equations numerically. Everything you did today with derivatives and integrals feeds directly into that — the finite difference formulas from Problem 1(b) are the building blocks of numerical ODE solvers."
