---
layout:     post
title:      "GSoC Update: A Curious Case of Dependency Handling" 
date:       2018-06-14 16:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [GSoC, Programming]
comments:   true

---

Dependency handling is the core part of my GSoC project. In this post, I will cover the existing dependency handling that is implemented
in DRM scheduler and the one that we are planning to implement for the project.

<!--more-->

I have already discussed the organization of the DRM scheduler in [this post]({{ site.baseurl }}{% post_url 2018-06-06-My-GSoC-Improving-the-GPU-scheduler %}),
which is a prerequisite for understanding this post. The below figure summarizes the organization of the DRM scheduler.


<p style="text-align:center">
<img src="{{ site.baseurl }}/assets/images/DRMSchedulerDependency.png" alt="drawing" width="600px"/>
</p>

Each job in an entity can depend on multiple jobs. The dependencies can be classified into three categories that are shown in the above figure.
The arrows in the figure represent the dependency among jobs.

* **A**: both the jobs belong to the same entity. 
* **B**: the jobs belong to different entities on the same hardware component.
* **C**: the jobs belong to different entities on different hardware components.

A job can only be executed after its dependent jobs have been executed. In addition to this the jobs in an entity should be executed in the
order they were pushed to the entity. 
### The existing dependency handling

The DRM scheduler uses DMA fences to handle the dependencies. I won't cover DMA fences, but in a nutshell, a process/thread can wait on a
fence until the fence is signaled by another process/thread. Each job has two fences, a scheduled fence, and a finished fence. As the name
suggests the scheduled fence is signaled when the job is pushed to the hardware queue and the finished fence is signaled when the job has
finished execution. The jobs with a dependency on other jobs wait on the finished fences of these jobs.

The hardware queue is a FIFO queue i.e., the jobs will be executed on the hardware component in the order they were pushed to the queue.
This allows room for optimizing the dependency handling. There are two specific cases that we can optimize. For this cases let's assume that a
job **J1** depends on **J2**, meaning that **J1** can only be scheduled once J2 is executed.

* **J1** and **J2** belong to the same entity (category **A** in the figure). We can just ignore this fence as we schedule the jobs in the order in which they are
    pushed into the entity.
* **J1** and **J2** are jobs of different entities on the same scheduler (category **B** in the figure). In this case, once **J2** is pushed to the hardware queue
    we can readily schedule **J1** as it will get pushed to the hardware queue after **J2** and will eventually be executed after **J2**. So instead of
    waiting on the finished fence of **J2** we can instead wait for the scheduled fence of **J2** for this case.

### Dependency handling for our case

In the existing scheduler the entities are assigned to the a hardware component by the driver based on a static scheduling policy. In our proposal, 
the entities are dynamically scheduled on these hardware components depending on the load. This also involves moving an already scheduled
entity from one scheduler to another. However, this load based scheduling may not always be beneficial.

We have the possibility of scheduling 2 jobs from the same entity on different schedulers however they cannot be scheduled parallelly as that
would violate their dependency hence we need to wait for the first job to be completed first. The first optimization is no longer valid in our case.

Similarly, the dependency of category **B** will be optimized in the existing case and we do not need to wait for the job to be completed.
But with our proposal the dependant entity could be moved to another scheduler then it will become a dependency of category **C** and
hence the second optimization is also not valid.

With our proposal both the dependency optimization are invalidated. But if we put some restrictions on our load balancing then we can have
dynamic scheduling as well as these optimizations. We put the following restrictions on moving a entity from scheduler:-

* If one of the jobs from the entity is in the hardware queue then we do not move this entity to any other scheduler. With this restriction, the
    first dependency optimization will always be valid.
* We don't move an entity which falls in the category **B** dependency. This is not a restriction with very direct benefit as it is more of a tradeoff,
    but we use this to keep things simple.

With the above restrictions in place we can utilize the optimizations as is. Once the project is done we plan on exploring these restrictions
in more depth to study the tradeoffs that are involved.

That's it folks!!
 
