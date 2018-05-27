---
author: cwilbourn3
categories:
- Alerting
comments: true
date: "2017-10-06T19:20:26Z"
link: http://codywilbourn.com/2017/10/06/overcoming-monitoring-alarm-flood/
slug: overcoming-monitoring-alarm-flood
tags:
- monitoring
title: Overcoming Monitoring Alarm Flood
wordpress_id: 788
---

You've most likely had 10, 20, 50, or even more alerts hit your inbox or pager in a short span of time or all at once. What do you call this situation?

It turns out, there's a name for this influx of alerts--"alarm flood".

"Alarm flood" originates in the power and process industries, but the concept can be applied to any industry. Alarm flood deals with the interaction between humans and computers--specifically more automated alerts than the human element can process, interpret, and correctly respond to. It is the result of multiple small changes, redesigns, and additions to a system over time: Why would you not want to “let the operator know” that a system has changed states?

Alarm flood has been discussed in those industries for at least the past 20 years, but it was formally defined in 2009 in the ANSI/ISA 18.2 Alarm Management Standard as 10 or more alarms in any 10 minute period, per operator.

In tech, the "operator" will be the person on call. In a smaller operation with only one or two engineers on call at a single time, any significant event could turn into a flood, making it difficult for the engineer to identify and address the root causes. How do we fix this flood state and provide better information to our engineers?

<!--more-->


## A Single Monitor Flood


I've actually been guilty of generating alarm floods by virtue of how I set up a monitoring alert.

It wasn't too long ago that I created an alert for an Apache Kafka topic and consumer group to measure delays. Kafka is a pub/sub system where a topic is a message queue, and the consumer group is the subscribers in aggregate. The delay is how far the consumers are behind the most recent message.

In order to be a scalable system, Kafka divides topics into partitions, each of which could have its own delay. It is possible that a single subscriber stops reading their partition while the rest proceed, leading to a condition where the system appears to be functioning, but 1 / n partitions worth of data is not being processed.

To avoid this scenario, I setup a monitor that alerts if the lag goes over a threshold for any given partition. When the alarm is received, we would remedy the dead consumer appropriately. Copy and paste that monitor around a few times for different topics and consumer groups, and presto -- full monitoring coverage.

The first few times the alert activated, everything was fine. We had a single process failure and handled accordingly.

One time, however, the cause of the delay was actually downstream of the consumers -- the database was having trouble keeping up with number of writes sent to it. This database slowdown put back-pressure on the consumer processes, who wouldn't read the next item from Kafka until their current item had the write acknowledged. In turn, these consumers slowed down reading new data, across the system.

Pretty much at once, every partition alert triggered. We received over a hundred emails in the span of a few minutes. First, it was the warning notifications. After a few minutes the critical threshold was reached and the pager notifications started. The person on-call couldn't even finish acknowledging the alerts they received before more came in, let alone read and interpret the messages.

After the failures were finally corrected, we discussed this alert in our postmortem analysis. We found that what we needed instead of a per-partition monitor was a simple maximum -- if _something_ was lagging, it needed remedied. Our remedy process didn't require us to identify the individual impacted consumer; we restarted all consumers in the group if any one failed. If we truly wanted to know _which partition_, we could look at the dashboards powering this alert.

Besides poorly designed monitors like the one I created, rapid state oscillations (also known as "[flapping](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/flapping.html)") can cause monitor floods. One oscillation is between `NO DATA` and `OK`, such as when a device goes into low-power mode and no longer sends data. Another oscillation is when the monitor values have a steady state approximately equal to the defined alert threshold. Small changes tip the monitored value to either side of the threshold, resulting in flapping.

Some monitoring systems can compensate for the state flapping and reduce the number of alerts they generate if you manually mark the monitor as potentially flapping. Otherwise the solution to this problem is to re-think what information you're actually watching for and how the alert should be triggered.


## Multi-Monitor Floods


While you can flood yourself with a single monitor, the "spirit" of an alarm flood is that you have multiple monitors from separate sources all indicating problems at once. This in turn becomes too much for the operator to handle, since they can't investigate ten or more different issues at once.  Operators may not even be aware that another alert came in, especially if these are audible alerts (common in industrial situations) and the sounds start bleeding into each other.

Most monitors are configured for state changes -- `run == good`, `stop == bad`. Switching between states will trigger alarms. `WARNING` and `CRITICAL` alerts are typically numeric thresholds to define a state transition.

In tech we can propagate errors throughout the entire environment and give ourselves too much information, especially with ill-tuned state transitions.

Consider the dreaded half-dead state. The system is neither fully alive, nor fully dead -- sending enough traffic to avoid being marked dead, but clearly not doing significant work. Maybe this traffic is even corrupted, sending bad responses some of the time.

How might this manifest in your environment?



	
  * Connection errors in the application

	
  * Connection resets at the TCP level

	
  * Parsing errors when the data comes back corrupted in or incomplete

	
  * Full disks, caused by errors or retries being logged to file

	
  * System automated failover and failback attempts

	
  * Autoscaling to handle incoming traffic load


All of these alerts can happen on the failed system, the peers to that failed system, or upstream to that system. The same type of alert can be triggered on multiple hosts at once. Find the root causes and resolve them.

Good luck.


## What To Do About Alarm Floods


Unfortunately, one of the reasons alarm flooding is so prevalent is that the issue is not easy to solve. Alarm flood is part of an entire topic of study called [alarm management](https://en.wikipedia.org/wiki/Alarm_management), more broadly spelled out in ANSI/ISA 18.2. [ISA has a white paper if you'd like to read further](https://www.isa.org/standards-and-publications/isa-publications/intech-magazine/white-papers/pas-understanding-and-applying-ansi-isa-18-2-alarm-management-standard/). Part of the 18.2 standard is a 10-stage lifecycle structure, and understanding that structure and using it as a guide or framework to develop your environment will help reduce monitoring alerts and floods.

Begin by keeping a log of your frequent, nuisance, or redundant (X is always triggered when Y is triggered) alarms. With your team, go over the alerts and determine if they're actionable and necessary. Beyond addressing individual alerts, work towards the the alarm lifecycle framework for your environment (it won't happen all at once!).

**Develop an alarm philosophy. **What is the specific, quantifiable goal you're trying to achieve with your alerting? What strategy will you take to accomplish this? Monitoring philosophies have been broadly covered in a number of monitoring books and talks, with ideas such as "prefer monitoring end-user experience". Have a clear pattern to the design of your alerts.

**Ensure alarms are rationalized. **Define a response to an alarm (e.g. a [runbook](https://en.wikipedia.org/wiki/Runbook)) and the reason behind it _before_ implementing the alarm. How does it fit into your alarm philosophy? Rationalization can help prevent knee-jerk reactions to outages where monitors are demanded to cover the specific problem that occurred, regardless of the likelihood of the event happening again in the future.

**Detail design prior to implementation. **Fit new alarms into your philosophy and your existing alerts. Ensure the humans interacting with the alarm can understand it, both in the context of the single alarm and the whole system. This can help during cases of alert flood. Detailed design may involve needing visualizations such as grouping related alarms, rollups, or physical diagrams of failures.

**Monitor and assess your alerts.** Verify newly created alerts perform as intended. Ensure your existing alerts are still functional and relevant in today's environment. Avoid stale monitors.

**Audit your environment. **Periodic reviews to verify everything is adhering to the other components of the life cycle. Are you checking for stale alarms, are the alarms hard to understand, are they not fully rationalized, do they follow the high level philosophy? You can track the frequency and time your environment is in alert flood, as well as the volume of messages, giving you quantitative measures to back up your audits.

Using frameworks like the ISA 18.2 Alert Management standard can help you identify and communicate weaknesses in your environment, so you can make the appropriate improvements. Consider your monitoring as a system that needs to be architected in the same way your applications need architected and designed, not as a feature bolted on.

Additional Reading:



	
  * _Alarm Management: A Comprehensive Guide, Second Edition_ by Bill Hollifield and Eddie Habibi

	
  * _Alarm Management for Process Control_ by Douglas Rothenberg





