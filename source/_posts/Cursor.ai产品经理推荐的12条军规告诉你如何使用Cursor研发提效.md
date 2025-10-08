---
title: 「AI观察」Cursor.ai产品经理推荐的12条军规告诉你如何使用Cursor研发提效
date: 2025-06-18 20:48:20
toc: true
categories:
  - AI
  - Prompt Engineering
tags:
  - AI与机器学习
  - Prompt工程
  - 研发提效
description: 本文详细解析Cursor.ai产品经理Ryo Lu推荐的12条Cursor使用规范，旨在帮助开发者和产品经理掌握AI编码工具的正确姿势，从而显著提升研发效率、优化代码质量，并有效规避常见误区。适合所有希望通过AI提效的专业人士阅读。
author: 虎太郎
excerpt: 还在为端到端研发交付效率低下而烦恼？Cursor.ai产品经理Ryo Lu分享了12条Cursor使用军规，助你告别面条式代码，高效产出干净代码，全面提升研发效能。立即学习这些实战技巧，让Cursor成为你提效路上的快刀！
---
## 原文翻译和解析

下面是原文摘自 [Ryo Lu](https://x.com/ryolu_/status/1914384195138511142)帖子的12个使用Cursor的规范，好的使用习惯将会是你提效路上的披荆斩棘的一把快刀。

作者认为，使用 Cursor 得当我们能够快速得到干净的代码，使用不当 = 需要你清理一整周的流水账一样的、面条式的混乱代码。下面是Lu整理的如何真正正确地使用Cursor的十二个规则。



---

Set 5-10 clear project rules upfront so Cursor knows your structure and constraints. Try /generate rules for existing codebases.预先设定 5-10 条清晰的项目规则，这样 Cursor 就能了解你的结构和约束。对于现有代码库，可以尝试使用 `/generate rules` 命令。

> 实践中，我会在项目创建开始或者开始修改现存代码时指定几个关键业务接口，让他Top-Down的往下梳理和阅读代码。整理出不理解的点，帮助AI生成一个cursor rule

---



Be specific in prompts. Spell out tech stack, behavior, and constraints like a mini spec. 在提示词中要具体。详细说明技术栈、行为和约束，就像写一份微型规格说明书。

> 我自己的习惯是生成一个1个tech-stack.mdc+N个biz_flow.mdc，用1个技术栈说明在每次编辑.java文件的时候自动attach上去。另外biz_flow.mdc在需要的时候手动 @提供给LLM



---

Work file by file; generate, test, and review in small, focused chunks. 逐个文件地工作；以小而专注的块为单位生成、测试和审查代码。



---

Write tests first, lock them, and generate code until all tests pass. 先写测试用例，锁定它们，然后生成代码直到所有测试通过。

> Tips：练习Leetcode的时候可以先让他在main方法里面使用Assertion断言N个TC，然后在迭代编写算法的时候不断的尝试执行他，效率Up





---

Always review AI output and hard‑fix anything that breaks, then tell Cursor to use them as examples. **务必审查 AI 的输出**，手动修复任何出错的地方，然后告诉 Cursor 将这些修复作为示例使用。

**PS：感觉这一步大部分人包括我自己有时候也会直觉性偷懒不去认真review代码**



---

Use @ file, @ folders, @ git to scope Cursor's attention to the right parts of your codebase. 使用 `@文件`、`@文件夹`、`@git` 来限定 Cursor 的注意力范围，使其关注代码库的正确部分。



---

Keep design docs and checklists in .cursor/ so the agent has full context on what to do next. 将设计文档和检查清单保存在 `.cursor/` 目录下，这样智能体就能完全了解下一步该做什么。



---

If code is wrong, just write it yourself. Cursor learns faster from edits than explanations. 如果代码是错的，就自己动手写。Cursor 从编辑中学到的比从解释中学到的更快。

**意思是自己修改写出来的代码能让AI更好的理解，相较于你告诉他"我们之前错在了XYZ地方"要更靠谱**。



---

Use chat history to iterate on old prompts without starting over. 利用聊天记录来迭代旧的提示词，无需从头开始。



---

Choose models intentionally. Gemini for precision, Claude for breadth. 有意识地选择模型。追求精确用 Gemini，追求广度用 Claude。

**到目前位置大模型质检的差距会小一些，包括对auto模式的选用，其实差距不会很大了。但是从各种社区的推荐的最佳实践来看，使用O3去做PRD等设计类活动，再让Claude4 sonnet去执行代码变更计划这种协作模式的非常多**



---

In new or unfamiliar stacks, paste in link to documentation. Make Cursor explain all errors and fixes line by line. 对于新的或不熟悉的技术栈，粘贴文档链接。让 Cursor 逐行解释所有错误及其修复方法。

**这里推荐Context7 MCP Server**



---

Let big projects index overnight and limit context scope to keep performance snappy. 让大型项目整夜建立索引，并限制上下文范围以保持性能迅捷。





---



Structure and control wins (for now)
结构和控制力才是制胜之道（目前如此）

Treat Cursor agent like a **powerful junior** — it can go far, fast, if you show it the way. （* **其实这句话我觉得还蛮醍醐灌顶的，对于Cursor来说，可能Cursor一定程度上会从powerful junior到more powerful junior，但是很难实现端到端的去将需求转换为成品的代码甚至于直接部署在生产环境**）。
把 Cursor 智能体当作一个能力很强的新人——如果你为其指明道路，它能走得很远、很快。

## 总结
**总结：**
1. 预设清晰规则。
2. 明确提示。
3. 分块迭代、测试先行、严格审查与纠正。
4. 善用上下文限定。
5. 提供完整设计文档。
6. 以编辑促学习、利用历史迭代。
7. 合理选择模型。
8. 及时补充文档。
9. 优化索引管理。