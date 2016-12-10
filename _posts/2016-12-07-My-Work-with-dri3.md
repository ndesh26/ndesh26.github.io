---
layout:     post
title:      My work with dri3
date:       2016-12-07 15:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [mesa, linux, graphics]
comments:   true
published: false

---
The semester got a whole lot busy after my last post hence I was not able to write about my work. Now Since the semester is over you can expect some posts 
about the work that I have been doing. This post describes my work implementing dri3 helper code for [PRIME](https://wiki.archlinux.org/index.php/PRIME) hardware. 
<!--more-->

I was all set to start implementing dri3 features for VDPAU state tracker but then I noticed that dri3 helper code was not implemented for my hardware i.e. PRIME
itself. So I started working on it. Overall it turned out to be a small patch but for someone like me who had a little knowledge about how [PRIME](https://wiki.archlinux.org/index.php/PRIME)
works it was not a trivial task. I will try and explain my patch here and what I learnt along the way.

### PRIME

NVIDIA camp up a technology which they Optimus for windows which allowed a second GPU to render 3D apps on to the screen of the first one.
It would allow one GPU to offload its work to the other and the two can share the work load. Dave Airlie tired to mimic the functionality
for linux and gave a proof of concept in 2010 and named it PRIME for obivous reasons. I have a hybrids laptop which means I have 2 GPU one
integrated intel one and a dedicated AMD one. The integrated one saves power is good for day to day activities, whereas the dGPU is needed for
more performance related acticities like gaming, video editing etc.

### DRI3

DRI deserves a post of its own, dri is the infrastructure to allow X clients to directly access hardware without going through X server to improve 
performance. So dri allows a userspace application to directly render to a GPU and hence the name Direct Rendering Infrastructure. There are 3 versions
of dri as of now i.e. dri1, dri2, dri3. I am not going to discuss all the dri verrsion

