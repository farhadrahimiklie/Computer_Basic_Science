# PART 1 — WHAT A NUMBER SYSTEM REALLY IS

Humans normally use **decimal (base-10)**:

Digits:

```
0 1 2 3 4 5 6 7 8 9
```

Why 10? Because humans have 10 fingers. That’s it. No magic.

A **number system** is defined by:

1. **Base (radix)** → how many symbols
2. **Place values** → powers of the base

In decimal:

| Position  | Value |
| --------- | ----- |
| ones      | 10^0  |
| tens      | 10^1  |
| hundreds  | 10^2  |
| thousands | 10^3  |

Example:

```
573 = 5×10^2 + 7×10^1 + 3×10^0
```

---

# PART 2 — BINARY (BASE-2)

Binary uses **only two digits**:

```
0 and 1
```

That’s because digital electronics uses **two voltage states**:

* Low voltage → 0
* High voltage → 1

This makes circuits reliable and cheap.

---

## 2.1 Binary Place Values

Binary positions are powers of 2:

| Position  | Power | Value |
| --------- | ----- | ----- |
| rightmost | 2^0   | 1     |
|           | 2^1   | 2     |
|           | 2^2   | 4     |
|           | 2^3   | 8     |
|           | 2^4   | 16    |
|           | 2^5   | 32    |
|           | 2^6   | 64    |
|           | 2^7   | 128   |

---

## 2.2 Reading a Binary Number

Example:

```
101101₂
```

Label positions:

```
1  0  1  1  0  1
^  ^  ^  ^  ^  ^
32 16 8 4 2 1
```

Compute:

```
1×32 + 0×16 + 1×8 + 1×4 + 0×2 + 1×1
= 32 + 8 + 4 + 1
= 45₁₀
```

---

# PART 3 — DECIMAL → BINARY (THE RIGHT WAY)

You **do not guess**. You divide by 2 repeatedly.

Example: Convert 45 to binary.

```
45 ÷ 2 = 22 remainder 1
22 ÷ 2 = 11 remainder 0
11 ÷ 2 = 5  remainder 1
5  ÷ 2 = 2  remainder 1
2  ÷ 2 = 1  remainder 0
1  ÷ 2 = 0  remainder 1
```

Read remainders **bottom to top**:

```
101101₂
```

---

# PART 4 — BITS, BYTES, WORDS

* **bit** = 1 binary digit
* **byte** = 8 bits
* **nibble** = 4 bits
* **word** = CPU dependent (32-bit, 64-bit)

Example byte:

```
11001010
```

---

# PART 5 — HEXADECIMAL (BASE-16)

Binary is perfect for machines.
But for humans? Painful to read:

```
1101111010100111
```

So we compress it using **hexadecimal**.

Base-16 means **16 symbols**:

```
0–9 and A–F
```

Where:

```
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
```

---

## 5.1 Hex Place Values

Powers of 16:

| Position  | Value |
| --------- | ----- |
| rightmost | 16^0  |
|           | 16^1  |
|           | 16^2  |
|           | 16^3  |

---

# PART 6 — WHY HEX MATCHES BINARY PERFECTLY

Because:

```
16 = 2^4
```

So **one hex digit = exactly four binary bits**.

This is critical.

Binary nibble:

```
0000 → 0
0001 → 1
...
1001 → 9
1010 → A
1011 → B
1100 → C
1101 → D
1110 → E
1111 → F
```

You must memorize this table.

---

# PART 7 — BINARY → HEX (FAST METHOD)

Example:

```
1101111010100111
```

Group from the right in 4s:

```
1101 1110 1010 0111
```

Convert each:

```
1101 → D
1110 → E
1010 → A
0111 → 7
```

Result:

```
0xDEA7
```

---

# PART 8 — HEX → BINARY

Reverse the process:

```
0x3F9
```

```
3 → 0011
F → 1111
9 → 1001
```

```
001111111001
```

---

# PART 9 — SIGNED INTEGERS (TWO’S COMPLEMENT)

This is where amateurs quit and professionals start.

Computers store negative numbers using **two’s complement**.

For an N-bit number:

* Highest bit = sign bit
* 0 = positive
* 1 = negative

---

## 9.1 How to Form −X

Steps:

1. Write X in binary
2. Invert all bits (one’s complement)
3. Add 1

Example: −13 in 8 bits.

13:

```
00001101
```

Invert:

```
11110010
```

Add 1:

```
11110011
```

That is −13.

---

## 9.2 Why It Works

Because arithmetic hardware becomes simple:
same adder for signed and unsigned numbers.

Overflow wraps naturally.

---

# PART 10 — RANGE OF VALUES

For N bits:

Unsigned:

```
0 → 2^N − 1
```

Signed:

```
−2^(N−1) → 2^(N−1) − 1
```

8-bit signed:

```
−128 → +127
```

---

# PART 11 — OVERFLOW

Add two numbers that exceed range:

```
127 + 1 = -128  (8-bit signed)
```

Binary:

```
01111111
+00000001
---------
10000000
```

Sign bit flipped → overflow.

C does **not** protect you here.

---

# PART 12 — BITWISE OPERATIONS (THE REAL POWER)

In C:

```
&   AND
|   OR
^   XOR
~   NOT
<<  left shift
>>  right shift
```

Example:

```c
#include <stdio.h>

int main() {
    unsigned char a = 0x3C; // 00111100
    unsigned char b = 0x0F; // 00001111

    printf("%02X\n", a & b); // 0C
    printf("%02X\n", a | b); // 3F
    printf("%02X\n", a ^ b); // 33
    printf("%02X\n", ~a);    // C3

    return 0;
}
```

---

# PART 13 — SHIFTING = MULTIPLY/DIVIDE BY POWERS OF 2

```
x << 1  == x * 2
x >> 1  == x / 2 (unsigned)
```

Example:

```c
int x = 5;      // 00000101
int y = x << 3; // 40
```

---

# PART 14 — MEMORY REPRESENTATION

Suppose:

```c
int x = 0x12345678;
```

In RAM (little endian):

```
78 56 34 12
```

Big endian:

```
12 34 56 78
```

Hex is used in debuggers because it maps cleanly to bytes.

---

# PART 15 — PRACTICE UNTIL IT’S INSTINCT

If you truly want mastery, you should be able to:

✔ Convert any number between decimal, binary, hex
✔ Recognize powers of two instantly
✔ Do two’s complement in your head
✔ Read memory dumps
✔ Predict overflow
✔ Understand masks
✔ Use shifts for packing data
✔ Debug binary protocols

---

If you want to continue **ruthless-mentor mode**, next we can go even deeper:

* Binary fractions (fixed-point & floating-point)
* IEEE-754 floats in hex
* Bitfields in C
* Masks & flags
* CPU registers
* Instruction encoding
* How compilers emit machine code

Say the word.

