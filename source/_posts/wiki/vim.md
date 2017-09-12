---
title: vim 常用操作
date: 2017-09-12 11:02:54
updated: 2017-09-12 11:02:54
tags: 
- vim
categories:
- vim
---

### vim选择不匹配的行并且删除

```
v/pattern/d
g!/pattern/d

pattern 是要匹配的正规
d 代表删除
v选项对应的英文应该是 invert 反转的意思
g代表全部 global
!是取反
g!也是取反之意
```
