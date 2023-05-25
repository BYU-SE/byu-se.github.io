---
layout: default
title: "Leaning from failure: initial insights"
---

Here in the BYU SE lab, we are analyzing the lessons that engineers (and others) learn from failure. Our data source is publicly published incident reports. For example, an (incident report published by Stripe)[https://byu-se.github.io/idb/incidents/11.html] listed completed and planned preventative actions such as:

* We implemented additional monitoring to alert us when [database] nodes stop reporting replication lag
* We are working with the database maintainers to develop a fix for the [failover defect]
* We are also introducing several changes to prevent failures of individual shards from cascading [including] circuit-breaking on failed operations.

We have also conducted several interviews. In analyzing the data we are basically taking a software evolution approach and are intersted in 

1. What is learned from incidents (*lessons learned*)
2. What changes are planned and why (*preventative actions*)

It is still early in our the project, but we've decided to share some of our initial insights. We'd love your feedback.

## Initial insights

### Pattern: Fix, monitor and isolate

After [a database cluster failed to elect a new primary in a very particular (and perhaps rare) scenario at Stripe](https://stripe.com/rcas/2019-07-10) the incident report described plans to work "with the database maintainers to develop a fix for this underlying fault", add monitoring to notify responders if node failures occur (which in this case triggered the defect), and changes to "prevent failures of individual shards from cascading across large fractions of API traffic". Why all three? Why not just fix the defect? Seemingly because other scenarios may lead to election failure (so it seems reasonable to monitor for various node failures), and of course there are many reasons a database shard might similarly fail and so "improving" the behavior of the system in such cases is also reasonable. 

### Pattern: Fast then better

An [AWS incident report](https://aws.amazon.com/message/11201/) (which documented a scenario in which thread starvation caused a front-end fleet to fail) proposed provisioning larger machines in "the very short term", shortly thereafter (after they could "finish testing an increase in thread count limits") changing the configured limits, and also in the "in the medium term" rearchitecting their systems (adding urgency to an already in progress project). The consequence, I suppose, of the first couple fixes is to buy time (mitigating immediate risk, by providing "significant headroom in thread count"), allowing time for the rearchitecting project to be completed, which will eventually be needed as utilization continues to grow.

For what we suppose are similar reasons, [at Cloudflare a failure](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/)  a regular expression with excessive backtracking caused a global outage (beginning with their fire wall), and they proposed to immediately "re-introduce the excessive CPU usage protection that got removed", about one week later to "introduce performance profiling for all rules to the test suite", about two weeks later to switch to a regex engine with "run-time guarantees", and "in the longer term" replace their fire wall engine completely (adding "yet another layer of protection"). Actually, in this case it seems each "better" solution was also another layer of protection. 

*(Question: in what sense is each one better and is better the right term?)*

### Pattern: Multi-stage defense

TODO

### Lightning round

* Some proposed actions, simply affect the prioritization and urgency of work that was already in progress. And it appears to be rare for a major project to come (only) from an incident analysis.