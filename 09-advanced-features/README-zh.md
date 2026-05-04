<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 高级功能

Claude Code 高级功能综合指南，涵盖计划模式、扩展思维、自动模式、后台任务、权限模式、打印模式（非交互式）、会话管理、交互功能、Channels-频道、语音输入、远程控制、Web 会话、桌面应用、任务列表、提示建议、Git 工作树、沙盒、托管设置及配置等内容。

---

## 概述

Claude Code 的高级功能通过计划、推理、自动化及控制机制扩展了核心能力。
这些功能可为复杂开发任务、代码审查、自动化以及多会话管理等场景提供复杂的工作流支持。

**主要的高级功能包括：**
- **计划模式**：在编码前制定详细的实现方案
- **扩展思维**：针对复杂问题进行深度推理
- **自动模式**：后台安全分类器在每次操作前进行审查（研究预览版）
- **后台任务**：在不阻塞对话的情况下运行耗时操作
- **权限模式**：控制 Claude 可以执行哪些操作（`default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`）
- **打印模式**：以非交互方式运行 Claude Code，适用于自动化及 CI/CD 场景（`claude -p`）
- **会话管理**：管理多个工作会话
- **交互功能**：键盘快捷键、多行输入及命令历史记录
- **语音输入**：支持 20 种语言语音转文字的按下说话功能
- **Channels-频道**：MCP 服务器向运行中的会话推送消息（研究预览版）
- **远程控制**：从 Claude.ai 或 Claude app 控制 Claude Code
- **Web 会话**：在浏览器中通过 claude.ai/code 运行 Claude Code
- **桌面应用**：用于可视化差异审查及多会话管理的独立应用
- **任务列表**：跨上下文压缩的持久化任务追踪
- **提示建议**：基于上下文的智能命令建议
- **Git 工作树**：用于并行工作的隔离分支工作树
- **沙盒**：操作系统级的文件系统与网络隔离
- **托管设置**：通过 plist、注册表或托管文件进行企业级部署
- **配置**：使用 JSON 配置文件自定义行为

---

## 计划模式

计划模式允许 Claude 在执行复杂任务之前进行深入思考，生成一份详细的计划供你审查和批准。

### 什么是计划模式？

计划模式采用两阶段的工作方式：
1. **计划阶段**：Claude 分析任务并制定详细的实现方案
2. **实施阶段**：在你批准方案后，Claude 执行该计划

### 何时使用计划模式

✅ 适用于以下场景：
- 复杂的多文件重构
- 新功能开发
- 架构变更
- 数据库迁移
- 重大 API 重新设计

❌ 不适用于以下场景：
- 简单的 Bug 修复
- 格式化调整
- 单文件编辑
- 快速查询

### 激活计划模式

**斜杠命令**：
```bash
/plan Implement user authentication system
```

**CLI 标志**：
```bash
claude --permission-mode plan
```

**设为默认**：
```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

**键盘快捷键**：
- `Shift + Tab` — 切换权限模式（包括计划模式）
- `Alt + M` — 切换权限模式（Windows/Linux 系统）

### 计划模式的优势

- **结构化的实现方法**：详细拆分实现步骤
- **审查与批准**：在执行前批准或调整计划
- **风险识别**：在实现前识别潜在问题
- **清晰的阶段划分**：有序的实现阶段和里程碑

### 示例：功能实现

```
User: /plan Add a real-time notification system

Claude: I'll create a plan for implementing real-time notifications.

## Implementation Plan

### Phase 1: Infrastructure Setup
1. Install WebSocket libraries (socket.io)
2. Set up WebSocket server
3. Configure connection management

### Phase 2: Backend Implementation
4. Create notification service
5. Implement notification queue
6. Add database schema for notifications
7. Create API endpoints

### Phase 3: Frontend Integration
8. Set up WebSocket client
9. Create notification UI components
10. Implement notification state management
11. Add notification permissions

### Phase 4: Testing & Deployment
12. Write unit tests
13. Write integration tests
14. Update documentation
15. Deploy to staging

Estimated time: 3-4 hours
Files to modify: 12
New files to create: 8

Ready to proceed? (yes/no/modify plan)

User: yes

Claude: [Begins implementation following the plan]
```

### 计划修改

```
User: Modify the plan - skip the queue for now, we'll add it later

Claude: Updated plan:
[Shows modified plan with queue removed]

User: Looks good, proceed

Claude: [Implements modified plan]
```

## 计划模式配置

计划模式通过命令行标志或斜杠命令激活：

```bash
# Activate plan mode via CLI
claude --permission-mode plan

# Or use the /plan slash command inside the REPL
/plan Implement user authentication system
```

**计划模式专属模型别名**：使用 `opusplan` 模型别名，即可调用 Opus 进行计划制定，再由 Sonnet 执行具体实施：

```bash
claude --model opusplan "design and implement the new API"
```

**在外部编辑器中编辑计划**：按 `Ctrl+G` 即可在当前外部编辑器中打开当前计划，进行详细修改。

### Ultraplan

使用 `/ultraplan <prompt>` 启动端到端的计划工作流：
Claude 起草一份详细的计划，在浏览器中打开以供审查，然后远程执行该计划或将其发送回终端供本地执行。

---

## 扩展思维

扩展思维让 Claude 能够在给出解决方案之前，投入更多时间对复杂问题进行深入推理。

### 什么是扩展思维？

扩展思维是一个审慎的、逐步推进的推理过程，Claude 在此过程中会：
- 拆分复杂问题
- 考量多种实施路径
- 评估各项利弊权衡
- 对边界情况进行逻辑推理

### 激活扩展思维

**键盘快捷键**：
- `Option + T`（macOS）/ `Alt + T`（Windows/Linux）— 切换扩展思维的开/关

**自动激活**：
- 所有模型默认启用（Opus 4.6、Sonnet 4.6、Haiku 4.5）
- Opus 4.6：自适应推理，支持工作强度级别：`low`（○）、`medium`（◐）、`high`（●）、`max`（仅限 Opus 4.6）
- 其他模型：固定预算最高可达 31,999 个 token

**配置方式**：
- 开关切换：`Alt+T` / `Option+T`，或通过 `/config`
- 查看推理过程：`Ctrl+O`（详细模式）
- 设置工作强度：`/effort` 命令或 `--effort` 标志

**自定义预算**：
```bash
export MAX_THINKING_TOKENS=1024
```

**工作强度级别**（仅限 Opus 4.6）：
```bash
export CLAUDE_CODE_EFFORT_LEVEL=high   # low (○), medium (◐), high (●), or max (Opus 4.6 only)
```

**CLI 标志**：
```bash
claude --effort high "complex architectural review"
```

**斜杠命令**：
```
/effort high
```

> **注意：** 提示词中出现关键词 "ultrathink" 可激活深度推理模式。工作强度级别 `low`、`medium`、`high` 及 `max`（仅限 Opus 4.6）用于控制 Claude 执行推理的深度。

### 扩展思维的优势

- **深度推理**：投入更多时间思考复杂问题
- **更优的架构决策**：经过全面评估后做出明智选择
- **周全考虑边缘情况**：全面思考潜在问题与场景
- **全面分析**：以结构化方法应对复杂问题求解

### 示例：架构决策

```
User: /think Should we use microservices or a monolith for our e-commerce platform?

Claude: Let me think through this architectural decision carefully...

<extended_thinking>
## Context Analysis
- Team size: 5 developers
- Expected traffic: 10k daily users initially
- Need to scale to 100k+ users
- Limited DevOps resources
- 6-month timeline to MVP

## Approach 1: Microservices
Pros:
+ Independent scaling
+ Technology flexibility
+ Team independence
+ Fault isolation

Cons:
- Operational complexity
- Network overhead
- Distributed tracing needed
- Requires DevOps expertise
- Slower initial development

## Approach 2: Monolith
Pros:
+ Simpler deployment
+ Faster initial development
+ Easier debugging
+ Lower operational overhead
+ Single codebase

Cons:
- Scaling challenges later
- All-or-nothing deployments
- Technology lock-in
- Potential for tight coupling

## Decision Factors
1. Team size (5 devs) → Favors monolith
2. DevOps resources (limited) → Favors monolith
3. Timeline (6 months) → Favors monolith
4. Future scale (100k users) → Achievable with monolith + caching
5. Complexity budget → Limited

## Recommendation
Start with a **modular monolith**:
- Single deployment unit
- Clear module boundaries
- Clean interfaces between modules
- Prepare for future extraction to microservices if needed

This gives you:
✓ Fast initial development
✓ Low operational complexity
✓ Option to extract services later
✓ Clear architecture

Migration path: When you reach 50k+ users or specific modules need independent scaling, extract those modules into microservices.
</extended_thinking>

**Recommendation**: Implement a modular monolith with clean module boundaries.

This approach balances your current constraints (team size, timeline, DevOps resources) with future scalability needs. You can migrate specific modules to microservices later when there's a clear business need.
```

### 扩展思维配置

扩展思维通过环境变量、键盘快捷键和CLI标志进行控制：

```bash
# Set thinking token budget
export MAX_THINKING_TOKENS=16000

# Set effort level (Opus 4.6 only): low (○), medium (◐), high (●), or max (Opus 4.6 only)
export CLAUDE_CODE_EFFORT_LEVEL=high
```

在会话中可通过 `Alt+T` / `Option+T` 切换开关，使用 `/effort` 设置工作强度，或通过 `/config` 进行配置。

---

## 自动模式

自动模式是研究预览版中的一种权限模式（2026年3月发布），它利用后台安全分类器在执行前对每一项操作进行审查。
这让 Claude 能够自主工作的同时，阻止危险操作。

### 使用要求

- **订阅方案**：Team、Enterprise 或 API（Pro 或 Max 方案尚不支持）
- **模型**：Claude Sonnet 4.6 或 Opus 4.6
- **服务提供商**：仅限 Anthropic API（不支持 Bedrock、Vertex 或 Foundry）
- **分类器**：在 Claude Sonnet 4.6 上运行（会产生额外的 Token 开销）

### 启用自动模式

```bash
# Unlock auto mode with CLI flag
claude --enable-auto-mode

# Then cycle to it with Shift+Tab in the REPL
```

或者将其设为默认权限模式：

```bash
claude --permission-mode auto
```

通过配置文件设置：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 分类器工作原理

后台分类器按以下决策顺序评估每一项操作：

1. **Allow/deny 允许/拒绝规则** -- 首先检查显式设置的权限规则
2. **Read-only/edits 只读/编辑操作自动批准** -- 文件读取与编辑操作自动放行
3. **Classifier 分类器** -- 后台分类器对该操作进行审查
4. **Fallback 回退机制** -- 连续 3 次或累计 20 次被阻止后，回退到提示模式

### 默认阻止的操作

自动模式默认阻止以下操作：

| 被阻止的操作 | 示例 |
|----------------|---------|
| Pipe-to-shell 管线命令安装 | `curl \| bash` |
| 对外发送敏感数据 | 通过网络传输 API 密钥、凭据 |
| 生产环境部署 | 针对生产环境的部署命令 |
| 大规模删除 | 对大目录执行 `rm -rf` |
| IAM 变更 | 权限与角色修改 |
| 强制推送到主分支 | `git push --force origin main` |

### 默认允许的操作

| 允许的操作 | 示例 |
|----------------|---------|
| 本地文件操作 | 读取、写入、编辑项目文件 |
| 已声明的依赖项安装 | 根据清单文件执行 `npm install`、`pip install` |
| 只读 HTTP 请求 | 获取文档的 `curl` 命令 |
| 推送到当前分支 | `git push origin feature-branch` |

### 配置自动模式

**以 JSON 格式打印默认规则：**
```bash
claude auto-mode defaults
```

**通过托管设置 `autoMode.environment` 配置受信任的基础设施**（适用于企业部署）。管理员可借此定义受信任的 CI/CD 环境、部署目标以及基础设施模式。

### 回退行为

当分类器无法确定时，自动模式会回退为向用户提示：
- 在分类器**连续 3 次**阻止之后
- 在会话中分类器**总计阻止 20 次**之后

这确保了当分类器无法对某项操作做出有信心的批准时，用户始终保有控制权。

### 植入等效于自动模式的权限（无需 Team 套餐）

若你没有 Team 套餐，或不希望使用后台分类器而想寻求更简单的方式，可以运行一个脚本，为你的 `~/.claude/settings.json` 文件植入一组保守且安全的权限规则基线。
该脚本首先仅开放只读及本地检查类规则，然后在你需要时才让你按需选择启用编辑、测试、本地 Git 写入、软件包安装以及 GitHub 写入等操作权限。

**文件:** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added (no changes written)
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply the conservative baseline
python3 09-advanced-features/setup-auto-mode-permissions.py

# Add more capability only when you need it
python3 09-advanced-features/setup-auto-mode-permissions.py --include-edits --include-tests
python3 09-advanced-features/setup-auto-mode-permissions.py --include-git-write --include-packages
```

该脚本添加的规则涵盖以下类别：

| 类别 | 示例 |
|----------|---------|
| 核心只读工具 | `Read(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)`、`WebFetch(*)` |
| 本地检查 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)`、`Bash(cat:*)` |
| 可选编辑操作 | `Edit(*)`、`Write(*)`、`NotebookEdit(*)` |
| 可选测试/构建 | `Bash(pytest:*)`、`Bash(python3 -m pytest:*)`、`Bash(cargo test:*)` |
| 可选 Git 写入 | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git stash:*)` |
| Git（本地写入） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建与测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 Shell 命令 | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

危险操作（`rm -rf`、`sudo`、强制推送、`DROP TABLE`、`terraform destroy` 等）被有意排除在外。该脚本具有幂等性--运行两次不会重复添加规则。

---

## 后台任务

后台任务允许长时间运行的操作在不阻塞对话的情况下执行。

### 什么是后台任务？

后台任务在你继续工作的同时异步运行：
- 长时间的测试套件
- 构建过程
- 数据库迁移
- 部署脚本
- 分析工具

**基本用法：**
```bash
User: Run tests in background

Claude: Started task bg-1234

/task list           # Show all tasks
/task status bg-1234 # Check progress
/task show bg-1234   # View output
/task cancel bg-1234 # Cancel task
```

### 启动后台任务

```
User: Run the full test suite in the background

Claude: Starting tests in background (task-id: bg-1234)
You can continue working while tests run.

[You can continue the conversation]

User: Meanwhile, let's refactor the auth module

Claude: [Works on auth module while tests run in background]

[Later, when tests complete]

Claude: 📢 Background task bg-1234 completed:
✅ 245 tests passed
❌ 3 tests failed
View results: /task show bg-1234
```

### 管理后台任务

**列出活跃任务**：
```
User: /task list

Active background tasks:
1. [bg-1234] Running tests (50% complete, 2min remaining)
2. [bg-1235] Building Docker image (25% complete, 8min remaining)
3. [bg-1236] Deploying to staging (90% complete, 30sec remaining)
```

**检查任务状态**：
```
User: /task status bg-1234

Task bg-1234: Running tests
Status: In progress
Progress: 120/245 tests (49%)
Started: 2025-11-08 10:30:15
Estimated completion: 2025-11-08 10:34:22
```

**查看任务输出**：
```
User: /task show bg-1234

[Shows live output from the test run]
```

**取消任务**：
```
User: /task cancel bg-1234

Cancelled background task bg-1234
```

### 示例：并行开发

```
User: Run the build in the background

Claude: Starting build... (task-id: bg-5001)

User: Also run the linter in background

Claude: Starting linter... (task-id: bg-5002)

User: While those run, let's implement the new API endpoint

Claude: [Implements API endpoint while build and linter run]

[10 minutes later]

Claude: 📢 Build completed successfully (bg-5001)
📢 Linter found 12 issues (bg-5002)

User: Show me the linter issues

Claude: [Shows linter output from bg-5002]
```

### 配置

```json
{
  "backgroundTasks": {
    "enabled": true,
    "maxConcurrentTasks": 5,
    "notifyOnCompletion": true,
    "autoCleanup": true,
    "logOutput": true
  }
}
```

---

## 定时任务

定时任务允许你按循环计划或一次性提醒的方式自动运行提示词。
任务是会话级别的，它们在 Claude Code 处于活跃状态时运行，并在会话结束时清除。
此功能自 v2.1.72 及以上版本起可用。

### `/loop` 命令

```bash
# Explicit interval
/loop 5m check if the deployment finished

# Natural language
/loop check build status every 30 minutes
```

标准 5 字段的 cron 表达式也受支持，用于精准调度。

### 一次性提醒

设置在特定时间触发一次的提醒：

```
remind me at 3pm to push the release branch
in 45 minutes, run the integration tests
```

### 管理定时任务

| 工具 | 描述 |
|------|-------------|
| `CronCreate` | 创建一条新的定时任务 |
| `CronList` | 列出所有活跃的定时任务 |
| `CronDelete` | 删除一条定时任务 |

**限制与行为**：
- 每个会话最多容纳 **50 个定时任务**
- 会话作用域 — 会话结束时即被清除
- 循环任务超过 **3 天** 后自动过期
- 任务仅在 Claude Code 运行时触发 — 错过的触发不会补执行

### 行为细节

| 方面 | 细节 |
|--------|--------|
| **循环任务抖动** | 最长可达间隔的 10%（最多 15 分钟） |
| **一次性任务抖动** | 在 :00/:30 边界上最多 90 秒 |
| **错过触发** | 不补执行 — 若 Claude Code 未运行则跳过 |
| **持久性** | 重启后不保留 |

### 云端定时任务

使用 `/schedule` 创建在 Anthropic 基础设施上运行的云端定时任务：

```
/schedule daily at 9am run the test suite and report failures
```

云端定时任务在重启后依然保留，且无需在本地运行 Claude Code。

### 禁用定时任务

```bash
export CLAUDE_CODE_DISABLE_CRON=1
```

### 示例：监控部署

```
/loop 5m check the deployment status of the staging environment.
        If the deploy succeeded, notify me and stop looping.
        If it failed, show the error logs.
```

> **提示**：定时任务是会话级别的。对于重启后仍能保留的持久化自动化需求，请改用 CI/CD 流水线、GitHub Actions 或桌面应用定时任务。

---

## 权限模式

权限模式控制 Claude 在无需明确批准的情况下可以执行哪些操作。

### 可用的权限模式

| 模式 | 行为 |
|---|---|
| `default` | 仅读取文件；其他所有操作均需提示确认 |
| `acceptEdits` | 可读取和编辑文件；执行命令需提示确认 |
| `plan` | 仅读取文件（研究模式，不可编辑） |
| `auto` | 所有操作均经过后台安全分类器检查（研究预览版） |
| `bypassPermissions` | 所有操作均无需权限检查（危险） |
| `dontAsk` | 仅执行预先批准的工具；其他所有操作均被拒绝 |

在命令行界面中按 `Shift+Tab` 可循环切换模式。
使用 `--permission-mode` 标志或 `permissions.defaultMode` 设置可设定默认模式。

### 激活方式

**键盘快捷键**：
```bash
Shift + Tab  # Cycle through all 6 modes
```

**斜杠命令**：
```bash
/plan                  # Enter plan mode
```

**CLI 标识**：
```bash
claude --permission-mode plan
claude --permission-mode auto
```

**设置**：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 权限模式示例

#### 默认模式
Claude 在执行重要操作前会请求确认：

```
User: Fix the bug in auth.ts

Claude: I need to modify src/auth.ts to fix the bug.
The change will update the password validation logic.

Approve this change? (yes/no/show)
```

#### 计划模式
在执行前审查实现方案：

```
User: /plan Implement user authentication system

Claude: I'll create a plan for implementing authentication.

## Implementation Plan
[Detailed plan with phases and steps]

Ready to proceed? (yes/no/modify)
```

#### 接受编辑模式
自动接受文件修改：

```
User: acceptEdits
User: Fix the bug in auth.ts

Claude: [Makes changes without asking]
```

### 使用场景

**代码审查**：
```
User: claude --permission-mode plan
User: Review this PR and suggest improvements

Claude: [Reads code, provides feedback, but cannot modify]
```

**结对编程**：
```
User: claude --permission-mode default
User: Let's implement the feature together

Claude: [Asks for approval before each change]
```

**自动化任务**：
```
User: claude --permission-mode acceptEdits
User: Fix all linting issues in the codebase

Claude: [Auto-accepts file edits without asking]
```

---

## 无头模式

打印模式（`claude -p`）允许 Claude Code 在没有交互输入的情况下运行，非常适合自动化及 CI/CD 场景。
这是一种非交互模式，取代了旧的 `--headless` 标志。

### 什么是打印模式？

打印模式可实现：
- 自动化脚本执行
- CI/CD 集成
- 批量处理
- 定时任务

### 在打印模式（非交互式）下运行

```bash
# Run specific task
claude -p "Run all tests"

# Process piped content
cat error.log | claude -p "Analyze these errors"

# CI/CD integration (GitHub Actions)
- name: AI Code Review
  run: claude -p "Review PR"
```

### 其他打印模式用法示例

```bash
# Run a specific task with output capture
claude -p "Run all tests and generate coverage report"

# With structured output
claude -p --output-format json "Analyze code quality"

# With input from stdin
echo "Analyze code quality" | claude -p "explain this"
```

### 示例：CI/CD 集成

**GitHub Actions**:
```yaml
# .github/workflows/code-review.yml
name: AI Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p --output-format json \
            --max-turns 3 \
            "Review this PR for:
            - Code quality issues
            - Security vulnerabilities
            - Performance concerns
            - Test coverage
            Output results as JSON" > review.json

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = JSON.parse(fs.readFileSync('review.json', 'utf8'));
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: JSON.stringify(review, null, 2)
            });
```

### 打印模式配置

打印模式（`claude -p`）支持多个用于自动化的标志：

```bash
# Limit autonomous turns
claude -p --max-turns 5 "refactor this module"

# Structured JSON output
claude -p --output-format json "analyze this codebase"

# With schema validation
claude -p --json-schema '{"type":"object","properties":{"issues":{"type":"array"}}}' \
  "find bugs in this code"

# Disable session persistence
claude -p --no-session-persistence "one-off analysis"
```

---

## 会话管理

有效管理多个 Claude Code 会话。

### 会话管理命令

| 命令 | 描述 |
|---------|-------------|
| `/resume` | 通过 ID 或名称恢复对话 |
| `/rename` | 为当前会话命名 |
| `/fork` | 将当前会话分支到新分支 |
| `claude -c` | 继续最近的对话 |
| `claude -r "session"` | 通过名称或 ID 恢复会话 |

### 恢复会话


**继续上一次对话**：
```bash
claude -c
```

**恢复已命名的会话**：
```bash
claude -r "auth-refactor" "finish this PR"
```

**重命名当前会话**（在 REPL 内部）：
```
/rename auth-refactor
```

### 创建会话分支

创建会话分支，以便在不丢失原始会话的情况下尝试替代方案：
```
/fork
```

或者，在 CLI 中执行：
```bash
claude --resume auth-refactor --fork-session "try OAuth instead"
```

### 会话持久性

会话会自动保存，并可随时恢复：

```bash
# Continue last conversation
claude -c

# Resume specific session by name or ID
claude -r "auth-refactor"

# Resume and fork for experimentation
claude --resume auth-refactor --fork-session "alternative approach"
```

---

## 交互功能

### 键盘快捷键

Claude Code 支持键盘快捷键以提高效率。以下是官方文档的完整参考：

| 快捷键 | 描述 |
|----------|-------------|
| `Ctrl+C` | 取消当前输入/生成 |
| `Ctrl+D` | 退出 Claude Code |
| `Ctrl+G` | 在外部编辑器中编辑计划 |
| `Ctrl+L` | 清除终端屏幕 |
| `Ctrl+O` | 切换详细输出（查看推理过程） |
| `Ctrl+R` | 反向搜索历史记录 |
| `Ctrl+T` | 切换任务列表视图 |
| `Ctrl+B` | 将正在运行的任务转入后台 |
| `Esc+Esc` | 回退代码/对话 |
| `Shift+Tab` / `Alt+M` | 切换权限模式 |
| `Option+P` / `Alt+P` | 切换模型 |
| `Option+T` / `Alt+T` | 切换扩展思维 |

**行编辑（标准 readline 快捷键）：**

| 快捷键 | 操作 |
|----------|--------|
| `Ctrl + A` | 移至行首 |
| `Ctrl + E` | 移至行尾 |
| `Ctrl + K` | 剪切至行尾 |
| `Ctrl + U` | 剪切至行首 |
| `Ctrl + W` | 向后删除一个词 |
| `Ctrl + Y` | 粘贴（yank） |
| `Tab` | 自动补全 |
| `↑ / ↓` | 命令历史记录 |

### 自定义按键绑定

运行 `/keybindings` 即可打开 `~/.claude/keybindings.json` 文件进行编辑，从而创建自定义键盘快捷键（v2.1.18 及以上版本）。

**配置格式**：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null,
        "ctrl+k ctrl+s": "chat:stash"
      }
    },
    {
      "context": "Confirmation",
      "bindings": {
        "ctrl+a": "confirmation:yes"
      }
    }
  ]
}
```

将某个绑定值设为 `null` 即可解除对应的默认快捷键。

### 可用上下文

按键绑定根据不同的 UI 上下文设定作用域：

| 上下文 | 关键操作 |
|---------|-------------|
| **聊天（Chat）** | `submit`（提交）、`cancel`（取消）、`cycleMode`（切换模式）、`modelPicker`（模型选择器）、`thinkingToggle`（思考开关）、`undo`（撤销）、`externalEditor`（外部编辑器）、`stash`（暂存）、`imagePaste`（图片粘贴） |
| **确认（Confirmation）** | `yes`（是）、`no`（否）、`previous`（上一个）、`next`（下一个）、`nextField`（下一个字段）、`cycleMode`（切换模式）、`toggleExplanation`（切换解释） |
| **全局（Global）** | `interrupt`（打断）、`exit`（退出）、`toggleTodos`（切换待办事项）、`toggleTranscript`（切换转录） |
| **自动补全（Autocomplete）** | `accept`（接受）、`dismiss`（取消）、`next`（下一个）、`previous`（上一个） |
| **历史搜索（HistorySearch）** | `search`（搜索）、`previous`（上一个）、`next`（下一个） |
| **设置（Settings）** | 上下文特定的设置导航 |
| **标签页（Tabs）** | 标签页切换与管理 |
| **帮助（Help）** | 帮助面板导航 |

总计有 18 种上下文，包括 `Transcript`（转录）、`Task`（任务）、`ThemePicker`（主题选择器）、`Attachments`（附件）、`Footer`（页脚）、`MessageSelector`（消息选择器）、`DiffDialog`（差异对话框）、`ModelPicker`（模型选择器）和 `Select`（选择）。

### 和弦支持

按键绑定支持和弦序列（即多键组合）：

```
"ctrl+k ctrl+s"   → Two-key sequence: press ctrl+k, then ctrl+s
"ctrl+shift+p"    → Simultaneous modifier keys
```

**按键语法**：
- **修饰键**：`ctrl`、`alt`（或 `opt`）、`shift`、`meta`（或 `cmd`）
- **大写字母表示 Shift**：`K` 等同于 `shift+k`
- **特殊键**：`escape`、`enter`、`return`、`tab`、`space`、`backspace`、`delete`、方向键

### 保留键与冲突键

| 按键 | 状态 | 说明 |
|-----|--------|-------|
| `Ctrl+C` | 已保留 | 无法重新绑定（中断操作） |
| `Ctrl+D` | 已保留 | 无法重新绑定（退出操作） |
| `Ctrl+B` | 终端冲突 | tmux 前缀键 |
| `Ctrl+A` | 终端冲突 | GNU Screen 前缀键 |
| `Ctrl+Z` | 终端冲突 | 进程挂起操作 |

> **提示**：若快捷键无法正常使用，请检查是否与你的终端模拟器或多路复用器存在冲突。

### Tab 键自动补全

Claude Code 提供智能的 Tab 键自动补全功能：

```
User: /rew<TAB>
→ /rewind

User: /plu<TAB>
→ /plugin

User: /plugin <TAB>
→ /plugin install
→ /plugin enable
→ /plugin disable
```

### 命令历史记录

访问之前的命令：

```
User: <↑>  # Previous command
User: <↓>  # Next command
User: Ctrl+R  # Search history

(reverse-i-search)`test': run all tests
```

### 多行输入

对于复杂查询，可使用多行模式：

```bash
User: \
> Long complex prompt
> spanning multiple lines
> \end
```

**示例:**

```
User: \
> Implement a user authentication system
> with the following requirements:
> - JWT tokens
> - Email verification
> - Password reset
> - 2FA support
> \end

Claude: [Processes the multi-line request]
```

### 内联编辑

在发送前编辑命令：

```
User: Deploy to prodcution<Backspace><Backspace>uction

[Edit in-place before sending]
```

### Vim 模式

启用 Vi/Vim 按键绑定以进行文本编辑：

**激活方式**：
- 使用 `/vim` 命令或通过 `/config` 启用
- 按 `Esc` 切换至普通模式，按 `i/a/o` 进入插入模式

**导航键**：
- `h` / `l` - 向左/向右移动
- `j` / `k` - 向下/向上移动
- `w` / `b` / `e` - 按词移动
- `0` / `$` - 移至行首/行尾
- `gg` / `G` - 跳至文本开头/结尾

**文本对象**：
- `iw` / `aw` - 词内/词周围
- `i"` / `a"` - 引号内字符串/引号周围
- `i(` / `a(` - 括号内/括号周围

### Bash 模式

使用 `!` 前缀直接执行 Shell 命令：

```bash
! npm test
! git status
! cat src/index.js
```

无需切换上下文，即可快速执行命令。

---

## 语音输入

语音输入为 Claude Code 提供了按下说话功能，让你可以直接说出提示词，而无需键入。

### 激活语音输入

```
/voice
```

### 功能特性

| 功能 | 描述 |
|---------|-------------|
| **按下说话** | 按住按键录音，松开发送 |
| **20 种语言** | 语音转文字支持 20 种语言 |
| **自定义按键绑定** | 通过 `/keybindings` 配置按下说话按键 |
| **账户要求** | 语音转文字处理需使用 Claude.ai 账户 |

### 配置

在按键绑定文件（`/keybindings`）中自定义按下说话的按键。
语音输入使用你的 Claude.ai 账户进行语音转文字处理。

---

## Channels-频道

频道是一项研究预览版功能，通过 MCP 服务器将来自外部服务的事件推送到正在运行的 Claude Code 会话中。
来源包括 Telegram、Discord、iMessage 以及任意 Webhook，使 Claude 能够无需轮询即可对实时通知做出响应。

### 订阅频道

```bash
# Subscribe to channel plugins at startup
claude --channels discord,telegram

# Subscribe to multiple sources
claude --channels discord,telegram,imessage,webhooks
```

### 支持的集成

| 集成方式 | 描述 |
|-------------|-------------|
| **Discord** | 在会话中接收并回复 Discord 消息 |
| **Telegram** | 在会话中接收并回复 Telegram 消息 |
| **iMessage** | 在会话中接收 iMessage 通知 |
| **Webhooks** | 从任意 Webhook 来源接收事件 |

### 配置

在启动时使用 `--channels` 标志来配置频道。
对于企业部署，可使用托管设置来控制允许哪些频道插件：

```json
{
  "allowedChannelPlugins": ["discord", "telegram"]
}
```

使用 `allowedChannelPlugins` 托管设置用于控制整个组织内允许使用哪些频道插件。

### 工作原理

1. MCP 服务器充当连接外部服务的频道插件
2. 传入的消息与事件被推送到正在运行的 Claude Code 会话中
3. Claude 可在会话上下文中读取并回复消息
4. 频道插件必须通过 `allowedChannelPlugins` 托管设置获得批准
5. 无需轮询，事件实时推送

---

## Chrome 浏览器集成

Chrome 浏览器集成可将 Claude Code 连接到你的 Chrome 或 Microsoft Edge 浏览器，实现实时网页自动化与调试。
这是一项自 v2.0.73 及以上版本起可用的 Beta 功能（Edge 支持自 v1.0.36 及以上版本起加入）。

### 启用 Chrome 浏览器集成

**启动时**：

```bash
claude --chrome      # Enable Chrome connection
claude --no-chrome   # Disable Chrome connection
```

**在会话中**：

```
/chrome
```

选择 "Enabled by default"（默认启用）即可为所有后续会话激活 Chrome 浏览器集成。
Claude Code 会共享你浏览器的登录状态，因此它可以与已认证的网页应用进行交互。

### 功能特性

| 功能 | 描述 |
|------------|-------------|
| **实时调试** | 读取控制台日志、检查 DOM 元素、实时调试 JavaScript |
| **设计验证** | 将渲染页面与设计稿进行比对 |
| **表单验证** | 测试表单提交、输入验证及错误处理 |
| **网页应用测试** | 与已认证的应用（Gmail、Google Docs、Notion 等）进行交互 |
| **数据提取** | 从网页中抓取并处理内容 |
| **会话录制** | 将浏览器交互过程录制为 GIF 文件 |

### 站点级权限

Chrome 扩展程序管理各个站点的访问权限。
你可以随时通过扩展程序弹窗授予或撤销对特定站点的访问权限。
Claude Code 仅与你明确允许的站点进行交互。

### 工作原理

Claude Code 在一个可见窗口中控制浏览器，你可以实时观察操作过程。
当浏览器遇到登录页面或验证码时，Claude 会暂停并等待你手动处理，然后再继续。

### 已知局限

- **浏览器支持**：仅支持 Chrome 和 Edge，不支持 Brave、Arc 及其他基于 Chromium 的浏览器
- **WSL**：在 Windows Subsystem for Linux 中不可用
- **第三方服务提供商**：不支持 Bedrock、Vertex 或 Foundry API 提供商
- **Service Worker 空闲**：Chrome 扩展程序的 Service Worker 在长时间会话中可能进入空闲状态

> **提示**：Chrome 浏览器集成是一项 Beta 功能。浏览器支持范围可能在未来版本中扩展。

---

## 远程控制

远程控制让你可以从手机、平板或任意浏览器继续操控本地正在运行的 Claude Code 会话。
你的本地会话始终在你自己的机器上运行，不会将任何内容移至云端。
此功能适用于 Pro、Max、Team 和 Enterprise 方案（v2.1.51 及以上版本）。

### 启动远程控制

**通过命令行启动**：

```bash
# Start with default session name
claude remote-control

# Start with a custom name
claude remote-control --name "Auth Refactor"
```

**在会话内启动**：

```
/remote-control
/remote-control "Auth Refactor"
```

**可用的标志**：

| 标志 | 描述 |
|------|-------------|
| `--name "title"` | 自定义会话标题，便于识别 |
| `--verbose` | 显示详细的连接日志 |
| `--sandbox` | 启用文件系统与网络隔离 |
| `--no-sandbox` | 禁用沙盒模式（默认） |

### 连接到会话

从其他设备连接有以下三种方式：

1. **会话 Session URL** — 会话启动时打印在终端上；在任意浏览器中打开即可
2. **二维码** — 启动后按空格键可显示扫码二维码
3. **按名称查找** — 在 claude.ai/code 或 Claude 移动应用（iOS/Android）中浏览你的会话

### 安全性

- **不会**在你的机器上开放**任何入站端口**
- 仅通过 TLS 进行**出站 HTTPS 连接**
- **作用域受限的凭据** — 多个短时效、窄作用域的令牌
- **会话隔离** — 每个远程会话相互独立

### 远程控制与 Claude Code 网页版对比

| 方面 | 远程控制 | Claude Code 网页版 |
|--------|---------------|-------------------|
| **执行位置** | 在你的机器上运行 | 在 Anthropic 云端运行 |
| **本地工具** | 完全访问本地 MCP 服务器、文件和命令行 | 无本地依赖项 |
| **适用场景** | 从其他设备继续本地工作 | 从任意浏览器全新开始 |

### 局限

- 每个 Claude Code 实例仅支持一个远程会话
- 主机终端必须保持打开状态
- 若网络不可达，会话约 10 分钟后超时

### 使用场景

- 离开工位时通过移动设备或平板操控 Claude Code
- 在保持本地工具执行的同时，使用功能更丰富的 claude.ai 界面
- 在移动中进行快速代码审查，同时可使用完整的本地开发环境

---

## Web 会话

Web 会话允许你直接在浏览器中通过 claude.ai/code 运行 Claude Code，或从命令行界面创建 Web 会话。

### 创建 Web 会话

```bash
# Create a new web session from the CLI
claude --remote "implement the new API endpoints"
```

这将在 claude.ai 上启动一个 Claude Code 会话，你可以通过任何浏览器访问。

### 本地恢复 Web 会话

如果你在 Web 上启动了某个会话，并希望转到本地继续：

```bash
# Resume a web session in the local terminal
claude --teleport
```

或者在交互式 REPL 中使用以下命令：
```
/teleport
```

### 使用场景

- 在一台机器上开始工作，在另一台机器上继续
- 与团队成员共享会话 URL
- 使用 Web UI 进行可视化差异审查，然后切换到终端执行

---

## 桌面应用

Claude Code 桌面应用提供独立的应用程序体验，支持可视化差异审查、并行会话以及集成连接器。
适用于 macOS 和 Windows 系统（Pro、Max、Team 和 Enterprise 方案）。

### 安装

从 [claude.ai](https://claude.ai) 下载对应平台的安装包：
- **macOS**：通用版本（支持 Apple Silicon 和 Intel 芯片）
- **Windows**：提供 x64 及 ARM64 安装程序

安装指引请参阅[桌面版快速入门](https://code.claude.com/docs/en/desktop-quickstart)。

### 从命令行界面交接

将当前命令行会话转移到桌面应用：

```
/desktop
```

### 核心功能

| 功能 | 描述 |
|---------|-------------|
| **差异视图** | 逐文件可视化审查，支持行内评论；Claude 可读取评论并据此修改 |
| **应用预览** | 自动启动开发服务器，内嵌浏览器实时验证 |
| **PR 监控** | GitHub CLI 集成，自动修复 CI 故障并在检查通过后自动合并 |
| **并行会话** | 侧边栏支持多会话，自动进行 Git 工作树隔离 |
| **定时任务** | 可设置循环任务（每小时、每日、工作日、每周），应用打开时自动运行 |
| **富媒体渲染** | 代码、Markdown 及图表的语法高亮渲染 |

### 应用预览配置

在 `.claude/launch.json` 中配置开发服务器行为：

```json
{
  "command": "npm run dev",
  "port": 3000,
  "readyPattern": "ready on",
  "persistCookies": true
}
```

### 连接器

接入外部服务以获取更丰富的上下文：

| 连接器 | 能力 |
|-----------|------------|
| **GitHub** | PR 监控、Issue 追踪、代码审查 |
| **Slack** | 通知推送、频道上下文 |
| **Linear** | Issue 追踪、迭代管理 |
| **Notion** | 文档、知识库访问 |
| **Asana** | 任务管理、项目追踪 |
| **Calendar** | 日程感知、会议上下文 |

> **注意**：连接器不适用于远程（云端）会话。

### 远程会话与 SSH 会话

- **远程会话**：运行在 Anthropic 云基础设施上；即使应用关闭仍可继续执行。可通过 claude.ai/code 或 Claude 移动应用访问
- **SSH 会话**：通过 SSH 连接远程机器，完全访问远程文件系统及工具。远程机器上需安装 Claude Code

### 桌面应用的权限模式

桌面应用支持与命令行相同的 4 种权限模式：

| 模式 | 行为说明 |
|------|----------|
| **询问权限(Ask permissions)**（默认） | 每次编辑和命令操作均需审查批准 |
| **自动接受编辑(Auto accept edits)** | 文件编辑自动批准；命令操作仍需手动批准 |
| **计划模式(Plan mode)** | 在任何更改前先审查实施方案 |
| **跳过权限(Bypass permissions)** | 自动执行（仅限沙盒模式，由管理员控制） |

### 企业级功能

- **管理员控制台**：为组织整体控制 Code 选项卡的访问及权限设置
- **MDM 部署**：通过 macOS 的 MDM 或 Windows 的 MSIX 进行部署
- **SSO 集成**：要求组织成员通过单点登录认证
- **托管设置**：集中管理团队配置及模型可用性

---

## 任务列表

任务列表功能提供持久化的任务跟踪，即使在上下文压缩（为适配上下文窗口而裁剪对话历史）后仍能保留。

### 切换任务列表

在会话中按 `Ctrl+T` 即可打开或关闭任务列表视图。

### 持久化任务

任务在上下文压缩后依然保留，确保在对话上下文被裁剪时，耗时较长的工作项不会丢失。
这对于复杂的多步骤实现尤其实用。

### 命名任务目录

使用 `CLAUDE_CODE_TASK_LIST_ID` 环境变量可创建跨会话共享的命名任务目录：

```bash
export CLAUDE_CODE_TASK_LIST_ID=my-project-sprint-3
```

这样多个会话就能共享同一个任务列表，非常适用于团队协作或多会话项目。

---

## 提示建议

提示建议会根据 Git 历史记录和当前对话上下文，以灰色示例命令的形式显示。

### 工作原理

- 建议内容以灰色文本显示在输入提示词下方
- 按 `Tab` 键接受建议
- 按 `Enter` 键接受并直接提交
- 建议具备上下文感知能力，会结合 Git 历史记录和对话状态生成

### 禁用提示建议

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

---

## Git 工作树

Git 工作树允许你在一个隔离的工作树中启动 Claude Code，从而能够在不同分支上并行工作，无需暂存或切换分支。

### 在工作树中启动

```bash
# Start Claude Code in an isolated worktree
claude --worktree
# or
claude -w
```

### 工作树位置

工作树创建在以下路径：
```
<repo>/.claude/worktrees/<name>
```

### 针对单一代码仓库的稀疏检出

使用 `worktree.sparsePaths` 设置在单一代码仓库（monorepo）中执行稀疏检出，可以减少磁盘占用和克隆时间：

```json
{
  "worktree": {
    "sparsePaths": ["packages/my-package", "shared/"]
  }
}
```

### 工作树工具与钩子

| 项目 | 描述 |
|------|-------------|
| `ExitWorktree` | 用于退出并清理当前工作树的工具 |
| `WorktreeCreate` | 工作树创建时触发的钩子事件 |
| `WorktreeRemove` | 工作树移除时触发的钩子事件 |

### 自动清理

若工作树中未进行任何更改，会话结束时将自动清理。

### 使用场景

- 在功能分支上工作的同时，保持主分支不被更改
- 在隔离环境中运行测试，避免影响工作目录
- 在可任意处置的环境中尝试实验性更改
- 在单一代码仓库中稀疏检出特定包，加快启动速度

---

## 沙盒

沙盒为 Claude Code 执行的 Bash 命令提供操作系统级别的文件系统和网络隔离。
这是对权限规则的补充，提供了额外的安全层。

### 启用沙盒

**斜杠命令**：
```
/sandbox
```

**CLI 标志**：
```bash
claude --sandbox       # Enable sandboxing
claude --no-sandbox    # Disable sandboxing
```

### 配置设置

| 设置项 | 描述 |
|---------|-------------|
| `sandbox.enabled` | 启用或禁用沙盒 |
| `sandbox.failIfUnavailable` | 若沙盒无法激活则报错 |
| `sandbox.filesystem.allowWrite` | 允许写入的文件路径 |
| `sandbox.filesystem.allowRead` | 允许读取的文件路径 |
| `sandbox.filesystem.denyRead` | 拒绝读取的文件路径 |
| `sandbox.enableWeakerNetworkIsolation` | 在 macOS 上启用较弱的网络隔离 |

### 配置示例

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "filesystem": {
      "allowWrite": ["/Users/me/project"],
      "allowRead": ["/Users/me/project", "/usr/local/lib"],
      "denyRead": ["/Users/me/.ssh", "/Users/me/.aws"]
    },
    "enableWeakerNetworkIsolation": true
  }
}
```

### 工作原理

- Bash 命令在文件系统访问受限的沙盒环境中运行
- 网络访问可以被隔离，以防止意外的外部连接
- 与权限规则协同作用，构建纵深防御
- 在 macOS 上，使用 `sandbox.enableWeakerNetworkIsolation` 实现网络限制（macOS 不支持完全的网络隔离）

### 使用场景

- 安全地运行不受信任的或自动生成的代码
- 防止意外修改项目外部文件
- 在自动化任务执行过程中限制网络访问

---

## 托管设置（企业版）

托管设置使企业管理员能够使用平台原生的管理工具，在整个组织内部署 Claude Code 的配置。

### 部署方式

| 平台 | 方式 | 最低版本要求 |
|----------|--------|-------|
| macOS | 托管 plist 文件（MDM） | v2.1.51+ |
| Windows | Windows 注册表 | v2.1.51+ |
| 跨平台 | 托管配置文件 | v2.1.51+ |
| 跨平台 | 托管配置附加文件（`managed-settings.d/` 目录） | v2.1.83+ |

### 托管配置附加文件

自 v2.1.83 版本起，管理员可将多个托管设置文件部署到 `managed-settings.d/` 目录中。
这些文件按字母顺序合并，使得各团队能够模块化地进行配置管理：

```
~/.claude/managed-settings.d/
  00-org-defaults.json
  10-team-policies.json
  20-project-overrides.json
```

### 可用的托管设置项

| 设置项 | 描述 |
|---------|-------------|
| `disableBypassPermissionsMode` | 禁止用户启用跳过权限模式 |
| `availableModels` | 限制用户可选择的模型范围 |
| `allowedChannelPlugins` | 控制允许使用哪些频道插件 |
| `autoMode.environment` | 为自动模式配置受信基础设施 |
| 自定义策略 | 组织特定的权限及工具策略 |

### 示例：macOS Plist 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>disableBypassPermissionsMode</key>
  <true/>
  <key>availableModels</key>
  <array>
    <string>claude-sonnet-4-6</string>
    <string>claude-haiku-4-5</string>
  </array>
</dict>
</plist>
```

---

## 配置与设置

### 配置文件位置

1. **全局配置**：`~/.claude/config.json`
2. **项目配置**：`./.claude/config.json`
3. **用户配置**：`~/.config/claude-code/settings.json`

### 完整配置示例

**核心高级功能配置：**

```json
{
  "permissions": {
    "mode": "default"
  },
  "hooks": {
    "PreToolUse:Edit": "eslint --fix ${file_path}",
    "PostToolUse:Write": "~/.claude/hooks/security-scan.sh"
  },
  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"]
      }
    }
  }
}
```

**扩展配置示例：**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Bash(git log:*)", "Read"],
    "disallowedTools": ["Bash(rm -rf:*)"]
  },

  "hooks": {
    "PreToolUse": [{ "matcher": "Edit", "hooks": ["eslint --fix ${file_path}"] }],
    "PostToolUse": [{ "matcher": "Write", "hooks": ["~/.claude/hooks/security-scan.sh"] }],
    "Stop": [{ "hooks": ["~/.claude/hooks/notify.sh"] }]
  },

  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    }
  }
}
```

### 环境变量

通过环境变量覆盖配置：

```bash
# Model selection
export ANTHROPIC_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5

# API configuration
export ANTHROPIC_API_KEY=sk-ant-...

# Thinking configuration
export MAX_THINKING_TOKENS=16000
export CLAUDE_CODE_EFFORT_LEVEL=high

# Feature toggles
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=true
export CLAUDE_CODE_DISABLE_CRON=1
export CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS=true
export CLAUDE_CODE_DISABLE_TERMINAL_TITLE=true
export CLAUDE_CODE_DISABLE_1M_CONTEXT=true
export CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK=true
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
export CLAUDE_CODE_ENABLE_TASKS=true
export CLAUDE_CODE_SIMPLE=true              # Set by --bare flag

# MCP configuration
export MAX_MCP_OUTPUT_TOKENS=50000
export ENABLE_TOOL_SEARCH=true

# Task management
export CLAUDE_CODE_TASK_LIST_ID=my-project-tasks

# Agent teams (experimental)
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Subagent and plugin configuration
export CLAUDE_CODE_SUBAGENT_MODEL=sonnet
export CLAUDE_CODE_PLUGIN_SEED_DIR=./my-plugins
export CLAUDE_CODE_NEW_INIT=1

# Subprocess and streaming
export CLAUDE_CODE_SUBPROCESS_ENV_SCRUB="SECRET_KEY,DB_PASSWORD"
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=80
export CLAUDE_STREAM_IDLE_TIMEOUT_MS=30000
export ANTHROPIC_CUSTOM_MODEL_OPTION=my-custom-model
export SLASH_COMMAND_TOOL_CHAR_BUDGET=50000
```

### 配置管理命令

```
User: /config
[Opens interactive configuration menu]
```

通过 `/config` 命令提供一个交互式菜单来切换设置，例如：
- 扩展思维的开/关
- 详细输出
- 权限模式
- 模型选择

### 按项目进行配置

在项目中创建 `.claude/config.json` 文件：

```json
{
  "hooks": {
    "PreToolUse": [{ "matcher": "Bash", "hooks": ["npm test && npm run lint"] }]
  },
  "permissions": {
    "mode": "default"
  },
  "mcp": {
    "servers": {
      "project-db": {
        "command": "mcp-postgres",
        "env": {
          "DATABASE_URL": "${PROJECT_DB_URL}"
        }
      }
    }
  }
}
```

---

## 智能体团队(AgentTeams)

Agent 团队是一项实验性功能，允许多个 Claude Code 实例协作完成一项任务。
该功能默认处于禁用状态。

### 启用Agent团队

通过环境变量或设置启用：

```bash
# Environment variable
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

或者添加到你的设置 JSON 中：

```json
{
  "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
}
```

### 工作方式

- **团队负责人**协调整体任务并将子任务委派给团队成员
- **团队成员**独立工作，各自拥有独立的上下文窗口
- **共享任务列表**使团队成员之间能够进行自我协调
- 使用子代理定义（`.claude/agents/` 或 `--agents` 标志）来设定团队成员的角色与专长

### 显示模式

Agent团队支持两种显示模式，可通过 `--teammate-mode` 标志进行配置：

| 模式 | 描述 |
|------|-------------|
| `in-process`（默认） | 团队成员在同一终端进程内运行 |
| `tmux` | 每个成员拥有一个专用的分屏窗格（需要 tmux 或 iTerm2） |
| `auto` | 自动选择最佳的显示模式 |

```bash
# Use tmux split panes for teammate display
claude --teammate-mode tmux

# Explicitly use in-process mode
claude --teammate-mode in-process
```

### 使用场景

- 大型重构任务，由不同团队成员处理不同模块
- 并行的代码审查与实现
- 跨代码库的协调多文件更改

> **注意**：Agent团队是实验性功能，可能在未来的版本中发生变化。完整参考请查阅 [code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams)

---

## 最佳实践

### 计划模式
- ✅ 适用于复杂的多步骤任务
- ✅ 在批准之前审查计划
- ✅ 需要时调整计划
- ❌ 不要用于简单任务

### 扩展思维
- ✅ 适用于架构决策
- ✅ 适用于解决复杂问题
- ✅ 审查思考过程
- ❌ 不要用于简单查询

### 后台任务
- ✅ 适用于长时间运行的操作
- ✅ 监控任务进度
- ✅ 妥善处理任务失败
- ❌ 不要同时启动过多并发任务

### 权限管理
- ✅ 使用 `plan` 模式进行代码审查（只读）
- ✅ 使用 `default` 模式进行交互式开发
- ✅ 使用 `acceptEdits` 模式处理自动化工作流
- ✅ 使用 `auto` 模式在安全护栏内自主工作
- ❌ 非绝对必要，不要使用 `bypassPermissions` 模式

### 会话管理
- ✅ 为不同任务使用独立的会话
- ✅ 保存重要的会话状态
- ✅ 清理旧会话
- ❌ 不要在同一个会话中混合处理不相关的工作

---

## 其他资源

如需了解有关 Claude Code 及相关功能的更多信息：

- [Official Interactive Mode Documentation](https://code.claude.com/docs/en/interactive-mode)
- [Official Headless Mode Documentation](https://code.claude.com/docs/en/headless)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Checkpoints Guide](../08-checkpoints/) - Session management and rewinding
- [Slash Commands](../01-slash-commands/) - Command reference
- [Memory Guide](../02-memory/) - Persistent context
- [Skills Guide](../03-skills/) - Autonomous capabilities
- [Subagents Guide](../04-subagents/) - Delegated task execution
- [MCP Guide](../05-mcp/) - External data access
- [Hooks Guide](../06-hooks/) - Event-driven automation
- [Plugins Guide](../07-plugins/) - Bundled extensions
- [Official Scheduled Tasks Documentation](https://code.claude.com/docs/en/scheduled-tasks)
- [Official Chrome Integration Documentation](https://code.claude.com/docs/en/chrome)
- [Official Remote Control Documentation](https://code.claude.com/docs/en/remote-control)
- [Official Keybindings Documentation](https://code.claude.com/docs/en/keybindings)
- [Official Desktop App Documentation](https://code.claude.com/docs/en/desktop)
- [Official Agent Teams Documentation](https://code.claude.com/docs/en/agent-teams)

---
**Last Updated**: April 2026
**Claude Code Version**: 2.1+
**Compatible Models**: Claude Sonnet 4.6, Claude Opus 4.6, Claude Haiku 4.5
