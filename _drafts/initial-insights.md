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
