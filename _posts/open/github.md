---
layout: post
title: how to use github
categories: [open]
description: how to use github
keywords: github
---


## ��������

git clone  �Լ�fork��˽��
git remote add upstream  �ٿ�url
git fetch upstream    ��ȡ�ٿ�����з�֧��ֻ������Զ�̵ķ�֧��ָ�룬��û�д������ط�֧.Ҳ�����Զ����뵽�����ĸ���֧
git checkout master
git checkout -b mybugfix 
git rebase upstream/master   ��--��mybugfix��upstream/masterͬ������

**�޸Ĵ���**

git commit -s --amend    ��--amend��ʹmybugfixֻ����һ��commit���ᵽgithub�Ͳ������ˡ�
git push -f origin ��֧��    ��--���͵�ǰ��֧��Զ��fork�ֿ�ġ���֧�����£�û�л��½�
��github��pull request


![k8s-flow](/images/posts/TLA+/k8s_github_flow.png   "k8s flow")  

## ��¼
