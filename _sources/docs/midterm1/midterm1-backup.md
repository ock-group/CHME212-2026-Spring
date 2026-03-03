# CHME 212 — Midterm Exam 1

**Time allowed: 30 minutes**

---

## Instructions

1. **Submission:** Enter all answers in the provided `submission_form.ipynb` notebook and submit it through **Gradescope** before time is called. Late submissions will not be accepted.

2. **Electronic devices:** All personal electronic devices (phones, tablets, smartwatches) are **strictly prohibited**. Internet access and AI/code assistant tools (ChatGPT, GitHub Copilot, etc.) are **strictly prohibited**. Violation will result in a grade of zero.

3. **Open notes:** You may use printed or handwritten notes only.

4. **Exam structure:**
   - **Part A — Multiple Choice** (40 pts): 10 questions, 4 pts each. Enter your answer (`'a'`, `'b'`, `'c'`, or `'d'`) in the corresponding variable in `submission_form.ipynb`.
   - **Part B — Coding Problem** (60 pts): 1 problem with 5 tasks. Write your code in the provided cells in `submission_form.ipynb`.

5. **Time management:** Part B requires more time than Part A. It is recommended to complete Part A first (~10 min), then spend the remaining time on Part B (~20 min).

6. **Before submitting:** Restart the kernel and run all cells to confirm your notebook runs without errors.

---

## Part A — Multiple Choice (40 points, 4 pts each)

Circle the best answer.

---

**1.** What is the output of the following code?

```python
x = 7 // 2
print(x)
```

(a) 3.5

(b) 1

(c) 3

(d) Error

---

**2.** A process engineer stores the following data in Python:

```python
stream = {"name": "Feed", "T": 350, "P": 2.5, "phase": "vapor"}
```

Which line correctly retrieves the temperature?

(a) stream[1]

(b) stream["T"]

(c) stream.T

(d) stream.get(1)

---

**3.** What does the following expression evaluate to?

```python
T = 420
P = 1.8
safe = (T < 500) and (P < 2.0)
print(safe)
```

(a) False

(b) True

(c) 420

(d) None

---

**4.** A student writes this loop to screen a list of temperatures:

```python
temps = [280, 310, 450, 295, 510]
for T in temps:
    if T > 400:
        print("High temp:", T)
        break
```

How many lines are printed?

(a) 0

(b) 1

(c) 2

(d) 5

---

**5.** What is the output of the following code?

```python
values = [10, 20, 30, 40]
values.append(50)
print(values[-2])
```

(a) 40

(b) 50

(c) 30

(d) Error

---

**6.** A function is defined as:

```python
def classify(pH):
    if pH < 7:
        return "Acidic"
    elif pH == 7:
        return "Neutral"
    else:
        return "Basic"
```

What does **classify(3.5)** return?

(a) "Basic"

(b) "Neutral"

(c) "Acidic"

(d) None

---

**7.** Which of the following code snippets correctly read an **existing** file without overwriting it?

(a) with open("log.txt", "w") as f: f.write("new line\n")

(b) with open("log.txt", "r") as f: f.write("new line\n")

(c) with open("log.txt", "a") as f: f.write("new line\n")

(d) with open("log.txt") as f: f.write("new line\n")

---

**8.** What is the output of the following code?

```python
count = 0
while count < 4:
    count += 2
print(count)
```

(a) 2

(b) 3

(c) 4

(d) Runs forever (infinite loop)

---

**9.** A student has the following project directory structure:

```
project/
├── main.py
└── utils/
    ├── __init__.py
    └── thermo.py
```

The file **thermo.py** contains a function called **vdw_pressure**. Which import statement in **main.py** (current location) correctly imports that function?

(a) import thermo

(b) from utils import thermo.vdw_pressure

(c) from utils.thermo import vdw_pressure

(d) import project.utils.thermo

---

**10.** A student writes the following code to parse a sensor reading:

```python
reading = "N/A"

try:
    value = float(reading)
    print("Value:", value)
except ValueError:
    print("Bad reading")
```

What is printed?

(a) Value: N/A

(b) Bad reading

(c) Nothing

(d) Error

---

## Part B — Coding Problems (60 points)

Submit your notebook file (.ipynb) through Gradescope.

---

### Problem 1 — Van der Waals Equation of State (60 pts)

The van der Waals equation gives the pressure of a real gas:

$$P = \frac{nRT}{V - nb} - \frac{n^2 a}{V^2}$$

You are given the following CO2 parameters and conditions:

- $n = 1$ mol, $R = 0.08206$ L·atm/(mol·K)
- $a = 3.640$ L²·atm/mol², $b = 0.04267$ L/mol
- Volume: $V = 1.5$ L (fixed)
- Temperatures to test: **[250, 300, 350, 400, 500]** K

Write a program that does **all** of the following:

1. **(5 pts)** Define a function **vdw_pressure(n, T, V, a, b)** that computes and returns the van der Waals pressure.

2. **(15 pts)** Loop over the temperature list. For each temperature:
   - Call your function to compute the pressure
   - Print the temperature and pressure formatted as shown below
   - Classify the result as **"Low"** (P < 10 atm), **"Medium"** (10–20 atm), or **"High"** (> 20 atm)

3. **(15 pts)** Write a script that saves the results to a file named **vdw_results.txt** with one line per temperature in the specified format, preserving the same decimal places and data types.
   ```
   T=250 K, P=12.3456 atm
   ```

4. **(10 pts)** After the loop, print the temperature at which pressure was **highest**. You must **iterate** over the temperatures to determine the highest value, rather than selecting it manually. DO NOT convert the data into a NumPy array.

5. **(15 pts)** Add a second function **ideal_pressure(n, T, V)** that computes $P_\text{ideal} = nRT/V$. Loop over the temperatures again and for each one:
   - Compute both $P_\text{vdW}$ and $P_\text{ideal}$
   - Calculate the **percent difference**:

$$\% \text{ difference} = \frac{|P_\text{ideal} - P_\text{vdW}|}{P_\text{vdW}} \times 100$$

   - Print the result formatted as shown below
   - After the loop, write **one or two sentences** as a Python comment (`#`) stating whether the ideal gas approximation is reasonable under these conditions.

```python
n = 1
R = 0.08206
a = 3.640
b = 0.04267
V = 1.5
temperatures = [250, 300, 350, 400, 500]

# Your code here
```


---

## Score Summary

| Part | Description | Points |
|------|-------------|--------|
| Part A | Multiple Choice (8 questions × 5 pts) | 40 pts |
| Part B | Coding Problem 1 | 60 pts |
| **Total** | | **100 pts** |

**Part B breakdown:**

| Task | Description | Points |
|------|-------------|--------|
| 1 | Define `vdw_pressure()` function | 5 pts |
| 2 | Loop, compute, print, classify | 15 pts |
| 3 | Write results to file | 15 pts |
| 4 | Find and print highest-pressure temperature | 10 pts |
| 5 | Ideal gas comparison and percent difference | 15 pts |
| **Total** | | **60 pts** |

---

## Answer Key

### Part A

| Q  | Answer | Explanation |
|----|--------|-------------|
| 1  | **(c) 3** | `//` is floor division: 7 / 2 = 3.5 → floor = 3 |
| 2  | **(b) `stream["T"]`** | Dictionary values accessed by key string |
| 3  | **(b) True** | 420 < 500 is True AND 1.8 < 2.0 is True → True |
| 4  | **(b) 1** | Loop hits 450 > 400, prints once, then breaks |
| 5  | **(a) 40** | After append, list is `[10,20,30,40,50]`; index `-2` is `40` |
| 6  | **(c) "Acidic"** | 3.5 < 7, so the first branch is taken |
| 7  | **(d)** | Default open mode is `'r'` (read-only); `with open("log.txt") as f` reads without overwriting |
| 8  | **(c) 4** | count: 0 → 2 → 4; at 4 the condition `4 < 4` is False, loop exits |
| 9  | **(c)** | `from utils.thermo import vdw_pressure` follows the `package.module` path from `main.py`; `__init__.py` makes `utils` a package |
| 10 | **(b) "Bad reading"** | `float("N/A")` raises `ValueError`; except block prints "Bad reading" |

---

### Part B

**Problem 1** — Sample solution:

```python
n = 1
R = 0.08206
a = 3.640
b = 0.04267
V = 1.5
temperatures = [250, 300, 350, 400, 500]

# Tasks 1-4
def vdw_pressure(n, T, V, a, b):
    P = (n * R * T) / (V - n * b) - (n**2 * a) / V**2
    return P

max_P = -1
max_T = None

with open("vdw_results.txt", "w") as f:
    for T in temperatures:
        P = vdw_pressure(n, T, V, a, b)

        if P < 10:
            label = "Low"
        elif P <= 20:
            label = "Medium"
        else:
            label = "High"

        print(f"T = {T} K  ->  P = {P:.4f} atm  [{label}]")
        f.write(f"T={T} K, P={P:.4f} atm\n")

        if P > max_P:
            max_P = P
            max_T = T

print(f"Highest pressure at T = {max_T} K")

# Task 5
def ideal_pressure(n, T, V):
    return (n * R * T) / V
print()
for T in temperatures:
    P_vdw   = vdw_pressure(n, T, V, a, b)
    P_ideal = ideal_pressure(n, T, V)
    diff = abs(P_ideal - P_vdw) / P_vdw * 100
    print(f"T = {T} K  ->  P_ideal = {P_ideal:.4f} atm,  P_vdW = {P_vdw:.4f} atm,  diff = {diff:.2f}%")

# The ideal gas law overestimates pressure by ~3-10% at these conditions;
# the approximation becomes less reasonable at lower temperatures where
# intermolecular attractions (a term) are more significant relative to kT.
```

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {inlineMath: [['$', '$']]},
  messageStyle: "none"
});
</script>
