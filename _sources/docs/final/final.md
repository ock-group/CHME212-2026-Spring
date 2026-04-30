# CHME 212 – Final Examination

**Name:** _______________________________ **Andrew ID:** _________________

**Date:** ________________________________ **Section:** ___________________

- This is a **closed-book, closed-notes** exam. No electronic devices.
- Write all answers in the space provided. Show all work for full credit.
- Total: **110 points** | Time allowed: **2 hours**

---

## Part I: Multiple Choice (2 pts each, 50 pts total)

*Circle the single best answer.*

---

**1.** What is the output of the following Python expression?

```
round(2.5)
```

(A) 2.5
(B) 3
(C) 2
(D) 2.0

---

**2.** Which of the following correctly accesses the **last** element of a list `data`?

(A) `data[last]`
(B) `data[-1]`
(C) `data[len(data)]`
(D) `data.last()`

---

**3.** A Python dictionary is best described as:

(A) An ordered collection of items accessible by integer index
(B) An immutable sequence of elements
(C) A collection of key-value pairs
(D) A set of unique, unordered values

---

**4.** Which f-string correctly formats the number `3.14159` to **two decimal places**?

(A) `f"{x:.2}"`
(B) `f"{x:2f}"`
(C) `f"{x:.2f}"`
(D) `f"{x:0.2}"`


---


**8.** Given the 2-D NumPy array below, what does `A[1, 2]` return?

```
A = np.array([[10, 20, 30],
              [40, 50, 60],
              [70, 80, 90]])
```

(A) 20
(B) 70
(C) 60
(D) 50


---

**11.** Which goodness-of-fit metric is expressed as a **dimensionless** number between 0 and 1, where values closer to 1 indicate a better fit?

(A) Mean Absolute Error (MAE)
(B) Mean Squared Error (MSE)
(C) Root Mean Squared Error (RMSE)
(D) Coefficient of Determination ($R^2$)


---

### "Fix the Code" Questions (Questions 21–25)

*Each snippet below contains a single conceptual bug. Identify what is wrong and choose the corrected version.*

---

**21.** A student implements bisection to find the root of `f` on `[a, b]`. What is wrong with this code?

```python
def bisection(f, a, b, tol=1e-6):
    while abs(b - a) > tol:
        m = (a + b) / 2
        if f(m) > 0:      # <-- bug here?
            b = m
        else:
            a = m
    return m
```

(A) The midpoint formula `(a + b) / 2` can cause floating-point overflow for large `a` and `b`
(B) The update rule does not use the **sign** of `f(a)·f(m)` — it assumes the root is always where `f < 0`, which breaks if the function approaches from the positive side
(C) The loop should check `abs(f(m)) > tol`, not the bracket width
(D) `m` should be updated as `m = a + (b - a) / 2` to avoid overflow

---


**23.** A student uses `fsolve` to find the conversion $X$ where the equilibrium condition $K(X) - K_{eq} = 0$ is satisfied, then immediately reports the answer.

```python
from scipy.optimize import fsolve

def equation(X):
    return K_eq - 2*X / (1 - X)**2

solution = fsolve(equation, x0=0.5)
print(f"Equilibrium conversion: {solution[0]:.4f}")
```

What critical step is missing?

(A) `fsolve` requires an initial guess array, not a scalar
(B) The function should return a list, not a scalar
(C) The student never checks whether the returned `solution` actually satisfies `equation(solution) ≈ 0` — `fsolve` can silently return a non-solution if it fails to converge
(D) `fsolve` only works for polynomial equations; `brentq` should be used instead

---

**24.** The following code solves the ODE $\frac{dC}{dt} = -k C$ using `solve_ivp`, but it produces incorrect results. What is the bug?

```python
from scipy.integrate import solve_ivp

k = 0.3

def rate(t, C):
    return -k * C          # returns a float

sol = solve_ivp(rate, [0, 10], [1.0], t_eval=np.linspace(0, 10, 100))
C = sol.y[0]
```

(A) The initial condition should be `y0=1.0` (a scalar), not `[1.0]` (a list)
(B) `solve_ivp` requires the function to accept `(y, t)` — the argument order is reversed from `(t, y)`
(C) The derivative function must return a **list** (e.g., `[-k * C]`), not a bare float; otherwise `solve_ivp` cannot handle it as a system
(D) `t_eval` must be the same length as `y0`

---

**25.** A student writes a Newton's method loop to find the root of $f(x) = x^2 - 4$ starting from $x_0 = 3$.

```python
x = 3.0
for i in range(10):
    x = x - f(x) / df(x)
    if abs(x) < 1e-6:       # <-- convergence check
        break
print(x)
```

The loop terminates early and prints a wrong answer. What is the conceptual mistake in the convergence check?

(A) The loop should run `while True` instead of `for i in range(10)`
(B) The convergence check tests `|x|` (the magnitude of the **iterate**) instead of `|f(x)|` or `|x_new − x_old|` (the size of the **update or residual**) — near the root $x = 2$, $|x|$ is never small
(C) `abs()` should be replaced with `np.abs()` for floating-point safety
(D) The update formula should be `x = x + f(x) / df(x)` (addition, not subtraction)

---

## Part II: Short Answer / Concept Problems (60 pts)

*Show all work and write answers clearly. Partial credit is awarded.*

---

### Problem 1 — Newton's Method by Hand (12 pts)

You want to find the root of $f(x) = x^3 - 2x - 5$.

**(a)** Compute $f'(x)$.

\vspace{2cm}

**(b)** Starting from the initial guess $x_0 = 2$, perform **two iterations** of Newton's method. Carry four significant figures throughout.

| Iteration $n$ | $x_n$ | $f(x_n)$ | $f'(x_n)$ | $x_{n+1}$ |
|:---:|:---:|:---:|:---:|:---:|
| 0 | 2.000 | | | |
| 1 | | | | |
| 2 | | | | — |

\vspace{2cm}

**(c)** After your two iterations, what is the approximate root? How would you verify that this is correct?

\vspace{2cm}

**(d)** Newton's method fails when $f'(x_n) \approx 0$. Describe geometrically what happens to the next iterate $x_{n+1}$ in that situation.

\vspace{2cm}

---

### Problem 2 — Bisection Method (8 pts)

Consider $g(x) = e^x - 3x$.

**(a)** Show by evaluation that $g(x)$ has a root in $[0, 2]$. (Compute $g(0)$ and $g(2)$ and apply the bracketing condition.)

\vspace{2cm}

**(b)** Perform **two iterations** of bisection on $[0, 2]$.

| Iteration | $a$ | $b$ | $m = (a+b)/2$ | $g(m)$ | New interval |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 0 | 2 | | | |
| 2 | | | | | |

\vspace{2cm}

**(c)** After $n$ bisection steps starting from an interval of width $L$, what is the width of the remaining bracket? How many steps are needed to reduce the width of $[0, 2]$ below $10^{-4}$?

\vspace{3cm}

---

### Problem 3 — Linear Algebra: Mass Balances (10 pts)

A three-stream mixing junction combines streams 1, 2, and 3 to produce an outlet stream. You are given the following species mass balances for components A and B, plus the overall balance:

$$F_1 + F_2 + F_3 = 100 \text{ kg/h}$$
$$0.30\,F_1 + 0.50\,F_2 + 0.20\,F_3 = 35 \text{ kg/h} \quad \text{(species A)}$$
$$0.10\,F_1 + 0.20\,F_2 + 0.40\,F_3 = 24 \text{ kg/h} \quad \text{(species B)}$$

**(a)** Write this system in matrix form $A\mathbf{x} = \mathbf{b}$. Identify $A$, $\mathbf{x}$, and $\mathbf{b}$ explicitly.

\vspace{4cm}

**(b)** A classmate tries to solve this by computing `np.linalg.inv(A) @ b`. This gives the correct numerical answer, but why is `np.linalg.solve(A, b)` generally preferred?

\vspace{3cm}

**(c)** After obtaining the solution vector $\mathbf{x} = [F_1, F_2, F_3]^T$, how would you verify the solution is correct without re-solving?

\vspace{2cm}

---

### Problem 4 — Numerical Integration: PFR Sizing (10 pts)

A plug flow reactor (PFR) is designed using the following design equation:

$$V = F_{A0} \int_0^{X_f} \frac{dX}{-r_A(X)}$$

The reaction rate is $-r_A = k C_{A0}^2 (1-X)^2$ with $k = 0.5 \text{ L/(mol·min)}$, $C_{A0} = 2 \text{ mol/L}$, $F_{A0} = 4 \text{ mol/min}$, and the target conversion is $X_f = 0.8$.

**(a)** Write out the integrand $\frac{1}{-r_A(X)}$ as a function of $X$ (substitute and simplify).

\vspace{2.5cm}

**(b)** Using the **trapezoidal rule** with two subintervals (i.e., $X = 0, 0.4, 0.8$), approximate $\int_0^{0.8} \frac{dX}{-r_A(X)}$.

\vspace{4cm}

**(c)** Compute the reactor volume $V$ using your result from (b).

\vspace{2cm}

**(d)** Would using more subintervals increase or decrease the error of the trapezoidal approximation, and why?

\vspace{2cm}

---

### Problem 5 — Ordinary Differential Equations: Euler's Method (10 pts)

A batch reactor follows first-order kinetics:

$$\frac{dC_A}{dt} = -k\,C_A, \quad C_A(0) = 1.0 \text{ mol/L}, \quad k = 0.5 \text{ min}^{-1}$$

**(a)** Write the exact (analytical) solution $C_A(t)$.

\vspace{2cm}

**(b)** Apply **Euler's method** with step size $h = 1$ min to estimate $C_A$ at $t = 1, 2, 3$ min. Fill in the table. Keep three decimal places.

| $n$ | $t_n$ (min) | $C_A^{(n)}$ (mol/L) | $f(t_n, C_A^{(n)}) = -k\,C_A^{(n)}$ | $C_A^{(n+1)}$ |
|:---:|:---:|:---:|:---:|:---:|
| 0 | 0 | 1.000 | | |
| 1 | 1 | | | |
| 2 | 2 | | | |
| 3 | 3 | — | — | — |

\vspace{1cm}

**(c)** Using the exact solution, compute the true values $C_A(1)$, $C_A(2)$, $C_A(3)$. What is the **absolute error** at $t = 3$ min?

\vspace{3cm}

**(d)** If you halved the step size to $h = 0.5$ min, how would you expect the global error at $t = 3$ min to change? Justify using the order of accuracy of Euler's method.

\vspace{2cm}

---

### Problem 6 — Monte Carlo & Statistics (10 pts)

A chemical process produces a product with purity governed by:

$$\text{Purity} = 100\% \times \left(1 - 0.2\,e^{-k(\tau)}\right), \quad k = 0.03 \text{ min}^{-1}$$

The residence time $\tau$ is normally distributed: $\tau \sim \mathcal{N}(\mu = 120 \text{ min},\; \sigma = 10 \text{ min})$.

The specification requires **Purity $\geq$ 97.5%**.

**(a)** Compute the **nominal purity** (evaluated at the mean residence time $\tau = 120$ min). Does the nominal operating point meet the spec?

\vspace{3cm}

**(b)** Explain, in 2–3 sentences, why the **fraction of batches failing the spec** could still be significant even though the nominal purity passes. What Monte Carlo simulation allows you to estimate that the nominal calculation cannot.

\vspace{3.5cm}

**(c)** In a Monte Carlo simulation with $N = 10{,}000$ samples, 640 batches fail the spec. Estimate the **failure probability** and compute a rough **95% confidence interval** on that estimate using the normal approximation:

$$\hat{p} \pm 1.96\sqrt{\frac{\hat{p}(1-\hat{p})}{N}}$$

\vspace{3cm}

**(d)** How many samples would you need to **halve** the width of the confidence interval found in (c)? Explain your reasoning.

\vspace{2cm}

---

*End of Examination. Check your work.*
