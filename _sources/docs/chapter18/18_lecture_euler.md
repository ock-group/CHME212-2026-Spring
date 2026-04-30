# Lecture Script: 18.2 Euler's Method
---

**Section:** 18.2  
**Estimated time:** 25 minutes  
**Prerequisites:** Students have just seen 18.1 — they know an ODE is dy/dt = f(t, y) with an initial condition y(t₀) = y₀, and that solving it means integrating forward from the starting point.

---

## Framing (2 min)

Open by connecting back to where 18.1 ended:

> "At the end of 18.1 we said: solving an ODE means evaluating this integral —
>
>     y(t) = y₀ + ∫ f(τ, y(τ)) dτ
>
> — but we can't do it directly because f depends on y, which is what we're trying to find. It's circular. So what do we do? We cheat — in a very controlled way. We pretend we know enough to take one small step forward, then use what we just found to take the next step, and repeat. That idea is Euler's method."

Write on the board and leave it there for the whole section:

```
The question: given dy/dt = f(t, y) and y₀, find y(t) for t > t₀.
The answer:   take small steps, each time using the current slope.
```

---

## The time grid (2 min)

> "The first thing Euler's method does is discretize time. Instead of a continuous curve, we're going to produce a sequence of values at specific time points."

Write on the board:

```
t₀,   t₁ = t₀ + h,   t₂ = t₀ + 2h,   t₃ = t₀ + 3h,   ...
```

> "The gap between consecutive time points is h — the **step size**. We choose h. A smaller h gives more points and more accuracy; a larger h gives fewer points but more error. We'll quantify that trade-off precisely in a moment."

> "Our goal is to find approximate values y₁, y₂, y₃, … that match the true solution y(t₁), y(t₂), y(t₃), … as closely as possible."

---

## The stepping formula — built from scratch (6 min)

> "Now for the key question: given that we're at (t₀, y₀), how do we estimate y₁?"

Write on the board:

```
We know:   current time  t₀
           current value y₀
           current slope f(t₀, y₀)   ← the ODE tells us this for free
```

> "The slope tells us the direction the curve is heading right now. The simplest possible thing to do is: follow that slope in a straight line for a time h."

Draw a small sketch on the board — a smooth curve, a point on it, and a straight tangent line stepping forward:

```
         /  ← exact curve curves upward
        /
       * ← y₁ (Euler estimate, below the true curve)
      /  ← tangent line: slope = f(t₀, y₀)
     *   ← y₀ (starting point, exact)
     |
     t₀  t₁
```

> "That straight-line step gives us:"

Write this clearly, one piece at a time:

```
rise = slope × run
     = f(t₀, y₀) × h

y₁ = y₀ + h · f(t₀, y₀)
```

> "That is the **Euler formula**. One line. We are adding the vertical rise of the tangent line to the current y value."

Now repeat the step from (t₁, y₁):

```
y₂ = y₁ + h · f(t₁, y₁)
```

> "Notice: we use y₁ — the approximate value we just computed — not the true y(t₁). The error carries forward. Each step inherits the mistakes of all previous steps. That's why errors accumulate."

Write the general pattern. This is the formula students should memorize:

```
y₁  =  y₀  +  h · f(t₀, y₀)
y₂  =  y₁  +  h · f(t₁, y₁)
y₃  =  y₂  +  h · f(t₂, y₂)
  ⋮
yₙ  =  yₙ₋₁ + h · f(tₙ₋₁, yₙ₋₁)
```

**Say this out loud and ask students to repeat it back:**

> "Each new value equals the previous value plus h times the slope at the previous point."

---

## Reading the visualization (4 min)

> "Now open the notebook and look at the figure for 18.2. The parameters are: ODE is dy/dt = k·y with k = 0.3, starting value y₀ = 0.5, step size h = 2. I made h deliberately large so all the labeled quantities are easy to see."

Walk through each labeled element one by one:

**The black curve** — the exact solution y(t) = 0.5 · e^(0.3t). This is what we are trying to approximate.

**The blue dot at y₀** — the starting point. This is exact — it's given by the initial condition.

**The blue arrow** — the first Euler step. It is a straight line with slope f(t₀, y₀) = k · y₀ = 0.3 × 0.5 = 0.15. It travels horizontally from t₀ to t₁ = t₀ + h.

**The gray horizontal bracket labeled h** — the step size, the horizontal distance of each arrow. Here h = 2 min.

**The blue vertical bracket labeled h·f(t₀, y₀)** — the vertical rise of the first step. This is the amount we added to y₀ to get y₁. Here: h · f(t₀, y₀) = 2 × 0.15 = 0.30, so y₁ = 0.5 + 0.30 = 0.80.

**The red square at t₁** — the exact value of the solution at t₁. The exact answer is y(2) = 0.5 · e^(0.6) ≈ 0.911.

**The red bracket labeled "error of y₁"** — the gap between where Euler landed (0.80) and where the exact solution is (0.911). Error = 0.111.

> "Why is there a gap? Because the true solution is not a straight line — it curves upward. Euler assumed a constant slope equal to the slope at the start of the step. But the slope is actually increasing throughout the step, since dy/dt = k·y and y is growing. Euler underestimates the rise."

**The orange arrow** — the second Euler step, starting from y₁ = 0.80 (already wrong). Slope at step 2 is f(t₁, y₁) = 0.3 × 0.80 = 0.24. Rise = 2 × 0.24 = 0.48. So y₂ = 0.80 + 0.48 = 1.28.

**The exact value at t₂** is y(4) = 0.5 · e^(1.2) ≈ 1.66. Error at step 2 = 0.38.

> "Notice the error grew — from 0.111 at step 1 to 0.38 at step 2. Two things are happening simultaneously: (1) each step introduces a new curvature error, and (2) the second step started from an already-wrong value. Errors pile up."

**Pose the question from the notebook:**

> "Looking at this picture — what would you do to reduce the error?"

Wait for responses. Students will say: make h smaller. Yes. Ask: *"How much smaller?"* Guide toward: half the step size, half the error. We'll verify that precisely.

---

## The printed table (2 min)

> "Below the figure the notebook prints a table. Let's read it together."

```
y0 = 0.5000   (exact: 0.5000)
y1 = 0.8000   (exact: 0.9111,  error = 0.1111)
y2 = 1.2800   (exact: 1.6620,  error = 0.3820)
```

> "Step 0 has zero error — it's the initial condition, which is given exactly."

> "Step 1 error is 0.111. Step 2 error is 0.382 — about 3.4 times larger, even though we only took one more step. That's because the curvature of an exponential grows as the function grows, so each step is worse than the last in absolute terms."

> "In relative terms: step 1 error / exact value = 0.111 / 0.911 = 12%. Step 2 error / exact value = 0.382 / 1.662 = 23%. Getting worse."

> "This is h = 2. Let's think about what happens when we make h smaller."

---

## Error analysis — how does error scale with h? (5 min)

> "We want to be precise about this. If I halve h from 2 to 1, how much better does my answer get?"

**Where does the error in one step come from?**

> "The exact value at t + h is given by the Taylor series:"

Write on the board:

```
y(t + h) = y(t) + h·y'(t) + (h²/2)·y''(t) + (h³/6)·y'''(t) + ...
                  ─────────  ──────────────────────────────────────
                  kept by      dropped by Euler
                  Euler
```

> "Euler keeps only the first two terms. Everything from h² onward is thrown away. The leading dropped term is (h²/2)·y''(t). That's the **local truncation error** — the mistake in a single step:"

```
e_local ≈ (h²/2) · y''(t)   ~  O(h²)
```

> "Halving h makes the local error 4 times smaller — because h² appears."

**But the global error over the whole interval is only O(h), not O(h²).** This is the key result.

> "Why? Over a total time T, we take N = T/h steps. Each step contributes roughly h²/2 · |y''| of error. The total accumulated error is approximately:"

Write the cancellation explicitly — make this unmissable:

```
e_global ≈  N  ×  e_local
          =  T/h  ×  (h²/2) · |y''|
          =  (T/2) · h · |y''|
          ~  O(h)
```

Draw a box around this:

```
┌──────────────────────────────────────────────────────────┐
│  (T/h) × h²  =  T·h                                     │
│                                                          │
│  One power of h from step count cancels one from error.  │
│  Net result: global error is O(h), not O(h²).            │
└──────────────────────────────────────────────────────────┘
```

> "This is what **first-order accuracy** means. Halve h → half the error. Tenth of h → tenth the error. Ten times the work for ten times the accuracy."

**Summarize in a table on the board:**

```
Quantity                        Order    Intuition
─────────────────────────────────────────────────────────
Local error per step            O(h²)    from Taylor: first dropped term ∝ h²
Number of steps over [0, T]     O(1/h)   halving h doubles the steps
Global error (accumulated)      O(h)     h² × (1/h) = h
```

**Numerical check to make it concrete:**

> "For our ODE dy/dt = k·y, we can differentiate to find y'' = k·y' = k²·y. So the local error at step 1 is exactly:"

```
e_local = (h²/2) · k² · y₀
         = (4/2) · (0.09) · 0.5
         = 0.09     (h = 2)
```

> "The actual error was 0.111. Close — the formula is for the leading term only, higher-order terms contribute the rest. But the scaling is confirmed."

---

## Effect of step size — the convergence plot (2 min)

> "The second notebook cell runs Euler with three different step sizes: h = 2.0, 1.0, 0.5. Look at the left panel."

> "As h decreases, the Euler trajectory gets closer to the exact curve. Each halving of h cuts the error roughly in half — consistent with O(h)."

> "The right panel is the **convergence plot**. x-axis is h (log scale), y-axis is the global error at t = 10 (log scale). The Euler errors fall along a straight line. The slope of that line on a log-log plot is 1 — the signature of first-order accuracy. The dashed reference line with slope 1 confirms this."

> "Read the convergence plot like this: every time h decreases by a factor of 10 (one unit left on the x-axis), the error decreases by a factor of 10 (one unit down on the y-axis). Straight diagonal line with slope 1."

---

## Why we don't use Euler in practice (1 min)

> "Euler has three problems, and all three matter:"

Write on the board:

```
Problem 1:  First-order accuracy only.
            Need 10× more steps for 10× better answer.
            Expensive.

Problem 2:  Fixed step size.
            Can't adapt when the solution changes fast in some regions
            and slowly in others.

Problem 3:  Single slope evaluation per step.
            Uses only f(tₙ, yₙ) — ignores how the slope changes
            within the interval [tₙ, tₙ₊₁].
```

> "All three of these are solved by `solve_ivp`, which we cover next. Euler was never meant to be the final answer — it was meant to build intuition. And it did: now you know exactly what every ODE solver is doing. It is always some version of: evaluate the slope(s), take a step, repeat."

---

## Worked example — two steps by hand (optional, if time allows)

> "Let me do two Euler steps completely by hand so you can see the arithmetic. This is exactly what an exam problem might ask."

**Given:** dy/dt = k·y, y(0) = 0.5, k = 0.3, h = 1.0.

**Step 0 → 1:**

```
t₀ = 0,    y₀ = 0.5
slope₀ = f(t₀, y₀) = k · y₀ = 0.3 × 0.5 = 0.15
y₁ = y₀ + h · slope₀ = 0.5 + 1.0 × 0.15 = 0.65
t₁ = 1.0
```

**Step 1 → 2:**

```
t₁ = 1.0,  y₁ = 0.65
slope₁ = f(t₁, y₁) = k · y₁ = 0.3 × 0.65 = 0.195
y₂ = y₁ + h · slope₁ = 0.65 + 1.0 × 0.195 = 0.845
t₂ = 2.0
```

**Check against exact:**

```
y(1) = 0.5 · e^(0.3) ≈ 0.675    error = |0.65 - 0.675| = 0.025
y(2) = 0.5 · e^(0.6) ≈ 0.911    error = |0.845 - 0.911| = 0.066
```

> "Notice: halving h from 2 to 1 at step 1 reduced the error from 0.111 to 0.025 — a factor of about 4.5. Not exactly 4, because the local error formula is only approximate, but in the right ballpark for O(h²) local error."

---

## Closing (1 min)

> "Euler's method in one sentence: at every time step, evaluate the slope from the ODE, multiply by h, add it to the current value, advance time. That's it."

Write the final boxed summary:

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│    yₙ = yₙ₋₁ + h · f(tₙ₋₁, yₙ₋₁)                  │
│                                                      │
│    Global error ~ O(h)    (first-order accuracy)     │
│    Halve h  →  half the error, twice the work        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

> "Next: `solve_ivp` does the same forward march, but with four slope evaluations per step instead of one, combined optimally. That single change takes the error from O(h) to O(h⁴) — halving h improves accuracy by 16× instead of 2×. Let's see how to use it."

---

## Common Student Mistakes

| Mistake | What happens | Fix |
|---------|-------------|-----|
| Using h too large | Large error, solution diverges from exact | Reduce h; check against smaller h |
| Forgetting to advance t | Infinite loop at t₀ | Always append both t and y |
| Using exact y instead of Euler y for the next slope | Not actually doing Euler | Use yₙ, not y(tₙ), for the slope |
| Confusing local and global error | Wrong expectations about accuracy | Local is O(h²); global is O(h) |

---

## Exam Checklist

When asked to "apply Euler's method" on an exam:

1. Write the formula: `yₙ = yₙ₋₁ + h · f(tₙ₋₁, yₙ₋₁)`
2. Identify f(t, y) from the given ODE
3. Plug in t₀, y₀, h and compute the slope: `slope = f(t₀, y₀)`
4. Compute y₁: `y₁ = y₀ + h · slope`
5. Advance time: `t₁ = t₀ + h`
6. Repeat from step 3 using t₁, y₁

Show each intermediate value explicitly — partial credit depends on it.
