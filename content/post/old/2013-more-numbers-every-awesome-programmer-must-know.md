---
title: More Numbers Every Awesome Programmer Must Know
date: 2013-01-25T09:38:00Z
summary: Mike Taulty shares a series of posts continuing 20 (so far) interesting and surprising aspects of the JavaScript Language
tags: [dev, link]
---

http://highscalability.com/blog/2013/1/15/more-numbers-every-awesome-programmer-must-know.html

[Colin Scott](http://www.eecs.berkeley.edu/~rcs/index.html), a Berkeley researcher, updated Jeff Dean’s famous [Numbers Everyone Should Know](http://highscalability.com/numbers-everyone-should-know) with his [Latency Numbers Every Programmer Should Know](http://www.eecs.berkeley.edu/~rcs/research/interactive_latency.html) interactive graphic. The interactive aspect is cool because it has a slider that let’s you see numbers back from as early as 1990 to the far far future of 2020. 

Colin explained his motivation for updating the numbers:

> The other day, a friend mentioned a latency number to me, and I realized that it was an order of magnitude smaller than what I had memorized from Jeff’s talk. The problem, of course, is that hardware performance increases exponentially! After some digging, I actually found that the numbers Jeff quotes are over a decade old

Since numbers without interpretation are simply data, take a look at [Google Pro Tip: Use Back-Of-The-Envelope-Calculations To Choose The Best Design](http://highscalability.com/blog/2011/1/26/google-pro-tip-use-back-of-the-envelope-calculations-to-choo.html). The idea is back-of-the-envelope calculations are estimates you create using a combination of thought experiments and common performance numbers to a get a good feel for which designs will meet your requirements.
And given most of these measures are in nanoseconds, to better understand the nanosecond you can do no better than [Grace Hopper To Programmers: Mind Your Nanoseconds!](http://highscalability.com/blog/2012/3/1/grace-hopper-to-programmers-mind-your-nanoseconds.html) 11.8 inches is the length of wire that light travels in a nanosecond, a billionth of a second.
Colin's post inspired some great threads [On Reddit](http://www.reddit.com/r/programming/comments/15fgxj/latency_numbers_every_programmer_should_know_by/) and [On Hacker News](http://news.ycombinator.com/item?id=4966363).

To the idea that these numbers are inaccurate [Beckneard](http://www.reddit.com/r/programming/comments/15fgxj/latency_numbers_every_programmer_should_know_by/c7m3e7y) counters:

> It's for putting things into perspective, no matter how much these vary you can be pretty sure that L1 cache latency is about 2 orders of magnitude faster than the memory latency which again is a few orders of magnitude faster than the SSD latence which is again much faster than an ordinary hard drive, and that IS really fucking important to know if you want to be a good programmer.