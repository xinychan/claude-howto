<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 检查点与回退(Checkpoints/Rewind)

检查点允许你保存对话状态，并在 Claude Code 会话中回退到之前的某个节点。
这对于探索不同方案、从错误中恢复或比较替代解决方案都极具价值。

## 概述

检查点允许你保存对话状态并回退到之前的节点，从而实现安全实验和多方探索。
它们是对话状态的快照，包括：
- 所有已交换的消息
- 已进行的文件修改
- 工具使用历史记录
- 会话上下文

在探索不同方案、从错误中恢复或比较替代解决方案时，检查点的作用不可估量。

## 核心概念

| 概念 | 说明 |
|---------|-------------|
| **检查点/Checkpoint** | 对话状态的快照，包括消息、文件和上下文 |
| **回退/Rewind** | 返回到之前的检查点，舍弃后续所做的更改 |
| **分支点/Branch Point** | 从中探索多种方案的检查点 |

## 访问检查点

你可以通过两种主要方式访问和管理检查点：

### 使用键盘快捷键

按两次 `Esc` 键（`Esc` + `Esc`）即可打开检查点界面并浏览已保存的检查点。

### 使用斜杠命令

使用 `/rewind` 命令（别名：`/checkpoint`）快速访问：

```bash
# Open rewind interface
/rewind

# Or use the alias
/checkpoint
```

## 回退选项

当你执行回退操作时，系统会呈现一个包含五个选项的菜单：

1. **恢复代码和对话/Restore code and conversation** -- 将文件和消息都回退到该检查点的状态
2. **仅恢复对话/Restore conversation** -- 只回退消息，保留当前代码不变
3. **仅恢复代码/Restore code** -- 只回退文件更改，保留完整的对话历史
4. **从此处开始总结/Summarize from here** -- 将从此节点往后的对话压缩为一段 AI 生成的摘要，以释放上下文窗口空间。所选节点之前的消息保持完整。磁盘上的文件不受任何影响。原始消息仍保留在会话转录中。你还可以选择性提供说明，以便将摘要聚焦于特定主题。
5. **取消/Never mind** -- 取消操作并返回当前状态

> **注意**：恢复对话或进行总结后，所选消息的原始提示词会恢复到输入框中，方便你重新发送或编辑。

## 自动检查点

Claude Code 会自动为你创建检查点：

- **每次用户提交提示词** -- 每次用户输入时都会生成新的检查点
- **持久化** -- 检查点可跨会话保留
- **自动清理** -- 超过 30 天的检查点会被自动清除

这意味着你随时可以回退到对话中的任意先前节点，无论是几分钟前还是数天前。

## 使用场景

| 场景 | 工作流程 |
|----------|----------|
| **方案探索** | 保存 → 尝试方案 A → 保存 → 回退 → 尝试方案 B → 对比 |
| **安全重构** | 保存 → 重构 → 测试 → 若失败：回退 |
| **A/B 测试** | 保存 → 设计方案 A → 保存 → 回退 → 设计方案 B → 对比 |
| **错误恢复** | 发现问题 → 回退到上一个正常状态 |

## 使用检查点

### 查看与回退

按两次 `Esc` 键或使用 `/rewind` 命令打开检查点浏览器。
你将看到一个包含时间戳的所有可用检查点列表。
选择任意检查点即可回退到该状态。

### 检查点详情

每个检查点会显示：
- 创建时间戳
- 已修改的文件
- 对话中的消息数量
- 已使用的工具

## 实用示例

### 示例 1：探索不同方案

```
User: Let's add a caching layer to the API

Claude: I'll add Redis caching to your API endpoints...
[Makes changes at checkpoint A]

User: Actually, let's try in-memory caching instead

Claude: I'll rewind to explore a different approach...
[User presses Esc+Esc and rewinds to checkpoint A]
[Implements in-memory caching at checkpoint B]

User: Now I can compare both approaches
```

### 示例 2：从错误中恢复

```
User: Refactor the authentication module to use JWT

Claude: I'll refactor the authentication module...
[Makes extensive changes]

User: Wait, that broke the OAuth integration. Let's go back.

Claude: I'll help you rewind to before the refactoring...
[User presses Esc+Esc and selects the checkpoint before the refactor]

User: Let's try a more conservative approach this time
```

### 示例 3：安全实验

```
User: Let's try rewriting this in a functional style
[Creates checkpoint before experiment]

Claude: [Makes experimental changes]

User: The tests are failing. Let's rewind.
[User presses Esc+Esc and rewinds to the checkpoint]

Claude: I've rewound the changes. Let's try a different approach.
```

### 示例 4：分支式探索

```
User: I want to compare two database designs
[Takes note of checkpoint - call it "Start"]

Claude: I'll create the first design...
[Implements Schema A]

User: Now let me go back and try the second approach
[User presses Esc+Esc and rewinds to "Start"]

Claude: Now I'll implement Schema B...
[Implements Schema B]

User: Great! Now I have both schemas to choose from
```

## 检查点保留策略

Claude Code 会自动管理你的检查点：

- 每次用户提交提示词时都会自动生成检查点
- 旧的检查点最多保留 30 天
- 检查点会被自动清理，以防止存储空间无限增长

## 工作流模式

### 探索时的分支策略

在探索多种方案时：

```
1. Start with initial implementation → Checkpoint A
2. Try Approach 1 → Checkpoint B
3. Rewind to Checkpoint A
4. Try Approach 2 → Checkpoint C
5. Compare results from B and C
6. Choose best approach and continue
```

### 安全重构模式

在进行重大更改时：

```
1. Current state → Checkpoint (auto)
2. Start refactoring
3. Run tests
4. If tests pass → Continue working
5. If tests fail → Rewind and try different approach
```

## 最佳实践

由于检查点是自动创建的，你可以专注于手头工作，无需为手动保存状态而分心。
不过，仍建议留意以下实践：

### 有效使用检查点

✅ **建议事项：**
- 在执行回退前，先浏览可用的检查点
- 当你想探索不同方向时，使用回退功能
- 保留检查点以便对比不同方案
- 了解每个回退选项的具体作用（恢复代码和对话、仅恢复对话、仅恢复代码，或进行总结）

❌ **避免事项：**
- 不要仅依赖检查点来保存代码
- 不要期望检查点会追踪外部文件系统的更改
- 不要将检查点当作 Git 提交的替代品来使用

## 配置

检查点是 Claude Code 中默认内置的行为，无需任何配置即可启用。
每次用户提交提示词时均会自动创建检查点。

唯一与检查点相关的设置是 `cleanupPeriodDays`，用于控制会话和检查点的保留天数：

```json
{
  "cleanupPeriodDays": 30
}
```

- `cleanupPeriodDays`：保留会话历史记录和检查点的天数（默认值：`30`）

## 局限性

检查点存在以下局限性：

- **不会追踪 Bash 命令的更改** - 文件系统中的 `rm`、`mv`、`cp` 等操作不会被捕获记录在检查点中
- **不会追踪外部更改** - 在 Claude Code 之外进行的更改（如在编辑器、终端中的操作）不会被捕获
- **不能替代版本控制** - 请使用 Git 来对代码库进行永久的、可审计的更改

## 故障排查

### 检查点缺失

**问题**：找不到预期的检查点

**解决方法**：
- 检查是否已清除了检查点
- 检查磁盘空间
- 确保 `cleanupPeriodDays` 设置得足够长（默认为 30 天）

### 回退失败

**问题**：无法回退到检查点

**解决方法**：
- 确保没有未提交的更改导致冲突
- 检查该检查点是否已损坏
- 尝试回退到另一个检查点

## 与 Git 集成

检查点是 Git 的补充（而非替代）：

| 功能 | Git | Checkpoints/检查点 |
|---------|-----|-------------|
| 作用域 | 文件系统 | 对话 + 文件 |
| 持久性 | 永久 | 基于会话 |
| 粒度 | 提交点 | 任意节点 |
| 速度 | 较慢 | 即时 |
| 共享性 | 支持 | 有限 |

两者结合使用：
1. 用检查点进行快速实验
2. 用 Git 提交来保存定稿的更改
3. 在 Git 操作前创建检查点
4. 将成功的检查点状态提交至 Git

## 快速入门指南

### 基础工作流

1. **正常工作** - Claude Code 会自动创建检查点
2. **想要返回？** - 按两次 `Esc` 或使用 `/rewind`
3. **选择检查点** - 从列表中选择要回退到的节点
4. **选择恢复内容** - 选择恢复代码和对话、仅恢复对话、仅恢复代码、从此处总结或取消
5. **继续工作** - 你已经回到了那个节点

### 键盘快捷键

- **`Esc` + `Esc`** - 打开检查点浏览器
- **`/rewind`** - 访问检查点的替代方式
- **`/checkpoint`** - `/rewind` 的别名

## 如何判断回退时机：上下文监控

检查点让你能够回退，但你如何知道*何时*应该回退？
随着对话的不断增长，Claude 的上下文窗口会被逐渐填满，模型质量也会悄然下降。
你可能在不知不觉中，正从一个视野受限的模型那里获得代码。

**[cc-context-stats](https://github.com/luongnv89/cc-context-stats)** 通过在你的 Claude Code 状态栏中添加实时**上下文区域**来解决这个问题。它追踪你在上下文窗口中的位置，从 **Plan**（绿色，适合规划和编码）、**Code**（黄色，避免开始新规划）到 **Dump**（橙色，完成工作并回退）。当你看到区域发生变化时，你就知道是时候创建检查点并重新开始了，而不是在质量下降的输出上继续推进。

## 相关概念

- **[Advanced Features](../09-advanced-features/)** - 规划模式及其他高级能力
- **[Memory Management](../02-memory/)** - 管理对话历史与上下文
- **[Slash Commands](../01-slash-commands/)** - 用户调用的快捷操作
- **[Hooks](../06-hooks/)** - 事件驱动的自动化
- **[Plugins](../07-plugins/)** - 打包的扩展功能集

## 其他资源

- [Official Checkpointing Documentation](https://code.claude.com/docs/en/checkpointing)
- [Advanced Features Guide](../09-advanced-features/) - Extended thinking and other capabilities

## 总结

检查点是 Claude Code 中的一项自动化功能，让你能够安全地探索不同的方案，不必担心丢失工作进度。
每次用户提交提示词时都会自动创建一个新的检查点，因此你可以回退到会话中的任意先前节点。

核心优势：
- 放心大胆地尝试多种方案
- 快速从错误中恢复
- 并行比较不同的解决方案
- 与版本控制系统安全集成

请记住：检查点并非 Git 的替代品。
用检查点进行快速实验，用 Git 保存永久性的代码更改。

---
**Last Updated**: April 2026
**Claude Code Version**: 2.1+
**Compatible Models**: Claude Sonnet 4.6, Claude Opus 4.6, Claude Haiku 4.5
