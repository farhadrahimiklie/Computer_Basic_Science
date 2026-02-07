# 🔥 PART 1 — Why Cache Exists

CPU cores are insanely fast.

Modern CPUs:

* ~3 GHz → 3 **billion** cycles/sec
* One cycle ≈ 0.3 ns

Main memory (DRAM):

* ~80–150 ns latency

That’s **250–500 CPU cycles** just waiting.

Without cache:
CPU would spend most of its life **doing nothing**.

---

### Memory Hierarchy

```
Registers   (1 cycle)
L1 Cache    (~3-5 cycles)
L2 Cache    (~10-15)
L3 Cache    (~30-60)
RAM         (~100+)
Disk        (millions)
```

Closer = faster + smaller + expensive.

---

# 🔥 PART 2 — What Is Cache?

Cache is:

> **Small, very fast memory that stores recently used memory blocks.**

Not bytes.
**BLOCKS.** Called **cache lines**.

Typical cache line:

```
64 bytes
```

If CPU loads:

```
int x = a[100];
```

It doesn't fetch just that int.

It fetches:
➡️ the **entire 64‑byte line** containing it.

This is because programs usually exhibit:

### 1️⃣ Temporal Locality

If you used it now → you'll likely use it again soon.

### 2️⃣ Spatial Locality

If you used address X → you’ll soon use X+1, X+2…

---

# 🔥 PART 3 — Cache Structure

A cache is NOT an array of bytes.

It is:

```
[ TAG | DATA BLOCK | VALID | DIRTY ]
```

Each entry:

* **TAG** → which memory block this is
* **DATA** → 64 bytes
* **VALID** → contains real data?
* **DIRTY** → modified?

---

# 🔥 PART 4 — Address Breakdown

A 64‑bit memory address is split:

```
| TAG | INDEX | OFFSET |
```

* OFFSET → which byte inside 64‑byte line (6 bits)
* INDEX → which set
* TAG → identifies the block

---

Example:

Cache:

* 32 KB L1
* 64B line
* 8‑way associative

Lines:

```
32 KB / 64 B = 512 lines
```

Sets:

```
512 / 8 = 64 sets
```

Index bits:

```
log2(64) = 6 bits
```

Offset:

```
log2(64) = 6 bits
```

Remaining bits → TAG.

---

# 🔥 PART 5 — Mapping Types

How memory maps to cache.

---

## 1️⃣ Direct‑Mapped

Each block → exactly one location.

Fast.
But collisions are brutal.

Two addresses that map to same index constantly evict each other.

---

## 2️⃣ Fully Associative

Any block anywhere.

Very flexible.
Expensive hardware.
Rare for large caches.

---

## 3️⃣ Set‑Associative (most common)

Each set has N ways.

8‑way = 8 possible places per index.

---

# 🔥 PART 6 — Replacement Policies

When set full, who gets kicked?

* **LRU** — least recently used
* **Pseudo‑LRU**
* **Random**
* **FIFO**

Real CPUs approximate LRU.

---

# 🔥 PART 7 — Write Policies

When CPU writes:

---

## Write‑Through

Immediately write to RAM.

Simple.
Slow.

---

## Write‑Back

Only update cache.
Mark line DIRTY.
Write back later when evicted.

Modern CPUs use this.

---

## Write Allocate

On write miss → load block then write.

## No Write Allocate

Write straight to memory.

---

# 🔥 PART 8 — Multicore = Cache Coherence

Each core has its own L1/L2.

What if two cores touch same variable?

💥 Inconsistency.

Solved via **MESI protocol**:

States:

* **M**odified
* **E**xclusive
* **S**hared
* **I**nvalid

Hardware snoops buses to keep caches synchronized.

---

# 🔥 PART 9 — False Sharing (VERY IMPORTANT)

Two threads:

```
struct Data {
    int a;
    int b;
};
```

Thread 1 writes `a`.
Thread 2 writes `b`.

Same cache line → cores invalidate each other constantly.

Even though variables differ.

This DESTROYS performance.

---

Fix:

```
struct Data {
    int a;
    char pad[60];
    int b;
};
```

Force into different cache lines.

---

# 🔥 PART 10 — Cache Miss Types

* **Cold miss** — first access
* **Capacity miss** — cache too small
* **Conflict miss** — mapping collisions

---

# 🔥 PART 11 — Prefetching

CPU guesses future accesses and loads early.

Also manual:

```c
__builtin_prefetch(ptr + 64);
```

GCC/Clang.

---

# 🔥 PART 12 — TLB ≠ Cache

TLB caches **page table translations**, not data.

Virtual → Physical mapping.

TLB miss = expensive.

Huge pages help.

---

# 🔥 PART 13 — Measuring Cache Behavior

Simple benchmark:

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define N (1024 * 1024 * 64)

int main() {
    int *arr = malloc(sizeof(int) * N);

    volatile long long sum = 0;

    clock_t start = clock();

    for (long long i = 0; i < N; i++)
        sum += arr[i];

    clock_t end = clock();

    printf("Time: %f\n",
           (double)(end - start) / CLOCKS_PER_SEC);

    free(arr);
}
```

Change stride:

```c
for (long long i = 0; i < N; i += 16)
```

Watch speed collapse once exceeding cache.

---

# 🔥 PART 14 — Cache‑Friendly Programming

---

## ❌ BAD

```c
int a[4096][4096];

for (int j = 0; j < 4096; j++)
    for (int i = 0; i < 4096; i++)
        sum += a[i][j];
```

Column‑wise → stride of 4096 ints.

Cache miss city.

---

## ✅ GOOD

```c
for (int i = 0; i < 4096; i++)
    for (int j = 0; j < 4096; j++)
        sum += a[i][j];
```

Row‑wise contiguous.

---

---

# 🔥 PART 15 — Blocking / Tiling

For matrix multiply:

---

## ❌ Naive

```c
for (int i = 0; i < N; i++)
  for (int j = 0; j < N; j++)
    for (int k = 0; k < N; k++)
      C[i][j] += A[i][k] * B[k][j];
```

---

## ✅ Cache‑Blocked

```c
#define BS 64

for (int ii = 0; ii < N; ii += BS)
  for (int jj = 0; jj < N; jj += BS)
    for (int kk = 0; kk < N; kk += BS)

      for (int i = ii; i < ii + BS; i++)
        for (int j = jj; j < jj + BS; j++)
          for (int k = kk; k < kk + BS; k++)
            C[i][j] += A[i][k] * B[k][j];
```

This is **real systems‑level optimization**.

---

# 🔥 PART 16 — How to THINK Like Cache

When writing code ask:

✔ Is memory contiguous?
✔ Am I reusing data soon?
✔ Do threads touch same lines?
✔ Is working set bigger than L1/L2?
✔ Are strides multiples of cache size?
✔ Am I thrashing sets?

---

---

# 🎯 MASTER LEVEL SUMMARY

Cache is:

• Hardware‑managed
• Line‑based
• Set‑associative
• Coherent
• Speculative (prefetching)
• Write‑back
• Critical to performance

If you understand cache, you understand:

➡️ why databases design pages
➡️ why games use SoA layouts
➡️ why compilers reorder loops
➡️ why parallel programs stall
➡️ why NUMA matters
➡️ why HPC exists



