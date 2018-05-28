---
author: "Cody Wilbourn"
categories:
- Alerting
comments: true
date: "2017-09-17T16:10:34Z"
slug: stresses-of-on-call
tags:
- pager
- stress
title: Reducing the Stresses of On-Call
---

Being on-call is stressful. It feels like the future of the company--or at the very least your job--depends on your vigilance. When will the pager alert come? How bad will it be?


## Where is this stress coming from?





	
  * **Urgency** - Typically on-call only has a certain amount of time to respond to an incident. The idea of being late to respond is stressful for many. There's also an implied urgency in that down = bad, so services should be restored as quickly as possible

	
  * **Uncertainty** - The failures could come at any time. Want to run to the store? Better be quick. Want to see a movie? You might have to miss the ending. Good luck sleeping, the alert could come as soon as your head hits the pillow or in the middle of the night or become your new alarm for tomorrow morning.

	
  * **Duration** - Long on-call rotations wear you down from the knowledge that there's still more to come. Too short and you have to check your calendar for everything to figure out if you're on-call that day. Frequency has a role too -- with only 1 or 2 people in a rotation your "week of on-call" quickly turns into half the year or the entire year.

	
  * **Expectations** - Either internally or externally, it's easy to be pressured by the expectations of on-call. For the rotation it's your job to fix what's broken. If the environment is broken _you must not be doing your job._


This is all on top of your normal job stress - responsibilities, deadlines, work environment, office politics. And we haven't touched on stress from life outside of work.


## What are the impacts of this stress?


Short answer? Mistakes. Health issues.

The National Institute for Occupational Safety and Health (NIOSH) has a great publication about the impact of stress at work. You can find it on the [CDC website here](https://www.cdc.gov/niosh/docs/99-101/pdfs/99-101.pdf), DHHS (NIOSH) Publication No. 99-101. I've pulled a few quotes from that document.


### Mistakes




<blockquote>The St. Paul Fire and Marine Insurance Company conducted several studies on the effects of stress prevention programs in hospital settings.

[...]

In one study, the frequency of medication errors declined by 50% after prevention activities were implemented in a 700-bed hospital. In a second study, there was a 70% reduction in malpractice claims in 22 hospitals that implemented stress prevention activities. In contrast, there was no reduction in claims in a matched group of 22 hospitals that did not implement stress prevention activities.
—Journal of Applied Psychology</blockquote>


You're thinking: "Great, they made a reduction in malpractice claims and medication errors, but I'm in tech, this doesn't relate".

It does relate. People make mistakes.



	
  * [GitLab](https://about.gitlab.com/2017/02/10/postmortem-of-database-outage-of-january-31/) - Admin ran command on production instead of secondary database, losing 6 hours of data.

	
  * [Reddit](https://www.reddit.com/r/announcements/comments/4y0m56/why_reddit_was_down_on_aug_11/) - 1.5 hours of downtime during planned migration because Puppet wasn't disabled.

	
  * [Amazon S3](https://aws.amazon.com/message/41926/) - Command entered incorrectly by admin, resulting in 2 hours of downtime. Even Amazon's status page broke in this outage.


To be clear: I'm not blaming these companies or employees involved for their outages. It's great that they have publicly available postmortems that we can all learn from.

I'm also not claiming stress reduction or prevention would have made a difference in these particular cases. People are going to make mistakes regardless, but why open yourself up to the possibility of having twice as many, when those could be reduced and you'd have a better workplace to boot?


### Health Issues




<blockquote>Health care expenditures are nearly 50% greater for workers who report high levels of stress.
—Journal of Occupational and Environmental Medicine</blockquote>




<blockquote>[W]orkers who must take time off work because of stress, anxiety, or a related disorder will be off the job for about 20 days.
—Bureau of Labor Statistics</blockquote>


If the business can't afford downtime, can they afford higher health care premiums, or to miss an employee for 20 days?


## How to reduce stress


Going back to the hospital studies (67 hospitals, 12,000 individuals), what did they do to reduce stress?



	
  1. Employee and management education on job stress

	
  2. Changes in hospital policies and procedures to reduce organizational
sources of stress

	
  3. Establishment of employee assistance programs (specifically help and counseling for work-related and personal problems)


Employee assistance programs, while helpful, are most likely not something you can implement if you don't have a role in employee benefits.

Reading this post and the NIOSH report on stress and then sending it to your friends, coworkers, and managers will help educate on on-call stress.

How do we in tech reduce organizational sources of stress?

**Blameless Postmortems**. [Etsy's blog explains](https://codeascraft.com/2012/05/22/blameless-postmortems/). Practicing blameless postmortems will help reduce the stress experienced by the team, reducing the external expectations, because the team culture isn't to attack mistakes. The person on-call knows this and should help with their own internal expectations.

**Proper on-call coverage**. On-call rotations where you're responsible for 24-hour coverage should ideally be a week long -- rotating more frequently leads to too many handoffs and is difficult to plan around. Longer durations impacts the employee's life outside of work. If you have at least 4 people in the on-call rotation, then you'll only be on-call for no more than one week a month. Allowing the on-call rotation to have proper fall-back coverage, for example having a secondary as backup or allowing people to get coverage for a few hours or day will let employees have the flexibility to be able to do meaningful things in their lives outside of work.

**Avoid too many cooks in the kitchen during outages**. If people are hovering over the on-call individual or team trying to solve the issue, asking for constant updates, this adds to the stress of urgency and expectations. Organizationally, you can limit this. Pagerduty has an excellent [on-call response guide](https://response.pagerduty.com/), which appears to be modeled after the [Incident Command System](https://en.wikipedia.org/wiki/Incident_Command_System) or [National Incident Management System](https://www.fema.gov/national-incident-management-system) if you want some ideas. Schedule check-ins for updates, giving time for work to get done.

**No on-call heroes**. One person's "heroic" effort to restore the environment is another person's worst nightmare. They don't want to spend hours solving the problem, or be the only one working on the outage where everyone is relying on them. There's no satisfaction for being a "hero", only dread. When they see others being praised for those actions they want to run the other way. Be mindful of whether you're promoting this sort of behavior in your team's culture or just acknowledging employee contributions. Strive to reduce the need for these situations in the future.

**Improve the systems**. Minimize the uncertainty of being on-call by making it reasonable to be on-call. Set a quantitative goal to reduce the number of after-hour pages within your organization. Some of changes will be easy to make, others may be more involved. But if the entire team is striving towards that goal, progress will be made and the whole team will be helped by it.

Anyone can help drive change in their team and organization to reduce sources of stress. Even small changes will help make the work environment more enjoyable, so do your part where you can.
