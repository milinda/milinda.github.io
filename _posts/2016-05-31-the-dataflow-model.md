---
type: post
layout: post
blog: true
title: "The Dataflow Model and Apache Beam"
categories: research
tags: dataflow, stream-processing, streaming-sql
---

**[The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)** Akidau et al., *VLDB 2015* 

As more and more people and devices get connected with each other via Internet to perform day to day tasks, unbounded, unordered and often unstructured high velocity and high volume data streams are the norm. To get most out of this vast amount of data available organisations often turn to stream processing. Stream processing allows us to react to changing conditions fast. Often referred to as *Data Stream Management*, it is an exciting field in data management due to diverse use cases and requirements. 

At one end we need scalable messaging platforms with low latency and high throughputs to transfer these unbounded data sources between interested entities and at the other end, we need low latency and highly scalable stream processing platforms to process these data streams on arrival. This post about the processing of data streams. Specifically about Google's [dataflow model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf) and [Apache Beam](http://beam.incubator.apache.org) which is an open source implementation of the dataflow model.

## Stream Processing Requirments

Latency requirements can vary from sub milliseconds to couple of seconds or more. Data streams often represent sequence of events happened in recent past and even-time based processing requirments are the norm. Due to unbounded nature, data stream often get splitted into sequence of chunks on the time domain to reduce the context of the query. Above mention chunking, often reffered to as windowing often require events in a stream to ordered based on event-time and windowing is some time based on fields of the events. Then there are requirements for different correctness guarantees. Some application may need approximate answeres, but application where money is involved often require correct answeres instead of approximations. 

Latency, windowing and correctness requirements are just three from the eight main requirements outlined in [The 8 Requirements of Real-Time Stream Processing](http://cs.brown.edu/~ugur/8rulesSigRec.pdf). For finer details please refer to [8 Requirements paper](http://cs.brown.edu/~ugur/8rulesSigRec.pdf).

Ideally stream processing platform should meet all the requirements above, but it is not that practicle in real world.

> At the same time, consumers of these datasets have evolved sophisticated requirements, such as event-time ordering and windowing by features of the data themselves, in addition to an insatiable hunger for faster answers. Meanwhile, practicality dictates that one can never fully optimize along all dimensions of correctness, latency, and cost for these types of input. As a result, data processing practitioners are left with the quandary of how to reconcile the tensions between these seemingly competing propositions, often resulting in disparate implementations and systems.

## The Dataflow Model

> We propose that a fundamental shift of approach is necessary to deal with these evolved requirements in modern data processing. We as a field must stop trying to groom unbounded datasets into finite pools of information that eventually become complete, and instead live and breathe under the assumption that we will never know if or when we have seen all of our data, only that new data will arrive, old data may be retracted, and the only way to make this problem tractable is via principled abstractions that allow the practitioner the choice of appropriate tradeoffs along the axes of interest: correctness, latency, and cost.

[The Dataflow Model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf) defines one such abstraction and provides set of guidelines on how to build systems based on those abstractions.

### Main Contributions

### Compare this with existing work

### Compare this with Streaming SQL

## Apache Beam