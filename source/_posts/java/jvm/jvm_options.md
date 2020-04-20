---

title: jvm options
date: 2017-09-12 09:06:17
updated: 2017-09-12 09:06:17
tags:
- java jvm
categories:
- java
- jvm
---

### jvm 参数

```bash
-Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError
#-Xmx JVM最大可用内存为
#-Xms 最小堆的大小
#-XX:+HeapDumpOnOutOfMemoryError jvm 内存溢出是生成快照hprof
#-XX:HeapDumpPath 指定快照的目录
#-Xss128k设置每个线程的堆栈大小

```

### heap 导出

```bash
# 当内存溢出是导出到指定目录
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/log/
# 手动导出
jmap -dump:live,format=b,file=tai.dump 16733
```

 

### javaCore 线程堆栈信息导出

```java
1.Getting a Thread Dump Using jstack
2.A Thread Dump Using jVisualVM
3.Generating in a Linux Terminal kill -3 java进程ID
```



