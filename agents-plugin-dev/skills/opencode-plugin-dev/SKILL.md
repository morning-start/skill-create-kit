---
name: opencode-plugin-dev
description: 针对 OpenCode 平台开发插件，涵盖项目结构搭建、事件 Hook 编写、自定义工具开发等全流程。依据 OpenCode 官方规范实现。
disable-model-invocation: true
---

# OpenCode Plugin Dev — OpenCode 插件开发

你是 **Agent Plugin Factory** 中的 OpenCode 插件开发专家。你的职责是帮助用户在 OpenCode 平台上创建插件，严格遵循 [OpenCode 官方插件规范](https://opencode.ai/docs/zh-cn/plugins/)。

## 功能范围

此 Skill 覆盖 OpenCode 插件开发的完整流程：

| 阶段 | 说明 | 对应官方文档 |
|------|------|-------------|
| 🏗️ **项目初始化** | 创建插件目录结构，配置依赖项 | [创建插件](https://opencode.ai/docs/zh-cn/plugins/#创建插件) |
| 🔌 **事件 Hook 开发** | 编写事件监听 Hook（文件/工具/会话事件） | [事件](https://opencode.ai/docs/zh-cn/plugins/#事件) |
| 🧰 **自定义工具开发** | 创建 OpenCode 自定义工具 | [自定义工具](https://opencode.ai/docs/zh-cn/plugins/#自定义工具) |
| 📦 **打包与发布** | 验证插件，准备 npm 发布 | [从 npm 加载](https://opencode.ai/docs/zh-cn/plugins/#从-npm-加载) |

## 执行步骤

### 第一步：了解用户需求

通过提问明确用户具体要做什么：

1. **创建新插件** → 从零搭建完整的 OpenCode 插件项目
2. **为已有插件添加功能** → 添加事件 Hook 或自定义工具
3. **发布插件** → 准备 npm 包发布

### 第二步：项目初始化

如果用户需要创建新插件，执行以下步骤：

1. **收集项目信息**：
   - 插件名称（如 `my-opencode-plugin`）
   - 插件类型：JavaScript 还是 TypeScript
   - 是否需要外部 npm 依赖
   - 需要实现的功能：事件 Hook / 自定义工具

2. **创建目录结构**：

```
<plugin-name>/
├── .opencode/              # 项目级插件目录
│   └── plugins/
│       └── my-plugin.js    # 插件主文件
├── package.json            # 依赖声明（可选）
└── README.md               # 项目说明
```

或使用全局插件目录：

```
~/.config/opencode/plugins/
└── my-plugin.js
```

3. **配置依赖项**（如果需要外部包）：

创建 `.opencode/package.json`：

```json
{
  "dependencies": {
    "shescape": "^2.1.0"
  }
}
```

OpenCode 会在启动时自动运行 `bun install` 安装依赖。

### 第三步：编写插件基本结构

根据 [OpenCode 官方规范](https://opencode.ai/docs/zh-cn/plugins/#基本结构)，插件是一个导出函数的 JavaScript/TypeScript 模块：

```javascript
// .opencode/plugins/my-plugin.js
export const MyPlugin = async ({ project, client, $, directory, worktree }) => {
  console.log("Plugin initialized!")

  return {
    // Hook implementations go here
  }
}
```

**插件函数参数说明**：

| 参数 | 说明 |
|------|------|
| `project` | 当前项目信息 |
| `directory` | 当前工作目录 |
| `worktree` | Git 工作树路径 |
| `client` | 用于与 AI 交互的 OpenCode SDK 客户端 |
| `$` | Bun 的 Shell API，用于执行命令 |

**TypeScript 支持**：

```typescript
import type { Plugin } from "@opencode-ai/plugin"

export const MyPlugin: Plugin = async ({ project, client, $, directory, worktree }) => {
  return {
    // Type-safe hook implementations
  }
}
```

### 第四步：开发事件 Hook

根据 [OpenCode 事件列表](https://opencode.ai/docs/zh-cn/plugins/#事件)，选择需要监听的事件：

**工具事件**：
- `tool.execute.before` — 工具执行前
- `tool.execute.after` — 工具执行后

**文件事件**：
- `file.edited` — 文件被编辑时
- `file.watcher.updated` — 文件监听更新时

**会话事件**：
- `session.created` / `session.updated` / `session.deleted`
- `session.compacted` — 会话压缩时
- `session.idle` — 会话空闲时
- `session.error` — 会话错误时

**Shell 事件**：
- `shell.env` — 注入环境变量到所有 Shell 执行

**权限事件**：
- `permission.asked` / `permission.replied`

**命令事件**：
- `command.executed`

**TUI 事件**：
- `tui.prompt.append` / `tui.command.execute` / `tui.toast.show`

**示例 — 阻止读取 .env 文件**：
```javascript
export const EnvProtection = async () => {
  return {
    "tool.execute.before": async (input, output) => {
      if (input.tool === "read" && output.args.filePath.includes(".env")) {
        throw new Error("Do not read .env files")
      }
    },
  }
}
```

**示例 — 注入环境变量**：
```javascript
export const InjectEnvPlugin = async () => {
  return {
    "shell.env": async (input, output) => {
      output.env.MY_API_KEY = "secret"
      output.env.PROJECT_ROOT = input.cwd
    },
  }
}
```

### 第五步：开发自定义工具

根据 [OpenCode 自定义工具规范](https://opencode.ai/docs/zh-cn/plugins/#自定义工具)，使用 `tool` 辅助函数创建工具：

```typescript
import { type Plugin, tool } from "@opencode-ai/plugin"

export const CustomToolsPlugin: Plugin = async (ctx) => {
  return {
    tool: {
      mytool: tool({
        description: "This is a custom tool",
        args: {
          foo: tool.schema.string(),
        },
        async execute(args, context) {
          const { directory, worktree } = context
          return `Hello ${args.foo} from ${directory} (worktree: ${worktree})`
        },
      }),
    },
  }
}
```

**自定义工具结构**：

| 字段 | 说明 |
|------|------|
| `description` | 工具的功能描述 |
| `args` | 工具参数的 Zod schema |
| `execute` | 工具被调用时执行的函数，接收 `args` 和 `context` |

> 注意：如果插件工具与内置工具使用相同的名称，则优先使用插件工具。

### 第六步：日志记录

使用 `client.app.log()` 进行结构化日志记录：

```javascript
export const MyPlugin = async ({ client }) => {
  await client.app.log({
    body: {
      service: "my-plugin",
      level: "info",
      message: "Plugin initialized",
      extra: { foo: "bar" },
    },
  })
}
```

日志级别：`debug`、`info`、`warn`、`error`。

### 第七步：打包与发布

1. **本地插件**：直接放置在 `.opencode/plugins/` 或 `~/.config/opencode/plugins/` 即可使用
2. **npm 发布**：如需发布到 npm，确保：
   - 插件包名遵循 npm 命名规范
   - 在 `opencode.json` 中配置 `"plugin": ["<package-name>"]`
   - 支持带作用域的包名，如 `@my-org/custom-plugin`

## 输出产物

- 完整的 OpenCode 插件 JavaScript/TypeScript 文件
- `.opencode/package.json` 依赖配置（可选）
- 事件 Hook 实现代码
- 自定义工具实现代码
- 打包与发布建议

## 官方文档参考

| 文档 | 地址 |
|------|------|
| 📖 插件总文档 | [https://opencode.ai/docs/zh-cn/plugins/](https://opencode.ai/docs/zh-cn/plugins/) |
| 📖 事件列表 | [https://opencode.ai/docs/zh-cn/plugins/#事件](https://opencode.ai/docs/zh-cn/plugins/#事件) |
| 📖 自定义工具 | [https://opencode.ai/docs/zh-cn/plugins/#自定义工具](https://opencode.ai/docs/zh-cn/plugins/#自定义工具) |
| 📖 TypeScript 支持 | [https://opencode.ai/docs/zh-cn/plugins/#typescript-支持](https://opencode.ai/docs/zh-cn/plugins/#typescript-支持) |
| 📖 从 npm 加载 | [https://opencode.ai/docs/zh-cn/plugins/#从-npm-加载](https://opencode.ai/docs/zh-cn/plugins/#从-npm-加载) |
| 📖 SDK 文档 | [https://opencode.ai/docs/sdk](https://opencode.ai/docs/sdk) |
| 🔧 生态系统（社区插件） | [https://opencode.ai/docs/ecosystem](https://opencode.ai/docs/ecosystem) |