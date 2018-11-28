---
layout: post
title: "Relic Defender"
published: true 
thumbnail: "/assets/relic-defenders/thumbnail.jpg" 
description: While studying each student got a base framework to extend and implement rendering features in, this was called the overlord engine. It was pretty bare bones and required us to implement common rendering features.

date:   2018-06-06 22:30:01 +0000
priority: 3
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