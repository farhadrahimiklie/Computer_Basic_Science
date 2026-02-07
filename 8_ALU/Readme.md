

# 🔥 WHAT IS AN ALU — REALLY?

An **ALU** is the part of a CPU that performs:

* Arithmetic operations

  * addition
  * subtraction
  * multiplication (sometimes separate unit)
  * division (sometimes separate unit)
  * increment/decrement

* Logical operations

  * AND
  * OR
  * XOR
  * NOT
  * shifts
  * rotates
  * comparisons

It does **not**:

* fetch instructions
* decode instructions
* manage memory
* control branching

Those belong to the **control unit**, **register file**, **cache**, etc.

The ALU is a **pure combinational circuit**:
➡ input bits go in → gates compute → output bits appear.

No memory inside the ALU itself.

---

# 🧱 LEVEL 1 — FROM BITS TO GATES

Computers operate on **binary**:

* 0 = low voltage
* 1 = high voltage

Operations are implemented with **logic gates**:

| Gate     | Operation            |
| -------- | -------------------- |
| AND      | 1 only if both are 1 |
| OR       | 1 if any is 1        |
| XOR      | 1 if different       |
| NOT      | invert               |
| NAND/NOR | universal gates      |

An ALU is **millions of these gates wired together**.

---

# ➕ LEVEL 2 — ADDITION: THE CORE OF EVERYTHING

Most ALU operations reduce to **addition**.

Even subtraction = addition with complements.

---

## Half Adder

Adds two bits:

Inputs: A, B

Outputs:

* SUM = A XOR B
* CARRY = A AND B

---

## Full Adder

Adds three bits:

* A
* B
* CarryIn

Outputs:

```
SUM = A XOR B XOR Cin
CARRY = (A AND B) OR (Cin AND (A XOR B))
```

---

## 32-bit adder?

Chain 32 full adders:

```
bit0 -> bit1 -> bit2 -> ... -> bit31
```

Carry ripples through → **ripple carry adder**.

Slow.

Modern CPUs use:

* Carry Lookahead Adders
* Carry Select
* Kogge-Stone trees

To reduce propagation delay.

---

# ➖ SUBTRACTION — TWO'S COMPLEMENT

ALUs don’t really subtract.

They:

```
A - B = A + (~B + 1)
```

Steps:

1. Invert all bits of B
2. Add 1
3. Add to A

Hardware:

* XOR gates to flip B when subtracting
* CarryIn forced to 1

So **same adder does add & subtract**.

---

# 🔢 LEVEL 3 — OTHER ARITHMETIC

## Increment

```
A + 1
```

## Decrement

```
A - 1
```

## Negation

```
~A + 1
```

---

## Multiplication

Usually not inside basic ALU.

Uses:

* shift left
* add partial products

Example:

```
13 * 5

13 = 1101
5  = 0101

1101
x0101
-----
1101
0000
1101
0000
-----
1000001
```

Modern CPUs use:

* Wallace trees
* Booth encoding

Dedicated **MUL unit**.

---

## Division

Even slower.

Repeated subtract + shift.

Dedicated **DIV unit**.

---

# 🧠 LEVEL 4 — LOGICAL OPERATIONS

Bitwise:

* AND → masking
* OR → setting bits
* XOR → toggling
* NOT → inversion

Example in C:

```c
unsigned int a = 0b11001100;
unsigned int b = 0b10101010;

unsigned int andv = a & b;
unsigned int orv  = a | b;
unsigned int xorv = a ^ b;
unsigned int notv = ~a;
```

These compile to ALU logic ops.

---

# ↔️ LEVEL 5 — SHIFTERS & ROTATORS

Often part of ALU:

* Logical shift left
* Logical shift right
* Arithmetic shift right
* Rotate left/right

Hardware uses **barrel shifters**.

C:

```c
unsigned int x = 0xF0F0;

unsigned int l = x << 3;
unsigned int r = x >> 3;
```

Arithmetic shift depends on signed type.

---

# 🚦 LEVEL 6 — FLAGS / STATUS REGISTER

ALU sets flags:

| Flag  | Meaning  |
| ----- | -------- |
| Z     | Zero     |
| N / S | Negative |
| C     | Carry    |
| V / O | Overflow |
| P     | Parity   |

---

## Carry vs Overflow

**Carry** → unsigned overflow
**Overflow** → signed overflow

Example:

8-bit signed:

```
100 + 50 = 150
01100100
00110010
---------
10010110 (-106)  ← overflow
```

Carry = 0
Overflow = 1

Hardware:

```
Overflow = carry_into_MSB XOR carry_out_MSB
```

---

C demo:

```c
#include <stdio.h>
#include <stdint.h>

int main() {
    int8_t a = 100;
    int8_t b = 50;
    int8_t c = a + b;

    printf("%d\n", c); // overflow in hardware, UB in C standard
}
```

(Real CPUs overflow; C standard says signed overflow is undefined.)

---

# 🧩 LEVEL 7 — CONTROL SIGNALS

ALU is controlled by:

* ALUOp bits from instruction decoder

Example:

| ALUOp | Function |
| ----- | -------- |
| 000   | ADD      |
| 001   | SUB      |
| 010   | AND      |
| 011   | OR       |
| 100   | XOR      |
| 101   | SHIFT    |

Multiplexers choose output.

---

# 🏗 LEVEL 8 — BLOCK DIAGRAM

Simplified:

```
        A ----->|        |
                | ADDER |---+
        B ----->|        |   |
                             |----> MUX ---> RESULT
        A ----->| AND  |---+ |
        B ----->|      |     |
                             |
        A ----->| OR   |-----+
        B ----->|      |

Flags computed from RESULT & carries
```

---

# 🧪 LEVEL 9 — EMULATING AN ALU IN C

We can simulate a 32-bit ALU:

```c
#include <stdint.h>
#include <stdio.h>

typedef struct {
    uint32_t result;
    int zero;
    int carry;
    int overflow;
    int negative;
} ALUFlags;

ALUFlags alu_add(uint32_t a, uint32_t b) {
    ALUFlags f;
    uint64_t wide = (uint64_t)a + b;

    f.result = (uint32_t)wide;
    f.carry = (wide >> 32) & 1;
    f.zero = (f.result == 0);
    f.negative = (f.result >> 31) & 1;

    uint32_t sa = a >> 31;
    uint32_t sb = b >> 31;
    uint32_t sr = f.result >> 31;

    f.overflow = (sa == sb) && (sa != sr);

    return f;
}

int main() {
    ALUFlags f = alu_add(0x7FFFFFFF, 1);

    printf("res=%x carry=%d ov=%d\n",
           f.result, f.carry, f.overflow);
}
```

That is **exactly what hardware ALU logic does**.

---

# 🧬 LEVEL 10 — ALU INSIDE PIPELINED CPU

In real processors:

Pipeline:

```
Fetch -> Decode -> Execute(ALU) -> Memory -> Writeback
```

ALU is used for:

* arithmetic
* branch comparison
* address calculation
* loop counters

Multiple ALUs may exist:

* integer ALUs
* vector ALUs
* FPUs
* SIMD units

---

# ⚡ LEVEL 11 — MODERN ALUs

Modern CPUs have:

* multiple ALUs per core
* out-of-order scheduling
* reservation stations
* rename registers
* speculative execution

Each ALU still obeys:

➡ combinational logic + inputs + control signals.

---

# 🧠 FINAL MENTOR TALK

You now understand:

✔ what ALU is
✔ gate-level design
✔ adders
✔ subtraction
✔ flags
✔ overflow
✔ shifters
✔ control lines
✔ CPU integration
✔ C-level emulation

This is **computer architecture foundation**.


