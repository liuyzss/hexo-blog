---
title: vim 常用操作
date: 2017-09-12 11:02:54
updated: 2017-09-12 11:02:54
tags: 
- wiki
- vim
categories:
- wiki
- vim
---

### vim 常用命令

- 选择不匹配的行并且删除

```bash
v/pattern/d 或者 g!/pattern/d
pattern 是要匹配的正规 d 代表删除 v选项对应的英文应该是 invert 反转的意思
g代表全部 global !是取反 g!也是取反之意
```

- 替换操作

```bash
:s/a/b/ #将当前行第一个a替换为b
:s/a/b/g #将当前行的所有a替换为b
:%s/a/b #将每行第一个a替换为b
:%s/a/b/g #将整个文件的所有a替换为b
:1,3s/a/b/ #将1至3行的第一个a替换为b
:1,3s/a/b/g #将1至3行的所有a替换为b
```

- [vim宏录制与重复操作](https://vimjc.com/vim-recording.html)