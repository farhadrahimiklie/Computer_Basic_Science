

# 🔥 What Is a Binary Calculator?

A **binary calculator** operates on numbers expressed in **base-2**, not base-10.

Decimal:

```
13
```

Binary:

```
1101
```

A binary calculator must:

1. Accept binary numbers as input.
2. Validate them.
3. Perform operations:

   * Addition
   * Subtraction
   * Multiplication
   * Division (optional but important)
4. Output binary results.
5. Handle:

   * Carry
   * Borrow
   * Overflow
   * Different lengths
   * Negative numbers (advanced stage)

We will start with **unsigned binary integers** first.

---

# 🧠 Binary Number System (You MUST Understand This)

Binary uses only:

```
0 and 1
```

Each digit is a **bit**.

Positions are powers of 2:

| Position | Value   |
| -------- | ------- |
| 0        | 2^0 = 1 |
| 1        | 2^1 = 2 |
| 2        | 2^2 = 4 |
| 3        | 2^3 = 8 |

Example:

```
1011₂
= 1×8 + 0×4 + 1×2 + 1×1
= 11₁₀
```

---

# ⚙️ Strategy: How Will WE Implement It?

We have two main choices:

### ✅ Option 1: Convert binary → decimal → compute → convert back

Easy, but **cheating** at low level.

### ✅ Option 2: Operate directly on bits

This is what real hardware does.
This is what makes you *master* it.

We will do **Option 2**.

---

# 🧱 Data Representation in C

We will represent binary numbers as:

```
char arrays (strings)
```

Example:

```
"101011"
```

Why?

* Arbitrary length
* Easy indexing
* Manual bit logic

---

# 🛑 Step 1 — Input Validation

A binary number must contain **only**:

```
'0' or '1'
```

C function:

```c
int isBinary(const char *s)
{
    for (int i = 0; s[i] != '\0'; i++)
    {
        if (s[i] != '0' && s[i] != '1')
            return 0;
    }
    return 1;
}
```

If input fails → reject.

No mercy.

---

# ➕ Step 2 — Binary Addition Algorithm

You add from **right to left**.

Rules:

| A | B | Carry In | Sum | Carry Out |
| - | - | -------- | --- | --------- |
| 0 | 0 | 0        | 0   | 0         |
| 0 | 1 | 0        | 1   | 0         |
| 1 | 0 | 0        | 1   | 0         |
| 1 | 1 | 0        | 0   | 1         |
| 1 | 1 | 1        | 1   | 1         |

### Algorithm:

1. Start from last index.
2. Convert chars → ints.
3. Add + carry.
4. Store result bit.
5. Compute new carry.
6. Continue.
7. If carry remains at end → prepend.

---

### C Implementation:

```c
#include <stdio.h>
#include <string.h>

void binaryAdd(const char *a, const char *b, char *result)
{
    int i = strlen(a) - 1;
    int j = strlen(b) - 1;
    int k = 0;

    int carry = 0;
    char temp[1024];

    while (i >= 0 || j >= 0 || carry)
    {
        int bitA = (i >= 0) ? a[i--] - '0' : 0;
        int bitB = (j >= 0) ? b[j--] - '0' : 0;

        int sum = bitA + bitB + carry;

        temp[k++] = (sum % 2) + '0';
        carry = sum / 2;
    }

    // reverse
    for (int x = 0; x < k; x++)
        result[x] = temp[k - x - 1];

    result[k] = '\0';
}
```

---

# ➖ Step 3 — Binary Subtraction

We assume:

```
a >= b
```

Rules with borrow:

```
0 - 1 → borrow → (10 - 1) = 1
```

Algorithm:

* Work right → left
* Track borrow
* Adjust bits

---

### C Code:

```c
void binarySubtract(const char *a, const char *b, char *result)
{
    int i = strlen(a) - 1;
    int j = strlen(b) - 1;
    int borrow = 0;
    int k = 0;

    char temp[1024];

    while (i >= 0)
    {
        int bitA = a[i] - '0' - borrow;
        int bitB = (j >= 0) ? b[j] - '0' : 0;

        if (bitA < bitB)
        {
            bitA += 2;
            borrow = 1;
        }
        else
            borrow = 0;

        temp[k++] = (bitA - bitB) + '0';

        i--;
        j--;
    }

    // Remove leading zeros
    while (k > 1 && temp[k - 1] == '0')
        k--;

    for (int x = 0; x < k; x++)
        result[x] = temp[k - x - 1];

    result[k] = '\0';
}
```

---

# ✖️ Step 4 — Binary Multiplication

Exactly like decimal multiplication:

* For each bit in B:

  * If bit = 1 → shift A left and add
  * If bit = 0 → skip

---

### C Code:

```c
void shiftLeft(const char *a, int shifts, char *out)
{
    strcpy(out, a);
    for (int i = 0; i < shifts; i++)
        strcat(out, "0");
}

void binaryMultiply(const char *a, const char *b, char *result)
{
    char temp[1024] = "0";
    char shifted[1024];
    char sum[1024];

    int lenB = strlen(b);

    for (int i = lenB - 1, shift = 0; i >= 0; i--, shift++)
    {
        if (b[i] == '1')
        {
            shiftLeft(a, shift, shifted);
            binaryAdd(temp, shifted, sum);
            strcpy(temp, sum);
        }
    }

    strcpy(result, temp);
}
```

---

# 🧪 Step 5 — Main Program

```c
int main()
{
    char a[512], b[512];
    char result[1024];

    printf("Enter first binary: ");
    scanf("%s", a);

    printf("Enter second binary: ");
    scanf("%s", b);

    if (!isBinary(a) || !isBinary(b))
    {
        printf("Invalid binary input!\n");
        return 1;
    }

    binaryAdd(a, b, result);
    printf("Addition: %s\n", result);

    binarySubtract(a, b, result);
    printf("Subtraction (a-b): %s\n", result);

    binaryMultiply(a, b, result);
    printf("Multiplication: %s\n", result);

    return 0;
}
```

---

# ⚠️ What You STILL Haven’t Mastered Yet

To be a **real** master, you must next implement:

1. ✅ Binary division
2. ✅ Two’s complement negative numbers
3. ✅ Overflow detection
4. ✅ Arbitrary precision dynamic memory
5. ✅ Menu-driven UI
6. ✅ Bitwise operators vs manual logic
7. ✅ Hex ↔ Binary ↔ Decimal conversion
8. ✅ Unit tests
9. ✅ Input limits
10. ✅ Performance optimization

---

# 🧠 Your Assignment (NO skipping)

Implement **binary division** manually:

* Repeated subtraction
* Long division method

Then add support for:

```
-10101
```

using two’s complement.


