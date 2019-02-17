---
layout: post
title: Exploring rust
categories: [blog, rust]
tags: [rust, internal, asm, cargo, mir, hir]
---
Rust provides a lot of language constructs to enable and empower the user to write memory safe and correct code. 
But what happens behind these constructs? In this post I will outline ways of exploring rust and it's compiler.
<!--more-->

Last weekend I went to [fosdem 2019][fosdem].
This is where I had the chance to attend a talk given by [Matthias Endler][mre]. 
In his talk he explained how rust has got a lot of syntactic sugar to help the programmers in writing safe and correct code, part of his talk was explaining `cargo-inspect` to analyse this syntax and see what's happening behind the scenes. This inspired me to dig a bit deeper and try out other tools. 

## The compiler
![rust-compiler-overview]({{ "/assets/rust-internals-compiler-flow-new.svg" }}){:class="img-responsive"}

The rust compiler goes through several stages when processing your source code. This is done in order to speed up certain tasks that happen in the compiling process. 
The first stage is translating your source code into `hir` which is a high-level intermediate representation. `hir` is a compiler friendly representation of the [AST] that is obtained after parsing, expanding macros and resolving names.

After having converted to `hir` it will convert that into `mir`. Which is another intermediate representation, this form is a very simplified form of rust.
Converting to `mir` is useful for flow-sensitive safety checks (borrow checker, heck yeah), optimization and code generation. 
Finally, the code is converted to LLVM IR which in turn gets converted to machine code. 

Instead of trying to explain everything in detail and make mistakes I strongly encourage you to read the [rustc book](https://rust-lang.github.io/rustc-guide/). Also read this [blog post](https://blog.rust-lang.org/2016/04/19/MIR.html) to understand the reasoning of why `mir` was created.

### Exploring rustc
A very neat thing of the `rustc` compiler is that it allows use to print out these intermediate compile steps (even though `mir` is not an actual string representation in the compiler).
For example, to print out the `hir` for a file called **foo.rs** you can write:
```
rustc +nightly -Zunpretty=hir foo.rs
```
Neat right? Well, there's more!

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

I already hear you asking, "but Jonathan, if I want to inspect a file I will have to do this everytime". Don't worry, in the next chapter I will outline and describe tooling for inspecting `hir`, macros and asm for either crates or files.

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

### Dissecting an iterator based loop 
In the talk by [Matthias][mre] you can find some examples to show of `cargo-inspect` [here](https://fosdem.org/2019/schedule/event/rust_cargo_inspect/). For the sake of keeping this post small I will skip over the most trivial examples and jump straight to a more interesting example, which is **iterators**!

Take the following code:
```rust
fn main() {
    let v = vec![1,2,3];
    for val in v {
        // Do stuff with v 
    }
}
```

On it's own this looks like fairly straightforward code. 
We construct a simple vector of 3 numbers using `vec!` and then we just visit each number using a `for` loop. 

Let's have a look at the output that [cargo-inspect] gives us:


```rust
// Omitted std::prelude header
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
                { 
                    // do stuff with v
                }
            },
        };
        _result
    }
}
```
Wowie, that's a lot of code. The code produced here is very explicit which makes it easier for the compiler to deal with it.

```rust
    let v = <[_]>::into_vec(box [1, 2, 3]);
```
is a simple expansion of the `vec!` macro which turns a slice on the heap into a vector (I think).

Looking down a couple of lines we can see what our `for` loop got expanded to. 

First we have to turn our `Vec` into an `Iterator` which is done on line 3 with `IntoIterator::into_iter(v)`. Notice how this is more explicit than `v.into_iter()`.  A `match` is done on the result of that to capture the iterator as a mutable variable. 

Inside of the match we enter a `loop` and repeatedly match `Iterator::next(&mut iter)`, (again notice that it's not `iter.next()`), eventually the value of the current element is moved into `val` and can be used in the code block that we have defined.


## Conclusion
The rust compiler has a great deal of options that can be used to understand it's behaviour and internals. It allows the community to build tools upon this compiler together with the `cargo` build system you can have a very straightforward and clear way of playing around with it.

A thing I noticed was that `rustc` and output an intermediate [graphviz] representation. In a future post I will outline how easy it is to generate this from the command line using [graphviz].

## Credits
- Thanks to [mre] for giving an awesome fosdem talk and creating [cargo-inspect][cargo-inspect-git]
- The [compiler internals book][rustc-book]
- This [post](https://blog.rust-lang.org/2016/04/19/MIR.html)

<!-- Links -->
[rustc-book]: https://rust-lang.github.io/rustc-guide/index.html
[mre]: https://github.com/mre
[cargo-inspect]: https://github.com/mre/cargo-inspect
[cargo-inspect-git]: https://github.com/mre/cargo-inspect
[cargo-expand-git]: https://github.com/dtolnay/cargo-expand
[cargo-asm-git]: https://github.com/gnzlbg/cargo-asm
[graphviz]: https://www.graphviz.org/
[AST]: https://en.wikipedia.org/wiki/Abstract_syntax_tree 
[fosdem]: https://fosdem.org/2019/