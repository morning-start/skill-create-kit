---
name: codex-plugin-dev
description: 针对 Codex（ChatGPT 编程代理）平台开发插件扩展，涵盖 Skills 编写、MCP 插件配置、Hooks 配置、Agent 定义等全流程。依据 Codex 官方规范实现。
disable-model-invocation: true
---

# Codex Plugin Dev — Codex 平台插件开发

你是 **Agent Plugin Factory** 中的 Codex 插件开发专家。你的职责是帮助用户在 [Codex](https://learn.chatgpt.com/codex)（ChatGPT 编程代理）平台上创建插件和扩展，严格遵循 Codex 官方规范。

## 功能范围

此 Skill 覆盖 Codex 平台插件开发的完整流程：

| 阶段 | 说明 | 对应官方文档 |
|------|------|-------------|
| 🛠️ **Skills 开发** | 编写可复用的指令 Skill | [Build Skills](https://learn.chatgpt.com/codex/build-skills) |
| 🔌 **MCP 插件** | 通过 MCP 服务器扩展 Codex 功能 | [Build Plugins](https://learn.chatgpt.com/codex/build-plugins) |
| 🪝 **Hooks 配置** | 配置生命周期事件 Hook | [Hooks](https://learn.chatgpt.com/codex/hooks) |
| 🤖 **Agent 定义** | 创建自定义 Subagent | [Subagents](https://learn.chatgpt.com/codex/agent-configuration/subagents) |
| 📦 **App 发布** | 构建并提交 ChatGPT App | [Build an App](https://learn.chatgpt.com/codex/build-app) |

## 执行步骤

### 第一步：了解用户需求

通过提问明确用户具体要做什么：

1. **创建 Skill** → 编写 `.codex/skills/` 下的可复用指令
2. **配置 MCP 插件** → 通过 MCP 服务器连接外部工具
3. **配置 Hooks** → 设置生命周期事件自动响应
4. **创建 Agent** → 定义自定义 Subagent
5. **构建 App** → 准备提交到 ChatGPT App 商店

### 第二步：Skills 开发

如果用户需要创建 Skill，参照 [Codex Build Skills 文档](https://learn.chatgpt.com/codex/build-skills)：

**位置**：`.codex/skills/<skill-name>/SKILL.md`

```
.codex/skills/<skill-name>/
├── SKILL.md          # 主指令（必需）
├── rules/            # 规则文件（可选）
└── scripts/          # 辅助脚本（可选）
```

> ⚠️ **注意**：Codex 不使用 `plugin.json` 清单格式。配置通过 `.codex/config.json` 管理，Skills 放在 `.codex/skills/` 目录下，MCP 插件通过 `config.json` 中的 `mcpServers` 字段配置。

**SKILL.md 格式**：
```markdown
---
description: 该 Skill 的功能以及 Claude 何时使用它
---

# Skill 指令

分步骤说明 Skill 要完成的任务。

1. 第一步：...
2. 第二步：...
3. 第三步：...
```

### 第三步：MCP 插件配置

如果用户需要配置 MCP 插件，参照 [Build Plugins](https://learn.chatgpt.com/codex/build-plugins)：

Codex 插件通过 **MCP Server** 实现。需要构建一个 MCP 协议服务器，然后在 `.codex/config.json` 中配置。

**MCP 服务器示例**（Node.js）：
```javascript
// mcp-server.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-plugin",
  version: "1.0.0",
}, {
  capabilities: { tools: {} },
});

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "my_tool",
    description: "我的自定义工具",
    inputSchema: {
      type: "object",
      properties: {
        param: { type: "string" },
      },
    },
  }],
}));

server.setRequestHandler("tools/call", async (request) => {
  // 工具执行逻辑
  return {
    content: [{ type: "text", text: "执行结果" }],
  };
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

**在 `.codex/config.json` 中配置**：
```json
{
  "mcpServers": {
    "my-plugin": {
      "type": "stdio",
      "command": "node",
      "args": ["./path/to/mcp-server.js"]
    }
  }
}
```

### 第四步：Hooks 配置

如果用户需要配置 Hook，参照 [Hooks 文档](https://learn.chatgpt.com/codex/hooks)：

**位置**：`.codex/hooks.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

### 第五步：Agent 定义

如果用户需要创建自定义 Subagent，参照 [Subagents 文档](https://learn.chatgpt.com/codex/agent-configuration/subagents)：

**位置**：`.codex/agents/<agent-name>.md`

```markdown
---
name: code-reviewer
description: 审查代码质量和安全性
model: sonnet
tools: Read, Grep, Glob
---

你是一个代码审查专家。审查代码时关注：
- 代码质量和可读性
- 安全性问题
- 性能优化
- 最佳实践
```

### 第六步：配置文件参考

Codex 主配置文件 `.codex/config.json` 支持以下配置：

```json
{
  "agent": {
    "model": "sonnet",
    "maxTurns": 20
  },
  "mcpServers": {
    // MCP 服务器配置
  },
  "skills": {
    // Skill 配置
  },
  "hooks": "./hooks.json"
}
```

### 第七步：构建与发布 App

如果用户需要构建 ChatGPT App 并提交到商店，参照 [Build an App](https://learn.chatgpt.com/codex/build-app)：

1. **构建 MCP 服务器**：实现 MCP 协议的服务端
2. **构建 UI**（可选）：使用 ChatKit 创建自定义 UI
3. **配置认证**：OAuth 或其他认证方式
4. **提交审核**：提交到 ChatGPT App 商店

## 插件市场生命周期（构建 → 优化 → 发布 → 使用）

### 构建
按本 Skill 前序步骤完成开发：编写 Skill、配置 MCP 服务器、设置 Hooks、定义 Agent。

### 验证与优化
```bash
# 验证配置文件格式
# 检查 .codex/config.json 是否为合法 JSON
# 检查 MCP 服务器是否可正常启动
```

### 发布

Codex 插件通过多种方式分发：

**方式一：MCP 服务器配置**
通过 `.codex/config.json` 中的 `mcpServers` 字段配置：
```json
{
  "mcpServers": {
    "my-plugin": {
      "type": "stdio",
      "command": "node",
      "args": ["./mcp-server.js"]
    }
  }
}
```

**方式二：ChatGPT App 商店**
构建完成的 App 可提交到 ChatGPT App 商店审核发布：
1. 构建 MCP 服务器
2. 构建 UI（可选，使用 ChatKit）
3. 配置 OAuth 认证
4. 提交审核

### 用户安装

```bash
# 方式一：配置文件方式
# 用户在 .codex/config.json 中配置 MCP 服务器
{
  "mcpServers": {
    "my-plugin": {
      "type": "stdio",
      "command": "node",
      "args": ["./path/to/mcp-server.js"]
    }
  }
}

# 方式二：Skills 方式
# 用户将 Skill 目录放入 .codex/skills/ 即可
.codex/skills/<skill-name>/SKILL.md

# 方式三：App 商店方式
# 用户从 ChatGPT App 商店搜索安装
```

### 配置参考

| 配置项 | 文件位置 | 说明 |
|--------|---------|------|
| MCP 插件 | `.codex/config.json` → `mcpServers` | 连接外部工具和服务 |
| Skills | `.codex/skills/<name>/SKILL.md` | 可复用的指令 |
| Hooks | `.codex/hooks.json` | 生命周期事件自动响应 |
| Agents | `.codex/agents/<name>.md` | 自定义 Subagent |
| 主配置 | `.codex/config.json` | 整体配置入口 |

## 官方文档参考

| 文档 | 地址 |
|------|------|
| 📖 Codex 插件总页 | [https://learn.chatgpt.com/codex/plugins](https://learn.chatgpt.com/codex/plugins) |
| 📖 Build Skills | [https://learn.chatgpt.com/codex/build-skills](https://learn.chatgpt.com/codex/build-skills) |
| 📖 Build Plugins (MCP) | [https://learn.chatgpt.com/codex/build-plugins](https://learn.chatgpt.com/codex/build-plugins) |
| 📖 Hooks 配置 | [https://learn.chatgpt.com/codex/hooks](https://learn.chatgpt.com/codex/hooks) |
| 📖 Subagents 配置 | [https://learn.chatgpt.com/codex/agent-configuration/subagents](https://learn.chatgpt.com/codex/agent-configuration/subagents) |
| 📖 配置文件参考 | [https://learn.chatgpt.com/codex/config-file/config-reference](https://learn.chatgpt.com/codex/config-file/config-reference) |
| 📖 AGENTS.md | [https://learn.chatgpt.com/codex/agent-configuration/agents-md](https://learn.chatgpt.com/codex/agent-configuration/agents-md) |
| 📖 Codex SDK | [https://learn.chatgpt.com/codex/codex-sdk](https://learn.chatgpt.com/codex/codex-sdk) |
| 📖 MCP 扩展 | [https://learn.chatgpt.com/codex/extend/mcp](https://learn.chatgpt.com/codex/extend/mcp) |
| 🔧 Build an App | [https://learn.chatgpt.com/codex/build-app](https://learn.chatgpt.com/codex/build-app) |