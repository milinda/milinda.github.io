---
type: post
layout: post
title: "Monitoring Play Apps with InfluxDB"
mtitle: Monitoring Play Apps
subtitle: With InfluxDB and metrics-play
categories: tech
tags: monitoring, play, java, InfluxDB
---

New Relic has awesome support for monitoring web applications written in Play Framework. But I couldn't find any open source libraries or tools closer to it. Recently, I deployed a InfluxDB, collectd and Grafana based solution for monitoring cluster of nodes in our [lab](http://d2i.indiana.edu) and it worked out nicely. So the best solution I could think of was to publish Play app metrics to InfluxDB and do a dashboard using Grafana. So I started to browse through different Play application monitoring solutions available on the web and found [metrics-play](https://github.com/kenshoo/metrics-play) which is a Play plugin written based on Dropwizard's [Metrics](https://dropwizard.github.io/metrics/3.1.0/) library with support for Graphite (InfluxDB has a native Graphite input plugin). 

Even though *metrics-play* doesn't include support for Graphite reporter, I [forked it](https://github.com/milinda/metrics-play) and added support for Graphite reporter. If anyone is interested you can grab the code from [here](https://github.com/milinda/metrics-play) and Graphite support is there in *graphite-publisher* branch. To add this to your Play application you have to do following.

* Add metrics-play with Graphite reporter into you Play app dependencies.  
   
{% highlight scala %}
libraryDependencies ++= Seq(
    "com.kenshoo" %% "metrics-play" % "2.3.0_0.2.1-graphite",
    javaJdbc,
    javaEbean,
    cache,
    javaWs)
{% endhighlight %}

* Next, add the Play plugin **com.kenshoo.play.metrics.MetricsPlugin** to your play.plugins file.

* Enable the Graphite reporter by putting following configuration to you application.conf.

{% highlight js %}
metrics {
  graphite {
    enabled = true
    period = 1
    unit = MINUTES
    host = localhost
    port = 2003
  }
}
{% endhighlight %}

* And make sure you have enabled Graphite input plugin in InfluxDB configuration like below.

{% highlight yaml %}
[input_plugins]
  # Configure the graphite api
  [input_plugins.graphite]
  enabled = true
  # address = "0.0.0.0" # If not set, is actually set to bind-address.
  port = 2003
  database = "play-influxdb"  # store graphite data in this database
  udp_enabled = true # enable udp interface on the same port as the tcp interface
{% endhighlight %}

If you got all the configurations correct you should see the Play app monitoring data including JVM stats in you InfluxDB.

Current *metrics-play* implementation supports only limited monitoring capabilities. I am planning to improve it to publish route level metrics such as number of requests and response times. So keep watching my [fork](https://github.com/milinda/metrics-play) for new features.

