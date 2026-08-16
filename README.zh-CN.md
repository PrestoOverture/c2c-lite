# c2c-lite

一个 Claude Code Skill，让你通过结构化 Goal Contract 将实现工作委派给 Claude 子代理——与 [claude2codex](https://github.com/PrestoOverture/c2c) 相同的架构师/实现者工作流，但不需要 Codex 或任何外部依赖。

## 为什么做这个

[claude2codex](https://github.com/PrestoOverture/c2c) 通过 MCP 服务器连接 Claude（架构师）和 Codex（实现者）。它运作良好，但需要 Codex 订阅和运行中的 MCP 服务器。

**c2c-lite** 将依赖降到零：Claude 使用 Claude Code 内置的 `Agent` 工具将任务委派给 Claude 子代理（默认 Sonnet）。相同的契约格式，相同的审查纪律，无需基础设施。

适用场景：
- Codex 不可用（没有订阅、API 宕机、限流）
- 任务不值得启动外部进程
- 想用契约工作流但不想做任何配置

## 工作原理

```
你（用户）
  → Claude Opus（架构师）— 起草 Goal Contract
    → Claude Sonnet（子代理/实现者）— 编写代码、运行测试
  ← Claude Opus — 审查产出、报告发现
← 你 — 批准或要求返工
```

没有 MCP 服务器。没有外部进程。只是 Claude 通过内置的 `Agent` 工具与 Claude 对话。

## 安装

```sh
mkdir -p ~/.claude/skills
curl -fsSL https://raw.githubusercontent.com/PrestoOverture/c2c-lite/main/c2c-lite.skill.md -o ~/.claude/skills/c2c-lite.md
```

或者手动复制——整个东西就是一个 Markdown 文件。

## 使用

在任何 Claude Code 会话中，输入 `/c2c-lite` 并描述你想做什么。Claude 会：

1. 起草 Goal Contract 并展示给你
2. 你批准后，生成一个子代理来实现
3. 独立审查子代理的工作
4. 报告发现供你批准

如果审查失败，Claude 向同一个子代理发送 Delta Contract（保留上下文）进行针对性返工。

### 模型选择

默认实现者模型是 Sonnet。你可以在批准时指定不同的模型：

- **Sonnet**（默认）— 快速、性价比高，适合大多数任务
- **Opus** — 更强的推理能力，适合复杂或模糊的任务
- **Haiku** — 最快、最便宜，适合简单/机械性的改动

## 与 claude2codex 的对比

| | c2c-lite | claude2codex |
|---|---|---|
| 实现者 | Claude 子代理 | OpenAI Codex |
| 依赖 | 无 | Codex CLI + MCP 服务器 |
| 配置 | 复制一个文件 | `npx claude2codex` + MCP 配置 |
| Goal loop | 单次执行（Agent 工具） | Codex `/goal` 循环（迭代直到完成） |
| 持久化 | 无（仅会话内） | 任务持久化到磁盘 |
| 成本预估 | 不可用 | `codex_estimate` 工具 |
| 并行任务 | 支持（多个 Agent 调用） | 支持（`C2C_MAX_CONCURRENT`） |
| 返工 | `SendMessage` 到同一代理 | `codex_rework` 恢复线程 |

## 协议

MIT
