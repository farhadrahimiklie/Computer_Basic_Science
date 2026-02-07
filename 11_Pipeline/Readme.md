## **1️⃣ What is a CPU Pipeline?**

Imagine a factory assembly line. Instead of building a car from scratch by one worker, each worker does **one step at a time** in parallel with others. The same principle applies to CPUs.

* **Without pipeline:**
  Each instruction completes **all steps before the next starts**.
  Time per instruction = (T_\text{instr} = T_\text{fetch} + T_\text{decode} + T_\text{execute} + T_\text{memory} + T_\text{writeback})

* **With pipeline:**
  Instructions are divided into **stages**, and multiple instructions are processed **overlapping in time**.

**Analogy:** Imagine 5-stage pipeline (classic RISC pipeline):

| Stage | Name               | Description                         |
| ----- | ------------------ | ----------------------------------- |
| IF    | Instruction Fetch  | Fetch instruction from memory       |
| ID    | Instruction Decode | Decode instruction & read registers |
| EX    | Execute            | Perform ALU operation               |
| MEM   | Memory Access      | Read/write memory if needed         |
| WB    | Write Back         | Write result back to register       |

**Pipeline diagram:**

```
Time →
Instr1: IF | ID | EX | MEM | WB
Instr2:    IF | ID | EX | MEM | WB
Instr3:       IF | ID | EX | MEM | WB
```

Notice how multiple instructions are in different stages **simultaneously**.

**Key advantage:** Throughput ↑ (more instructions/sec), **Latency of single instruction** may not decrease.

---

## **2️⃣ Pipeline Stages in Detail**

Let’s break down each stage like a surgeon would:

### **Stage 1: Instruction Fetch (IF)**

* CPU fetches **binary instruction** from memory (program counter points here).
* PC (Program Counter) increments to next instruction.

**C simulation snippet:**

```c
int fetch_instruction(int PC, int *memory) {
    return memory[PC]; // Fetch instruction from memory
}
```

### **Stage 2: Instruction Decode (ID)**

* Decode instruction type (ADD, SUB, LOAD, STORE, etc.)
* Determine source registers & destination register
* Generate **control signals** for other stages

**C simulation snippet:**

```c
typedef struct {
    int opcode;
    int src1, src2, dest;
} Instruction;

Instruction decode(int binary_instruction) {
    Instruction instr;
    instr.opcode = (binary_instruction >> 12) & 0xF; // Example
    instr.src1   = (binary_instruction >> 8) & 0xF;
    instr.src2   = (binary_instruction >> 4) & 0xF;
    instr.dest   = binary_instruction & 0xF;
    return instr;
}
```

### **Stage 3: Execute (EX)**

* Perform ALU operations
* Calculate memory address if LOAD/STORE

```c
int execute(Instruction instr, int *registers) {
    switch(instr.opcode) {
        case 0: return registers[instr.src1] + registers[instr.src2]; // ADD
        case 1: return registers[instr.src1] - registers[instr.src2]; // SUB
        // Other opcodes...
    }
    return 0;
}
```

### **Stage 4: Memory Access (MEM)**

* Only for instructions accessing memory
* LOAD: read memory
* STORE: write memory

```c
int memory_access(Instruction instr, int alu_result, int *memory, int *registers) {
    if(instr.opcode == 2) { // LOAD
        return memory[alu_result];
    } else if(instr.opcode == 3) { // STORE
        memory[alu_result] = registers[instr.src1];
    }
    return alu_result;
}
```

### **Stage 5: Write Back (WB)**

* Write result back to destination register

```c
void write_back(Instruction instr, int result, int *registers) {
    registers[instr.dest] = result;
}
```

---

## **3️⃣ Pipeline Hazards**

Pipelines are not magic. They create **hazards** that stall execution. There are **3 main types**:

1. **Structural Hazards**

   * When hardware cannot support all stages simultaneously (e.g., 1 memory port for IF & MEM stage)
   * Solution: Add hardware resources, or stall.

2. **Data Hazards**

   * When an instruction depends on the result of a previous instruction still in the pipeline
   * Example:

     ```asm
     ADD R1, R2, R3
     SUB R4, R1, R5  ; R1 not ready yet
     ```
   * Solutions:

     * Stall pipeline
     * Forwarding (bypass)

3. **Control Hazards**

   * Caused by branches/jumps
   * CPU doesn’t know next instruction until branch resolves
   * Solutions:

     * Branch prediction
     * Delay slots

---

## **4️⃣ Pipeline Performance**

* **Ideal CPI (cycles per instruction):** 1 (one instruction completes every cycle after pipeline is full)
* **Real CPI:** >1 due to hazards
* **Speedup:**

[
\text{Speedup} = \frac{\text{Time without pipeline}}{\text{Time with pipeline}}
]

* Pipeline **depth** matters:

  * Deeper pipeline → higher throughput, more hazard complexity
  * Shallow pipeline → less hazards, lower throughput






### **Step 2.1: Define Instruction and Registers**

```c
#include <stdio.h>
#include <stdlib.h>

#define NREG 16   // Number of registers
#define NMEM 256  // Memory size
#define NINS 10   // Number of instructions

// Opcodes
#define ADD 0
#define SUB 1
#define LOAD 2
#define STORE 3
#define NOP 4

typedef struct {
    int opcode;
    int src1;
    int src2;
    int dest;
    int immediate; // For load/store address offset
} Instruction;

// Pipeline register between stages
typedef struct {
    Instruction instr;
    int alu_result;
    int mem_data;
    int valid; // 1 if instruction is valid, 0 if bubble
} PipelineReg;
```

---

### **Step 2.2: Initialize Registers & Memory**

```c
int registers[NREG];
int memory[NMEM];
Instruction program[NINS];
```

* `registers[]` → holds CPU registers
* `memory[]` → simple memory model
* `program[]` → list of instructions

---

### **Step 2.3: Fetch Stage (IF)**

```c
void IF_stage(PipelineReg *IF_ID, int PC, Instruction *program) {
    if (PC < NINS) {
        IF_ID->instr = program[PC];
        IF_ID->valid = 1;
    } else {
        IF_ID->instr.opcode = NOP; // Insert NOP if program ends
        IF_ID->valid = 0;
    }
}
```

---

### **Step 2.4: Decode Stage (ID)**

```c
void ID_stage(PipelineReg *IF_ID, PipelineReg *ID_EX) {
    if (IF_ID->valid) {
        ID_EX->instr = IF_ID->instr;
        ID_EX->valid = 1;
    } else {
        ID_EX->instr.opcode = NOP;
        ID_EX->valid = 0;
    }
}
```

---

### **Step 2.5: Execute Stage (EX)**

```c
void EX_stage(PipelineReg *ID_EX, PipelineReg *EX_MEM, int *registers) {
    Instruction instr = ID_EX->instr;
    int result = 0;

    switch(instr.opcode) {
        case ADD: result = registers[instr.src1] + registers[instr.src2]; break;
        case SUB: result = registers[instr.src1] - registers[instr.src2]; break;
        case LOAD: result = registers[instr.src1] + instr.immediate; break; // address calc
        case STORE: result = registers[instr.src1] + instr.immediate; break; // address calc
        case NOP: break;
    }

    EX_MEM->instr = instr;
    EX_MEM->alu_result = result;
    EX_MEM->valid = ID_EX->valid;
}
```

---

### **Step 2.6: Memory Stage (MEM)**

```c
void MEM_stage(PipelineReg *EX_MEM, PipelineReg *MEM_WB, int *memory, int *registers) {
    Instruction instr = EX_MEM->instr;
    int mem_data = 0;

    if (!EX_MEM->valid) {
        MEM_WB->instr.opcode = NOP;
        MEM_WB->valid = 0;
        return;
    }

    switch(instr.opcode) {
        case LOAD:
            mem_data = memory[EX_MEM->alu_result];
            break;
        case STORE:
            memory[EX_MEM->alu_result] = registers[instr.dest];
            break;
        default:
            mem_data = EX_MEM->alu_result;
            break;
    }

    MEM_WB->instr = instr;
    MEM_WB->alu_result = EX_MEM->alu_result;
    MEM_WB->mem_data = mem_data;
    MEM_WB->valid = EX_MEM->valid;
}
```

---

### **Step 2.7: Write Back Stage (WB)**

```c
void WB_stage(PipelineReg *MEM_WB, int *registers) {
    if (!MEM_WB->valid) return;

    Instruction instr = MEM_WB->instr;

    switch(instr.opcode) {
        case ADD:
        case SUB:
            registers[instr.dest] = MEM_WB->alu_result;
            break;
        case LOAD:
            registers[instr.dest] = MEM_WB->mem_data;
            break;
        case STORE:
        case NOP:
            break;
    }
}
```

---

### **Step 2.8: Hazard Detection & Forwarding**

Here we will **do simple forwarding** for EX stage:

```c
void forward(PipelineReg *ID_EX, PipelineReg *EX_MEM, PipelineReg *MEM_WB, int *registers) {
    Instruction instr = ID_EX->instr;

    // Forward from MEM_WB to EX stage
    if (MEM_WB->valid) {
        if (MEM_WB->instr.dest == instr.src1) {
            registers[instr.src1] = (MEM_WB->instr.opcode == LOAD) ? MEM_WB->mem_data : MEM_WB->alu_result;
        }
        if (MEM_WB->instr.dest == instr.src2) {
            registers[instr.src2] = (MEM_WB->instr.opcode == LOAD) ? MEM_WB->mem_data : MEM_WB->alu_result;
        }
    }

    // Forward from EX_MEM to EX stage
    if (EX_MEM->valid) {
        if (EX_MEM->instr.dest == instr.src1) {
            registers[instr.src1] = EX_MEM->alu_result;
        }
        if (EX_MEM->instr.dest == instr.src2) {
            registers[instr.src2] = EX_MEM->alu_result;
        }
    }
}
```

> This handles **data hazards** using forwarding instead of stalling.

---

### **Step 2.9: Main Pipeline Loop**

```c
int main() {
    // Example program: ADD R1=R2+R3, SUB R4=R1-R5, LOAD R6=[R4+2], STORE R6->[R7+3]
    program[0] = (Instruction){ADD, 2, 3, 1, 0};
    program[1] = (Instruction){SUB, 1, 5, 4, 0};
    program[2] = (Instruction){LOAD, 4, 0, 6, 2};
    program[3] = (Instruction){STORE, 6, 0, 7, 3};
    program[4].opcode = NOP; program[5].opcode = NOP; // fill NOPs
    int PC = 0;

    PipelineReg IF_ID = {0}, ID_EX = {0}, EX_MEM = {0}, MEM_WB = {0};

    // Initialize registers & memory
    for (int i=0; i<NREG; i++) registers[i]=i;
    for (int i=0; i<NMEM; i++) memory[i]=i*10;

    int cycles = 0;
    while (PC < NINS || IF_ID.valid || ID_EX.valid || EX_MEM.valid || MEM_WB.valid) {
        cycles++;

        WB_stage(&MEM_WB, registers);
        MEM_stage(&EX_MEM, &MEM_WB, memory, registers);
        EX_stage(&ID_EX, &EX_MEM, registers);
        forward(&ID_EX, &EX_MEM, &MEM_WB, registers);
        ID_stage(&IF_ID, &ID_EX);
        IF_stage(&IF_ID, PC, program);

        PC++;

        // Print register state
        printf("Cycle %d: Registers: ", cycles);
        for(int i=0;i<NREG;i++) printf("%d ", registers[i]);
        printf("\n");
    }

    printf("Pipeline execution completed in %d cycles.\n", cycles);
    return 0;
}
```

---

✅ **What this does:**

* Implements a **5-stage pipeline** in C
* Handles **data hazards** using **forwarding**
* Executes a small program with **ADD, SUB, LOAD, STORE**
* Prints **register state every cycle**
* Stops when **all instructions are fully executed**


