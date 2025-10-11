# Front-matter 详细说明指南

## 什么是 Front-matter

Front-matter 是文章开头的 YAML 或 JSON 代码块，用于配置文章的元数据和显示设置。

## 基本格式

### YAML 格式（推荐）
```yaml
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

### JSON 格式
```json
"title": "Hello World",
"date": "2013/7/13 20:46:25"
;;;
```

## 完整属性说明

### 核心属性（必须包含）

| 属性 | 类型 | 描述 | 示例 |
|------|------|------|------|
| `title` | String | 文章标题 | `title: AQS总成为你大厂面试的"滑铁卢"？` |
| `date` | DateTime | 发布日期 | `date: 2025-06-22 19:57:57` |
| `author` | String | 作者 | `author: 虎太郎` |
| `toc` | Boolean | 是否显示目录 | `toc: true` |
| `categories` | Array | 文章分类 | 见下方分类示例 |
| `tags` | Array | 文章标签 | 见下方标签示例 |
| `excerpt` | String | 文章摘要 | 用于SEO和文章列表展示 |

### 常用属性（推荐包含）

| 属性 | 类型 | 描述 | 使用建议 |
|------|------|------|---------|
| `updated` | DateTime | 更新日期 | 当文章有修改时添加 |
| `cover` | String | 封面图片 | 使用完整的URL链接 |
| `layout` | String | 布局类型 | 通常使用默认值 |
| `comments` | Boolean | 开启评论功能 | 默认为 `true` |

### 偶见属性（按需使用）

| 属性 | 类型 | 描述 |
|------|------|------|
| `copy_from` | String | 转载来源 |
| `permalink` | String | 自定义永久链接 |
| `lang` | String | 文章语言 |
| `published` | Boolean | 是否发布 |

## 分类和标签详解

### 分类（Categories）

**特点：**
- 按顺序应用，形成层次结构
- 支持多级分类
- 用于文章的主要归类

**单级分类：**
```yaml
categories:
  - AI
```

**多级分类：**
```yaml
categories:
  - Java
  - 分布式中间件
```

**复杂分类（同时属于多个分类）：**
```yaml
categories:
  - [Java, 技术]
  - [分布式, 中间件]
```

**分类层次示例：**
```yaml
categories:
  - [Sports, Baseball]
  - [MLB, American League, Boston Red Sox]
  - [MLB, American League, New York Yankees]
  - Rivalries
```

### 标签（Tags）

**特点：**
- 同一层次，顺序不重要
- 用于文章的关键词标记
- 支持多个标签

**基础标签设置：**
```yaml
tags:
  - AI
  - Prompt Engineering
  - Cursor
```

**技术文章标签示例：**
```yaml
tags:
  - Java
  - JUC
  - 并发
  - 多线程
  - AQS
  - ReentrantLock
```

## 本项目标准模板

### 技术文章模板
```yaml
---
title: 文章标题
date: 2025-06-22 19:57:57
updated: 2025-06-22 19:57:57
author: 虎太郎
toc: true
categories:
  - Java
  - 分布式中间件
tags:
  - AQS
  - Java
  - JUC
  - 并发
  - 多线程
  - ReentrantLock
description: 文章的详细描述，用于SEO优化
excerpt: 文章摘要内容，显示在文章列表中
cover: https://example.com/cover-image.jpg
---
```

### AI相关文章模板
```yaml
---
title: 文章标题
date: 2025-06-18 20:48:20
author: 虎太郎
toc: true
categories:
  - AI
  - Prompt Engineering
tags:
  - AI与机器学习
  - Prompt工程
  - 研发提效
  - Cursor
description: 详细描述文章内容和价值
excerpt: 简短摘要，吸引用户点击阅读
---
```

### 工具使用文章模板
```yaml
---
title: 文章标题
date: 2025-07-21 10:30:00
author: 虎太郎
toc: true
categories:
  - 技术
  - 开发工具
tags:
  - 故障排查
  - Context7
  - MCP
  - 服务器连接
  - 问题解决
description: 问题解决过程和经验总结
excerpt: 记录一次Context7 MCP服务器连接失败的解决过程
---
```

## 日期格式说明

### 推荐日期格式
```yaml
date: 2025-06-22 19:57:57    # 年-月-日 时:分:秒
```

### 更新日期设置
```yaml
updated: 2025-06-22 20:30:00  # 手动设置更新时间
# 或者留空，Hexo 会自动使用文件修改时间
```

## SEO 优化建议

### 重要 SEO 属性
```yaml
title: 包含关键词的吸引人标题     # 搜索引擎重要
description: 150字内的详细描述    # 搜索结果显示
excerpt: 100字内的精彩摘要        # 社交媒体分享
tags:                           # 相关标签提升搜索
  - 关键词1
  - 关键词2
  - 长尾关键词
```

### 封面图片优化
```yaml
cover: https://example.com/image.jpg  # 建议使用高质量图片
```

## 特殊用法

### 草稿文章
```yaml
---
title: 草稿文章
published: false
---
```
（放在 `_posts/` 目录但不会发布）

### 自定义永久链接
```yaml
---
title: 文章标题
permalink: custom-url.html
---
```

### 禁用评论
```yaml
---
title: 文章标题
comments: false
---
```

## 最佳实践

1. **保持一致性**：遵循项目统一的 Front-matter 格式
2. **合理分类**：使用有意义的分类，避免分类过深
3. **精准标签**：标签要准确反映文章内容
4. **优化SEO**：填写好 title、description、excerpt
5. **及时更新**：修改文章时更新 updated 字段

## 常见问题

### Q: 分类和标签的区别？
A: 分类是层级结构，用于文章归类；标签是扁平结构，用于关键词标记。

### Q: 日期格式有什么要求？
A: 支持 `YYYY-MM-DD HH:mm:ss` 或 `YYYY/MM/DD HH:mm:ss` 格式。

### Q: 如何设置多级分类？
A: 使用数组形式，如 `categories: - Java - 分布式中间件`

### Q: 封面图片如何设置？
A: 使用完整的 URL 链接，建议使用高质量图片。