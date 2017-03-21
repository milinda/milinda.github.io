---
type: post
layout: post
blog: true
title: Discussion about Constraint Programming Bin Packing Models [Paper Summary]
mtitle: Monitoring Play Apps
subtitle: With InfluxDB and metrics-play
categories: tech
tags: perf-models
---

This post summarize following paper I found while looking for pub-sub system performance modeling resources. 

*Jean-Charles Régin and Mohamed Rezgui. 2011.* [Discussion about Constraint Programming Bin Packing Models](http://dl.acm.org/citation.cfm?id=2908672)

## Bin Packing Problem


> In the bin packing problem, objects of different volumes must be packed into a finite number of bins or containers each of volume V in a way that minimizes the number of bins used.
> [*Wikipedia*](https://en.wikipedia.org/wiki/Bin_packing_problem)

This [paper](http://dl.acm.org/citation.cfm?id=2908672) talks about bin packing problem in the context of [cloud computing](https://en.wikipedia.org/wiki/Cloud_computing). In cloud context, bin packing is mainly about assigning virtual machines to bare metal servers under a variety of constraints.

In cloud computing it is not always about minimizing number of bare metal servers used:

> However, the problem is more gen- eral than the classical bin packing problem: minimizing the number of used servers costs less energy but several side constraints must also be considered (e.g., agility, reliability, sustainability).

## Constraint Programming

A robust method is required in scenarios such as clouds where there are external constraints that cannot be defined in advance.

> Constraint Programming is such a method having the capabilities to be easily adapted for considering new constraints.

> In order to obtain a more robust and more general method capable to deal with large scale problems, we need first to clearly identify the advantages and the drawbacks of the current constraint programming models. Mainly, we need to identify what parts of the model are really important and what other parts are secondary. Then, we would like to study the scalability of the current models and identify the current limits.

## Basic Model

* \\( \mathbf{I}\\) - The set of items
* \\( \mathbf{B}\\) - The set of bins
* \\(\mathbf{ci}\_{k} \\) - The capacity associated with item \\( \mathbf{k} \\)
* \\(\mathbf{cb}\_{j} \\) - The capacity associated with bin \\( \mathbf{j} \\)
* \\( \mathbf{x}\_{ij} \\) - The membership variable. \\( \mathbf{x}\_{ij} \\) is equal to 1 if item \\( \mathbf{i} \\) is assigned to bin \\( \mathbf{j} \\) and 0 otherwise.
* \\( \mathbf{y}\_{j} \\) - Is 1 if bin \\( \mathbf{j} \\) has been used

Then the minimization problem becomes:

\\[ min \displaystyle\sum_{j=1}^{m} y_i \\]

With following contraints:

\\[ \forall j \displaystyle\sum_{i=1}^{n} ci_ix_{ij} \le cb_j y_j \tag 1 \\]
\\[ \forall i \displaystyle\sum_{j=1}^{m} x_{ij} = 1 \tag 2 \\] 

> Constraint (1) ensures that the capacity of a bin is not exceeded by the sum of the capacities of the items put in that bin. Constraint (2) states that an item is put in exactly one bin.

> Some slack variables may be introduced in order to add some implicit constraints. Constraint (1) can be refined in where $$ s_j $$ is a variable expressing the capacity of bin j that has not been used.

\\[ \forall j \displaystyle\sum_{i=1}^{n} ci_ix_{ij} + s_j = cb_j y_j \\]

Then we can introduce following constraint about whole system:

\\[ \displaystyle\sum_{i=1}^{n} ci_i + \displaystyle\sum_{j=1}^{m} s_j = \displaystyle\sum_{j=1}^{m} cb_j \\]

## Solutions

We can use two approaches to find the minimum number of bins

1. Start with a large number of bins and try to decrease the number of bins until there is no viable solution.
2. Compute a lower bound and increase the number of bins until we find a solution.

Then there are different ways of bin packing

* first fit decreasing - First bin in which an item can be put is selected,
* best fit decreasing - Select a bin such that remaining capacity after item is pack is minimal,
* worst fit decreasing - Select a bin such that remaining capacity is the largest after packing

Then often we may need to perform symmetry breaking. For example items may have same capacity and they may be interchangeable. But in real life additional contstraints such as data locality limits the symmetry breaking strategies.

Capacitated variables are used to reduce the memory complexity by grouping items that have the same capacity.

The paper describes two concepts called *sum constraint* and *subset sum variables* which I don't really understand.

And in the real world we will have more contraints than above, for example in a cloud environment we may have constraints such that some items (VMs) are can only be assigned to some bins (bare metal servers). For example, VMs with GPUs can only be assigned to servers with GPUs.

I think constraint based bin packing is useful in many other situations as well. For example Kafka partition balancing discussed [here](http://blog.typeobject.com/a-constraint-based-approach-to-kafka-partition-balancing). 
