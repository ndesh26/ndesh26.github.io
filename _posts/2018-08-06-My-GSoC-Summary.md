---
layout:     post
title:      "GSoC Project Summary"
date:       2018-08-07 15:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [GSoC, Programming]
comments:   true

---
This post will act as the final Work Product for my GSoC project. [Christian König](https://www.linkedin.com/in/christian-k%C3%B6nig-35b7bbaa/)
mentored me for the project. I will start the post with the [initial project abstract](#initial-abstract) and how that converted into
the [actual project](#the-actual-project) that I worked on. After I will discuss the [result](#the-result) of my project followed by a [summary of the work](#work-summary) that I did
during the summer. I will end the post with a short discussion on [future work](#future-work). The patches that I wrote as part of the GSoC have
been merged upstream and are also available [here](https://drive.google.com/open?id=1UoHu-cE2cfXtT_CCoo65ZVIW0apMXfXi)

<!--more-->

## Initial Abstract

All the GPU drivers have a scheduler component that schedules the job received from the applications on the GPU hardware. Recently the
amdgpu’s (AMD’s graphics driver) scheduler was shifted to a common space (now called DRM GPU scheduler) so that the other drivers can
reuse the code. The GPU scheduler is now used by amdgpu and etnaviv (graphics driver for Vivante GPUs). It provides entities which allow
userspace to push jobs into queues which are then executed by a hardware run queue. Now amdgpu has multiple identical hardware queues and
we currently map round robin to the software queues provided by the GPU scheduler when those are created. To better balance the load we
could extend the scheduler to feed multiple hardware queues from just one software queue provided by the GPU scheduler.

## The Actual Project

My project revolves around the gpu-sched module that is part of the DRM subsystem in the linux kernel. A GPU can have multiple engines that
are meant for same/different tasks. For each of this engines a seprate instance of gpu-sched needs to be created. This drivers that are
using this scheduler is responsible for distributing the work among the engines. And once a job is assigned to a scheduler than that scheduler
is responsible for executing that job. We call this static scheduling. My project involved incorporating this allocation of job to a sch0duler
in the gpu-sched module. And shifting the job between different engines based on the current load on the system. In simple terms, my project
was to implement dynamic load balancing in the gpu-sched module.

## The Result 

I am happy to state that I was able to implement most of the things that I had promised in my initial abstract. And most importantly I was 
able to get my code committed upstream. One of the things that I had promised but was unable to deliver was the driver side code. Both me
and Christian decided that I won't be able to understand the driver side and implement the necessary changes in the timeline of the project.
Christian was gracious enough to take care of the driver side of code for me.

## Work Summary

In this section, I will discuss the code that I contributed during my project.

### First Coding Period

During this majority of my time was spent reading the scheduler code and understanding it. In this period I fixed minor things in the
code and added documentation to the module and converted the existing documentation to the kernel-doc format. The patches contributed during
this duration are:


* [drm/scheduler: fix function name prefix in comments](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=652470ac55543fbbdcbce25492a7e370d23a38a0)
* [drm/scheduler: fix a corner case in dependency optimization](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=6201e033d77fae5ed6798d3d122643c2fe3c24dd)
* [drm/scheduler: add documentation](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=2d33948e4e00b501b91367fed21243a948426591)
* [drm/doc: add a chapter for gpu scheduler](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=677e8622a9ae8cd9d351f98ecf120fb1c83b59d1)

Some of the code that I wrote during this period is part of kernel v4.18 release. The documentation patches that I have added are queued for
merging as past v4.19 release window.

### Second Coding Period

Some of my initial time was spent on working with the `libdrm` so that I can add a testcases to it that I can use to test my patches. 
At this point I was well aware with the scheduler code. And I started discussing the potential ways to implement the dynamic scheduling. We
decided on an implementation and I started the coding part. A significant amount of my time during this period was spent on
debugging. I was unable to get it working by the end of this coding period but I started sending some initial patches to the list to get them committed.
The patches that I contributed during this period are:

* [drm/scheduler: add a pointer to scheduler in the rq](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=37a5521ce21bd17cfbb5f1434ef080dfee33830c)
* [drm/scheduler: modify args of drm_sched_entity_init](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=4add75ef7ea78435a76051d96cb3d106d1cff320)

### Third Coding Period

The debugging continued from the previous period and I kept sending more patches upstream. Finally, I was able to complete the debugging and
get a working set of patches. The rest of the time was spent on polishing the patches and getting them upstream. The patches that I
contributed during this period are:

* [drm/scheduler: modify API to avoid redundancy](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=3dd4a58184e894df052ddea1f18c81d2168d1a6f)
* [drm/scheduler: remove sched field from the entity](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=f56628751ce0d41e05250f91fc368b2573dd38f3)
* [drm/scheduler: add a list of run queues to the entity](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=711c989d235ffd044fb04b971049e204b33fde3f)
* [drm/scheduler: add counter for total jobs in scheduler](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=bf2c5567a8d3acddaac17de9bc2f1db6337aecfe)
* [drm/scheduler: add new function to get least loaded sched v2](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=c41e5944c1f3c1d76fd17fd361a100149e84e8d8)
* [drm/scheduler: move idle entities to scheduler with less load v2](https://cgit.freedesktop.org/~agd5f/linux/commit/?h=amd-staging-drm-next&id=53d5f1e4a6d91457678a24b03d0a66edafb800ea)

My work is likely to be part of the v4.20 release of the linux kernel.

## Future Work

Though I was able to implement all the things that I had promised, the work on scheduler is far from over. The dynamic scheduling that we
implemented only targets specific cases and can be improved upon. Some of existing TODOs:

* Have a better criteria for calculating load. We only use the number of loads queued to calculate the load but we can use other things like
    number of entities as well.
* We only shift empty entities. We can start shifting non empty entities but that will require some effort as we need to handle the dependencies
    in that case properly. 
* We only shift an entity when we push a job to it. This is not the ideal scenario as it could lead to an imbalance of load, we instead should do balancing when
    a job completes. But doing that is going to be a difficult job, even I have not formulated how this could be implemented yet.

If you are looking for TODOs in scheduler other than load balancing then you should definitely contact [Christian
König](maito:christian.koenig@amd.com), he has a lot TODOs up his sleeve at all times. 

## Thanks

I would like to thank Christian König for all his support, for being extremely responsive and for being patient with me throughout the project.
Without his constant support, this project would've been a lot harder and a lot less fun to do.

I would also like to thank the dri-devel community for being helpful throughout the project.
