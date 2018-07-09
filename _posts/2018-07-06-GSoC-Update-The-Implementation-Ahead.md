---
layout:     post
title:      "GSoC Update: The Implementation Ahead" 
date:       2018-07-09 16:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [GSoC, Programming]
comments:   true

---

I am back with another update. In this post, I will discuss the implementation plan that we came up with to add dynamic scheduling to GPU scheduler
and also some of the initial problems that I faced during the implementation. I will first start by explaining some of the functions before I
start talking about my work. As usual, I will assume that you have read the [posts]({{ site.baseurl }}/categories/#GSoC) that I have written
before this, if not please give them a read before going over this.

<!--more-->

I have borrowed some of this text from the [scheduler documentation](). The previous link is empty :P, I just realized that my documentation patch
would land in kernel version 4.19, will patch this link once that is out. But you can read the documentation from the source [file](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/scheduler/gpu_scheduler.c?h=drm-next-4.19)
directly.


 * `drm_sched_entity_init`: Init a context entity used by the scheduler when submitting to HW ring. This function requires a scheduler argument
     hence an entity is bound to a scheduler when it is initialized. It also requires the run queue (will be referred as rq) on which it will
     be enqueued. A scheduler has multiple queues of entities (each queue has different priority) called the run queues.
 * `drm_sched_job_init`: Init a scheduler job, we need to specify the entity and the scheduler at the time of job. However, the scheduler
     field is not required until the job gets scheduled. 
 * `drm_sched_entity_push_job`: Submit a job to the entity's job queue. This adds the entity to the scheduler's run queue if this is 
     the first job of the entity. 
 <!--* `drm_sched_entity_pop_job`: Pop a job from the entity's job queue and scheduler it on the hardware.-->
 <!--* `drm_sched_job_begin`: Sets up various callbacks before we can submit the job to hardware. -->
 <!--* `run_job`: The callback provided by the driver to submit the job to the hardware. -->
 <!--* `drm_sched_process_job`: Called right after job has finished execution. Signals the finished fence.-->
 <!--* `drm_sched_job_finish`: Called after finished fence signaled.-->
 <!--* `drm_sched_hw_job_reset`: Stop the scheduler if it contains the bad job-->
 <!--* `drm_sched_job_recovery`: Recover jobs after a reset-->

We came up with the following **implementation plan**:

1. Add a pointer from the rq to the scheduler to which this rq belongs.
2. Remove the scheduler and rq parameters from `drm_sched_entity_init`.
3. Provide the rq as parameter to `drm_sched_entity_push_job`.
4. Modify `drm_sched_entity_push_job` to only change the rq when it is the first job and the last fence is either NULL or signaled (we have
   already covered the reason for this [here]({{ site.baseurl }}{% post_url 2018-06-13-GSoC-Update-A-Curious-Case-of-Dependency-Handling %})).
5. Change `drm_sched_entity_push_job` to not only take one rq, but an array of queues where it can pick one from depending on the load.

There were two **important considerations** regarding this plan:

 * We will modify `drm_sched_entity_push_job` to reschedule entity based on the load when the job is pushed. This means **our dynamic
   rescheduling will only come into picture when a job is pushed**. Now for a case when both the sdma rings have equal total jobs/entities
   will be considered as having same load. Let's say no more jobs are pushed to any of the rings. And the jobs on sdma0 are faster then
   sdma1. All the jobs on sdma0 will finish quickly and it is waiting idly while jobs are stalled on sdma1.

   This is obviously a hypothetical scenario but it can be handled if we do a similar kind of dependency handling in
   `drm_sched_entity_clear_dep` as well. This is when an entity is ready for being scheduled and the entity can potentially be rescheduled if
   required

 * We **will not reschedule jobs that are using dependency optimization**. This leads to a tradeoff as discussed in the previous [post]({{ site.baseurl }}{% post_url 2018-06-13-GSoC-Update-A-Curious-Case-of-Dependency-Handling %}).
   For example, let's consider jobs J1 and J2, J1 depends on J2 and both belong to a different entity on the same scheduler. Now J2 is waiting for
   its turn to be scheduled, now this is the case of dependency optimization and hence we should not reschedule J2 as doing so will break the
   optimization. But if the other ring is free breaking the optimization and rescheduling J2 might lead to better performance.
 
This issues can be resolved but would add extra complexity to the design. Christian gave me the freedom to decide on a design that I would
like to pursue. In his own words:
> It's your decision what to do, you can aim for a really cool implementation which probably needs more time or you can stick to the simpler one which has less risk to fail a deadline.
 
I decided that going with a simpler plan is better as I can add complexity at a later stage after the basic thing starts working.
With this plan in place, I started working on the coding part. However, I hit the wall soon enough when I tried to shift the rq parameter from
`drm_sched_entity_init` to `drm_sched_entity_push_job`. It required significant changes in the amdgpu driver and would require me to go
through the amdgpu code before I can make those changes. I soon realized that learning about amdgpu wouldn't be feasible during the timeline
of GSoC and I discussed the same with Christian. He quickly came up with a new plan which would require much less changes to the
amdgpu code. 

He proposed that we should provide a list of rqs in `drm_sched_entity_init` itself and carry it as part of the entity. In
`drm_sched_entity_push_job` we will have a list of rqs to schedule the entity depending on the load on these run queues. I have started
working on the implementation part of this new plan. Stay tuned to know how it goes. 


