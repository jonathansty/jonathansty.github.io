---
layout: post
title: "Dream Defense"
published: true 
thumbnail: "/assets/google-vr/thumbnail.jpg" 
description: Dream defense is a gamejam game created during the Creaweek in DAE. The goal was to create a small google cardboard VR game using 1 button inputs.
date:   2017-01-01 20:30:01 +0000
---

{% assign image_path = 'assets/google-vr/Menu.png' | relative_url %}
<a href="{{ image_path }}" style="clear:left" >
<img class="alignleft size-medium wp-image-113" src="{{ image_path }}" alt="Main image" />
</a>

Dream defense is a gamejam game created during the Creaweek in DAE. The goal was to create a small google cardboard VR game using 1 button inputs. It's a tower defense game 
from a standing point of view controlled by aiming your head at a point and pressing the button to interact with the world.

The goal of the game is to survive the incoming wave of monsters that want to invade the dreams of the people. 
To do this you are equiped with a couple of powerful weapons and tower placement tools.

This was a fun little project to explore the limitations of android, Google cardboard and limited input options.

## Images

{% assign images = "Explosion, laser" | split: ", " %}
{% for image in images %}
{% assign image_path = image | prepend: 'assets/google-vr/' | append: '.png' | relative_url %}
<a href="{{ image_path }}" style="float:left" ><img class="alignleft size-medium wp-image-113" src="{{ image_path }}" alt="" width="300" height="159" /></a>
{% endfor %}

<div style="clear:left"></div>
