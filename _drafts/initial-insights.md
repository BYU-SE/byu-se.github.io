---
layout: default
title: "Leaning from failure: some initial insights from a new research project"
---

# Leaning from failure: some initial insights from a new project

We are analyzing the lessons that engineers (and others) learn from failure. Our data source is publicly published incident reports, which tend to include lessons learned and/or actions planned to prevent similar incidents in the future. Broadly, we are interested in:

1. What is learned from incidents (*lessons learned*)
2. What changes are planned and why (*preventative actions*)

It is still early in the project, but we've decided to share some of our initial insights. We'd love your feedback.

## Situating our work

The work is really about system evolution and in particular evolution driven by production failures and the analysis of those failures. So rather than being driven by changing business requirements, or ... the impetus comes from having experienced a failure in that system, gaining insight into the scenarios that may occur in production and they way the system (mis)behaves in those scenarios. It is also worth mentioning that we are considering the evolution of the system in the broadest sense: the people, the infrastructure, the deployment pipelines, etc.

We suppose that the goal of these evolutionary efforts is less about the functionality and more about the reliability, resilience, etc of the system.

(TODO: use related work to explain the space of software evolution and situate our work in that space.)

## Data extraction

So far we have reviewed 10 incident reports, extracting a list of lessons learned and planned actions. For each action in our data we are capturing a summary of the *event* that motivated the action, the planned *action*, the system that is the *target* of the action, and the *goal* or intention of the action. And for each fo those four dimensions our extracted data includes both a summary and a category. [A nice write up by the folks at Travis CI](https://www.traviscistatus.com/incidents/sxrh0l46czqn) listed four actions they plan to take "going forward". Here is how we have "extracted" one of those, just to give you a rough idea of what we are doing with this data:

<table>
  <tr>
    <td>Event</td>
    <td>Defect was not caught in testing (unaccounted for variety)</td>
    <td>Defect deployed</td>
  </tr>
  
  <tr>
    <td>Action</td>
    <td>Validate more failure scenarios (increase diversity of tests) </td>
    <td>Add tests</td>
  </tr>  
  <tr>
    <td>Target</td>
    <td>Test suite in staging environment</td>
    <td>Test suite</td>
  </tr>  
  <tr>
    <td>Goal</td>
    <td>Catch "these kinds of regressions" pre-production</td>
    <td>Catch regression</td>
  </tr>
</table>

We have also conducted several interviews. 

## Initial insights

*(Note: Many of the patterns seem to be about multiple ways to deal with the "same" issue but from different perspectives and in different places in the system. And interesting question is why pursue all of them? Is it for defense in depth or something else?)*

### Pattern: Fix, monitor and isolate

[A database cluster failed to elect a new primary in a very particular (and perhaps rare) scenario at Stripe](https://stripe.com/rcas/2019-07-10) and the incident report described plans to work "with the database maintainers to develop a fix for this underlying fault", add monitoring to notify responders if node failures occur (which in this case triggered the defect), and changes to "prevent failures of individual shards from cascading across large fractions of API traffic". Why all three? Why not just fix the defect? Seemingly because other scenarios may lead to election failure (so they plan to monitor for various node failures), and of course there are many reasons a database shard might similarly fail (so they plan to improve the behavior of the system in such cases, also). 

*(Question: on the isolate topic, is there something interesting here about dependencies and learning about (non-functional) relationships between systems?)*

### Pattern: Fast then better

An [AWS incident report](https://aws.amazon.com/message/11201/) (which documented a scenario in which thread starvation caused a front-end fleet to fail) proposed provisioning larger machines in "the very short term", shortly thereafter (after they could "finish testing an increase in thread count limits") changing the configured limits, and also in the "in the medium term" rearchitecting their systems (adding urgency a project already in progress). The goal of the first couple fixes may be to buy time (mitigating immediate risk, by providing "significant headroom in thread count"), allowing time for the rearchitecting project to be completed, which will eventually be needed as utilization continues to grow.

For what we suppose are similar reasons, [at Cloudflare a failure](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/) in which a regular expression with excessive backtracking caused a global outage (beginning with their fire wall), they proposed to immediately "re-introduce the excessive CPU usage protection that got removed", about one week later to "introduce performance profiling for all rules to the test suite", about two weeks later to switch to a regex engine with "run-time guarantees", and "in the longer term" replace their fire wall engine completely (adding "yet another layer of protection"). In this case it seems each "better" solution was also another layer of protection. 

*(Question: in what sense is each one better and is better the right term?)*

### Pattern: Multi-stage defense

In some cases planned actions address the same issue at different development stages. Again from the Cloudflare incident report, the authors proposed ensuring that no expensive regex caused a similar interruption by: 

* Performing code inspections,
* Adding performance profiling to tests,
* Adding staging to their deployments, and
* Adding protection in production. 

### Pattern: Defense along the cascading path

[A recent incident at Slack](https://slack.engineering/slacks-incident-on-2-22-22/) was caused by "complex interactions between our application, the Vitess datastores, caching system, and our service discovery system." A maintenance action cascaded between these systems in an unexpected way. In response, they are proposing actions or describing lessons learned from each of the major steps along the path of the cascading failure path: (1) the maintenance action, (2) the role of the control plane, (3) database schema and query performance, (3) caching hosts. I think a key insight here is that the incident "highlighted some other risks" including other scenarios that might trigger similar failures.

*(Note: the list above is just a placeholder and could be expanded here, but also needs some attention in 43.yaml.)*

### Concept: "Similar" failures or "repeat" incidents

There is a sense that, when proposing actions, incident report authors are attempting to prevent, ["significantly reduce the likelihood of similar events in the future"](https://stripe.com/rcas/2019-07-10), or simply be notified about ["similar errors in the future"](https://www.traviscistatus.com/incidents/sxrh0l46czqn). This got us thinking about the possible meanings of the word *similar* (or repeat or reoccur or ...), as understood through the proposed actions. So far we have seen actions that seem to be considering the possibility of:

1. Future failures with the same root-cause, trigger, or requiring the same conditions.
2. Future failures with the same symptoms (and therefore similar impacts on the rest of the system), even if the cause is quite different.
3. Current and future queries, regular expressions, code, etc with similar (performance, say) issues. (Ie., the same problem, and therefore same risk, in other places.)
4. Future incidents with similar characteristics and response requirements (again, even if the cause is quite different).

One of our interviewees expressed scepticism that incidents ever repeat(and warned about "fighting the last war"), and some of the incident reports suggest that a number of (rare) events needed to coincide for the incident to happen. Nevertheless these incident reports discuss future looking actions. One explanation for this is that even if the exact incident may never occur, future incidents may be similar in some ways. I like to think about it this way: *in this incident we learned something about how our sociotechnical system behaves under a particular scenario, which is input to the ongoing evolution of our system.* While some proposed actions may aim to prevent a precise repeat incident, others may aim to improve the resilience (etc) of the system more generally, with a whole class of similar scenarios (etc) in mind.

### Lightning round

* Some proposed actions, simply affect the *prioritization* and urgency of work that was already in progress. And it appears to be rare for a major project to come (only) from an incident analysis.

* Understanding deeply all of the conditions needed for the incident to occur (in the way that it did) helps asses the risk of a reoccurrence, and presents ["many opportunities to protect the service against any similar event reoccurring"](https://aws.amazon.com/message/65648/).

* One simple contribution from this work could be a list of *types of scenarios* that may tend to be missed in test suites (etc): various restart scenarios (all VMs at once, etc), pause and resume of a subsystem (now with queued items), network instability (as distinct from "down"), subsystems in mixed (health) states, moving between data centers, (geographically) different cluster topologies, single shard overload, and cache states (including completely cold).

* What is the difference when a learning is articulated as a lesson learned (LL) vs a preventative action (PA)? LLs are lower stakes and do not represent the same kind of a commitment for team. They may also be repeated and eventually lead to some action.

* We have yet to see a report that lists no learnings and not actions. But why couldn't the conclusion be "given our perception of the risks, the impacts, the cost involved in changing we decided not to change anything"? After all there are risks associated with making changes, some of the gains may be marginal, and the risk of a repeat incident may be low.