

# **Part 1: Undefined Behavior (UB) in C**

## **1. What is Undefined Behavior?**

In C, **Undefined Behavior** is when the C standard **does not prescribe what should happen** for a particular piece of code. The compiler is free to do anything: crash, silently miscompute, or even appear correct.

Think of UB like a “black hole” in your program: the moment you step in, all bets are off.

**Formally:**

> “The behavior of a program is undefined if the C standard imposes no requirements on what happens.” — ISO C Standard

**Key point:** UB ≠ compiler bug. UB is legal C, but *the standard does not define what should happen*.

---

## **2. Why Undefined Behavior Exists**

1. **Performance**: Compilers can optimize aggressively without checking for safety.
2. **Hardware abstraction**: Some operations might not exist on all hardware.
3. **Historical reasons**: Early C was closer to hardware; some operations had unpredictable outcomes.

---

## **3. Common Sources of Undefined Behavior**

### **a. Memory Access Issues**

1. **Dereferencing NULL or invalid pointers**

```c
int *p = NULL;
printf("%d\n", *p); // UB
```

2. **Out-of-bounds array access**

```c
int arr[3] = {1,2,3};
printf("%d\n", arr[5]); // UB
```

### **b. Uninitialized Variables**

```c
int x;
printf("%d\n", x); // UB (value is indeterminate)
```

### **c. Signed Integer Overflow**

```c
int a = INT_MAX;
int b = a + 1; // UB
```

> **Important**: Unsigned overflow is **well-defined** (wraps around), but signed overflow is UB.

### **d. Type-Punning / Strict Aliasing Violation**

```c
float f = 3.14;
int *p = (int*)&f;
printf("%d\n", *p); // UB due to strict aliasing
```

* C assumes objects of different types do not alias.
* Violating this gives UB; compilers can reorder or eliminate code based on this assumption.

### **e. Modifying `const` objects**

```c
const int x = 10;
int *p = (int*)&x;
*p = 20; // UB
```

### **f. Use After Free**

```c
int *p = malloc(sizeof(int));
free(p);
*p = 5; // UB
```

### **g. Multiple Modifications Without Sequence Point**

```c
int i = 0;
i = i++ + 1; // UB, modifying i twice without sequence point
```

> In C11 and later, the term **“sequence points”** is replaced with **“sequenced before”**, but the principle is the same.

---

## **4. UB Examples in Practice**

* Crash (segfault)
* Silent wrong result
* Strange optimization effects:

```c
int a = 1, b = 2;
if (a > 0 && (b = 5 / (a-1))) {
    // UB might cause 'b' to stay 2 instead of triggering divide by zero
}
```

---

## **5. Detecting and Preventing UB**

1. **Compiler Warnings**:

```bash
gcc -Wall -Wextra -pedantic
```

2. **Sanitizers**:

* `-fsanitize=undefined` (UBSan)
* `-fsanitize=address` (ASan for memory issues)
* Example:

```bash
gcc -fsanitize=undefined -g test.c -o test
./test
```

3. **Code discipline**:

* Avoid signed overflow
* Always initialize variables
* Avoid dangling pointers
* Respect strict aliasing
* Bounds-check arrays

---

# **Part 2: Alignment in C**

Alignment is closely tied to UB because **misaligned accesses often cause UB on some hardware**.

## **1. What is Alignment?**

* Every type has an **alignment requirement**: memory addresses it can safely occupy.
* For example:

  * `int` might require **4-byte alignment** (addresses divisible by 4)
  * `double` might require **8-byte alignment**

> Accessing a variable at the wrong alignment may cause UB (on x86 it may work but be slow, on ARM it may crash).

---

## **2. Why Alignment Exists**

* CPU hardware is optimized for aligned accesses
* Misaligned accesses:

  * Might be slower (extra memory cycles)
  * Might be UB on strict architectures

---

## **3. Alignment Rules in C**

1. **Address must be multiple of type size or `alignof(type)`**

```c
#include <stdio.h>
#include <stdalign.h>

int main() {
    printf("Alignment of int: %zu\n", alignof(int));
    printf("Alignment of double: %zu\n", alignof(double));
}
```

2. **Structs and Padding**

```c
struct S {
    char c;
    int i;
};
```

* `sizeof(struct S)` might be 8, not 5, due to **padding**.
* Padding aligns `int i` to 4-byte boundary.

---

## **4. Misaligned Access UB Example**

```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>

int main() {
    char buf[10];
    int *p = (int*)(buf + 1); // misaligned
    *p = 42; // UB on strict architectures
}
```

* On x86: might work
* On ARM: may crash

---

## **5. Fixing Alignment Issues**

1. **Use proper types**

```c
struct S {
    char c;
    int i;
} __attribute__((aligned(4)));
```

2. **Use `memcpy` for type-punning** instead of casting pointers

```c
float f = 3.14;
int i;
memcpy(&i, &f, sizeof(i)); // safe
```

3. **Use `alignas` in C11**

```c
#include <stdalign.h>

alignas(16) int data; // force 16-byte alignment
```

---

## **6. Key UB & Alignment Takeaways**

* UB happens when **you break C rules**; alignment issues are one common source.
* Undefined behavior is **not predictable**—never rely on it.
* Alignment ensures the CPU accesses memory safely.
* On modern x86, some misaligned accesses “work” but **they are still UB** in C.
* Using tools like **UBSan** and **ASan** is essential for mastering this.

---

## **7. Combined Example of UB due to Alignment**

```c
#include <stdio.h>
#include <stdint.h>

struct __attribute__((packed)) Misaligned {
    char c;
    int i; // misaligned
};

int main() {
    struct Misaligned m;
    m.c = 'A';
    m.i = 42; // UB: i may not be aligned
    printf("%d\n", m.i);
}
```

* `__attribute__((packed))` removes padding → can cause UB.
* Proper approach: **don’t pack unless necessary**, or access via `memcpy`.

---

✅ **Mastery Tip**: If you truly want to master UB and alignment:

1. Write small programs that trigger UB in controlled ways.
2. Compile with sanitizers to see how compilers detect it.
3. Examine struct memory layouts with `sizeof()` and `alignof()`.
4. Understand the hardware—x86 vs ARM handle misaligned access differently.


