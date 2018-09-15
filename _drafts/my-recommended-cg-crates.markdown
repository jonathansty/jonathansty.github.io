---
layout: post
title: "My recommended crates for computer graphics"
summary: A shortlist of useful crates that I personally like and enjoy using in rust!
tags: rust graphics compute
---
It's been a while since my last post! Lately I've been quite demotivated to actually produce something worth while for doing a writeup about. Instead I offer you some thoughts on my recent findings using rust and some crates 
I found particularly useful for computer graphics.

In this post I will be discussing some crates for graphics and I will very briefly give an overview of the maths libraries that rust currently offers.

# What to use for graphics?
By far the most common rust crates for graphics include:
* [gfx](https://crates.io/crates/gfx)
* [vulkano](https://crates.io/crates/vulkano)
* [gl](https://crates.io/crates/gl)
* [ash](https://crates.io/crates/ash)

There is also wrappers for things like `allegro`, `piston2d-graphics`. I did not find the time to research any of these so I don't think I could give an accurate opinion about these.
Another disclaimer would be that out of the 4 listed above I've actually only used `vulkano` and `gl`. 

Choosing the perfect crate really depends on what you want to get out of a crate. 
Do you want to learn or write graphics code using very close to the driver API's, or would you rather get things done without having to worry about what backend, api and drivers it is using?

For the latter choice I really recommend using `gfx`. 
The description on [crates.io](https://crates.io) for this crate tells us that it's intended for high-performance use cases in rust. 

The good thing about gfx is that it implements most of the important graphics APIs. 
There is already even projects using and supporting it, I won't list any of these because I will definitely forget some. 
For a comprehensive list please refer to the [crates.io page](https://crates.io/crates/gfx).


*However*, if you are like me and want to either learn or have full control over graphics you should probably use a crate that wraps that specific API of your choice. 
I'm not sure what the state of crates for DirectX is like as I've mainly focused on trying to use `gl` and `vulkano`. 

In previous blog posts I discussed how I create the Game of life on the GPU using the `gl` crate. For some reason I thought it was a good idea to write a small wrapper around `gl` that would allow me to use it as 'safe' as possible. 
Although... at the time I didn't realise I actually am not a good rust programmer (yet) and I suck at designing APIs and like to over complicate stuff.

So in the end I decided to ditch `gl` and ended up using `vulkano` for the following reasons:
* Vulkano offers safe abstractions to interface with Vulkan
* No FFI and rust to C pointer conversions
* Vulkan > OpenGL (imo)
* No more `unsafe{}` blocks:
    * For some reason I had the idea that `unsafe` blocks would completely remove borrow checking and other safety features which is not the case. [Steve Klabnik](https://words.steveklabnik.com) does a great job at explaining this in [this post](https://words.steveklabnik.com/you-can-t-turn-off-the-borrow-checker-in-rust) 

# What to use for maths?
For maths I've only come across 2 major crates:
* [cgmath](https://crates.io/crates/cgmath):
    * A library specifically targeted at computer graphics.
* [nalgebra](https://www.nalgebra.org/):
    * A more heavy weight library. It seems to be more maintained than cgmath.
    
Both of these libraries are perfect for graphics. 
The only gotcha for `nalgebra` would be that you need to make sure that the matrices and vectors that you bind in uniform buffers have the correct data layout. 
For nalgebra refer to [a page for computer graphics](https://www.nalgebra.org/cg_recipes/). 
`cgmath` seems to automatically do what is expected.


## TL;DR

I'm still learning rust, using `gl` as a crate that just exposes functions to raw OpenGL functions wasn't the greatest thing to start with in my opinion. 
Having switched to `vulkano` I can say that I'm much happier and actually enjoying writing graphics code that uses this.

As a teaser enjoy my very nicely rendered cube with vulkano:
![Cube rendered with vulkano]({{ "/assets/vulkano-cube.png"}} "Cube rendered with vulkano")

I'm planning on doing a short usage overview of vulkano which will explain how to do things like buffer creation, images, swapchain and so on. Keep an eye on my twitter or subscribe to this RSS feed. 

Thanks for reading all!


