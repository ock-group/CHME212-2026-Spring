# CHME 212 — Midterm Exam 1

**Time allowed: 30 minutes**

---

## Instructions

1. **Submission:** Enter all answers in the provided `submission_form.ipynb` notebook (Canvas-Files-Exam) and submit it through **Gradescope** before time is called. Late submissions will not be accepted.

2. **Electronic devices:** All personal electronic devices (phones, tablets, smartwatches) are **strictly prohibited**. Internet access and AI/code assistant tools (ChatGPT, GitHub Copilot, etc.) are **strictly prohibited**. Any violation will be treated as a violation of academic integrity.

3. **Open notes:** You may use printed or handwritten notes only.

4. **Exam structure:**
   - **Part A — Fix the Code**: Each snippet contains one bug. Write the corrected line(s) in the provided cell in `submission_form.ipynb`.
   - **Part B — Coding Problem**: Write your code in the provided cells in `submission_form.ipynb`.

5. **Time management:** Part B requires more time than Part A. It is recommended to complete Part A first (~10 min), then spend the remaining time on Part B (~20 min).

6. **Before submitting:** Restart the kernel and run all cells to confirm your notebook runs without errors.

---

## Part A — Fix the Code (20 points, 5 pts each)

Each snippet below contains **one bug**. Write the corrected line(s) in the provided cell in `submission_form.ipynb`.

---

**1.** The following code should print **"Number is smaller than 20"**, but it raises an error. Fix it.

```python
number = 15
if number < 20:
print("Number is smaller than 20")
```

---

**2.** The following code should print the **last element** of the list, but it raises an error. Fix it.

```python
my_list = [1, 2, 3, 4, 5]
print(my_list[5])
```

---

**3.** The following code should retrieve the temperature from the dictionary, but it prints the wrong value. Fix it.

```python
stream = {"name": "Feed", "T": 350, "P": 2.5, "phase": "vapor"}
# I want to retrieve the temperature value (350)
print(stream[1])
```

---

**4.** The following code should keep looping until `count` reaches 10, but it runs forever. Fix it.

```python
count = 0
while count < 10:
    print(count)
```



---

## Part B — Coding Problems (60 points)

---

The van der Waals equation gives the pressure of a real gas:

$$P = \frac{nRT}{V - nb} - \frac{n^2 a}{V^2}$$

You are given the following CO2 parameters and conditions:

- $n = 1$ mol, $R = 0.08206$ L·atm/(mol·K)
- $a = 3.640$ L²·atm/mol², $b = 0.04267$ L/mol
- Volume: $V = 1.5$ L (fixed)
- Temperatures to test: **[250, 300, 350, 400, 500]** K

Write a program that does **all** of the following. 

Note. Python functions are covered in Chapter 7, which is not included in Midterm 1 coverage, but you may use them if you feel more comfortable. You can also solve this problem without defining any functions.

1. **(40 pts)** Compute the van der Waals pressure for each temperature, classify each result as **"Low"** (P < 10 atm), **"Medium"** (10–20 atm), or **"High"** (> 20 atm), and save all results to a file named **vdw_results.txt** in the following format:
   ```
   T=250 K, P=xx.xxxx atm, Low
   T=300 K, P=xx.xxx atm, Medium
   ...
   ```

   > **Tip:** Use a `for` loop over the temperature list and a `with open(filename, 'w')` block to write the file.

2. **(20 pts)** For each temperature, compute the ideal gas pressure ($P_\text{ideal} = nRT/V$) alongside the van der Waals pressure, calculate the percent difference, and print the results in the following format:
   ```
   T = 250 K, P_ideal = xx.xxxx atm, P_vdW = xx.xxxx atm, diff = xx.xx%
   ```
   After the loop, write **one or two sentences** as a Python comment (`#`) discussing what the trend in percent difference tells you about when the ideal gas approximation breaks down.

   $$\% \text{ difference} = \frac{|P_\text{ideal} - P_\text{vdW}|}{P_\text{vdW}} \times 100$$

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
| Part A | Fix the Code (4 questions × 5 pts) | 20 pts |
| Part B | Coding Problem 1 | 60 pts |
| **Total** | | **80 pts** |


---

## Answer Key

### Part A

| Q  | Bug | Fix |
|----|-----|-----|
| 1  | Missing indentation on `print` | Indent `print("Number is smaller than 20")` by 4 spaces |
| 2  | `my_list[5]` — index out of range (valid indices: 0–4) | `print(my_list[4])` or `print(my_list[-1])` |
| 3  | `stream[1]` — dictionaries use keys, not integer positions | `print(stream["T"])` |
| 4  | `count` is never incremented — infinite loop | Add `    count += 1` inside the loop |

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

# Tasks 1
with open("vdw_results.txt", "w") as f:
    for T in temperatures:
        P = (n * R * T) / (V - n * b) - (n**2 * a) / V**2

        if P < 10:
            label = "Low"
        elif P <= 20:
            label = "Medium"
        else:
            label = "High"

        f.write(f"T={T} K, P={P:.4f} atm, {label}\n")

# Task 2
print()
for T in temperatures:
    P_vdw   = (n * R * T) / (V - n * b) - (n**2 * a) / V**2
    P_ideal = (n * R * T) / V
    diff = abs(P_ideal - P_vdw) / P_vdw * 100
    print(f"T = {T} K , P_ideal = {P_ideal:.4f} atm, P_vdW = {P_vdw:.4f} atm, diff = {diff:.2f}%")

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
