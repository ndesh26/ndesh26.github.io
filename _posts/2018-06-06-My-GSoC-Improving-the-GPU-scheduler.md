---
layout:     post
title:      "My GSoC Project: Improving the GPU scheduler"
date:       2018-06-09 15:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [GSoC, Programming]
comments:   true

---

I'll be spending my summers with the Xorg community (more specifically dri-devel community) to Improve the GPU scheduler which is part of
the Linux DRM subsystem. This post sheds more details on my project and is a somewhat extended version of my GSoC proposal.

<!--more-->

Today's GPUs have various hardware components that perform specific functions. And there are multiple copies of some of these hardware
components. The userspace application submits commands to the GPU driver which is then responsible for converting this commands into jobs
that can be executed by these hardware components. The GPU driver is also responsible for scheduling them on these hardware components. 
Each of these hardware components has a hardware queue and the GPU driver pushes job onto this queue. 


All the GPU drivers have a scheduler component that schedules these job 
on the GPU hardware. Recently the amdgpu’s (AMD’s graphics driver)
scheduler was shifted to a shared space (now called DRM GPU scheduler) so that the other
drivers can reuse the code. The GPU scheduler is now used by amdgpu and etnaviv (graphics
driver for Vivante GPUs). My project involves working with the DRM GPU scheduler, but for explaining my project we first need to understand the
organization of the GPU scheduler. The scheduler provides a generic interface which can then be used by GPU drivers to submit jobs to hw
queues. 

The basic entity for scheduling is a job. The jobs have some restrictions regarding the order that they are to be scheduled. The scheduler
is responsible for ensuring the correctness of scheduling.

1. Each hardware component (with a hw queue) has one scheduler
2. Each scheduler has multiple software queues with different priorities (e.g., HIGH_HW, HIGH_SW, KERNEL, NORMAL). The drivers  push
   entities into these software queues we will get to what entities are in a bit.
3. Entities are a collection of jobs. They have a queue of jobs. But the jobs of an entity are to be scheduled in the order they
   were pushed to the entity. This is one of the first restrictions that we have on job scheduling.

Some jobs also depend on other jobs and hence cannot execute before their dependencies have been executed. The scheduler takes cares of this
dependency handling.

As I mentioned earlier there could be multiple copies of a hardware component and the jobs can be scheduled on any of them. As of now, the
driver is responsible for scheduling an entity on a particular hardware component and pushing the entity on the scheduler of that hw component.
This leads to static scheduling of jobs on these similar components which in the worst case could lead to a load imbalance among this
components. My project is to make this scheduling dynamic to have a better load balancing.

This is a high-level overview of what I want to accomplish. There are various implementations possible for this idea and we are trying to
explore some of them. I will be writing the GSoC status update soon.

Thanks for reading!!
