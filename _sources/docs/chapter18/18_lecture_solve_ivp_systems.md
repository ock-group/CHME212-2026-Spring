# Lecture Script — 18.3 & 18.4: `solve_ivp` and Systems of ODEs

---

## 18.3 Solving ODEs in Python with `solve_ivp`

### Opening

So far we've seen what an ODE is and how Euler's method works step by step. Now we're going to hand that work off to Python. The tool we'll use is `scipy.integrate.solve_ivp`. It does exactly what the numerical ODE solver does — marches forward from an initial condition — but with a much smarter algorithm under the hood.

There are three things to learn here:
1. How to write your ODE as a Python function
2. How to call `solve_ivp` and read what it gives back
3. What the solver is actually doing (optional, but useful to know)

---

### 18.3.1 Writing the ODE as a Python function

`solve_ivp` needs your ODE in a specific form. You write a function that takes the current time `t` and the current state `y`, and returns the derivatives:

```python
def rhs(t, y):
    return [dy_dt]
```

Two things that trip people up every time:

**First:** `y` is always a list, even when you only have one variable. So if there's only one vairable, you may unpack it, like `C_A = y[0]`.

**Second:** you always return a list. Never return a bare number.

Here's the simplest example, first-order decay:

```python
k = 0.3

def rhs(t, y):
    C_A = y[0]
    return [-k * C_A]
```

For two coupled variables, same idea — unpack both, return both derivatives:

```python
k1, k2 = 0.4, 0.1

def rhs(t, y):
    C_A, C_B = y
    return [-k1 * C_A,
             k1 * C_A - k2 * C_B]
```

The shape going in matches the shape coming out. That's the rule.

---

### 18.3.2 Calling `solve_ivp` and reading the result

```python
from scipy.integrate import solve_ivp

sol = solve_ivp(fun, t_span, y0, t_eval=..., args=(...))
```

The four things you always pass:

| Argument | What it is |
|----------|-----------|
| `fun` | your `rhs(t, y)` function |
| `t_span` | `(t_start, t_end)` — the integration window |
| `y0` | `[initial_value, ...]` — one entry per variable |
| `t_eval` | `np.linspace(t0, tf, N)` — where you want output saved |

And what comes back — the `sol` object:

| Attribute | What it contains |
|-----------|-----------------|
| `sol.t` | array of time points, shape `(N,)` |
| `sol.y` | solution array, shape `(n_vars, N)` — row `i` is variable `i` |
| `sol.success` | `True` if the solver converged |

The indexing that gets people: `sol.y[0]` is the full trajectory of the first variable. `sol.y[0, -1]` is its final value. `sol.y[1]` is the second variable if you have a system.

Quick example:

```python
k = 0.3

def rhs(t, y):
    return [-k * y[0]]

sol = solve_ivp(rhs, (0, 10), [2.0], t_eval=np.linspace(0, 10, 200))

print(sol.success)        # True
print(sol.y[0, -1])       # C_A at t=10
```

---

### 18.3.3 What is `solve_ivp` doing? (Optional)

The default method is `'RK45'`. It's still doing what Euler does — stepping forward — but instead of one slope per step, it evaluates the slope four times per step and takes a weighted average:

$$
y_{n+1} = y_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)
$$

That weighted average tracks the curve far better than Euler's single slope.

The "45" means it actually computes two estimates — 4th order and 5th order — and compares them. If they disagree too much, it rejects the step and tries a smaller `h`. If they agree closely, the next step uses a larger `h`. You never choose `h` yourself — the solver adapts it.

| Method | Error order | Step size |
|--------|------------|-----------|
| Euler | $O(h)$ | fixed, you choose |
| RK4 | $O(h^4)$ | fixed, you choose |
| **RK45** | adaptive | **automatic** |

Two tolerances control accuracy:
- `rtol=1e-3` (default) — relative: error / |y| stays below this
- `atol=1e-6` (default) — absolute: guards when y is near zero

Defaults are fine for most ChE problems. Tighten to `rtol=1e-6, atol=1e-9` when you need higher precision.

---

### 18.3.4 Passing parameters with `args`

When your ODE has physical parameters — rate constants, temperatures — pass them through `args` instead of hard-coding them as globals:

```python
def rhs(t, y, k):
    return [-k * y[0]]

sol = solve_ivp(rhs, (0, 10), [2.0], args=(0.3,))
#                                          ^^^^^ must be a tuple
```

The common mistake: `args=(k)` is just `k` in parentheses — not a tuple. Write `args=(k,)` with a trailing comma.

Why bother? Because now you can sweep over parameter values without rewriting `rhs`:

```python
for k_val in [0.1, 0.2, 0.3, 0.5]:
    sol = solve_ivp(rhs, (0, 10), [2.0], args=(k_val,), t_eval=t_eval)
    ax.plot(sol.t, sol.y[0], label=f'k = {k_val}')
```

---

## 18.4 Systems of ODEs

### What is a system of ODEs?

Up until now we've had one variable and one equation. A **system of ODEs** is when you have two or more equations, and they're **coupled** — the rate of change of each variable depends on the current values of the others.

Instead of one unknown $y(t)$, we track a vector $\mathbf{y}(t) = [y_1, y_2, \ldots, y_n]$:

$$
\frac{dy_1}{dt} = f_1(t,\, y_1, y_2, \ldots, y_n)
$$
$$
\frac{dy_2}{dt} = f_2(t,\, y_1, y_2, \ldots, y_n)
$$
$$
\vdots
$$
$$
\frac{dy_n}{dt} = f_n(t,\, y_1, y_2, \ldots, y_n)
$$

In Python: `y` is a list, `rhs` returns a list of the same length. That's it — `solve_ivp` doesn't care whether it's one variable or ten.

Two places systems come from in ChE:
1. **Multiple species** — each component's material balance is one ODE, coupled through shared reaction rates
2. **Higher-order ODEs** — a second-order ODE becomes two first-order ODEs by substituting $y_2 = dy_1/dt$

---

### 18.4.1 Consecutive Reactions: A → B → C

Classic ChE system. Reactant A converts to intermediate B, which converts to product C:

$$A \xrightarrow{k_1} B \xrightarrow{k_2} C$$

Write the material balance for each species:

$$\frac{dC_A}{dt} = -k_1 C_A$$
$$\frac{dC_B}{dt} = k_1 C_A - k_2 C_B$$
$$\frac{dC_C}{dt} = k_2 C_B$$

Three equations, three unknowns. In Python, `y = [C_A, C_B, C_C]`.

Notice the coupling: the $dC_B/dt$ equation depends on $C_A$. And $dC_C/dt$ depends on $C_B$. You can't solve them independently — they're linked. That's what makes it a system.

Also notice: $C_A + C_B + C_C = \text{const}$ (total moles conserved). Always check this after solving — it's a free verification of correctness.

```python
def consecutive_rxn(t, y, k1, k2):
    C_A, C_B, C_C = y
    dCA = -k1 * C_A
    dCB =  k1 * C_A - k2 * C_B
    dCC =  k2 * C_B
    return [dCA, dCB, dCC]

sol = solve_ivp(consecutive_rxn,
                t_span=(0, 20),
                y0=[1.0, 0.0, 0.0],
                args=(0.4, 0.1),
                t_eval=np.linspace(0, 20, 400))

# Verify mass conservation
total = sol.y[0] + sol.y[1] + sol.y[2]
print(f"Max deviation from 1.0: {np.max(np.abs(total - 1.0)):.2e}")
```

What you'll see in the plot: $C_A$ decays monotonically. $C_B$ rises, peaks, then falls as it gets consumed into $C_C$. The peak of $C_B$ is where $dC_B/dt = 0$, i.e., where $k_1 C_A = k_2 C_B$.

---

### 18.4.2 Second-Order ODE: Controlled Tank Level

**The situation.** A liquid tank has an inlet controlled by a spring-loaded valve. A sensor measures the tank level $h$ and sends an error signal to the valve: if $h$ is below the setpoint $h_{ss}$, open more; if above, close. This is the feedback controller.

The valve has a physical plug with mass $m_v$ and a spring with constant $k_s$. Because of that inertia, the valve can't respond instantly — it accelerates toward the commanded position, then decelerates as it arrives. This mechanical lag introduces a second derivative into the level dynamics.

**The ODE derivation.** Start with the tank material balance:

$$A_t \frac{dh}{dt} = F_{in}(t) - F_{out}$$

The controller drives $F_{in}$ based on the level error, but the valve dynamics obey a force balance:

$$m_v \ddot{x} + b\dot{x} + k_s x = K_c(h_{ss} - h)$$

where $x$ is valve displacement and $b$ is friction. Substituting $F_{in} \propto x$ and rearranging in terms of $h$ gives the standard second-order form:

$$\tau^2 \frac{d^2h}{dt^2} + 2\zeta\tau\frac{dh}{dt} + h = h_{ss}$$

where:
- $\tau = \sqrt{m_v / k_s}$ — the valve's natural time constant
- $\zeta = b\,/\,(2\sqrt{m_v k_s})$ — the damping ratio (friction relative to spring stiffness)

| $\zeta$ | Valve character | What the level does |
|---------|----------------|---------------------|
| $< 1$ | Lightly damped | Overshoots, oscillates before settling |
| $= 1$ | Critically damped | Reaches $h_{ss}$ fastest, no overshoot |
| $> 1$ | Heavily damped | Creeps slowly to $h_{ss}$ |

**Reducing to a first-order system.** `solve_ivp` only handles first-order ODEs. Introduce:
- $y_1 = h$
- $y_2 = dh/dt$

Then:

$$\frac{dy_1}{dt} = y_2 \qquad \frac{dy_2}{dt} = \frac{1}{\tau^2}\left(h_{ss} - y_1 - 2\zeta\tau\, y_2\right)$$

One second-order ODE → two first-order ODEs. Same trick works for any higher-order ODE.

```python
def damped_oscillator(t, y, tau, zeta, h_ss):
    h, dhdt = y
    d2hdt2 = (h_ss - h - 2*zeta*tau*dhdt) / tau**2
    return [dhdt, d2hdt2]

tau  = 2.0   # min
h_ss = 1.0   # m — setpoint, step change from h=0

for zeta in [0.3, 1.0, 2.0]:
    sol = solve_ivp(damped_oscillator, (0, 20), [0.0, 0.0],
                    args=(tau, zeta, h_ss), t_eval=np.linspace(0, 20, 400))
    ax.plot(sol.t, sol.y[0], label=f'ζ = {zeta}')
```

Initial conditions `[0.0, 0.0]` mean: tank starts empty, valve at rest.

**What to look for in the plot:** all three curves start at 0 and end at 1. The underdamped ($\zeta = 0.3$) curve overshoots and rings. The critically damped ($\zeta = 1.0$) curve reaches the setpoint cleanest. The overdamped ($\zeta = 2.0$) curve barely makes it in the same window — the sluggish valve is the bottleneck.

---

## Summary

| Topic | Key point |
|-------|-----------|
| `rhs(t, y)` signature | `y` is always a list; always return a list |
| `solve_ivp` output | `sol.y[i]` = trajectory of variable `i`; `sol.y[i, -1]` = final value |
| `args` | pass parameters as a tuple `(k,)` — never `(k)` |
| System of ODEs | one entry in `y0` per variable; `rhs` returns one derivative per variable |
| Higher-order ODE | substitute $y_2 = \dot{y}_1$ to reduce to first-order system |
| Verify results | check conservation laws (mass, energy) after solving |
