---
layout: default
title: "Components types and the role they play in cascading failures"
author: "Matt Pope"
---

_Here in the BYU SE lab, we've been analyzing the way that failures cascade between system components. From [40 publicly published incident reports](https://byu-se.github.io/idb/), we have identified over **100 failure pairs**, which are simply two failures, where the first failure is described as being the cause of the second failure. In a series of posts we'll be sharing some of our early findings. For more details about our data set see [Barbara Chamberlin's MS thesis](https://scholarsarchive.byu.edu/etd/9474/)._

A failure may cascade across many parts of a system and at times the severity of the failure increases as the failure propagates through the system. There is an enormous amount of engineering effort and resources brought to bear to increase the resiliency of a system and prevent failures--and yet we still observe (with unfortunate frequency) the failure of these systems. [Dr. Richard Cook's "How Complex Systems Fail"](https://how.complexsystems.fail/) asserts that:

> **Complex systems contain changing mixtures of failures latent within them.**
> The complexity of these systems makes it impossible for them to run without multiple flaws being present. Because these are individually insufficient to cause failure they are regarded as minor factors during operations. Eradication of all latent failures is limited primarily by economic cost but also because it is difficult before the fact to see how such failures might contribute to an accident. The failures change constantly because of changing technology, work organization, and efforts to eradicate failures.

## Component types or roles

One of our research questions in this project has been *"does the role of the component in the system play a part in the propagation of failures?"* To answer this question we have analyzed the components that were involved in 105 failure pairs. We'll call the first component in the pair (the one that experienced the initial failure) **A** and the second component in the pair (the one that experienced the resulting failure) **B**. In some cases we also noticed that a third component (**C**) was involved in the cascade, even though it didn't fail. We grouped the components into general categories and counted how many times each component type was A, B, and C. Here are some observations we think are worth mentioning.

<p align="center">
  <image src="/assets/component-types.png" width="75%"/>
</p>

- **Edge systems (disproportionally) receive failures**: These components often functionally depend on other components but no other components directly depend on them. They are much more often B than A or C, meaning that a cascading failure rarely begins with components of this type, but more often failures cascade to components of this type.

- **Storage systems (disproportionally) initiate failures**: We suspect that storage components are prevalent because they represent hard dependencies for many services and have additional failure modes stemming from complexity involved in adding scalability, redundancy, etc.

- **Support systems (disproportionally) mediate failures**: Support and infrastructure components are, in particular, involved in facilitating a cascading failure between two components. These systems play a role in monitoring, health checking, proxying, replicating state between, managing, and controlling, which means they can play a role in propagation without failing themselves.

Our intention in sharing these numbers is not to make general statistical claims about all failures in every system, but simply to characterize the data we have collected. We hope to generate discussion regarding these results - and help researchers and practitioners improve system resiliency by looking at cascading failure in a systematic way.




