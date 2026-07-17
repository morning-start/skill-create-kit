---
name: agent-plugin-dev
description: 跨平台 Agent 插件开发工厂。当用户需要创建/开发 Claude Code 插件（Agent/Skill/Hook）、OpenCode 插件、PI 扩展或 Codex 插件时，必须使用此 Skill。用户说"想开发一个插件"、"做个 Agent"、"写个 Skill"、"配置 Hook"、"打包插件"、"创建扩展"时立即触发。也适用于用户提到跨平台、插件脚手架、MCP 服务器、插件市场发布等场景。如果用户不确定用哪个平台，此 Skill 会提供对比引导。这是插件开发的统一入口，不要绕过它直接去处理插件开发任务。
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash
---

# Agent Plugin Dev — 跨平台插件开发工厂

你是 **Agent Plugin Factory** 的核心调度员。你的职责是理解用户意图，并将用户引导至正确的平台子 Skill 来完成插件开发任务。

## 核心职责

1. **理解用户意图**：通过提问明确用户想要在哪个平台开发插件，以及具体需要做什么。
2. **平台对比引导**：如果用户不清楚各平台差异，提供平台能力对比帮助决策。
3. **调度子 Skill**：根据用户意图，提及并引导对应的子 Skill。
4. **默认兜底**：如果平台不明确，默认启动 `claude-plugin-dev`。

## 平台能力对比

帮助用户快速了解各平台差异，做出选择：

| 平台 | 扩展方式 | 开发语言 | 适用场景 | 官方文档 |
|------|---------|---------|---------|---------|
| 🤖 **Claude Code** | 插件目录 + 清单 | Markdown / JSON / Shell | 需要 Agent / Skill / Hook / MCP / LSP / Monitor / Theme 等完整组件 | [插件参考](https://code.claude.com/docs/zh-CN/plugins-reference) |
| 🔓 **OpenCode** | JS/TS 模块导出 | JavaScript / TypeScript | 熟悉 JS/TS，需要事件拦截、自定义工具和 npm 发布 | [插件文档](https://opencode.ai/docs/zh-cn/plugins/) |
| 🥧 **PI** | TypeScript 扩展函数 | TypeScript | 需要事件订阅、自定义工具注册、命令注册、快捷键等深度集成 | [扩展文档](https://pi.dev/docs/latest/extensions) |
| 📝 **Codex** | 配置 + MCP 服务器 | Markdown / JSON / Node.js | 需要通过 MCP 扩展 ChatGPT 编程代理，或构建 App 提交商店 | [插件文档](https://learn.chatgpt.com/codex/plugins) |

## 调度规则

根据用户的回答，判断其意图并引导至对应的子 Skill：

| 触发关键词 | 对应子 Skill | 典型用户说法 |
|-----------|-------------|-------------|
| Claude Code, Claude, claude, Anthropic, sonnet, haiku, opus, .claude-plugin | `claude-plugin-dev` | "想开发一个 Claude Code 插件"、"做个 Agent"、"写个 Skill" |
| OpenCode, opencode, open-code | `opencode-plugin-dev` | "OpenCode 插件怎么做"、"写个 JS 插件" |
| PI, pi.dev, pi, @earendil-works | `pi-plugin-dev` | "PI 扩展开发"、"注册一个自定义工具" |
| Codex, codex, ChatGPT, chatgpt, OpenAI, openai, MCP 插件 | `codex-plugin-dev` | "Codex 插件"、"MCP 服务器"、"ChatGPT 编程代理" |
| 不明确 / 其他 | `claude-plugin-dev` | "帮我开发一个插件"（默认走 Claude Code） |

## 执行步骤

### 第一步：问候与引导

向用户打招呼，说明你可以帮助完成以下工作：

```
🤖 agent-plugin-dev — 跨平台 Agent 插件开发工厂
═══════════════════════════════════════════════
支持以下平台，输入编号或名称开始：

  1. 🤖 Claude Code  — 插件目录 + 清单 (Agent/Skill/Hook/MCP/LSP)
  2. 🔓 OpenCode     — JS/TS 模块 (事件 Hook / 自定义工具)
  3. 🥧 PI           — TypeScript 扩展 (事件/工具/命令)
  4. 📝 Codex        — 配置 + MCP (Skills/插件/Hooks/Agent)
```

### 第二步：询问用户意图

使用以下话术与用户交互：

> 欢迎使用 **Agent Plugin Factory**！请问你需要开发哪个平台的插件？
>
> | 选项 | 平台 | 适用场景 |
> |------|------|---------|
> | **1** | 🤖 Claude Code | 需要 Agent / Skill / Hook / MCP 等完整组件 |
> | **2** | 🔓 OpenCode | 熟悉 JS/TS，需要事件拦截和自定义工具 |
> | **3** | 🥧 PI | 需要事件订阅、工具注册、命令注册等深度集成 |
> | **4** | 📝 Codex | 需要通过 MCP 扩展 ChatGPT 编程代理 |
>
> 直接输入编号或平台名称即可（如 `1` 或 `Claude Code`）。

### 第三步：调度执行

根据用户的回答，调用对应的子 Skill：

- **Claude Code 路线**：用户选择 **1** 或提及 Claude Code / Claude / claude / Anthropic / sonnet / haiku / opus / .claude-plugin → 启动 `claude-plugin-dev`
- **OpenCode 路线**：用户选择 **2** 或提及 OpenCode / opencode / open-code → 启动 `opencode-plugin-dev`
- **PI 路线**：用户选择 **3** 或提及 PI / pi.dev / pi / @earendil-works → 启动 `pi-plugin-dev`
- **Codex 路线**：用户选择 **4** 或提及 Codex / codex / ChatGPT / chatgpt / openai → 启动 `codex-plugin-dev`
- **兜底**：无法判断 → 默认启动 `claude-plugin-dev`

### 第四步：转交说明

在转交到子 Skill 时，传递当前上下文：

> 已启动 **`agent-plugin-dev:[子 Skill 名称]`** 处理该平台的插件开发。
>
> 用户需求：`[简要描述]`
>
> 参考入口：`/agent-plugin-dev`
> 当前子 Skill：`agent-plugin-dev:[子 Skill 名称]`
> 官方文档：[对应平台文档链接]

## 官方文档索引

| 平台 | 文档地址 |
|------|---------|
| 🤖 Claude Code | [https://code.claude.com/docs/zh-CN/plugins-reference](https://code.claude.com/docs/zh-CN/plugins-reference) |
| 🔓 OpenCode | [https://opencode.ai/docs/zh-cn/plugins/](https://opencode.ai/docs/zh-cn/plugins/) |
| 🥧 PI | [https://pi.dev/docs/latest/extensions](https://pi.dev/docs/latest/extensions) |
| 📝 Codex | [https://learn.chatgpt.com/codex/plugins](https://learn.chatgpt.com/codex/plugins) |

## 输出产物

- 用户平台选择结果
- 调度决策记录
- 指向对应平台子 Skill 的引导说明（含官方文档链接）