---
author: "Cody Wilbourn"
categories:
- Alerting
comments: true
date: "2018-03-22T16:27:37Z"
slug: sustainable-on-call
tags:
- monitoring
- pager
title: Sustainable On-Call
---

I saw a tweet by Charity Majors that got me thinking--


<blockquote class="twitter-tweet">
Yes, yes. On call sucks and can destroy your life. I know this. Bored now.

On call is a fact of life for anyone who cares about developing high quality software for the long run. So how can we make it *not* suck?

— Charity Majors (@mipsytipsy) [January 31, 2018](https://twitter.com/mipsytipsy/status/958712239648919552?ref_src=twsrc%5Etfw)</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>




On-call is stressful, overwhelmingly the negative type of stress, distress, rather than positive, eustress. I've written about the [stresses of on-call]({{< relref "2017-09-17-stresses-of-on-call" >}}) before--urgency, uncertainty, duration, and expectations.  We all know that distress can contribute to burnout, but individually those four factors are fairly benign. Expectations are part of any job. People on oil rigs work 12+ hours a day for two weeks straight. A number of jobs have uncertain and urgent of tasks, such as first responders or doctors. If these components can be managed, why then can on-call be so miserable?

[Digging deeper]({{< relref "2018-01-02-root-cause-is-plural" >}}), I came to the conclusion that the worst part of on-call revolves around **frequency** and **volume**. Everything we do around improving on-call I believe tries to attack these two causes. Why do these factors impact on-call, and how can they be mitigated?

<!--more-->


# The Problem With Frequency


When I think of frequent notifications, my email comes to mind--the near constant chime of new unread messages, calling out "Hey! Listen!". Email completely dominates a day's work if I let it, even if the emails require no response. That's why people have written so much advice about effectively managing email (exhibits [A](https://lifehacker.com/tim-ferris-email-auto-responses-can-help-you-manage-yo-1605397285), [B](https://hbr.org/2014/04/8-ways-not-to-manage-your-email-and-5-tactics-that-work), [C](http://calnewport.com/blog/2016/03/22/the-case-against-email-strengthens/)). But while a frequent occurrence, email can be time-boxed and dealt with a variety of strategies

The primary difference to me is that in on-call rotations, notifications become the method of tasking. These electronic notifications turn from a request for action to a demand for one. One-off alerts are manageable, but as the pace increases it becomes a crazed game of [Simon Says](https://en.wikipedia.org/wiki/Simon_Says). A new task comes in just as the last was completed, or even before.

This pacing contributes to distress, particularly because on-call is taking place on what is traditionally one's own time. If I was supposed to be working, I'd be at work. I'm not though, I'm at home. On-call exists in this half-state of "living life" and "working," whose nebulousness makes it more bothersome than problems encountered at work during work hours. This is compounded by the fact that, at least in my experience, most on-call repairs are performed remotely without driving into the office, further blurring the line between work and personal time.

Life outside of work has its own sources of stressors. On-call adds interruptions to those stressors, further increasing their intensity. For example: being about to sit down for dinner with your family and receiving a pager alert. There's only so much time in a day to be with them, and the alert takes away from it.

Once in a while these interruptions are manageable, hopefully compensated by extra pay. As the frequency of those interruptions increases, however, the more life outside of work is impacted, and the more on-call starts to drain life like a vampire.


# Mitigating Frequency


As the number of alerts increases in frequency, the more routine on-call becomes. Night after night, doing the same fixes. On-call should not be routine, it is supposed to be a break-glass emergency situation only, otherwise the business arguably needs a shift to cover its needs.

There's a number of strategies that people discuss to deal with making on-call more bearable, and I believe those strategies can be grouped into either human strategies or technology strategies.

The human-based strategies seek to manage frequency by affecting when a given person has to deal with on-call incidents, in order to help prevent burnout.

The technology-based strategies seek to reduce the generation of new alerts.

To reach a sustainable on-call rotation, I believe both strategies are needed.


## Human Strategies




### Compensation Time


Compensation time, or "comp time", is given to employees who have had to deal with on-call incidents--typically a few hours of time off, up to a day's worth. The bar for awarding comp time varies between companies--any time there is  an incident, overnight/weekend incidents, or only after particularly arduous incidents. Sometimes time is automatically given, sometimes manager's discretion. However time is allocated, the goal of this strategy is to give impacted employees recovery time from the on-call alerts and prevent burnout.

The best comp time strategies I've seen involves self-directed comp time with encouragement from management. People will on their own come in late the next morning, or if they have scheduled meetings, leave early, based on the time they felt was needed to recover. For particularly bad on-call weeks or for marathon issues, management steps in and says, "Thanks for all your hard work. We're taking the pager from you, go home and get some rest." Removing the pager keeps the impacted person from being dragged back to work while they are supposed to be recuperating.


### Hiring More People


The cries of every front-line employee, and the dreams of their managers.

Let's face it, hiring more people to deal with an on-call problem is extremely rare. For starters, hiring new employees doesn't happen overnight. Interviews take time; meanwhile, employees are treading water. Hiring also means budget allocations--always short in supply--making hiring a longer term solution than comp time, which can be implemented today.

But if hiring new employees can be justified, which I have seen done before with meticulous documentation, there are three options for the new hires:



	
  1. A new employee, added to the on-call rotation. The on-call frequency for each person has been reduced by spacing out the rotation.

	
  2. "Follow the sun" employees. By hiring people in a significantly different timezone, on-call can be split into two 12-hour shifts covered by different people. This stops the frequency entirely for overnight events, while keeping the daytime frequency. However, overhead is significantly increased, particularly if this is the first overseas location.

	
  3. A 24x7 help desk. This is the most expensive staffing option, and up levels the existing on-call from tier one to tier two support. The 24x7 desk will reduce the number of issues sent to the existing on-call staff by solving some of the problems themselves. Note this does not change the frequency of alerts sent to the 24x7 desk, unless that's a problem they're tasked with reducing in between alerts.




## Technical Strategies


There are also a number of technical strategies that can be implemented to reduce the frequency of alerts, which I'm broadly going to split into two categories -- changing the alerts, and changing the systems generating alerts. As a technical employee, it is far more likely that I can implement these solutions rather than convince management to hire more people.


### Alerting Improvements


[Alerts need to be actionable]({{< relref "2017-07-21-take-that-vacation-eliminate-alerts-dragging-you-back-to-the-office" >}}). Any alerts that cannot be followed by an immediate action contribute to on-call chatter. The action to take when receiving one of these inactionable alerts is to either make it actionable or remove it, thereby reducing the frequency of incoming calls.

The false positive rate should be as close to zero as possible, and an alert's thresholds tuned until it is. Arguably, the false positive rate should be zero, if not by the current monitoring methodology then by some other.

If the alert has a history of crying wolf, on-call will absolutely hold this up to justify why they performed no investigation or action. From a certain perspective, this is another human strategy to deal with the frequency of alerts--minimize actions taken by ignoring what has historically been false. However, this is a negative reaction to dealing with frequency, which benefits neither the business nor the on-call operator, so it should not be justified as a valid strategy to use.


### System Improvements


Remove the source of problematic monitors to reduce the frequency at which alerts are generated.

If a system is sending repeated actionable alerts, then it is a problem. Customers are impacted by the problems associated with this system, which leads to distrust, SLA breaches, and a general reputation of being undependable. This is costing the business money, which can justify fixing those systems.

Add the code, redundancies, or components needed to improve the reliability of infrastructure. Some of the improvements will be more difficult or expensive than others, but over time the health of systems will improve, bringing on-call up with it.


## Both Strategies Needed


Technological solutions alone won't solve the issue of on-call alerts, as it relates to frequency. Certainly, improving the alerting should be prioritized because of the low-hanging fruit with false alarms and inactionable alerts. Almost everything needed to make a strong difference is right there.

What about after the monitors generating the alerts are in a healthy state? Engineers might be inclined to start on the system improvements. That will help, up until the point when the existing environment becomes too large or too complex for the existing on-call staff. Fortune 500 companies have 24x7, multi-tier operations for a reason.


# The Problem With Volume


Another issue encountered on problematic on-call rotations is alert volume. Many monitoring systems will send out bursts of notifications when problems arise, above and beyond what a typical hour or day looks like. Sometimes this is due to overlapping monitor scopes--when one system fails, multiple monitors alert, but other times this could be due to repeat notifications.

Dozens of alerts when things are broken sounds like a wonderful idea initially--everything broken is visible in the notification channel, whether that is email, chat, or a pager. On-call can read the notifications, and fix everything. In my experience, however, this is overwhelming and could lead to decision paralysis on where to start fixing the problem. Which service is most critical and needs repaired first? Is there something more critical that I've missed?


## Solutions


While I split out solutions into categories for frequency, volume pertains to human-computer interaction. This means the solutions are linked between the human component and the technological component. Humans must be able to comprehend the information being given them by the monitoring system.

Some monitors need to be prioritized. Suppose I have a monitor for whether a process is running (to determine if the application exited and auto-restarts failed), and a second if its host is down (to determine hardware failure). When that host fails, both checks send notifications because a downed host cannot run a process. All I care about is that the host failed, as it is the higher priority notification of the two. Once the system is back up, then I care about whether the process fails to start. The method by which monitoring system can do this will vary, but composite monitors--a monitor made to alert based on the combined status of multiple metrics--could be a starting point.

Another option is to only alert on the worst value of a metric. As I mentioned in my post on [overcoming alarm flood]({{< relref "2017-10-06-overcoming-monitoring-alarm-flood" >}}), I had once created a Kafka monitor that alerted per partition lag, when in fact all I cared about that something was bad. After moving to a "worst" monitor for each consumer group, the workflow became: fix the immediate issue, then be informed of the next issue. This cycle repeats until all of the issues have been resolved. I leverage the monitoring system to give a prioritization of work in the on-call situation rather than attempting to make a prioritization from too much information. If I did want more context, I have access to dashboards.

Reduce the overlap between monitors. Some monitoring noise is due to very similar, but slightly different monitors in practice. Separate those differences and make them into distinct checks. Or, if there is no benefit to having the nuances, remove all of the duplicates except one.

Alert volume due to cascading failures, where one system failure causes multiple systems to fail, can be mitigated within those systems. The goal is to decrease the volume of alerts by staying partially up and hopefully not triggering as many alerts as going completely down.



	
  * Adding system redundancies such as failover or high availability

	
  * Caching

	
  * Read-only data replicas

	
  * Exponential Backoff and/or Jitter of request retries

	
  * [Circuit Breakers](https://martinfowler.com/bliki/CircuitBreaker.html)

	
  * Disabling features via feature flags


Of course there's the "more bodies" solution. Adding a secondary or shadow to on-call rotation is a wonderful component of on-calls I've been on. It not provides a human redundancy to the rotation if an alert is missed, but it is also a lifeline for complex or high volume issues. A second pair of eyes can decrease the time required to resolve the issue. The secondary on-call shouldn't be raised on every issue however, only the large or complex ones. Otherwise, they should be considered primary and alerts should dispatch both people simultaneously.


# Closing


What a painless on-call has, in my experience, is culture and sustainability. The on-call culture has to understand that people are delivering the software, and people maintain the software. High quality software demands high quality operations. They're two sides of the same coin. This doesn't mean servers never go down or the code has no bugs; instead, it's how things recover and move forward.

Culture and sustainability sounds vague, because it is. How is it defined? What makes up these components? Different people and companies will define it differently, which is why there is no copy and paste solution for on-call. I believe it's generally accepted that having tests makes software better, but people will argue whether that's manual tests, a QA department, TDD, BDD, or something else that's best. What mix? What percent code coverage is needed? Culture and sustainability will have just as many questions and answers.

I'd love to hear your thoughts. My inbox is open, you can find my info on the [contact page]({{< ref "contact" >}}).
