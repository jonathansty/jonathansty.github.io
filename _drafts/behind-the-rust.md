---
layout: post
title: Behind the Rust
---
Rust provides a lot of language constructs to enable and empower the user to write memory safe and correct code. 
But what happens behind these constructs?
<!--more-->

Last weekend I went to [fosdem 2019](fosdem.org).
This is where I had the chance to attend a talk given by [Matthias Endler][mre]. 
In his talk he explained how rust has got a lot of syntactic sugar to help the programmers in writing safe and correct code, part of his talk was explaining `cargo-inspect` to analyse this syntax and see what's happening behind the scenes.

## The compiler
![rust-compiler-overview]({{ "/assets/rust-internals-compiler-flow-new.svg" }}){:class="img-responsive"}

The rust compiler goes through several stages when processing your source code. 
The first stage is translating your source code into HIR( High-level intermediate representation ). 
`HIR` is a compiler friendly representation of the AST that is obtained after parsing, expanding macros and resolving names. For more info you can visit the [HIR section](https://rust-lang.github.io/rustc-guide/hir.html) of the rust book.

After having converted to `hir` it will convert that into `mir`. 
Which is another intermediate representation, this form is a very simplified form of rust. 
Converting to `mir` is useful for flow-sensitive safety checks (borrow checker, heck yeah), optimization and code generation. 

Finally, the code is converted to LLVM IR which in turn gets converted to machine code. 

The reasoning behind creating these intermediate language is somewhat explained in the [rustc book](https://rust-lang.github.io/rustc-guide/) and also in a concise [blog post](https://blog.rust-lang.org/2016/04/19/MIR.html)

A very neat thing of the `rustc` compiler is that it allows use to print out these intermediate compile steps (even though `mir` is not an actual string representation in the compiler).

For example, to print out the `hir` for a file called **foo.rs** we write:
```
rustc +nightly -Zunpretty=hir foo.rs
```
*Caveat: `-Z` is a nightly only flag.*

The compiler supports various types of `unpretty`:
```
        `expanded`, 
        `expanded,identified`,
        `expanded,hygiene` (with internal representations),
        `flowgraph=<nodeid>` (graphviz formatted flowgraph for node),
        `flowgraph,unlabelled=<nodeid>` (unlabelled graphviz formatted flowgraph for node),
        `everybody_loops` (all function bodies replaced with `loop {}`),
        `hir` (the HIR), `hir,identified`,
        `hir,typed` (HIR with types for each node),
        `hir-tree` (dump the raw HIR),
        `mir` (the MIR), or `mir-cfg` (graphviz formatted MIR)
```

These are pretty self-explanatory. 
One that really peaked my interested is the `flowgraph` to get a [graphviz] formatted flowgraph. 
In a next post I might outline how to render this to a graph using [graphviz].

Ofcourse exploring each and every rust file in your crate like this can be tedious.
There are various tools created to make this process easier and more fun to do. 

## Tools 
To inspect rust `hir`, `mir` and assembly I've currently come across 3 different crates/tools that can do this for you directly from your terminal with nice formatting and other features.
The tools are:
+ `cargo-inspect`
   + "de-sugar"s the rust expressions and shows the `hir` in the terminal
+ `cargo-expand`
   + Similar to `cargo-inspect` but more aimed at expanding macros in rust
+ `cargo-asm`
   + Outputs the assembly of a rust crate/function/symbol

Internally the tools call `rustc` with the correct command line parameters and then pass the output through some formatters and pretty printers.

## Examples
The following examples are all inspected using `cargo-inspect`. 
You can also use `cargo-expand` but it will give very similar results. 
I won't be discussing `cargo-asm` in this post.

### Most simple program
```rust
fn main() {}
```
Running `cargo-inspect` on the above snippet produces the following output:
```rust
#[prelude_import]
use ::std::prelude::v1::*;
#[macro_use]
extern crate std;
fn main() { }
```

Here we can see what is included into our program by default when we compile it. Things like the prelude (`Box`, `Option`, ...) and the std crate.

### Macro Expansion
```rust
fn main() {
    let v = vec![1,2,3];
    let v2 = vec![1;4];
}
```
The above code snippet demonstrates the usage of `vec!` to construct a vector. It also demonstrates the use of the `vec!` macro using a value and a size to create a vector with the element 4 times.
```rust
// prelude,std import omitted
fn main() {
    let v = <[_]>::into_vec(box [1, 2, 3]);
    let v2 = $crate::vec::from_elem(1, 4);
}
``` 


### if let expansions  
```rust
fn main() {
    if let Some(val) = Some(15) {
        //` do stuff with 15
    }
}
```
The `if let` simply gets expanded to a match and the code that needs to be executed gets put into the `Some` part of the match.

```rust
fn main() {
    match Some(15) {
        Some(val) => {
            // do stuff with 15
        }
        _ => (),
    }
}
```
### while let 

```rust
fn main() {
    while let Some(v) = 15 {}
}
```

`while let` loops seem to get expanded to a normal loop with a match that breaks.

```rust
fn main() {
    loop {
        match 15 {
            Some(v) => {}
            _ => break,
        }
    }
}
```

This is interesting because when I tested a normal `while` loop it did not get expanded into a normal `loop`. 

However this does get converted to a simple loop using `goto` when converted into `mir` according to 
[this](https://blog.rust-lang.org/2016/04/19/MIR.html) post.

### Iterator looping
```rust
fn main() {
    let v = vec![1,2,3];
    for val in v {
        // Do stuff with v 
    }
}
```
Above code produces quite a large output in `hir` but it also demonstrates how certain constructs get parsed/converted.

```rust
fn main() {
    let v = <[_]>::into_vec(box [1, 2, 3]);
    {
        let _result = match ::std::iter::IntoIterator::into_iter(v) {
            mut iter => loop {
                let mut __next;
                match ::std::iter::Iterator::next(&mut iter) {
                    ::std::option::Option::Some(val) => __next = val,
                    ::std::option::Option::None => break,
                }
                let val = __next;
                {}
            },
        };
        _result
    }
}
```

The reason this is that internally this is just a while loop over an iterator until the iterator is `None`. 
As we've seen in the previous chapters `while let` loops are expanded to a normal loop with a match.

Another thing you can notice here is that converting your source code into `hir` makes it more explicit. 
A prime example is the usage of `IntoIterator::into_iter` and `Iterator::next` instead of the associated functions ( `v.into_iter()`, `iter.next()`)

## The future
In a future post I will discuss how to visualize the graph use [graphviz]  

## Credits
- Thanks to [mre] for giving an awesome fosdem talk and creating [cargo-inspect][cargo-inspect-git]
- The [compiler internals book][rustc-book]
- This [post](https://blog.rust-lang.org/2016/04/19/MIR.html)

<!-- Links -->
[rustc-book]: https://rust-lang.github.io/rustc-guide/index.html
[mre]: https://github.com/mre
[cargo-inspect-git]: https://github.com/mre/cargo-inspect
[cargo-expand-git]: https://github.com/dtolnay/cargo-expand
[cargo-asm-git]: https://github.com/gnzlbg/cargo-asm
[graphviz]: https://www.graphviz.org/