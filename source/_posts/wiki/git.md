---
title: git wiki
date: 2016-12-07 09:42:28
updated: 2016-12-07 09:42:28
tags:
- git 
- wiki
categories:
- wiki
- git
---

### 常见问题

### git push fatal: unable to create thread: Resource temporarily unavailable

```
# 系统内存不足导致
git config --global pack.windowMemory "100m"
git config --global pack.SizeLimit "100m"
# git config --global pack.threads "1" (不需要执行？)

```
