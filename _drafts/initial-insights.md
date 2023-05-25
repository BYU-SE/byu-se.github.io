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

### Concept: "Similar" failures or "repeat" incidents

There is a sense that, when proposing actions, incident report authors are attempting to prevent, ["significantly reduce the likelihood of similar events in the future"](https://stripe.com/rcas/2019-07-10), or simply be notified about ["similar errors in the future"](https://www.traviscistatus.com/incidents/sxrh0l46czqn). This got us thinking about the possible meanings of the word similar (or repeat or reoccur or ...), as understood through the proposed actions. So far we have seen actions that seem to be considering the possibility of:

1. Future failures with the same root-cause, trigger, or requiring the same conditions.
2. Future failures with the same symptoms (and therefore similar impacts on the rest of the system), even if the cause is quite different.
3. Current and future queries, regular expressions, code, etc with similar (performance, say) issues. (Ie., the same problem, and therefore risk, in other places.)
4. Future incidents with similar characteristics and response requirements (again, even if the cause is quite different).

One of our interviewees expressed scepticism that incidents ever repeat, and some of the incidents reports suggest that a number of (rare) events needed to coincide for the incident to happen. Nevertheless incident reports report on actions that are thinking about future possibilities. I mean, if it is never going to repeat, why bother? One explanation for this is that even if the exact incident may never occur, future incidents may be similar in some ways. I like to think about it this way: *in this incident we learned something about how our sociotechnical system behaves under a particular scenario, which is input to the ongoing evolution of our system.* While some proposed actions may aim to prevent a precise repeat incident, others may aim to improve the resilience (etc) of the system more generally, with a whole class of similar scenarios (etc) in mind.

### Lightning round

* Some proposed actions, simply affect the prioritization and urgency of work that was already in progress. And it appears to be rare for a major project to come (only) from an incident analysis.
* Understanding deeply all of the conditions needed for the incident to occur (in the way that it did) helps asses the risk of a reoccurrence, and presents ["many opportunities to protect the service against any similar event reoccurring"](https://aws.amazon.com/message/65648/).
