---
layout: single
title: "Relic Defender"
published: true 
excerpt: A DirectX 11 C++ project to explore the basics and beginnings of graphics and engine programming
header:
    teaser: /assets/relic-defenders/relic-defenders-th.jpg
date: 2023-08-27
group: project
---
<!-- VIDEO -->
{% include youtubePlayer.html id='4kTTrdDM96Y' %}

Defend you relic against waves of evil monsters. Each wave gets harder and harder over time. Build and upgrade towers and see your force prevail!

The game is built upon the overlord engine (c++) which I heavily adjusted. Extended the engine with technologies like Shadow mapping, post processing, Behaviour trees, A* pathfinding, cloth etc. During the creation of this game I also had to dig around in the engine and fix problems that had arisen. This allowed me to learn more about underlying technologies of graphics programming and game engines. 

## Technical Details
- Type: Strategy
- Engine: OverlordEngine
- Platform: Windows
- Language: c++

## Features
- Post processing filters (Edge detection, greyscale, blur)
- Font rendering
- Basic UI rendering and interaction
- Jean "Cloth" Van Damme (Cloth rendering and physics)
- Shadow mapping
- Skinned animations
- Particles
- Graphics reacting to gameplay
- Physics integration using PhysX
 

## Images
{% assign images = "0, 1, 2, 3" | split: ", " %}
{% for image in images %}
{% assign image_path = image | prepend: 'assets/relic-defenders/2DAE1_Jonathan_Steyfkens_GP2Exam2016_PROJECT_' | append: '.png' | relative_url %}
<a href="{{ image_path }}" style="float:left" ><img class="alignleft size-medium wp-image-113" src="{{ image_path }}" alt="" width="300" height="159" /></a>
{% endfor %}

<div style="clear:left"></div>