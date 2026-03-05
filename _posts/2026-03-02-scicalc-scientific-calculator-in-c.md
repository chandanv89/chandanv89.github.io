---
layout: post
title: "SciCalc — I Built a Full Scientific Calculator in C (and It Got Out of Hand)"
date: 2026-03-02
categories: [projects, c-programming, engineering]
tags:
  [
    c,
    calculator,
    scientific-calculator,
    recursive-descent-parser,
    engineering,
    college-project,
  ]
description: "The story of how a simple 'Hello World'-era C assignment spiraled into a 13-module, 158-test scientific calculator that rivals a TI-36X Pro. A deep dive into every feature, the math behind it, and what I learned along the way."
image: /assets/images/scicalc/scicalc-banner.png
math: true
toc: true
---

![SciCalc Banner — A terminal-based scientific calculator written in C](https://images.unsplash.com/photo-1587145820266-a5951ee6f620?w=1200&q=80 "Scientific calculators: now with 100% more terminal vibes")

## The Backstory: From printf to… This

Picture this: it's 2007. I'm a wide-eyed first-year Computer Science student at **[KIT, Tiptur](https://www.kittiptur.ac.in/)**. The air smells of freshly printed lab manuals and _samosas_ from the canteen. Our professor walks in and writes something on the board that would change my life forever:

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

I was _hooked_. Something about making a machine obey your commands felt like wielding a spell. Within a week, I'd gone from `Hello World` to a simple calculator that could add two numbers. Within a month… well, let's just say things escalated.

> **What started as a humble `a + b` calculator mutated into a 13-module, 158-test, recursive-descent-parsing beast of a scientific calculator.**

Fast forward to 2026. I'm a full-time **Senior Java Developer** now — my days are spent wrangling Spring Boot microservices, arguing about pull requests, and writing enough Confluence pages to wallpaper a house. C feels like a distant memory, a language I once dreamed in but haven't spoken fluently in years. I miss it. I miss the raw, close-to-the-metal honesty of it. No frameworks. No annotations. Just you, a pointer, and a prayer.

So when I recently discovered this project while cleaning out my Google Drive and old backups — buried under folders of lab records and assignment PDFs — my heart genuinely skipped a beat. There it was: **SciCalc**.

And here's the part that _floored_ me: I unzipped the archive, ran `make`, and **it compiled. First try. Zero errors. Zero warnings. Not a single line of code changed.** Nearly two decades later, and `gcc` just… built it. C89/90 (GNU89) rocks! That's the beauty of a language with a stable standard and no runtime magic. I sat there staring at the terminal, equal parts stunned and giddy.

That moment — that rush of nostalgia, that thrill of reliving my college days — is exactly why I'm writing this blog post. Call it a trip down memory lane. Call it rekindling an old flame with a programming language. Past-me went _absolutely feral_ with this thing, and present-me is here to tell the story. 🤓

---

## What Is SciCalc?

**SciCalc** is a fully-featured, modular scientific calculator written in **C11**<sup>^[1](#footnotes)</sup>. It's designed for engineering, science, and mathematics — think [TI-36X Pro](https://en.wikipedia.org/wiki/TI-36) or [Casio fx-991EX](https://en.wikipedia.org/wiki/Casio_fx-991EX), but it runs in your terminal and was written by a college kid who clearly had too much free time.

Here are the numbers that make me simultaneously proud and concerned about my younger self:

| Metric               | Value    |
| -------------------- | -------- |
| Test cases           | **158**  |
| Compiler warnings    | **Zero** |
| Modules              | **13**   |
| C Standard           | **C11**  |
| Built-in functions   | **43+**  |
| Physical constants   | **12**   |
| Lines of parser code | **~540** |

![Terminal showing a calculator REPL](https://images.unsplash.com/photo-1629654297299-c8506221ca97?w=800&q=80 "The natural habitat of SciCalc: a terminal window")

---

## The Architecture: How It's Built

Before we dive into the features, let's appreciate the structure. For a first-year student, I'm retroactively impressed (and also a little suspicious) that I organized it this cleanly:

```
SciCalc/
├── Makefile              # GNU Make build system
├── include/              # 13 header files — one per module
│   ├── calculator.h      # Master header that aggregates everything
│   ├── constants.h       # Math & physics constants
│   ├── arithmetic.h      # Basic math, roots, GCD/LCM
│   ├── trigonometry.h    # Trig, inverse trig, hyperbolic
│   ├── logarithmic.h     # Logs & exponentials
│   ├── combinatorics.h   # Factorials, permutations, Fibonacci
│   ├── statistics.h      # Descriptive stats, regression, distributions
│   ├── matrix.h          # Matrix algebra (up to 10×10)
│   ├── complex_num.h     # Complex number arithmetic
│   ├── conversion.h      # Unit & base conversions
│   ├── memory.h          # Memory slots, variables, history
│   ├── parser.h          # Expression parser
│   └── utils.h           # String utilities, formatting
├── src/                  # Implementation files (.c)
└── tests/                # 158 automated tests
```

Huge thanks to magazines like **Linux For You** (now, _[OpenSource For You](https://magazine.opensourceforu.com/)_) that introduced me to these best practices early on. Looking back now, it's definitely worth spending my pocket money on LFY ;)

### Design Principles (That I Apparently Understood at 18)

- **Modularity**: Each mathematical domain lives in its own header/implementation pair. [Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) before I even knew the term.
- **No dynamic allocation**: Everything uses fixed-size arrays. This thing could theoretically run on an [embedded system](https://en.wikipedia.org/wiki/Embedded_system).
- **Strict compilation**: `-Wall -Wextra -Wpedantic` with zero warnings. Past-me was ruthless.
- **Error handling via status codes**: Every function that can fail returns a status code (`CALC_SUCCESS`, `CALC_ERR_DIV_ZERO`, `CALC_ERR_DOMAIN`, etc.). No silent failures.

---

## Feature Deep Dive

Now let's get to the good stuff. Buckle up — there are _a lot_ of features.

### 1. The Expression Parser

This is the crown jewel. I wrote a **hand-rolled [recursive-descent parser](https://en.wikipedia.org/wiki/Recursive_descent_parser)** that correctly handles [operator precedence](https://en.wikipedia.org/wiki/Order_of_operations). No `eval()` cheating. No libraries. Just pure, hand-crafted grammar parsing.

| Precedence  | Operators                                                 | Associativity |
| :---------: | --------------------------------------------------------- | :-----------: |
| 1 (lowest)  | `+` `-`                                                   |     Left      |
|      2      | `*` `/` `%`                                               |     Left      |
|      2      | Implicit multiplication: `2pi`, `3(4+5)`                  |     Left      |
|      3      | `^` (exponentiation)                                      |   **Right**   |
|      4      | Unary `+` `-`                                             |     Right     |
|      5      | Postfix `!` (factorial)                                   |     Left      |
| 6 (highest) | `()`, $\lvert\ldots\rvert$, functions, numbers, constants |       —       |

**What makes it cool:**

- **[Scientific notation](https://en.wikipedia.org/wiki/Scientific_notation)**: `6.022e23` just works. Avogadro would be proud.
- **Implicit multiplication**: Type `2pi` and get $2\pi \approx 6.283$. Type `3(4+5)` and get $27$. The parser figures it out.
- **Absolute value bars**: `|-7|` returns `7`. Nested absolute values work too.
- **30+ single-argument functions** and **13 two-argument functions** recognized by the parser.
- **Descriptive error messages**: Tells you _where_ in your expression things went wrong.

```
[RAD]> 2 + 3 * 4
  = 14

[RAD]> 2pi
  = 6.28318530718

[RAD]> |-42.5|
  = 42.5

[RAD]> 3(4+5)^2
  = 243
```

> **Further reading**: [Recursive Descent Parsing (Wikipedia)](https://en.wikipedia.org/wiki/Recursive_descent_parser) · [Crafting Interpreters — Parsing Expressions](https://craftinginterpreters.com/parsing-expressions.html)

---

### 2. Arithmetic Operations

The foundation. Every calculator needs these, but SciCalc goes beyond the basics with number-theoretic functions:

| Function       | Syntax                    | Example       | Result   |
| -------------- | ------------------------- | ------------- | -------- |
| Addition       | `a + b`                   | `17 + 25`     | `42`     |
| Subtraction    | `a - b`                   | `100 - 58`    | `42`     |
| Multiplication | `a * b`                   | `6 * 7`       | `42`     |
| Division       | `a / b`                   | `84 / 2`      | `42`     |
| Modulo         | `a % b` or `mod(a, b)`    | `17 % 5`      | `2`      |
| Power          | `a ^ b` or `pow(a, b)`    | `2 ^ 10`      | `1024`   |
| Square root    | `sqrt(x)`                 | `sqrt(144)`   | `12`     |
| Cube root      | `cbrt(x)`                 | `cbrt(27)`    | `3`      |
| Nth root       | `root(x, n)`              | `root(81, 4)` | `3`      |
| Absolute value | `abs(x)` or `\|x\|`       | `abs(-42)`    | `42`     |
| Sign           | `sign(x)`                 | `sign(-7)`    | `-1`     |
| Ceiling        | `ceil(x)`                 | `ceil(3.2)`   | `4`      |
| Floor          | `floor(x)`                | `floor(3.8)`  | `3`      |
| Round          | `round(x)`                | `round(3.5)`  | `4`      |
| Truncate       | `trunc(x)`                | `trunc(-3.7)` | `-3`     |
| GCD            | `gcd(a, b)`               | `gcd(84, 36)` | `12`     |
| LCM            | `lcm(a, b)`               | `lcm(12, 18)` | `36`     |
| Min / Max      | `min(a, b)` / `max(a, b)` | `max(42, 17)` | `42`     |
| Primality test | `prime N` (command)       | `prime 97`    | `Prime!` |

The **GCD** uses the classic [Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm) — one of the oldest algorithms still in use today (circa 300 BC, thanks Euclid). The **primality test** uses [trial division](https://en.wikipedia.org/wiki/Trial_division) up to $\sqrt{n}$.

```
[RAD]> gcd(84, 36)
  = 12

[RAD]> sqrt(2) ^ 2
  = 2

[RAD]> root(32, 5)
  = 2
```

> **Further reading**: [Euclidean Algorithm (Wikipedia)](https://en.wikipedia.org/wiki/Euclidean_algorithm) · [Modular Arithmetic (Khan Academy)](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/what-is-modular-arithmetic)

---

### 3. Trigonometric Functions

A full suite of [trigonometric](https://en.wikipedia.org/wiki/Trigonometric_functions), [inverse trigonometric](https://en.wikipedia.org/wiki/Inverse_trigonometric_functions), and [hyperbolic](https://en.wikipedia.org/wiki/Hyperbolic_functions) functions. Because what's a scientific calculator without `sin`?

**Standard trig:**

| Function  | Syntax   | Domain                        |
| --------- | -------- | ----------------------------- |
| Sine      | `sin(x)` | All real numbers              |
| Cosine    | `cos(x)` | All real numbers              |
| Tangent   | `tan(x)` | $x \neq \frac{\pi}{2} + n\pi$ |
| Cosecant  | `csc(x)` | $x \neq n\pi$                 |
| Secant    | `sec(x)` | $x \neq \frac{\pi}{2} + n\pi$ |
| Cotangent | `cot(x)` | $x \neq n\pi$                 |

**Inverse trig:**

| Function           | Syntax        | Domain    | Range                             |
| ------------------ | ------------- | --------- | --------------------------------- |
| Arcsine            | `asin(x)`     | $[-1, 1]$ | $[-\frac{\pi}{2}, \frac{\pi}{2}]$ |
| Arccosine          | `acos(x)`     | $[-1, 1]$ | $[0, \pi]$                        |
| Arctangent         | `atan(x)`     | All real  | $(-\frac{\pi}{2}, \frac{\pi}{2})$ |
| Two-arg arctangent | `atan2(y, x)` | All real  | $(-\pi, \pi]$                     |

**Hyperbolic functions:**

| Function           | Syntax    | Inverse                    |
| ------------------ | --------- | -------------------------- |
| Hyperbolic sine    | `sinh(x)` | `asinh(x)`                 |
| Hyperbolic cosine  | `cosh(x)` | `acosh(x)` ($x \ge 1$)     |
| Hyperbolic tangent | `tanh(x)` | `atanh(x)` ($\|x\| \lt 1$) |

**Three angle modes** — switchable at any time during a session:

| Mode                                                      | Full Circle | Command |
| --------------------------------------------------------- | ----------- | ------- |
| [Radians](https://en.wikipedia.org/wiki/Radian) (default) | $2\pi$      | `rad`   |
| [Degrees](<https://en.wikipedia.org/wiki/Degree_(angle)>) | 360°        | `deg`   |
| [Gradians](https://en.wikipedia.org/wiki/Gradian)         | 400 grad    | `grad`  |

```
[RAD]> sin(pi/6)
  = 0.5

[RAD]> deg
  Angle mode: DEG

[DEG]> sin(30)
  = 0.5

[DEG]> asin(0.5)
  = 30

[RAD]> cosh(0)
  = 1
```

Fun fact: `sin(pi/4)` and `sqrt(2)/2` give the same answer. SciCalc proves it:

```
[RAD]> sin(pi/4) - sqrt(2)/2
  = 0
```

> **Further reading**: [Trigonometric Functions (Wikipedia)](https://en.wikipedia.org/wiki/Trigonometric_functions) · [Hyperbolic Functions (Brilliant)](https://brilliant.org/wiki/hyperbolic-functions/) · [Unit Circle (Khan Academy)](https://www.khanacademy.org/math/algebra2/x2ec2f6f830c9fb89:trig/x2ec2f6f830c9fb89:unit-circle/v/unit-circle-definition-of-trig-functions-1)

---

### 4. Logarithmic & Exponential Functions

[Logarithms](https://en.wikipedia.org/wiki/Logarithm) and [exponentials](https://en.wikipedia.org/wiki/Exponential_function) — essential for everything from earthquake magnitudes to pH scales to compound interest.

| Function               | Syntax                 | Example        | Result         |
| ---------------------- | ---------------------- | -------------- | -------------- |
| Natural log (base $e$) | `ln(x)`                | `ln(e)`        | `1`            |
| Common log (base 10)   | `log(x)` or `log10(x)` | `log(1000)`    | `3`            |
| Binary log (base 2)    | `log2(x)`              | `log2(1024)`   | `10`           |
| Arbitrary-base log     | `logb(x, base)`        | `logb(81, 3)`  | `4`            |
| Exponential            | `exp(x)`               | `exp(1)`       | `2.71828...`   |
| Base-2 exponential     | `exp2(x)`              | `exp2(10)`     | `1024`         |
| Base-10 exponential    | `exp10(x)`             | `exp10(3)`     | `1000`         |
| $e^x - 1$ (stable)     | `expm1(x)`             | `expm1(0.001)` | `0.0010005...` |
| $\ln(1 + x)$ (stable)  | `log1p(x)`             | `log1p(0.001)` | `0.0009995...` |

The `expm1` and `log1p` functions are especially interesting — they provide **numerically stable** computation for values close to zero, avoiding [catastrophic cancellation](https://en.wikipedia.org/wiki/Catastrophic_cancellation). This is the kind of detail that separates a toy calculator from a real one.

```
[RAD]> log(1000)
  = 3

[RAD]> log2(65536)
  = 16

[RAD]> logb(81, 3)
  = 4

[RAD]> exp(1)
  = 2.71828182846
```

> **Further reading**: [Logarithm (Wikipedia)](https://en.wikipedia.org/wiki/Logarithm) · [Why We Need log1p and expm1](https://nhigham.com/2023/01/10/on-log1p-and-expm1-functions/) · [Change of Base Formula](https://en.wikipedia.org/wiki/Logarithm#Change_of_base)

---

### 5. Combinatorics & Number Theory

[Combinatorics](https://en.wikipedia.org/wiki/Combinatorics) — the mathematics of counting. This module handles everything from "how many ways can you arrange a deck of cards" to the Fibonacci sequence.

| Function       | Syntax                      | Example      | Result        |
| -------------- | --------------------------- | ------------ | ------------- |
| Factorial      | `n!` or `factorial(n)`      | `5!`         | `120`         |
| Gamma function | `gamma(x)`                  | `gamma(5)`   | `24` ($= 4!$) |
| Permutation    | `perm(n, r)` or `nPr(n, r)` | `perm(5, 3)` | `60`          |
| Combination    | `comb(n, r)` or `nCr(n, r)` | `nCr(52, 5)` | `2598960`     |
| Fibonacci      | `fib(n)`                    | `fib(10)`    | `55`          |
| Summation      | `sum(n)` (menu)             | `sum(100)`   | `5050`        |

The **poker hand calculation** is a fun one: $\binom{52}{5} = 2{,}598{,}960$ possible five-card hands from a standard deck. That's also why poker is interesting — the probability space is enormous.

The **[Gamma function](https://en.wikipedia.org/wiki/Gamma_function)** ($\Gamma(x)$) is implemented using the **[Lanczos approximation](https://en.wikipedia.org/wiki/Lanczos_approximation)** with [Euler's reflection formula](https://en.wikipedia.org/wiki/Reflection_formula) for negative arguments: $\Gamma(x)\Gamma(1-x) = \frac{\pi}{\sin(\pi x)}$. This is genuinely advanced numerical computing.

```
[RAD]> 5!
  = 120

[RAD]> nCr(52, 5)
  = 2598960

[RAD]> gamma(5)
  = 24

[RAD]> fib(10)
  = 55
```

> **Further reading**: [Factorial (Wikipedia)](https://en.wikipedia.org/wiki/Factorial) · [Gamma Function (Wolfram MathWorld)](https://mathworld.wolfram.com/GammaFunction.html) · [Fibonacci Sequence (OEIS)](https://oeis.org/A000045)

---

### 6. Statistics & Data Analysis

A comprehensive statistics module accessible via the interactive `stats` menu. Enter a dataset and get a complete statistical profile.

**Descriptive Statistics:**

| Statistic                                                                                              | Description                                                                     | Formula                                   |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- | ----------------------------------------- |
| [Arithmetic Mean](https://en.wikipedia.org/wiki/Arithmetic_mean)                                       | Average of all values                                                           | $\mu = \frac{\sum x_i}{n}$                |
| [Median](https://en.wikipedia.org/wiki/Median)                                                         | Middle value (sorted)                                                           | —                                         |
| [Mode](<https://en.wikipedia.org/wiki/Mode_(statistics)>)                                              | Most frequent value                                                             | —                                         |
| [Range](<https://en.wikipedia.org/wiki/Range_(statistics)>)                                            | $\max - \min$                                                                   | —                                         |
| [Population Variance](https://en.wikipedia.org/wiki/Variance)                                          | Spread of data                                                                  | $\sigma^2 = \frac{\sum(x_i - \mu)^2}{n}$  |
| [Sample Variance](https://en.wikipedia.org/wiki/Variance#Sample_variance)                              | With [Bessel's correction](https://en.wikipedia.org/wiki/Bessel%27s_correction) | $s^2 = \frac{\sum(x_i - \bar{x})^2}{n-1}$ |
| [Population Std Dev](https://en.wikipedia.org/wiki/Standard_deviation)                                 | $\sigma = \sqrt{\sigma^2}$                                                      | —                                         |
| [Sample Std Dev](https://en.wikipedia.org/wiki/Standard_deviation#Corrected_sample_standard_deviation) | $s = \sqrt{s^2}$                                                                | —                                         |

**Regression & Correlation:**

| Feature                                                                                    | Description                                            |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| [Pearson Correlation](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient)       | $r \in [-1, 1]$ — measures linear correlation strength |
| [Simple Linear Regression](https://en.wikipedia.org/wiki/Simple_linear_regression)         | Best-fit line $y = mx + b$                             |
| [Coefficient of Determination](https://en.wikipedia.org/wiki/Coefficient_of_determination) | $R^2$ — goodness of fit                                |

**Probability Distributions:**

| Function                                                                                         | Description                                                                                |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| [Normal PDF](https://en.wikipedia.org/wiki/Normal_distribution)                                  | Gaussian probability density $\varphi(x; \mu, \sigma)$                                     |
| [Normal CDF](https://en.wikipedia.org/wiki/Normal_distribution#Cumulative_distribution_function) | Cumulative distribution via [error function](https://en.wikipedia.org/wiki/Error_function) |

The dataset size supports up to **1,000 data points**, which is plenty for classroom exercises and many real-world applications.

> **Further reading**: [Descriptive Statistics (Khan Academy)](https://www.khanacademy.org/math/statistics-probability/summarizing-quantitative-data) · [Linear Regression (Stat Trek)](https://stattrek.com/regression/linear-regression) · [Normal Distribution (3Blue1Brown)](https://www.youtube.com/watch?v=zeJD6dqJ5lo)

---

### 7. Matrix Operations

Full [matrix algebra](<https://en.wikipedia.org/wiki/Matrix_(mathematics)>) for systems up to **10 × 10**, accessible via the interactive `matrix` menu:

| Operation                                                                  | Method                                                                                                         | Syntax (Menu-driven)          |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| Addition / Subtraction                                                     | Element-wise                                                                                                   | Enter two matrices            |
| Multiplication                                                             | Standard $O(n^3)$                                                                                              | Enter two compatible matrices |
| Scalar Multiplication                                                      | $cA$                                                                                                           | Enter matrix and scalar       |
| [Transpose](https://en.wikipedia.org/wiki/Transpose)                       | $A^T$                                                                                                          | Enter a matrix                |
| [Trace](<https://en.wikipedia.org/wiki/Trace_(linear_algebra)>)            | $\sum a_{ii}$                                                                                                  | Sum of diagonal elements      |
| [Determinant](https://en.wikipedia.org/wiki/Determinant)                   | [LU decomposition](https://en.wikipedia.org/wiki/LU_decomposition) with partial pivoting                       | Enter a square matrix         |
| [Inverse](https://en.wikipedia.org/wiki/Invertible_matrix)                 | [Gauss–Jordan elimination](https://en.wikipedia.org/wiki/Gaussian_elimination#Finding_the_inverse_of_a_matrix) | Enter a non-singular matrix   |
| [Rank](<https://en.wikipedia.org/wiki/Rank_(linear_algebra)>)              | [Row echelon form](https://en.wikipedia.org/wiki/Row_echelon_form)                                             | Enter any matrix              |
| [Solve $Ax = b$](https://en.wikipedia.org/wiki/System_of_linear_equations) | [Gaussian elimination](https://en.wikipedia.org/wiki/Gaussian_elimination) with back-substitution              | Enter $A$ and $b$             |
| [Identity Matrix](https://en.wikipedia.org/wiki/Identity_matrix)           | $I_n$                                                                                                          | Specify dimension             |

The determinant computation uses **LU decomposition with [partial pivoting](https://en.wikipedia.org/wiki/Pivot_element#Partial_and_complete_pivoting)** for numerical stability — this is the real deal, not the naive cofactor expansion that blows up for large matrices.

Example: solving a system of linear equations $Ax = b$:

$$
\begin{bmatrix} 2 & 1 \\ 5 & 3 \end{bmatrix} \begin{bmatrix} x_1 \\ x_2 \end{bmatrix} = \begin{bmatrix} 4 \\ 7 \end{bmatrix}
$$

The calculator applies Gaussian elimination to find $x_1 = 5$, $x_2 = -6$.

> **Further reading**: [Matrix Algebra (MIT OpenCourseWare)](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/) · [LU Decomposition (Wikipedia)](https://en.wikipedia.org/wiki/LU_decomposition) · [3Blue1Brown: Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)

---

### 8. Complex Number Arithmetic

Full [complex number](https://en.wikipedia.org/wiki/Complex_number) support accessible via the `complex` menu:

| Operation                                                                       | Description                                                                                                      |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Create                                                                          | Rectangular $(a + bi)$ and [polar form](https://en.wikipedia.org/wiki/Polar_coordinate_system) $(r\angle\theta)$ |
| Arithmetic                                                                      | $+$, $-$, $\times$, $\div$                                                                                       |
| [Magnitude](https://en.wikipedia.org/wiki/Absolute_value#Complex_numbers)       | $\|z\| = \sqrt{a^2 + b^2}$                                                                                       |
| [Phase / Argument](<https://en.wikipedia.org/wiki/Argument_(complex_analysis)>) | $\arg(z) = \text{atan2}(b, a)$                                                                                   |
| [Conjugate](https://en.wikipedia.org/wiki/Complex_conjugate)                    | $z^* = a - bi$                                                                                                   |
| Complex Power                                                                   | $z^n$                                                                                                            |
| Complex Square Root                                                             | Principal $\sqrt{z}$                                                                                             |
| [Complex Exponential](https://en.wikipedia.org/wiki/Euler%27s_formula)          | $e^z = e^a(\cos b + i \sin b)$ — [Euler's formula](https://en.wikipedia.org/wiki/Euler%27s_formula)!             |
| [Complex Logarithm](https://en.wikipedia.org/wiki/Complex_logarithm)            | $\ln(z) = \ln\|z\| + i\arg(z)$                                                                                   |

The complex exponential is arguably the most beautiful equation in mathematics, connecting $e$, $i$, $\pi$, 1, and 0 in [Euler's identity](https://en.wikipedia.org/wiki/Euler%27s_identity):

$$e^{i\pi} + 1 = 0$$

Now you can verify it in SciCalc.

> **Further reading**: [Complex Numbers (Brilliant)](https://brilliant.org/wiki/complex-numbers/) · [Euler's Formula (3Blue1Brown)](https://www.youtube.com/watch?v=mvmuCPvRoWQ) · [Visual Complex Analysis (Needham)](https://en.wikipedia.org/wiki/Visual_Complex_Analysis)

---

### 9. Unit Conversions

Practical engineering [unit conversions](https://en.wikipedia.org/wiki/Conversion_of_units) accessible via the `convert` menu:

**Temperature** ([conversion formulas](https://en.wikipedia.org/wiki/Conversion_of_scales_of_temperature)):

| From → To            | Formula                           |
| -------------------- | --------------------------------- |
| Celsius → Fahrenheit | $F = C \times \frac{9}{5} + 32$   |
| Fahrenheit → Celsius | $C = (F - 32) \times \frac{5}{9}$ |
| Celsius → Kelvin     | $K = C + 273.15$                  |
| Kelvin → Celsius     | $C = K - 273.15$                  |

**Length:**

| Conversion           | Factor                               |
| -------------------- | ------------------------------------ |
| Meters ↔ Feet        | $1 \text{ m} = 3.28084 \text{ ft}$   |
| Kilometres ↔ Miles   | $1 \text{ km} = 0.621371 \text{ mi}$ |
| Inches ↔ Centimetres | $1 \text{ in} = 2.54 \text{ cm}$     |

**Mass:** Kilograms ↔ Pounds ($1 \text{ kg} = 2.20462 \text{ lbs}$)

**Energy:**

| Conversion              | Factor                                           |
| ----------------------- | ------------------------------------------------ |
| Joules ↔ Calories       | $1 \text{ cal} = 4.184 \text{ J}$                |
| Electron-volts ↔ Joules | $1 \text{ eV} = 1.602 \times 10^{-19} \text{ J}$ |

**Pressure:**

| Conversion            | Factor                                  |
| --------------------- | --------------------------------------- |
| Pascals ↔ Atmospheres | $1 \text{ atm} = 101{,}325 \text{ Pa}$  |
| Pascals ↔ PSI         | $1 \text{ PSI} = 6{,}894.76 \text{ Pa}$ |

**Angle:** Degrees ↔ Radians ↔ Gradians

> **Further reading**: [SI Units (NIST)](https://www.nist.gov/pml/owm/metric-si/si-units) · [Conversion of Units (Wikipedia)](https://en.wikipedia.org/wiki/Conversion_of_units)

---

### 10. Number Base Conversions

[Positional numeral system](https://en.wikipedia.org/wiki/Positional_notation) conversions via the `base` menu — essential for every CS student:

| Base | Name                                                     | Prefix | Use Case                                  |
| ---- | -------------------------------------------------------- | ------ | ----------------------------------------- |
| 2    | [Binary](https://en.wikipedia.org/wiki/Binary_number)    | `0b`   | Digital electronics, bit manipulation     |
| 8    | [Octal](https://en.wikipedia.org/wiki/Octal)             | `0o`   | Unix file permissions (`chmod 755`)       |
| 10   | [Decimal](https://en.wikipedia.org/wiki/Decimal)         | —      | Human-readable numbers                    |
| 16   | [Hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) | `0x`   | Memory addresses, color codes (`#FF5733`) |

Bidirectional conversion: decimal → binary/octal/hex and back.

```
Decimal 255 in different bases:
  Binary:      11111111
  Octal:       377
  Hexadecimal: FF
```

> **Further reading**: [Number Bases (Khan Academy)](https://www.khanacademy.org/math/algebra-home/alg-intro-to-algebra/algebra-alternate-number-bases/v/number-systems-introduction) · [Hexadecimal (Wikipedia)](https://en.wikipedia.org/wiki/Hexadecimal)

---

### 11. Physical & Mathematical Constants

Built-in constants usable directly in expressions — because nobody wants to type `3.14159265358979` every time:

**Mathematical Constants (usable in expressions):**

| Constant                                                                    | Symbol  | Value                    |
| --------------------------------------------------------------------------- | ------- | ------------------------ |
| [Pi](https://en.wikipedia.org/wiki/Pi)                                      | `pi`    | $3.14159265358979\ldots$ |
| [Euler's number](<https://en.wikipedia.org/wiki/E_(mathematical_constant)>) | `e`     | $2.71828182845905\ldots$ |
| [Golden ratio](https://en.wikipedia.org/wiki/Golden_ratio)                  | `phi`   | $1.61803398874989\ldots$ |
| [$\sqrt{2}$](https://en.wikipedia.org/wiki/Square_root_of_2)                | `sqrt2` | $1.41421356237310\ldots$ |
| [$\sqrt{3}$](https://en.wikipedia.org/wiki/Square_root_of_3)                | `sqrt3` | $1.73205080756888\ldots$ |
| Infinity                                                                    | `inf`   | $\infty$                 |

**Physical Constants ([SI units](https://en.wikipedia.org/wiki/International_System_of_Units), [2019 redefinition](https://en.wikipedia.org/wiki/2019_redefinition_of_the_SI_base_units)):**

| Constant                                                                       | Value                   | Unit        |
| ------------------------------------------------------------------------------ | ----------------------- | ----------- |
| [Speed of light](https://en.wikipedia.org/wiki/Speed_of_light)                 | $299{,}792{,}458$       | m/s         |
| [Gravitational constant](https://en.wikipedia.org/wiki/Gravitational_constant) | $6.674 \times 10^{-11}$ | m³ kg⁻¹ s⁻² |
| [Planck's constant](https://en.wikipedia.org/wiki/Planck_constant)             | $6.626 \times 10^{-34}$ | J·s         |
| [Boltzmann constant](https://en.wikipedia.org/wiki/Boltzmann_constant)         | $1.381 \times 10^{-23}$ | J/K         |
| [Avogadro's number](https://en.wikipedia.org/wiki/Avogadro_constant)           | $6.022 \times 10^{23}$  | mol⁻¹       |
| [Elementary charge](https://en.wikipedia.org/wiki/Elementary_charge)           | $1.602 \times 10^{-19}$ | C           |
| [Vacuum permittivity](https://en.wikipedia.org/wiki/Vacuum_permittivity)       | $8.854 \times 10^{-12}$ | F/m         |
| [Vacuum permeability](https://en.wikipedia.org/wiki/Vacuum_permeability)       | $1.257 \times 10^{-6}$  | H/m         |
| [Electron mass](https://en.wikipedia.org/wiki/Electron_mass)                   | $9.109 \times 10^{-31}$ | kg          |
| [Proton mass](https://en.wikipedia.org/wiki/Proton#Mass)                       | $1.673 \times 10^{-27}$ | kg          |
| [Standard gravity](https://en.wikipedia.org/wiki/Standard_gravity)             | $9.80665$               | m/s²        |
| [Standard atmosphere](<https://en.wikipedia.org/wiki/Atmosphere_(unit)>)       | $101{,}325$             | Pa          |

```
[RAD]> 2pi
  = 6.28318530718

[RAD]> e^(pi*sqrt(163))
  = 262537412640768744  (Ramanujan's constant — nearly an integer!)
```

> **Further reading**: [NIST Fundamental Physical Constants](https://physics.nist.gov/cuu/Constants/) · [Mathematical Constants (Wolfram)](https://mathworld.wolfram.com/Constant.html)

---

### 12. Memory, Variables & History

Because a calculator without memory is just an abacus with extra steps:

| Feature            | Description                                             | Capacity                                                                       |
| ------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Memory slots**   | Store/recall numbered slots with M+, M− operations      | 10 slots                                                                       |
| **User variables** | Named variables: `let x = 3*pi`                         | 50 variables                                                                   |
| **Answer history** | Recall previous results with `ans`; view with `history` | 100 entries ([circular buffer](https://en.wikipedia.org/wiki/Circular_buffer)) |

**REPL commands for memory:**

```
[RAD]> let radius = 5
  radius = 5

[RAD]> let area = pi * radius ^ 2
  area = 78.5398163397

[RAD]> ms 0 42
  Stored 42 in memory slot 0

[RAD]> mr 0
  = 42

[RAD]> m+ 0 8
  Added 8 to slot 0

[RAD]> mr 0
  = 50

[RAD]> history
  [shows last calculations]

[RAD]> ans
  = [last result]
```

The history system uses a **[circular buffer](https://en.wikipedia.org/wiki/Circular_buffer)** — a ring data structure that overwrites the oldest entry when full. Elegant and memory-efficient.

> **Further reading**: [Circular Buffer (Wikipedia)](https://en.wikipedia.org/wiki/Circular_buffer) · [REPL (Wikipedia)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)

---

## Key Algorithms Under the Hood

Here's a table of the serious algorithms that power SciCalc:

| Algorithm                                                                           | Used In               | Complexity            |
| ----------------------------------------------------------------------------------- | --------------------- | --------------------- |
| [Recursive-descent parsing](https://en.wikipedia.org/wiki/Recursive_descent_parser) | Expression parser     | $O(n)$ per expression |
| [Lanczos approximation](https://en.wikipedia.org/wiki/Lanczos_approximation)        | Gamma function        | $O(1)$                |
| [Euler's reflection formula](https://en.wikipedia.org/wiki/Reflection_formula)      | Gamma (negative args) | $O(1)$                |
| [Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm)            | GCD                   | $O(\log \min(a,b))$   |
| [LU decomposition](https://en.wikipedia.org/wiki/LU_decomposition)                  | Matrix determinant    | $O(n^3)$              |
| [Gauss–Jordan elimination](https://en.wikipedia.org/wiki/Gaussian_elimination)      | Matrix inverse        | $O(n^3)$              |
| [Gaussian elimination](https://en.wikipedia.org/wiki/Gaussian_elimination)          | Linear system solver  | $O(n^3)$              |
| [Error function approximation](https://en.wikipedia.org/wiki/Error_function)        | Normal CDF            | $O(1)$                |
| [Trial division](https://en.wikipedia.org/wiki/Trial_division)                      | Primality test        | $O(\sqrt{n})$         |

---

## Building & Running

### Prerequisites

- **GCC 4.9+** (or any [C11](<https://en.wikipedia.org/wiki/C11_(C_standard_revision)>)-compatible compiler)
- **GNU Make 3.81+**
- **C math library** (linked via `-lm`)

### Build It

```bash
# Clone the repository
git clone https://github.com/chandanv89/scicalc.git
cd scicalc

# Build the optimized release binary
make

# Run it!
./build/scicalc
```

### Other Build Targets

```bash
make debug      # Debug build with AddressSanitizer
make test       # Run all 158 tests
make clean      # Clean build artifacts
sudo make install   # Install to /usr/local/bin
make help       # Show all targets
```

### Pipe Mode

You can also pipe expressions directly:

```bash
echo "sqrt(144) + sin(pi/6)" | ./build/scicalc
# Output: = 12.5

echo "2^16 - 1" | ./build/scicalc
# Output: = 65535
```

---

## Testing

```bash
make test
```

This runs **158 automated tests** across **10 test suites**:

| Suite         | What's Tested                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------------ |
| Arithmetic    | Basic ops, power/root, `abs`/`sign`, rounding, `gcd`/`lcm`, primality                                  |
| Trigonometry  | `sin`/`cos`/`tan`, `csc`/`sec`/`cot`, inverse, hyperbolic, angle modes                                 |
| Logarithmic   | `ln`, `log10`, `log2`, `logb`, `exp`, `exp2`                                                           |
| Combinatorics | Factorial (0–20), $\Gamma(x)$, `nPr`, `nCr`, Fibonacci                                                 |
| Statistics    | Mean, median, mode, range, variance, std dev, correlation, regression                                  |
| Matrix        | Identity, multiply, determinant, inverse, rank, solve $Ax = b$                                         |
| Complex       | Arithmetic, magnitude, phase, conjugate, polar↔rectangular, `exp`, `ln`                                |
| Conversion    | Temperature, length, mass, energy, pressure, base conversions                                          |
| Memory        | Store/recall, M+/M−, clear, user variables, history                                                    |
| Parser        | Precedence, unary ops, implicit multiplication, 30+ functions, scientific notation, nested expressions |

The test framework uses lightweight assertion macros (`ASSERT_NEAR`, `ASSERT_EQ_INT`, `ASSERT_TRUE`) with a tolerance of $10^{-9}$.

---

## Reflections: What I Learned

Looking back nearly two decades later — now as a Senior Java Developer who spends his days in IntelliJ instead of Vim, and writes `public static void main` instead of `int main` — here's what strikes me about this project:

**1. Scope creep is real — and sometimes it's glorious.**
What started as "add two numbers" became a full scientific calculator with matrix operations and probability distributions. Sometimes the best learning happens when you follow your curiosity down the rabbit hole.

**2. C teaches you discipline that stays with you forever.**
There's no garbage collector to save you. No exception handling to catch your falls. Every buffer has a size. Every pointer has a responsibility. C forces you to think about _everything_, and that's a superpower. Honestly, I'm a better Java developer _because_ I started with C. When you've manually managed memory, you never take a garbage collector for granted. When you've debugged a segfault at 2 AM, a `NullPointerException` feels almost… gentle.

**3. Writing a parser is a rite of passage.**
Every programmer should write a recursive-descent parser at least once. It's the moment you go from "I use programming languages" to "I understand how programming languages work."

**4. Modular design matters — even in college projects.**
Splitting the code into clean modules with well-defined interfaces made it possible to keep adding features without the whole thing collapsing under its own weight. The same principle serves me every day in Java — except now the modules are called "microservices" and they communicate over HTTP instead of function calls. Sometimes I wonder if we've overcomplicated things.

**5. Tests are a safety net.**
158 tests meant I could refactor confidently. Change the implementation of `sin()`? Run the tests. Add implicit multiplication to the parser? Run the tests. Past-me understood this instinctively.

**6. I really, truly miss coding in C.**
Java is powerful, productive, and pays the bills. But C has a certain _purity_ to it. There's no magic. No hidden allocations. No framework doing things behind your back. You write what you mean, and the machine does exactly that. Finding this project and watching it compile without a single change after all these years reminded me why I fell in love with programming in the first place. It wasn't about the frameworks or the design patterns — it was about the sheer joy of making a computer do something _cool_.

---

## The Full Function Reference

For quick reference, here are all functions recognized by the expression parser:

**Single-argument functions (30+):**

```
sin  cos  tan   asin  acos  atan  csc   sec  cot
sinh cosh tanh  asinh acosh atanh
ln   log  log10 log2  exp   exp2
sqrt cbrt abs   ceil  floor round trunc sign
factorial gamma fib   deg   rad
```

**Two-argument functions (13):**

```
pow(a,b)   root(x,n)  mod(a,b)   max(a,b)    min(a,b)
gcd(a,b)   lcm(a,b)   logb(x,b)  atan2(y,x)
perm(n,r)  comb(n,r)  nPr(n,r)   nCr(n,r)
```

---

## Wrapping Up

SciCalc is a love letter to the C programming language, written by a first-year engineering student who didn't know when to stop. And honestly? That's the best kind of project. The kind where curiosity takes the wheel and you end up building something far beyond what you originally imagined.

Finding this code in an old backup, unzipping it, and watching `gcc` compile it without a single modification after nearly two decades — that was a moment. It reminded me that good code transcends time. That the fundamentals matter. And that sometimes, the most rewarding thing a Senior Java Developer can do on a Sunday afternoon is fire up a terminal and `#include <stdio.h>` just for old times' sake.

If you're a student just starting out with C (or any language), here's my advice: **take the simplest assignment you're given and make it absurdly over-engineered.** Build a calculator that does matrix operations. Write a to-do app that has a query language. Make a sorting visualizer that supports 15 algorithms. You'll learn more from one passion project than from a semester of lab exercises. And who knows — maybe 18 years from now, you'll unzip it from a backup and smile.

The source code is on GitHub: **[github.com/chandanv89/scicalc](https://github.com/chandanv89/scicalc)**

Star it if you like it. Fork it if you want to add eigenvalue decomposition (I clearly ran out of time). Or just marvel at what a caffeinated engineering freshman can accomplish with `gcc` and a dream.

---

#### Footnotes

[[1](#what-is-scicalc)] _About C89/90 and C11 Compatibility_

> <small>C11 is a superset of C89/C90 — nearly all valid C89 code is valid C11. There are only a handful of minor breaking changes introduced across C99 and C11:
>
> <small>Implicit int removed (C99) — e.g., main() without int return type. But your code already uses int main().
> Implicit function declarations removed (C99) — calling a function without a prior declaration/include. Well-written code with proper #include headers (like yours) is unaffected.
> gets() removed (C11) — replaced by fgets(). Your code uses fgets()-style input via util_read_line().
>
> <small>That's basically it. If the code was reasonably well-written C89 — used explicit types, included proper headers, didn't rely on gets() — it compiles under -std=c11 with zero issues. >Which is exactly what happened with SciCalc.
>
> <small>The code was written in the C89/C90 era (2007–08), but because C is so backward-compatible, it compiles cleanly under modern GCC with -std=c11. The "C11" in the Makefile and table > >just reflects the current build target, not the era it was written in. That's actually part of what makes the story compelling — C's stability across decades.
