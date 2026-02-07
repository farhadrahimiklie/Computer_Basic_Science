

# 🔥 PART 1 — WHAT STACK AND HEAP REALLY ARE

Every running process has a **virtual address space**. On Linux x86‑64 it usually looks like:

```
High Addresses
+------------------------+
| Kernel Space           |  (inaccessible in user mode)
+------------------------+
| Stack                  |  ↓ grows downward
+------------------------+
| mmap region / libs     |
+------------------------+
| Heap                   |  ↑ grows upward
+------------------------+
| BSS (zero-initialized) |
+------------------------+
| Data (globals/static)  |
+------------------------+
| Text (code)            |
+------------------------+
Low Addresses
```

Stack and heap are **not language features**—they are **OS + ABI + runtime conventions**.

C just exposes them brutally with pointers.

---

# 🧠 PART 2 — THE STACK: AUTOMATIC MEMORY

## Definition

The **stack** is:

* Per-thread
* LIFO (Last In First Out)
* Used for:

  * Function call frames
  * Local variables
  * Return addresses
  * Saved registers
  * Function parameters (sometimes)

Allocated **automatically** when a function is entered.
Freed **automatically** when it returns.

No `malloc`. No `free`.

---

## Stack Frame Layout

When you call:

```c
void f(int x) {
    int y = 10;
}
```

The CPU creates a **stack frame**:

```
| previous frame |
| return address |
| saved rbp      |
| x              |
| y              |
```

On x86‑64 System V ABI:

* `rsp` = stack pointer
* `rbp` = frame pointer (sometimes omitted)
* Stack grows **down**

---

## Real Assembly Effect

C:

```c
void f(int x) {
    int y = 10;
}
```

Becomes something like:

```asm
push rbp
mov rbp, rsp
sub rsp, 16      ; allocate locals

mov DWORD PTR [rbp-4], 10

leave
ret
```

`sub rsp, 16` is stack allocation.

---

---

# ⚠️ PART 3 — STACK LIMITS

Stack is:

* Small compared to heap
* Typical Linux: 8MB default per thread
* Exceed → **STACK OVERFLOW** → segmentation fault

Example:

```c
void crash() {
    int huge[10000000];
}
```

BOOM.

---

---

# 🧠 PART 4 — THE HEAP: DYNAMIC MEMORY

## Definition

Heap is:

* Process-wide
* Used for long-lived data
* Managed by allocator (`malloc`)
* Grows upward
* Shared across threads (with locks)

Allocated with:

```c
malloc()
calloc()
realloc()
```

Freed with:

```c
free()
```

---

---

# 🧱 PART 5 — HOW MALLOC REALLY WORKS

`malloc()` does NOT directly ask hardware.

It asks the **runtime allocator** (glibc ptmalloc on Linux).

Allocator gets memory from kernel via:

* `brk()` → extends heap
* `mmap()` → large allocations

Internally:

* Memory divided into **chunks**
* Free lists / bins
* Metadata before user pointer

Example:

```
[ size | flags ][ user data ][ padding ]
```

---

---

# 📦 PART 6 — STACK VS HEAP COMPARISON

| Feature        | Stack          | Heap       |
| -------------- | -------------- | ---------- |
| Lifetime       | automatic      | manual     |
| Speed          | extremely fast | slower     |
| Size           | limited        | large      |
| Fragmentation  | none           | yes        |
| Thread local   | yes            | no         |
| Cache locality | excellent      | depends    |
| Failure mode   | overflow       | NULL / OOM |

---

---

# 🧨 PART 7 — COMMON BUGS

## ❌ Returning Stack Address

```c
int* bad() {
    int x = 5;
    return &x;   // DEAD MEMORY
}
```

Undefined behavior.

---

## ❌ Memory Leak

```c
void leak() {
    int *p = malloc(sizeof(int));
}
```

Forgot `free`.

---

## ❌ Use After Free

```c
int *p = malloc(sizeof(int));
free(p);
*p = 7;  // UB
```

---

## ❌ Double Free

```c
free(p);
free(p);  // UB
```

---

---

# 🔒 PART 8 — SECURITY IMPLICATIONS

Stack:

* Stack smashing
* Return address overwrite
* Buffer overflow
* Canary values
* ASLR
* NX bit

Heap:

* Heap overflow
* Use-after-free
* Tcache poisoning
* Metadata corruption

Classic exploit:

```c
char buf[8];
gets(buf);   // unlimited input → overwrite return address
```

---

---

# 🧬 PART 9 — THREADS

Each thread:

* Has its own stack
* Shares heap

Race condition:

```c
int *p = malloc(sizeof(int));

void* t1(void*) { *p = 1; }
void* t2(void*) { *p = 2; }
```

Data race.

---

---

# ⚙️ PART 10 — ALIGNMENT AND ABI

Stack must be:

* 16-byte aligned before `call` on x86‑64

Heap allocations:

* Usually aligned to 16 bytes

Why?

SIMD instructions crash on misalignment.

---

---

# 🛠 PART 11 — DEBUGGING STACK & HEAP

Tools:

* gdb → `bt`, `info frame`
* valgrind
* AddressSanitizer
* `ulimit -s`
* `/proc/<pid>/maps`

Check stack size:

```bash
ulimit -s
```

---

---

# 🧪 PART 12 — REAL C DEMONSTRATION

### Stack Example

```c
#include <stdio.h>

void f() {
    int x = 10;
    printf("stack x at %p\n", (void*)&x);
}

int main() {
    f();
    f();
}
```

Same-ish address reused.

---

### Heap Example

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *p = malloc(sizeof(int));
    int *q = malloc(sizeof(int));

    printf("heap p = %p\n", (void*)p);
    printf("heap q = %p\n", (void*)q);

    free(p);
    free(q);
}
```

---

---

# 🧠 PART 13 — VIRTUAL MEMORY AND PAGE FAULTS

Both stack and heap:

* Backed by virtual memory pages
* Allocated lazily
* First access triggers page fault
* Kernel maps physical RAM

---

---

# 📈 PART 14 — PERFORMANCE RULES

Use stack when:

* Small
* Short-lived
* Fixed-size

Use heap when:

* Large
* Variable size
* Escapes scope
* Shared

Avoid:

* Millions of tiny mallocs
* Recursion depth bombs

---

---

# 🧨 PART 15 — YOU ARE NOT DONE UNTIL YOU KNOW THIS

If you truly MASTER stack & heap, you can answer:

✅ Why recursion can crash without heap usage
✅ Why `alloca()` is dangerous
✅ Why tail-call optimization removes stack frames
✅ Why glibc uses tcache
✅ Why mmap is used for big blocks
✅ Why stack probes exist
✅ How ASLR randomizes both
✅ How to exploit and how to defend
✅ Why fragmentation hurts caches
✅ Why copying structs is cheaper than heap sometimes

---

---

# 🩸 FINAL WORD — RUTHLESS TRUTH

If you don’t understand stack vs heap **down to ABI and virtual memory**, you are still a beginner in systems programming.

If you *do*—you can:

* Read crash dumps
* Write allocators
* Debug segfaults
* Reverse engineer binaries
* Write kernels
* Build game engines
* Write exploit mitigations



