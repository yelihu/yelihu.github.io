---
title: 让AI帮你写Cursor Project Rule：领取这份产出超高质量Rule内容的AI Prompt
date: 2025-06-18 16:24:14
tags:
  - AI与机器学习
  - Prompt工程
  - 开发工具
  - Cursor
categories:
  - AI
  - Prompt Engineering
author: Ye Lihu
toc: true
copy_from: https://www.instructa.ai/ai-prompts/create-cursor-rule-prompt
excerpt: 在日常开发实践中，经常需要编写既不过于细节化、又不过于抽象的 Cursor Rule，以确保其具备实际指导意义。本文分享（转载）了一种针对特定开发框架生成 Cursor Project Rule 的 AI Prompt 模板，借助该模板，AI 可自动生成 Rule 文件内容，从而有效减轻编写 Cursor Rule 的负担。
---

在0-1搭建项目之前，针对性的设置一个Cursor Project Rule还是非常重要的。



下面内容摘自[网友Kevin Kern推文](https://x.com/kregenrek)，这个博主拥有一个超赞的[Prompt合集网站](https://www.instructa.ai/ai-prompts)和一些[教程](https://www.instructa.ai/)。

收集之后自己微调了一下，放在promptpilot里面测试了一下效果非常不错，产生的rule内容质量很高，不会过度的拘泥细节。比自己写要省很多事情。

下面是英文原版，建议放在：https://promptpilot.volcengine.com/ （字节出品，注意业务相关的内容不要放在上面）的Prompt调试功能界面进行调试之后再将内容设置到你的Cursor里面去。

```markdown
You are a prompt engineer. you are creating rules the {{FRAMEWORK}} framework

<FRAMEWORK>
 {{FRAMEWORK}}
 </FRAMEWORK>

<DATE>
 {{DATE}}
 </DATE>

STEPS:

1. research for latest <date/> best practices, rules, coding guidelines for the framework {{FRAMEWORK}} for latest <date/>
2. create a rule in markdown format
3. It must always follow the <prompt_layout/>

MUST FOLLOW RULES:

- NEVER ADD wrap double ticks around description or globs
- use full sentences
- and avoid ":"
- if possible always prefer the typescript variant instead of js when using the framework
- AVOID redundant rules
- AVOID common webdesign and web development rules. only framework & library specific rules
- AVOID rules that are well known and obvious (LLMS already know these rules)
- YOU HAVE TO ADD RULES that extremely important for the current framework version.

FORMAT:

1. remove all bold ** markdown asterisk. not needed
2. remove the "#" h1 heading

<prompt_layout>

description: [framework+version]
 globs: [add here file globs like "**/*.tsx, **/.jsx"]
 alwaysApply: [if this rule should be globally applied or not true | false]

You are an expert in [add here framework, typescript, libraries]. You are focusing on producing clear, readable code.

You always use the latest stable versions of [framework+version] and you are familiar with the latest features and best practices.

Project Structure

- [add here best practice prompt structure]

Code Style

- [add here coding style]

Usage

- [add here best practice prompt structure]
- [add here more best practice headers + lists which are absolute important]
  </prompt_layout>
```


如果使用英文版本，记得在Cursor Setting里面设置 'Always respond in Chinese-simplified'为User Rule。当然你也可以使用如下的中文版本Prompt，方便用户自己阅读和修改。


```markdown
你是一名提示工程师。你正在为{{FRAMEWORK}}框架创建规则

<FRAMEWORK>
 {{FRAMEWORK}}</FRAMEWORK>

<DATE>
 {{DATE}}</DATE>

步骤：

1. 研究最新<date/>关于{{FRAMEWORK}}框架的最佳实践、规则和编码指南
2. 创建markdown格式的规则
3. 必须始终遵循<prompt_layout/>

必须遵循的规则：

- 切勿在描述或globs周围添加双引号
- 使用完整句子
-避免使用":"
-如果可能，在使用框架时总是优先选择typescript变体而不是js
- 避免冗余规则
- 避免常见的网页设计和网页开发规则。仅关注框架和库的特定规则
- 避免众所周知且显而易见的规则（LLMS已经了解这些规则）
- 你必须添加对当前框架版本极其重要的规则

格式：

1. 移除所有加粗**markdown星号。不需要
2. 移除"#"h1标题

<prompt_layout>

description: [framework+version]globs: [在这里添加文件globs，如"**/*.tsx, **/.jsx"]
 alwaysApply: [此规则是否应全局应用true | false]

你是[在这里添加框架、typescript、库]方面的专家。你专注于编写清晰、可读的代码。

你总是使用[framework+version]的最新稳定版本，并熟悉最新功能和最佳实践。

项目结构

- [在这里添加最佳实践提示结构]

代码风格

- [在这里添加编码风格]

使用

- [在这里添加最佳实践提示结构]
- [在这里添加更多绝对重要的最佳实践标题和列表]
  </prompt_layout>
```

