# Chapter 17: Symbolic Mathematics with SymPy — Lecture Script

---

## Opening (before opening the notebook)

Alright, let's get started on Chapter 17 — Symbolic Mathematics with SymPy.

Every chapter up to this point has been about *numbers*. You give NumPy an array of floats, it crunches through them fast. Today we're doing something fundamentally different — we're going to do *algebra* in Python. Exact algebra, the way you'd do it on a whiteboard, but automated.

The library is called **SymPy**, and by the end of this chapter you'll be able to take a derivative, evaluate an integral, and solve an equation — all symbolically — in about five lines of code.

---

## 17.1 — Motivation: Why Symbolic Math?

Let me start with the core question: why do we need this at all?

Open the comparison table in Section 17.1. Look at the difference between NumPy and SymPy:

- NumPy stores **floating-point numbers**. When you ask it for `sqrt(2)`, you get `1.41421356...` — a rounded approximation.
- SymPy stores **exact mathematical objects**. `sqrt(2)` stays as $\sqrt{2}$ until you explicitly ask for a decimal.

That distinction matters in ChE more than you might think. Here are the situations where symbolic math saves you:

1. You're working with an equation of state and you need $\partial P / \partial T$ analytically — not approximated with finite differences.
2. You want to integrate $C_p(T)$ exactly from a polynomial fit, not numerically from tabulated data.
3. You need the critical point of the van der Waals equation — that means solving two equations simultaneously for $V_c$ and $T_c$ in terms of $a$, $b$, and $R$.
4. You have an ODE like $dC/dt = -kC$ and you want the exact solution before you simulate anything numerically.

The workflow we'll use throughout this chapter is:

> **Derive symbolically → convert to a fast numerical function with `lambdify` → evaluate or plot**

Keep that in mind — it's the pattern behind every example today.

---

## 17.2 — Symbols, Expressions, and Basic Operations

### 17.2.1 — Defining Symbols

Run the imports cell first.

```python
import sympy as sp
sp.init_printing(use_unicode=True)
```

`init_printing` tells SymPy to display expressions with nice Unicode characters in the terminal. In a notebook, it renders full LaTeX.

Everything in SymPy starts with `sp.symbols()`. This is the step that makes a variable symbolic — before you call this, Python treats `x` as an ordinary variable name.

The notation is flexible — you can define one symbol or several at once:

```python
x = sp.symbols('x')              # single symbol
x, y, z = sp.symbols('x y z')    # multiple — space-separated string
a, b, c = sp.symbols('a, b, c')  # commas also work
```

For **subscripts and Greek letters**, use the LaTeX naming convention inside the string:

```python
x1, x2      = sp.symbols('x_1 x_2')       # renders as x₁, x₂
alpha, beta = sp.symbols('alpha beta')     # Greek letters
T_inf       = sp.symbols('T_inf')          # renders as T∞
```

Run that code cell and look at how these print — SymPy renders them with the proper Unicode.

Now the important part: **assumptions**. You can tell SymPy the mathematical domain of a symbol:

```python
T, P, V = sp.symbols('T P V', positive=True)   # T, P, V > 0
n       = sp.symbols('n', integer=True)         # n ∈ ℤ
x       = sp.symbols('x', real=True)            # x ∈ ℝ
```

Run the assumptions code cell. Look at the three versions of `sqrt(x²)`:

- No assumption → SymPy keeps it as `sqrt(x**2)` because $x$ could be complex.
- `real=True` → SymPy writes `Abs(x)`, since $\sqrt{x^2} = |x|$ for real $x$.
- `positive=True` → SymPy simplifies all the way to `x`, since $x > 0$ guarantees it.

For physical quantities like temperature, pressure, and volume, always declare `positive=True` — it keeps your expressions clean.

There's also a lower-level form, `sp.Symbol('x')` (singular), which is identical to `sp.symbols('x')` for a single variable. You'll see both in documentation; they behave the same way.


Run the key operations cell. Before the operations, notice that `expr1` and `expr3` are defined at the top:

```python
expr1 = x**2 + 2*x + 1       # x² + 2x + 1
expr3 = a*x**2 + b*x + c      # general quadratic
```

These are symbolic expressions built with ordinary Python operators — SymPy overloads `+`, `*`, `**`, etc. to return symbolic objects instead of numbers.

Now look at what each operation does:

- **`expand`**: multiplies out — $(x+1)^3$ becomes $x^3 + 3x^2 + 3x + 1$.
- **`factor`**: goes the other way — $x^3 - x^2 - x + 1$ becomes $(x-1)^2(x+1)$.
- **`simplify`**: catches cancellations — $(x^2 - 1)/(x - 1)$ simplifies to $x + 1$.
- **`subs`**: substitutes a value for a symbol. Single substitution: `expr.subs(x, 3)`. Multiple at once: `expr.subs([(a, 1), (b, -3), (c, 2)])`.

One thing to notice: `expr1.subs(x, 3)` returns `16` — an exact integer, not `16.0`. SymPy never introduces floating-point error unless you ask for it. You can also substitute symbolic values like `sp.pi` and the expression stays exact.

### 17.2.3 — Exact Arithmetic

Run the exact arithmetic cell. Look at this line:

```python
print("Python float  :", 1/3 + 1/6)          # 0.5
print("SymPy rational:", sp.Rational(1,3) + sp.Rational(1,6))  # 1/2
```

Both give 0.5, but the Python version gets there by rounding two finite binary fractions. The SymPy version carries the exact fraction through every step. For engineering derivations, that matters — accumulated rounding can change which root of a cubic you land on.

The key exact types to know:

- `sp.Rational(p, q)` — exact fraction $p/q$
- `sp.sqrt(n)` — exact square root, simplified automatically (`sp.sqrt(8)` → $2\sqrt{2}$)
- `sp.pi`, `sp.E`, `sp.oo`, `sp.I` — symbolic $\pi$, $e$, $\infty$, and $i$

To convert any of these to a decimal, use `sp.N(expr, n)` or equivalently `expr.evalf(n)`, where `n` is the number of significant figures. `sp.N(sp.pi, 50)` gives you $\pi$ to 50 significant figures. Both forms do the same thing — `sp.N` is a function, `.evalf()` is the method form on the expression itself.

---

## 17.3 — Symbolic Calculus

### 17.3.1 — Differentiation

The syntax is `sp.diff(expression, variable)`. Run the cell and look at the output:

```python
sp.diff(x**4, x)          # → 4x³
sp.diff(sp.sin(x), x)     # → cos(x)
sp.diff(sp.exp(-x**2), x) # → -2x·exp(-x²)  (chain rule, automatic)
```

SymPy applies the product rule, chain rule, and all differentiation rules exactly — you don't write them, SymPy knows them.

For **higher-order derivatives**, pass an integer as the third argument:

```python
sp.diff(sp.sin(x), x, 2)  # → -sin(x)  (d²/dx²)
sp.diff(x**5, x, 3)        # → 60x²    (d³/dx³)
```

For **partial derivatives**, just pass a different symbol — SymPy treats everything else as a constant automatically. Look at the ideal gas example:

```python
P_ideal = R * T / V
sp.diff(P_ideal, T)   # → R/V       (∂P/∂T, V held constant)
sp.diff(P_ideal, V)   # → -RT/V²   (∂P/∂V, T held constant)
```

For **mixed partial derivatives**, chain the variables in a single call:

```python
sp.diff(expr, x, y)      # ∂²expr/∂x∂y
sp.diff(expr, x, 2, y)   # ∂³expr/∂x²∂y
```

The result is always a new symbolic expression — you can pass it to `simplify`, `subs`, or `lambdify` just like any other SymPy expression.

### 17.3.2 — Integration

The syntax breaks into two forms:

```python
sp.integrate(expr, x)            # indefinite integral w.r.t. x  (+ C implied)
sp.integrate(expr, (x, a, b))    # definite integral from a to b
```

The limits in the tuple can be numbers, symbolic values, or SymPy constants:

```python
sp.integrate(sp.sin(x), (x, 0, sp.pi))    # limits are 0 and π
sp.integrate(sp.exp(-x), (x, 0, sp.oo))   # improper integral to ∞
sp.integrate(f, (x, a, b))                # symbolic limits — result stays symbolic
```

For **multiple integrals**, chain the variable tuples:

```python
sp.integrate(expr, (x, 0, 1), (y, 0, 1))  # ∫₀¹ ∫₀¹ expr dy dx
```

Run the cell. Notice:

```python
sp.integrate(1/x, x)                        # → ln(x)
sp.integrate(sp.exp(-x), (x, 0, sp.oo))    # → 1
```

`sp.oo` is SymPy's infinity. The integral of $e^{-x}$ from 0 to $\infty$ is exactly 1 — SymPy confirms this symbolically.

If SymPy can't find a closed form, it returns an unevaluated `Integral` object rather than raising an error. You can then call `.evalf()` on it to get a numerical result.

Now look at the ChE example. The $C_p$ of CO₂ is given as a polynomial in $T$:

$$C_p(T) = 20.71 + 0.06745T - 4.498 \times 10^{-5} T^2 + 1.119 \times 10^{-8} T^3$$

To get $\Delta H$, we integrate from 400 to 900 K:

```python
dH = sp.integrate(Cp, (T, 400, 900))
```

SymPy evaluates this exactly, then we convert to a decimal with `float(dH)`. Compare this to the trapezoidal rule from Chapter 15 — same answer, but this one is exact.

### 17.3.3 — Limits

Before the syntax, let's be clear on what a limit actually is.

The limit of $f(x)$ as $x$ approaches a point $a$ is the value $L$ that $f(x)$ gets arbitrarily close to — written:

$$\lim_{x \to a} f(x) = L$$

This is a two-sided limit: $x$ can approach $a$ from either direction. Sometimes we only care about one side:

$$\lim_{x \to a^+} f(x) \quad \text{(from the right)} \qquad \lim_{x \to a^-} f(x) \quad \text{(from the left)}$$

The two-sided limit $L$ exists if and only if both one-sided limits exist and agree:

$$\lim_{x \to a^+} f(x) = \lim_{x \to a^-} f(x) = L$$

If they disagree — say $\ln(x)$ as $x \to 0$ — the two-sided limit does not exist.

`sp.limit(expr, var, point)` evaluates the limit as `var → point`. Use `sp.oo` for $\infty$ and pass `'+'` or `'-'` as a fourth argument for one-sided limits:

```python
sp.limit(expr, x, point)       # two-sided limit as x → point
sp.limit(expr, x, 0, '+')      # one-sided from the right (x → 0⁺)
sp.limit(expr, x, 0, '-')      # one-sided from the left  (x → 0⁻)
sp.limit(expr, x, sp.oo)       # limit as x → ∞
sp.limit(expr, x, -sp.oo)      # limit as x → -∞
```

The point argument can be a number, `sp.pi`, `sp.oo`, or a symbolic variable. Run the cell:

```python
sp.limit(sp.sin(x)/x, x, 0)          # → 1
sp.limit((1 + 1/x)**x, x, sp.oo)     # → e
sp.limit(sp.ln(x), x, 0, '+')        # → -∞
```

When one-sided limits differ, `sp.limit` returns the right-hand limit by default. To check whether a limit exists, compare the `'+'` and `'-'` results explicitly.

Now look at the ChE application. We want to verify that the van der Waals equation reduces to the ideal gas law when $V \to \infty$. The compressibility factor $Z = PV/RT$ should approach 1:

```python
P_vdw = R*T/(V - b) - a/V**2
sp.limit(P_vdw * V / (R * T), V, sp.oo)   # → 1  ✓
```

This is a sanity check you can run in seconds. If someone hands you a new equation of state, you verify ideal-gas limiting behavior with one line.

---

## 17.4 — Solving Equations

### 17.4.1 — Algebraic Equations with `sp.solve`

`sp.solve(equation, variable)` returns all solutions. If you pass an expression, SymPy assumes it equals zero. If you pass `sp.Eq(lhs, rhs)`, it solves lhs = rhs.

Run the cell. Look at the quadratic formula derivation:

```python
roots = sp.solve(a*x**2 + b*x + c, x)
```

SymPy returns the two roots with the exact $\pm\sqrt{b^2-4ac}$ — the quadratic formula, derived automatically. You don't need to know the formula; SymPy derives it for you.

For a **system** of equations, pass a list of equations and a list of unknowns:

```python
eq1 = sp.Eq(2*x + 3*y, 7)
eq2 = sp.Eq(x - y, 1)
sol = sp.solve([eq1, eq2], [x, y])
```

The result is a dictionary: `{x: 2, y: 1}`. You access solutions by symbol: `sol[x]`, `sol[y]`.

For the ideal gas example — solving $PV = nRT$ for $V$:

```python
V_sol = sp.solve(sp.Eq(P*V, n*R*T), V)
# → [nRT/P]
```

One line, exact answer.

### 17.4.2 — Ordinary Differential Equations with `sp.dsolve`

This is one of the most powerful features. `sp.dsolve(ode, func)` solves ODEs symbolically.

You define the unknown function with `sp.Function`:

```python
C = sp.Function('C')
```

Then write the ODE using `.diff()`:

```python
ode1 = sp.Eq(C(t).diff(t), -k * C(t))   # dC/dt = -kC
sol1 = sp.dsolve(ode1, C(t))
```

The result is `C(t) = C1 * exp(-k*t)` — the general solution with an integration constant $C_1$.

To apply the initial condition $C(0) = C_0$, substitute $t=0$ and solve for $C_1$:

```python
C1_const = sp.solve(sol1.rhs.subs(t, 0) - C0, sp.Symbol('C1'))[0]
```

Now look at the energy balance ODE:

```python
ode3 = sp.Eq(T_f(t).diff(t), alpha - beta*T_f(t))
```

This is $\frac{dT}{dt} = \alpha - \beta T$, which comes from a tank energy balance where $\alpha = Q/(mC_p)$ and $\beta = UA/(mC_p)$. SymPy solves it and gives you the exponential approach to steady state. Notice the steady-state temperature is $\alpha/\beta$ — SymPy confirms this without any algebra on your part.

---

## 17.5 — ChE Application: van der Waals Equation of State

This is where everything comes together.

The van der Waals equation:

$$P = \frac{RT}{V - b} - \frac{a}{V^2}$$

Run the cell. We do four things in sequence.

**Step 1 — Define `P_vdw` symbolically.** Straightforward — just translate the formula into SymPy.

**Step 2 — Solve analytically for $V$.**

```python
V_solutions = sp.solve(P_vdw - P, V)
```

Passing `P_vdw - P` tells SymPy to solve $P_\text{vdw} = P$ for $V$ — "given a pressure $P$, what volumes are consistent with it?" Multiplying through by $V^2(V-b)$ clears the denominators and gives a **cubic in $V$**:

$$PV^3 - (Pb + RT)V^2 + aV - ab = 0$$

SymPy solves this cubic analytically (via Cardano's formula) and returns three roots — one per physical branch: liquid, two-phase, and vapor. Above the critical temperature only one real root exists; below it, three do.

**Step 3 — Compute $dP/dV$ and $d^2P/dV^2$.**

```python
dPdV  = sp.diff(P_vdw, V)
d2PdV2 = sp.diff(P_vdw, V, 2)
```

These are the spinodal and critical-point conditions. The spinodal curve is where $(\partial P/\partial V)_T = 0$ — the boundary of mechanical instability inside the two-phase region.

**Step 4 — Find the critical point.**

The **critical point** is the unique state $(T_c, P_c, V_c)$ where the liquid and vapor phases become identical — the meniscus between them disappears. Mathematically, it is where the $P$–$V$ isotherm has both zero slope and zero curvature:

$$\left(\frac{\partial P}{\partial V}\right)_T = 0 \qquad \text{and} \qquad \left(\frac{\partial^2 P}{\partial V^2}\right)_T = 0$$

The first condition gives a horizontal tangent (liquid and vapor compressibilities merge). The second ensures it is an inflection point, not just a local extremum — the two-phase hump has flattened entirely. Above $T_c$, no amount of pressure can condense the gas into a distinct liquid phase.

We solve both conditions simultaneously for $V_c$ and $T_c$:

```python
crit_sol = sp.solve([dPdV.subs(...), d2PdV2.subs(...)], [V_c, T_c])
```

SymPy returns:

$$V_c = 3b, \qquad T_c = \frac{8a}{27Rb}, \qquad P_c = \frac{a}{27b^2}$$

These are the classical van der Waals critical point expressions that you'd derive in a thermodynamics textbook over two pages of algebra. SymPy does it in a few seconds.

**One important note about `sp.solve` with systems:** it can return either a dictionary or a list of tuples depending on the SymPy version. The code handles both cases:

```python
if isinstance(crit_sol, list):
    crit_sol = crit_sol[0]   # list of tuples
    Vc_val, Tc_val = crit_sol[0], crit_sol[1]
else:
    Vc_val = crit_sol[V_c]   # dict
    Tc_val = crit_sol[T_c]
```

Always write defensive code like this when you use `sp.solve` on systems.

### 17.5.1 — Plotting the P–V Isotherm

Now we use `lambdify` to convert `P_vdw` into a numerical function and plot isotherms for CO₂.

```python
P_num = sp.lambdify((V, T_s, R_s, a_s, b_s), P_vdw, 'numpy')
```

We then evaluate `P_num` on a NumPy array of volumes. Look at the plot — below the critical isotherm, the curves show a region where pressure *increases* with volume. That's unphysical — it's the van der Waals artifact that requires the Maxwell equal-area construction to fix, which is a topic for your thermodynamics courses.

The critical isotherm (red, ~304 K for CO₂) has exactly one inflection point — the critical point.

---

## 17.6 — `lambdify`: Bridging SymPy and NumPy

Let me make the `lambdify` pattern explicit, because you'll use it in every project where SymPy is involved.

In short: `lambdify` is a translator. SymPy expressions are symbolic objects — they can't do arithmetic on numbers or arrays. `lambdify` wraps one into a plain Python function that can. It's equivalent to writing this by hand:

```python
# What lambdify does for you automatically:
def f_num(x):
    return x**2 + np.sin(x)
```

So instead of manually translating every SymPy symbol to its NumPy equivalent, you write:

```python
f_sym = x**2 + sp.sin(x)
f_num = sp.lambdify(x, f_sym, 'numpy')   # generates the function above
f_num(2.0)                                # scalar — works
f_num(np.linspace(0, 10, 100))            # array  — also works
```

For multiple variables, pass a tuple:

```python
g_num = sp.lambdify((x, y), x**2 + y**2, 'numpy')
g_num(3, 4)    # → 25
```

You can lambdify any SymPy expression — a derivative, an integral result, a simplified expression — not just the original formula.

Run the syntax examples cell first to get comfortable with these basics. Then run the visualization example. We start with $f(x) = \sin(x) \cdot e^{-x/3}$, compute $f'(x)$ and $f''(x)$ symbolically, then `lambdify` all three and plot them together.

The workflow is three steps:

1. **Derive** — `sp.diff(f_sym, x)`
2. **Convert** — `sp.lambdify(x, df_sym, 'numpy')`
3. **Evaluate** — `df(x_arr)` on a NumPy array

Then, at the bottom, we find the critical points of $f(x)$ by detecting sign changes in `df` and using `scipy.optimize.brentq` to locate the exact zeros. This is the symbolic-to-numerical bridge in action: SymPy gave us the exact derivative, NumPy evaluates it on an array, SciPy finds the roots.

---

## 17.7 — Summary

Look at the summary table. The key takeaways for this chapter:

| What you need | Tool |
|---|---|
| Derivative of an expression | `sp.diff(expr, x)` |
| Definite/indefinite integral | `sp.integrate(expr, (x, a, b))` |
| Roots or symbolic solutions | `sp.solve(eq, x)` |
| ODE general solution | `sp.dsolve(ode, f(t))` |
| Convert to numerical | `sp.lambdify(x, expr, 'numpy')` |

**The golden rule** — say it out loud: *Use SymPy to derive. Use NumPy to compute. Connect with `lambdify`.*

Don't use SymPy to do arithmetic on arrays — it's slow. Don't use NumPy to derive formulas — it can't. Each tool has its domain. `lambdify` is the bridge.

**Common mistakes to avoid:**

1. Forgetting to pass `x` as the second argument to `sp.diff` — SymPy needs to know which variable to differentiate with respect to.
2. Using `sp.solve` and assuming it always returns a dict — it can return a list. Check the type.
3. Trying to call a SymPy expression like a function — always `lambdify` first.
4. Omitting `positive=True` on physical symbols — SymPy may produce expressions with absolute values or complex branches that simplify away once you add the assumption.

---

## Closing

That's Chapter 17. Next chapter we'll look at how to use SymPy together with the numerical ODE solvers from Chapter 16 — derive the Jacobian symbolically, then hand it off to `scipy.integrate.solve_ivp` to get much faster and more stable integration.

Any questions before we move on?
