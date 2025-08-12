---
title: 尝试使用git来团队式地修改代码
categories:
  - Git
description: 尝试使用git来团队式地修改代码
date: 2025-06-21 22:02:58
tags:
---

想要重构一下 task 模块地代码，正好来尝试一下团队式地修改代码，让代码更安全

## 常用命令

```bash

# 查看所有分支
git branch -a

# 创建并切换到新分支
git checkout -b [branch-name]
```


## 1. 切换到不同分支

### 初始设置

不同的新代码的有不同约定名称：  

```bash
# 使用约定式提交格式
git commit -m "feat: add comprehensive constructor options"
git commit -m "fix: resolve parameter order issue in constructor"
git commit -m "refactor: consolidate updatedAt property usage"
git commit -m "docs: update TaskTemplate usage examples"
```

创建一个重构的分支：  
`git checkout -b refactor/task-module-structure`

此时就切换到新的分支了

## 2. **分支管理策略**

### Git Flow 模型
```bash
# 主要分支
main/master    # 生产环境代码
develop        # 开发环境代码

# 功能分支
feature/       # 新功能开发
hotfix/        # 紧急修复
release/       # 发布准备
```

### GitHub Flow (推荐简单项目)
```bash
# 只有 main 分支和功能分支
main           # 主分支
feature/*      # 功能分支
```

## 3. 提交部分代码

每重构完一部分就可以在当前分支提交代码：  
```bash
git add .
git commit -m "refactor: reorganize task module structure following DDD"
```

## 4. 合并分支

最重要的一步，将新代码合并到主代码中

### merge

1. 切回主分支
2. 拉去最新代码
3. 用 merge 合并分支

### rebase

1. 切回主分支
2. 拉去最新代码
3. 切回功能分支
4. 将功能分支变基到主分支
5. 解决冲突
6. 切回主分支
7. 用 merge 合并

### 相关命令
```bash
# 切换回主分支
git checkout main

# 合并重构分支
git merge refactor/task-module-ddd

# 或者使用rebase保持历史整洁
git rebase refactor/task-module-ddd

# 当出现合并冲突时
git status                    # 查看冲突文件
# 手动编辑冲突文件
git add .                     # 添加解决后的文件
git commit -m "resolve merge conflicts"
```


## 5. **Pull Request 工作流**

### 创建 PR 前的准备
```bash
# 确保代码是最新的
git checkout main
git pull origin main
git checkout feature/your-branch
git rebase main

# 运行测试
npm test                      # 或相应的测试命令

# 推送到远程
git push origin feature/your-branch
```

### PR 审查过程
1. **创建 PR** - 在 GitHub/GitLab 平台创建
2. **代码审查** - 团队成员审查代码
3. **修改反馈** - 根据反馈修改代码
4. **合并** - 审查通过后合并到主分支

## 6. **实用 Git 命令**

### 查看状态和历史
```bash
git status                    # 查看当前状态
git log --oneline            # 查看提交历史
git log --graph --oneline    # 图形化查看分支历史
git diff                     # 查看修改差异
```

### 撤销和重置
```bash
git checkout -- <file>       # 撤销文件修改
git reset HEAD <file>        # 取消暂存
git reset --soft HEAD~1      # 撤销最后一次提交(保留修改)
git reset --hard HEAD~1      # 撤销最后一次提交(删除修改)
```

### 远程仓库管理
```bash
git remote -v                # 查看远程仓库
git fetch origin            # 获取远程更新
git push -u origin <branch>  # 首次推送分支
```

## 7. **团队规范建议**

### 分支命名规范
```bash
feature/task-template-refactor    # 功能开发
bugfix/constructor-parameter-fix  # 错误修复
hotfix/critical-security-patch   # 紧急修复
release/v1.2.0                   # 发布分支
```

### 提交信息模板
```bash
<type>(<scope>): <subject>

<body>

<footer>

# 例如:
feat(TaskTemplate): add comprehensive constructor options

- Add support for all metadata fields in constructor
- Improve type safety with better parameter organization
- Maintain backward compatibility

Closes #123
```

## 个人理解

网上的信息中 rebase 的目的如下：  
是为了让 git 历史变为线性，会将分支转移到主分支最新的节点处（通过重新应用提交来模拟“直接基于最新代码开发”的效果），然后再合并分支

rebase，变基  

但是通过 `git rebase -i`，还可以合并（squash）、编辑（edit）或删除提交，进一步定制历史。  
让 git 历史线性，只是重新提交历史到最末端，也是对历史记录的一种改变，是 rebase 功能的一部分。

所以我认为 rebase 本质是对历史记录的修改  

所以避免让他人的历史冲突，应该避免对已经推送到远程，被他人使用的历史变基。一般用来整理自己本地的历史，让他更简洁。  

## 一些问题

### git 是怎么合并代码的

合并策略：ORT（三路合并算法）  
原理：通过比较两个分支的提交（A 和 B）及其最近共同祖先（Base），生成合并结果。若 A 和 B 对同一行代码的修改均不同于 Base，则触发冲突。  

### git 合并中会遇到的冲突有哪些  


1. **内容冲突（Edit-Edit Conflict）**  
   - **场景**：两个分支修改了同一文件的同一行代码，Git 无法自动选择保留哪个版本。  
   - **示例**：  
     ```text
     <<<<<<< HEAD  
     Hello, this is the **updated project note** by A.  
     =======  
     Hello, this is the **feature note** by B.  
     >>>>>>> feature
     ```  
     需要手动选择或整合两者修改。

2. **删除-修改冲突（Delete-Modify Conflict）**  
   - **场景**：一个分支删除了文件，另一个分支修改了同一文件。Git 无法决定保留修改还是删除文件。

3. **重命名-修改冲突（Rename-Modify Conflict）**  
   - **场景**：一个分支重命名了文件，另一个分支修改了原文件名下的内容。Git 无法自动关联重命名后的文件与修改。

4. **目录-文件冲突（Directory-File Conflict）**  
   - **场景**：一个分支将文件改为目录，另一分支仍将其作为文件修改。Git 无法处理这种结构变化。

5. **子模块冲突（Submodule Conflict）**  
   - **场景**：两个分支引用了同一子模块的不同提交，Git 无法自动选择版本。

6. **二进制文件冲突（Binary File Conflict）**  
   - **场景**：二进制文件（如图片、PDF）被两个分支修改，Git 无法按行比较差异，需手动选择保留哪个版本。

7. **空格冲突（Whitespace Conflict）**  
   - **场景**：因空格、换行符等格式修改触发的冲突，可通过 Git 选项（如 `ignore-space-change`）自动忽略。
