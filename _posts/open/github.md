---
layout: post
title: how to use github
categories: [open]
description: how to use github
keywords: github
---


## 基本操作

git clone  自己fork的私库
git remote add upstream  官库url
git fetch upstream    获取官库的所有分支。只创建到远程的分支的指针，但没有创建本地分支.也不会自动合入到本地哪个分支
git checkout master
git checkout -b mybugfix 
git rebase upstream/master   《--把mybugfix与upstream/master同步对齐

**修改代码**

git commit -s --amend    《--amend促使mybugfix只会有一个commit，提到github就不会乱了。
git push -f origin 分支名    《--推送当前分支到远端fork仓库的“分支名”下，没有会新建
在github上pull request


![k8s-flow](/images/posts/TLA+/k8s_github_flow.png   "k8s flow")  

## 附录
