<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# MCP (Model Context Protocol) 模型上下文协议

此文件夹包含了有关 MCP 服务配置的详细说明和示例，以及 Claude Code 如何使用 MCP。

## 概述

MCP 是一种标准化方式，能让 Claude 访问外部工具、应用程序编程接口（API）以及实时数据源。
和 Memory 记忆不同，MCP 能够实时访问不断变化的数据。

主要特点：
- 可实时访问外部服务
- 可实时数据同步
- 可扩展架构
- 具备安全认证
- 基于工具的交互

## MCP 架构

```mermaid
graph TB
    A["Claude"]
    B["MCP Server"]
    C["External Service"]

    A -->|Request: list_issues| B
    B -->|Query| C
    C -->|Data| B
    B -->|Response| A

    A -->|Request: create_issue| B
    B -->|Action| C
    C -->|Result| B
    B -->|Response| A

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 生态系统

```mermaid
graph TB
    A["Claude"] -->|MCP| B["Filesystem<br/>MCP Server"]
    A -->|MCP| C["GitHub<br/>MCP Server"]
    A -->|MCP| D["Database<br/>MCP Server"]
    A -->|MCP| E["Slack<br/>MCP Server"]
    A -->|MCP| F["Google Docs<br/>MCP Server"]

    B -->|File I/O| G["Local Files"]
    C -->|API| H["GitHub Repos"]
    D -->|Query| I["PostgreSQL/MySQL"]
    E -->|Messages| J["Slack Workspace"]
    F -->|Docs| K["Google Drive"]

    style A fill:#e1f5fe,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#f3e5f5,stroke:#333,color:#333
    style F fill:#f3e5f5,stroke:#333,color:#333
    style G fill:#e8f5e9,stroke:#333,color:#333
    style H fill:#e8f5e9,stroke:#333,color:#333
    style I fill:#e8f5e9,stroke:#333,color:#333
    style J fill:#e8f5e9,stroke:#333,color:#333
    style K fill:#e8f5e9,stroke:#333,color:#333
```

## MCP 安装方法

Claude Code 支持多种传输协议用于与 MCP 服务进行连接

### HTTP 传输协议（推荐）

```bash
# Basic HTTP connection
claude mcp add --transport http notion https://mcp.notion.com/mcp

# HTTP with authentication header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### 标准传输协议（本地）

对于本地运行的 MCP

```bash
# Local Node.js server
claude mcp add --transport stdio myserver -- npx @myorg/mcp-server

# With environment variables
claude mcp add --transport stdio myserver --env KEY=value -- npx server
```

### SSE 传输协议（废弃）

Server-Sent Events 服务器推送事件传输协议已经被弃用，现推荐使用 `http` 方式，但仍保留支持

```bash
claude mcp add --transport sse legacy-server https://example.com/sse
```

### WebSocket 传输协议

WebSocket 用于建立持久双向连接

```bash
claude mcp add --transport ws realtime-server wss://example.com/mcp
```

### Windows 系统特别说明

在原生的 Windows 系统（非 WSL 环境）中，使用 `cmd /c` 来执行 npx 命令

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### OAuth 2.0 认证

Claude Code 为需要使用 OAuth 2.0 协议的 MCP 提供了支持。
在连接到支持 OAuth 协议的 MCP 时，Claude Code 会负责整个认证流程：

```bash
# Connect to an OAuth-enabled MCP server (interactive flow)
claude mcp add --transport http my-service https://my-service.example.com/mcp

# Pre-configure OAuth credentials for non-interactive setup
claude mcp add --transport http my-service https://my-service.example.com/mcp \
  --client-id "your-client-id" \
  --client-secret "your-client-secret" \
  --callback-port 8080
```

| 特性 | 说明 |
|---------|-------------|
| 交互式 OAuth | 使用 `/mcp` 来触发基于浏览器的 OAuth 流程 |
| 预配置的 OAuth 客户端 | 为常见的服务（如 Notion、Stripe 等）提供了内置的 OAuth 客户端（v2.1.30 及以上版本）|
| 预配置的凭证 | `--client-id`, `--client-secret`, `--callback-port` 这些标识可以自动化设置 |
| Token 存储 | Tokens 安全地存储在系统密钥库中 |
| 升级认证 | 支持对特权操作的升级认证 |
| 发现元数据缓存 | OAuth 发现元数据会被缓存以实现更快的重新连接 |
| 元数据覆盖 | 在 `.mcp.json` 中 `oauth.authServerMetadataUrl` 可用来覆盖默认的 OAuth 元数据发现相关设置 |

#### 覆盖 OAuth 元数据发现相关设置

如果 MCP 在标准的 OAuth 元数据端点 (`/.well-known/oauth-authorization-server`) 上返回错误，但提供了有效的 OIDC 端点，你可以告诉 Claude Code 从特定的 URL 获取 OAuth 元数据。

可通过 `oauth` 中的 `authServerMetadataUrl` 设置。

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

注意，URL 必须使用 `https://` 格式
此配置需要 Claude Code v2.1.64 或更高版本

### Claude.ai MCP 服务连接

在 Claude.ai 账户中配置的 MCP 服务会自动在 Claude Code 中生效。
这意味着通过 Claude.ai 网页界面设置的任何 MCP 连接都无需额外配置即可使用。

Claude.ai MCP 连接在 `--print` 模式下也可使用（v2.1.83 及以上版本），这使得其能够实现非交互式和脚本化的使用。

如果要在 Claude Code 中禁用 Claude.ai 的 MCP 服务器，可以设置 `ENABLE_CLAUDEAI_MCP_SERVERS` 环境变量设置为 `false`

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

> **注意：** 该功能仅对使用 Claude.ai 账户登录的用户开放。

## MCP 设置流程

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Config as Config File
    participant Service as External Service

    User->>Claude: Type /mcp
    Claude->>Claude: List available MCP servers
    Claude->>User: Show options
    User->>Claude: Select GitHub MCP
    Claude->>Config: Update configuration
    Config->>Claude: Activate connection
    Claude->>Service: Test connection
    Service-->>Claude: Authentication successful
    Claude->>User: ✅ MCP connected!
```

## MCP 工具搜索

当 MCP 工具描述内容所占的范围超过上下文窗口的 10% 时，Claude Code 会自动启用工具搜索功能，以便高效地选择合适的工具，同时不会使模型上下文变得过于复杂。

| 设置 | 值 | 说明 |
|---------|-------|-------------|
| `ENABLE_TOOL_SEARCH` | `auto` （默认值） | 当 MCP 工具描述内容超出上下文内容的 10% 时自动启用 |
| `ENABLE_TOOL_SEARCH` | `auto:<N>` | 在特定阈值（`N`个工具）达到时自动启用 |
| `ENABLE_TOOL_SEARCH` | `true` | 不论 MCP 工具数量多少，始终启用工具搜索 |
| `ENABLE_TOOL_SEARCH` | `false` | 禁用工具搜索；所有 MCP 工具描述内容将完整的发送给大模型 |

> 注意：MCP 工具搜索功能需要使用 Sonnet 4 或更高版本，或者 Opus 4 或更高版本。Haiku 模型不支持工具搜索。

## MCP 动态更新

Claude Code 支持 MCP 的 `list_changed` 通知功能。
当 MCP 服务器动态地添加、删除或修改其可用工具时，Claude Code 会接收到更新信息，并自动调整其工具列表，而无需重新连接或重启。

## MCP 应用程序

MCP 应用程序是首个官方的 MCP 扩展程序，它使得 MCP 工具调用结果能够以交互式用户界面组件的形式直接在聊天界面中呈现。
MCP 服务不再仅提供简单的文本回复，而是能够提供丰富的仪表板、表单、数据可视化以及多步骤工作流程，所有内容均会直接显示在对话框内，无需离开当前对话界面。

## MCP 交互

MCP 服务可以通过交互式对话框向用户请求结构化的输入（从 v2.1.49 版本开始支持）。
这使得 MCP 服务能够在工作流程进行过程中请求额外的信息；例如，提示进行确认、从列表选项中进行选择，或者填写必填字段；从而为 MCP 服务的交互增加了互动性。

## 工具描述与指令上限

自 v2.1.84 版本起，Claude Code 对每个 MCP 服务的工具描述与指令实施 2KB 的上限。
这能够避免个别 MCP 服务因冗长的工具定义而过度占用上下文资源，从而缓解上下文膨胀并保持交互效率。

## MCP 提示词作为斜杠命令使用

MCP 服务器可以对外暴露提示词，这些提示词在 Claude Code 中会以斜杠命令的形式显示。
提示词可通过以下命名约定来调用：

```
/mcp__<server>__<prompt>
```
例如，如果 `github` 的 MCP 服务提供了一个名为 `review` 的提示框，那么可以通过输入 `/mcp__github__review` 来调用它。

## MCP 服务去重

当同一个 MCP 服务在多个范围（本地、项目、用户）中被定义时，本地配置将优先生效。
这使得你能够通过本地自定义设置来覆盖项目级别或用户级别的 MCP 设置，而不会产生冲突。

## @ 语法引用 MCP 资源

你可以在提示中使用 `@` 语法直接引用 MCP 资源

```
@server-name:protocol://resource/path
```

例如，要引用特定的数据库资源

```
@database:postgres://mydb/users
```

这使得 Claude 能够将 MCP 资源内容直接嵌入到对话内容中，使其成为对话背景的一部分。

## MCP 作用域

MCP 配置可以存储在不同的作用域，并具有不同的共享级别。

| 作用域 | 存储位置 | 说明 | 共享对象 | 是否需要批准 |
|-------|----------|-------------|-------------|------------------|
| 本地（默认） | `~/.claude.json` （位于项目路径下） | 仅对当前用户、当前项目私有 (旧版本中称为 `project`) | 仅你本人 | 否 |
| 项目 | `.mcp.json` | 提交至 Git 仓库 | 团队成员 | 是（首次使用时）|
| 用户 | `~/.claude.json` | 在所有项目中均可用 (旧版本中称为 `global`) | 仅你本人 | 否 |

### 使用项目作用域

将项目特定的 MCP 配置存储在 `.mcp.json` 文件中

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.github.com/mcp"
    }
  }
}
```

团队成员在首次使用项目 MCP 时，将看到批准提示。

## MCP 配置管理

### 添加 MCP 服务

```bash
# Add HTTP-based server
claude mcp add --transport http github https://api.github.com/mcp

# Add local stdio server
claude mcp add --transport stdio database -- npx @company/db-server

# List all MCP servers
claude mcp list

# Get details on specific server
claude mcp get github

# Remove an MCP server
claude mcp remove github

# Reset project-specific approval choices
claude mcp reset-project-choices

# Import from Claude Desktop
claude mcp add-from-claude-desktop
```

## 可用 MCP 服务器列表

| MCP 服务器 | 用途 | 常用工具 | 认证方式 | 是否实时 |
|------------|---------|--------------|------|-----------|
| Filesystem | 文件操作 | read, write, delete | 操作系统权限 | 是 |
| GitHub | 仓库管理 | list_prs, create_issue, push | OAuth | 是 |
| Slack | 团队沟通 | send_message, list_channels | Token | 是 |
| Database | SQL 查询 | query, insert, update | Credentials | 是 |
| Google Docs | 文档访问 | read, write, share | OAuth | 是 |
| Asana | 项目管理 | create_task, update_status | API Key | 是 |
| Stripe | 支付数据 | list_charges, create_invoice | API Key | 是 |
| Memory | 持久化记忆 | store, retrieve, delete | 本地 | 否 |

## 实用示例

### 示例 1: GitHub MCP 配置

**文件:** `.mcp.json` (项目根目录)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**可用的 GitHub MCP 工具:**

#### 合并请求管理

- `list_prs` - 列出仓库中的所有合并请求
- `get_pr` - 获取合并请求详情（含差异对比）
- `create_pr` - 创建新的合并请求
- `update_pr` - 更新合并请求的描述/标题
- `merge_pr` - 将合并请求合并到主分支
- `review_pr` - 添加审查评论

**示例请求:**
```
/mcp__github__get_pr 456

# Returns:
Title: Add dark mode support
Author: @alice
Description: Implements dark theme using CSS variables
Status: OPEN
Reviewers: @bob, @charlie
```

#### 议题管理

- `list_issues` - 列出所有议题
- `get_issue` - 获取议题详情
- `create_issue` - 创建新议题
- `close_issue` - 关闭议题
- `add_comment` - 为议题添加评论

#### 仓库信息

- `get_repo_info` - 仓库详情
- `list_files` - 文件树结构
- `get_file_content` - 读取文件内容
- `search_code` - 搜索代码库

#### 提交操作

- `list_commits` - 提交历史
- `get_commit` - 指定提交详情
- `create_commit` - 创建新提交

**设置**:
```bash
export GITHUB_TOKEN="your_github_token"
# Or use the CLI to add directly:
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

### 配置中的环境变量扩展

MCP 配置支持环境变量扩展，并可指定回退默认值。
`${VAR}` 和 `${VAR:-default}` 这两种语法可在以下字段中使用：`command`, `args`, `env`, `url`, 和 `headers`

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}",
        "X-Custom-Header": "${CUSTOM_HEADER:-default-value}"
      }
    },
    "local-server": {
      "command": "${MCP_BIN_PATH:-npx}",
      "args": ["${MCP_PACKAGE:-@company/mcp-server}"],
      "env": {
        "DB_URL": "${DATABASE_URL:-postgresql://localhost/dev}"
      }
    }
  }
}
```

变量在运行时展开：
- `${VAR}` - 使用环境变量，若未设置则报错
- `${VAR:-default}` - 使用环境变量，若未设置则回退至默认值

### 示例 2: 数据库 MCP 设置

**配置：**

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-database"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

**示例用法：**

```markdown
User: Fetch all users with more than 10 orders

Claude: I'll query your database to find that information.

# Using MCP database tool:
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING COUNT(o.id) > 10
ORDER BY order_count DESC;

# Results:
- Alice: 15 orders
- Bob: 12 orders
- Charlie: 11 orders
```

**设置**:

```bash
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
# Or use the CLI to add directly:
claude mcp add --transport stdio database -- npx @modelcontextprotocol/server-database
```

### 示例 3: 多 MCP 工作流

**场景：生成每日报告**

```markdown
# Daily Report Workflow using Multiple MCPs

## Setup
1. GitHub MCP - fetch PR metrics
2. Database MCP - query sales data
3. Slack MCP - post report
4. Filesystem MCP - save report

## Workflow

### Step 1: Fetch GitHub Data
/mcp__github__list_prs completed:true last:7days

Output:
- Total PRs: 42
- Average merge time: 2.3 hours
- Review turnaround: 1.1 hours

### Step 2: Query Database
SELECT COUNT(*) as sales, SUM(amount) as revenue
FROM orders
WHERE created_at > NOW() - INTERVAL '1 day'

Output:
- Sales: 247
- Revenue: $12,450

### Step 3: Generate Report
Combine data into HTML report

### Step 4: Save to Filesystem
Write report.html to /reports/

### Step 5: Post to Slack
Send summary to #daily-reports channel

Final Output:
✅ Report generated and posted
📊 47 PRs merged this week
💰 $12,450 in daily sales
```

**设置**:

```bash
export GITHUB_TOKEN="your_github_token"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="your_slack_token"
# Add each MCP server via the CLI or configure them in .mcp.json
```

### 示例 4：文件系统 MCP 操作

**配置：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**可用操作：**

| 操作 | 命令 | 用途 |
|-----------|---------|---------|
| 列出文件 | `ls ~/projects` | 显示目录内容 |
| 读取文件 | `cat src/main.ts` | 读取文件内容 |
| 写入文件 | `create docs/api.md` | 创建新文件 |
| 编辑文件 | `edit src/app.ts` | 修改文件 |
| 搜索 | `grep "async function"` | 在文件中搜索 |
| 删除 | `rm old-file.js` | 删除文件 |

**设置**:

```bash
# Use the CLI to add directly:
claude mcp add --transport stdio filesystem -- npx @modelcontextprotocol/server-filesystem /home/user/projects
```

## MCP vs Memory：决策矩阵

```mermaid
graph TD
    A["Need external data?"]
    A -->|No| B["Use Memory"]
    A -->|Yes| C["Does it change frequently?"]
    C -->|No/Rarely| B
    C -->|Yes/Often| D["Use MCP"]

    B -->|Stores| E["Preferences<br/>Context<br/>History"]
    D -->|Accesses| F["Live APIs<br/>Databases<br/>Services"]

    style A fill:#fff3e0,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#fff3e0,stroke:#333,color:#333
    style D fill:#f3e5f5,stroke:#333,color:#333
    style E fill:#e8f5e9,stroke:#333,color:#333
    style F fill:#e8f5e9,stroke:#333,color:#333
```

## 请求/响应模式

```mermaid
sequenceDiagram
    participant App as Claude
    participant MCP as MCP Server
    participant DB as Database

    App->>MCP: Request: "SELECT * FROM users WHERE id=1"
    MCP->>DB: Execute query
    DB-->>MCP: Result set
    MCP-->>App: Return parsed data
    App->>App: Process result
    App->>App: Continue task

    Note over MCP,DB: Real-time access<br/>No caching
```

## 环境变量

将敏感凭据存储在环境变量中：

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxx"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="xoxb-xxxxxxxxxxxxx"
```

然后在 MCP 配置中引用它们：

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```

## Claude 作为 MCP 服务器（`claude mcp serve`）

Claude Code 自身可作为 MCP 服务器供其他应用程序使用。
这使得外部工具、编辑器及自动化系统能够通过标准 MCP 协议调用 Claude 的各项能力。

```bash
# Start Claude Code as an MCP server on stdio
claude mcp serve
```

其他应用程序随后即可像连接任何基于标准输入输出（stdio）的 MCP 服务器一样连接到此服务器。
例如，若要在另一个 Claude Code 实例中将 Claude Code 添加为 MCP 服务器：

```bash
claude mcp add --transport stdio claude-agent -- claude mcp serve
```

这对于构建多智能体工作流非常有用，其中一个 Claude 实例可以编排另一个实例。

## 托管 MCP 配置（企业版）

对于企业部署而言，IT 管理员可通过 `managed-mcp.json` 配置文件来强制实施 MCP 服务器策略。
该文件可对允许或禁止在整个组织范围内使用的 MCP 服务器进行排他性控制。

**位置：**

- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `~/.config/ClaudeCode/managed-mcp.json`
- Windows: `%APPDATA%\ClaudeCode\managed-mcp.json`

**特性：**

- `allowedMcpServers`——允许使用的服务器白名单
- `deniedMcpServers`——禁止使用的服务器黑名单
- 支持按服务器名称、命令及 URL 模式进行匹配
- 组织级 MCP 策略优先于用户配置执行
- 防止未经授权的服务器连接

**配置示例：**

```json
{
  "allowedMcpServers": [
    {
      "serverName": "github",
      "serverUrl": "https://api.github.com/mcp"
    },
    {
      "serverName": "company-internal",
      "serverCommand": "company-mcp-server"
    }
  ],
  "deniedMcpServers": [
    {
      "serverName": "untrusted-*"
    },
    {
      "serverUrl": "http://*"
    }
  ]
}
```

> **注意：** 当 `allowedMcpServers` 与 `deniedMcpServers` 同时匹配某个服务器时，拒绝规则优先。

## 插件提供的 MCP 服务器

插件可以打包自有的 MCP 服务器，当插件安装后这些服务器便会自动可用。
插件提供的 MCP 服务器可通过以下两种方式定义：

1. 独立的 `.mcp.json` 文件：在插件根目录中放置一个 `.mcp.json` 文件
2. 内嵌于 `plugin.json` 中：直接在插件清单文件内定义 MCP 服务器

使用 `${CLAUDE_PLUGIN_ROOT}` 变量来引用相对于插件安装目录的路径：

```json
{
  "mcpServers": {
    "plugin-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/dist/mcp-server.js"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

## Subagent 作用域的 MCP

MCP 服务器可在 Agent 前言元数据（frontmatter）中使用 `mcpServers:` 字段进行内联定义，从而将其作用域限定在特定 Subagent 而非整个项目。
当某个 Agent 需要访问工作流中其他 Agent 并不需要的特定 MCP 服务器时，此功能非常实用。

```yaml
---
mcpServers:
  my-tool:
    type: http
    url: https://my-tool.example.com/mcp
---

You are an agent with access to my-tool for specialized operations.
```

## MCP 输出限制

Claude Code 对 MCP 工具的输出实施限制，以防止上下文溢出：

| 限制类型 | 阈值 | 行为 |
|-------|-----------|----------|
| 警告 | 10,000 tokens | 显示输出内容过大的警告 |
| 默认最大值 | 25,000 tokens | 超出此限制的输出内容将被截断 |
| 磁盘持久化 | 50,000 字符 | 超过 50K 字符的工具结果将持久化至磁盘 |

最大输出限制可通过 `MAX_MCP_OUTPUT_TOKENS` 环境变量进行配置：

```bash
# Increase the max output to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

## 通过代码执行解决上下文膨胀问题

随着 MCP 采用规模扩大，连接数十台服务器及成百上千个工具会带来一个重大挑战：**上下文膨胀**
这可以说是 MCP 规模化过程中最大的问题，而 Anthropic 工程团队已提出一种优雅的解决方案：使用代码执行来替代直接工具调用。

> **来源**：[基于 MCP 的代码执行：构建更高效的智能体](https://www.anthropic.com/engineering/code-execution-with-mcp) —— Anthropic 工程博客

### 问题：Token 浪费的2个源头

**1. 工具定义使上下文窗口不堪重负**

大多数 MCP 客户端会预先加载全部工具定义。
当连接了数千个工具时，模型甚至在读取用户请求之前就必须处理数十万个 Token。

**2. 中间结果消耗额外 token**

每一个中间工具结果都会流经模型的上下文。
试想将一场会议的文字记录从 Google Drive 传输至 Salesforce，这份完整记录会在上下文中流过 2 次：一次是读取时，另一次是写入目标位置时。
一场两小时的会议记录可能意味着额外消耗 50k+ token。

```mermaid
graph LR
    A["Model"] -->|"Tool Call: getDocument"| B["MCP Server"]
    B -->|"Full transcript (50K tokens)"| A
    A -->|"Tool Call: updateRecord<br/>(re-sends full transcript)"| B
    B -->|"Confirmation"| A

    style A fill:#ffcdd2,stroke:#333,color:#333
    style B fill:#f3e5f5,stroke:#333,color:#333
```

### 解决方案：将 MCP 工具作为代码 API 使用

Agent 不再通过上下文窗口传递工具定义及结果，而是 **编写代码** 将 MCP 工具作为 API 来调用。
该代码在沙盒化的执行环境中运行，只有最终结果会返回给模型。

```mermaid
graph LR
    A["Model"] -->|"Writes code"| B["Code Execution<br/>Environment"]
    B -->|"Calls tools directly"| C["MCP Servers"]
    C -->|"Data stays in<br/>execution env"| B
    B -->|"Only final result<br/>(minimal tokens)"| A

    style A fill:#c8e6c9,stroke:#333,color:#333
    style B fill:#e1f5fe,stroke:#333,color:#333
    style C fill:#f3e5f5,stroke:#333,color:#333
```

#### 工作原理

MCP 工具以带类型的函数文件树形式呈现：

```
servers/
├── google-drive/
│   ├── getDocument.ts
│   └── index.ts
├── salesforce/
│   ├── updateRecord.ts
│   └── index.ts
└── ...
```

每个工具文件都包含一个带类型的封装器：

```typescript
// ./servers/google-drive/getDocument.ts
import { callMCPTool } from "../../../client.js";

interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

export async function getDocument(
  input: GetDocumentInput
): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>(
    'google_drive__get_document', input
  );
}
```

Agent 随后编写代码来编排这些工具：

```typescript
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

// Data flows directly between tools — never through the model
const transcript = (
  await gdrive.getDocument({ documentId: 'abc123' })
).content;

await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

结果：Token 用量从约 150,000 降至约 2,000，降幅达 98.7%。

### 核心优势

| 优势 | 说明 |
|---------|-------------|
| 渐进式披露 | Agent 通过浏览文件系统仅加载所需工具定义，而非预先加载全部工具 |
| 上下文高效的结果 | 数据在返回模型前于执行环境中完成过滤/转换处理 |
| 强大的控制流 | 循环、条件判断与错误处理在代码中运行，无需反复经由模型周转 |
| 隐私保护 | 中间数据（个人身份信息、敏感记录等）留存于执行环境，绝不进入模型上下文 |
| 状态持久化 | Agent 可将中间结果保存至文件，并构建可复用的技能函数 |

#### 示例：过滤大型数据集

```typescript
// Without code execution — all 10,000 rows flow through context
// TOOL CALL: gdrive.getSheet(sheetId: 'abc123')
//   -> returns 10,000 rows in context

// With code execution — filter in the execution environment
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(
  row => row["Status"] === 'pending'
);
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // Only 5 rows reach the model
```

#### 示例：无需反复和模型往返的循环操作

```typescript
// Poll for a deployment notification — runs entirely in code
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({
    channel: 'C123456'
  });
  found = messages.some(
    m => m.text.includes('deployment complete')
  );
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

### 需要权衡的取舍

代码执行本身也带来了额外的复杂性。
运行 Agent 生成的代码需要：

- 一个具备适当资源限制的**安全沙盒执行环境**
- 对已执行代码进行**监控与日志记录**
- 相比直接工具调用，需承担额外的**基础设施开销**

由此带来的收益，如降低 token 成本、减少延迟、改进工具组合能力，需与上述实现成本进行权衡。
对于仅连接少数 MCP 服务器的 Agent 而言，直接工具调用可能更简单。
而对于大规模部署（数十台服务器、数百个工具）的 Agent 来说，代码执行则是一项显著的改进。

### MCPorter：MCP 工具组合的运行时

[MCPorter](https://github.com/steipete/mcporter) 是一个 TypeScript 运行时与命令行工具包，可让调用 MCP 服务器变得轻松便捷、无需样板代码，并通过选择性工具暴露及带类型的封装器来帮助缓解上下文膨胀问题。

它解决的问题： MCPorter 允许您按需发现、检查并调用特定工具，而不是预先从所有 MCP 服务器加载全部工具定义，从而让您的上下文保持精简。

**核心特性：**

| 特性 | 描述 |
|---------|-------------|
| 零配置发现 | 自动从 Cursor、Claude、Codex 或本地配置中发现 MCP 服务器 |
| 带类型的工具客户端 | `mcporter emit-ts` 生成 `.d.ts` 接口及可直接运行的封装器 |
| 可组合 API | `createServerProxy()` 将工具暴露为驼峰式方法，并提供 `.text()`、`.json()`、`.markdown()` 辅助方法 |
| 命令行工具生成 | `mcporter generate-cli` 可将任意 MCP 服务器转换为独立命令行工具，并支持 `--include-tools` / `--exclude-tools` 筛选 |
| 参数隐藏 | 可选参数默认保持隐藏，减少模式定义的冗余 |

**安装：**

```bash
npx mcporter list          # No install required — discover servers instantly
pnpm add mcporter          # Add to a project
brew install steipete/tap/mcporter  # macOS via Homebrew
```

**示例，在 TypeScript 中组合工具：**

```typescript
import { createRuntime, createServerProxy } from "mcporter";

const runtime = await createRuntime();
const gdrive = createServerProxy(runtime, "google-drive");
const salesforce = createServerProxy(runtime, "salesforce");

// Data flows between tools without passing through the model context
const doc = await gdrive.getDocument({ documentId: "abc123" });
await salesforce.updateRecord({
  objectType: "SalesMeeting",
  recordId: "00Q5f000001abcXYZ",
  data: { Notes: doc.text() }
});
```

**示例，命令行工具调用：**

```bash
# Call a specific tool directly
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'

# List available servers and tools
npx mcporter list
```

MCPorter 通过提供以带类型的 API 方式调用 MCP 工具所需的运行时基础设施，对上述代码执行方案形成了有力补充，这使得将中间数据排除在模型上下文之外的操作变得简便易行。

## 最佳实践

### 安全考量

#### 建议事项

- 所有凭据均使用环境变量
- 定期轮换 token 和 API 密钥（建议每月一次）
- 尽可能使用只读 token
- 将 MCP 服务器访问范围限制在所需的最低限度
- 监控 MCP 服务器的使用情况和访问日志
- 外部服务如提供 OAuth，优先使用
- 对 MCP 请求实施速率限制
- 投入生产前先测试 MCP 连接
- 记录所有活跃的 MCP 连接
- 保持 MCP 服务器软件包为最新版本

#### 禁忌事项

- 请勿在配置文件中硬编码凭据
- 请勿将 token 或密钥提交至 Git 仓库
- 请勿在团队聊天或邮件中共享 tokens
- 请勿将个人 token 用于团队项目
- 请勿授予不必要的权限
- 请勿忽视身份验证错误
- 请勿将 MCP 端点公开暴露于公网
- 请勿以 root/admin 权限运行 MCP 服务器
- 请勿在日志中缓存敏感数据
- 请勿禁用身份验证机制

### 配置最佳实践

1. 版本控制：将 `.mcp.json` 纳入 Git 管理，但通过环境变量存放机密信息
2. 最小权限：仅授予每个 MCP 服务器所需的最低权限
3. 隔离运行：尽可能在不同进程中运行不同的 MCP 服务器
4. 监控记录：记录所有 MCP 请求和错误，以便审计追溯
5. 测试验证：投入生产环境前，测试所有 MCP 配置

### 性能优化提示

- 在应用层面缓存频繁访问的数据
- 使用针对性强的 MCP 查询，减少数据传输量
- 监控 MCP 操作的响应时间
- 对外部 API 实施适当的速率限制
- 执行多项操作时采用批量处理

## 安装说明

### 前提条件

- 已安装 Node.js 与 npm
- 已安装 Claude Code CLI
- 持有外部服务的 API 令牌/凭据

### 分步设置指南

1. 添加首个 MCP 服务器，通过 CLI 操作（以 GitHub 为例）：

```bash
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

   Or create a `.mcp.json` file in your project root:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```
2. 设置环境变量：

```bash
export GITHUB_TOKEN="your_github_personal_access_token"
```

3. 测试连接：

```bash
claude /mcp
```

4. 使用 MCP 工具：

```bash
/mcp__github__list_prs
/mcp__github__create_issue "Title" "Description"
```

### 特定服务的安装说明

GitHub MCP:

```bash
npm install -g @modelcontextprotocol/server-github
```

Database MCP:

```bash
npm install -g @modelcontextprotocol/server-database
```

Filesystem MCP:

```bash
npm install -g @modelcontextprotocol/server-filesystem
```

Slack MCP:

```bash
npm install -g @modelcontextprotocol/server-slack
```

## 故障排查

### 找不到 MCP 服务器

```bash
# Verify MCP server is installed
npm list -g @modelcontextprotocol/server-github

# Install if missing
npm install -g @modelcontextprotocol/server-github
```

### 身份验证失败

```bash
# Verify environment variable is set
echo $GITHUB_TOKEN

# Re-export if needed
export GITHUB_TOKEN="your_token"

# Verify token has correct permissions
# Check GitHub token scopes at: https://github.com/settings/tokens
```

### 连接超时

- 检查网络连通性：`ping api.github.com`
- 确认 API 端点可访问
- 检查 API 的速率限制
- 尝试在配置中增加超时时间
- 排查防火墙或代理问题

### MCP 服务器崩溃

- 检查 MCP 服务器日志：`~/.claude/logs/`
- 确认所有环境变量均已设置
- 确保文件权限正确
- 尝试重新安装 MCP 服务器软件包
- 排查是否存在端口冲突的进程

## 相关概念

### Memory vs MCP

- **Memory**：存储持久不变的静态数据（偏好设置、上下文、历史记录）
- **MCP**：访问实时变化的动态数据（API、数据库、实时服务）

### 使用场景区分

- **使用 Memory 的场景**：用户偏好、对话历史、已习得的上下文
- **使用 MCP 的场景**：当前的 GitHub 议题、实时数据库查询、实时数据

### 与其他 Claude 功能的集成

- 将 MCP 与 Memory 结合，构建丰富的上下文
- 在提示词中使用 MCP 工具以增强推理能力
- 利用多个 MCP 服务器实现复杂的工作流

## 其他资源

- [Official MCP Documentation](https://code.claude.com/docs/en/mcp)
- [MCP Protocol Specification](https://modelcontextprotocol.io/specification)
- [MCP GitHub Repository](https://github.com/modelcontextprotocol/servers)
- [Available MCP Servers](https://github.com/modelcontextprotocol/servers)
- [MCPorter](https://github.com/steipete/mcporter) — TypeScript runtime & CLI for calling MCP servers without boilerplate
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — Anthropic's engineering blog on solving context bloat
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude API Documentation](https://docs.anthropic.com)

---
**Last Updated**: April 2026
**Claude Code Version**: 2.1+
**Compatible Models**: Claude Sonnet 4.6, Claude Opus 4.6, Claude Haiku 4.5
