---
title: AI观察：Kiro IDE 上手：核心概念和Cursor对比
date: 2025-07-20 15:41:52
updated: 2025-07-20 15:41:52
toc: true
categories:
  - AI
  - AI产品
tags:
  - AI与机器学习
  - 开发工具
  - Cursor
  - Kiro IDE
  - 产品对比
description: 深度体验 Kiro IDE 的核心功能，详细对比其与 Cursor 的差异，探讨 AI 驱动开发工具的不同设计理念。
author: 虎太郎
excerpt: 我有幸在 Kiro IDE 内测期间体验了这款新兴的 AI 开发工具，并深入研究了它的三大核心概念：Steering、Spec 和 Hook。相比 Cursor 追求极致的 vibe 体验，Kiro 更像是一位能独立思考和执行的"AI项目经理"，强调规范化、自动化和长期可维护性。特别是 Spec 功能，从需求输入到设计蓝图再到技术方案执行，让开发过程更加工程化和可追溯。如果你正在寻找一个更注重软件工程规范的 AI 开发助手，这篇详细的对比分析一定会给你带来启发。
---
# Kiro IDE 上手：核心概念和Cursor对比

## 写在前面

Kiro截止2025-07-20还是处于不开放内测阶段，我由于在上周新用户注册还开放的时候已经下载并注册了一个名额，所以现在有幸"上车"体验一下。并且在今天仔细阅读了文档，并创建了一个Demo实践了几个核心概念。

Claude 4模型的使用目前还不受限制，并且没有前几天那种经常出现繁忙的情况，虽然在一些"代码回滚到checkpoint"的小细节上和Cursor的体验有一些差距，但是整体体验下来效果还是非常不错，尤其是Spec的特性，带来了AI和软件工程亲密结合的惊喜。

下面我简单介绍一下Kiro的核心概念。

## Steering

官方文档的关于Steering的介绍「*Steering files provide context about your project, helping Kiro understand your codebase, conventions, and requirements.*」意思是Steering文件给项目提供上下文，帮助AI理解代码仓库，规约和需求，一般是存储：

- 产品和目标
- 技术栈和框架
- 项目结构和规约

Steering类似Cursor Rule，在Cursor里面`/Generate cursor rule`的调用会自动读取项目类似pom.xml之类的文件，帮你生成一些类似技术栈描述等文件。

在内容上也不限制默认生成的三个文件，和Cursor Rule一样，你可以自由地补充"编码标准、工作流程和团队最佳实践"。

## Spec

Specs transform high-level feature ideas into detailed implementation plans through three phases: Spec用3个阶段，帮助我们从高层次的feature想法转换为具体的实施计划：

1. **要求**- EARS 符号中符合验收标准的用户故事
2. **设计**——技术架构和实施方法
3. **任务**——离散、可跟踪的实施步骤

> **EARS** 是 **"Everyday Architectural Reasoning and Solutions"** 的缩写，它是一种用于撰写**架构需求**的结构化方法。EARS 最初由 **Software Engineering Institute (SEI)** 机构提出，旨在帮助架构师以一种清晰、一致和可验证的方式表达软件架构相关的非功能需求（NFRs）。

Spec 不同于Steering文件，Spec从一个需求触发，比如 *"添加具有登录、注销和密码重置功能的用户身份验证系统"*。

### 以要求Kiro引入PostgreSQL数据库为例

![image-20250720125712436](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720125712436.png)

录入需求之后，根据现有的steering，自动进入需求校对、设计和计划列表的review阶段

![image-20250720125632401](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720125632401.png)

三个阶段的文档不是一次性生成的，先产出requirement文档之后，会让你进入确认过程，这个时候，你可以修改一些user story，减少不必要的开发。

进入下一个阶段（design、task list）都需手动点击确认，比如下方我点击了从design到task list环节的继续按钮，kiro才会继续生成tasks.md文件。

![image-20250720130907420](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720130907420.png)

只有确认好方案和任务之后，再回来点击finalize task list按钮才行

![image-20250720131057219](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720131057219.png)

切换到 task list的tab，这里清晰展示了kiro帮我们生成的开发计划，每一步骤非常清晰地陈列在下面：

![image-20250720131314715](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720131314715.png)

可见，除了计划，每个步骤都可以单点执行，点击start task就可以进入任务队列里面等待AI执行。在这里还可以继续修改任务里面的细节。**我觉得最出色的是，每一个任务都会关联到前面任务的实现，标注着该步骤的完成涉及到哪些需求的实现**。相比起Cursor，这个特性其实更加适合软件开发的工程特性，每一步实现都清晰可循，关联需求。

在完善好TaskList之后，返回到会话页面，你可以点击按钮，也可以直接对话让AI结束整个Spec环节。

![image-20250720132620735](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720132620735.png)

每一次任务完成，kiro就会更新Task List的UI，完成的任务的开头会有一个Task Completed的标识。

![image-20250720133910713](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720133910713.png)

接下来只需在Task List界面挨个点击Start Task按钮即可，相比Cursor能够更加细粒度地掌控每一个开发步骤。无形中也促使我们去紧盯每一个实现的细节，促使我们去Review AI生成的代码，保证质量的同时，避免类似Cursor一样默默地产出一些容易被忽视的'隐患'。

## Hook

Hook是一个回调操作，可以在如下情况自动执行预定义的操作来消除手动工作：

1. 文件创建删除保存操作触发之后触发Hook
2. 手动调用Hook（类似Cursor里面主动在聊天框里 @某一个 Rule）
3. 指定某一个regex pattern的文件，当它被修改之后，触发Hook（也类似Cursor的Rule文件头配置）

在Hook里面核心就是对回调操作进行描述，AI会根据你的描述去执行操作。

### 以要求Kiro创建一个「为Controller补充swagger注解的Hook」

先使用自然语言描述一个hook，在Kiro的UI界面上添加一个Hook，然后输入你想要实现的回调效果即可。

确定之后，自动在Chat界面生成一个对话session，包含刚刚输入的描述。

![image-20250720145518376](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720145518376.png)

经过处理之后，跳转到Hook详细配置页面。

这里有点像Cursor的Rule上面的文件头的配置，你可以指定当出现什么情况/处理完某一个regex pattern的文件的时候，调用这次Hook，Hook本质上其实也是一次AI的对话调用，所以下面就是instruction（其实就是Prompt）。

![image-20250720145738100](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250720145738100.png)

## 总结

相比Cursor，Kiro更加强调规范、自动化和长期可维护性，个人非常看好这种模式——软件工程本质上也是一门工程学，随意的vibe coding虽然可以带来早期快速创建的正反馈，但是熟悉企业级开发的开发者都知道，一旦项目超过一定的代码行数，AI Coding的时候就很难vibe起来。

Spec是我觉得Kiro最明显的特性，从需求输入 → 产出需求详细说明文档 → 到设计蓝图 → 产出技术方案一步步执行。对我来说，这种流程让我更加关注AI实现的技术细节和步骤，每一次的Task complete，我都会关注到它到底做了哪些事情、编写了什么代码、每一次步骤执行是否都能闭环地完成（比如run一个单元测试/执行一个curl接口调用...）。Spec会持久化地留存在.kiro文件里面，确保了后续操作的有据可依。Spec文件在团队协作上，也可以随着Git推送上去，让别人看到这份改动。另外最重要的是，这份文件可以**持续地随着代码改动不断同步，保持新鲜、有效**。

Cursor Rule在Kiro里，可以同时在Steering和Hook看到它们的影子。Steering和Cursor不同，每次AI对话都会全量地将所有Steering全部阅读，所以在这里，按照官方建议就是存放一些技术栈、编码规约、产品设计等高层的东西。而Hook因为可以匹配文件pattern和监听文件操作事件，更加适合做一些细节的指令，不需要放在Steering内让AI每次操作文件都要读取这份上下文。

相比起Cursor这个优秀的副驾驶或者像Ryo Lu说的，是一个"强大的初级工程师"，Kiro更像是一位能独立思考和执行的"AI项目经理"。当追求极致的vibe体验的时候，Cursor实实在在是一个非常棒的选择，如果是着眼于工程效率和项目长期的健康程度（编码规范性、技术方案文档完整性、代码可测性...），选择Kiro提供的开发范式一定是更好的选择。未来两个IDE一定会互相融合彼此的优点，产品形态上一定会更加相似。等下一次Kiro开放注册名额，一定会涌入大量因为Cursor断供高级模型而选择Kiro的用户。不知道Kiro上面的Claude 4还能使用多久...