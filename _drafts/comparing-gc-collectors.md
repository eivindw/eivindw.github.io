---
layout: post
title:  "Comparing GC Collectors"
date:   2015-12-20 20:30:00 +0100
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

{% lightbox assets/images/mem-gc-default.png --data="mem_gc" --title="Default GC" --img_style="max-width:90%;" %}

{% lightbox assets/images/mem-gc-serial.png --data="mem_gc" --title="Serial GC" --img_style="max-width:90%;" %}

{% lightbox assets/images/mem-gc-cms.png --data="mem_gc" --title="CMS GC" --img_style="max-width:90%;" %}

{% lightbox assets/images/mem-gc-g1.png --data="mem_gc" --title="G1 GC" --img_style="max-width:90%;" %}

### References

* [Java Platform, Standard Edition HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)
* [GCViewer](https://github.com/chewiebug/GCViewer)
