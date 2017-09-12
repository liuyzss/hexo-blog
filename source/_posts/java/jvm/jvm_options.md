---
title: jvm options
date: 2017-09-12 09:06:17
updated: 2017-09-12 09:06:17
tags:
- java jvm
categories:
- java
---

### jvm 参数

```
-Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError
#-Xmx JVM最大可用内存为
#-Xms 最小堆的大小
#-XX:+HeapDumpOnOutOfMemoryError jvm 内存溢出是生成快照hprof
#-XX:HeapDumpPath 指定快照的目录
#-Xss128k设置每个线程的堆栈大小

```
