---
layout: post
title: "Realtime metaballs"
published: true 
description: realtime metaballs implemented using D3D11
---

{% include youtubePlayer.html id="7w5PWmYU8to" %}

# Summary
Geometry shader created with HLSL in a framework provided by Howest (overlord). The framework is programmed in
c++ using the DirectX api. Everything is built from scratch: gui, rendering, geometry generation. The
algorithm is based on marching cubes but adjusted to marching tetrahedra for use within HLSL.

I first prototyped metacirces in 2D to understand the mathematical concepts. You can find links to both applications below.


# Technical Details
- Engine: Overlord Engine ( DAE )
- Language: C++
- Graphics API: DirectX 11
- Version Control: Git

# Research

This application was part of the module Graphics Programming 2 of DAE. 
We were tasked for creating a geometry shader using 
the DirectX 11 and the DirectX 11 effects framework. 
This really thought me a lot what to do and 
what not to do in graphics programming (mostly what not to do).
I’ve also written a paper highlighting my research.

The paper can be found [here](http://jonathansteyfkens.com/releases/Papers/2DAE1_Steyfkens_Jonathan_GP2Exam_Paper.pdf)