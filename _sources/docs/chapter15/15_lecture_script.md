# Chapter 15 Lecture Script: Numerical Integration and Differentiation

---

## Opening / Motivation

Alright, let's get started. Today we're on Chapter 15 — numerical integration and differentiation.

I want to start with two problems that I think will feel very familiar to you.

The first one: you're working with nitrogen gas and you need to figure out how much heat it takes to warm it up from 300 K to 1200 K. The formula is straightforward — it's just the integral of the heat capacity over temperature. But here's the catch: the heat capacity of nitrogen isn't constant. It's a function of temperature. NIST gives you a polynomial equation for it called the Shomate equation. You *could* integrate that polynomial by hand, but what if your C-p data comes from a table of experimental measurements? There's no nice formula anymore. You just have a list of numbers. How do you integrate that?

The second problem: you're running a batch reactor experiment. You're measuring concentration C-A at discrete time points — maybe every 10 seconds. The reaction rate is defined as minus dC-A over dt. But you don't have a formula for C-A as a function of time. You have a table. How do you compute a derivative from a table?

Both of these problems need **numerical** methods. And that's exactly what today is about. 

---

## 15.1 The Core Idea: Geometry First

Before we write a single line of code, I want to make sure the geometry is clear in your head, because everything we do today flows from these two pictures.

**Run the cell that shows the two plots.**

Look at the left panel. Integration is the area under a curve between two limits. That shaded region — that's your integral. It has units. If f of x has units of joules per kelvin and x is in kelvin, then the area has units of joules. That's your delta H.

Look at the right panel. Differentiation is the slope of the curve at a point. That red dashed line is the tangent at x equals 1. The slope of that tangent is the derivative. 

When you have a smooth analytic function, calculus gives you exact answers. When you have discrete data or a complicated expression, you use numerical approximations. That's where numpy and scipy come in.

---

## 15.2 Numerical Differentiation: `np.gradient`

### The Math

Let's think about what a derivative actually is. The definition is the limit of delta f over delta x as delta x goes to zero.

When you have data at discrete points — x-zero, x-one, x-two, all the way to x-n — you can't take that limit. Delta x is fixed by your experiment. You can't make it smaller. So instead, you approximate.

There are three formulas, and they correspond to three different situations.

**The central difference** is what you use at interior points. The formula is:

f-prime at x-i equals f at x-i-plus-one minus f at x-i-minus-one, all divided by x-i-plus-one minus x-i-minus-one.

Think about what this is doing geometrically. You're drawing a line between the point to the left and the point to the right, and using the slope of that line as your approximation for the derivative at x-i. You're using symmetric information on both sides. That symmetry is why this formula is better — it's second-order accurate, meaning the error shrinks like delta-x squared. Double your data density, error goes down by a factor of four.

**The forward difference** is what you use at the left endpoint, x-zero. You don't have a left neighbor at x-zero, so you can only look forward:

f-prime at x-zero is approximately f at x-one minus f at x-zero, divided by x-one minus x-zero.

This is only first-order accurate. The error shrinks like delta-x, not delta-x squared.

**The backward difference** is what you use at the right endpoint, x-n. Same idea — no right neighbor, so you look backward:

f-prime at x-n is approximately f at x-n minus f at x-n-minus-one, divided by x-n minus x-n-minus-one.

Also first-order accurate.

The key takeaway: your endpoints are slightly less accurate than your interior points. This is just a fundamental consequence of asymmetry.

**Run the three-panel visualization.**

Look at these three panels — green, blue, red. Each one shows a different formula in action. The colored points are the ones actually used in the calculation. The dashed line is the secant line whose slope approximates the derivative. Notice how the green panel at x-zero only uses x-zero and x-one. The blue panel at x-i uses both x-i-minus-one and x-i-plus-one — that's the symmetric pair. The red panel at x-n only uses x-n-minus-one and x-n.

This is *exactly* what `np.gradient` does under the hood. It automatically picks the right formula for each point and returns an array the same length as your input — one derivative value per data point.

### Syntax

The syntax is simple:

```python
dydx = np.gradient(y, x)
```

y is your array of function values. x is your array of x positions. Always pass x. If you write `np.gradient(y)` with no x, NumPy assumes the spacing between points is 1. If your x-axis is in seconds or kelvin, you'll get the wrong answer.

### Syntax Practice

**Point to the practice cell.**

let's just get the syntax into muscle memory. I have position data — a particle moving, sampled every half second. I want you to fill in the two question marks to compute the velocity at each time point.

Take 30 seconds and try it. What goes inside `np.gradient`?

...

Okay. The answer is `np.gradient(x, t)`. x is your data — the positions. t is where those measurements were taken — the times. The output, v, is the velocity at each point.

**Run the solution cell.**

Look at the output. v at index zero — that's 2.4. Let's check it manually: x-one minus x-zero is 1.2 minus 0.0 equals 1.2. t-one minus t-zero is 0.5. So 1.2 over 0.5 is 2.4. That matches — forward difference.

v at index two — 1.5. Check: x-three minus x-one is 2.7 minus 1.2 equals 1.5. t-three minus t-one is 1.0. So 1.5 over 1.0 is 1.5. That's the central difference, using the points two steps away from index two.

v at the last index — 0.6. Check: x-four minus x-three is 3.0 minus 2.7 equals 0.3. t-four minus t-three is 0.5. So 0.3 over 0.5 is 0.6. Backward difference.

Three formulas, one function call.

### 15.2.1 Verification on a Known Function

Now let's verify that `np.gradient` actually works correctly. The cleanest way to do this is to differentiate a function whose derivative we know analytically. Sine of x has derivative cosine of x.

**Run the sin/cos cell.**

Look at the right panel. The black line is the exact derivative — cosine. The red dashed line is what `np.gradient` gives us with 50 points. They're essentially on top of each other.

The max error is printed at the top. It's around 10 to the minus 3. The notebook explains why: with 50 points over zero to two pi, the spacing h is about 0.13. Central difference error goes like h-squared, so you'd expect error around 0.017. We're seeing something smaller because the function's third derivative is small near the middle of the interval — but that's in the right ballpark.

The point is: it works. And the error is controlled — more points, smaller error.

### 15.2.2 Reaction Rate from Concentration Data

Now the real application. Batch reactor. You've measured C-A at seven time points.

**Run the reactor cell.**

Look at the printed table. This is exactly what I described earlier. Seven time points, seven C-A values, seven reaction rate values — all from one call to `np.gradient`.

Look at the plots. Left: concentration is decaying exponentially, which makes sense for this kind of reaction. Right: the reaction rate is highest at t equals zero when there's the most reactant, and it drops as the reactor runs out of A. That's physically exactly right.

Notice that the rate at t equals 0 and t equals 60 seconds came from the forward and backward difference formulas. The other five used central difference. You can verify this from the printed numbers if you want — the notebook hints at how to check.

---

## 15.3 Numerical Integration of Discrete Data: Trapezoidal Rule

### The Math

Now let's flip to integration. Same situation: you have data at discrete points. You need the area under the curve.

The trapezoidal rule is the obvious thing to do: connect adjacent data points with straight lines and compute the area of each resulting trapezoid.

For two adjacent points — x-i, f-i and x-i-plus-one, f-i-plus-one — the trapezoid area is:

A-i equals x-i-plus-one minus x-i times f-i plus f-i-plus-one, divided by two.

Say that in words: it's the base times the average height. The base is the horizontal distance between the two x values. The height averages the function values at the two endpoints. That's a trapezoid.

To get the total integral, you just sum all these up from i equals zero to i equals n minus two.

For a uniform grid with spacing h, this simplifies to: h times the quantity f-zero over two plus f-one plus f-two plus dot dot dot plus f-n-minus-two plus f-n-minus-one over two. The endpoints get half weight because each endpoint belongs to only one trapezoid. Every interior point is shared between two trapezoids, so it gets full weight.

**Run the visualization cell.**

Look at the left panel. Each trapezoid is a different shade of blue. You can see how they tile the area under the curve. The labeled points x-zero through x-n on the axis show you exactly which data points were used. On the curve, f-zero through f-n mark the function values.

Now look at the right panel. I've zoomed into one trapezoid. The blue double-headed arrow is the base — x-i-plus-one minus x-i. The orange double-headed arrow is the average height — f-i plus f-i-plus-one divided by two. Multiply those together and you get the area of that one trapezoid. Do it for every pair of adjacent points and add them up — that's the full integral.

The accuracy here: trapezoidal rule is second-order accurate. Error goes like h-squared. Doubling your number of data points cuts the error by four.

### Syntax

```python
from scipy.integrate import trapezoid

area = trapezoid(y, x)
```

y is your function values. x is your x positions. Always pass x. Without it, NumPy assumes spacing of 1.

Returns a single float — the total area.

### 15.3.1 Visualizing Trapezoids on a Known Integral

**Run the pi cell.**

I'm integrating 4 over 1 plus x-squared from 0 to 1. The exact answer is pi — I'll spare you the substitution, but it's a standard arctangent integral.

With 5 points — that's a very coarse grid — we get 3.1317. Pi is 3.1416. Error of about 0.01. Look at the plot: you can see where the straight-line segments cut corners off the curved function. That gap between the curve and the straight lines is the truncation error.

If I crank up N to 50 or 100, those corners shrink and the error drops. This is the h-squared convergence — double the points, quarter the error.

### 15.3.2 Heat from Tabulated Cp Data

**Run the steam cell.**

Now the real thing. We have seven measured C-p values for steam at seven temperatures from 400 K to 1000 K. One line:

```python
delta_H = trapezoid(Cp_data, T_data)
```

That's it. We get delta-H in joules per mole. The plot shows the area we're computing — the shaded region under the C-p curve.

This is actually something you might do in a real process design. You have tabulated data from NIST or a textbook, you can't integrate it analytically, and `trapezoid` gives you the answer immediately.

---

## 15.4 Integration of a Python Function: `scipy.integrate.quad`

The trapezoidal rule works great when you have discrete data. But what if you have a Python function — something you can evaluate at any point? Then you can do much better.

`quad` uses adaptive quadrature. Instead of equally-spaced points, it figures out where the function is changing quickly and puts more points there, and fewer points where the function is smooth. Under the hood it's using Gaussian quadrature — the sample points are chosen to be the roots of Legendre polynomials, which is optimal. If there are regions where the error is too large, it splits that subinterval in half and tries again.

The result: near machine precision, around 14 significant digits, with far fewer function evaluations than a fine trapezoidal grid would need.

### Syntax

```python
from scipy.integrate import quad

result, error = quad(f, a, b)
```

f is a callable — a function or a lambda that takes a single float and returns a float. a and b are your limits. quad always returns two values: the integral and an error estimate. Always unpack both.

Important: do not write `result = quad(f, a, b)`. That assigns the entire tuple — a pair of numbers — to result. You'll get confusing bugs. Always unpack: `result, error = quad(f, a, b)`.

### 15.4.1 Basic Usage

**Run the x-squared cell.**

Integrate x-squared from 0 to 3. Exact answer is 9. We get 9.0000000000 to ten decimal places. Error bound is about 10 to the minus 13. That's machine precision — essentially as good as it's physically possible to be in floating point arithmetic.

### 15.4.2 Delta-H for N2

**Run the N2 cell.**

Here's the full motivation problem from the opening. The NIST Shomate equation for nitrogen. We want delta-H from 300 K to 1200 K.

With `quad`: we get the answer to essentially machine precision.
With `trapezoid` and 10 points: error of several joules.
With `trapezoid` and 100 points: error is tiny but still exists.

If you have the function, use `quad`. If you only have data, use `trapezoid`.

---

## 15.5 Cumulative Integration: `cumulative_trapezoid`

So far, `trapezoid` gives you one number — the total area. But sometimes you need the integral as a function. You want to know: what's the accumulated area up to every single point?

This is exactly `cumulative_trapezoid`. It applies the trapezoidal rule step by step and keeps track of the running total. The math is just:

F at x-zero is zero. F at x-one is F at x-zero plus the first trapezoid. F at x-two is F at x-one plus the second trapezoid. And so on. Each step adds one more trapezoid to the running sum.

### Syntax

```python
from scipy.integrate import cumulative_trapezoid

F = cumulative_trapezoid(y, x, initial=0)
```

The `initial=0` sets the starting value to zero, so the output has the same length as y. Without it, the output has one fewer element — which makes your arrays mismatched and causes headaches. Always use `initial=0`.

### Application: Enthalpy as a Function of Temperature

**Run the cumulative enthalpy cell.**

Left panel: C-p of N2 as a function of temperature. Right panel: the cumulative enthalpy change from 298 K — delta-H of T.

This curve answers a different question than a single integral. It's not "what's the total heat for one specific process?" It's "if I heat N2 to any arbitrary temperature, how much heat does it take?" That's the entire curve, all at once.

If you're doing process design and you want to know the energy requirement for many different temperature targets, this gives you everything in one shot.

At 1200 K, we can read off the value and check it against our earlier `quad` result — they should match.

---

## 15.6 Chemical Engineering Application: PFR Design

Let's put all of this together in a classic ChE problem.

### The Physics

A plug flow reactor — a PFR — is a tubular reactor. Fluid flows through, no mixing in the flow direction. Each fluid element sees the same history.

A mole balance on species A over a differential volume element gives you, after some algebra, the **PFR design equation**:

V equals F-A-zero times the integral from zero to X-f of dX over negative r-A of X.

This says: the reactor volume you need equals the molar feed rate times the area under the Levenspiel integrand from zero to your target conversion. The Levenspiel integrand is 1 over the reaction rate.

For our problem: liquid-phase second-order reaction, rate equals k times C-A-zero squared times one minus X squared. Substituting in:

V equals F-A-zero over k times C-A-zero squared, times the integral from zero to X-f of dX over one minus X squared.

This integral has an analytic solution — it gives X-f over one minus X-f. So the exact answer is:

V-exact equals F-A-zero over k times C-A-zero squared, times X-f over one minus X-f.

With our numbers: 5.0 over 0.05 times 4.0, times 0.8 over 0.2. That's 25 times 4, equals 100 liters. We'll use this to verify our numerical result.

### Part A: The Levenspiel Plot

**Run the Levenspiel plot cell.**

This is a standard plot in reactor design courses. The x-axis is conversion, the y-axis is 1 over negative r-A.

Notice what happens as X approaches 1. The rate goes to zero — you've consumed all your reactant. One over zero diverges to infinity. The curve shoots up.

Graphically, the reactor volume is F-A-zero times the shaded area under this curve from zero to X-f equals 0.8. The red dashed line marks our target conversion.

The physical insight: as you push toward higher and higher conversion, the area under the curve grows faster and faster. The last few percent of conversion are very expensive in terms of reactor volume. This is why 100% conversion is never economically practical in real reactors.

### Part B: Numerical Integration with `quad`

**Run the quad integration cell.**

One call to `quad` on the Levenspiel integrand:

```python
integral, err = quad(levenspiel, 0, X_f)
V_numerical = F_A0 * integral
```

We get 100.0 liters. The analytical answer is exactly 100 liters. Relative error is around 10 to the minus 13. Machine precision.

This is the workflow: define the integrand as a Python function, call `quad`, multiply by F-A-zero. Done.

### Part C: Reactor Sizing Chart

**Run the cumulative trapezoid cell.**

Instead of one target conversion, let's build the full design chart using `cumulative_trapezoid`. For every possible conversion from 0 to 95%, we compute the required volume.

This curve is your design space. The red lines mark X equals 0.8 and V equals 100 liters — confirming our result from Part B. But now you can also read off: what if your target was 70%? Just find that horizontal line and read the corresponding volume.

This is exactly how process engineers think about reactor design — not just one operating point, but the full curve so you can understand tradeoffs.

---

## Summary

Let's close with the decision tree.

**Do you have discrete data?**
- For derivatives: `np.gradient(y, x)`
- For integrals: `trapezoid(y, x)`
- For the running cumulative integral: `cumulative_trapezoid(y, x, initial=0)`

**Do you have a Python function?**
- Use `quad(f, a, b)` — it's far more accurate than trapezoid and you should always prefer it when you have the function available.

**Two things to never forget:**
1. Always pass x as the second argument to `np.gradient` and `trapezoid`. Without it, Python assumes unit spacing and you get the wrong answer with no error message.
2. Always unpack both return values from `quad`: `result, error = quad(f, a, b)`. If you write `result = quad(...)`, result is the tuple, not the number.

These four tools cover the vast majority of integration and differentiation problems you'll face in numerical work. The underlying math — finite differences, trapezoids, Gaussian quadrature — is good to understand, but the practical power is that you can trust these functions and focus on the engineering problem.

Any questions?
