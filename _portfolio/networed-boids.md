---
layout: post
title: "Networked boids simulation"
published: true 
description: A research project to learn about the TCP/IP stack and networking for real-time applications such as games
tags: ['networking']
---
# Summary
A small real-time application demonstrating the knowledge of the networking stack in C++ using SFML2 for window creation, shape rendering and wrapping around WinSock. The architecture is a multi-server based implementation where each server communicates in a p2p based fashion with the other servers. Clients merely receive the information from the current server they are connected to and display that locally on the screen. The goal to reduce usage and allow the visualization to run on low end devices.

# Video
{% include youtubePlayer.html id="LNfDdVuFYHs"  %}

# Code
The source code can be found on my [GitHub](https://github.com/jonathansty/Networked-Boids). There is a README and scripts provided for running the project.