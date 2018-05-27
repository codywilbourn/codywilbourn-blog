---
author: cwilbourn3
categories:
- Alerting
comments: true
date: "2018-01-11T15:48:45Z"
link: http://codywilbourn.com/2018/01/11/book-review-incident-management-for-operations/
slug: book-review-incident-management-for-operations
tags:
- incident management
title: 'Book Review: Incident Management for Operations'
wordpress_id: 1513
---

I have an interest in bringing ideas from outside of the tech industry and seeing how they fit. After working with Kerim Satirli ([@ksatirli](http://sysadvent.blogspot.com/2017/12/twitter.com/ksatirli)) on my [SysAdvent post about multiple root causes](http://codywilbourn.com/2018/01/02/root-cause-is-plural/), he was kind enough to send me a book "Incident Management for Operations". The book focuses on using the [Incident Management System](https://en.wikipedia.org/wiki/National_Incident_Management_System), pioneered in emergency services for fighting wildfires, in managing outages in tech.

"Incident Management for Operations" was authored by Rob Schnepp, Ron Vidal, and Chris Hawley of [Blackrock 3 Partners](http://www.blackrock3.com/the-team.html). You can find the book on [Amazon](https://www.amazon.com/Incident-Management-Operations-Rob-Schnepp/dp/1491917628) or [Safari Books Online](https://www.safaribooksonline.com/library/view/incident-management-for/9781491917619/).


## In A Nutshell


The authors have adapted the Incident Management System (IMS) for use in IT operations. IMS is a standardized, scalable method for incident response to facilitate coordination between responders. This translates nicely to organizations where separate departments or teams are responsible for different pieces of a business's IT infrastructure, and multiple disciplines are required for incident resolution.

The book lays out the framework for IMS and includes examples of applying the framework to IT. Since implementation can vary in practice (alignment with DevOps, ITIL, etc.), the book stops short of prescribing how to setup organizations, but gives enough information to determine how your organization could adapt to IMS.

The authors provide a number of mnemonics such as "CAN" (Conditions, Actions, Needs), "STAR" (Size up, Triage, Act, Review), and "TIME" (Tone, Interaction, Management, Engagement) to aid in implementing IMS and effectively leading as an Incident Commander. If your organization implements IMS, I'd suggest making a quick reference card with these mnemonics to put on your ID badge holder in case you forget during a 3 a.m. incident.

<!--more-->


## Who Is This Book For?


"Incident Management for Operations" is an introduction to the IMS framework for anyone who would be included on call or needed to interact with people in the Incident Response Team. The Incident Management System itself seems to be geared towards medium to large sized companies who are struggling with coordination between various groups while managing incidents.

The book is a short read at only 150 pages; I finished it in under four hours while taking notes and with some interruptions. If you're short on time, however, there are certain chapters you should prioritize. Chapter 2, "The Incident Management System", is the overview of the framework and should be read by all. Beyond that, you can concentrate on the areas relating to your role or organization:



	
  * If you could or would assume the role of _Incident Commander (IC)_ during an incident, Chapter 3 covers a variety of strategies to effectively lead an incident.

	
  * If you will primarily operate as a _Subject Matter Expert (SME)_, responsible for diagnostics and repairs, I recommend reading Chapter 6, "After Action Review", for the detailed scenarios of what Incident Response Teams should look like in practice. Skim Chapter 3 to become familiar with the Incident Commander role and how to interact with the IC.

	
  * If you are in an extremely large organization, Chapter 4, "Scaling the Incident Response," and Chapter 5,  "Unified Command," cover larger scale outages that may involve additional "kingdoms"--business groups or lines of business.




## My Takeaways


This is not the first time I've seen IMS as an idea, but it is the first time I've explored it and how it relates to IT operations in depth. I thoroughly enjoyed the ideas presented in the book.

While I have been on-call in a number of different systems over the years, I came to the realization that I've never spent enough time thinking about the response process (how to evaluate your response process is covered in Chapter 1). The way by which you respond to incidents is another facet of operations that can be measured, optimized and improved. This framework aims to identify and reliably reproduce key aspects of well-run incident responses.

Ideally, IMS would be a standard across IT like it is in government coordinated emergency response. One of the troubles with on-boarding new employees to on-call rotations is the knowledge of "how we do on-call," which is rarely documented by teams. By having a largely standardized framework, employees could contribute faster and more effectively, while companies could leverage external resources for training--reducing the amount of in-house documentation required.

While smaller companies would benefit the most by being able to reference to an external standard, as far as I could tell the minimum number of people required to respond to each incident is two--an _Incident Commander_ and _Subject Matter Expert_. The authors specifically call out against Incident Commanders playing double-duty. Additionally, the authors recommend against bifurcating your incident process--use IMS in everything from "see it, fix it," to multi-hour outages, in order to ensure people have sufficient practice with the IMS roles. Following both of these suggestions, IMS might prove burdensome to smaller organizations.

If there is a way to scale IMS down to a handful of people, while providing enough respite from pager alerts, I would love to hear about it.

Reading through the book I came to the realization that using IMS will result in an argument for improving your system automation and resiliency if you are having trouble justifying those actions up front. If you are requiring two people to be alerted and dispatched for almost every incident, the largest optimization you can make is to not alert them.

Further Reading: [Pagerduty's incident response documentation](https://response.pagerduty.com/), also based off the information taught by Blackrock 3 Partners. ([Blackrock 3's guest blog post on Pagerduty](https://www.pagerduty.com/blog/peacetime-wartime-devops/)). From that documentation, it seems Pagerduty only triggers the Incident Management System on SEV-1 or SEV-2 (highest severity) outages, or SEV-3 issues that may become SEV-2.
