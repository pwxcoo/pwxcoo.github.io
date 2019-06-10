---
title: Untrack files already added to git repository based on .gitignore
date: 2018-09-25 12:07:28
categories: memo
tags:
- git
---

因为平时用 `git add *` 习惯了。。往往在第一次 commit 之后，发现有几个文件应该 untrack 的，但是这个时候添加到 .gitignore 已经失效了。。

这个时候应该要 rm 掉，重新 commit 。

## Step0: commit all changes

提交当前所有提交，确保更新后的 .gitignore 被提交了。

## Step1: Remove everything from the repository

```bash
$ git rm -r --cached .
```

## Step2: Re add everythin

```bash
$ git add .
```

## Step3: Commit

```bash
$ git commit -m "fix: .gitignore"
```

然后就好了，clean again。

## References

- [Untrack files already added to git repository based on .gitignore](http://www.codeblocq.com/2016/01/Untrack-files-already-added-to-git-repository-based-on-gitignore/)