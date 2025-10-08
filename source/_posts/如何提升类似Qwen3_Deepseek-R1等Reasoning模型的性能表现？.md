---
title: 「AI观察」旧瓶如何装新酒，如何提升Qwen3/Deepseek-R1等Reasoning模型在老Prompt上的性能表现？
date: 2025-06-18 16:35:00
tags:
  - AI
  - Prompt Engineering
  - Qwen3
  - Deepseek-R1
categories:
  - AI
author: 虎太郎
toc: true
---

传统的非Reasoning模型的Prompt或许在Reasoning模型Qwen3/Deepseek-R1上表现不佳，比如，你需要如何修改呢？

<!-- more --> 

- TRICK：prompt里面加入COT的trick，要求他think step by step，通过这种方式拉长模型的时间。因为思考的时间和内容越多，越能在正确的方向上激活模型更多的相关参数，对于答案质量的帮助是**绝对地清晰可见**。
- BUDGET：拉长思考时间在Qwen3上面可以增加thinking_budget。在Claude的模型产品上应该也有类似的设计。根据[Qwen团队的官方技术报告](https://github.com/QwenLM/Qwen3/blob/main/Qwen3_Technical_Report.pdf)，大部分场景在8k、32k token的时候评测分数会明显到达一个峰值。建议Qwen3设置在4-8K这个区间，最好是8K，再往后会有个平缓的边际效益发生（但是再往后又会有一个小突破）

![image-20250618230647141](https://hexo-yelihu.oss-cn-hangzhou.aliyuncs.com/hexo/img/image-20250618230647141.png)

总结：维护好prompt很重要，注意区分Reasoning和非Reasoning模型在同一份Prompt上的表现差异，可以使用类似promptpilot的产品去评测和维护。