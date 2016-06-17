---
type: post
layout: post
blog: true
title: Performance Evaluation
categories: tech
tags: perf-eval, perf-models
---

I started to reading [Performance Evaluation for Parallel Systems: A Survey](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.30.7619&rep=rep1&type=pdf) to understand basics of performance evaluation of parallel and distributed systems. Below are some important points from the paper's introduction that can be valuable to anyone who works on performance evaluation.

### Objective of performance evaluation

>  Selecting a proper architecture for an application is problem oriented. One architecture that suits one kind of problems may not at all suit another. After a particular architecture is chosen, the following questions may be asked. How will the system perform? What criteria should be used to evaluate the performance? What techniques can/should we use to get the performance values? To answer these questions is the objective of performance evaluation.

### Definition

> Performance evaluation can be defined as assigning quantitative values to the indices of the performance of the system under study. 

### What is performance

> To answer this question is not easy, for performance involves many aspects.

> some factors that must be considered: *functionality, reliability, speed, and economicity*.

Before considering about *reliability*, *speed* or *economicity* the system should be functional, and it should perform what its designer wants it to do. Then the system must be *reliable*. It is hard to achieve 100% reliability. So probability of errors should be studied. After we have a functional and reliable system, we can consider evaluating for speed and economicity (efficiency).

### Performance evaluation

> To evaluate the performance of a system or to compare two or more systems, one must first choose some criteria. These criteria are called metrics.

> To know metrics, their relationships and their effects on performance parameters is the first step in performance studies.

> selecting proper workload is almost equally important.

> After choosing proper metrics and workload, one must consider what technique or techniques should be used.
 
> there are three techniques commonly used in performance evaluation. They are *measurement, simulation and analytical modelling*.

> at the early design stage when the system has not yet been constructed, measurement is obviously impossible, instead, a simple *analytical model* is practical. As the design process goes on, more and more details about the system are obtained. At this stage, *simulation* or more *sophisticated analytical modelling* techniques could be used. Finally, when the system design has been completed and a real system constructed, *measuring* becomes possible.

The [paper](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.30.7619&rep=rep1&type=pdf) is quite old. But I think this paper discuss fundamental stuff everyone should know about evaluating the performance of parallel systems. 





