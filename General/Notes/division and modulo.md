# Division and Modulo in JavaScript — A Detailed Guide

## 1. Division (`/`)

JavaScript has **one** division operator: `/`. Unlike languages like C, Java, or Python (in Python 2), JavaScript does **not** have a separate "integer division" operator. Every division in JS is done using **floating-point arithmetic** (IEEE 754 double-precision), even when both operands are whole numbers.

### Basic examples

```javascript
10 / 2;   // 5
7 / 2;    // 3.5
1 / 3;    // 0.3333333333333333
-10 / 3;  // -3.3333333333333335
```

Notice `7 / 2` gives `3.5`, not `3`. There is no automatic truncation — JS always gives you the real, exact (floating-point-precision) result.

### Division by zero

This is one of the most distinctive JS behaviors compared to most other languages (where dividing by zero throws an error or crashes the program). In JavaScript:

```javascript
5 / 0;    // Infinity
-5 / 0;   // -Infinity
0 / 0;    // NaN
```

- A positive number divided by `0` gives `Infinity`.
- A negative number divided by `0` gives `-Infinity`.
- `0 / 0` gives `NaN` (Not-a-Number) — this is the one case where the sign gives no consistent result, so JS gives up and returns a special "invalid number" value.

`Infinity`, `-Infinity`, and `NaN` are all valid values of JavaScript's single `number` type — there's no separate error thrown.

### Getting an integer result

If you want the "integer division" behavior that other languages give by default, you have to ask for it explicitly. Common ways:

```javascript
Math.floor(7 / 2);   // 3   → rounds toward negative infinity
Math.trunc(7 / 2);   // 3   → chops off the decimal part
Math.ceil(7 / 2);    // 4   → rounds toward positive infinity
Math.round(7 / 2);   // 4   → rounds to nearest
```

`Math.floor` and `Math.trunc` behave **differently with negative numbers** — this trips people up constantly:

```javascript
Math.floor(-7 / 2);  // -4   (rounds down, i.e., toward -Infinity)
Math.trunc(-7 / 2);  // -3   (just removes the decimal, toward 0)
```

A shorthand some people use for truncating division of positive integers is the bitwise OR trick:
```javascript
(7 / 2) | 0;   // 3
```
This works because bitwise operators force their operand into a 32-bit signed integer first, discarding the fractional part. It's a common "clever" trick, but it's limited to values that fit in 32 bits (roughly ±2.1 billion) and is considered less readable than `Math.trunc()`. Prefer `Math.trunc()` in real code.

---

## 2. The Remainder Operator (`%`)

JavaScript's `%` is officially called the **remainder operator**, not "modulo" — this naming distinction matters and we'll explain exactly why below.

### Basic examples

```javascript
7 % 2;    // 1
8 % 2;    // 0
9 % 4;    // 1
```

The way to think about `%` is: **"what's left over after dividing as many whole times as possible?"**

Formally, JavaScript computes `%` using this relationship:
```
a % b === a - (b * Math.trunc(a / b))
```
That is: it divides, truncates toward zero to get the integer quotient, multiplies back, and subtracts to find the leftover.

### Why it's "remainder," not "modulo" — the negative number problem

This is the single most important, most-often-misunderstood fact about `%` in JavaScript.

**True mathematical modulo** always returns a result with the **same sign as the divisor**.
**JavaScript's remainder (`%`)** always returns a result with the **same sign as the dividend** (the first number).

```javascript
5 % 3;     // 2   (both positive — no confusion here)
-5 % 3;    // -2  ← in true modulo, this would be 1
5 % -3;    // 2   ← in true modulo, this would be -1
-5 % -3;   // -2
```

Walk through `-5 % 3` step by step using the formula above:
```
Math.trunc(-5 / 3) = Math.trunc(-1.666...) = -1   (truncates toward 0, not floor!)
-5 - (3 * -1) = -5 - (-3) = -5 + 3 = -2
```
So JS gives `-2`. A "true modulo" operation (as used in Python, for instance) would instead use `Math.floor` in that formula rather than `Math.trunc`, giving `1`. Neither is "wrong" — they're different, well-defined mathematical conventions. JavaScript simply chose the "remainder" (truncating) convention, matching C, Java, and most C-derived languages.

### How to get "true" mathematical modulo in JS

If you need the always-positive-when-divisor-is-positive behavior (e.g., wrapping an index around an array, or a clock face calculation), you need to adjust manually:

```javascript
function trueMod(a, b) {
  return ((a % b) + b) % b;
}

trueMod(-5, 3);   // 1
trueMod(5, -3);   // -1
```

This works by: taking the (possibly negative) remainder, adding the divisor to push it into positive range if it was negative, then taking `% b` again to correct for cases where it didn't need adjusting (adding `b` when the result was already non-negative would overshoot, so the second `% b` brings it back down).

**A very common real-world use case — wrapping around an array:**
```javascript
const arr = ['a', 'b', 'c'];
let i = -1;

arr[i % arr.length];              // arr[-1] → undefined (wrong!)
arr[trueMod(i, arr.length)];      // arr[2] → 'c'  (correct wraparound)
```

### Remainder with non-integers

`%` isn't limited to integers — it works with floating-point numbers too:

```javascript
5.5 % 2;    // 1.5
7.3 % 2.1;  // 0.9999999999999982 (floating-point imprecision — see below)
```

### Remainder with zero

```javascript
5 % 0;    // NaN
0 % 5;    // 0
```
Any number modulo `0` is `NaN` (undefined operation — you can't find a remainder of dividing by nothing). But zero modulo anything (nonzero) is cleanly `0`.

---

## 3. Floating-Point Precision Gotchas

Because JS uses IEEE 754 double-precision floats for **all** numbers (there's no separate integer type in plain JS), both `/` and `%` can produce results that look slightly "off" due to how binary floating-point represents decimal fractions:

```javascript
0.1 + 0.2;        // 0.30000000000000004  (a classic, unrelated to / or %, but same root cause)
0.3 % 0.1;        // 0.09999999999999998  (you might expect 0, mathematically)
```

This isn't a bug in `/` or `%` specifically — it's a fundamental limitation of binary floating-point representation, and it affects nearly every language that uses IEEE 754 floats (which is almost all of them). If you need exact decimal arithmetic (e.g., for money), you should avoid raw floats entirely and either:
- work in integer cents/smallest-unit values, or
- use a library designed for arbitrary-precision decimal math.

---

## 4. `BigInt` — Division and Remainder for Arbitrary-Precision Integers

JavaScript's regular `number` type can only safely represent integers up to `2^53 - 1` (`Number.MAX_SAFE_INTEGER`). For larger whole numbers, JS has a separate type: `BigInt`.

```javascript
10n / 3n;    // 3n   ← BigInt division TRUNCATES automatically, no decimals allowed
10n % 3n;    // 1n
```

The `n` suffix marks a `BigInt` literal. Because `BigInt` has no fractional representation at all, `/` behaves like integer division automatically — there's no floating-point result to produce. This is a genuinely different behavior from regular numbers, where `/` always keeps decimals.

You cannot mix a regular `number` and a `BigInt` in the same operation — this throws:
```javascript
10n / 3;   // TypeError: Cannot mix BigInt and other types
```
You must explicitly convert one side (`BigInt(3)` or `Number(10n)`) before operating on them together.

---

## 5. Quick Reference Table

| Expression | Result | Notes |
|---|---|---|
| `7 / 2` | `3.5` | Always floating-point |
| `5 / 0` | `Infinity` | No error thrown |
| `-5 / 0` | `-Infinity` | |
| `0 / 0` | `NaN` | |
| `Math.trunc(7 / 2)` | `3` | Truncates toward 0 |
| `Math.floor(7 / 2)` | `3` | Rounds toward -Infinity |
| `Math.floor(-7 / 2)` | `-4` | Differs from `Math.trunc` on negatives |
| `7 % 2` | `1` | Sign matches dividend |
| `-5 % 3` | `-2` | NOT true mathematical modulo |
| `((a % b) + b) % b` | — | Pattern for true modulo |
| `5 % 0` | `NaN` | |
| `10n / 3n` | `3n` | BigInt division truncates automatically |

---

## 6. Summary

- **`/`** always produces a full-precision floating-point result in JS — there's no built-in integer division operator for regular numbers.
- **`%`** is a *remainder* operator, not a true mathematical *modulo* — its sign follows the dividend, not the divisor. This causes a very common bug when people assume Python-style modulo behavior.
- Division by zero never throws in JS — it produces `Infinity`, `-Infinity`, or `NaN` depending on the signs involved.
- For array-wraparound or clock-style calculations, use the `((a % b) + b) % b` pattern to get consistently non-negative results.
- `BigInt` division (`/`) truncates automatically because BigInts have no fractional part at all — this is a genuine behavioral difference from regular number division, not just a precision difference.
