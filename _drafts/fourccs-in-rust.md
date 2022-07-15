---
layout: post
title: FourCCs in rust
---

<!-- more -->

```rust 
macro_rules! make_fourcc {
    ($a:literal, $b:literal, $c:literal, $d:literal) => {
        (($d as u32) << 24) | (($c as u32) << 16) | (($b as u32) << 8) | ($a as u32)
    };
}
```
