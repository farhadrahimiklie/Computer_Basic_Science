

# 🔥 PART 1 — What “Bitwise” Really Means

Computers store numbers in **binary**.

Example:

```
Decimal: 13
Binary : 00001101
```

Each digit is a **bit** (0 or 1).

Bit positions (right → left):

```
Bit:     7 6 5 4 3 2 1 0
Value: 128 64 32 16 8 4 2 1
```

13 = 8 + 4 + 1

Bitwise operations work on **individual bits**, not whole numbers.

---

# 🔥 PART 2 — Bitwise Operators in C

C has **six** main bitwise operators:

| Operator | Name        |    |
| -------- | ----------- | -- |
| `&`      | AND         |    |
| `        | `           | OR |
| `^`      | XOR         |    |
| `~`      | NOT         |    |
| `<<`     | Left shift  |    |
| `>>`     | Right shift |    |

These operate **bit by bit**.

---

# 🔥 PART 3 — AND (`&`)

### Rule:

```
1 & 1 = 1
otherwise = 0
```

Truth table:

| A | B | A & B |
| - | - | ----- |
| 0 | 0 | 0     |
| 0 | 1 | 0     |
| 1 | 0 | 0     |
| 1 | 1 | 1     |

---

### Example:

```
a = 12  -> 1100
b = 10  -> 1010
----------------
a & b = 1000 = 8
```

### C Code:

```c
#include <stdio.h>

int main() {
    int a = 12;
    int b = 10;

    printf("%d\n", a & b); // 8
    return 0;
}
```

---

### Real Use: **Check a Bit**

To test if bit 3 is ON:

```c
if (x & (1 << 3)) {
    printf("Bit 3 is set\n");
}
```

---

# 🔥 PART 4 — OR (`|`)

Rule:

```
0 | 0 = 0
otherwise = 1
```

Example:

```
a = 12 -> 1100
b = 10 -> 1010
----------------
a | b = 1110 = 14
```

C:

```c
printf("%d\n", a | b);
```

---

### Real Use: **Set a Bit**

Turn bit 2 ON:

```c
x = x | (1 << 2);
```

(Shortcut form:)

```c
x |= (1 << 2);
```

---

# 🔥 PART 5 — XOR (`^`)

Rule:

```
same bits -> 0
different -> 1
```

Truth:

| A | B | A^B |
| - | - | --- |
| 0 | 0 | 0   |
| 0 | 1 | 1   |
| 1 | 0 | 1   |
| 1 | 1 | 0   |

Example:

```
a = 12 -> 1100
b = 10 -> 1010
----------------
a^b = 0110 = 6
```

---

### XOR Special Properties

**1️⃣ Toggle a bit**

```c
x ^= (1 << 4);
```

**2️⃣ Swap two numbers WITHOUT temp**

```c
a ^= b;
b ^= a;
a ^= b;
```

(Yes, works. Rarely used in production—understand it anyway.)

---

# 🔥 PART 6 — NOT (`~`)

Flips every bit.

Example (8-bit view):

```
x = 00001101 (13)
~x= 11110010 (-14 in signed two's complement)
```

⚠ **Important:** C uses **two’s complement** for signed ints on almost all systems.

For signed numbers:

```
~x == -x - 1
```

Example:

```c
int x = 13;
printf("%d\n", ~x); // -14
```

---

# 🔥 PART 7 — SHIFT OPERATORS

## Left Shift (`<<`)

Moves bits left, fills with 0.

```
5 = 00000101
5 << 1 = 00001010 = 10
```

### Effect:

```
x << n  ==  x * 2^n   (if no overflow)
```

C:

```c
int y = 5 << 3; // 40
```

---

## Right Shift (`>>`)

Shifts right.

For **unsigned** → fills with 0
For **signed** → implementation-defined (usually fills with sign bit).

Example:

```
20 = 10100
20 >> 2 = 00101 = 5
```

---

### Use unsigned for safety:

```c
unsigned int x = 20;
x >>= 2;
```

---

# 🔥 PART 8 — BIT MASKING (THIS IS CRITICAL)

Mask = a number where specific bits are 1.

---

### ✔ Check bit:

```c
if (x & (1 << k))
```

---

### ✔ Set bit:

```c
x |= (1 << k);
```

---

### ✔ Clear bit:

```c
x &= ~(1 << k);
```

---

### ✔ Toggle bit:

```c
x ^= (1 << k);
```

---

### ✔ Extract bits 4..7:

```c
int mask = 0b1111 << 4;
int result = (x & mask) >> 4;
```

---

# 🔥 PART 9 — Two’s Complement & Negative Numbers

You MUST understand this.

For 8 bits:

```
+5 = 00000101
Invert = 11111010
+1     = 11111011  => -5
```

Why `~x + 1 = -x`.

This is why:

```
~x = -x - 1
```

---

# 🔥 PART 10 — Undefined & Dangerous Cases (REAL MASTERY)

### ❌ Shifting negative numbers:

```c
int x = -5;
x << 1;   // Undefined behavior
```

---

### ❌ Shifting >= width:

```c
int x = 1;
x << 32;  // UB on 32-bit int
```

---

### ❌ Overflow in signed shifts:

```c
int x = 1 << 31; // Undefined for signed int
```

---

### ✔ Use unsigned:

```c
unsigned int x = 1u << 31;
```

---

# 🔥 PART 11 — Flags & Permissions (REAL WORLD)

Classic OS permissions:

```
READ  = 1 << 0
WRITE = 1 << 1
EXEC  = 1 << 2
```

Usage:

```c
int perm = 0;

perm |= READ;
perm |= WRITE;

if (perm & WRITE) {
    printf("Can write\n");
}
```

---

# 🔥 PART 12 — Counting Bits (Popcount)

Manual:

```c
int count = 0;
while (x) {
    count += x & 1;
    x >>= 1;
}
```

Brian Kernighan trick:

```c
while (x) {
    x &= (x - 1);
    count++;
}
```

---

# 🔥 PART 13 — Detect Power of Two

```c
if (x > 0 && (x & (x - 1)) == 0)
```

---

# 🔥 PART 14 — Endianness vs Bitwise

Bitwise ops ignore byte order—endianness matters only in memory layout, not `& | ^`.

---

# 🔥 PART 15 — Professional Rules

If you want mastery, follow these:

✅ Use unsigned for bit manipulation
✅ Comment masks
✅ Never shift by variable without bounds check
✅ Know your integer width (`stdint.h`)
✅ Prefer `uint32_t`, `uint64_t`

---

# 🎯 FINAL TEST FOR YOU (DO NOT SKIP)

Solve in C:

1️⃣ Set bits 1, 4, 6 in a variable
2️⃣ Clear bit 3
3️⃣ Toggle bit 0
4️⃣ Extract bits 2–5
5️⃣ Check if exactly one bit is set

When you answer, I will correct it ruthlessly.


