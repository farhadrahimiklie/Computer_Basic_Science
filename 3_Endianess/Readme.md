

# 1. What Is Endianness — Precisely?

**Endianness is the byte ordering of multi-byte data in memory.**

Key phrase: **multi-byte**.

If you store:

```
0x12345678
```

That is a 32-bit integer = 4 bytes:

```
12 34 56 78   (hex bytes)
```

Endianness defines:

> Which byte goes at the lowest memory address?

---

## Little-endian

Lowest address contains **least significant byte**.

```
Address:   0x1000  0x1001  0x1002  0x1003
Memory:     78      56      34      12
```

---

## Big-endian

Lowest address contains **most significant byte**.

```
Address:   0x1000  0x1001  0x1002  0x1003
Memory:     12      34      56      78
```

---

### Important:

Registers don’t have endianness.

**Memory does.**

The CPU register holding `0x12345678` is just bits.
Only when stored byte-by-byte in memory does endianness matter.

---

# 2. Why Endianness Exists at All

Because memory is **byte-addressable**.

You don’t store a 32-bit value in one location.
You store 4 bytes in 4 addresses.

Someone has to decide:

> Which part goes first?

Historical reasons:

* **Little-endian**: easier arithmetic on early CPUs (carry propagation)
* **Big-endian**: natural for humans reading hex dumps
* Some CPUs supported both (ARM, MIPS, PowerPC)

---

# 3. Endianness vs Bit Order — DO NOT CONFUSE THEM

Endianness is about **bytes**.

Inside a byte, bit order is separate.

Example:

```
Byte: 0b10110010
```

That bit numbering has nothing to do with endianness.

Endianness does **not** change bit order inside each byte.

---

# 4. Your First Ruthless Experiment (C Program)

Let’s directly inspect memory.

```c
#include <stdio.h>
#include <stdint.h>

int main(void)
{
    uint32_t x = 0x12345678;

    unsigned char *p = (unsigned char *)&x;

    for (int i = 0; i < 4; i++)
    {
        printf("byte %d: 0x%02X\n", i, p[i]);
    }

    return 0;
}
```

### Output possibilities:

Little-endian:

```
byte 0: 0x78
byte 1: 0x56
byte 2: 0x34
byte 3: 0x12
```

Big-endian:

```
byte 0: 0x12
byte 1: 0x34
byte 2: 0x56
byte 3: 0x78
```

That’s endianness **exposed through pointer aliasing**.

---

# 5. How CPUs Implement It

Internally, CPUs have:

* Load/store units
* Byte-lane routing
* Cache lines

When executing:

```
MOV R0, [0x1000]
```

The CPU:

1. Fetches bytes from memory
2. Reorders them if needed
3. Assembles them into the register

Little-endian CPUs wire byte 0 to LSB.
Big-endian CPUs wire byte 0 to MSB.

Some CPUs are:

* **Bi-endian** → configurable at boot (ARM, MIPS)
* **Mixed-endian** → integers one way, floats another (rare but real)

---

# 6. Alignment Is NOT Endianness

Alignment = address multiple.

Example:

```
uint32_t x;
```

Aligned: address divisible by 4.

Unaligned: address like 0x1002.

You can have:

* little-endian + strict alignment
* big-endian + unaligned allowed

Totally separate concepts.

---

# 7. Struct Layout + Endianness

Struct fields are laid out in memory sequentially (with padding).

Example:

```c
struct S {
    uint16_t a;
    uint32_t b;
};
```

On little-endian:

```
a low byte
a high byte
padding?
b low
...
```

Endianness affects **inside fields**, not field order.

---

# 8. Network Byte Order (EXTREMELY IMPORTANT)

Internet protocols defined:

> **Network byte order = big-endian**

Why? Because old network gear and IBM systems were big-endian.

So:

* Sending integer → convert host → network
* Receiving → convert network → host

POSIX gives:

```
htons()  host to network short (16)
htonl()  host to network long  (32)

ntohs()
ntohl()
```

Example:

```c
uint32_t x = 0x12345678;
uint32_t net = htonl(x);
```

If host is little-endian → bytes reversed.
If big-endian → unchanged.

---

# 9. Files and Serialization

If you write raw structs to disk:

```c
fwrite(&x, sizeof(x), 1, f);
```

You just wrote **machine-endian** bytes.

Move that file to another architecture?

💥 Possibly broken.

Correct approach:

* Define file format endian explicitly
* Convert before writing
* Convert after reading

---

# 10. Manual Byte Swapping

Sometimes you do it yourself.

```c
uint32_t swap32(uint32_t x)
{
    return ((x & 0x000000FF) << 24) |
           ((x & 0x0000FF00) << 8)  |
           ((x & 0x00FF0000) >> 8)  |
           ((x & 0xFF000000) >> 24);
}
```

Compilers often replace this with a single CPU instruction (`bswap`).

---

# 11. Detecting Endianness at Runtime

Classic trick:

```c
int is_little_endian(void)
{
    uint32_t x = 1;
    return *(unsigned char *)&x == 1;
}
```

---

# 12. Compile-Time Detection

Compiler macros:

```
__BYTE_ORDER__
__ORDER_LITTLE_ENDIAN__
__ORDER_BIG_ENDIAN__
```

GCC/Clang:

```c
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
    #define HOST_LITTLE 1
#endif
```

---

# 13. Floating-Point Endianness

IEEE-754 defines bit layout.

But:

* Some systems store double words swapped.
* Most modern PCs: little-endian floats.

Never assume—serialize floats carefully.

---

# 14. Mixed / Bi-Endian Systems

ARM:

* Can boot LE or BE.
* Linux mostly uses little-endian.

PowerPC:

* Historically big-endian.
* Modern often little.

---

# 15. Common Bugs

### ❌ Casting network buffer to struct:

```c
struct Packet *p = (struct Packet *)buf;
```

Broken if endian differs.

---

### ❌ Writing binary files with fwrite(struct)

---

### ❌ Assuming x86 everywhere

---

# 16. When Endianness Does NOT Matter

* Byte arrays
* Text
* Single-byte types
* When using conversion APIs correctly

---

# 17. Mental Model You Must Burn Into Your Brain

Memory is a **row of bytes**.

Multibyte objects are **constructed** from them.

Endianness defines:

> which byte goes first.

Everything else flows from that.

---

# 18. Your Ruthless Homework

Do these without looking:

1️⃣ Write a program that prints memory of a 64-bit integer byte-by-byte.
2️⃣ Write swap16 / swap32 / swap64.
3️⃣ Write your own htons/htonl.
4️⃣ Serialize a struct into big-endian manually.



