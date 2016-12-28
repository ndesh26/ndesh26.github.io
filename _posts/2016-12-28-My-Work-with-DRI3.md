---
layout:     post
title:      My work with DRI3
date:       2016-12-28 17:50:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [mesa, linux, graphics]
comments:   true

---
The semester got a whole lot busy after my last post. Hence I was not able to write. Now Since the semester is over you can expect some posts.
This post describes my work implementing DRI3 helper code for [PRIME](https://wiki.archlinux.org/index.php/PRIME) hardware.
<!--more-->

I was all set to start implementing DRI3 features for VDPAU state tracker but then I noticed that my hardware i.e. PRIME was not supported in DRI3 helper.
So I started working on it. Overall it turned out to be a small patch but for someone like me who had a little knowledge about how [PRIME](https://wiki.archlinux.org/index.php/PRIME)
works it was not a trivial task. I will try and explain my patch and what I learned along the way.

### PRIME

In 2010 NVIDIA camp up with a technology for Microsoft windows which they called Optimus which allowed seamless switching between 2 GPU's.
Dave Airlie tried to mimic the functionality for linux and gave a proof of concept in 2010 and named it PRIME for obvious reasons.
It would allow one GPU to offload its work to the other and the two can share the workload. I have a hybrid laptop which means I have 2 GPU's an
integrated intel one and a dedicated AMD one. The integrated one saves power is good for day to day activities, whereas the dGPU is needed for
more performance intensive activities like gaming, video editing etc. PRIME is a solution for GPU offloading that uses DMA-BUF sharing to share the
resulting framebuffers between the drivers of the discrete and the integrated GPU.

### DRI3

DRI deserves a post of its own, DRI is the infrastructure to allow X clients to directly access 3D hardware without going through X server to improve
performance. DRI allows a userspace application to directly render to a GPU and hence the name Direct Rendering Infrastructure. There are 3 versions
of DRI as of now i.e. DRI1, DRI2, DRI3. If a client wants to use the 3D hardware acceleration then it has to use a 3D API such OpenGL which is provided by mesa.
Although we can do the rendering directly on the hardware but to display the content on the screen we need to go through the X server.
Due to the limited memory of earlier video cards DRI1 had a single instance of front buffer and back buffer and when the back buffer is ready for display,
it is swapped with the front buffer and the front buffer is then send to the display. In DRI1 all the clients were supposed to share the same back buffer and only render
in the windows allocated to them. With the increase in memory of video cards came DRI2 which allocated a private buffer for each client.
The allocation of private offscreen buffers is done by Xserver itself. Then came the DRI3 which allowed clients to allocate their own buffers which reduce the
artifacts that were caused due to lack of synchronisation between buffer sizes of client and server and also leads to a better performance as the clients save the
extra round trip waiting for X server to send the render buffers.

### My Work

Mesa already has a DRI3 helper but it does not support PRIME. To handle the case of PRIME I needed to have a separate linear buffer(and not tiled buffer as the other GPU won't be able to read it).
It was necessary to have a separate buffer as the dGPU could have rewritten the buffer while it was being copied by the iGPU.
So, In case of PRIME each buffer should have an extra linear buffer with it and when the buffer is ready to be sent to the X server we first need
to copy the buffer to the corresponding linear buffer and then send the linear buffer to the X server, so that the buffer could be used by the iGPU.
Without the specific knowledge it seemed like a herculean task but as I look back it wasn't that difficult task.

Here's a link to the [code](https://cgit.freedesktop.org/mesa/mesa/commit/?id=853e80f5a09f85477167aac2789a91a2755e23f0). Leo Liu, Michel Dänzer, and Christian König
helped me a lot in completing this patch.

The VDPAU state tracker still doesn't take the full
advantage of DRI3 as there is an extra copying involved, which will be prevented if we could directly send the buffer to X server. I am working on it
now and hopefully it will be a part of Mesa soon.
