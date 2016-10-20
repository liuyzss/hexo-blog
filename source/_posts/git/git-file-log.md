---
title: git查看文件修改记录
date: 2016-10-18 16:01:44
updated: 2016-10-18 16:01:44
tags:
    - git
categories:
    - git
---

> 有时候在比对代码时，看到某些改动，但不清楚这个改动的作者和原因，也不知道对应的BUG号，也就是说无从查到这些改动的具体原因了～

### cd pwd

```shell
cd path
```

### git log --pretty

然后使用下面的命令可列出文件的所有改动历史，注意，这里着眼于具体的一个文件，而不是git库，如果是库，那改动可多了去了～

```shell
git log --pretty=oneline filename
# git log -p [filename] //查看详细修改内容
```
如：
![git-log](pic01.png)

### git show

如上所示，打印出来的就是针对文件的所有的改动历史，每一行最前面的那一长串数字就是每次提交形成的哈希值，接下来使用git show即可显示具体的某次的改动的修改～

```shell
git show 8076b7e13513df335864b21adc043479ab5f9ebd filename
# git show [commit_id] [file] //查看某次提交的修改内容
```
如：
![git-log](pic02.png)

这样的话，很高效、快速的查看指定文件的提交记录和记录详情。
