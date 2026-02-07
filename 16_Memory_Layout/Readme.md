

# **Mastering Memory Layout in C**

When we talk about **memory layout in C**, we are referring to **how a program's memory is organized at runtime**. Understanding this is critical for performance, debugging, security, and low-level programming.

A C program's memory is typically divided into several regions:

1. **Text (Code) Segment**
2. **Data Segment (Initialized and Uninitialized)**
3. **BSS Segment**
4. **Heap**
5. **Stack**
6. **Memory-mapped region (optional, OS-dependent)**

Let's go **one by one**, ruthlessly detailed.

---

## **1. Text (Code) Segment**

* This is where the **compiled machine code** of your program lives.
* It is **read-only** in modern OSes (prevents accidental modification).
* Contains all your **functions**, including `main()`.
* Shared among processes if the program is loaded multiple times (saves memory).

**Example:**

```c
#include <stdio.h>

void func() {
    printf("Hello from code segment!\n");
}

int main() {
    func();
    return 0;
}
```

* The **machine instructions** for `func()` and `main()` live in the **text segment**.
* You **cannot safely modify them** at runtime (doing so can cause segmentation faults).

---

## **2. Data Segment**

The **data segment** is split into two:

### **a. Initialized Data Segment**

* Stores **global and static variables that are explicitly initialized**.
* Example:

```c
#include <stdio.h>

int global_var = 100;      // Initialized global
static int static_var = 200; // Initialized static

int main() {
    printf("%d %d\n", global_var, static_var);
    return 0;
}
```

* `global_var` and `static_var` live here.
* Memory is **persistent for the lifetime of the program**.
* Read/write permissions exist (you can change values).

### **b. Uninitialized Data Segment (BSS)**

* **BSS = Block Started by Symbol**
* Stores **global and static variables initialized to 0 or not initialized at all**.
* Example:

```c
#include <stdio.h>

int global_uninit;        // Default 0
static int static_uninit; // Default 0

int main() {
    printf("%d %d\n", global_uninit, static_uninit);
    return 0;
}
```

* The compiler ensures these variables **start with zero at runtime**.
* Memory is zeroed out by the OS.

**Key difference:**

* Data segment → initialized globals/statics
* BSS segment → uninitialized globals/statics (or zeroed)

---

## **3. Heap**

* **Dynamic memory allocation** lives here (`malloc`, `calloc`, `realloc`, `free`).
* Grows **upwards** (toward higher memory addresses) as you allocate memory.
* Managed manually by the programmer.
* Example:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *arr = (int*)malloc(5 * sizeof(int));
    for (int i = 0; i < 5; i++) arr[i] = i * 10;

    for (int i = 0; i < 5; i++) printf("%d ", arr[i]);
    free(arr);
    return 0;
}
```

* `arr` is a **pointer stored on the stack**, but the memory it points to is on the **heap**.
* Heap memory is **persistent until freed**.
* If you forget `free()`, you cause **memory leaks**.
* Heap fragmentation is a concern in long-running programs.

---

## **4. Stack**

* Stores **function call information**, including:

  * Local variables
  * Function parameters
  * Return addresses
  * CPU registers (saved context)
* Grows **downwards** (toward lower memory addresses) on most architectures.
* Managed automatically (LIFO).
* Limited size (usually a few MBs).

Example:

```c
#include <stdio.h>

void func() {
    int local = 10;   // Lives on stack
    printf("%p\n", &local);
}

int main() {
    func();
    return 0;
}
```

* Each function call creates a **stack frame**.
* Exiting the function **destroys its frame**.
* Recursive functions can **cause stack overflow** if too deep.

---

## **5. Memory Layout Map**

Putting it all together (simplified):

```
High Memory Addresses
---------------------------
|     Stack               |
|  (grows downward)       |
---------------------------
|     Heap                |
|  (grows upward)         |
---------------------------
|     BSS                 |
|  (uninitialized globals)|
---------------------------
|     Data                |
|  (initialized globals)  |
---------------------------
|     Text/Code           |
|  (instructions)         |
---------------------------
Low Memory Addresses
```

* **OS may add memory-mapped region** for shared libraries, etc.
* **Pointers on the stack** can reference heap memory or data segment.

---

## **6. Static vs Automatic Variables**

| Feature        | Static                     | Automatic (local)        |
| -------------- | -------------------------- | ------------------------ |
| Lifetime       | Whole program              | Function call only       |
| Storage        | Data/BSS segment           | Stack                    |
| Initialization | Default 0 if uninitialized | Garbage (stack)          |
| Example        | `static int x;`            | `int y;` inside function |

---

## **7. Code Example to Explore Memory Layout**

We can **print memory addresses** to see the layout:

```c
#include <stdio.h>
#include <stdlib.h>

int global_init = 10;
int global_uninit;

void func() {
    static int static_var = 20;
    int local_var = 30;
    int *heap_var = (int*)malloc(sizeof(int));
    *heap_var = 40;

    printf("Text (func address)   : %p\n", func);
    printf("Initialized global    : %p\n", &global_init);
    printf("Uninitialized global  : %p\n", &global_uninit);
    printf("Static local          : %p\n", &static_var);
    printf("Stack local           : %p\n", &local_var);
    printf("Heap memory           : %p\n", heap_var);

    free(heap_var);
}

int main() {
    func();
    return 0;
}
```

When you run this, you typically see:

```
Text (func address)   : 0x4005d6
Initialized global    : 0x60104c
Uninitialized global  : 0x601050
Static local          : 0x601048
Stack local           : 0x7ffc1234
Heap memory           : 0x602000
```

Notice:

* **Stack addresses are higher than heap** in most Linux/x86 systems.
* **Static and global variables are near each other**.
* **Text/code addresses are far below**.

---

## **8. Why Understanding Memory Layout Matters**

1. **Performance**

   * Local variables on stack → fast.
   * Heap allocations → slower.
   * Cache locality matters.

2. **Debugging**

   * Stack overflow, use-after-free, segmentation faults.

3. **Security**

   * Buffer overflows exploit stack layout.
   * Proper understanding prevents vulnerabilities.

4. **Embedded / Low-level Programming**

   * Microcontrollers often have **fixed memory segments**.
   * Knowing memory layout is critical.



