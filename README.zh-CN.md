# c2c-lite

让 Claude 把写代码的活儿派给另一个 Claude——不需要 Codex，不需要 MCP，装一个文件就能用。

*[English](./README.md)*

## 项目简介

你用过 [claude2codex](https://github.com/PrestoOverture/c2c) 吗？那个项目让 Claude 当架构师，把实现工作通过 MCP 服务器派给 Codex 去做。效果挺好，但你得有 Codex 订阅，还得跑一个 MCP 服务器。

**c2c-lite** 把这个模式简化到了极致：不需要 Codex，不需要 MCP，Claude 直接派活给自己的一个"分身"（子代理，默认用 Sonnet）。合同格式一模一样，审查流程一模一样，但**零配置**。

什么时候用它：
- Codex 欠费了 / API 挂了 / 被限流了
- 任务不大，不值得折腾外部工具
- 就想用合同工作流，但懒得装任何东西

## 如何运作

说白了就是一个 Claude 扮演两个角色：

```
你
  → Opus（老板）—— 写需求合同，审查代码
    → Sonnet（打工人）—— 写代码，跑测试
  ← Opus —— 告诉你结果怎么样
← 你 —— 拍板通过，或者让它返工
```

没有外部进程。Claude Code 里内置的 `Agent` 工具就够了。

## 安装

一行命令：

```sh
mkdir -p ~/.claude/skills && curl -fsSL https://raw.githubusercontent.com/PrestoOverture/c2c-lite/main/c2c-lite.skill.md -o ~/.claude/skills/c2c-lite.md
```

就这样。整个东西就是一个 Markdown 文件。

## 怎么用？

在 Claude Code 里输入 `/c2c-lite`，然后说你想做什么。比如"给我加一个 /health 接口"。

Claude 会：

1. **写一份 Goal Contract**（目标、约束、验收条件）给你看
2. 你说"行"，它就 **spawn 一个子代理**去写代码
3. 子代理写完了，Claude **自己跑一遍测试**验证
4. 把结果 **报给你**——过了就过了，没过就告诉你哪里有问题

没过的话？Claude 写一份 Delta Contract（只说哪里不对），发给**同一个子代理**让它改。上下文保留着，不用从头来。

## 选模型

默认让 Sonnet 干活。批准合同的时候你可以换：

- **Sonnet**（默认）—— 又快又便宜，大多数活儿够用
- **Opus** —— 推理更强，逻辑复杂的活儿用这个
- **Haiku** —— 最快最便宜，简单改动用这个

## 和 claude2codex 有什么区别？

| | c2c-lite | claude2codex |
|---|---|---|
| 干活的是谁 | Claude 子代理 | OpenAI Codex |
| 要装什么 | 复制一个文件 | Codex CLI + MCP 服务器 |
| 配置 | 零 | `npx claude2codex` + MCP 配置 |
| 执行方式 | 单次（Agent 工具） | Codex goal 循环（自动迭代直到搞定） |
| 任务持久化 | 没有（会话结束就没了） | 有（写到磁盘，重启不丢） |
| 成本预估 | 没有 | 有（`codex_estimate`） |
| 并行任务 | 能（多个 Agent 同时跑） | 能（`C2C_MAX_CONCURRENT`） |
| 返工 | 给同一个子代理发消息 | `codex_rework` 恢复线程 |

简单说：**c2c-lite 是轻量版，开箱即用；claude2codex 是完整版，功能更多但要配置。**

## 协议

MIT
