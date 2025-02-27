---
title: IDEA中Git Reset选项说明
date: 2024-08-28 22:58:04
tags: Git
categories: 技巧
---

#### 1 . 目标

```
`演示下图的git reset 各选项的效果。`


```
![](./2024/08/28/IDEA中Git-Reset选项说明/1.png)

* * *

#### 2\. Git Reset操作说明

图中说明：  
This will reset the current branch head to the selected commit, and update the working tree and the index accoding to the seleted mode.  
意思是：  
该操作会重置当前分支指针到所选择的提交点，并且更新记录点和根据所选选项更新index状态。

这句话意味着该项操作会影响两件事：提交的记录 和 当前工作区中的文件状态。

* * *

#### 3\. 环境说明

为了简化演示，本次使用master分支。master分支初始状态为下图所示：

![](./2024/08/28/IDEA中Git-Reset选项说明/2.png)

本篇各个选项的效果演示均在“回退到版本1”这个需求下进行。

![](./2024/08/28/IDEA中Git-Reset选项说明/3.png)

弹出选项框

![](./2024/08/28/IDEA中Git-Reset选项说明/4.png)

* * *

#### 4\. 各选项效果说明

**Soft选项：**在选择的回退点之后的所有更改将会保留并被git追踪下来。这就意味着可以在 Version Control 的 Local Changes 面板中查看到它们。

创建新文件demo.txt 并index。使得demo.txt文件能够被git追踪版本。

![](./2024/08/28/IDEA中Git-Reset选项说明/5.png)
![](./2024/08/28/IDEA中Git-Reset选项说明/6.png)

此时我们是在版本2的工作区基础上进行创建的demo.txt，此时我们想要回退到版本1，并使用Soft模式回退。回退的结果如下：

![](./2024/08/28/IDEA中Git-Reset选项说明/7.png)

**Mixed模式：**在选择的回退点之后的所有更改将会保留但不会被git追踪下来。

创建新文件demo.txt 并index。使得demo.txt文件能够被git追踪版本。

![](./2024/08/28/IDEA中Git-Reset选项说明/8.png)

![](./2024/08/28/IDEA中Git-Reset选项说明/9.png)

此时我们是在版本2的工作区基础上进行创建的demo.txt，此时我们想要回退到版本1，并使用Mixed模式回退。效果如下：

![](./2024/08/28/IDEA中Git-Reset选项说明/10.png)

**Hard模式：**在选择的回退点之后的所有更改都会被丢弃。（包括被追踪的和已提交的文件）

在版本2基础上新增文字，形成未提交的版本3.

![](./2024/08/28/IDEA中Git-Reset选项说明/11.png)

回退到版本1，以Hard模式。

![](./2024/08/28/IDEA中Git-Reset选项说明/12.png)

**Keep模式：**在选择的回退点之后的所有已提交的更改会被丢弃。但本地修改的会被完整地保存下来。

在版本2基础上新增文字，形成未提交的版本3

![](./2024/08/28/IDEA中Git-Reset选项说明/13.png)

选择了Keep模式进行回退到版本1的效果如下： 

![](./2024/08/28/IDEA中Git-Reset选项说明/14.png)

说明：上图出现了Git Reset Problem对话框是因为Keep模式会保留工作区修改的内容，所以在回退到指定的提交点后，Idea接下来要处理就是这些在工作区修改的内容，所以询问用户是否有必要保留这些内容。如果没必要保留，就完全可以Hard reset；如果有必要，通常情况下下一步就会需要解决冲突问题。

Hard Reset效果如 4.3 所示，点击[Smart](https://so.csdn.net/so/search?q=Smart&spm=1001.2101.3001.7020) Reset后效果如下：  
![](./2024/08/28/IDEA中Git-Reset选项说明/15.png)

没错，这里的stash 和 unstash 都是自动完成的。

说明：用户也可以自己手动 stash 和 unstash操作，类似压栈和弹栈操作。这一机制在“暂时不想提交现已修改的，但现在必须马上在未修改之前的版本上着手开发另一套事情”的尴尬场景下，可以帮助我们有一个解决方法。这一概念像CPU被中断后如何保存中断现场，在处理完其他任务后，能够恢复当时现场。这里也是：当前开发版本被中断后如何保存当前未提交内容，在开发完成其他事情后，再恢复到这些内容。

* * *

#### 5\. 总结

Soft：在选定提交点之后所做的所有更改都将被暂存（这意味着可以到 Version Control 窗口（Alt+9）的Local Changes 选项卡，以便您可以查看它们，并在必要时稍后提交）。

Mixed：在所选提交之后所做的更改将被保留，但不会暂存以进行提交。

Hard：在所选提交之后所做的所有更改都将被丢弃（已暂存的和已提交的）。

Keep：在选定的提交之后所做的提交更改将被丢弃，但本地更改将保持不变。