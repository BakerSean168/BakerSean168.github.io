---
title: git相关库使用
categories:
  - nodejs
description: git相关库使用
date: 2025-01-10 11:02:02
tags:
---

# git 功能

child_rocess
git-essentials

## simple-git

### git.status

git.status() 返回的数据，来自 copilot
```ts
// Current branch information
  current: string             // Name of current branch (e.g., "main", "master")
  tracking: string           // Name of upstream branch being tracked (e.g., "origin/main")
  
  // Ahead/Behind information
  ahead: number             // Number of commits ahead of remote
  behind: number            // Number of commits behind remote
  
  // File status arrays
  staged: string[]          // Files staged for commit
  not_added: string[]       // Untracked files (files not yet added to git)
  created: string[]         // New files added to git but not committed
  modified: string[]        // Files that have been modified but not staged
  deleted: string[]         // Files that have been deleted but not staged
  renamed: string[]         // Files that have been renamed
  conflicted: string[]      // Files with merge conflicts
  
  // Detailed file status
  files: Array<{
    path: string           // File path
    index: string         // Status in index (staging area)
    working_dir: string   // Status in working directory
  }>
  
  // Helper methods
  isClean(): boolean      // Returns true if working directory is clean
  
  // Raw status data
  detached: boolean       // True if in detached HEAD state
  hash: string           // Current commit hash

files.index
' ' - Unmodified
'M' - Modified
'A' - Added
'D' - Deleted
'R' - Renamed
'C' - Copied
'U' - Updated but unmerged
'?' - Untracked
'!' - Ignored
```

### git.log

把 git.log 里的参数都去除了，就能正常返回 log 记录  

```ts
// 使用示例
async getLog(): Promise<GitResponse> {
  if (!this.git) {
    return { success: false, error: 'Git not initialized' }
  }

  try {
    const logResult = await this.git.log([
      '--pretty=format:%H%n%an%n%ad%n%s%n%P',  // 添加父提交哈希
      '--date=iso',
      '--all',                                  // 显示所有分支
      '--graph',                                // 显示ASCII图形
      '-n',
      '50'
    ])

    const commits = logResult.all.map(commit => {
      const [hash, author_name, date, message, parents] = commit.hash.split('\n')
      return {
        hash,
        author_name,
        date,
        message,
        parents: parents ? parents.split(' ') : []  // 解析父提交
      }
    })

    return {
      success: true,
      data: { commits }
    }
  } catch (error) {
    console.error('Git log error:', error)
    return { 
      success: false, 
      error: 'Failed to get commit history' 
    }
  }
}
```
```
参数说明：  

--pretty=format:"%H|%an|%ad|%s"
%H: 完整的提交哈希值 (40个字符)
%an: 作者名字 (author name)
%ad: 作者提交日期 (author date)
%s: 提交信息的第一行 (subject)
|: 自定义分隔符，用于后续解析字符串

--date=iso
将日期格式化为 ISO 8601 格式
例如：2025-02-21 13:09:21 +0800
这种格式便于解析和处理

-n 50
限制返回的提交数量为最近的50条
这是为了性能考虑，避免在大型仓库中获取太多历史记录

其他：  
%h  - 简短的提交哈希值 (7个字符)
%cn - 提交者名字
%cd - 提交日期
%cr - 相对时间 (例如：2 days ago)
%d  - ref names (分支、标签等)
%b  - 提交信息主体
%e  - 编码
%P  - 父提交的哈希值
```

logResult 结果
```ts
Commits: {
  all: [
    {
      hash: '474c48c41373fb5252296380c34fb78af74a5f6f\n' +
        'BakerSean\n' +
        '2025-02-21 18:21:05 +0800\n' +
        '123\n' +
        '89c9cf7b15b18b3b9f9852927c356902fed07e96\n' +
        'BakerSean\n' +
        '2025-02-21 13:09:21 +0800\n' +
        '123',
      date: '',
      message: '',
      refs: '',
      body: '',
      author_name: '',
      author_email: ''
    }
  ],
  latest: {
    hash: '474c48c41373fb5252296380c34fb78af74a5f6f\n' +
      'BakerSean\n' +
      '2025-02-21 18:21:05 +0800\n' +
      '123\n' +
      '89c9cf7b15b18b3b9f9852927c356902fed07e96\n' +
      'BakerSean\n' +
      '2025-02-21 13:09:21 +0800\n' +
      '123',
    date: '',
    message: '',
    refs: '',
    body: '',
    author_name: '',
    author_email: ''
  },
  total: 1
}

Commits: {
  all: [
    {
      hash: '474c48c41373fb5252296380c34fb78af74a5f6f',
      date: '2025-02-21T18:21:05+08:00',
      message: '123',
      refs: 'HEAD -> master',
      body: '',
      author_name: 'BakerSean',
      author_email: 'bakersean@foxmail.com'
    },
    {
      hash: '89c9cf7b15b18b3b9f9852927c356902fed07e96',
      date: '2025-02-21T13:09:21+08:00',
      message: '123',
      refs: '',
      body: '',
      author_name: 'BakerSean',
      author_email: 'bakersean@foxmail.com'
    }
  ],
  latest: {
    hash: '474c48c41373fb5252296380c34fb78af74a5f6f',
    date: '2025-02-21T18:21:05+08:00',
    message: '123',
    refs: 'HEAD -> master',
    body: '',
    author_name: 'BakerSean',
    author_email: 'bakersean@foxmail.com'
  },
  total: 2
}
```
