---
title: mcp-filesystem
categories:
  - null
description: mcp-filesystem 的配置使用，记录一下；基于 vscode 插件
date: 2025-09-15 13:40:14
tags:
  - MCP
  - AI
---


## 安装

Copilot MCP 插件，用于搜索、安装 MCP 服务

@modelcontextprotocol/server-filesystem

这个包是 mcp 官方的包。

## 配置

在 插件 -> MCP Servers-installed 中右击 服务，显示配置（JSON），就能修改相对应的配置

```JSON
{
	"servers": {
		"github": {
			"url": "https://api.githubcopilot.com/mcp/",
			"type": "http"
		},
		"@agent-infra/mcp-server-filesystem": {
			"command": "npx",
			"args": [
				"@agent-infra/mcp-server-filesystem@latest",
				"--allowed-directories",
				"D:\\myPrograms\\DailyUse" // 只能传一个文件路径，好像得用数组传，暂时不清楚怎么传
			],
			"env": {},
			"type": "stdio"
		}
	},
	"inputs": []
}
```

### 遇到的问题

#### 1. 自己创建settings.json 来配置 是错的，应该用 mcp.json

当前
```bash
D:\myPrograms\DailyUse> pnpm dlx @modelcontextprotocol/server-filesystem 'D:\myPrograms\DailyUse'

Secure MCP Filesystem Server running on stdio
```
他能运行，但是 copilot chat 并不能使用

```json
// .vscode/settings.json
// MCP (Model Context Protocol) 配置 - 优化版本
"github.copilot.chat.experimental.modelContextProtocol": {
  "enabled": true,
  "servers": {
    // 使用已安装的 MCP 文件系统服务器
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "d:\\myPrograms\\DailyUse"],
      "env": {
        "NODE_ENV": "development"
      }
    }
  }
},
```

#### 2. filesystem 的配置

- `too many arguments. Expected 0 arguments but got 2.`
- `[server stderr] Usage: mcp-server-filesystem --allowed-directories <allowed-directories>`
- `Error:  Error: ENOENT: no such file or directory, stat 'd:\myPrograms\DailyUse,D:\workProgram\olp-admin'`

```json
"@agent-infra/mcp-server-filesystem": {
  "command": "npx",
  "args": [
    "@agent-infra/mcp-server-filesystem@latest"
  ],
  "env": {
    "ALLOWED_DIRECTORIES": "d:\\myPrograms\\DailyUse,D:\\workProgram\\olp-admin"
  },
  "type": "stdio"
}
```

这里:参数变量不能有，又要传入 `--allowed-directories <allowed-directories>`,传入的文件夹路径应该以 string[] 的格式传入