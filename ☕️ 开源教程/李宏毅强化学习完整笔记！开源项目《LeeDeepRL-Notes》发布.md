提起李宏毅老师，熟悉强化学习的读者朋友一定不会陌生。很多人选择的强化学习入门学习材料都是李宏毅老师的台大公开课视频。 

现在，强化学习爱好者有更完善的学习资料了！**来自 Datawhale 的朋友整理、总结了李宏毅老师的深度强化学习视频教程，并添加了课程笔记，实现了课程内容的完整复现**。

目前，项目已完全开源，包括课程内容、配套的习题和项目，供大家使用。


![](https://imgkr2.cn-bj.ufileos.com/afa617f0-efec-43e6-9032-531e63d7a1b9.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=Q2n6rbFsZbBdm52kMwE3PQXPOkA%253D&Expires=1606049269)


## 1. 李宏毅深度强化学习简介
李宏毅老师现任台湾大学电气工程系副教授，主要研究方向是机器学习，特别是深度学习。他有一系列公开的强化学习课程视频，也是很多人入门的教程。

![](https://imgkr2.cn-bj.ufileos.com/3fe3fd38-2888-4581-9df8-79ae972761fb.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=PbRnSXlXOTQI%252Fd89mZ1kK%252FS9Asg%253D&Expires=1606049298)


李宏毅老师的课程包括很多常见的强化学习算法，比如策略梯度、PPO、DQN、DDPG、演员-评论员算法、模仿学习、稀疏奖励等算法。此外，我们还补充了马尔可夫决策过程、Q-learning、Sarsa、REINFORCE 等强化学习常见的算法及概念。

![「策略梯度」课程中的 PPT，解释了策略梯度的过程](https://imgkr2.cn-bj.ufileos.com/b0fb0f51-5e6b-470c-be27-bbee3a504a66.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=Y8510e9MiyYIWQkWGUAjC0d6Y3Q%253D&Expires=1606049545)


![「近端策略优化算法」课程中的 PPT，展示了重要性采样的问题](https://imgkr2.cn-bj.ufileos.com/75ff270b-0b46-4bb7-a068-7b83515a9b48.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=OJStFmHI0ZzZgw9tzAOdsWlIVcU%253D&Expires=1606049601)

李宏毅老师的《深度强化学习》是强化学习领域经典的中文教程之一。李老师幽默风趣的上课风格让晦涩的强化学习理论变得轻松易懂，他会通过很多有趣的例子来讲解强化学习理论。比如老师经常会用玩 Atari 游戏的例子来讲解强化学习算法。

此外，为了课程的完整性，**我们整理了周博磊老师的《强化学习纲要》、李科浇老师的《百度强化学习》以及多个强化学习的经典资料作为补充。** 对于想入门强化学习又想看中文讲解的人来说绝对是非常推荐的。


但是，考虑到很多强化学习爱好者对于课程笔记的需求，我们不仅仅需要的是教学视频。我们需要一份课程笔记，能够引领学习者的思路，帮助引导他们进入这个领域。因此，就诞生了这款《LeeDeepRL-Notes》李宏毅深度强化学习笔记。

##  2.《LeeDeepRL-Notes》李宏毅深度强化学习笔记
LeeDeepRL-Notes 是 Datawhale 开源组织自《李宏毅机器学习笔记》后的又一开源学习项目，由团队成员王琦、杨毅远、江季历时四个月精心打磨而成，实现了李宏毅老师深度强化学习课程内容的 100% 复现，并且在此基础上补充了有助于学习理解的相关资料和内容，对重难点公式进行了补充推导。

期间，Datawhale 开源组织打造了《深度强化学习基础》的组队学习，在众多学习者共同的努力下，对该内容进行了迭代和补充。下面，让我们来详细了解下工作详情吧。

具体工作：
- 2020 年   6 月 --  2020 年   7 月：笔记整理初级阶段，视频 100% 复现；
- 2020 年   7 月 --  2020 年 10 月：添加相关的习题和项目，对笔记内容及排版迭代优化；
- 2020 年 10 月 --  2020 年 11 月：组队学习《深度强化学习基础》并对内容进行迭代完善；
* 2020 年 11 月：最后内容修正，正式推广。

![10月《深度强化学习基础》组队学习中学习者的评价](https://imgkr2.cn-bj.ufileos.com/2590bc72-7a00-4c6e-b2ed-78a9de6bf786.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=D75aWoeIO78YHhfg0hWJrIO9ZRc%253D&Expires=1606049716)


##  3.《LeeDeepRL-Notes》学习笔记框架
### 3.a 亮点
这份学习笔记具有以下优点：

* 完全将李宏毅老师的讲课内容转为文字，方便学习者查阅参考。
* 为了课程的完整性，我们还整理了周博磊老师的《强化学习纲要》、李科浇老师的《百度强化学习》以及多个强化学习的经典资料作为补充。
* 配有相关的习题和项目。

### 3.b 笔记框架

内容在整体框架上与李宏毅老师的深度强化学习课程保持一致。建议学习过程中将李宏毅老师的视频和这份资料搭配使用，效果极佳。笔记也和课程视频完全同步。

内容导航见下：
![](https://imgkr2.cn-bj.ufileos.com/e606a12e-a8b9-4ea4-9224-1a440dd3f4c3.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=UBzBFOQLzBr2NWqAgG%252F4EQP%252FzHo%253D&Expires=1606049802)


## 4. 笔记内容细节展示
### 4.a 对 Q-learning 概念的解析

![在笔记中重新整理 PPT 内容，并增加了一些注释](https://imgkr2.cn-bj.ufileos.com/ce4bdb9d-cc15-48f2-b2c8-e8efe4edaaf1.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=1ilHDgxVfC543OiGgd5RJB0CvhE%253D&Expires=1606049817)


### 4.b Actor-Critc 算法的引入

![根据内容整理成知识点，方便读者理解阅读](https://imgkr2.cn-bj.ufileos.com/231aa656-d016-47f1-b178-03343bea0585.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=5tO0RT8RtcZjz7qphD30oKgMHRw%253D&Expires=1606049826)

在整理过程中，我们并不对视频语音直接转文字，而是根据内容整理成知识点，方便读者理解阅读。

### 4.c 利用贴近学生的例子解释知识点

![强化学习基本概念的解释](https://imgkr2.cn-bj.ufileos.com/ff0d30fe-c06a-402d-b041-c15f0d111e9c.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=GnrUPIaI5BLWUAymXNTmPofSk%252BU%253D&Expires=1606050224)


## 5. 习题（查漏补缺）
只有教程怎么够，来点儿课后习题和关键字总结帮助大家查漏补缺也是极好的。我们根据每一章的内容，并结合其他的网络资料，原创了课后习题以及关键字的总结，辅助你在更短的时间内查漏补缺，令你更快的将“零碎、无序”的知识“拼接”完整。

### 5.a 关键字让你快速 get 到文章的要点

在每章教程的后面，我们都会结合每章的内容，将定义、具体算法、专业名词等关键字和知识点，使用最短、最精确且最白话的方式总结，供大家吸收与巩固。

![教程第二章部分关键字示意图](https://imgkr2.cn-bj.ufileos.com/42d7bf43-d0c0-4c26-abfc-fe5db3ac36e1.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=UvgEXzztDyAYRXuLcMHk6%252FxDnXM%253D&Expires=1606050242)

### 5.b 习题与参考答案助力你的查漏补缺

除了关键词，我们还提供了章节对应的习题供大家查漏补缺，并且结合其他资料，提供了详细、易懂的答案供大家参考。

![教程第一章部分习题以及对应参考答案示意图](https://imgkr2.cn-bj.ufileos.com/a314ec4f-fad4-49db-abb0-bd4dd0c39a8b.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=faEGmH2QtjKEp4P9YJiMeRKY2Bg%253D&Expires=1606050259)

## 6. 项目（动手实践）
强化学习少了实践怎么行，这边挑了三个项目，都基于流行的 OpenAI gym 环境，让你快速入门，循序渐进，主要包括：

### 6.a 对项目的简易描述

![](https://imgkr2.cn-bj.ufileos.com/7ac7ea56-c7d8-4ca0-9ba3-9ff80a1862e0.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=hrVwnvPM5J5Za8F6AAp3PpCfeG8%253D&Expires=1606050276)

### 6.b 层次清晰的手写代码

![](https://imgkr2.cn-bj.ufileos.com/fc01bf33-05b2-4e4e-821f-9dd78b9e30f1.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=enGH9f0coCPubfyUsBedvObk%252BBY%253D&Expires=1606050285)


将整个强化学习过程分成以上几个子模块，方便拆解与改动，并且契合原论文的伪代码，在```main.py```中提供基本接口：

![](https://imgkr2.cn-bj.ufileos.com/79e2a88a-e178-4ca5-a935-b21115f622e4.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=%252F%252FmtLRCdQcmuAjNn%252BZOzYYi2mpI%253D&Expires=1606050295)


### 6.c 使用 Tensorboard 进行可视化

![](https://imgkr2.cn-bj.ufileos.com/5087a058-b806-4fda-8b87-ed7615afefaf.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=0R%252FQoQKNr7mh1%252Bn6Ad3pLe7eC%252BM%253D&Expires=1606050304)


### 6.d 丰富的持续更新

![](https://imgkr2.cn-bj.ufileos.com/fd280ac9-1aa7-4132-b9df-a9630df37cbc.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=%252FDZgbObvTvAvo2%252FYw6IOdb8rgUg%253D&Expires=1606050312)

在刚刚结束的组队学习中，助教耐心地解答了大家的疑惑，并且会根据反馈的情况，在之后的一个月内，持续更新项目的设计方法和详细的代码思路讲解，敬请期待～

## 7. 配套视频

**视频地址**：
https://www.bilibili.com/video/BV1MW411w79n

## 8. 开源地址

**项目地址**：
https://github.com/datawhalechina/leedeeprl-notes
或点击阅读原文获取，欢迎star！