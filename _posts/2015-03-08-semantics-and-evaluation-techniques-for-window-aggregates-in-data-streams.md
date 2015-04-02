---
type: post
layout: post
title: Sliding Window Query Evaluation over Data Streams
note: Interesting ideas on sliding window aggregates and joins in data stream processing.
---

## Introduction 

Interest on scalable and reliable software systems/frameworks which enable organizations to query or process data in flight has gained lot of interest in last couple of years due to the need for acting as fast as possible in data driven economy. These systems, generally knows as data stream management systems have lot of interesting properties which set them apart from traditional data management systems due to unbounded nature of streams, low-latency requirements and some operators such as join and aggregates like median require the whole input to be available before performing the computation. 

The blocking operations mentioned above make it impossible to evaluate some queries over streams using traditional techniques. So, *window* operator which divide the stream into possibly overlapping subsets  is used to reduce the scope of the query to a *window extent*.

In this post, I am going to discuss about following two papers which discussed several techniques to efficiently execute window queries over data streams.

1. [Semantics and Evaluation Techniques for Window Aggregates in Data Streams](http://web.cecs.pdx.edu/~tufte/papers/WindowAgg.pdf)
2. [Incremental Evaluation of Sliding-Window Queries over Data Streams](https://www.cs.purdue.edu/homes/ake/pub/TKDE-0148-0405-1.pdf)


## Semantics and Evaluation Techniques for Window Aggregates in Data Streams

Framework for defining window semantics proposed in this paper tag each tuple with a window identifier referred to as *window-id* and the rest of the framework is based on mapping tuples to window-ids and window-ids to tuples. This window definition framework consists of three main functions which does the mapping.

* **windows:** Defines window ids to use for the type of windows specified using the framework
* **extent:** Mapping from specific *window-id* to tuples belong to the window extent denoted by the id
* **wids:** Inverse of *extent*, which maps a tuple to set of *window-ids*

Based on above window specification framework, they propose a one pass window evaluation strategy called Window-ID(WID) approach which 

* reduces memory requirements and execution time
* can utilizes punctuations to gracefully handle disorders in streams
 
Tagging of tuples with a window-id makes the implementation of window aggregates easy because it transform window aggregate to group-by aggregate with *wid* as a additional group-by field. Punctuations are integrated to WID approach to encode ordering information allowing implementations to avoid ordering of out-of-order tuples.

| Input Tuples *(seg-id, speed, ts)*   | After going through tagging *(seg-id, speed, ts, wid)*|
|----------------|-----------------------------|
|t1(s5, 47, 12:10:05)|(s5, 47, 12:10:05, 10-14)| 
|t2(s6, 48, 12:10:02)|(s6, 48, 12:10:02, 10-14)|
|t3(s6, 50, 12:10:30)|(s6, 50, 12:10:30, 10-14)|
|p1(s6, *, 12:11:00)|(s6,*, 12:11:00)|
|t4(s5, 46, 12:10:30)|(s5,46, 12:10:30)|
|p2(s5, *, 12:11:00)|(s5,*, 12:11:00)| 

Authors present three different variants of *windows* and *extent* function based on different characteristics of windows listed below:

* Window queries where range and slide values are specified on window attribute such as event timestamp
* Slide-by-tuple windows range and slide values are specified on range attribute such as timestamp and slide attribute such as row number.
* Partitioned tuple-based windows

Refer the Section 3 of the paper [1] for more details.

For the query evaluation, authors categorized different window types into several groups based on different types of information needed for mapping tuples to set of window-ids. They first identify two different information contexts.

Given a tuple *t*,

* **Forward context** -  its forward context is information about tuples that will arrive after tuple *t*
* **Backward context** - its backward context is information about tuples have arrived before tuple *t*

Having to maintain backward-context is not a issue and backward-context requirement doesn't prevent *wids* function from determining tuple's window-ids upon arrival. Classification of windows is mainly based on forward-context requirement where *wids* function cannot determine window-ids for tuple immediately upon arrival and tuple may get buffered or delayed until tuples which arrive after the current tuple. So there are two main categories based on above:

* **Forward context free (FCF)** - *wids* implementation doesn't require information about forward context. Time-based windows, tuple-based sliding windows, and partitioned tuple-based windows comes under this category.
* **Forward context aware (FCA)** - *wids* implementation requires information about forward-context. Slide-by-tuple windows and variants of that are FCA windows.

Under *FCF*, comes *context free (CF)* windows where implementation of *wids* function doesn't require forward or backward context information. Tuple-based and time-based sliding widows are context free. CF windows doesn't require window operator to maintain any state information for mapping tuple to window-ids.

### Query Evaluation

#### FCF Windows

### Limitaitons

In the framework described above, windows should be explicit. Cannot handle query like following which is essentially a tumbling window aggregate in the context of data streams (when *orders* is a stream). 
  
{% highlight sql %}
SELECT STREAM FLOOR(rowtime TO HOUR) as rowtime, product, count(*) as c 
	FROM orders 
	GROUP BY FLOOR(rowtime TO HOUR), product
{% endhighlight %}
	  

## Incremental Evaluation of Sliding-Window Queries over Data Streams

Coming Soon.

