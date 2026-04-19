<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# Hooks 钩子

钩子是在 Claude Code 会话期间响应特定事件而自动执行的脚本。
它们可用于实现自动化、验证、权限管理以及自定义工作流。

## 概述

钩子是一种自动化动作（Shell 命令、HTTP Webhook、LLM 提示词或子代理评估），当 Claude Code 中发生特定事件时会自动执行。
它们接收 JSON 输入，并通过退出码和 JSON 输出进行通信。

**核心特性：**
- 事件驱动的自动化
- 基于 JSON 的输入/输出
- 支持命令、提示词、HTTP 及代理等多种钩子类型
- 可针对特定工具进行模式匹配

## 配置

钩子在设置文件中按特定结构进行配置：

- `~/.claude/settings.json` - 用户级设置（适用于所有项目）
- `.claude/settings.json` - 项目级设置（可共享，需提交）
- `.claude/settings.local.json` - 本地项目设置（不提交）
- 托管策略 - 组织级设置
- 插件 `hooks/hooks.json` - 插件作用域内的钩子
- Skill/Agent frontmatter 前言元数据 - 组件生命周期钩子

### 基本配置结构

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

**关键字段说明：**

| 字段 | 描述 | 示例 |
|-------|-------------|---------|
| `matcher` | 匹配工具名称的模式（区分大小写） | `"Write"`、`"Edit\|Write"`、`"*"` |
| `hooks` | 钩子定义的数组 | `[{ "type": "command", ... }]` |
| `type` | 钩子类型：`"command"`（Bash 命令）、`"prompt"`（LLM 提示词）、`"http"`（Webhook）或 `"agent"`（子代理） | `"command"` |
| `command` | 要执行的 Shell 命令 | `"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh"` |
| `timeout` | 可选超时时间，单位为秒（默认 60 秒） | `30` |
| `once` | 若设为 `true`，则每个会话只运行一次该钩子 | `true` |

### 匹配器模式

| 模式 | 描述 | 示例 |
|---------|-------------|---------|
| 精确字符串 | 匹配特定工具名称 | `"Write"` |
| 正则表达式 | 匹配多个工具 | `"Edit\|Write"` |
| 通配符 | 匹配所有工具 | `"*"` 或 `""` |
| MCP 工具 | 匹配 MCP 服务器及工具 | `"mcp__memory__.*"` |

**InstructionsLoaded 指令加载时可匹配的值：**

| 匹配的值 | 描述 |
|---------------|-------------|
| `session_start` | 会话启动时加载的指令 |
| `nested_traversal` | 嵌套目录遍历期间加载的指令 |
| `path_glob_match` | 通过路径 Glob 模式匹配加载的指令 |

## 钩子类型

Claude Code 支持四种钩子类型：

### Command 命令钩子

默认的钩子类型。执行一个 Shell 命令，并通过 JSON 标准输入/输出及退出码进行通信。

```json
{
  "type": "command",
  "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py\"",
  "timeout": 60
}
```

### HTTP 钩子

> 于 v2.1.63 版本新增。

远程 Webhook 端点，接收与命令钩子相同的 JSON 输入。
HTTP 钩子通过 POST 请求将 JSON 发送至指定 URL，并接收 JSON 格式的响应。
当沙盒功能启用时，HTTP 钩子会经由沙盒进行路由。
出于安全考虑，URL 中的环境变量插值需要显式指定 `allowedEnvVars` 列表。

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "http",
      "url": "https://my-webhook.example.com/hook",
      "matcher": "Write"
    }]
  }
}
```

**关键属性：**
- `"type": "http"` - 标识这是一个 HTTP 钩子
- `"url"` - Webhook 端点 URL
- 当沙盒启用时，请求会通过沙盒路由
- 若 URL 中包含任何环境变量插值，需显式提供 `allowedEnvVars` 列表

### Prompt 提示词钩子

LLM 评估的提示词，其中钩子内容由 Claude 进行评估的提示词构成。
主要用于 `Stop` 和 `SubagentStop` 事件，以进行智能的任务完成检查。

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude completed all requested tasks.",
  "timeout": 30
}
```

LLM 对提示词进行评估并返回结构化的决策结果（详见[基于提示词的钩子](#prompt-based-hooks)）。

### Agent 钩子

基于子代理的验证钩子，它会生成一个专用代理来评估条件或执行复杂的检查。
与提示词钩子（单轮 LLM 评估）不同，代理钩子可以使用工具并进行多步骤推理。

```json
{
  "type": "agent",
  "prompt": "Verify the code changes follow our architecture guidelines. Check the relevant design docs and compare.",
  "timeout": 120
}
```

**关键属性：**
- `"type": "agent"` - 标识这是一个代理钩子
- `"prompt"` - 子代理的任务描述
- 该代理可使用工具（如 Read、Grep、Bash 等）来执行评估
- 返回与提示词钩子类似的结构化决策结果

## Hook Events 钩子事件

Claude Code 支持 **26 种钩子事件**：

| 事件 | 触发时机 | 匹配器输入 | 是否可阻断 | 常见用途 |
|-------|---------------|---------------|-----------|------------|
| **SessionStart** | 会话开始/恢复/清空/压缩 | startup/resume/clear/compact | 否 | 环境设置 |
| **InstructionsLoaded** | CLAUDE.md 或规则文件加载后 | （无） | 否 | 修改/筛选指令 |
| **UserPromptSubmit** | 用户提交提示词 | （无） | 是 | 验证提示词 |
| **PreToolUse** | 工具执行前 | 工具名称 | 是（允许/拒绝/询问） | 验证、修改输入 |
| **PermissionRequest** | 权限对话框显示时 | 工具名称 | 是 | 自动批准/拒绝 |
| **PermissionDenied** | 用户拒绝权限请求 | 工具名称 | 否 | 日志记录、分析、策略执行 |
| **PostToolUse** | 工具成功执行后 | 工具名称 | 否 | 添加上下文、反馈 |
| **PostToolUseFailure** | 工具执行失败 | 工具名称 | 否 | 错误处理、日志记录 |
| **Notification** | 通知发送时 | 通知类型 | 否 | 自定义通知 |
| **SubagentStart** | 子代理生成时 | 代理类型名称 | 否 | 子代理设置 |
| **SubagentStop** | 子代理结束时 | 代理类型名称 | 是 | 子代理验证 |
| **Stop** | Claude 完成响应时 | （无） | 是 | 任务完成检查 |
| **StopFailure** | API 错误结束轮次 | （无） | 否 | 错误恢复、日志记录 |
| **TeammateIdle** | 代理团队成员空闲时 | （无） | 是 | 队友协调 |
| **TaskCompleted** | 任务标记为完成时 | （无） | 是 | 任务后续操作 |
| **TaskCreated** | 通过 TaskCreate 创建任务 | （无） | 否 | 任务追踪、日志记录 |
| **ConfigChange** | 配置文件变更时 | （无） | 是（策略除外） | 响应配置更新 |
| **CwdChanged** | 工作目录变更时 | （无） | 否 | 目录特定设置 |
| **FileChanged** | 受监控文件变更时 | （无） | 否 | 文件监控、重新构建 |
| **PreCompact** | 上下文压缩前 | manual/auto | 否 | 压缩前操作 |
| **PostCompact** | 压缩完成后 | （无） | 否 | 压缩后操作 |
| **WorktreeCreate** | 创建工作树时 | （无） | 是（返回路径） | 工作树初始化 |
| **WorktreeRemove** | 移除工作树时 | （无） | 否 | 工作树清理 |
| **Elicitation** | MCP 服务器请求用户输入 | （无） | 是 | 输入验证 |
| **ElicitationResult** | 用户响应引导请求时 | （无） | 是 | 响应处理 |
| **SessionEnd** | 会话终止时 | （无） | 否 | 清理、最终日志记录 |

### PreToolUse

在 Claude 生成工具参数之后、实际处理之前运行。
用于验证或修改工具的输入参数。

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py"
          }
        ]
      }
    ]
  }
}
```

**常用匹配器：** `Task`、`Bash`、`Glob`、`Grep`、`Read`、`Edit`、`Write`、`WebFetch`、`WebSearch`

**输出控制：**
- `permissionDecision`：`"allow"`（允许）、`"deny"`（拒绝）或 `"ask"`（询问）
- `permissionDecisionReason`：决策的说明原因
- `updatedInput`：修改后的工具输入参数

### PostToolUse

在工具完成执行后立即运行。
可用于验证结果、记录日志，或向 Claude 返回上下文信息。

**配置：**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `"block"` 决策会向 Claude 提示反馈信息
- `additionalContext`：为 Claude 添加的额外上下文信息

### UserPromptSubmit

当用户提交提示词后、Claude 处理之前运行。

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `decision`：设为 `"block"` 可阻止处理
- `reason`：若被阻止，提供说明原因
- `additionalContext`：添加到提示词的额外上下文信息

### Stop and SubagentStop

当 Claude 完成响应（`Stop` 事件）或子代理执行完毕（`SubagentStop` 事件）时运行。支持基于提示词的评估，用于智能检查任务是否完成。

**附加输入字段：** `Stop` 和 `SubagentStop` 钩子的 JSON 输入中均包含 `last_assistant_message` 字段，其中记录了 Claude 或子代理在停止前返回的最后一条消息。该字段有助于评估任务的完成情况。

**配置：**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude completed all requested tasks.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### SubagentStart

当子代理开始执行时运行。
匹配器的输入参数为代理类型名称，这使得钩子能够针对特定的子代理类型进行精确绑定。

**配置：**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "code-review",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/subagent-init.sh"
          }
        ]
      }
    ]
  }
}
```

### SessionStart

当会话启动或恢复时运行。可以持久化环境变量。

**匹配器：** `startup`、`resume`、`clear`、`compact`

**特殊功能：** 使用 `CLAUDE_ENV_FILE` 可持久化环境变量（此功能在 `CwdChanged` 和 `FileChanged` 钩子中同样可用）：

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=development' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

### SessionEnd

当会话结束时运行，用于执行清理或最终日志记录。无法阻止会话终止。

**原因字段（`reason`）的可选值：**
- `clear` - 用户清空了会话
- `logout` - 用户已注销
- `prompt_input_exit` - 用户通过提示输入退出了会话
- `other` - 其他原因

**配置：**
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-cleanup.sh\""
          }
        ]
      }
    ]
  }
}
```

### Notification Event

通知事件的匹配器更新：
- `permission_prompt` - 权限请求通知
- `idle_prompt` - 空闲状态通知
- `auth_success` - 认证成功通知
- `elicitation_dialog` - 向用户显示的对话窗口

## Component-Scoped Hooks 组件作用域钩子

钩子可以附加到特定组件（技能、代理、命令）的前言元数据中：

**在 SKILL.md、agent.md 或 command.md 文件中：**

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true  # Only run once per session
---
```

**组件钩子支持的事件：** `PreToolUse`、`PostToolUse`、`Stop`

这样便可将钩子直接定义在使用它的组件内部，使相关代码得以聚合在一起。

### 子代理前言元数据中的钩子

当在子代理的前言元数据中定义了 `Stop` 钩子时，该钩子会自动转换为作用域限定于该子代理的 `SubagentStop` 钩子。
这确保了该停止钩子仅在该特定子代理完成时触发，而非在主会话停止时触发。

```yaml
---
name: code-review-agent
description: Automated code review subagent
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify the code review is thorough and complete."
  # The above Stop hook auto-converts to SubagentStop for this subagent
---
```

## PermissionRequest Event

处理权限请求，并支持自定义输出格式：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": {},
      "message": "Custom message",
      "interrupt": false
    }
  }
}
```

## Hook 钩子输入与输出

### JSON 输入（通过标准输入 stdin）

所有钩子均通过标准输入接收 JSON 格式的输入数据：

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_use_id": "toolu_01ABC123...",
  "agent_id": "agent-abc123",
  "agent_type": "main",
  "worktree": "/path/to/worktree"
}
```

**常见字段：**

| 字段 | 描述 |
|-------|-------------|
| `session_id` | 唯一会话标识符 |
| `transcript_path` | 对话转录文件的路径 |
| `cwd` | 当前工作目录 |
| `hook_event_name` | 触发钩子的事件名称 |
| `agent_id` | 运行此钩子的代理标识符 |
| `agent_type` | 代理类型（`"main"`、子代理类型名称等） |
| `worktree` | 代理所在 Git 工作树的路径（若存在） |

### 退出码

| 退出码 | 含义 | 行为 |
|-----------|---------|----------|
| **0** | 成功 | 继续执行，解析 JSON 标准输出 |
| **2** | 阻断性错误 | 阻断操作，标准错误输出显示为错误信息 |
| **其他** | 非阻断性错误 | 继续执行，标准错误输出仅在详细模式下显示 |

### JSON 输出（标准输出 stdout，退出码为 0）

```json
{
  "continue": true,
  "stopReason": "Optional message if stopping",
  "suppressOutput": false,
  "systemMessage": "Optional warning message",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "File is in allowed directory",
    "updatedInput": {
      "file_path": "/modified/path.js"
    }
  }
}
```

## 环境变量

| 变量 | 可用范围 | 描述 |
|----------|-------------|-------------|
| `CLAUDE_PROJECT_DIR` | 所有钩子 | 项目根目录的绝对路径 |
| `CLAUDE_ENV_FILE` | SessionStart、CwdChanged、FileChanged | 用于持久化环境变量的文件路径 |
| `CLAUDE_CODE_REMOTE` | 所有钩子 | 在远程环境中运行时值为 `"true"` |
| `${CLAUDE_PLUGIN_ROOT}` | 插件钩子 | 插件目录的路径 |
| `${CLAUDE_PLUGIN_DATA}` | 插件钩子 | 插件数据目录的路径 |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd 钩子 | 为 SessionEnd 钩子配置的超时时间（毫秒），可覆盖默认值 |

## Prompt-Based Hooks 基于提示词的钩子

对于 `Stop` 和 `SubagentStop` 事件，你可以使用基于 LLM 的评估方式：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if all tasks are complete. Return your decision.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**LLM 响应结构：**
```json
{
  "decision": "approve",
  "reason": "All tasks completed successfully",
  "continue": false,
  "stopReason": "Task complete"
}
```

## 示例

### 示例 1：Bash 命令验证器（PreToolUse）

**文件：** `.claude/hooks/validate-bash.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"\brm\s+-rf\s+/", "Blocking dangerous rm -rf / command"),
    (r"\bsudo\s+rm", "Blocking sudo rm command"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name != "Bash":
        sys.exit(0)

    command = input_data.get("tool_input", {}).get("command", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, command):
            print(message, file=sys.stderr)
            sys.exit(2)  # Exit 2 = blocking error

    sys.exit(0)

if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\""
          }
        ]
      }
    ]
  }
}
```

### 示例 2：安全扫描器（PostToolUse）

**文件：** `.claude/hooks/security-scan.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

SECRET_PATTERNS = [
    (r"password\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded password"),
    (r"api[_-]?key\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded API key"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name not in ["Write", "Edit"]:
        sys.exit(0)

    tool_input = input_data.get("tool_input", {})
    content = tool_input.get("content", "") or tool_input.get("new_string", "")
    file_path = tool_input.get("file_path", "")

    warnings = []
    for pattern, message in SECRET_PATTERNS:
        if re.search(pattern, content, re.IGNORECASE):
            warnings.append(message)

    if warnings:
        output = {
            "hookSpecificOutput": {
                "hookEventName": "PostToolUse",
                "additionalContext": f"Security warnings for {file_path}: " + "; ".join(warnings)
            }
        }
        print(json.dumps(output))

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 3：自动格式化代码（PostToolUse）

**文件：** `.claude/hooks/format-code.sh`

```bash
#!/bin/bash

# Read JSON from stdin
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_name', ''))")
FILE_PATH=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_input', {}).get('file_path', ''))")

if [ "$TOOL_NAME" != "Write" ] && [ "$TOOL_NAME" != "Edit" ]; then
    exit 0
fi

# Format based on file extension
case "$FILE_PATH" in
    *.js|*.jsx|*.ts|*.tsx|*.json)
        command -v prettier &>/dev/null && prettier --write "$FILE_PATH" 2>/dev/null
        ;;
    *.py)
        command -v black &>/dev/null && black "$FILE_PATH" 2>/dev/null
        ;;
    *.go)
        command -v gofmt &>/dev/null && gofmt -w "$FILE_PATH" 2>/dev/null
        ;;
esac

exit 0
```

### 示例 4：提示词验证器（UserPromptSubmit）

**文件：** `.claude/hooks/validate-prompt.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"delete\s+(all\s+)?database", "Dangerous: database deletion"),
    (r"rm\s+-rf\s+/", "Dangerous: root deletion"),
]

def main():
    input_data = json.load(sys.stdin)
    prompt = input_data.get("user_prompt", "") or input_data.get("prompt", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, prompt, re.IGNORECASE):
            output = {
                "decision": "block",
                "reason": f"Blocked: {message}"
            }
            print(json.dumps(output))
            sys.exit(0)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 5：智能停止钩子（Prompt-Based）

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if Claude completed all requested tasks. Check: 1) Were all files created/modified? 2) Were there unresolved errors? If incomplete, explain what's missing.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 示例 6：上下文用量追踪器（成对钩子 Hook Pairs）

结合使用 `UserPromptSubmit`（消息发送前）和 `Stop`（响应结束后）钩子，追踪每次请求的令牌消耗。

**文件：** `.claude/hooks/context-tracker.py`

```python
#!/usr/bin/env python3
"""
Context Usage Tracker - Tracks token consumption per request.

Uses UserPromptSubmit as "pre-message" hook and Stop as "post-response" hook
to calculate the delta in token usage for each request.

Token Counting Methods:
1. Character estimation (default): ~4 chars per token, no dependencies
2. tiktoken (optional): More accurate (~90-95%), requires: pip install tiktoken
"""
import json
import os
import sys
import tempfile

# Configuration
CONTEXT_LIMIT = 128000  # Claude's context window (adjust for your model)
USE_TIKTOKEN = False    # Set True if tiktoken is installed for better accuracy


def get_state_file(session_id: str) -> str:
    """Get temp file path for storing pre-message token count, isolated by session."""
    return os.path.join(tempfile.gettempdir(), f"claude-context-{session_id}.json")


def count_tokens(text: str) -> int:
    """
    Count tokens in text.

    Uses tiktoken with p50k_base encoding if available (~90-95% accuracy),
    otherwise falls back to character estimation (~80-90% accuracy).
    """
    if USE_TIKTOKEN:
        try:
            import tiktoken
            enc = tiktoken.get_encoding("p50k_base")
            return len(enc.encode(text))
        except ImportError:
            pass  # Fall back to estimation

    # Character-based estimation: ~4 characters per token for English
    return len(text) // 4


def read_transcript(transcript_path: str) -> str:
    """Read and concatenate all content from transcript file."""
    if not transcript_path or not os.path.exists(transcript_path):
        return ""

    content = []
    with open(transcript_path, "r") as f:
        for line in f:
            try:
                entry = json.loads(line.strip())
                # Extract text content from various message formats
                if "message" in entry:
                    msg = entry["message"]
                    if isinstance(msg.get("content"), str):
                        content.append(msg["content"])
                    elif isinstance(msg.get("content"), list):
                        for block in msg["content"]:
                            if isinstance(block, dict) and block.get("type") == "text":
                                content.append(block.get("text", ""))
            except json.JSONDecodeError:
                continue

    return "\n".join(content)


def handle_user_prompt_submit(data: dict) -> None:
    """Pre-message hook: Save current token count before request."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Save to temp file for later comparison
    state_file = get_state_file(session_id)
    with open(state_file, "w") as f:
        json.dump({"pre_tokens": current_tokens}, f)


def handle_stop(data: dict) -> None:
    """Post-response hook: Calculate and report token delta."""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # Load pre-message count
    state_file = get_state_file(session_id)
    pre_tokens = 0
    if os.path.exists(state_file):
        try:
            with open(state_file, "r") as f:
                state = json.load(f)
                pre_tokens = state.get("pre_tokens", 0)
        except (json.JSONDecodeError, IOError):
            pass

    # Calculate delta
    delta_tokens = current_tokens - pre_tokens
    remaining = CONTEXT_LIMIT - current_tokens
    percentage = (current_tokens / CONTEXT_LIMIT) * 100

    # Report usage
    method = "tiktoken" if USE_TIKTOKEN else "estimated"
    print(f"Context ({method}): ~{current_tokens:,} tokens ({percentage:.1f}% used, ~{remaining:,} remaining)", file=sys.stderr)
    if delta_tokens > 0:
        print(f"This request: ~{delta_tokens:,} tokens", file=sys.stderr)


def main():
    data = json.load(sys.stdin)
    event = data.get("hook_event_name", "")

    if event == "UserPromptSubmit":
        handle_user_prompt_submit(data)
    elif event == "Stop":
        handle_stop(data)

    sys.exit(0)


if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ]
  }
}
```

**工作原理：**
1. `UserPromptSubmit` 在您的提示词被处理前触发 - 保存当前的令牌数量
2. `Stop` 在 Claude 响应后触发 - 计算增量并报告使用情况
3. 通过临时文件名中的 `session_id` 实现各会话间的隔离

**Token 计数方法：**

| 方法 | 准确度 | 依赖项 | 速度 |
|--------|----------|--------------|-------|
| 字符估算 | 约 80-90% | 无 | <1ms |
| tiktoken (p50k_base) | 约 90-95% | `pip install tiktoken` | <10ms |

> **注意：** Anthropic 尚未发布官方的离线令牌计数工具。上述两种方法均为近似估算。转录内容包含用户提示词、Claude 的响应以及工具输出，但**不包含**系统提示词或内部上下文。

### 示例 7：种子化自动模式权限（一次性设置脚本）

一个一次性设置脚本，用于向 `~/.claude/settings.json` 文件中植入约 67 条安全权限规则，效果等同于 Claude Code 的自动模式基线--无需任何钩子，也无需记住后续选择。
运行一次即可；可重复运行（已存在的规则会被跳过）。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply
python3 09-advanced-features/setup-auto-mode-permissions.py
```

**添加的内容包括：**

| 类别 | 示例 |
|----------|---------|
| 内置工具 | `Read(*)`、`Edit(*)`、`Write(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)` |
| Git 读取操作 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)` |
| Git 写入操作（本地） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建与测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 Shell 命令 | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

**有意排除的内容**（此脚本绝不会添加）：
- `rm -rf`、`sudo`、强制推送、`git reset --hard`
- `DROP TABLE`、`kubectl delete`、`terraform destroy`
- `npm publish`、`curl | bash`、生产环境部署

## Plugin Hooks 插件钩子

插件可在其 `hooks/hooks.json` 文件中包含钩子定义：

**文件：** `plugins/hooks/hooks.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

**插件钩子中的环境变量：**
- `${CLAUDE_PLUGIN_ROOT}` - 插件目录的路径
- `${CLAUDE_PLUGIN_DATA}` - 插件数据目录的路径

这使得插件能够包含自定义的验证和自动化钩子。

## MCP Tool Hooks （MCP 钩子）

MCP 工具的命名模式为 `mcp__<server>__<tool>`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"systemMessage\": \"Memory operation logged\"}'"
          }
        ]
      }
    ]
  }
}
```

## 安全考量

### 免责声明

**请自行承担使用风险**：钩子会执行任意 Shell 命令。你对以下事项负有全部责任：
- 你所配置的命令
- 文件访问及修改权限
- 潜在的数据丢失或系统损坏
- 投入生产环境前，需在安全环境中充分测试钩子

### 安全注意事项

- **需要工作区信任**：`statusLine` 和 `fileSuggestion` 钩子输出命令现需获得工作区信任接受后方可生效。
- **HTTP 钩子与环境变量**：HTTP 钩子需显式提供 `allowedEnvVars` 列表，方可在 URL 中使用环境变量插值。此举可防止敏感环境变量意外泄露至远程端点。
- **托管设置层级**：`disableAllHooks` 设置现已遵循托管设置层级体系，这意味着组织级别的设置可强制禁用钩子，且单个用户无法覆盖该设置。

### 最佳实践

| 建议事项 | 避免事项 |
|-----|-------|
| 验证并清洗所有输入 | 盲目信任输入数据 |
| 引用 Shell 变量：`"$VAR"` | 不引用：`$VAR` |
| 阻止路径遍历（`..`） | 允许任意路径 |
| 使用绝对路径配合 `$CLAUDE_PROJECT_DIR` | 硬编码路径 |
| 跳过敏感文件（`.env`、`.git/`、密钥文件） | 处理所有文件 |
| 先独立测试钩子 | 部署未测试的钩子 |
| 为 HTTP 钩子显式指定 `allowedEnvVars` | 将所有环境变量暴露给 Webhook |

## 调试

### 启用调试模式

使用调试标志运行 Claude，以获取详细的钩子日志：

```bash
claude --debug
```

### 详细模式

在 Claude Code 中使用 `Ctrl+O` 可启用详细模式，以查看钩子的执行进度。

### 独立测试钩子

```bash
# Test with sample JSON input
echo '{"tool_name": "Bash", "tool_input": {"command": "ls -la"}}' | python3 .claude/hooks/validate-bash.py

# Check exit code
echo $?
```

## 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/format-code.sh\"",
            "timeout": 30
          },
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-init.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify all tasks are complete before stopping.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## 钩子执行细节

| 方面 | 行为说明 |
|--------|----------|
| Timeout-超时时间 | 默认 60 秒，可按命令单独配置 |
| Parallelization-并行执行 | 所有匹配的钩子并行运行 |
| Deduplication-去重机制 | 相同的钩子命令会被去重处理 |
| Environment-运行环境 | 在当前目录下使用 Claude Code 的环境变量运行 |

## 故障排查

### 钩子未执行
- 验证 JSON 配置语法是否正确
- 检查匹配器模式是否与工具名称相符
- 确保脚本存在且具有可执行权限：`chmod +x script.sh`
- 运行 `claude --debug` 查看钩子执行日志
- 确认钩子从标准输入（stdin）读取 JSON，而非通过命令行参数

### 钩子意外阻断
- 使用示例 JSON 测试钩子：`echo '{"tool_name": "Write", ...}' | ./hook.py`
- 检查退出码：应为 0（允许）或 2（阻断）
- 检查标准错误输出（退出码为 2 时会显示）

### JSON 解析错误
- 始终从标准输入读取，而非命令行参数
- 使用正确的 JSON 解析方式（避免字符串操作）
- 妥善处理缺失字段的情况

## 安装

### 步骤 1：创建钩子目录
```bash
mkdir -p ~/.claude/hooks
```

### 步骤 2：复制示例钩子
```bash
cp 06-hooks/*.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/*.sh
```

### 步骤 3：在设置中配置
编辑 `~/.claude/settings.json` 或 `.claude/settings.json`，添加上方所示的钩子配置。

## 相关概念

- **[Checkpoints and Rewind](../08-checkpoints/)** - Save and restore conversation state
- **[Slash Commands](../01-slash-commands/)** - Create custom slash commands
- **[Skills](../03-skills/)** - Reusable autonomous capabilities
- **[Subagents](../04-subagents/)** - Delegated task execution
- **[Plugins](../07-plugins/)** - Bundled extension packages
- **[Advanced Features](../09-advanced-features/)** - Explore advanced Claude Code capabilities

## 其他资源

- **[Official Hooks Documentation](https://code.claude.com/docs/en/hooks)** - Complete hooks reference
- **[CLI Reference](https://code.claude.com/docs/en/cli-reference)** - Command-line interface documentation
- **[Memory Guide](../02-memory/)** - Persistent context configuration

---
**Last Updated**: April 2026
**Claude Code Version**: 2.1+
**Compatible Models**: Claude Sonnet 4.6, Claude Opus 4.6, Claude Haiku 4.5
