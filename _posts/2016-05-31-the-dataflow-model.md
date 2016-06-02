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

At one end we need scalable messaging platforms with low latency and high throughputs to transfer these unbounded data sources between interested entities and at the other end, we need low latency and highly scalable stream processing platforms to process these data streams on arrival. This post is more about the processing of data streams. Specifically about Google's [dataflow model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf) and [Apache Beam](http://beam.incubator.apache.org) which is an open source implementation of the dataflow model.

Latency requirements can vary from sub milliseconds to couple of seconds or more. Data streams often represent sequence of events happened in recent past and even-time based processing requirments are the norm. Due to unbounded nature, data stream often get splitted into sequence of chunks on the time domain to reduce the context of the query. Above mention chunking, often reffered to as windowing often require events in a stream to ordered based on event-time and windowing is some time based on fields of the events. Then there are requirements for different correctness guarantees. Some application may need approximate answeres, but application where money is involved often require correct answeres instead of approximations. 

> At the same time, consumers of these datasets have evolved sophisticated requirements, such as event-time ordering and windowing by features of the data themselves, in addition to an insatiable hunger for faster answers. Meanwhile, practicality dictates that one can never fully optimize along all dimensions of cor- rectness, latency, and cost for these types of input. As a result, data processing practitioners are left with the quandary of how to reconcile the tensions between these seemingly competing propositions, often resulting in disparate implementations and systems.

In [The Data Flow Model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf), Akidau et al. propose a programming abstraction based on dataflow model that can be use to express refined data processing requirements mentioned above. Proposed abstraction is general enough to achieve balance between, correctness, latency and cost based on different implementation strategies. The abstractions proposed in this paper is available as an open source project with Flink, Spark and Google Cloud Dataflow backends and SDKs in Java and Python.

## Main Contributions



## Compare this with existing work

## Compare this with Streaming SQL