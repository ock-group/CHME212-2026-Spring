# Lab 11 Lecture Script — Root Finding

**Course:** CHME 212  
**Time:** ~25–30 min demo + ~45 min student work

---

## Before Class

Pull up `11_lab.ipynb`. Have the Chapter 16 notebook open in another tab as a reference. The key table to keep visible:

| Task | Tool | Syntax |
|------|------|--------|
| Polynomial roots | `np.roots` | `np.roots([c0, c1, ..., cn])` |
| Single equation | `brentq` | `brentq(f, a, b)` |
| System of equations | `fsolve` | `fsolve(F, x0)` |

---

## Opening (3 min)

"Today we're doing something different — we're not computing *forward* from a known equation. Instead, we're asking: *given that this equation equals zero, what is x?* That's root finding."

"You've been doing this since high school with the quadratic formula. The quadratic formula is just a special case of root finding where someone worked out the algebra for you. For anything more complicated — cubic equations of state, equilibrium expressions, coupled reactor balances — there's no formula. You hand it to an algorithm."

"We have three tools today. Let me show you each one on a real ChE example, then you'll use all three on the practice problems."

---

## The Workflow — Write It on the Board (2 min)

Before touching any code, write these four steps where students can see them the whole time:

```
1. Rewrite your equation as f(x) = 0
2. PLOT f(x) — count roots, find brackets
3. Choose your tool and solve
4. VERIFY: print f(x*) — it must be ≈ 0
```

"Step 4 is not optional. `fsolve` in particular can give you a wrong answer with no warning. Always check."

---

## Example 1: van der Waals Molar Volume with `brentq` (10 min)

**The physical setup:**

"We have CO₂ at 310 K and 70 bar — that's above room temperature, well above ambient pressure. We want to know the molar volume. The ideal gas law says $V = RT/P$, let's compute that quickly."

```python
V_ideal = R * T / P
print(f"Ideal gas V = {V_ideal:.4f} L/mol")
```

"We get about 0.37 L/mol. But CO₂ at 70 bar is not ideal — it's being compressed significantly. The van der Waals equation corrects for molecular size and attractions."

**Step 1 — rewrite as f(V) = 0:**

"We can't solve van der Waals analytically for V — well, technically it's a cubic so it has three roots, but the formula is a mess. Instead we rewrite it as f(V) = 0."

```python
def f_vdw(V):
    return (P + a / V**2) * (V - b) - R * T
```

"f(V) is the left side minus the right side. When f(V) = 0, the equation is satisfied."

**Step 2 — plot:**

"Before calling any solver, always plot. This is non-negotiable."

```python
V_range = np.linspace(b + 0.001, 1.0, 500)

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(V_range, f_vdw(V_range), 'steelblue', linewidth=2)
ax.axhline(0, color='k', linewidth=1)
ax.set_ylim(-5, 10)
ax.set_xlabel('Molar volume V (L/mol)')
ax.set_ylabel('f(V)')
ax.set_title('van der Waals: f(V) = 0')
plt.tight_layout()
plt.show()
```

"What do I see? The function crosses zero somewhere around 0.2 L/mol. I can see f(0.1) is negative and f(0.5) is positive — that's my bracket. Notice I started the range at b + 0.001, not zero, because the equation blows up when V = b."

**Step 3 — solve:**

"Now `brentq`. It needs the function and a bracket — two x values where f has opposite signs."

```python
V_root = brentq(f_vdw, a=0.1, b=0.5)
```

"Note: `a` and `b` here are the bracket endpoints, not the van der Waals constants — different variable names, be careful."

**Step 4 — verify:**

```python
print(f"van der Waals V = {V_root:.5f} L/mol")
print(f"Ideal gas V     = {V_ideal:.5f} L/mol")
print(f"Residual f(V*)  = {f_vdw(V_root):.2e}")
print(f"Compressibility Z = PV/RT = {P*V_root/(R*T):.4f}")
```

"Z is about 0.59 — CO₂ is much more compressed than ideal gas predicts. The residual is around 1e-14, essentially machine zero. That's how you know the answer is right."

**Key point to emphasize:**

"Why `brentq` and not Newton's method? Because `brentq` is guaranteed to find a root as long as your bracket has a sign change. Newton can diverge. `brentq` combines bisection's safety with near-Newton speed. It's the practical workhorse for single-variable root finding."

---

## Example 2: Two Curves Intersecting with `fsolve` (8 min)

**The physical setup:**

"Now we have *two* unknowns and *two* equations. Single-variable solvers can't handle this — we need `fsolve`."

"Where does $y = x^2$ intersect $y = 2 - x$? Geometrically it's obvious — a parabola and a line. But if this were a CSTR mole balance and energy balance instead of algebra, we'd have to do this numerically."

**Step 1 — rewrite as F(x) = 0:**

"The trick with `fsolve` is packaging everything into one function that takes a *list* of unknowns and returns a *list* of residuals."

```python
def equations(vars):
    x, y = vars
    eq1 = y - x**2       # parabola: y = x^2 rearranged
    eq2 = y + x - 2      # line: y = 2-x rearranged
    return [eq1, eq2]
```

"Two unknowns in, two residuals out. Always match the count."

**Solving — initial guess matters:**

"Unlike `brentq`, `fsolve` doesn't need a bracket. It needs an initial *guess*. And unlike `brentq`, it is not guaranteed to converge — or to find the root you want. Use different guesses to find different solutions."

```python
sol1 = fsolve(equations, x0=[-2.0, 3.0])
sol2 = fsolve(equations, x0=[ 1.0, 1.0])
```

**Step 4 — verify:**

"This step is critical for `fsolve`. It can silently return a wrong answer."

```python
print(f"Solution 1: x = {sol1[0]:.6f}, y = {sol1[1]:.6f}")
print(f"  residuals: {equations(sol1)}")
print(f"Solution 2: x = {sol2[0]:.6f}, y = {sol2[1]:.6f}")
print(f"  residuals: {equations(sol2)}")
```

"Both residuals should be near 1e-12 or smaller. If you get residuals that are large, your solver converged to a non-solution."

**Visualize to confirm:**

```python
x_p = np.linspace(-2.5, 2.0, 300)
fig, ax = plt.subplots(figsize=(5, 5))
ax.plot(x_p, x_p**2,  'steelblue',  lw=2, label='$y = x^2$')
ax.plot(x_p, 2 - x_p, 'darkorange', lw=2, label='$y = 2 - x$')
ax.scatter([sol1[0], sol2[0]], [sol1[1], sol2[1]],
           color='red', s=80, zorder=5, label='Solutions')
ax.set_xlim(-2.5, 2.0); ax.set_ylim(-0.5, 5)
ax.legend()
plt.tight_layout()
plt.show()
```

**When to use `fsolve` vs `brentq`:**

"If you have one equation and one unknown — use `brentq`. It's safer. If you have N equations and N unknowns — `fsolve` is your only option. Remember: always plot what you can before calling it, and always verify the residuals."

---

## Warm-Up Exercises — Student Work (5 min)

"Three quick exercises to get the syntax in your fingers before the main problems. Work through them now — call me over if you're stuck. Five minutes."

**What to watch for while circulating:**

- **Exercise 1:** Students forgetting to plot before choosing a bracket. Make sure they identify where f changes sign visually.
- **Exercise 2:** Students passing coefficients in the wrong order to `np.roots` (must be highest-to-lowest degree). The polynomial is $V^3 - 6V^2 + 11V - 6$, so coefficients are `[1, -6, 11, -6]`. Roots are 1, 2, 3.
- **Exercise 3:** Students trying to find both solutions with the same initial guess. Stress that different guesses are needed for different roots.

---

## Problem 1 Walkthrough — Equilibrium vs. Temperature (5 min intro)

"Problem 1 is a gas-phase equilibrium: A ⇌ 2B at 3 atm. The equilibrium constant depends on temperature through $K_{eq}(T) = \exp(4.0 - 3000/T)$."

**Setting up f(X):**

"The equilibrium condition gives us this expression for f(X):"

$$f(X_e) = \frac{4X_e^2}{1 - X_e^2} \cdot P - K_{eq}(T)$$

"Notice f depends on both X and T. In part (a) T is fixed at 800 K. In part (b) you loop over temperatures and solve for X at each one."

**Hint for part (a):**

"Your `brentq` call will look like `brentq(lambda X: f_eq(X, T_val), 0.01, 0.99)`. The lambda wraps away the T argument so `brentq` only sees a function of X."

**Hint for part (b) — the loop:**

"For the temperature sweep, you're solving the same root-finding problem 200 times, once per temperature. Call `brentq` inside the loop with the current T value."

**Physical intuition to prompt students:**

"As you plot the result, ask yourself: does conversion go up or down as temperature increases? And does that make sense given the sign of $\Delta H$? The $K_{eq}$ formula here has a positive coefficient for T, meaning K increases with T, meaning equilibrium shifts toward products. That's endothermic behavior."

---

## Problem 2 Walkthrough — CSTR Multiple Steady States (5 min intro)

"Problem 2 is the most important one. Real CSTRs can have multiple steady states — same reactor, same operating conditions, but the system can sit at completely different temperatures and conversions depending on how you start it up."

**The equations:**

"The mole balance says: rate of reaction times residence time equals conversion. The energy balance says: heat released by reaction equals heat absorbed by the fluid. Both depend on X and T simultaneously, so we need `fsolve`."

**Setting up the residuals:**

```python
def cstr_balances(vars):
    X, T = vars
    mole_balance   = tau * k_arr(T) * (1 - X) - X
    energy_balance = X * (-dHrxn) - rho_Cp * (T - T_feed)
    return [mole_balance, energy_balance]
```

"Two equations, two unknowns. Always unpack the solution as `X_ss, T_ss = sol`."

**Part (b) — the interesting part:**

"Try three very different initial guesses. You might get the same answer every time — or you might get different ones. Either result is informative. If you get the same answer, it suggests there's only one stable steady state for these parameters. If you find multiple, you've discovered that this reactor has ignition/extinction behavior."

"The example output shows the solver converging to one low-conversion cold state regardless of guess — that's a valid physical result. Different parameter sets will give multiple solutions. What matters is that you always verify residuals."

**Common mistake to warn about:**

"If your residuals are large — like 1e-2 or bigger — `fsolve` did not converge to a solution. Try a different initial guess. Don't report a solution with bad residuals."

---

## Closing (2 min)

"Let me summarize what we used today:

- **`np.roots`** — only works for polynomials, but finds *all* roots at once including complex ones. Filter for the physically meaningful real one.
- **`brentq`** — guaranteed convergence if you give it a valid bracket. The go-to for any single-equation problem.
- **`fsolve`** — handles systems but is not guaranteed. Always verify residuals. Use multiple initial guesses when you suspect multiple solutions.

The workflow never changes: rewrite as f = 0, plot, choose your tool, verify. That order protects you from wrong answers."

"For the lab submission: complete the warm-up and both practice problems. Make sure every root-finding result has a residual printed next to it."

---

## Common Questions

**"My brentq is giving a ValueError."**  
The bracket endpoints have the same sign. Go back and plot f(x) — find where it crosses zero and pick one point on each side.

**"fsolve gave me an answer but the residuals are like 1e-2."**  
That is not a solution. Try a different initial guess. Sometimes the solver converges to a stationary point instead of a root.

**"How do I know which root is the physical one from np.roots?"**  
Filter by `np.isreal(r)` and `r.real > 0` (or whatever physical constraint applies). For the van der Waals cubic you want the real positive root larger than b.

**"My temperature sweep crashes with a ValueError in brentq."**  
f(0.01) and f(0.99) may have the same sign at some temperatures — meaning the equilibrium conversion falls outside your bracket. Add a try/except or check f at the bracket endpoints before calling brentq.
