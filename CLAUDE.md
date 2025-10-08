# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 项目概述

这是一个使用 Icarus 主题的 Hexo 静态网站生成器博客。Hexo 是一个由 Node.js 驱动的快速、简单且强大的博客框架，可将 Markdown 内容转换为静态 HTML 网站。

## 常用命令

### 开发
- `hexo server` - 启动本地开发服务器（默认端口：4000）
- `hexo server -p <port>` - 在指定端口启动服务器（当 4000 端口被占用时很有用）
- `hexo generate` 或 `hexo g` - 生成静态文件
- `hexo clean` - 清理生成的文件和缓存

### 内容管理
- `hexo new <title>` - 创建新博客文章
- `hexo new page <name>` - 创建新页面
- `hexo new draft <title>` - 创建草稿文章

### 部署
- `hexo deploy` - 部署生成的文件（需要在 _config.yml 中配置部署设置）

## 配置架构

### 主要配置
- `_config.yml` - Hexo 主配置（站点元数据、主题选择、部署）
- `_config.icarus.yml` - Icarus 主题特定配置

### 关键配置区域
- **主题选择**：在 `_config.yml` 中设置 `theme: icarus`
- **站点元数据**：标题、作者、描述在 `_config.yml` 中
- **主题定制**：所有视觉设置在 `_config.icarus.yml` 中
- **静态资源**：图片和媒体在 `source/` 目录中

### 目录结构
- `source/` - 用户内容目录（生成后镜像到站点根目录）
  - `_posts/` - Markdown 格式的博客文章
  - `img/` - 图片资源
  - `css/` - 自定义 CSS
- `themes/` - 主题文件（当前为空，主题在 node_modules 中）
- `public/` - 生成的静态站点（由 `hexo generate` 创建）
- `node_modules/` - 依赖项，包括已安装的主题

## Icarus 主题配置

Icarus 主题配置（`_config.icarus.yml`）包括：
- **品牌标识**：logo、favicon 路径在 `source/img/` 中
- **导航**：菜单项和链接在 `navbar` 部分
- **侧边栏小部件**：个人资料、目录、分类、归档、标签在 `widgets` 部分
- **插件**：搜索、评论、分析在各自部分
- **样式**：配色方案、字体、CDN 设置在 `providers` 部分

## 主题变体和定制

Icarus 支持变体（默认、赛博朋克）和广泛的小部件定制。主题使用模块化小部件系统，小部件可以放置在左右侧边栏，具有各种类型（个人资料、目录、分类等）。

## Hexo Front-matter 体系

Front-matter 是文章开头的 YAML 或 JSON 代码块，用于配置文章设置。

### 基本格式

**YAML 格式**：
```yaml
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

**JSON 格式**：
```json
"title": "Hello World",
"date": "2013/7/13 20:46:25"
;;;
```

### 常用设置项

| 设置 | 描述 | 默认值 |
|------|------|--------|
| `layout` | 布局类型 | `config.default_layout` |
| `title` | 文章标题 | 文章的文件名 |
| `date` | 建立日期 | 文件建立日期 |
| `updated` | 更新日期 | 文件更新日期 |
| `comments` | 开启文章评论功能 | `true` |
| `tags` | 标签（不适用于分页） | - |
| `categories` | 分类（不适用于分页） | - |
| `permalink` | 覆盖文章永久链接 | `null` |
| `excerpt` | 页面摘要 | - |
| `lang` | 设置语言 | 继承自 `_config.yml` |
| `published` | 文章是否发布 | `_posts` 下为 `true`，`_draft` 下为 `false` |

### 分类和标签

- **分类**：按顺序应用，形成层次结构
- **标签**：同一层次，顺序不重要

**示例**：
```yaml
categories:
  - Sports
  - Baseball
tags:
  - Injury
  - Fight
  - Shocking
```

**多分类层次结构**：
```yaml
categories:
  - [Sports, Baseball]
  - [MLB, American League, Boston Red Sox]
  - [MLB, American League, New York Yankees]
  - Rivalries
```

## 本项目 Front-matter 规范

基于项目现有文章分析，制定了以下 Front-matter 属性规范：

### 核心属性（必须包含）

| 属性 | 描述 | 示例格式 |
|------|------|---------|
| `title` | 文章标题 | `title: AQS总成为你大厂面试的"滑铁卢"？别让它成为送命题了` |
| `date` | 发布日期 | `date: 2025-06-22 19:57:57` |
| `author` | 作者 | `author: 虎太郎` |
| `toc` | 是否显示目录 | `toc: true` |
| `categories` | 文章分类（支持层级） | 见下方分类示例 |
| `tags` | 文章标签 | 见下方标签示例 |

### 常用属性（推荐包含）

| 属性 | 描述 | 使用建议 |
|------|------|---------|
| `updated` | 更新日期 | 当文章有修改时添加 |
| `excerpt` | 文章摘要 | 用于SEO和文章列表展示 |
| `cover` | 封面图片 | 使用完整的URL链接 |
| `description` | 文章描述 | 更详细的文章说明 |

### 偶见属性（按需使用）

| 属性 | 描述 |
|------|------|
| `copy_from` | 转载来源 |

### 属性示例

**标准格式**：
```yaml
---
title: AQS总成为你大厂面试的"滑铁卢"？别让它成为送命题了
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
excerpt: 文章摘要内容
cover: https://example.com/image.jpg
---
```

**分类层级示例**：
- 单级分类：`categories: - AI`
- 多级分类：`categories: - Java - 分布式中间件`
- 复杂分类：`categories: - [Java, 技术] - [分布式, 中间件]`

**标签设置**：
```yaml
tags:
  - AI
  - Prompt Engineering
  - Cursor
```

## 开发注意事项

- `source/` 中的静态资源在站点根目录提供服务（例如：`source/img/logo.svg` → `/img/logo.svg`）
- 主题文件位于 `node_modules/hexo-theme-icarus/` 中，但不应直接修改
- 自定义 CSS 应放在 `source/css/` 中以覆盖主题样式
- 文章中引用的图片应使用相对路径或放在 `source/img/` 中
- 文章文件应放在 `source/_posts/` 目录下，支持 Markdown 格式
- **重要**：创建新文章时请遵循上述 Front-matter 规范，确保文章元数据的一致性和完整性