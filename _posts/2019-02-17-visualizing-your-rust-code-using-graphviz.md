---
layout: single
title: Visualizing your rust code using graphviz
categories: [blog, rust]
tags: [rust, internal, cargo, flowgraph]
permalink: /posts/visualizing-rust.html
---
In a previous post I mentioned that the rust compiler allows you to output interesting intermediate languages/formats in a number of different ways.  
`hir`, `mir` and even flowgraphs! 
In this post I will be giving a brief overview of the flowgraph format and also instructions on how to generate images from your code.
<!--more-->

In my [previous post][exploring_rust] I gave an introduction and overview of the process your source code goes through during compilation. 
The post also included some ways of inspecting the intermediate representations of the stages.

I found an interesting debug flag to print out a `dot` compatible representation of your code, which essentially is a flowgraph of your code and wanted to investigate visualizing this.
`dot` graphs can be visualized using something like [`graphviz`][graphviz-url].

### Visualizing your code as a flowgraph
#### Compiling to flowgraph IR
To compile your source file to a flowgraph output you can adapt the following command: 
```bash
rustc +nightly -Zunpretty="flowgraph=main" src/main.rs
```
This command will generate a flowgraph out starting at the symbol `main` and then prints the output the `stdout`, to get a graph file out of it you can pipe it to one easily. 
Let's assume I piped it to `main.dot`. 

To render this out you need to have [`graphviz`][graphviz-url] installed and on your `PATH`. You can then use `dot` to render this to an image:
```bash
dot -Tpng main.dot -o"main.png"
```
This will produce the final image. 

[`cargo-inspect`][cargo-inspect] now has functionality to render to an image when specifying `flowgraph` as the unpretty mode. This got added in [PR-27](https://github.com/mre/cargo-inspect/pull/27)!
```bash
cargo inspect --unpretty=flowgraph=[symbol]  
```

***Important: If you are piping through `powershell` you might have to check if the output file is UTF8, I've found that the `dot` does not work with UTF16 encoded text files.***


### Conclusions
Generating visual graphs of the flow in a program can help to build a mental model of the program, except I haven't really found a use case for it using the output produced by rust. I suppose the main reason this exists is to debug the compiler and the code it produces. Also generating flowgraphs of big nested code does not prove to be very useful, either `dot` can't render it big enough. Or the image is just way too long and hard to inspect.

<!--  Links -->
[exploring_rust]: {% post_url 2019-02-09-exploring-rust %}
[graphviz-url]: https://www.graphviz.org/
[cargo-inspect]: https://github.com/mre/cargo-inspect