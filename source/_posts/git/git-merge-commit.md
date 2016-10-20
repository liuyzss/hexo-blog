---
title: git合并分支上指定的commit
date: 2016-10-20 17:25:11
updated: 2016-10-20 17:25:11
tags:
    - git
categories:
    - git
---
merge 能够胜任平常大部分的合并需求。但也会遇到某些特殊的情况，例如正在开发一个新的功能，线上说有一个紧急的bug要修复。bug修好了但并不像把仍在开发的新功能代码也提交到线上去。这时候也许想要一个只合并指定某些 commit 的功能。

假设分支结构如下：

```
dd2e86 - 946992 - 9143a9 - a6fd86 - 5a6057 [master]
                  \
                  76cada - 62ecb3 - b886a0 [feature]
```
再假设 62ecb3 的提交修复了bug，这时候可以用 cherry-pick 合并单个 commit
具体操作：

```
git checkout master
git cherry-pick 62ecb3
```
就这么简单。62ecb3 已经应用在 master 上了（作为一个新的commit）。

> cherry pick 连续多个commit

cherry pick 虽好，但一次只能合并一个commit。合并多个就要用到 rebase 了。再次假设想要把 76cada 和 62ecb3 合并到 master 上。
操作：

```
git checkout -b newbranch 62ecb3
git rebase —onto master 76cada
```
76cada 表示从 76cada 的 commit 开始合并（作为新的commit）。这样就完成了 76cada 到 62ecb3 合并到 master。

参考:
https://ariejan.net/2010/06/10/cherry-picking-specific-commits-from-another-branch/

关于 cherry pick:
https://git-scm.com/docs/git-cherry-pick

关于 rebase:
http://git-scm.com/book/en/v2/Distributed-Git-Maintaining-a-Project#Rebasing-and-Cherry-Picking-Workflows
