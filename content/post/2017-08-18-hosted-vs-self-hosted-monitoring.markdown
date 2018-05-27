---
author: cwilbourn3
categories:
- Alerting
comments: true
date: "2017-08-18T21:41:33Z"
link: http://codywilbourn.com/2017/08/18/hosted-vs-self-hosted-monitoring/
slug: hosted-vs-self-hosted-monitoring
tags:
- monitoring
title: Hosted vs Self-Hosted Monitoring
wordpress_id: 487
---

Traditionally, to monitor your servers, your company would set up in-house monitoring with something like Nagios, and that was how things were done.

With the increase in Cloud services, there came an increase in hosted monitoring. If you opt to host on a platform like Heroku or AWS Lambda, you won't even have a datacenter to self-host on!

<!--more-->


## Local (Self-Hosted) Monitoring


**Pros**:

	
  * All data stays in-house, which may be a hard requirement for your company
	
  * Customizable data retention
	
  * Custom integration into your existing reporting, business intelligence (BI), or other internal tools

	
  * Variety of paid and open source packages available to chose from. The more popular software packages will have a large ecosystem of books, training, consultants, and pre-made monitoring checks to leverage.


**Cons:**



	
  * Requires a server to run the software, and people trained in its operation

	
  * Scaling issues in large or distributed datacenters are your problem

	
  * Has a "How do you monitor the monitoring" problem?


In general, the self-hosted route gives you flexibility and full control of your data, so long as you have the resources to run it yourself -- both in terms of server maintenance and application knowledge.

The data retention is going to be a big selling point for most organizations who go this route. Hosted monitoring may get you only 6 months to 2 years of data retention. This is fine for cloud servers, which are created and destroyed frequently, or even bare-metal servers, which may run for 4-5 years and be repurposed multiple times, but it will flop at retaining business metrics like site volume or payments processed. Self-hosted will give you all the data retention you want, so long as you provide the necessary storage. I've seen business metrics going back over 10 years on a self-hosted solution -- the data was just continually migrated as monitoring systems changed over the years.

The "awesome-sysadmin" repository on Github has a curated list of open source applications, including monitoring: [https://github.com/n1trux/awesome-sysadmin#monitoring](https://github.com/n1trux/awesome-sysadmin#monitoring). It's a good place to start researching options if you want to go this route. Some of those monitoring systems listed have "enterprise" editions, which add additional compliance or authentication features on top of the open source "community" editions, in addition to support agreements.


## Hosted Monitoring


**Pro****s:**



	
  * Maintenance and support is covered in your bill

	
  * Typically work better with temporary hosts, like autoscaling servers. Some of the self-hosted packages I've experimented with expect a list of hostnames in a configuration file.

	
  * Increasing number of options, since popular self-hosted tools are adding paid hosted options. You can have your favorite tool and not have to maintain it.


**Cons:**



	
  * Retention periods or number of metrics are tied to your plan, possibly requiring oversized purchases to get the data you need

	
  * The per-metric pricing structures of some companies are hard to project

	
  * May be difficult or impossible to export your data. If you need exporting, check prior to committing to a vendor.


Most hosted solutions require you to install an agent on your servers for best functionality, but many of them do support alternative data feeds like Amazon Cloudwatch metrics.  Expect to pay in the realm of $15-20 per host per month for medium-sized installations, whether it's billed by host or by metric. The list price on the website isn't necessarily what you'll pay -- nonprofit/educational, volume, and annual commitment discounts are available from most vendors.

Per-metric billing seems like it could be inexpensive, but it quickly adds up when you're trying to reach parity with a per-host billed system. To match the CPU metrics graph, you need to track user CPU, system CPU, wait CPU, and idle CPU. Maybe you'd also need steal and nice CPU percentages. That's 4-6 metrics right there. However, if you just want to graph and alert on a few application metrics, per-metric billing could be a low cost way to implement it.


