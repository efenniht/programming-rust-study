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
  img { mix-blend-mode: multiply; float: left; margin-right: 2em; }
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

Aliasable: you can make a reference of value.
