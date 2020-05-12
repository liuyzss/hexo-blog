---
title: shell wiki
date: 2016-12-15 10:44:57
updated: 2016-12-15 10:44:57
tags:
- wiki
- shell
categories:
- wiki
- shell
---

### 常用命令

#### xargs

``` shell
# xargs与find经常结合来进行文件操作

find . -type f -name "*.log" | xargs rm -rf *

# xargs  -i 参数或者-I参数配合{}即可进行文件的操作

find . -type f -name "*.txt" | xargs -I {} cp {}  /tmp/n/


```

#### 根据行号截取某一段日志

``` shell
grep -n "reg" file
# 找到你要找的日志的行号
cat file | tail -n +line_no | head -n 100
# 查看line_no行 后的100行
```
