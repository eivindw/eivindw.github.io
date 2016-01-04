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
* Intro
  * GC goals (footprint, throughput, latency)
  * 4 collectors (serial, parallel, cms, g1)
* Tools/process
  * Turn on collector + logging
  * Java VisualVM
  * GCViewer
* Results
  * Graphs
  * Numbers
* Conclusion

### Results

#### Serial Collector

The serial collector has low CPU consumption. It runs often and seems to do a good job at keeping the memory footprint low. It has only committed 50% of the 2GB available heap:

{% lightbox assets/images/001/mem-gc-serial.png --data="mem_gc" --title="Serial Collector" %}

#### Parallel Collector

The parallel collector has a much higher CPU consumption than the serial collector, which is expected. It has a significant pattern in the memory usage graph, clearly showing the minor and major collection cycles.

{% lightbox assets/images/001/mem-gc-default.png --data="mem_gc" --title="Parallel Collector" %}

#### Concurrent Mark Sweep (CMS) Collector

The CMS collector has a much less regular pattern both in CPU usage and memory. It seems to be collecting much more memory in the minor collections than the parallel collector, but also has a few very big major collections.

{% lightbox assets/images/001/mem-gc-cms.png --data="mem_gc" --title="CMS GC" %}

#### Garbage-First (G1) Collector

The G1 collector has a significantly higher CPU consumption than all the other collectors. It has a very regular memory footprint, allocating max memory right after startup and keeping it at max. We can also clearly see that it runs small collection cycles constantly.

{% lightbox assets/images/001/mem-gc-g1.png --data="mem_gc" --title="G1 GC" %}

#### Comparison by numbers

These numbers are calculated from the `gc.log`, using the [GCViewer](https://github.com/chewiebug/GCViewer) tool. Seems like the collectors deliver what they promise for my little example application. The serial collector has the smallest memory footprint, but also the lowest throughput and worst (longest) average and minimum pause times. The parallel collector (gc_default.log in the table) has the second best throughput, but the worst maximum pause time. The CMS collector delivers, somewhat surprisingly, the best throughput in the test. CMS also has generally low pause times, except a few maximum pause times lasting more than a second. At last, the G1 collector has the best pause times (minimum, maximum and average), but delivers the second worst throughput in the test.

{% lightbox assets/images/001/comparison.png --data="mem_gc" --title="Comparison by numbers" %}

### Conclusion

These tests shows a clear difference between the alternative collectors that can be chosen on the standard Oracle JVM. It is interesting to see how they utilize CPU and memory resources to achieve their goals.

These tests are really only valid for a small example application. My recommendation is to monitor GC-performance in every application trying differenct collectors over time. Choose the one that best suits your application needs.

### References

* [Java Platform, Standard Edition HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)
* [GCViewer](https://github.com/chewiebug/GCViewer)
