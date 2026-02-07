

# **Mastering Branch Prediction**

Branch prediction is **one of the most critical topics for performance optimization in modern CPUs**. CPUs execute instructions **much faster than memory can supply them**, so pipelines are used to keep execution units busy. A pipeline is like an assembly line: if one stage stalls, everything behind it waits. Branches (like `if`, `for`, `while`) **can disrupt this pipeline**, because the CPU doesn’t always know which path will be taken.

Branch prediction **guesses the outcome** of a branch **before it is fully resolved**, allowing the CPU to continue fetching and executing instructions. If the guess is correct, execution is fast. If wrong, the pipeline must **flush**, which wastes cycles.

---

## **1. Why Branch Prediction is Important**

* Modern CPUs have **deep pipelines** (15-30+ stages in Intel CPUs).
* Branch mispredictions **cost a lot**: can stall the CPU for 15–20 cycles or more.
* Without branch prediction, **most conditional code would halve CPU throughput**.
* In tight loops or `if-else` chains, **accurate branch prediction can mean the difference between 1x and 10x performance**.

### Example:

```c
for (int i = 0; i < 1000000; i++) {
    if (i % 2 == 0) {
        sum += i;
    }
}
```

* Without prediction: CPU has to **wait to know `i % 2 == 0`**.
* With prediction: CPU guesses even/odd, keeps pipeline running.

---

## **2. How Branch Prediction Works**

### **2.1 Static Prediction**

The CPU guesses **before runtime**, based on fixed rules:

* **Always Taken**: Assume the branch will always happen. (`if (condition)` is true)
* **Always Not Taken**: Assume the branch will not happen.
* **Backward Taken, Forward Not Taken**: Loops often go backward (`i--` or `i++`). Forward branches are usually if-conditions, less frequent.

**Pros:** Simple, no extra memory needed.
**Cons:** Wrong often in complex, dynamic code.

---

### **2.2 Dynamic Prediction**

The CPU tracks **branch history at runtime** and predicts based on **patterns**.

**Components:**

1. **Branch Target Buffer (BTB)**: Stores **address of the branch and where it goes**.
2. **Pattern History Table (PHT)**: Stores **past outcomes** of branches.
3. **Global/Local History Registers**: Track **branch outcomes** globally (all branches) or locally (per branch).

**How it works:**

* CPU sees branch at address `0x400123`.
* Checks BTB: "I remember this branch; last time it was taken/not taken".
* Uses PHT: predicts outcome based on **pattern**, e.g., `T-T-N-T` → likely **taken**.
* Executes predicted path.

---

## **3. Types of Dynamic Predictors**

### **3.1 1-bit Predictor**

* Stores **last outcome** (taken/not taken).
* Predicts the same next time.
* Problem: loops may mispredict at **loop exit**.

Example:

```c
for (int i = 0; i < 10; i++) {
    if (i < 9) // Taken 9 times, not taken last time
        sum += i;
}
```

* 1-bit predicts **taken** every time → mispredicts **last iteration**.

---

### **3.2 2-bit Predictor**

* Uses **two-bit saturating counter**:

  * 00 → Strongly Not Taken
  * 01 → Weakly Not Taken
  * 10 → Weakly Taken
  * 11 → Strongly Taken
* Requires **two wrong predictions to switch** state.
* Solves the **loop misprediction problem**.

**C-like representation:**

```c
typedef enum {SN=0, WN=1, WT=2, ST=3} PredictorState;

PredictorState update(PredictorState state, int taken) {
    switch (state) {
        case SN: return taken ? WN : SN;
        case WN: return taken ? WT : SN;
        case WT: return taken ? ST : WN;
        case ST: return taken ? ST : WT;
    }
    return SN;
}
```

---

### **3.3 Two-Level Predictors**

* Uses **local and global history**.
* **Local:** tracks **individual branch history**.
* **Global:** tracks **pattern across multiple branches**.
* Can detect complex patterns like `TNTTNT` for specific loops.

---

### **3.4 Tournament Predictors**

* Combines multiple predictors (local + global).
* Chooses **which predictor is likely more accurate**.
* Used in **modern Intel/AMD CPUs**.

---

## **4. Branch Prediction in C**

Let's make a **C simulation** of a 2-bit predictor:

```c
#include <stdio.h>
#include <stdlib.h>

typedef enum {SN=0, WN=1, WT=2, ST=3} PredictorState;

PredictorState update(PredictorState state, int taken) {
    switch (state) {
        case SN: return taken ? WN : SN;
        case WN: return taken ? WT : SN;
        case WT: return taken ? ST : WN;
        case ST: return taken ? ST : WT;
    }
    return SN;
}

int predict(PredictorState state) {
    return (state >= WT) ? 1 : 0;
}

int main() {
    PredictorState state = ST; // Strongly Taken
    int branch_outcomes[] = {1,1,1,0,1,0,0,1,1,1};
    int mispredictions = 0;

    for(int i=0; i<10; i++) {
        int prediction = predict(state);
        int actual = branch_outcomes[i];
        if(prediction != actual) mispredictions++;
        state = update(state, actual);
        printf("Step %d: Prediction=%d, Actual=%d, State=%d\n", i, prediction, actual, state);
    }

    printf("Total Mispredictions = %d\n", mispredictions);
    return 0;
}
```

This simulates a **2-bit branch predictor**, showing how predictions adapt.

---

## **5. CPU Optimizations & Tricks**

* **Loop unrolling**: fewer branches → fewer mispredictions.
* **Branchless code**: use ternary operators or bitwise tricks.

Example: Replace:

```c
if (x > y) max = x; else max = y;
```

With branchless:

```c
max = x ^ ((x ^ y) & -(x < y));
```

* **Profile-guided optimization**: compiler rearranges code based on runtime statistics.
* Modern CPUs use **speculative execution** + prediction → even out-of-order instructions can execute ahead.

---

## **6. Pipeline and Misprediction Costs**

| CPU Stage | Misprediction Effect                     |
| --------- | ---------------------------------------- |
| Fetch     | Fetch wrong instructions → wasted cycles |
| Decode    | Decoding useless instructions            |
| Execute   | Flush arithmetic/logic instructions      |
| Memory    | Wasted cache/memory accesses             |
| Writeback | Overwrites canceled                      |

> On modern Intel CPUs, a misprediction can cost **15–20 cycles**, sometimes more. Deep pipelines amplify the penalty.

---

## **7. Real-World Example**

```c
for (int i=0; i<1000000; i++) {
    if (i % 2 == 0) sum += i;   // predictable
    if (rand() % 2) sum -= 1;   // unpredictable → misprediction risk
}
```

* **Even/odd check** → CPU predicts **always taken/not taken** → almost zero misprediction.
* **rand()%2** → unpredictable → CPU mispredicts → pipeline flush → performance drops.

---

✅ **Key Takeaways to Master Branch Prediction**

1. Understand **pipeline stalls** and why branches matter.
2. Learn **static vs dynamic prediction**.
3. Implement **1-bit and 2-bit predictors** in C.
4. Recognize **loop and pattern behaviors** that affect predictors.
5. Optimize **branch-heavy code** with loop unrolling, branchless techniques, or profile-guided optimization.
6. Study **modern CPU predictor designs**: tournament, global/local, BTB/PHT.
7. Measure real misprediction rates with **tools like `perf` on Linux**.

