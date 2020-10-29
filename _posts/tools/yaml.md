---
layout: post
title: YAML language
categories: [YAML]
description: YAML language
keywords: YAML
---

## YAML
YAML是“另一种标记语言”的外语缩写[1]  （见前方参考资料原文内容）；但为了强调这种语言以数据做为中心，而不是以置标语言为重点，而用返璞词重新命名。它是一种直观的能够被电脑识别的数据序列化格式，是一个可读性高并且容易被人类阅读，容易和脚本语言交互，用来表达资料序列的编程语言。

YAML是"YAML Ain't a Markup Language"（YAML不是一种置标语言）的递归缩写。
在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种置标语言）

YAML比json更简化，但对格式要求高（如缩进）。

### 多行缩进
数据结构可以用类似大纲的缩排方式呈现，结构通过缩进来表示，**连续的项目通过减号“-”来表示**，map结构里面的key/value对用冒号“:”来分隔。样例如下：
```
house:
  family:
    name: Doe
    parents:
      - John
      - Jane
    children:
      - Paul
      - Mark
      - Simone
  address:
    number: 34
    street: Main Street
    city: Nowheretown
    zipcode: 12345
```

注意：
**字串不一定要用双引号标识；**
在缩排中空白字符的数目并不是非常重要，只要相同阶层的元素左侧对齐就可以了（不过不能使用TAB字符）；
允许在文件中加入选择性的空行，以增加可读性；
在一个档案中，可同时包含多个文件，并用“--”分隔；
选择性的符号“...”可以用来表示档案结尾（在利用串流的通讯中，这非常有用，可以在不关闭串流的情况下，发送结束讯号）。

### 单行缩写
YAML也有用来描述好几行相同结构的数据的缩写语法，数组用'[]'包括起来，hash用'{}'来包括。因此，上面的这个YAML能够缩写成这样:
```
house:
  family: { name: Doe, parents: [John, Jane], children: [Paul, Mark, Simone] }
  address: { number: 34, street: Main Street, city: Nowheretown, zipcode: 12345 }
```


### YAML 支持的数据结构有三种。
对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
纯量（scalars）：单个的、不可再分的值


数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。
```
-
 - Cat
 - Dog
 - Goldfish
```
纯量是最基本的、不可再分的值。以下数据类型都属于 JavaScript 的纯量。
字符串   字符串默认不使用引号表示.单引号和双引号都可以使用，双引号不会对特殊字符转义。
布尔值   布尔值用true和false表示
整数
浮点数
Null     null用~表示。
时间     时间采用 ISO8601 格式。iso8601: 2001-12-14t21:59:43.10-05:00 
日期     日期采用复合 iso8601 格式的年、月、日表示 ,date: 1976-07-31


YAML 允许使用两个感叹号，强制转换数据类型。
```
e: !!str 123
f: !!str true
转为 JavaScript 如下。
{ e: '123', f: 'true' }
```

注释：#




