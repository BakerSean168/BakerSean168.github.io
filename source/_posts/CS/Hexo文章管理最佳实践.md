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


# tags 和 categories

- 定义层级  
  - categories（分类）：层级/主题化的分组，通常是树状或少量顶级项，用来表示文章的主要归属（例如「前端」「后端」「工具」）。  
  - tags（标签）：扁平的关键词集合，用来描述文章的细节属性或多个维度（例如「Vite」「性能优化」「插件」）。

- 用途差异  
  - categories：帮助构建站点的主导航和目录，用户通过分类快速定位某一类文章。  
  - tags：便于多维度检索、交叉过滤和相关文章推荐（标签云、相关文章列表）。

- 数量与管理  
  - categories：数量少、稳定、层级化；策划好后不常改动。  
  - tags：数量多、可自由添加，便于细化内容，但应避免过度碎片化（同义词合并、规范化）。

- SEO 与 URL 影响  
  - categories 常用于生成页面路径（/category/xxx），对站点结构和目录性有帮助。  
  - tags 更多用于站内检索，单个标签页也可被索引，但对站点主题定位作用不如分类明显。

- 展示与交互  
  - categories：通常在顶部/侧边栏作为主菜单或目录树展示。  
  - tags：常在文章底部、标签云或侧边栏作为快速筛选入口。

- 实践建议（规则）  
  1. 每篇文章设置 1 个主 category（或最多 1-2 个顶层分类）。  
  2. 使用 3–8 个有意义的 tags，覆盖技术点、工具、场景等。  
  3. 规范标签命名（小写/统一词形），合并相近标签。  
  4. 定期维护分类与标签，避免无效或重复项膨胀。  

- 示例 front-matter（Hexo/Jekyll 等）  
```markdown
---
title: Vite 性能优化实战
categories:
  - 前端
  - 性能
tags:
  - vite
  - tree-shaking
  - 代码分割
date: 2025-08-18
---
```

简短总结：categories 用于大类、站点结构和导航；tags 用于细粒度描述、检索和关联内容。