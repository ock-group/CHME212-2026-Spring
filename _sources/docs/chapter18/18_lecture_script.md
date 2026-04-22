# Chapter 18 Lecture Script: Ordinary Differential Equations
### Sections 18.1 – 18.3
---

## Before You Begin

**What students need from earlier chapters:**
- Derivatives and the idea that dy/dt is a slope
- Numpy arrays and matplotlib basics
- Writing Python functions

**Learning goals for today:**
By the end of this lecture students should be able to:
1. Recognize and write down a first-order ODE from a word description
2. Explain what an initial condition does and why it is required
3. Carry out two Euler steps by hand given an ODE and a step size
4. Translate any first-order ODE (or system) into a Python function for `solve_ivp`
5. Call `solve_ivp`, check whether it converged, and extract the solution

**Approximate timing:** 75 minutes total (25 / 25 / 25)

---

## Opening Hook (3 min)

Start by putting this equation on the board with no context:

```
dC_A/dt = -k * C_A
```

Ask the room: *"What is this telling you?"*

Let students respond. Likely answers: "how concentration changes", "the rate of reaction", "an ODE".

Then say:

> "Every process you will ever model in chemical engineering — a reactor, a heat exchanger, a distillation column during startup, a drug moving through the bloodstream — can eventually be written down as one of these. An ordinary differential equation. Today we learn what that means, and by the end of this lecture you will be able to solve any first-order ODE in Python in about ten lines of code."

---

---

# 18.1  What Is an ODE? (25 min)

---

## Opening contrast (3 min)

Write side by side on the board:

```
Algebraic equation:    2x + 3 = 7
ODE:                   dC_A/dt = -k * C_A
```

Ask: *"What is the difference between these two?"*

The key point to draw out:
- The algebraic equation tells you the **value** of x directly. You solve it and you're done.
- The ODE does **not** tell you C_A. It tells you the **rule by which C_A changes**. You know the slope everywhere, but you don't yet know the curve.

Say:

> "An ODE is not an answer — it's a rule. To get the answer — the actual trajectory C_A(t) — you have to integrate it. And to integrate it, you need to know where to start. That starting point is the initial condition."

---

## 18.1.1  The standard form (5 min)

Write on the board:

```
dy/dt = f(t, y),    y(t₀) = y₀
```

Walk through each piece:

**y(t)** — this is the unknown function we want to find. In a batch reactor it is concentration. In a heat exchanger it is temperature. In a tank it is liquid level. It is whatever quantity is evolving.

**f(t, y)** — this is the right-hand side. It is an expression that tells you the slope at any given time t and any given value y. Crucially, we know this. The ODE gives it to us.

**y(t₀) = y₀** — the initial condition. This is the value of y at the starting time t₀. Without it, we cannot determine the solution uniquely.

**The car analogy:**

> "Imagine I give you a speedometer that tells you the car's velocity at every instant. Can you tell me where the car is? No. You also need to know where it started. The initial condition is that starting location. The ODE is the speedometer."

Write this mapping explicitly:

```
velocity   ↔   dy/dt = f(t, y)    (the rule of change, given)
position   ↔   y(t)               (what we want)
start loc  ↔   y(t₀) = y₀        (initial condition, given)
```

**Pause and check:** Ask students: "In the batch reactor equation dC_A/dt = -k*C_A, what plays the role of f(t,y)?" Answer: f(t, C_A) = -k * C_A. Note that it doesn't actually depend on t here — that's common but not always the case.

---

## 18.1.2  Order of an ODE (4 min)

> "The order of an ODE is just the highest derivative that appears."

Write the three cases:

```
1st order:   dy/dt  = f(t, y)
2nd order:   d²y/dt² = f(t, y, dy/dt)
nth order:   involves d^n y/dt^n
```

Give ChE examples for each:
- 1st order: batch reaction, Newton's cooling, CSTR startup
- 2nd order: spring-mass system, liquid tank with spring-loaded valve, vibrating pipeline

Then make the key point that students should write down:

> "Any higher-order ODE can always be rewritten as a **system** of first-order ODEs."

Show the conversion for a second-order equation:

```
d²x/dt² = -ω²x

Let:  y₁ = x           (position)
      y₂ = dx/dt       (velocity)

Then:
  dy₁/dt =  y₂
  dy₂/dt = -ω²y₁
```

> "We introduced a new variable for each derivative. Now we have two first-order equations instead of one second-order equation. This trick is how `solve_ivp` handles everything — it only knows how to do first-order systems, but that's enough for all of engineering."

---

## 18.1.3  Solving an ODE = integration (4 min)

Start from the standard form and rearrange:

```
dy/dt = f(t, y)
dy = f(t, y) dt
```

Integrate both sides from t₀ to t:

```
y(t) = y₀ + ∫[t₀ to t]  f(τ, y(τ)) dτ
```

> "This is just integration. But notice the problem: f depends on y(τ), which is the very thing we are trying to find. It's circular. You can't evaluate the integral without knowing y, but you can't know y without evaluating the integral."

> "This is why we need numerical methods. They break the circularity by doing something very practical: they start at y₀, which we know, compute f(t₀, y₀), which gives the slope right now, take a small step forward using that slope, arrive at an approximate y₁, and repeat. Each step uses only information that we already have."

This is the bridge into 18.2.

---

## 18.1.4  Verifying an analytical solution (4 min)

> "For simple ODEs like dC_A/dt = -k*C_A, we actually can find the solution analytically. Let me show you how to verify it — this is a skill you'll use to check numerical results."

Write the proposed solution:

```
C_A(t) = C_A0 * e^(-kt)
```

Verify by substitution — differentiate the right side and check it equals the ODE:

```
d/dt [C_A0 * e^(-kt)] = -k * C_A0 * e^(-kt) = -k * C_A(t)  ✓
```

> "It satisfies the ODE. Also check the initial condition: at t=0, C_A(0) = C_A0 * e^0 = C_A0. ✓"

> "In the notebook we plot this solution. Notice that the curve starts at C_A0 and decays exponentially. Most ODEs do not have such clean analytical solutions — that's exactly why we need numerical methods."

**Key takeaway to state explicitly:**

> "Whenever you solve an ODE numerically, always ask yourself: does this limit case have an analytical solution I can use to verify my code? For first-order linear ODEs with constant coefficients, the answer is usually yes."

---

## 18.1.5  Python syntax primer (5 min)

> "Before we write a single solver call, I want to make sure everyone is comfortable translating a math equation into a Python function. This is a purely mechanical translation — once you see the pattern, it becomes automatic."

Write the pattern on the board:

```python
def rhs(t, y):
    # t : scalar — current time
    # y : list   — current values of all variables
    return [list of derivatives]
```

**Two rules. Repeat them.**

Rule 1: `y` is always a list. Even if you only have one variable, you unpack it as `y[0]`.

Rule 2: You always return a list. Even if you only have one derivative.

Show a single-variable example first:

```python
# ODE: dC_A/dt = -k * C_A
k = 0.3

def rhs(t, y):
    C_A = y[0]
    return [-k * C_A]
```

Walk through it line by line:
- `y[0]` unpacks C_A from the state list
- `-k * C_A` is f(t, y) — the right-hand side of the ODE
- We return it inside a list `[...]`

Then show a two-variable example:

```python
# System: dC_A/dt = -k1*C_A
#         dC_B/dt =  k1*C_A - k2*C_B
k1, k2 = 0.4, 0.1

def rhs_system(t, y):
    C_A, C_B = y        # unpack both variables
    return [-k1 * C_A,
             k1 * C_A - k2 * C_B]
```

> "Notice: two variables go in, two derivatives come out. The list you return must always have the same number of entries as the list y."

**Quick test for the room:** Write on the board: `dT/dt = (T_in - T) / tau`. Ask: "Write me the Python function for this ODE." Give 60 seconds. Expected answer:

```python
def rhs(t, y):
    T = y[0]
    return [(T_in - T) / tau]
```

---

---

# 18.2  Euler's Method (25 min)

---

## Framing (2 min)

> "We now know what an ODE is, and we know that solving it means finding y(t) given the rule dy/dt = f(t,y) and a starting point. Let's build the simplest possible numerical method from scratch. This method, called Euler's method, is not what we use in practice — but understanding it completely will make everything that follows obvious."

---

## 18.2.1  The stepping formula (8 min)

> "Here is the core idea. Suppose we are at time t₀ and we know y₀. The ODE tells us the slope right now: it is f(t₀, y₀). If we assume the slope stays constant over a short time interval h, then after time h the value of y is approximately:"

Write on the board, building it up step by step:

```
Start at:   (t₀, y₀)
Slope now:  f(t₀, y₀)
After h:    y₁ = y₀ + h * f(t₀, y₀)
```

> "That's it. That one line is Euler's method. Now we're at (t₁, y₁). We do it again."

```
At (t₁, y₁):
Slope now:  f(t₁, y₁)
After h:    y₂ = y₁ + h * f(t₁, y₁)
```

Write the general pattern:

```
y₁ = y₀ + h * f(t₀, y₀)
y₂ = y₁ + h * f(t₁, y₁)
y₃ = y₂ + h * f(t₂, y₂)
...
yₙ = yₙ₋₁ + h * f(tₙ₋₁, yₙ₋₁)
```

> "Each step uses only the result from the previous step. We are marching forward in time, one step at a time. Notice we never need to know y at any other point — just the immediate previous value."

**Now refer to the notebook visualization.**

> "Look at the figure in the notebook. The exact solution is the smooth black curve. The blue arrow is step 1: we start at y₀, we know the slope there — that's f(t₀, y₀) — and we follow it in a straight line for horizontal distance h. That straight-line landing point is y₁."

Walk through the labeled quantities:
- **h**: the horizontal distance — the step size — shown by the gray bracket
- **h·f(t₀, y₀)**: the vertical rise of that straight-line step — shown by the blue bracket
- **error of y₁**: the red dotted gap between y₁ (where Euler landed) and the exact curve at t₁

> "Why is there an error? Because the slope is not actually constant over [t₀, t₁]. The true solution is curved. Euler assumed it was straight. The difference is the error."

> "The orange arrow is step 2. Notice that Euler starts from y₁, which is already a little wrong. So the second step has two sources of error: the curvature mistake within this step, plus the fact that we started from a wrong value."

**The student question from the notebook.** Pose it directly:

> "Looking at this picture, what would you do to reduce the error?"

Take responses. Guide toward: make h smaller. The steps would be shorter, the tangent-line approximation would be more accurate within each step, and the errors would accumulate more slowly. This is exactly right — and we'll quantify exactly how much smaller.

---

## 18.2.2  Why does the error behave the way it does? (10 min)

> "Let's think carefully about where the error comes from, because this will tell us how to make it smaller."

**The Taylor series argument.** Write on the board:

> "The exact value of y at t + h is given by the Taylor series:"

```
y(t + h) = y(t) + h·y'(t) + (h²/2)·y''(t) + (h³/6)·y'''(t) + ...
```

> "This expansion is exact — it's not an approximation. It is the precise value of y(t+h) expressed in terms of derivatives at t."

> "Euler's method keeps only the first two terms:"

```
y(t + h) ≈ y(t) + h·y'(t)
                = y(t) + h·f(t, y)   ← because y' = f(t,y)
```

> "So the **local truncation error** — the error we make in a single step — is everything we dropped:"

```
e_local = (h²/2)·y''(t) + higher order terms
        ≈ (h²/2)·y''(t)
        ~ O(h²)
```

Pause here and make sure this lands. Ask: *"If I cut h in half, what happens to the local error?"* It goes down by a factor of 4 (since h² appears). Good.

**But the global error is only O(h), not O(h²).** This is the crucial point that trips students up.

> "Over a total time T, how many steps do we take? N = T/h. So we make N errors, each of size roughly h²/2 * y''. How big is their sum?"

```
e_global ≈ N * e_local
         = (T/h) * (h²/2) * |y''|
         = (T/2) * h * |y''|
         ~ O(h)
```

Write this cancellation explicitly:

```
(1/h) × h² = h
```

> "One power of h cancels. The global error is only first-order in h, not second-order. This means: to get 10 times more accurate, you need 10 times more steps. That's a lot of work for not much payoff."

Summarize in a table on the board:

```
Local truncation error (1 step):   O(h²)   ← halving h → 4× smaller
Number of steps:                   O(1/h)  ← halving h → 2× more steps
Global error (total):              O(h)    ← halving h → 2× smaller
```

**Concrete verification.** Bring this to life with the batch reactor:

> "For dC_A/dt = -k·C_A, we know y'' = k²·C_A. So the local error at the first step is exactly (h²/2)·k²·C_A0. Let's check: if h = 1, k = 0.3, C_A0 = 2.0, then e_local ≈ (0.5)(0.09)(2.0) = 0.09 mol/L. That's almost 5% error in a single step — already not great. But if h = 0.1, e_local ≈ 0.0009 mol/L. Much better."

> "The notebook shows the convergence plot: as h decreases, the error falls along a straight line on a log-log scale with slope 1. That slope-1 line is the signature of first-order accuracy."

---

## 18.2.3  Implementing Euler in Python (5 min)

> "Let's code Euler's method. This is just the formula in a loop."

Write on the board:

```python
def euler(f, y0, t_span, h):
    t0, tf = t_span
    t_arr = [t0]
    y_arr = [y0]
    t_cur, y_cur = t0, y0
    while t_cur < tf:
        y_cur = y_cur + h * f(t_cur, y_cur)
        t_cur = t_cur + h
        t_arr.append(t_cur)
        y_arr.append(y_cur)
    return t_arr, y_arr
```

Walk through line by line:

- We initialize lists with the starting point
- Each iteration: compute the new y using the formula, advance time by h
- Append both to our lists
- Return arrays when done

> "Notice this function takes `f` as an argument — the ODE right-hand side. It doesn't care what the ODE is. You write your ODE as a Python function, and you pass it in."

**Run it mentally:** Say aloud the first two iterations for dC_A/dt = k*C_A with k=0.3, y₀=0.5, h=2:

```
t=0:  y=0.5,    slope = 0.3*0.5 = 0.15,   y₁ = 0.5 + 2*0.15 = 0.8
t=2:  y=0.8,    slope = 0.3*0.8 = 0.24,   y₂ = 0.8 + 2*0.24 = 1.28
```

> "Students, check: the exact value at t=4 is 0.5*e^(0.3*4) = 0.5*e^1.2 ≈ 1.66. Euler got 1.28. That's an error of about 0.38 — almost 23%. This is why we don't use Euler in practice."

---

## 18.2.4  Wrap-up: the limitations of Euler (2 min)

Summarize at the board:

> "Euler's method has three problems:"

```
1. First-order accuracy — need 10× more steps to get 10× better answer
2. Fixed step size — can't adapt to regions where the solution changes fast
3. Single slope evaluation — uses only the slope at the start of the step,
   ignoring how the slope changes within the step
```

> "All three of these are fixed by the methods inside `solve_ivp`. That's what we cover next."

---

---

# 18.3  Solving ODEs in Python with `solve_ivp` (25 min)

---

## Framing (1 min)

> "Everything in 18.2 was about building intuition. Now we switch to the tool you will actually use for the rest of this course and your careers: `scipy.integrate.solve_ivp`. The workflow has three steps. I'll do each step in turn, with code."

Write on the board:

```
Step 1: Write the ODE as a Python function
Step 2: Call solve_ivp and read the result
Step 3: Understand what the solver is doing
```

---

## 18.3.1  Step 1 — Writing the ODE function (7 min)

> "The first thing to internalize is that `solve_ivp` does not know your equation. You have to give it the right-hand side as a Python function. The contract is fixed and simple."

Write the signature on the board:

```python
def rhs(t, y):
    # inputs:
    #   t : scalar — current time
    #   y : list   — current state, one entry per variable
    # output:
    #   list of derivatives, same length as y
    return [dy_dt]
```

**Rule 1: y is always a list.** Even for a single variable. Say this twice.

> "I see this mistake every semester. Someone writes `return -k * y` and gets an error because y is a list, not a number. Always write `y[0]` to get the first variable."

**Rule 2: return a list, not a scalar.** Say this twice.

> "If you return just a number, `solve_ivp` will complain. Always wrap it: `return [-k * y[0]]`."

Write two examples side by side on the board:

```python
# Single ODE: dC_A/dt = k * C_A
k = 0.3

def rhs_single(t, y):
    C_A = y[0]
    return [k * C_A]      # list with one entry
```

```python
# Two ODEs:  dC_A/dt = -k1 * C_A
#            dC_B/dt =  k1*C_A - k2*C_B
k1, k2 = 0.4, 0.1

def rhs_system(t, y):
    C_A, C_B = y          # unpack both
    return [-k1 * C_A,
             k1 * C_A - k2 * C_B]   # list with two entries
```

**Quick debugging trick:**

> "Before you call the solver, test your function manually. Call `rhs_single(0, [0.5])` and check you get `[0.15]`. This takes 5 seconds and catches 90% of syntax bugs."

Write this on the board:

```python
print(rhs_single(0, [0.5]))   # should print [0.15]
```

---

## 18.3.2  Step 2 — Calling `solve_ivp` and reading the result (10 min)

Write the full call on the board:

```python
from scipy.integrate import solve_ivp

sol = solve_ivp(fun, t_span, y0, t_eval=..., args=(...))
```

Go through each argument one at a time. Do not rush this.

**`fun`** — your ODE function. Just pass the name, no parentheses.

**`t_span`** — a tuple `(t_start, t_end)`. This tells the solver the interval to integrate over. Example: `(0, 10)` means from t=0 to t=10 minutes.

**`y0`** — the initial conditions, as a list. One value per variable. Example: `[0.5]` for a single variable starting at 0.5. `[1.0, 0.0, 0.0]` for a three-variable system.

**`t_eval`** — optional, but almost always used. A numpy array of times where you want the solution saved. If you omit it, the solver picks its own internal times — often inconvenient for plotting. Use `np.linspace(0, 10, 200)` to get 200 evenly-spaced points.

**`args`** — optional tuple of extra parameters to pass to your ODE function. Covered in 18.3.4.

**Now: reading the result.**

> "`solve_ivp` returns an object — let's call it `sol`. Three things you always check:"

Write on the board:

```python
sol.success    # True if the solver converged — always check this first!
sol.t          # array of time points, shape (N,)
sol.y          # array of solution values, shape (n_vars, N)
```

Spend extra time on `sol.y` — this is the one that confuses students.

> "`sol.y` is a 2D array. Row 0 is the trajectory of variable 0. Row 1 is variable 1. And so on."

Draw this on the board as a literal 2D table:

```
sol.y:
         t₀      t₁      t₂    ...   t_N
var 0 [  0.50    0.65    0.84  ...   10.2  ]
var 1 [  0.00    0.03    0.08  ...    0.1  ]
```

> "So `sol.y[0]` gives the entire trajectory of variable 0 — a 1D array of length N. `sol.y[0, -1]` gives the final value of variable 0. `sol.y[1]` gives the trajectory of variable 1."

**Live example — work through it together:**

```python
k = 0.3

def rhs(t, y):
    return [k * y[0]]

sol = solve_ivp(
    fun    = rhs,
    t_span = (0, 10),
    y0     = [0.5],
    t_eval = np.linspace(0, 10, 6)
)

print(sol.success)     # True
print(sol.t)           # [0, 2, 4, 6, 8, 10]
print(sol.y[0])        # six values of C_A
print(sol.y[0, -1])    # C_A at t=10
```

Ask students to predict the answer before running: "C_A(10) = 0.5 * e^(0.3*10) = 0.5 * e^3 ≈ 10.04."

> "The notebook cell runs this and shows the output. Notice how closely the numerical solution tracks the exact curve — the markers sit right on the line. This is the difference between Euler and RK45."

---

## 18.3.3  Step 3 — Under the hood: what is RK45? (5 min)

> "You don't need to understand this to use solve_ivp, but knowing the idea will help you debug and choose solver settings."

Write side by side:

```
Euler:   y_{n+1} = yₙ + h * k₁
                          ↑
                   slope at start only

RK45:    y_{n+1} = yₙ + (h/6) * (k₁ + 2k₂ + 2k₃ + k₄)
```

> "RK45 takes four slope measurements within each step — at the start, two midpoints, and the end — and combines them with the weights 1, 2, 2, 1. This weighted average tracks the curvature of the true solution much more closely than a single slope at the start."

Explain the error order comparison:

```
Euler:  global error O(h)    — halving h → 2× more accurate
RK4:    global error O(h⁴)   — halving h → 16× more accurate
RK45:   adaptive h           — error controlled by rtol, atol
```

> "The '45' in RK45 means it runs both a 4th-order and a 5th-order estimate at each step. The difference between them is the error estimate. If the error is too large, the step is rejected and retried with smaller h. If the error is much smaller than needed, the next step is bigger. You never choose h — the solver finds the right h automatically."

**The tolerances:**

> "Two keywords control accuracy: `rtol` (relative tolerance, default 1e-3) and `atol` (absolute tolerance, default 1e-6). For most ChE problems the defaults are fine. If you need higher accuracy — say, for a phase portrait or stiff reactor — use `rtol=1e-6, atol=1e-9`."

Show the convergence plot from the notebook:

> "Here is the key picture: Euler errors fall along a slope-1 line on the log-log plot. The RK45 error is the flat horizontal line at the bottom. Notice it is orders of magnitude lower with far fewer function evaluations."

---

## 18.3.4  Passing parameters with `args` (3 min)

> "Your ODE function will almost always have parameters — rate constants, temperatures, volumes. There are two ways to handle them."

**Bad way — global variables:**

```python
k = 0.3                    # global variable

def rhs(t, y):
    return [k * y[0]]      # silently uses k from outer scope
```

> "This works, but it's fragile. If you have multiple ODEs with different k values, or you want to run a loop over k, you'll need to redefine the function every time."

**Good way — `args`:**

```python
def rhs(t, y, k):          # k is a parameter
    return [k * y[0]]

sol = solve_ivp(rhs, (0, 10), [0.5], args=(0.3,))
```

> "`solve_ivp` will call your function as `rhs(t, y, 0.3)` at every step. The `args` tuple is appended after t and y."

**The trailing comma.** Write this on the board and underline it:

```python
args=(0.3,)    ← correct: tuple with one element
args=(0.3)     ← WRONG:  just parentheses around 0.3, not a tuple
```

> "Python requires a trailing comma to make a one-element tuple. `(0.3)` is just 0.3 in parentheses. `(0.3,)` is a tuple. The difference will give you a cryptic TypeError."

Now show the parameter sweep from the notebook:

```python
for k_val in [0.1, 0.2, 0.3, 0.5]:
    sol = solve_ivp(rhs, (0, 10), [0.5], args=(k_val,), t_eval=t_eval)
    ax.plot(sol.t, sol.y[0], label=f'k = {k_val}')
```

> "Four different k values, one function definition, four lines of code. This is the payoff of using `args` instead of global variables."

---

## Closing: the complete workflow (2 min)

Write the four-line workflow on the board and leave it there:

```
1. Write the ODE function:
       def rhs(t, y, *params):
           return [f(t, y)]

2. Call solve_ivp:
       sol = solve_ivp(rhs, (t0, tf), [y0], t_eval=..., args=(...))

3. Always check:
       assert sol.success

4. Extract results:
       sol.t      — time
       sol.y[0]   — first variable
       sol.y[1]   — second variable (if system)
```

> "That's the entire workflow. In the next section we'll apply it to consecutive reactions, second-order dynamics, and the non-isothermal batch reactor. Every one of those problems fits into exactly this framework."

---

---

## Frequently Asked Questions

**Q: What if my ODE doesn't depend on t explicitly?**
A: You still need `t` as the first argument — `solve_ivp` always passes it. Just ignore it in the function body. Example: `def rhs(t, y): return [-k * y[0]]` — t is there but never used.

**Q: Do I have to use `t_eval`?**
A: No. If you omit it, `solve_ivp` saves the solution at its own internal adaptive time points. These may not be evenly spaced. Usually you want `t_eval` for plotting.

**Q: When should I worry about `rtol` and `atol`?**
A: Default values work for most problems. Tighten them (e.g., `rtol=1e-6`) when: (a) you are comparing to an analytical solution and need high accuracy, (b) the problem has sharp features like a sudden temperature rise, or (c) you're finding a steady state by integrating to large t.

**Q: `sol.success` is False — what do I do?**
A: First, print `sol.message` — it will tell you what went wrong. Common causes: (1) the ODE blows up to infinity (check your physics), (2) the tolerances are too tight for the problem (loosen `rtol`), (3) the equation is stiff (switch to `method='Radau'`).

**Q: Why does Euler still appear in textbooks if it's so bad?**
A: Because it is the clearest possible illustration of the core idea — and every more sophisticated method (RK4, RK45, Radau) is a direct improvement on Euler. Understanding Euler means understanding all numerical ODE solvers at the conceptual level.

---

## Exam and HW Tips

- If asked to "apply Euler's method," show two full steps by hand with numbers: compute the slope, multiply by h, add to the current value, advance time.
- If asked to "solve an ODE in Python," always show: the `def rhs(t, y):` function, the `solve_ivp` call with `t_span`, `y0`, and `t_eval`, and the `sol.y[0]` extraction.
- For a system of N ODEs: `y0` has N entries, `rhs` returns a list of N entries, `sol.y` has shape (N, n_timepoints).
- Always verify your function with a manual print call before running the solver.
