---
layout:     post
title:      "The ISCA Experience"
date:       2018-07-12 16:31:19
excerpt_separator: <!--more-->
categories: Personal 
tags: [personal, architecture]
comments:   true

---

<img src="{{ site.baseurl }}/assets/images/isca-logo.png" alt="drawing" align="left" style="margin-right: 25px" width="200px"/>
This post is about my recent trip to Los Angeles, USA for presenting my work titled *"DFCM++: Augmenting DFCM with Early Update and Data
Dependency-driven Value Estimation"* at **Value Prediction Championship (CVP)** held in conjunction with ISCA 2018. 

<!--more-->

In simple words value prediction involves guessing the output of machine level instructions so that the instructions that depend on the
output of this instruction can start executing without waiting. Correct guess by the value predictor improves performance whereas wrong guess
leads to a penalty. The Value Prediction Championship involves comparing different predictors and selecting the one which provides the best
performance improvement. 

## Before conference 

[Prof Biswabandan Panda](https://www.cse.iitk.ac.in/users/biswap/) (will be referred as Biswa in the rest of the post) was the one who
informed me about CVP and encouraged me to participate. Since my course load for the last semester was not much, I decided to participate in CVP.
The next step was to form a team (I was initially skeptical of this but realized later on how important it was). Biswa suggested that the
ideal group size will be 3. In the previous semester, I did a course with Snehil who was also interested in Computer Architecture and had
some research experience as well I thought he would be the ideal candidate to be a teammate. Biswa later introduced me to a junior of mine,
Prakhar Agrawal who was also interested in participating in CVP. Hence the three of us formed a team and with the guidance of Biswa and
[Prof Mainak Chaudhuri](https://www.cse.iitk.ac.in/users/mainakc/) started working on the framework.

We had regular meetings to decide the ideas that we were going to work on and discuss how the ideas from the previous meeting fared. 
I still remember that for last week it was all nightouts. We would sleep in the morning at around 7-8am and then reconvene at 3pm in the
afternoon. I realized the importance of team during this time, other than definitely reducing the workload, whenever we would hit a wall
someone or the other would come up with a idea that would keep us going. That's one of the main benefits of having a team, you have a group of a diverse set of people who
have a different perspective on the problem. We finally submitted on 1st April and got the announcement of acceptance at CVP on 10th April.
Though our code was not working at this point and we had to debug it before the camera ready deadline but that's a story for another blog. After that it was
smooth sailing.

Me, Snehil and Biswa were the ones who traveled to the conference. Unfortunately, Prakhar and Prof Mainak were not able to
join us due to their respective constraints.

## During conference

The conference was from 2-6 June. We reached LA on 1st June in the afternoon and had nothing planned in the evening so we set out to explore
the LA. The first thing that comes to mind when one thinks of LA is the Hollywood
sign, and that became our first destination. We spent the evening around the area of Hollywood sign, got some photos clicked and visited the
Griffith Observatory.

<p>
<img src="{{ site.baseurl }}/assets/images/ISCA/day0_1.jpg" alt="drawing" style="width: 49.5%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day0_0.jpg" alt="drawing" style="width: 49.5%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day0_2.jpg" alt="drawing" style="width: 49.5%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day0_3.jpg" alt="drawing" style="width: 49.5%; display: initial"/>
</p>

I was staying at the [USC village](https://uscvillage.com/) which is the residential campus for the students studying in USC. The USC village was recently built and had a very
nice infrastructure. The USC village serves as a housing facility for conferences held during the summers. The conference started on 2nd June with the
workshops and tutorials. The first two days of the conference consists of workshops and tutorials and the main symposium will start from day 3.
The Value Prediction Championship was a workshop which was supposed to be held on day 2. I half heartedly attended some workshops on day 1 as
I was more worried about my presentation on day 2. I didn't meet a lot of people during day 1 as this was my first time attending a conference and
I didn't dare to approach new people. During the same time, I was also facing the effects of jet lag which led to sleepy evenings and 
sleepless nights. The workshops for day 1 ended at 4pm, and we decided to visit the Hollywood Walk of Fame. I
wanted to work more on the presentation but I had already had been doing that during the workshops so I thought that a break might be useful.

<p>
<img src="{{ site.baseurl }}/assets/images/ISCA/day1_1.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day1_2.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day1_3.jpg" alt="drawing" style="width: 32%; display: initial"/>
</p>

I slept early on day 1 (around 10pm) and woke up at 3am to work on the presentation. By 6am I had gone through the presentation k number of
times but I was still feeling pretty nervous. This was the most important day of the conference for me and yet the start of the day was not all that rosy.
To reach the conference venue I first had to walk for around 30 minutes to the nearest metro station and then take metro for
10 minutes from there. After I had walked for 15 minutes from my room I realized that I forgot my metro tap card as well my ISCA id card back in my room.
I didn't have a choice of walking back to my room as that would delay me by half an hour and today was the only day when I was required to
reach the conference on time. I decided that I will get a new tap card at the metro station and ask the organizers to let me in without my
id card. Luckily both of them worked out in my favor and I was able to reach before time (even got some spare time for breakfast :P).

The workshop started smoothly. My presentation was scheduled at the last so I had to sit through all the presentations nervously. Finally, my turn
for presentation came and as expected my brain mostly stopped working at this point due to nervousness but because of the all the practice
presentations that I did in the morning I was able to speak up some of the things that I had prepared. It was a pretty nerve wrecking experience. Surprisingly I got
pretty positive feedback about my presentation. Even Prof Andre Seznec(who the value prediction championship in all tracks) personally told me how
much he liked my presentation, I was elated. The results were announced at the end of the workshop, we secured the **second position** in the
unlimited track. The workshop ended before lunch

<p style="text-align:center">
<img src="{{ site.baseurl }}/assets/images/ISCA/certi.jpeg" alt="drawing" align="center" />
</p>

There was a bias busting session after the lunch sponsored by Google. The session itself was not that interesting but had some group
exercises in between which allowed me to have my first interactions with people there. I met a guy from UCSB during the bias busting
workshop who introduced me to a lot of people at various points of time during the conference. After the bias busting session, Snehil and I
roamed the streets of LA while Biswa was busy networking at the conference.

<p>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_1.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_3.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_4.jpg" alt="drawing" style="width: 32%; display: initial"/>
</p>

<p>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_2.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_5.jpg" alt="drawing" style="width: 32%; display: initial"/>
<img src="{{ site.baseurl }}/assets/images/ISCA/day2_6.jpg" alt="drawing" style="width: 32%; display: initial"/>
</p>

In the evening was the opening reception. Snehil and I were roaming around trying to talk to some of the big guns in the industry and
academia. However, we were unable to talk to any of them and ended up talking to first year PhD and MS students. After this, we attended a
dinner sponsored by Qualcomm for all the participants of CVP. Having an informal chat with the participants was nice. Biswa and I are
vegetarian, so we were struggling to get veg food for most of the time during the dinner. 

The main symposium started on day 3 with the first of the three keynotes. The paper presentations started soon after that. I attended some
of the presentations and benched out from the others to do some networking during the conference. The Turing lecture for the year was
supposed to be held at ISCA this time and I was lucky enough to attend it. It was scheduled on day 3's evening. The Turing lecture was an
inspiring one to encourage people to work in computer architecture. By day 3 I had started approaching PhDs and discussing their research
with them. 

By day 4 I had gathered enough courage to start conversations with new peoples. I also met a lot of famous peoples on day 4 and had some
interesting discussions with them. Some of them include Prof Andre Seznec, Prof Onur Mutlu, Ameer Jaleel. The evening was reserved for the
conference banquet at the [Dorothy Chandler Pavilion](http://www.dorothychandlerpavilion.net/). The conference unofficially ends after the
banquet as very less people stay for the last day. 

Overall ISCA 2018 was a really great experience it provided me the much needed exposure to the field that I am interested in and I did some
networking too which should be helpful in the long run. I would like to thank the [Research-I Foundation](https://www.cse.iitk.ac.in/users/rif/)
for funding my visit. I would also like to thank Biswa and Prof Mainak for guiding us throughout without them this wouldn't have been possible.
 

