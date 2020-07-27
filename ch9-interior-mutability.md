---
marp: true
theme: gaia
paginate: true
style: |
  /* Add "Page" prefix and total page number */
  section::after {
    font-size: 0.6em;
    content: attr(data-marpit-pagination) ' / ' attr(data-marpit-pagination-total);
  }
  thead:first-child {
      font-size: 0.8em;
      font-weight: 100;
  }
  td:first-child {
      font-size: 0.8em;
  }

  table { margin-left: auto; margin-right: auto; }
---
<!-- _class: invert lead -->
<!-- _paginate: false -->

# Programming Rust<br /> Interior Mutability

Sanguk Park <small>`efenniht@furiosa.ai`</small>

---
<!-- _class: invert lead -->

Primitive idea:

## "Aliasable XOR Mutable"

---

### Aliasable

```rust
let mut var: i32 = 42;

let p = &var;
let q = &var;

*p; *q;
```

From [Notes on a smaller Rust](https://boats.gitlab.io/blog/post/notes-on-a-smaller-rust/):

> ... values can be **mutated only if they are not aliased**, ...

---

## Why Aliasable XOR Mutable

 - Programmers' expect

   ```c
   void memcpy(char *to, char *from, size_t len) {
       while (len > 0) {
         *to = *from;
         len -= 1; to += 1; from += 1;
       }
   }
   ```

   If `to` and `from` overlap $\to$ :bug:

---

## Why Aliasable XOR Mutable

 - Programmers' expect
 - To prevent data race

   ```c
   int a = 0;
   
   void thread_1() {              void thread_2() {
     a += 1;                        a -= 1;
   }                              }
   ```

   After concurrent execution, `a` can be 0, 1, -1, or :collision:

---

## Why Aliasable XOR Mutable

 - Programmers' expect
 - To prevent data race
 - More optimization

   e.g. Type-Based Alias Analysis

   ```
   void func(int *a, float *b) {
     // Compiler assumes (char *)a != (char *)b here
     ...
   }
   ```

---

## `&mut`: exclusively borrowed reference

If you have a `&mut` reference ...

 - The reference **must not** alias.
 - You **can modify** its value.  (Aliasable XOR *Mutable*)
 - But it isn't the only way of mutation.

Tip: `mut` in `let mut x` means you can make `&mut` reference of `x`

---

## `&`: shared reference

If you have a `&` reference ...

 - References **may** alias.
 - You **cannot modify** its value.  (_Aliasable_ XOR Mutable)

But sometimes, you need to modify shared variables.

---

## Interior Mutability

Some wrapper types provide a way to modify its inner value with `&`:

|                 | Cannot make `&mut T` | Check alias at runtime   |
| --------------- | -------------------- | ------------------------ |
| Single-threaded | `Cell<T>`            | `RefCell<T>`             |
| Multi-threaded  | `AtomicT`            | `Mutex<T>` / `RwLock<T>` |

We say that they have **interior mutability**.

---

## `Cell<T>`

Mutation is restricted: `get` and `set`.

### Pros

 - No runtime overhead
 - Never panic

### Cons

 - `get` requires `T: Copy` (workaround: `Cell<Option<T>>` & `take`)
 - Memory copy happens for each mutation (suitable for small `T`)

---

## `RefCell<T>`

Check that `&mut T` uniquely exists XOR many `&T` exist in **runtime**.

### Pros

 - Can get `&mut T` without copy

### Cons

 - Panics if trying to get multiple `&mut T`, etc.
 - Runtime overhead

---

## Multi-threaded interior mutability

 - Neither `Cell<T>` nor `RefCell<T>` are `Sync`
 - Use `AtomicT`, `Mutex<T>` or `RwLock<T>` instead

More details in Chapter 19

---

## References

 - How interior mutability are implemented:
   see [`UnsafeCell` in Rust std document](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html)


 - What is formal defintion of 'Aliasable XOR Mutable':
   see [Stacked Borrows: An Aliasing Model for Rust](https://plv.mpi-sws.org/rustbelt/stacked-borrows/paper.pdf)
