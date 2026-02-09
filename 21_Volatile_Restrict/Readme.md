

# ✅ Part 2 — `restrict` in C (Complete Deep Dive)

Unlike `volatile`, which **limits** compiler optimizations,
**`restrict` does the opposite** — it **allows more aggressive optimization** by promising something very specific about pointers.

---

# 🔵 What is `restrict`?

`restrict` is a keyword introduced in **C99**.

It can only be used with **pointers**.

When you write:

```c
int * restrict p;
```

you are telling the compiler:

> **“For the lifetime of this pointer, the object it points to will only be accessed through this pointer (and pointers directly derived from it).”**

In simpler words:

👉 **No other pointer aliases the same memory.**

This lets the compiler:

* reorder loads/stores
* vectorize loops
* keep values in registers
* eliminate redundant reloads

---

# 🔵 Why does aliasing matter?

Consider:

```c
void add(int *a, int *b, int *c, int n) {
    for (int i = 0; i < n; i++)
        a[i] = b[i] + c[i];
}
```

Without knowing anything else, the compiler must assume:

* `a`, `b`, and `c` **might overlap**

So it must be conservative.

---

# 🔵 With `restrict`

```c
void add(int * restrict a,
         int * restrict b,
         int * restrict c,
         int n)
{
    for (int i = 0; i < n; i++)
        a[i] = b[i] + c[i];
}
```

Now the compiler knows:

✔ `a`, `b`, `c` point to **non-overlapping memory**

This often produces **much faster** code.

---

# 🔵 The Promise You Make ⚠️

When you use `restrict`, **you promise the compiler that aliasing will not happen**.

If that promise is broken:

❌ **Undefined Behavior**

Example:

```c
int x[10];

add(x, x, x, 10);   // ❌ UB if parameters are restrict
```

Because `a`, `b`, and `c` all alias the same array.

---

# 🔵 Lifetime of `restrict`

The guarantee only applies:

🕒 **during the scope of that pointer**

Example:

```c
void f(int * restrict p) {
    int *q = p;   // q is derived from p — allowed
}
```

But:

```c
void g(int * restrict p, int *q) {
    q = p;   // ❌ violates if q used to access same object
}
```

---

# 🔵 restrict vs const

They are completely different:

| Keyword    | Means                        |
| ---------- | ---------------------------- |
| `const`    | You won’t modify the object  |
| `restrict` | No other pointer accesses it |
| `volatile` | Value may change externally  |

You can combine them:

```c
void f(const int * restrict p);
```

---

# 🔵 Practical Example — Matrix Multiply

```c
void matmul(int n,
            double * restrict A,
            double * restrict B,
            double * restrict C)
{
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            double sum = 0;
            for (int k = 0; k < n; k++)
                sum += A[i*n+k] * B[k*n+j];
            C[i*n+j] = sum;
        }
}
```

This is a classic HPC optimization.

---

# 🔵 restrict in Local Variables

```c
void copy(int n, int *src, int *dst) {
    int * restrict s = src;
    int * restrict d = dst;

    for (int i = 0; i < n; i++)
        d[i] = s[i];
}
```

You assert inside the function that they don’t overlap.

---

# 🔵 What `restrict` Does NOT Do ❌

🚫 It does not check anything at runtime
🚫 It does not prevent modification
🚫 It does not allocate memory
🚫 It does not enforce safety
🚫 It does not work on non-pointers

It’s **purely a compile-time promise**.

---

# 🔵 C++ Note

C++ does **not** have standard `restrict`.

Compilers provide extensions:

* GCC/Clang: `__restrict__`
* MSVC: `__restrict`

---

# 🔵 Summary of `restrict`

| Property               | restrict |
| ---------------------- | -------- |
| Only for pointers      | ✅        |
| Enables optimization   | ✅        |
| Prevents aliasing      | ✅        |
| Runtime checking       | ❌        |
| Safe if misused        | ❌        |
| Common in HPC/embedded | ✅        |

---

---

# 🧠 Final Big Picture

| Qualifier  | Purpose                               |
| ---------- | ------------------------------------- |
| `volatile` | Stop compiler from assuming stability |
| `restrict` | Tell compiler memory doesn’t alias    |
| `const`    | Prevent modification                  |


