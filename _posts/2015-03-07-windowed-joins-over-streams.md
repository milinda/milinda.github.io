---
type: post
layout: post
title: Short Survey on Windowed Joins Over Streams
note: Join is an important concepts in data management, which allows to combine fields from multiple data sources by using values common to each. Joins are really inyeteresting in the context of data stream management due to unbounded nature of streams and joins need entire input available to it before the computation.
---

# Introduction

Interest on scalable and reliable software systems/frameworks which enable organisations to query or process data in flight has gained lot of interest in last couple of years due to the need for acting as fast as possible in data driven economy. These systems, generally knows as data stream management systems have lot of interesting properties which set them apart from traditional data management systems due to unbouned nature of streams and low-latency requirements. 

**Join** which is a fundamental operation in standard SQL allows us to combine fields from two or more data sources by values common to each. Join is really interesting in the context of data stream processing due to 

- streams are inherently unbounded 
- and join operator needs the entire input before the computation

Because of the above properties of data streams and join, joins are calculated over windows in the context of data streams. In the rest of the post, I am going to discuss more about properties of windowed joins and different approaches and algorithms used to compute windowed joins in the context of data stream management systems.