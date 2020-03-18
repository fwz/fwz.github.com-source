title: Google Technical Writing Notes
date: 2020-03-15 23:36:19
tags: [Writing]
categories: [Engineering]
---

Google 出了个良心的技术写作课程，于是花了几个晚上进行阅读，整理出一些笔记，分享出来给大家参考或自查。

整个课程分为两部分，第一部分是基础的文法（适合大部分介绍类文档 ），第二部分有更多的「技术」相关内容，完整阅读 / 学习原课程大概需要 3-4 小时。

# Part 1

## 单词
* 有新词或者用户不熟悉的词出现，要么 link 到他的定义上，要么做一个准确的解释。有太多新词的话，就整理一个术语表。
* 对用词保持一致。中途对变量名改名，程序就无法编译。说明文档也一样，中途切换名称的叫法，用户的大脑也难以准确感知。
* 使用人们不熟悉的缩写词时，在缩写词首次出现时添加全称，后续就可以使用缩写词，记住不要来回切换。
* 不要发明一些可能没什么人用、不会频繁使用的缩写。
* 消除代词歧义（例如他、它、这、那）。不合适的代词使用就是写给读者的空指针。大部分情况下，使用原来的名词会比使用代词更精确，特别是原词和代词之间假如插入了新词，就不应该使用代词。推荐两个词之间不超过 5 个词。

### 练习
Python is interpreted, while C++ is compiled. It has an almost cult-like following.

## 语态
* 主动语态比被动语态更清楚。大多数的读者也是会先将被动语态转化为主动语态然后理解。
* 有时被动语态会省略主语，这样会增大理解难度。

### 使句子清晰
* 挑选精确、具体的动词

| 含糊的动词	| 准确的 Verb |
| --- | --- |
| 点击提交按钮就会**发生**错误 |点击提交按钮就会**触发**错误|
| This error message **happens** when...	| The system **generates** this error message when... |
| We are very careful to **ensure**...	| We carefully **ensure**...  |


* 减少使用 there is/there are
* 减少使用特定的形容词，它们的定义较宽松
* Feed your technical readers factual data instead of marketing speak.

### 简短的句子
* 拆分长句，让每个句子只聚焦于一个观点、想法或者概念。
> The late 1950s was a key era for programming languages because IBM introduced FORTRAN in 1957 and John McCarthy introduced Lisp the following year, which gave programmers both an iterative way of solving problems and a recursive way.

> The late 1950s was a key era for programming languages. IBM introduced FORTRAN in 1957. John McCarthy invented Lisp the following year. Consequently, by the late 1950s, programmers could solve problems iteratively or recursively.
* 拆分带着连词或并列语义的长句，用列表来说明
> To alter the usual flow of a loop, you may use either a break statement (which hops you out of the current loop) or a continue statement (which skips past the remainder of the current iteration of the current loop).

To alter the usual flow of a loop, call one of the following statements:

break, which hops you out of the current loop.
continue, which skips past the remainder of the current iteration of the current loop.

### 减少无用词语

>> Changing the sentence from passive voice to active voice **enhances the clarification of** the key points.
>> Changing the sentence from passive voice to active voice **clarifies** the key points.

### 减少使用从句
假如从句不是延伸主句的语义，而是衍生一个另外的语义，那建议用另外的句子承载。

> Python is an interpreted language, which means that the language can execute source code directly. 

从句进一步延伸主题思想，所以从句的运用是合适的。

> Bash is a modern shell scripting language that takes many of its features from KornShell 88, which was developed at Bell Labs. 

第一个从句延伸主题思想，但第二个从句向另外一个方向发展了，所以第二个从句需要拆分。


## 使用列表和表格
区分无序列表和有序列表。
* 调整列表的列表项顺序不会对无序列表的含义产生影响
* 调整列表的列表项顺序会对有序列表的含义产生影响

列表项之间是平行关系，他们的语法一致，逻辑层次一致，大消息一致，标点符号一致
例子：以下列表项是平行的
* 苹果
* 雪梨
* 橙子

以下列表项不是并列关系：
* 苹果
* 雪梨
* 一江春水向东流

有序列表项最好通过动词来开始

### 创建有用的表格
* 注意创建有意义的表头
* 每个格子内的内容不宜太长，超过两句话的话，建议考虑拆分

### 介绍每个列表和表格
推荐向用户介绍列表和表格对应的上下文。
推荐在介绍的语句加上「以下」两字

## 段落
写作的目的很简单：将和主题相关的内容拆解开，用一条顺畅的逻辑链串联这些内容。

### 写强而有力的开头
段首句很重要，时间紧张的读者会只读段首句。
好的段首句建立段落的中心思想。
Good
> A loop runs the same block of code multiple times. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.

Bad
> A block of code is any set of contiguous code within the same function. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.

### 每个段落只应聚焦一个主题
段落应该展现一段独立的逻辑，并只展现这段逻辑。
修改文章时，应该删掉这些和主题无关的内容，或者移动到其他段落。
### 不要让段落太长，或太短
3 - 5 个句子的段落是读者喜闻乐见的。多于 7 个句子的语句就太长了，通篇是 1 个句子的段落也是有问题。

### 好的段落回答三个问题
* 我们告诉读者什么信息
* 为什么这个信息对读者来说很重要
* 读者如何运用信息，或如何向读者证实此信息

> `<Start of What>` The garp() function returns the delta between a dataset's mean and median.`<Start of Why>` Many people believe unquestioningly that a mean always holds the truth. However, a mean is easily influenced by a few very large or very small data points. `<Start of How>`Call garp() to help determine whether a few very large or very small data points are influencing the mean too much. A relatively small garp() value suggests that the mean is more meaningful than when the garp() value is relatively high.

## 受众
> 好的文档 = 受众完成任务所需的知识或技能 - 受众当前的知识或技能

好的文档提供受众所未具备的信息，所以我们通过以下三步来进行：
1. 定义受众
2. 定义什么问题是受众需要学习的
3. 让文档适合受众阅读

### 定义受众
受众角色可能包含：
* 软件工程师
* 技术工种但非工程师（例如项目经理）
* 科学家 / 研究员
* 专业人士（例如物理学家，生物学家）
* 在读研究生 / 本科生
* 非技术人员

相同角色的受众大概会具备一定程度的基础知识。不同角色的受众的知识背景可能大相径庭。
即使是相同角色的用户，由于专业的细分，他们对特定知识的理解程度可能也不同。
时间也是一个因素，大多数工程师学过代数，但由于工作中不一定使用，所以这部分知识会慢慢变得模糊。经验老到的工程师的知识也会比新工程师丰富。

```
The target audience for Project Zylmon falls into the following roles:

* software engineers
* technical product managers

The target audience has the following proximity to the knowledge:

* My target audience already knows the Zyljeune APIs, which are somewhat similar to the Zylmon APIs.
* My target audience knows C++, but has not typically built C++ programs in the new Winged Victory development environment.
* My target audience took linear algebra in university, but many members of the team need a refresher on matrix multiplication.
```

### 定义受众的收获

用一个列表写下受众可以收获的知识或技能。

### 使内容适合受众
写出一份适合受众阅读的文档需要**同理心**。 你需要提供足够的解释来满足观众的好奇心，而非你自己的好奇心。

* 让术语接近受众，妥善处理缩写。参考「单词」部分。
* 解释对方不一定熟悉，或一定不熟悉的上下文知识（例如项目实现、数据结构等）

> C is a mid-level language, higher than assembly language but lower than Python and Java. The C language provides programmers fine-grained control over all aspects of a program. For example, using the C Standard Library, it is easy to allocate and free blocks of memory. In C, manipulating pointers directly is mundane.

以上段落，
* 对一个没有编程经验的物理学家来说，以下的单词是不容易被直接接受的：
    * language
    * mid-level language
    * assembly language
    * Python
    * Java
    * program
    * C Standard Library
    * allocate and free blocks of memory
    * pointers
* 对一个 Python（但不熟悉 C 的) 程序员来说，难以意识到操作内存和指针的区别。介绍段落可以增加一些 C 和 Python 的对比。

### 使用简单的词语
* 英语成为技术文档的主流语言，按英语并不一定是读者的母语。所以我们倾向用简单的词汇，避免使用晦涩、抽象、[冗长](https://www.google.com/search?q=sesquipedalian&hl=zh-cn)、或罕见的单词吓跑读者。

### 克制文化或俗语特色
假如文档面向全球的读者，那么「任务容易完成」(This task is easy) 比 「小菜一碟」（a piece of cake) 和「瞬间搞定」(Bob's your uncle) 都要好。


## 文档
写好句子和段落以后，就可以开始写文档了。

### 定义文档的范围
一个好的文档会在开篇定义文档的范围，例如：
> 这个文档介绍 NSC 项目的整体设计

更好的文档会额外定义文档不包含的内容（这部分可能是用户期望你介绍的），例如：
> 这篇文章不会介绍 NSC 项目中的电力资源系统 Powerhouse 项目的设计

定义范围除了帮助读者以外，还能帮助作者思考结构，保持内容的组织足够聚焦。

### 定义受众的范围
一篇好的文档会显式定义受众人群，例如：
> 这篇文档是为 NSC 的运维工程师准备的。

如有需要，声明一些前置的知识或者经验，甚至前置读物会更好，例如：
> 这篇文档假设读者理解基本的线性回归方法原理。读者可以阅读[线性回归](https://en.wikipedia.org/wiki/Linear_regression) 获取相关知识。

### 在开篇建立主题
专业作家会在书的第一页花极大的精力来增加读者读下去的可能性。冗长的首页很难让人坚持，所以要做好重复修改首页的准备。
对于长篇文档，提供一个概要。这个概要可能相当短，但仍然要花时间去写。一个沉闷或者模糊的摘要会让潜在的读者失去耐心或者信心。

### 为读者写作
关于如何定义读者，我们可以思考三个问题：
* 谁是你的目标受众
* 受众已经知道了什么？
* 阅读完文档以后，受众可以知道什么？

例如假如你发明了一种新的排序算法，下面的列表可能可以回答上面的问题：
* 文档的读者是我们公司的软件工程师们。
* 大多数读者都在学校学过排序算法，但 25% 已经有多年没有实现过任何的排序算法了。
* 读完文档以后：
  * 读者可以知道算法的工作流程
  * 读者可以在他们熟悉的语言中实现这个算法
  * 读者可以知道哪些情况下这个算法比快速排序要快
  * 读者可以知道哪些极端情况下性能会下降

### 组织文档结构
关于上面的主题，文档的大纲可以如下文所示：
1. 算法简介
   1. O(N)
   2. 伪代码实现
2. 用 C 来实现
   1. 在用其他语言实现时要注意的地方
3. 进一步分析算法
   1. 算法适用的数据集
   2. 极端场景

### 模块化文档
我们通过模块化代码来获得更好的可读性、可维护性、复用性，文档也一样。

## 总结
技术写作需要注意以下几点：
* 使用一致的术语表达
* 避免含糊的代词
* 采取主动语态
* 挑选精确的动词
* 每个句子聚焦一个明确的观点
* 将长句拆成列表
* 注意列表是否有序
* 保持列表项之间的并列关系
* 有序列表项采用动词开头
* 恰当地引入列表或表格
* 段首句要点明段落的中心
* 每个段落聚焦一个主题
* 确定受众需要了解什么
* 为受众编写适合他们背景的文档
* 文章开头要点明文档的中心内容

## 参考资料
[Google Technical writing resources](https://developers.google.com/tech-writing/resources?hl=zh-cn)

## 修改
* 采用一个成熟的样式指引（例如 [style-guide highlights](https://developers.google.com/style/highlights?hl=zh-cn)）
* 对于代码，用成熟的风格，加上代码高亮来展示
* **把自己当成读者**，带着目标读者的上下文读一遍文档
* **朗读文档**，能帮助我们发现有没有使用不恰当的短语，冗长的句子，以及其他不自然的感觉。
* **调整语气**，营造一个对话感强、友好、尊敬读者的文档氛围，尝试让自己听上去成为一个富有智识的朋友。参考[Style and authorial tone](https://developers.google.com/style/tone?hl=zh-cn)
  * 不要过分使用「请」

| Too informal	| Just about right	| Too formal |
| -- | -- | -- |
| Dude! This API is totally awesome!	|This API lets you collect data about what your users like.	| The API documented by this page may enable the acquisition of information pertaining to user preferences.|
| Just like a certain pop star, this call gets your "Telephone" number. The easy way to ask for someone's digits!	|To get the user's phone number, call user.phoneNumber.get().	|The telephone number can be retrieved by the developer via the simple expedient of using the get() method on the user object's phoneNumber property.|
| Then—BOOM—just garbage-collect (or collecter des garbáge, as they say in French), and you're golden.	|To clean up, call the collectGarbage() method.	|Please note that completion of the task requires the following prerequisite: executing an automated memory management function.|

## 组织大型文档

### 长文档的组织形式
组织大型文档的方法有两种：
* 整合到一个大文档中
* 拆分到一批相互链接的中小文档中

我们应该如何选择组织方式，有以下几个考虑点：
* How-to 指引，介绍性总览，概念性介绍，建议用短小的文档
* 详细的教程、最佳实践、命令行参考，较长的文档也无妨
* 假如读者可能来回搜索，那么大型文档会是不错的选择

### 如何组织文档
1. 设计文档的大纲。大纲帮助我们调整我们想讨论的主题的位置。设计大纲没有定式，但可以参考下面的一些方法。
   * 大纲顺序：先告知读者「为什么」，再告知读者「怎么做」
   * 大纲项：无需太详细，告知读者我们要「描述一个概念」/「完成一个任务」即可
   * 大纲项：省略和文档主旨无关的内容。例如介绍项目的使用时，除非有特定的用意，否则可以省略项目的历史，或者通过一个外链，引导到另外一个文档。
   * 概念的解释和实际的运用能成对出现的话，能引起读者的兴趣
   * 评审：假如文档需要被评审的话，那先草拟出大纲并评审，评审通过后再草拟其他细节内容。


2. 练习
调整以下大纲
```
## The history of the project

Describes the history of the development of the project.

## Prerequisites

Lists concepts the reader should be familiar with prior to starting, as well as
any software or hardware requirements.

## The design of the system

Describes how the system works.

## Audience

Describes who the tutorial is aimed at.

## Setting up the tutorial

Explains how to configure your environment to follow the tutorial.

## Troubleshooting

Explains how to diagnose and solve potential problems that might occur when
working through the tutorial.

## Useful terminology

Lists definitions of terms that the reader needs to know to follow the
tutorial.
```

```
## Audience

Describes who the tutorial is aimed at.

## Prerequisites

Lists concepts the reader should be familiar with prior to starting, as well as
any software or hardware requirements.

## Useful terminology

Lists definitions of terms that the reader needs to know to follow the
tutorial.

## Setting up the tutorial

Explains how to configure your environment to follow the tutorial.

```

### 介绍部分

   * 可维护性：你会希望文档也容易维护，所以不要在介绍部分涵盖太多内容
   * 与文档的对应：完成文档初稿后，建议检查一下是否已经准确地覆盖了文档的介绍部分

### 增加导航
好的文章导航内容可以是以下这些内容：
   * 介绍和总结部分
   * 清晰和富有逻辑的主题发展
   * 帮助用户理解主题的标题和子标题
   * 目录（table of content)
   * 指向更深入的信息的链接
   * 指向其他待学习的信息的链接

### 在子标题下增加文字
大部分读者乐意在子标题下看到文字介绍，而非直接增加一个子标题。避免在标题下直接新增子标题，直接例如：

```
## 创建网站
### 运行 hexo 命令
```
增加简要的文字介绍能帮助引导读者，例如：

```
## 创建网站

要创建网站，你可以运行 `hexo init` 命令。命令会展示几个选项帮助你设置网站

### 运行 hexo 命令
```

标题拟定练习：

* About this tutorial
* Advanced topics
* Build the asset navigation tree
* Define resource paths
* Defining and building projects
* Launch the development environment
* Defining and building resources
* What's next
* Define image resources
* Audience
* See also
* Build an image resource
* Define an image project
* Build an image project
* Setting up the tutorial
* Select the tutorial asset root
* About this guide

可能的答案
* About this tutorial
  * Audience
  * About this guide
  * Advanced topics
* Setting up the tutorial
  * Select the tutorial asset root
  * Launch the development environment
  * Build the asset navigation tree
  * Define resource paths
* Defining and building resources
  * Define image resources
  * Build an image resource
* Defining and building projects
  * Define an image project
  * Build an image project
* Defining and building databases
  * Define a database
  * Build a database
* Pushing, publishing, and viewing a database
  * Push a database
  * Publish a database
  * View a database
* Configuring display rules for point data
  * Define, configure, and build vector data
* See also
  * Sample data files
* What's next

### 循序渐进披露信息
注意读者接收信息的速度，避免短期介绍大量的新概念。使用以下的方式有助帮助读者理解。

* 在文档介绍部分介绍术语和概念
* 适度将大面积的文字拆分成表格、图标、标题
* 适度将过长的清单进行聚合
* 使用简单的例子进行介绍，然后逐步增加其他的有趣或复杂的技巧

## 插图
研究指出，为读者提供图片（无论图片的质量如何），都能提高读者对文章的好感。但当然只有有用的图片才能让读者受益。有以下几种做法，帮助我们用图片来传递信息。

* 先写图标的标题，并根据标题创建插图。
* 限制一张图中传达的内容，假如整体内容太多太细，先对内容进行整合。例如将复杂系统先拆分成几个子系统。介绍完子系统组成的完整系统以后，逐个介绍子系统的内部信息。
* 通过高亮关键信息，获取用户的注意
* 插图也需要调整：它是否足够简洁，是否需要拆成多个图，内容中的字体可读性如何，对比度是否足够，有什么价值

## 样例代码
好的样例代码是程序员最希望能看到的。好的样例代码指的是：**正确**，**简洁**，**容易理解**，**易于复用**，且**最小化副作用** 的代码。

### 正确性
样例代码必须满足以下条件：
* （在各个版本都）能通过编译
* 能正确完成他声称能完成的工作
* 尽可能达到「可上线」状态。例如样例代码不应该存在安全隐患
* 遵从语言的语法习惯

好的文档会介绍如何运行样例代码，例如环境安装，依赖库安装，环境变量修改，IDE 调整之类。
作者也可以考虑将样例代码的预期运行结果贴出，以便读者参考。

### 简洁性
样例代码应该足够短，只包含最重要的部分。但不要用差劲的实践来缩短代码，同时，正确性要高于简洁性。

### 易于理解
跟从以下的指引去构建样例代码：
* 使用合适的类、方法、变量名
* 避免使用过于艰深的编程技巧
* 避免多重嵌套的代码
* 使用粗体或不同的颜色来吸引用户注意他应该注意的地方


测试一下，以下哪个样例代码是最好的？
``` go
MyLevel = go.so.Level(5, 28, 48)
MyLevel = go.so.Level(rank=5, 28, 48)
MyLevel = go.so.Level(rank=5, dimension=28, opacity=48)
```

结果应该不言自明了吧。

### 注释
往样例代码增加注释时，建议考虑以下的推荐方式：
* 控制注释的长度，但正确性高于简洁性
* 避免为显而易见的代码写注释，但对专家显而易见不一定对新手也显而易见。
* 为真正需要解释的、不直观的代码写注释
* 当读者已经对相关技术比较熟悉的时候，我们通过注释来解释「为什么」，而不是「做什么」

### 可重用

为了读者能很容易应用或复制样例代码的运行，建议：
* 提供读者能运行代码的充分信息（包括依赖项的信息，如何安装）
* 提供易于扩展或定制化的有用代码
* 减少副作用（你肯定不希望读者用了你的代码上线造成事故）

## 总结
进阶部分的内容，总结如下：
* 使用样式指南
* 站在读者立场思考
* 大声朗读文档（以找到改进点）
* Return to documents well after you've written the draft.
* 找一个靠谱的同伴来审稿.
* 可以先发散地写内容点，然后组织大纲
* 阐述文档的范围和前置条件
* 使用任务式的标题
* 循序渐进地介绍内容
* 插入图片前，先为图片想好标题
* 控制一张图片中的信息量
* 在过渡中引导读者的注意力
* 创建易于理解的样例代码
* 代码注释最好简洁，但不能因为简洁失去清晰
* 不要为显而易见的代码写注释
* 为真正需要解释的、不直观的代码写注释
* 提供正确的做法和错误的做法
* 提供一组可以完成不同复杂度任务的代码
* 持续修改文档
* 为不同类型的用户提供不同类型的文档
* 将文档内容和一些已有的内容进行对比
* 在教程类文档中，通过 example 来强化概念
* 在教程类文档中，指出陷阱或者容易犯的错误