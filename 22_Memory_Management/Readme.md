

# 🔥 PART 6 — Virtual Memory & How Allocation Really Works

Before touching `malloc`, you must understand **virtual memory**, because *all* heap allocation depends on it.

---

## 🧠 What Is Virtual Memory?

Your program does **not** see real RAM directly.

Instead, the OS gives each process its own **virtual address space**:

```
Process A:
0x0000 -----> 0x7fff_ffff

Process B:
0x0000 -----> 0x7fff_ffff
```

Both think they own the whole memory — but the OS maps these to real RAM using **page tables**.

### Why this is powerful:

✅ Isolation between programs
✅ Protection (no writing kernel memory)
✅ Allows swapping to disk
✅ Simplifies programming

---

## 📄 Memory Pages

Memory is divided into fixed-size blocks called **pages**.

Typical size:
👉 **4 KB** (sometimes 2MB huge pages)

Virtual memory works at **page granularity**.

Heap memory is built from pages.

---

## 🔍 When You Call `malloc`

In C:

```c
int *p = malloc(100);
```

This does NOT usually go straight to RAM.

Instead:

1️⃣ `malloc` asks the runtime allocator
2️⃣ Allocator checks its free lists
3️⃣ If not enough space → asks OS for more pages
4️⃣ OS maps pages into your process
5️⃣ Allocator carves a chunk for you

---

## 🧩 OS Interfaces

Under Linux:

* `brk()` / `sbrk()` → grow heap
* `mmap()` → map pages directly

Modern allocators mostly use **mmap** for large blocks.

---

## 📊 Simplified Heap View

```
Heap:
+--------+--------+--------+
| block  | free   | block  |
+--------+--------+--------+
```

Allocator keeps metadata around blocks:

```
[ size | used/free ][ user memory ][ footer? ]
```

---

# ✅ PART 7 — malloc / calloc / realloc / free

Let’s study each.

---

## 🔵 malloc

```c
void* malloc(size_t size);
```

* Allocates `size` bytes
* Uninitialized
* Returns NULL on failure

Example:

```c
int *arr = malloc(10 * sizeof(int));
if (!arr) {
    perror("malloc failed");
    exit(1);
}
```

---

## 🟢 calloc

```c
void* calloc(size_t n, size_t size);
```

* Allocates `n * size`
* Initializes to **zero**
* Slower than malloc

---

## 🟡 realloc

```c
void* realloc(void *ptr, size_t new_size);
```

* Resize block
* May move memory
* Old pointer invalid if moved

Correct pattern:

```c
int *tmp = realloc(arr, new_size);
if (!tmp) {
    // arr still valid!
} else {
    arr = tmp;
}
```

---

## 🔴 free

```c
free(ptr);
```

* Releases memory to allocator
* NOT necessarily to OS
* After free → pointer is invalid

Good habit:

```c
free(p);
p = NULL;
```

---

# ⚠️ PART 8 — Alignment

Allocators must return memory aligned for the CPU.

If misaligned → crash or slow.

Example:

* `int` → 4-byte aligned
* `double` → 8-byte aligned

`malloc` guarantees suitable alignment for any type.

---

# ⚠️ PART 9 — Fragmentation

Two kinds:

### 🔹 External fragmentation

Free memory split into small pieces.

### 🔹 Internal fragmentation

You get more than requested.

Example:

Request 13 bytes → allocator gives 16.

---

# 🧠 Mental Model So Far

```
OS pages -> allocator -> blocks -> your pointer
```

---

# 🧪 Exercise

What’s wrong here?

```c
int *p = malloc(sizeof(int));
free(p);
*p = 5;
```

Answer: **use-after-free** ❌


