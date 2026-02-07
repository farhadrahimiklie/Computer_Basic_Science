# 🔥 TWO’S COMPLEMENT — COMPLETE MASTERCLASS

---

## 1. Why Signed Numbers Are Hard in Binary

Binary naturally represents **unsigned integers**:

```
0000 = 0
0001 = 1
...
1111 = 15  (4 bits)
```

But computers must represent **negative numbers**.

We need:

* Efficient addition/subtraction
* One representation for zero
* Hardware simplicity
* Predictable overflow
* No special logic for sign

Two’s complement satisfies all of these.

---

---

# 2. Historical Alternatives (So You Appreciate Two’s Complement)

Before two’s complement:

### ❌ Sign-Magnitude

```
1001 = -1
0001 = +1
```

Problems:

* Two zeros (+0 and -0)
* Arithmetic complicated

---

### ❌ One’s Complement

Negative = bitwise NOT

```
5  = 0101
-5 = 1010
```

Still:

* Two zeros
* End-around carry required
* Hardware uglier

---

### ✅ Two’s Complement

Fixes everything.

---

---

# 3. Definition of Two’s Complement

For an **N-bit** number:

> A two’s complement number represents values from
> **−2^(N−1)** to **+2^(N−1) − 1**

Example for 8 bits:

```
-128 to +127
```

---

### 🔑 How to form −X:

1. Write X in binary.
2. Invert all bits.
3. Add 1.

That’s two’s complement.

---

Example:

```
+13 = 00001101
invert = 11110010
+1     = 11110011  →  -13
```

---

---

# 4. Why It Works (Mathematical Truth)

This is the part most people skip. We will **not** skip it.

An N-bit two’s complement number is interpreted as:

```
b[N-1]*(-2^(N-1)) + Σ b[i]*2^i   for i=0..N-2
```

The MSB has **negative weight**.

Example:

```
11110011
```

Bits:

```
1*(-128) +
1*(64) +
1*(32) +
1*(16) +
0*(8) +
0*(4) +
1*(2) +
1*(1)
```

Sum:

```
-128 + 64 + 32 + 16 + 2 + 1 = -13
```

🔥 THAT is two’s complement—not magic.

---

---

# 5. Modular Arithmetic View (Core Insight)

Two’s complement is **modulo arithmetic**.

For N bits, arithmetic wraps modulo:

```
2^N
```

Example: 8 bits → mod 256.

So:

```
256 - 13 = 243
```

243 in binary:

```
11110011
```

Which is -13.

NEGATIVE numbers are stored as:

```
2^N - |value|
```

---

---

# 6. Range Proof

Because MSB has weight:

```
-2^(N-1)
```

Max positive:

```
01111111 = 127
```

Min negative:

```
10000000 = -128
```

Asymmetric range is unavoidable.

---

---

# 7. Arithmetic — Why Addition “Just Works”

Example: 7 + (-3)

```
  00000111
+ 11111101
-----------
1 00000100   (carry dropped)
-----------
  00000100  = 4
```

🔥 Same adder hardware as unsigned.

This is why CPUs love two’s complement.

---

---

# 8. Subtraction Is Addition

```
A - B = A + (-B)
```

Hardware never subtracts. It negates and adds.

---

---

# 9. Overflow — The Subtle Part

Signed overflow happens when:

* adding two positives → negative result
* adding two negatives → positive result

In binary:

```
if sign(A) == sign(B) and sign(result) != sign(A)
```

---

### C Code:

```c
int add_overflow(int a, int b, int *result) {
    *result = a + b;

    if ((a > 0 && b > 0 && *result < 0) ||
        (a < 0 && b < 0 && *result > 0))
        return 1;

    return 0;
}
```

---

---

# 10. NEGATION Edge Case — MIN VALUE

In 8 bits:

```
-128 = 10000000
```

Invert +1:

```
01111111 + 1 = 10000000
```

It maps to itself.

🔥 There is NO +128.

This matters in C:

```c
int x = INT_MIN;
x = -x;   // undefined behavior!
```

---

---

# 11. Shifts and Sign Extension

### Arithmetic right shift:

Preserves sign bit:

```
11110000 >> 2 = 11111100
```

Logical shift:

```
11110000 >>> 2 = 00111100
```

C:

* `>>` on signed is implementation-defined but usually arithmetic.

---

---

# 12. Casting & Bit Patterns in C

```c
signed char a = -5;
unsigned char b = (unsigned char)a;
```

Bits identical.

Only interpretation changes.

---

---

# 13. Detecting Sign Bit

```c
int is_negative(int x) {
    return (x >> (sizeof(int)*8 - 1)) & 1;
}
```

---

---

# 14. Absolute Value Without Branch

```c
int abs_tc(int x) {
    int mask = x >> 31;
    return (x ^ mask) - mask;
}
```

⚠️ Breaks for INT_MIN.

---

---

# 15. Why All Modern CPUs Use Two’s Complement

* Single zero
* Fast arithmetic
* Simple ALUs
* Wraparound math
* Bitwise ops consistent
* Signed/unsigned share hardware

---

---

# 16. Visual Number Line (8 bits)

```
00000000 = 0
...
01111111 = 127
10000000 = -128
...
11111111 = -1
```

---

---

# 17. How Hardware Actually Negates

To compute `-x`:

1. XOR with all-ones mask
2. Add 1

ALU literally has inverter + carry-in.

---

---

# 18. Multiplication & Division

Implemented via:

* shift + add
* Booth’s algorithm (uses two’s complement patterns heavily)

---

---

# 19. Rules You MUST Internalize

If you want mastery, memorize these:

✅ MSB is negative weight
✅ Negative = 2^N − |x|
✅ Addition wraps modulo 2^N
✅ One zero only
✅ Range is asymmetric
✅ INT_MIN has no positive
✅ Overflow depends on sign bits
✅ Bit pattern ≠ meaning

---

---

# 20. Exercises (You Should Do These)

If you *really* want mastery, solve:

1. Represent −37 in 10 bits.
2. What is 0b10110110 in 8-bit signed?
3. Show binary for 45 − 83.
4. Detect overflow using only bitwise ops.
5. Why does `(unsigned)x` reinterpret bits?


