---
layout: post
title: "Why Load Average Is Less Important Than You Think"
category: sysadmin
tags: [monitoring]
---
It's not uncommon for someone to come to me and proclaim that this or that server has a load average of *10* and this server only has four cores. Isn't that *bad*?

Well, maybe.

It depends on the workload, and many other variables besides. There are probably many much more important metrics you could measure and alert on which would give you a better idea of whether or not your servers are experiencing a *problem*.

## Just What Does Load Average Measure?

I hear a great number of responses to this question, especially given that it's a common phone interview question in the sysadmin world. Here's the trick: *It changes*.

Recently at work we deployed a newer kernel, and load average universally increased. Did the load increase? No. Did the CPU idle go down? No. I/O wait? Nope.

Same load. Same response times. Same throughput. Different load average.

Linux Journal wrote a fantastic article called [Examining Load Average](http://www.linuxjournal.com/article/9001) back in 2006. It's probably the most informative article I've discovered on the topic, even going so far as to show the portions of the kernel code which deal with the calculation. Sadly, it's now almost six years later and it's a bit out of date. I figured I could contribute by writing up an explanation of load average as it exists in Linux 3.6.5.

According to the comments in the kernel itself (kernel/sched/core.c:2169), "The global load average is an exponentially decaying average of `nr_running` + `nr_uninterruptible`."

(If you're interested, following from that line there's actually a fair bit of interesting commentary on how hard it is to calculate load average on modern systems without introducing a lot of overhead, and the kinds of compromises they make as a result.)

Great, so what does "exponentially decaying average" mean? And what counts as `nr_running` or `nr_uninterruptible`?

## Rate of Decay

When I talk to people about what they think load average represents, most of them expect that it is a mean average of some number over some period (1/5/15 minutes). Exponentially decaying averages are rather different.

Here's the exponential decay algorithm used (kernel/sched/core.c:2246):

{% highlight c %}

    /*
    * a1 = a0 * e + a * (1 - e)
    */
    static unsigned long
    calc_load(unsigned long load, unsigned long exp, unsigned long active)
    {
        load *= exp;
        load += active * (FIXED_1 - exp);
        load += 1UL << (FSHIFT - 1);
        return load >> FSHIFT;
    }

{% endhighlight %}

Each time the load average is recalculated, the kernel gets the list of active processes, and calls that function once for each average (1/5/15 minutes), with the prior load average for that period, an exp determined by the period being measured, and the number of actives.

Here are some useful definitions (include/linux/sched.h:124):

{% highlight c %}

    #define FSHIFT      11      /* nr of bits of precision */
    #define FIXED_1     (1<<FSHIFT) /* 1.0 as fixed-point */
    #define LOAD_FREQ   (5*HZ+1)    /* 5 sec intervals */
    #define EXP_1       1884        /* 1/exp(5sec/1min) as fixed-point */
    #define EXP_5       2014        /* 1/exp(5sec/5min) */
    #define EXP_15      2037        /* 1/exp(5sec/15min) */

{% endhighlight %}

So, let's say as an example, we're going to be updating the one minute average. The last sample was 0.17, and we have 5 active processes. The math in the kernel is done with fixed-point. I'll try to demonstrate it as floating point. Hopefully it'll be easier to grasp that way.

* `load` starts as 0.17, and is then multiplied by 1/exp(5/60): 0.1564
* We then add 5 * (1 - 1/exp(5/60)), which is 0.3997: 0.5661
* You can see the code adds what amounts to 0.5, which is to force rounding up where appropriate before shifting the end off. (This was added in November 2010.)

Now, say the active processes then drops to zero by the next calculation:

* `load` is 0.3997, which is mulitplied by 1/exp(5/60): 0.3677
* We then add 0 * (1 - 1/exp(5/60)), which is 0! The new load is 0.3677

If we then stay at 0 actives for several successive iterations, the proportion that load average falls by stays the same relatively, but gets smaller in absolute terms. A sample will lose about 25% of its value about 15 seconds in, but hits 50% after 40 seconds. Hitting 75% takes a full 80 seconds. (Contrary to what you'd expect of a one-minute average.)

With the value used for the five-minute average, you get

* Lost 25% after ~85 seconds
* Lost 50% after ~210 seconds
* Lost 75% after ~420 seconds

After five minutes, a sample decayed by the five-minute constant has only lost 63.3% of its value. This is about the same for the other constants.

So after the nominal period reflected by an average, a sample is still represented in the number you're seeing. That surprised me quite a bit when I first found out about it.

Here's a graph to really hammer home how the exponential decay averages work out. It's based on a load average of exactly 1.0, followed by samples at 0, so you can see how long it takes to work off one process. I've added a horizontal line to show the point where one would *expect* the average to have hit zero.

![graph of load average exponential decay](/assets/images/exponential-decay.png)

## Okay, So What Counts As "Active"?

Now let's delve into what the kernel considers an active process for the purposes of this calculation.

Load averages are only recalculated every five seconds. (NOTE: Double check this?) If it counted every process that was at all active during those five seconds, there would be pretty bad inflation.
