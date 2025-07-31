---
title: Hexo文章管理最佳实践
categories:
  - 博客
description: Hexo博客文章管理方法
date: 2025-05-12 10:50:00
tags:
  - Hexo
  - 博客管理
---

# Hexo 文章管理方法

## 1. 分类目录结构

推荐将文章按照类别组织到子文件夹中：

```plain
source/_posts/
  ├── frontend/           # 前端相关文章
  │   ├── react/         # React相关
  │   └── vue/          # Vue相关
  ├── backend/           # 后端相关文章
  │   ├── nodejs/       # Node.js相关
  │   └── python/       # Python相关
  └── others/           # 其他文章
```

## 2. Front-matter 管理

为每篇文章添加完整的 front-matter 信息：

```yaml
---
title: 文章标题
date: 2025-05-12 10:50:00
updated: 2025-05-12 10:50:00
categories: 
  - 分类1
  - 分类2
tags:
  - 标签1
  - 标签2
permalink: custom-url    # 自定义链接
description: 文章描述
---
```

## 3. 使用 Asset 文件夹

对于包含图片等资源的文章，开启文章资源文件夹功能：

1. 在 `_config.yml` 中设置：
```yaml
post_asset_folder: true
```

2. 创建文章时会自动生成同名文件夹存放资源：
```plain
source/_posts/
  ├── 文章.md
  └── 文章/          # 资源文件夹
      └── image.png
```

## 4. 使用标签插件

使用 Hexo 标签插件组织内容：

```markdown
{% post_link 文章文件名 %}   # 文章链接
{% asset_img 图片名 %}      # 引用图片
{% raw %}                  # 原始内容
{% endraw %}
```

## 5. 使用脚本管理

创建自定义脚本来批量管理文章：

```bash
hexo new draft "新文章"     # 创建草稿
hexo publish 文章名         # 发布草稿
hexo new page "页面名"      # 创建页面
```

## 6. Git 版本控制

将博客目录使用 Git 管理：

1. 追踪文章变更历史
2. 多设备同步
3. 作为备份方案

## 7. 元数据索引

创建文章索引文件，记录所有文章的元数据，便于管理和查找：

```yaml
posts:
  - title: 文章1
    date: 2025-05-12
    category: 前端
    status: published
  - title: 文章2
    date: 2025-05-13
    category: 后端
    status: draft
```