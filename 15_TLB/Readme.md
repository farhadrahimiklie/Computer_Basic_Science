## **1. What is a TLB?**

**TLB** stands for **Translation Lookaside Buffer**. It’s a **special high-speed cache** used by the CPU **to reduce the time taken to access virtual memory addresses**.

To understand TLB fully, you must understand **virtual memory**:

* Modern operating systems use **virtual memory** to give each process its own address space.
* Memory addresses used by programs are **virtual addresses**.
* CPU needs to convert these virtual addresses to **physical addresses** in RAM.
* This translation is done using a **page table**.

But **accessing page tables in main memory for every memory reference is slow**. This is where the **TLB** comes in:

> TLB is a **small cache in the CPU** that stores **recent virtual-to-physical address translations**, so the CPU doesn’t have to go to memory for every translation.

---

## **2. Anatomy of a TLB**

A TLB stores:

1. **Virtual Page Number (VPN)** – the high-order bits of the virtual address.
2. **Physical Frame Number (PFN)** – the physical address mapping.
3. **Valid bit** – indicates if the entry is valid.
4. **Access permissions** – e.g., read, write, execute.
5. **Dirty/Reference bits** (sometimes) – for replacement policies.

**Example:**

If you have a 32-bit virtual address with 4KB pages:

* Page size = 4KB → offset = 12 bits
* VPN = upper 20 bits
* TLB stores mappings from **VPN → PFN**

So, TLB entry might look like this:

| VPN   | PFN   | Valid | R/W/X | LRU counter |
| ----- | ----- | ----- | ----- | ----------- |
| 0x01A | 0x3F2 | 1     | RWX   | 5           |

---

## **3. How TLB Works**

Every memory access by CPU:

1. CPU generates **virtual address (VA)**.
2. CPU splits VA into:

   * **VPN** (Virtual Page Number)
   * **Page Offset**
3. CPU checks **TLB** for VPN:

   * **TLB Hit:** Return PFN instantly → form physical address → memory access
   * **TLB Miss:** CPU must walk **page table** → find PFN → update TLB → access memory

**Physical Address Calculation:**

```
Physical Address = PFN | Page Offset
```

---

## **4. TLB Hit & Miss**

* **TLB Hit:** Very fast, ~1 CPU cycle or two.
* **TLB Miss:** Expensive:

  1. Walk page table in memory (1-3 memory accesses for multi-level page tables)
  2. Load mapping into TLB
  3. Retry memory access

> CPU performance is heavily dependent on **TLB hit rate**.

**Example:**

* Memory access = 100 ns
* TLB access = 1 ns
* TLB hit rate = 99%

Average memory access time:

```
AMAT = (TLB_hit_rate * TLB_access + TLB_miss_rate * (TLB_access + page_table_walk + memory_access))
AMAT = (0.99 * 1) + (0.01 * (1 + 300 + 100)) = 0.99 + 4.01 = 5 ns
```

Without TLB: 100 ns per access → huge performance boost.

---

## **5. Types of TLB**

**1. Fully Associative TLB:**

* Any VPN can go into any slot.
* Fastest for small TLBs.
* Expensive hardware-wise.

**2. Set-Associative TLB:**

* Divided into sets, each set has multiple slots.
* VPN mapped to one set → search within set.
* Common in modern CPUs.

**3. Direct-Mapped TLB:**

* Each VPN maps to exactly one TLB entry.
* Simple, but high conflict misses.

---

## **6. TLB Replacement Policies**

When TLB is full, we need to **replace an entry**:

1. **LRU (Least Recently Used)** – replace the least recently used entry
2. **FIFO (First In, First Out)** – replace the oldest entry
3. **Random** – randomly pick an entry

> Most modern CPUs use **pseudo-LRU** because perfect LRU is expensive.

---

## **7. TLB & Page Table Levels**

* Modern CPUs have **multi-level page tables** (x86-64: 4 levels)
* Walking 4 levels → memory access slow (~400 ns)
* TLB avoids this 99% of the time if working set fits in TLB

**TLB types:**

* **Instruction TLB (iTLB)** – caches instruction fetch addresses
* **Data TLB (dTLB)** – caches data addresses
* Sometimes unified TLB (uTLB) handles both

---

## **8. TLB Shootdown**

* In multi-core CPUs, if one core changes page tables, other cores' TLBs may have stale entries → **TLB shootdown**.
* OS must invalidate entries on other cores → costly operation.

---

## **9. TLB Miss Handling in OS**

Steps on a **TLB miss**:

1. CPU traps → **TLB miss exception**
2. OS **page table walk** to find PFN
3. Load PFN into TLB
4. Retry memory access

---

## **10. Implementing a Simple TLB in C**

We can write a **simple software-simulated TLB** to understand it fully.
We’ll use **fully associative TLB** for simplicity.

```c
#include <stdio.h>
#include <stdlib.h>

#define TLB_SIZE 4
#define PAGE_TABLE_SIZE 16

typedef struct {
    int vpn;   // virtual page number
    int pfn;   // physical frame number
    int valid;
} TLBEntry;

typedef struct {
    int vpn;
    int pfn;
} PageTableEntry;

// Simple page table
PageTableEntry page_table[PAGE_TABLE_SIZE];

// TLB
TLBEntry tlb[TLB_SIZE];

// Simple counter for replacement (FIFO)
int tlb_index = 0;

// Initialize page table
void init_page_table() {
    for (int i = 0; i < PAGE_TABLE_SIZE; i++) {
        page_table[i].vpn = i;
        page_table[i].pfn = i + 100; // arbitrary PFN
    }
}

// Search TLB for VPN
int search_tlb(int vpn) {
    for (int i = 0; i < TLB_SIZE; i++) {
        if (tlb[i].valid && tlb[i].vpn == vpn) {
            return tlb[i].pfn; // TLB hit
        }
    }
    return -1; // TLB miss
}

// Add entry to TLB (FIFO)
void add_tlb(int vpn, int pfn) {
    tlb[tlb_index].vpn = vpn;
    tlb[tlb_index].pfn = pfn;
    tlb[tlb_index].valid = 1;
    tlb_index = (tlb_index + 1) % TLB_SIZE;
}

// Access memory
int access_memory(int vpn) {
    int pfn = search_tlb(vpn);
    if (pfn != -1) {
        printf("TLB Hit! VPN %d -> PFN %d\n", vpn, pfn);
        return pfn;
    }

    printf("TLB Miss! VPN %d -> Searching Page Table...\n", vpn);
    // Walk page table
    for (int i = 0; i < PAGE_TABLE_SIZE; i++) {
        if (page_table[i].vpn == vpn) {
            pfn = page_table[i].pfn;
            add_tlb(vpn, pfn); // Update TLB
            printf("Page Table Hit! Loaded VPN %d -> PFN %d into TLB\n", vpn, pfn);
            return pfn;
        }
    }
    printf("Page Fault! VPN %d not in memory\n", vpn);
    return -1;
}

int main() {
    init_page_table();

    int accesses[] = {1, 2, 3, 1, 4, 2, 5, 1};
    int n = sizeof(accesses)/sizeof(accesses[0]);

    for (int i = 0; i < n; i++) {
        access_memory(accesses[i]);
    }

    return 0;
}
```

**Explanation of the code:**

* We simulate a **TLB** and **page table**.
* TLB is **small** (4 entries), page table is bigger (16 entries).
* `search_tlb()` → checks if VPN is in TLB.
* `add_tlb()` → adds VPN→PFN mapping using **FIFO replacement**.
* `access_memory()` → simulates memory access and prints TLB hits/misses.

This is how the **hardware TLB works internally**, but in software.

---

## ✅ **11. Key Points to Master**

1. **TLB is all about speed**: caching translations.
2. **TLB hit rate matters**: high hit rate = huge performance boost.
3. **Replacement policy**: critical for efficiency.
4. **Hardware vs software TLB**:

   * Most CPUs: hardware-managed TLB (automatic)
   * Some OSs: software-managed TLB (like MIPS)
5. **Separate instruction/data TLB** in modern CPUs.
6. **TLB shootdown in multi-core systems** → OS coordination.


