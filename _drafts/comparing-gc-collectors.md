---
layout: post
title:  "Comparing GC Collectors"
date:   2015-12-20 20:30:00 +0100
comments: true
tags:
- jvm
- gc
- java
---

Automatic memory management through advanced Garbage Collection has been one of the main selling points for Java and the JVM from the beginning. We can simply create the objects we need and the JVM makes sure they are removed when no longer in use. In this article I look at the alternative garbage collectors that are available in the JVM and try to identify the differences between them.

## GC Tuning Overview

Before I show my comparison I give a brief overview of garbage collection tuning goals and the available collectors on the standard Java HotSpot VM.

### GC Goals

When tuning garbage collection there are two primary measures of performance. These give us two competing tuning goals - maximum throughput or minimum latency:

* **Throughput** - This is the percentage of total time not spent in garbage collection considered over long periods of time. So if the application spends 10% of its time in garbage collection the throughput is 90%. If you want your application to perform as many calculations as possible you typically want to tune the garbage collection to maximize throughput.
* **Pause-times/latency** - This is how long the application pauses/appears unresponsive because of garbage collection. It is interesting to consider both maximum pause time, and various statistics like average and standard deviation.

### Available Collectors

There are 4 available collectors in the Java HotSpot VM:

* The **Serial** collector uses a single thread to perform all garbage collection work.
* The **Parallel** collector performs minor collections in parallel.
* The mostly concurrent collectors perform most of their work concurrently. The Java HotSpot VM offers two mostly concurrent collectors:
  * The **Concurrent Mark Sweep (CMS)** collector.
  * The **Garbage First (G1)** collector.

I will show a comparison of both throughput and latency for all 4 collectors in a simple test application.

## Process and Tools

I have created a simple test-application that produces a lot of garbage. It runs for one minute before shutting itself down. The application was created using [Dropwizard](http://www.dropwizard.io) and can be seen on my [github page](https://github.com/eivindw/mem-gc-test/blob/master/server/src/main/java/eivindw/TestApp.java).

The application is run with a maxium heap of 2G with full gc logging turned on. For example with the serial collector:

    java -Xmx2g -Xloggc:gc_serial.log -XX:+UseSerialGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps eivindw.TestApp server

While the application is running I used [Java VisualVM](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jvisualvm.html) to create a graph showing CPU and memory usage. This gives us a nice visual way to compare the application running with the different collectors.

After running I used a tool called [GCViewer](https://github.com/chewiebug/GCViewer) to calculate statistics from the gc log. This creates a report with throughput, pause-times and other metrics.

## Results

### Serial Collector

The serial collector has low CPU consumption. It runs often and seems to do a good job at keeping the memory footprint low. It has only committed 50% of the 2GB available heap:

{% lightbox assets/images/001/mem-gc-serial.png --data="mem_gc" --title="Serial Collector" %}

### Parallel Collector

The parallel collector has a much higher CPU consumption than the serial collector, which is expected. It has a significant pattern in the memory usage graph, clearly showing the minor and major collection cycles.

{% lightbox assets/images/001/mem-gc-default.png --data="mem_gc" --title="Parallel Collector" %}

### Concurrent Mark Sweep (CMS) Collector

The CMS collector has a much less regular pattern both in CPU usage and memory. It seems to be collecting much more memory in the minor collections than the parallel collector, but also has a few very big major collections.

{% lightbox assets/images/001/mem-gc-cms.png --data="mem_gc" --title="CMS GC" %}

### Garbage-First (G1) Collector

The G1 collector has a significantly higher CPU consumption than all the other collectors. It has a very regular memory footprint, allocating max memory right after startup and keeping it at max. We can also clearly see that it runs small collection cycles constantly.

{% lightbox assets/images/001/mem-gc-g1.png --data="mem_gc" --title="G1 GC" %}

### Comparison by numbers

These numbers are calculated from the `gc.log`, using the [GCViewer](https://github.com/chewiebug/GCViewer) tool. Seems like the collectors deliver what they promise for my little example application. The serial collector has the smallest memory footprint, but also the lowest throughput and worst (longest) average and minimum pause times. The parallel collector (gc_default.log in the table) has the second best throughput, but the worst maximum pause time. The CMS collector delivers, somewhat surprisingly, the best throughput in the test. CMS also has generally low pause times, except a few maximum pause times lasting more than a second. At last, the G1 collector has the best pause times (minimum, maximum and average), but delivers the second worst throughput in the test.

{% lightbox assets/images/001/comparison.png --data="mem_gc" --title="Comparison by numbers" %}

## Conclusion

These tests shows a clear difference between the alternative collectors that can be chosen on the Java HotSpot VM. It is interesting to see how they utilize CPU and memory resources to achieve their goals.

These tests are really only valid for a small example application. My recommendation is to monitor GC-performance in every application trying differenct collectors over time. Choose the one that best suits your application needs.

## References/Tools

* [Java Platform, Standard Edition HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)
* [GCViewer](https://github.com/chewiebug/GCViewer)
* [Dropwizard](http://www.dropwizard.io)
* [Example application](https://github.com/eivindw/mem-gc-test)
* [Java VisualVM](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jvisualvm.html)
