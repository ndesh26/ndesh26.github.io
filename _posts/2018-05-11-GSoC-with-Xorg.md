---
layout:     post
title:      Google Summer of Code with Xorg
date:       2018-05-05 15:31:19
excerpt_separator: <!--more-->
categories: Personal GSoC 
tags: [updates]
comments:   true

---
I overjoyed to tell you that I will spend my summers working with the Xorg community as part of Google Summer of Code. I will be working
on DRM subsystem of the Linux kernel. I will be mentored by Christian König for this project.
<!--more-->
I will be working on the GPU scheduler which is part of DRM subsystem. The Abstract for my proposal, which is also available [here](https://summerofcode.withgoogle.com/projects/#5155537382014976)

> All the GPU drivers have a scheduler component that schedules the job received from the applications on the GPU hardware. Recently the amdgpu’s (AMD’s graphics driver) scheduler was shifted to a common space (now called DRM GPU scheduler) so that the other drivers can reuse the code. The GPU scheduler is now used by amdgpu and etnaviv (graphics driver for Vivante GPUs). It provides entities which allow userspace to push jobs into queues which are then executed by a hardware run queue. Now amdgpu has multiple identical hardware queues and we currently map round robin to the software queues provided by the GPU scheduler when those are created. To better balance the load we could extend the scheduler to feed multiple hardware queues from just one software queue provided by the GPU scheduler.

I will start my work by going through the GPU scheduler code and trying to understand the changes that are required to schedule the same
entity on different schedulers. I will be updating my progress on this blog. Stay tuned :)

Thanks for reading!!

