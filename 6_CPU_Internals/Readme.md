# 1. WHAT A CPU REALLY IS

A CPU is **not software**.
It is a **synchronous digital circuit** built from:

* transistors → logic gates
* gates → combinational logic + flip-flops
* flip-flops → registers
* registers + ALUs + control → datapath
* datapath + memory hierarchy → processor

Everything runs on a **clock**.

At every rising edge:

* registers latch new values
* combinational logic computes next state

That’s it.
All “instructions” are just **patterns of bits** flowing through hardware.

---

# 2. TRANSISTORS → GATES → REGISTERS

Modern CPUs use CMOS transistors.

From them we build:

* NOT
* AND
* OR
* XOR

Using gates we build:

### Half Adder

Adds two bits:

* sum = A XOR B
* carry = A AND B

### Full Adder

```
sum  = A ^ B ^ Cin
cout = (A & B) | (Cin & (A ^ B))
```

Chain 64 of these → **64-bit adder**.

Registers are built from **flip-flops**.

A flip-flop stores **1 bit**.

64 flip-flops → 64-bit register.

---

# 3. THE BASIC CPU DATAPATH

The simplest CPU has:

* Program Counter (PC)
* Instruction Memory
* Register File
* ALU
* Data Memory
* Control Unit

Flow:

1. PC fetches instruction
2. Decode opcode
3. Read registers
4. Execute ALU op
5. Memory access (if load/store)
6. Write back result
7. PC += 4 (usually)

This is called the **fetch–decode–execute cycle**.

---

# 4. INSTRUCTION SET ARCHITECTURE (ISA)

ISA is the **contract** between hardware and software.

It defines:

* registers
* instructions
* memory model
* addressing modes
* privilege levels

Examples:

* x86-64 (CISC)
* ARMv8 (RISC)
* RISC-V

C code compiles to ISA instructions.

Example:

```c
int add(int a, int b) {
    return a + b;
}
```

Might become (RISC-V):

```
add a0, a0, a1
ret
```

CPU doesn’t know C.
It only sees **binary opcodes**.

---

# 5. PIPELINING — HOW REAL CPUs GET FAST

Single-cycle CPUs are slow.

So we pipeline:

Stages:

1. IF – Instruction Fetch
2. ID – Decode / Register Read
3. EX – Execute
4. MEM – Memory
5. WB – Write Back

While instruction A is in EX,
instruction B is in ID,
instruction C is in IF.

Like an assembly line.

---

## Pipeline Hazards

### 1. Structural

Two stages need same hardware.

### 2. Data

```
add r1, r2, r3
sub r4, r1, r5   // depends on r1
```

Solved with:

* forwarding/bypassing
* stalls

### 3. Control (branches)

Branch changes PC → pipeline flushed.

Huge performance killer.

---

# 6. BRANCH PREDICTION

Modern CPUs **guess** branches.

Predict:

* taken or not
* target address

Static: always taken
Dynamic: history tables

Two-bit saturating counter:

```
00 strongly not taken
01 weakly not taken
10 weakly taken
11 strongly taken
```

Wrong guess → flush pipeline → 15–20 cycles wasted.

---

# 7. SUPERSCALAR & OUT-OF-ORDER

Modern CPUs issue **multiple instructions per cycle**.

To do that:

* register renaming
* reservation stations
* reorder buffer (ROB)

### Register Renaming

Eliminates fake dependencies:

```
r1 = r2 + r3
r1 = r4 + r5
```

Rename to:

```
p7 = p2 + p3
p9 = p4 + p5
```

Physical registers ≠ architectural registers.

---

### Out-of-Order Execution

Instructions execute when operands are ready,
not strictly in program order.

But retirement happens **in order** using ROB.

---

# 8. CACHE HIERARCHY

RAM is slow.

So we use:

* L1 cache (~1–4 cycles)
* L2 (~10)
* L3 (~30–50)
* DRAM (~200+)

Caches are:

* line based (64 bytes typical)
* set associative

Address split:

```
| tag | index | offset |
```

---

### Cache Miss Types

* compulsory
* capacity
* conflict

---

# 9. CACHE COHERENCE (MULTICORE)

Each core has its own cache.

If core 0 writes memory, core 1 must see it.

Protocols like **MESI**:

* Modified
* Exclusive
* Shared
* Invalid

Hardware sends snoop messages.

---

# 10. VIRTUAL MEMORY & MMU

Every process thinks it owns all memory.

Virtual address → physical address via:

* page tables
* MMU
* TLB

Typical:

* page size: 4KB
* multi-level page tables

TLB caches translations.

Miss → page table walk → hundreds of cycles.

---

# 11. PRIVILEGE LEVELS

x86 rings:

* Ring 0 → kernel
* Ring 3 → user

ARM:

* EL0–EL3

Some instructions only legal in kernel:

* page table updates
* I/O
* halt

---

# 12. INTERRUPTS & CONTEXT SWITCHING

Interrupt:

* hardware saves PC/state
* jumps to handler
* kernel runs

Context switch:

* save registers
* save PC
* load new process state
* restore MMU mappings

Costs thousands of cycles.

---

# 13. SIMD / VECTOR UNITS

AVX, NEON, SSE.

Operate on many values at once.

C example:

```c
void add_arrays(float *a, float *b, float *c, int n) {
    for (int i = 0; i < n; i++)
        c[i] = a[i] + b[i];
}
```

Compiler may turn into vector loads/adds.

---

# 14. SPECULATION & SECURITY

CPUs speculate past branches.

If wrong → discard architectural results.

But microarchitectural state (cache) remains.

This led to:

* Spectre
* Meltdown

Timing attacks observe cache behavior.

---

# 15. POWER & FREQUENCY

Dynamic power:

```
P ≈ C × V² × f
```

So CPUs:

* scale voltage
* throttle frequency
* power-gate units

---

# 16. HOW C MAPS TO THE CPU

Consider:

```c
int sum(int *a, int n) {
    int s = 0;
    for (int i = 0; i < n; i++)
        s += a[i];
    return s;
}
```

CPU behavior:

* loads a[i]
* cache hit/miss?
* address translation
* pipeline fill
* branch prediction on loop
* forwarding s
* retirement in ROB

Performance depends more on **memory access** than arithmetic.

---

# REALITY CHECK

If you truly want mastery:

You must also study:

* digital design
* Verilog/VHDL
* computer architecture textbooks:

  * *Hennessy & Patterson*
  * *Computer Organization and Design*
* read Intel/ARM manuals
* write simulators
* build CPUs in HDL








# 🔴 PART 1 — DIGITAL FOUNDATIONS (NO ESCAPE)

Before pipelines, caches, or branch predictors:

**CPU = synchronous digital machine.**

Everything depends on:

* Boolean algebra
* Combinational logic
* Sequential logic
* Clocking

---

## 1.1 Bits, Voltage, and Switching

Transistors act as switches.

* 0 ≈ low voltage
* 1 ≈ high voltage

A gate is just a transistor network.

Example truth table for AND:

| A | B | OUT |
| - | - | --- |
| 0 | 0 | 0   |
| 0 | 1 | 0   |
| 1 | 0 | 0   |
| 1 | 1 | 1   |

---

## 1.2 Combinational vs Sequential

**Combinational**: output = function(inputs)

Example: adder, mux.

**Sequential**: remembers state.

Built from **flip-flops** or latches.

---

## 1.3 Flip-Flop (D-type)

At rising clock edge:

```
Q <= D;
```

That is *literally* how registers work.

---

## 1.4 Multiplexers (MUX)

Select between inputs.

2:1 MUX:

```
out = sel ? b : a;
```

Hardware uses this everywhere:

* register write-back
* ALU input select
* PC source

---

## 1.5 Building an ALU

An ALU is:

* add/sub
* AND/OR/XOR
* shifts
* comparisons

Add = ripple carry or carry-lookahead.

Sub = add two’s complement.

---

## 1.6 A C MODEL OF A HARDWARE ADDER

This is NOT fast software—it is **behavioral model** of hardware:

```c
#include <stdint.h>

uint64_t ripple_add(uint64_t a, uint64_t b) {
    uint64_t sum = 0;
    uint64_t carry = 0;

    for (int i = 0; i < 64; i++) {
        uint64_t abit = (a >> i) & 1;
        uint64_t bbit = (b >> i) & 1;

        uint64_t s = abit ^ bbit ^ carry;
        carry = (abit & bbit) | (carry & (abit ^ bbit));

        sum |= (s << i);
    }

    return sum;
}
```

That loop is conceptually **64 full adders chained**.

---

## 1.7 Register File

A register file has:

* N registers
* M read ports
* K write ports

Example: 32 regs, 2 read, 1 write.

Hardware decodes index → selects word.

---

## 1.8 Timing Reality

Between clocks:

* combinational logic settles
* signals propagate

Clock must be slow enough:

```
Tclock ≥ Tlogic + Tsetup + Tskew
```

This defines max GHz.

---

# 🔵 PART 2 — SINGLE-CYCLE DATAPATH

Now we assemble a CPU.

Components:

* PC register
* Instruction memory
* Decoder
* Register file
* ALU
* Data memory
* Writeback mux
* PC mux

Flow:

```
PC → IMEM → decode
             ↓
        RegFile → ALU → DMEM → WB
```

Every instruction finishes in ONE cycle.

Slow → clock dictated by worst-case load instruction.

---

## Example C MODEL of a Tiny CPU

This is a software simulator for a toy ISA:

```c
#include <stdint.h>

#define MEM_SIZE 65536
uint32_t mem[MEM_SIZE];
uint32_t regs[32];
uint32_t pc;

enum { OP_ADD=1, OP_LOAD=2, OP_STORE=3, OP_JMP=4 };

void step() {
    uint32_t inst = mem[pc >> 2];
    pc += 4;

    uint32_t op  = inst >> 28;
    uint32_t rd  = (inst >> 23) & 31;
    uint32_t rs1 = (inst >> 18) & 31;
    uint32_t rs2 = (inst >> 13) & 31;
    uint32_t imm = inst & 0x1FFF;

    switch (op) {
        case OP_ADD:
            regs[rd] = regs[rs1] + regs[rs2];
            break;

        case OP_LOAD:
            regs[rd] = mem[(regs[rs1] + imm) >> 2];
            break;

        case OP_STORE:
            mem[(regs[rs1] + imm) >> 2] = regs[rs2];
            break;

        case OP_JMP:
            pc = regs[rs1] + imm;
            break;
    }
}
```

That is literally the **architectural** view.

---

# 🔥 THIS IS ONLY THE START

We have covered:

✅ logic
✅ adders
✅ registers
✅ muxes
✅ timing
✅ single-cycle CPU

---


