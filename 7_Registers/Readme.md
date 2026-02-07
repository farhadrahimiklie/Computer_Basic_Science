# 1. What Is a Register—Really?

A **CPU register** is:

> The smallest, fastest storage location directly inside the CPU core.

Registers:

* Are implemented as flip-flops or latches in silicon.
* Sit *right next to* the execution units (ALUs, FPUs).
* Can be read/written in **1 CPU cycle** (or close).
* Are orders of magnitude faster than cache or RAM.

Hierarchy:

```
Registers  <-- fastest, tiny (tens to hundreds bytes total)
L1 Cache
L2 Cache
L3 Cache
RAM
Disk       <-- slowest
```

If RAM is a warehouse, registers are **the worker’s hands**.

---

# 2. Registers Exist in an ISA (Instruction Set Architecture)

Registers are **defined by the CPU architecture**, not by C.

Examples:

| Architecture | Register Type                                  |
| ------------ | ---------------------------------------------- |
| x86-64       | RAX, RBX, RCX, RDX, RSP, RBP, RSI, RDI, R8–R15 |
| ARM64        | X0–X30, SP                                     |
| RISC-V       | x0–x31                                         |
| MIPS         | $t0–$t9, $s0–$s7                               |

C does **not** define registers.
The compiler maps C variables → registers.

---

# 3. Categories of Registers

## 3.1 General-Purpose Registers (GPRs)

Used for:

* integers
* addresses
* loop counters
* parameters
* return values

Example x86-64:

```
RAX  accumulator / return value
RBX  callee-saved
RCX  counter
RDX  data
RSI  src pointer
RDI  dest pointer
RSP  stack pointer
RBP  frame pointer
R8–R15 extra
```

---

## 3.2 Special-Purpose Registers

### Stack Pointer

Points to top of stack.

x86-64: `RSP`
ARM64: `SP`

### Instruction Pointer / Program Counter

Holds next instruction address.

x86-64: `RIP`
ARM64: `PC`

---

## 3.3 Flags / Status Registers

Contain condition bits:

* Zero flag
* Carry
* Overflow
* Sign
* Parity

x86: `RFLAGS`

Used for:

```asm
cmp rax, rbx
je equal_label
```

---

## 3.4 Floating-Point / Vector Registers

Used for:

* float/double
* SIMD
* multimedia

x86-64:

* XMM0–XMM15 (128-bit)
* YMM (256)
* ZMM (512)

ARM:

* V0–V31

---

# 4. How C Uses Registers (Important)

When you write:

```c
int add(int a, int b) {
    return a + b;
}
```

The compiler might generate (x86-64 System V ABI):

```
a → EDI
b → ESI
return → EAX
```

So:

```
EDI + ESI → EAX
```

You never see registers in C unless:

* debugging assembly
* using inline asm
* writing kernel/embedded code

---

# 5. Calling Conventions (CRITICAL)

Registers are governed by the **ABI**.

ABI decides:

* Which registers pass arguments
* Which hold return values
* Which must be preserved

---

## x86-64 System V ABI (Linux/macOS)

Arguments:

1 → RDI
2 → RSI
3 → RDX
4 → RCX
5 → R8
6 → R9

Return: RAX

Callee-saved:

RBX, RBP, R12–R15

Caller-saved:

RAX, RCX, RDX, RSI, RDI, R8–R11

---

This matters for:

* writing assembly
* inline asm
* OS kernels
* JIT compilers

---

# 6. Registers vs Stack vs Memory

Compiler decides:

* Hot variables → registers
* Spill when not enough → stack

Example:

```c
int f(int a, int b, int c, int d, int e, int f) {
    int x = a + b + c + d + e + f;
    return x;
}
```

If registers run out, some values go to stack.

This is called:

> **Register spilling**

---

# 7. Register Allocation (Compiler Theory)

Compilers use algorithms:

* graph coloring
* linear scan

Each variable is **live** over a range.

Two variables that overlap in time cannot share a register.

Example:

```c
int a = 1;
int b = 2;
int c = a + b;
```

`a` and `b` are live simultaneously → need 2 registers.

---

# 8. The `register` Keyword in C

Old C:

```c
register int x;
```

Means: *please put in register*.

Modern truth:

* Compiler ignores it.
* Deprecated in C23.

You **cannot**:

* take address of register variable:

```c
register int x;
int *p = &x;  // illegal in old C
```

Modern compilers optimize better than humans.

---

# 9. Volatile and Registers

`volatile` prevents optimization.

```c
volatile int flag;

while (!flag) { }
```

Compiler **must reload** `flag` from memory each time, not cache in register.

Used for:

* memory-mapped I/O
* hardware registers
* ISR communication

---

# 10. Memory-Mapped Hardware Registers (Embedded)

In microcontrollers:

```c
#define GPIO_PORT (*(volatile unsigned int*)0x40021000)

GPIO_PORT |= 0x01;
```

This is **not CPU registers**—these are **peripheral registers mapped into memory**.

Different concept:

* CPU register → internal
* Hardware register → external device state

---

# 11. Inline Assembly in C

GCC:

```c
int result;

asm volatile (
    "movl $5, %%eax\n"
    "addl $3, %%eax\n"
    "movl %%eax, %0\n"
    : "=r"(result)
    :
    : "eax"
);
```

Meaning:

* output operand → register
* clobbers EAX

You must tell compiler:

* what registers you modify
* what memory you touch

Otherwise → undefined behavior.

---

# 12. Reading Registers in Debugging

GDB:

```
info registers
```

Shows:

RAX, RBX, RCX, RIP, RSP, FLAGS.

To see mapping:

```
disassemble
```

---

# 13. Pipeline, Renaming, and Modern CPUs

Modern CPUs:

* Rename registers internally
* Out-of-order execution
* Physical registers > architectural registers

Architectural register:

RAX

Physical registers:

P17, P42, etc.

Renaming avoids false dependencies.

---

# 14. Hazards Involving Registers

### Data hazard:

```
RAX = RAX + 1
```

CPU stalls if previous write not finished.

Renaming solves WAR/WAW hazards.

---

# 15. Why Registers Matter for Performance

Registers:

* avoid cache misses
* reduce loads/stores
* allow vectorization

Bad code:

```c
for (int i = 0; i < n; i++)
    sum += arr[i];
```

Good compiler:

* keeps `sum` in register
* `i` in register
* prefetches array

---

# 16. How to *Actually* Master Registers

You must:

1. Learn one ISA deeply (x86-64 or ARM64).
2. Read ABI docs.
3. Compile C with:

```
gcc -O2 -S file.c
```

4. Inspect assembly.
5. Write inline asm.
6. Write small OS kernels or bootloaders.
7. Use GDB to track registers.
8. Learn compiler backend theory.

---

# 17. Practice Exercise (Serious)

Compile this:

```c
long mul_add(long a, long b, long c) {
    long x = a * b;
    return x + c;
}
```

Run:

```
gcc -O2 -S test.c
```

Then answer:

* Which registers hold `a`, `b`, `c`?
* Which holds `x`?
* Is stack used?










# PART 1 — The Complete x86-64 Register File

x86-64 defines **16 general-purpose registers**, each 64-bit:

```
RAX  RBX  RCX  RDX
RSI  RDI  RBP  RSP
R8   R9   R10  R11
R12  R13  R14  R15
```

Plus:

* RIP → instruction pointer
* RFLAGS → status/control flags
* Segment registers (CS, DS, ES, FS, GS, SS)
* Vector registers: XMM0–XMM31 (architecture dependent)
* Control registers: CR0–CR4, CR8 (kernel only)
* Debug registers: DR0–DR7 (kernel/debugger)

User-space C code normally touches:

* GPRs
* RIP (implicitly)
* RFLAGS
* XMM/YMM

---

# PART 2 — Sub-Register Hierarchy (CRITICAL)

Each GPR has **views**:

Example RAX:

```
RAX  (64)
 EAX (32)
  AX (16)
  AH / AL (8/8)
```

Writing to **EAX zero-extends into RAX**.

This is fundamental:

```
mov eax, 1
```

→ RAX becomes 1 (upper 32 bits cleared).

Other registers:

```
RBX → EBX → BX → BH/BL
RCX → ECX → CX → CH/CL
R8  → R8D → R8W → R8B
```

R8–R15 **do not have AH/BH**.

---

# PART 3 — Calling Convention: System V AMD64 ABI

### Integer / pointer arguments:

| Arg | Register |
| --- | -------- |
| 1   | RDI      |
| 2   | RSI      |
| 3   | RDX      |
| 4   | RCX      |
| 5   | R8       |
| 6   | R9       |

Return value:

* RAX (primary)
* RDX:RAX for 128-bit

---

### Floating-point args:

* XMM0–XMM7

Return FP:

* XMM0

---

### Caller-Saved (volatile)

May be clobbered by callee:

```
RAX RCX RDX RSI RDI R8 R9 R10 R11
XMM0–XMM15
```

---

### Callee-Saved (non-volatile)

Must be preserved:

```
RBX RBP R12 R13 R14 R15
```

---

RSP **must be restored** before return.

---

# PART 4 — Stack Frame & Registers

Stack grows **downwards**.

Typical function prologue:

```
push rbp
mov rbp, rsp
sub rsp, 32
```

Epilogue:

```
mov rsp, rbp
pop rbp
ret
```

Locals that don’t fit in registers live at:

```
[rbp - offset]
```

---

# PART 5 — RFLAGS: Every Important Bit

RFLAGS is 64-bit; key bits:

| Flag | Meaning          |
| ---- | ---------------- |
| ZF   | zero             |
| SF   | sign             |
| CF   | carry            |
| OF   | overflow         |
| PF   | parity           |
| AF   | auxiliary        |
| IF   | interrupt enable |
| DF   | direction        |

Example:

```
cmp rax, rbx
```

sets flags based on rax - rbx.

Then:

```
je label   ; ZF=1
jl label   ; signed less
jb label   ; unsigned less (CF=1)
```

---

# PART 6 — Vector Registers

Modern x86-64:

* XMM0–15 → 128-bit
* YMM0–15 → 256-bit (AVX)
* ZMM0–31 → 512-bit (AVX-512)

C uses them for:

```c
double a, b, c = a + b;
```

→ XMM registers.

---

# PART 7 — How GCC Maps C → Registers

Compile:

```
gcc -O2 -S test.c
```

Example:

```c
long add(long a, long b) {
    return a + b;
}
```

Likely:

```
add:
    lea rax, [rdi + rsi]
    ret
```

a → RDI
b → RSI
return → RAX

---

# PART 8 — Spilling to Stack

If too many live values:

```
mov QWORD PTR [rsp+8], r10
```

That is register spill.

---

# PART 9 — Inline Assembly (Correct Way)

GCC syntax:

```c
long foo(long a, long b) {
    long r;
    asm volatile (
        "addq %2, %1\n"
        "movq %1, %0\n"
        : "=r"(r)
        : "r"(a), "r"(b)
        : "cc"
    );
    return r;
}
```

Compiler chooses registers.

Never hardcode unless needed:

```
"addq %%rsi, %%rdi"
```

---

# PART 10 — Hardware Register Renaming

Architectural:

RAX

Physical:

P0, P17, P38...

CPU maps automatically.

Prevents:

* WAR
* WAW hazards

---

# PART 11 — Flags vs Registers in Optimization

Compilers like:

```
test rax, rax
jne label
```

No compare immediate—sets ZF from AND.

---

# PART 12 — How You Practice Like a Professional

You:

1. Write C.
2. Compile with -O0 and -O2.
3. Diff assembly.
4. Track registers.
5. Use GDB:

```
break add
run
info registers
```

---

# FINAL DRILL

Study this:

```c
long f(long a, long b, long c, long d,
       long e, long f, long g) {
    long x = a + b + c + d + e + f + g;
    return x;
}
```


