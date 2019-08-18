---
layout:     post
title:      "GSoC Update: Understanding the GPU scheduler"
date:       2018-06-09 16:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [gsoc, programming]
comments:   true

---

It's been 3 weeks since I started working on my GSoC project. I am working on the DRM GPU scheduler to improve the load balancing and have
more dynamic scheduling of jobs, for more details read this [post]({{ site.baseurl }}/gsoc/2018/06/09/My-GSoC-Improving-the-GPU-scheduler/).
This is my first update for the project. I wanted to be more frequent with the updates but I was unable to due to my trip. I had planned on
doing bi-weekly updates for the project but this update got delayed by 1.5 weeks. 

<!--more-->

Well it is here now. I need to be more punctual from the next time. I guess that could be my first lesson, one of the things that I wanted to
learn from GSoC is how to work in professional environment, learn the do's and don'ts, hopefully by the end of the project. 
Without further adieu let's start with the update. First I had to get acquainted with the existing scheduler code. I
decided to follow a little different approach this time, I went through the code line-by-line and tried to understand what each meant and
why it's necessary. Now I do understand what most of the code does, but I am still not so sure of why it was written that way e.g.,
scheduler uses `work_struct` for a particular feature, I do understand the code for `work_struct` but still not so very sure why it was
chosen.

Reading the entire code turned out to a painstaking more so because whenever I encountered some kernel structure, I had to go back all the
way to learn about the kernel API before I could go back to my code. The situation kept improving slowly as I kept reading more code, until I became
familiar with the most of the kernel API that was being used. One of the major problems that I faced with code reading was dealing with
synchronization, this is the first time I was working with a code in which we need to take care of multicore synchronization. The synchronization
code still gives me nightmares. Hopefully, it will get better with time. 

Initially, I had planned on writing a neat and clean blog which explains the working of scheduler with some pointer to the code. I am glad that I
didn't write such a code and instead worked on the scheduler documentation instead. I realized that the chances of someone running into my
blog who wants to learn about the scheduler is really and decided to follow the path of documentation. Shouts to Alex and Christian who helped
with the patch.

I have started discussing implementation details with Christian regarding my project, I plan on writing a blog which covers this discussion.
Lookout for the next post.

