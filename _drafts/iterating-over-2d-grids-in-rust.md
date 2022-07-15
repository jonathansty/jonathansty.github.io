---
layout: post
title: Iterating over 2D grids in rust
---
We all know the double loop to iterate over a 2D grid for images or 2D array processing. What if I told you there is a better way? 
There is! And it's in rust!
<!--more-->

Clickbait lols

```rust
#[derive(Copy, Clone, Debug)]
pub struct Rect {
    pub x0: u32,
    pub y0: u32,
    pub x1: u32,
    pub y1: u32,
}

impl Rect {
    pub fn grid(&self) -> impl Iterator<Item=(u32, u32)> {
        let x0 = self.x0;
        let x1 = self.x1;
        let y0 = self.y0;
        let y1 = self.y1;
        (y0..y1).flat_map(move |y| (x0..x1).map(move |x| (x,y)))
    }
}
```