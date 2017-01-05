---
layout: page
title: Projects 
---
<h3>Xorg(Mesa): Implement presentation features in VDPAU state tracker</h3>
<i>Mentor: Christian König (Senior Developer, AMD)</i>  <a class="right" href="https://cgit.freedesktop.org/mesa/mesa/log/?qt=author&q=Nayan+Deshmukh">Code</a>
<ul>
<li>Implemented luma keying as part of color space conversion code and bicubic and lanczos interpolation algorithms as fragment shaders in TGSI</li>
<li>Reworked the VPDAU mixer implementation so that it uses temporary buffer while applying filters</li>
<li>Implemented DRI3 helper code for the Gallium interface for hardware with PRIME GPU offloading</li>
</ul>

<h3>Edge-disjoint spanning trees in undirected graphs</h3>
<i>Mentor: Dr. Ovidiu Daescu (Professor, The University of Texas at Dallas)</i>
<ul>
<li>Worked on finding 2 edge-disjoint spanning trees in a special class of graphs with 2n-2 edges and a double edge</li>
<li>Proved a lemma regarding allocation of edges of 2,3 and 4 degree vertex</li>
<li>Conceptualized an algorithm to construct the two trees using the lemma, currently trying to implement and prove it</li>
</ul>

<h3>Rust bindings for libevdev</h3>
<i>Mentor: Peter Hutterer (Senior Software Engineer, RedHat)</i> <a class="right" href="https://github.com/ndesh26/evdev-rs">Code</a>
<ul>
<li>libevdev is a wrapper library for handling evdev kernel devices</li>
<li>Implemented a safe wrapper to libevdev library, also wrote documentation and tests for the bindings</li>
</ul>

<h3>Social Robot</h3>
<i>Mentor: [Prakhar Jawre](http://home.iitk.ac.in/~jprakhar/)</i> 
<a class="right" href="{{ site.baseurl }}/assets/projects/ZIZO101/SocialRobot.pdf" target="_new">Documentation &nbsp;</a>
<a class="right" href="https://www.youtube.com/watch?v=pDPp6o4OWQk" target="_new">&nbsp; Project Video &nbsp;</a>
<a class="right" href="https://github.com/ndesh26/ZIZO101" target="_new">&nbsp; Code</a>
<ul>
<li>Developed an animatronic head capable of human interaction via speech and through its Twitter handle</li>
<li>Used Radxa Rock as the main development board along with Arduino Mega for controlling servos and Python as the primary language</li>
<li>Implemented Speech to text using Google’s speech API, Artificial Intelligence through Pandorabots API, text to audio using eSpeak and connected to Twitter using TweetPony API</li> 
</ul>
![Social Robot]({{ site.baseurl }}/assets/images/socialrobot2.jpg)
![Social Robot]({{ site.baseurl }}/assets/images/socialrobot1.jpg)

<h3>NachOS</h3>
<i>Course Project under Dr. Mainak Chaudhuri(Associate Professor, IIT Kanpur)</i><a class="right" href="https://github.com/ndesh26/OS-Assignments">Code</a>
<ul>
<li>Implemented system calls pertaining to fork, exec, join, yield, sleep and exit for NachOS </li>
<li>Implemented process scheduling algorithms like UNIX scheduling, round-robin, SJF, non preemptive</li>
<li>Implemented page replacement algorithms such as random page allocation, FIFO, LRU and LRU clock</li>
</ul>

<h3>App Timer</h3>
<i>Mentor: [Prashant Jalan](http://prashantjalan.com/)</i><br>
AppTimer is a android library which provides the time spent by the user on each app. AppTimer supports until Android Kitkat. It uses GET_TASKS permission to retrieve the app which is running in the foreground.
<ul>
<li>Explored various ways to calculate the time spent by user on each ways and implemented the most efficient one</li>
<li>Developed a background service which would run at specific intervals and record the application which was running in foreground at that time and store it in SQL database</li> 
<li>The Code is available on <a  href="https://github.com/ndesh26/AppTimer" target="_new">github</a>
</li>
</ul>
<h3>Stable Marriage Problem</h3>
<i>Mentor: Dr. Rajat Mittal (Professor, IIT Kanpur)</i> <br>
The Report is available [here]({{ site.baseurl }}/assets/projects/stable-marriage/report.pdf)
<ul>
<li>Studied the Stable Marriage Problem and Analysed the proof of the algorithm to solve it</li>
<li>Studied various standard techniques used in solving the problem and its variants</li>
<li>Analysed theorems regarding existence of a stable marriage in case of incomplete lists</li> 
</ul>


