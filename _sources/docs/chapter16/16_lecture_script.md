# Chapter 16 Lecture Script: Root Finding & Newton's Method

---

## Opening

Today's topic is **root finding**.

Here is the problem we keep running into. You know the van der Waals equation:

$$\left(P + \frac{a}{V^2}\right)(V - b) = RT$$

You know $T$, $P$, $a$, $b$ — you want $V$. But $V$ appears three times and the equation expands into a cubic. There is no simple rearrangement. Same story with the Antoine equation, or a reaction equilibrium expression with complex stoichiometry. The unknown is trapped inside the equation in a way you cannot untangle by hand easily.

The strategy in all these cases: **rewrite as $f(x) = 0$ and find the root numerically.**

---

## 16.1 — What Is a Root?

A **root** (or zero) of $f(x)$ is any $x^*$ such that $f(x^*) = 0$.

Almost any equation fits this form. If you have $e^x = 2x + 1$, subtract the right side: $f(x) = e^x - 2x - 1 = 0$. If you have the van der Waals equation, move everything to one side: $f(V) = (P + a/V^2)(V - b) - RT = 0$.

> *Point to the table in the notebook.* Every equation in the table gets rewritten the same way — subtract one side from the other, call it $f$, find where it is zero.

Geometrically, a root is where the curve $f(x)$ **crosses the x-axis**.

> *Show the plot of $f(x) = e^x - 2x - 1$.* This function has two roots: one at $x^* = 0$ (you can verify by hand: $e^0 - 0 - 1 = 0$) and one near $x^* \approx 1.256$ which you cannot get analytically.

**The most important habit:** always plot $f(x)$ before running any solver. The plot tells you how many roots exist, roughly where they are, and whether your equation is well-behaved.

---

## 16.2 — `np.roots`: Polynomial Roots

If your equation happens to be a polynomial, NumPy gives you all roots at once with a single call.

**Syntax:** pass the coefficients from highest to lowest degree.

For example, the van der Waals equation expands to a cubic in $V$:
$$PV^3 - (Pb + RT)V^2 + aV - ab = 0$$

The four coefficients are $[P,\ -(Pb+RT),\ a,\ -ab]$.

> *Run cell 6.* `np.roots` returns three roots. Two of them are complex — those are not physical. The one real root greater than $b$ is the molar volume we want.

Notice what we did not have to do: no initial guess, no bracket, no iteration. This is a direct algebraic method.

### How does `np.roots` actually work?

> *Show cells 7 and 8.*

`np.roots` does not iterate. It uses linear algebra. For a polynomial of degree $n$, NumPy builds an $n \times n$ matrix called the **companion matrix**, then computes its eigenvalues. The eigenvalues are the roots.

Let me show you why this works, starting with the simplest case.

**Degree-2 example:** Take $p(x) = x^2 + a_1 x + a_2$. The companion matrix is:

$$C = \begin{bmatrix} -a_1 & -a_2 \\ 1 & 0 \end{bmatrix}$$

Now compute the characteristic polynomial of $C$ — this is the polynomial whose roots are the eigenvalues of $C$. You set $\det(C - \lambda I) = 0$:

$$\det \begin{bmatrix} -a_1 - \lambda & -a_2 \\ 1 & -\lambda \end{bmatrix} = (-a_1 - \lambda)(-\lambda) - (-a_2)(1) = \lambda^2 + a_1\lambda + a_2$$

That is exactly $p(\lambda)$. The characteristic polynomial of $C$ is identical to the original polynomial $p$. So $\det(C - \lambda I) = 0$ is the same equation as $p(\lambda) = 0$ — **the eigenvalues of $C$ are the roots of $p$.**

> *Run the 2×2 demo in cell 8.* For $p(x) = x^2 - 5x + 6$, the companion matrix is:
>
> $$C = \begin{bmatrix} 5 & -6 \\ 1 & 0 \end{bmatrix}$$
>
> Computing $\det(C - \lambda I) = \lambda^2 - 5\lambda + 6$. Eigenvalues: 2 and 3. That matches `np.roots([1, -5, 6])` exactly.

The same construction works for degree $n$. The first row of the $n \times n$ companion matrix holds the negated coefficients $[-a_1, -a_2, \ldots, -a_n]$, and there is a sub-diagonal of 1s below. NumPy then calls `np.linalg.eigvals` on that matrix.

> *Run the 3×3 demo in cell 8.* For $x^3 - 6x^2 + 11x - 6$, eigenvalues give exactly $[1, 2, 3]$.

**Key point:** `np.roots` is exact (up to floating point), fast, requires no initial guess, and finds all roots including complex ones simultaneously. The only limitation is that your equation must be a polynomial.

---

## 16.3 — Bisection Method

Now what if your equation is not a polynomial? We need an iterative method.

The bisection method is the simplest root finder. It is grounded in a theorem from calculus you have already seen: the **Intermediate Value Theorem (IVT)**.

> If $f$ is continuous on $[a, b]$ and $f(a) \cdot f(b) < 0$, then there exists at least one root $x^* \in (a, b)$.

The sign change $f(a) \cdot f(b) < 0$ is not just a programming check — it is a mathematical guarantee. If the function is positive at one end and negative at the other, it had to cross zero somewhere in between.

**The algorithm:**
1. Start with a bracket $[a, b]$ where $f(a) \cdot f(b) < 0$
2. Compute the midpoint $m = (a + b) / 2$
3. Check which half contains the root:
   - If $f(a) \cdot f(m) < 0$: root is in the left half → set $b = m$
   - Otherwise: root is in the right half → set $a = m$
4. Repeat until $(b - a)/2 < \epsilon$

> *Walk through the 6-panel visualization in cell 11, one panel at a time.*

**Panel 1:** We start with $a = 1.0$, $b = 2.0$. The shaded red region is the bracket — we know the root is somewhere in there. The midpoint is $m = (1.0 + 2.0)/2 = 1.5$. We compute $f(a) = f(1.0) \approx 0.718$ and $f(m) = f(1.5) \approx -0.518$. Do they have the same sign? No — $f(a)$ is positive and $f(m)$ is negative. That means the root is in the left half $[1.0, 1.5]$. So we set $b \leftarrow 1.5$. The bracket width was 1.0 and is now 0.5.

**Panel 2:** Now $a = 1.0$, $b = 1.5$. Midpoint $m = 1.25$. Compute $f(1.0) \approx 0.718$ and $f(1.25) \approx 0.117$. Both positive — same sign as $f(a)$. Root is in the right half $[1.25, 1.5]$. Set $a \leftarrow 1.25$. Width is now 0.25.

> *Continue through remaining panels.* Notice the red shaded region getting narrower each time, and the midpoint converging toward the root. The annotation box in each panel shows the exact arithmetic: what $m$ is, what $f(a)$ and $f(m)$ are, and which endpoint gets updated.

**Panel 3:** $a = 1.25$, $b = 1.5$, $m = 1.375$. Check signs. $f(1.25) > 0$, $f(1.375) < 0$ — different signs, set $b \leftarrow 1.375$.

**Panel 4–6:** The bracket keeps halving. By step 6, the bracket width is $1.0/2^6 = 0.015625$ and the midpoint is already very close to the true root.

### Convergence plot

> *Run cell 12 and look at the convergence plot.*

Let's take a look at how it’s implemented in Python.

First, let’s look at the function definition:

def bisection(f, a, b, tol=1e-8, max_iter=100)

This function finds a root of f(x) = 0 within the interval [a, b].
Here, f is the function, a and b define the interval, tol is the tolerance for accuracy, and max_iter is the maximum number of iterations.

Next, we have an important assumption:

assert f(a) * f(b) < 0

This means that f(a) and f(b) must have opposite signs. In other words, the function crosses zero somewhere between a and b. This guarantees that a root exists in the interval.

We also initialize a list:

history = []

This will store all midpoint values so we can track how the solution converges.

Now we enter the main loop:

for i in range(max_iter):

At each iteration, we compute the midpoint:

m = (a + b) / 2.0

This is why the method is called “bisection” — we keep cutting the interval in half.

We store this midpoint:

history.append(m)

Then we check if we should stop:

if abs(f(m)) < tol or (b - a) / 2 < tol:

We stop if either the function value at m is close enough to zero, or the interval is sufficiently small.

If not, we decide which half of the interval to keep:

if f(a) * f(m) < 0:
    b = m
else:
    a = m

If the sign change is between a and m, we keep the left half.
Otherwise, we keep the right half.

We repeat this process, continuously shrinking the interval that contains the root.

Finally, we return:

return m, history

Here, m is our estimated root, and history contains all intermediate guesses.

Overall, the bisection method is simple, robust, and guaranteed to converge, as long as the initial interval satisfies the sign condition.



After $n$ iterations, the bracket width is $(b - a)/2^n$ and the error is bounded by:
$$|x_n - x^*| \leq \frac{b - a}{2^n}$$

This is **linear convergence** — on a log scale, the error decreases as a straight line, halving exactly every iteration. To reach tolerance $\epsilon = 10^{-8}$ from an interval of width 1, you need $\log_2(10^8) \approx 27$ iterations. Slow, but completely predictable. You can calculate in advance exactly how many iterations you need.

**Strengths of bisection:**
- Always works if the initial bracket is valid
- No derivative needed
- Error bound is known in advance

**Weakness:** Slow. Linear convergence means 27 iterations for 8 decimal places.

---

## 16.4 — Newton's Method

Bisection is safe but slow because it only uses the sign of $f$ — it ignores the actual values. Newton's method uses the derivative to make a much smarter prediction of where the root is.

### The derivation

Start from the current guess $x_n$. Write the first-order Taylor expansion of $f$ around $x_n$:

$$f(x) \approx f(x_n) + f'(x_n)(x - x_n)$$

We want to find $x$ where $f(x) = 0$. Substitute $f(x) = 0$ and solve for $x$:

$$0 = f(x_n) + f'(x_n)(x_{n+1} - x_n)$$

$$\boxed{x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}}$$

**Geometric interpretation:** draw the tangent line to $f$ at the point $(x_n, f(x_n))$. The tangent line is $y = f(x_n) + f'(x_n)(x - x_n)$. Set $y = 0$ and solve — you get exactly the formula above. So $x_{n+1}$ is the x-intercept of the tangent line at $x_n$.

### Step-by-step walkthrough

> *Walk through the 6-panel visualization in cell 15, one panel at a time.*

We are solving $f(x) = e^x - 2x - 1 = 0$, starting from $x_0 = 2.0$. The derivative is $f'(x) = e^x - 2$.

**Panel 1 (Step 1):** Current guess $x_0 = 2.0$. The orange dot sits on the curve at $(2.0,\ f(2.0))$. We evaluate:
- $f(x_0) = e^2 - 4 - 1 \approx 2.389$
- $f'(x_0) = e^2 - 2 \approx 5.389$

The dashed orange line is the tangent at this point. We follow it down to where it crosses zero:
$$x_1 = 2.0 - \frac{2.389}{5.389} \approx 1.557$$

The triangle on the x-axis marks $x_1$. Error dropped from 0.644 to 0.113.

**Panel 2 (Step 2):** Now at $x_1 = 1.557$. Evaluate $f$ and $f'$ there, draw the new tangent line, follow it to zero:
$$x_2 = 1.557 - \frac{f(1.557)}{f'(1.557)} \approx 1.295$$

Error is now 0.019. Already much better — and we have only done 2 steps.

**Panel 3 (Step 3):** At $x_2 = 1.295$. The tangent line is much flatter now (we're closer to the root), and the next step jumps very close. Error drops to $\approx 5 \times 10^{-4}$.

> *Note to students: watch how the error column in the annotation box shrinks — not by half each time like bisection, but much faster. Step 3 error is already 100× smaller than step 2.*

**Panels 4–6:** By step 4, the error is already $10^{-7}$. By step 5, $10^{-13}$. By step 6, we are at machine precision.

### Stopping criteria

> *Point to cell 14 markdown.*

We stop when either:
- $|f(x_n)| < \epsilon$ — the residual (function value) is small enough
- $|x_{n+1} - x_n| < \epsilon$ — the step size is negligible

In the code, `tol=1e-12` means we stop when $|f(x)| < 10^{-12}$.

### Convergence: quadratic vs. linear

> *Run cell 16 to see the convergence table.*



First, we define the function and its derivative:

def f(x):   return np.exp(x) - 2*x - 1
def df(x):  return np.exp(x) - 2

Here, f(x) is the function whose root we want to find, and df(x) is its derivative.
Notice that the derivative of e^x is still e^x, so f'(x) = e^x - 2.

Next, we choose an initial guess:

x0 = 2.0

Newton’s method requires a starting point, and the quality of this guess can affect convergence.

We then create a list to store all iterates:

iterates = [x0]

This allows us to track how the solution evolves over each step.

Now we perform the iterations:

for _ in range(4):

We will run exactly 4 iterations of Newton’s method.

Inside the loop, we take the most recent estimate:

xn = iterates[-1]

Then we apply the Newton update formula:

xn - f(xn) / df(xn)

This comes from the idea of approximating the function with a tangent line and finding where that tangent crosses zero.

We append the new estimate to our list:

iterates.append(xn - f(xn) / df(xn))

Each iteration should move us closer to the true root.

After the loop, the list “iterates” contains the full sequence:
x0, x1, x2, x3, x4

This allows us to observe how quickly Newton’s method converges.

The key idea is that Newton’s method uses both the function value and its derivative to rapidly refine the estimate of the root.


The error $e_n = |x_n - x^*|$ satisfies:

$$e_{n+1} \approx \frac{|f''(x^*)|}{2|f'(x^*)|} \cdot e_n^2$$

The error is **squared** at each step — this is **quadratic convergence**. If you have 1 correct decimal place, the next step gives 2, then 4, then 8, then 16.

Look at the last column of the table: $e_{n+1}/e_n^2$. That ratio stays roughly constant at about 0.27. The constant value is the proof — if convergence were quadratic, this ratio should be constant, and it is.

Contrast with bisection:

| | Newton | Bisection |
|---|---|---|
| Error at step $n$ | $\sim C \cdot e_{n-1}^2$ | $\sim (b-a)/2^n$ |
| Iterations for $10^{-16}$ | ~5 | ~53 |
| Needs derivative | Yes | No |
| Needs bracket | No | Yes |

> *Run cell 18 for the convergence comparison plot.* On a log scale, bisection is a straight line (linear). Newton's curve bends sharply downward — the slope doubles each iteration. That visual difference is the definition of quadratic vs. linear convergence.

### When Newton's method fails

- **$f'(x_n) \approx 0$:** the tangent line is nearly horizontal and the next guess $x_{n+1} = x_n - f/f'$ shoots to infinity. Division by near-zero.
- **Poor initial guess:** if $x_0$ is far from the root, the tangent at $x_0$ may point toward a completely different region of the function.
- **Multiple roots close together:** the method can oscillate between them.

**Rule of thumb:** always plot first, pick $x_0$ close to the root you want, and verify $f'(x_0) \neq 0$.

---

## 16.5 — `brentq`: The Practical Choice

In practice, you will rarely hand-code bisection or Newton. For a single equation, `scipy.optimize.brentq` is the standard tool.

Brent's method (1973) is a hybrid that switches between three techniques at each step:

1. **Bisection** — always makes progress, but slow
2. **Secant method** — Newton-like but approximates the derivative using two recent points:
   $$x_{n+1} = x_n - f(x_n) \cdot \frac{x_n - x_{n-1}}{f(x_n) - f(x_{n-1})}$$
   Superlinear convergence (~order 1.6), but no bracket guarantee.
3. **Inverse quadratic interpolation** — fits a quadratic through three recent $(f, x)$ pairs and uses its inverse to predict the root. Even faster when $f$ is smooth.

Brent's method tries the fast methods first. If either would step outside the current bracket, it falls back to bisection. You get **near-Newton speed** with **bisection's guarantee**.

```python
from scipy.optimize import brentq

root = brentq(f, a, b)               # basic usage
root = brentq(f, a, b, xtol=1e-12)   # tighter tolerance
root = brentq(f, a, b, args=(p1,))   # pass extra parameters to f
```

**Requirements:** $f(a)$ and $f(b)$ must have opposite signs. If not, you get a `ValueError`. Always plot first.

### Application: van der Waals molar volume

> *Run cells 21–22.*

We define $f(V) = (P + a/V^2)(V - b) - RT$ and plot it. The plot shows exactly one sign change in a physically meaningful range. We read off the bracket and call `brentq`. The result gives the molar volume with residual $\approx 10^{-12}$ — essentially exact.

We can also compare to the ideal gas: $V_{\text{ideal}} = RT/P$. The van der Waals result is smaller because intermolecular attractions effectively compress the gas.

---

## 16.6 — `fsolve`: Systems of Equations

`brentq` handles one equation, one unknown. For $N$ equations and $N$ unknowns, use `fsolve`.

`fsolve` generalizes Newton's method to multiple dimensions. Recall the scalar update:
$$x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$$

In $N$ dimensions, the scalar derivative $f'(x)$ becomes the **Jacobian matrix** $J$:
$$J_{ij} = \frac{\partial F_i}{\partial x_j}$$

The update becomes:
$$\mathbf{x}_{n+1} = \mathbf{x}_n - J^{-1}\mathbf{F}(\mathbf{x}_n)$$

`fsolve` estimates $J$ numerically using finite differences — you do not need to derive it by hand.

```python
from scipy.optimize import fsolve

def F(vars):
    x, y = vars
    return [eq1, eq2]   # one residual per equation

solution = fsolve(F, x0=[x_guess, y_guess])
print(F(solution))       # always check: must be near [0, 0]
```

**Critical warning:** Unlike `brentq`, `fsolve` has no bracket guarantee. It can converge silently to the wrong answer. **Always print the residuals** after solving.

> *Run cell 25.* We find the intersections of $y = x^2$ and $y = 2 - x$ by solving the system simultaneously. Using two different initial guesses finds both intersection points. Check `equations(solution)` each time — it should return `[0, 0]`.

---

## 16.7 — Application: Reaction Equilibrium

> *Run cells 27–30.*

For the gas-phase reaction A ⇌ 2B, starting from pure A, the equilibrium condition is:

$$f(X_e) = \frac{(2X_e/(1+X_e))^2}{(1-X_e)/(1+X_e)} \cdot P - K_{eq} = 0$$

**Step 1 (always):** plot $K_p(X)$ alongside the horizontal line $K_{eq} = 1.5$. The intersection is the equilibrium conversion. The plot confirms there is exactly one root in $(0, 1)$.

**Step 2:** call `brentq` on $[0.01, 0.99]$. Result: $X_e \approx 0.44$, or about 44% conversion.

**Verify:** print `f_equil(X_eq)` — it should be $\approx 10^{-15}$.

**Le Chatelier check:** loop over pressures from 0.5 to 10 atm, solving `brentq` each time. The plot shows $X_e$ decreasing monotonically with pressure. Increasing pressure shifts A ⇌ 2B toward fewer moles (toward A), exactly as Le Chatelier's principle predicts — and now we have the quantitative curve.

---

## Summary

| Method | Tool | Speed | Requires | Use when |
|--------|------|-------|----------|----------|
| Polynomial roots | `np.roots` | Exact (eigenvalue) | Coefficients | Equation is a polynomial |
| Bisection | hand-coded | Linear ($2^n$) | Bracket $[a,b]$ | Learning / guaranteed fallback |
| Newton | hand-coded | Quadratic | $f'(x)$ + good $x_0$ | You can compute the derivative |
| Brent's method | `brentq` | Superlinear | Bracket $[a,b]$ | **Single equation, single unknown** |
| Newton (N-D) | `fsolve` | Quadratic | Initial guess | **$N$ equations, $N$ unknowns** |

**The workflow every time:**
1. **Rewrite** as $f(x) = 0$
2. **Plot** $f(x)$ — count roots, find brackets or good initial guesses
3. **Choose your tool** — polynomial → `np.roots`; single eq → `brentq`; system → `fsolve`
4. **Verify** — always check that $f(x^*) \approx 0$

**Common pitfalls:**
- `np.roots` returns complex roots — filter for real, physically meaningful values
- `brentq` needs a sign change — if $f(a)$ and $f(b)$ have the same sign you get a `ValueError`
- `fsolve` can silently give the wrong answer — always print residuals
- Newton diverges when $f'(x_n) \approx 0$ — plot first and pick a good starting point
