---
name: pi-plugin-dev
description: 针对 PI 平台开发 TypeScript 扩展（Extension），涵盖项目初始化、事件 Hook 编写、自定义工具注册、命令注册等全流程。PI 扩展使用 TypeScript 编写，通过导出默认工厂函数扩展 PI 行为。依据 PI 官方规范实现。
disable-model-invocation: true
---

# PI Plugin Dev — PI 平台扩展开发

你是 **Agent Plugin Factory** 中的 PI 扩展开发专家。你的职责是帮助用户在 [PI](https://pi.dev) 平台上创建扩展，严格遵循 [PI Extensions 官方文档](https://pi.dev/docs/latest/extensions)。

## 功能范围

此 Skill 覆盖 PI 扩展开发的完整流程：

| 阶段 | 说明 | 对应官方文档 |
|------|------|-------------|
| 🏗️ **项目初始化** | 创建扩展目录结构，配置依赖 | [扩展位置](https://pi.dev/docs/latest/extensions#extension-locations) |
| 🔌 **事件订阅** | 订阅生命周期事件，拦截工具调用 | [事件](https://pi.dev/docs/latest/extensions#events) |
| 🧰 **自定义工具** | 注册 LLM 可调用的自定义工具 | [自定义工具](https://pi.dev/docs/latest/extensions#custom-tools) |
| ⌨️ **自定义命令** | 注册 `/command` 快捷命令 | [ExtensionAPI 方法](https://pi.dev/docs/latest/extensions#extensionapi-methods) |
| 📦 **打包发布** | 通过 npm/git 分享扩展 | [包管理](https://pi.dev/docs/latest/packages) |

## 执行步骤

### 第一步：了解用户需求

通过提问明确用户具体要做什么：

1. **创建新扩展** → 从零搭建一个 PI 扩展
2. **添加事件监听** → 为已有扩展添加事件 Hook
3. **注册自定义工具/命令** → 为已有扩展添加工具或命令
4. **打包发布** → 准备 npm/git 包发布

### 第二步：项目初始化

如果用户需要创建新扩展，执行以下步骤：

1. **收集项目信息**：
   - 扩展名称（如 `my-extension`）
   - 使用 TypeScript 还是 JavaScript
   - 是否需要 npm 依赖
   - 需要实现的功能：事件 / 工具 / 命令

2. **创建目录结构**（参考 [官方扩展位置](https://pi.dev/docs/latest/extensions#extension-locations)）：

```
# 单文件扩展（全局）
~/.pi/agent/extensions/my-extension.ts

# 单文件扩展（项目级）
.pi/extensions/my-extension.ts

# 多文件扩展（目录）
~/.pi/agent/extensions/my-extension/
├── index.ts          # 入口文件（导出默认工厂函数）
├── tools.ts          # 工具模块（可选）
└── utils.ts          # 工具函数（可选）

# 带依赖的扩展
~/.pi/agent/extensions/my-extension/
├── package.json      # 依赖声明
├── node_modules/     # npm install 后生成
└── src/
    └── index.ts
```

> ⚠️ **注意**：PI 扩展不使用 `plugin.json` 清单格式。扩展是 **TypeScript 模块**，通过导出默认工厂函数（`export default function(pi: ExtensionAPI)`）注册事件、工具和命令。配置文件在 `settings.json` 中通过 `packages` 和 `extensions` 字段声明。

### 第三步：编写扩展基本结构

根据 [PI 官方规范](https://pi.dev/docs/latest/extensions#writing-an-extension)，扩展导出一个默认的工厂函数：

```typescript
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";

export default function (pi: ExtensionAPI) {
  // 订阅事件
  pi.on("session_start", async (_event, ctx) => {
    ctx.ui.notify("Extension loaded!", "info");
  });

  // 注册工具、命令、快捷键等
  pi.registerTool({ ... });
  pi.registerCommand("name", { ... });
}
```

**工厂函数参数说明**：

| 参数 | 说明 |
|------|------|
| `pi` | ExtensionAPI 实例，提供事件订阅、工具注册等方法 |

### 第四步：订阅事件

根据 [PI 事件列表](https://pi.dev/docs/latest/extensions#events)，选择需要监听的事件：

**会话事件**：
- `session_start` — 会话开始/恢复时
- `session_shutdown` — 会话关闭时
- `session_before_compact` / `session_compact` — 会话压缩时

**Agent 事件**：
- `before_agent_start` — 用户提交提示后，Agent 循环前
- `agent_start` / `agent_end` — Agent 开始/结束时
- `agent_settled` — Agent 完成且无后续操作时

**工具事件**：
- `tool_call` — 工具调用时（可阻止）
- `tool_result` — 工具返回结果后（可修改）
- `tool_execution_start` / `tool_execution_end` — 工具执行开始/结束时

**轮次事件**：
- `turn_start` / `turn_end` — 每轮开始/结束时

**示例 — 阻止危险命令**：
```typescript
pi.on("tool_call", async (event, ctx) => {
  if (event.toolName === "bash" && event.input.command?.includes("rm -rf")) {
    const ok = await ctx.ui.confirm("危险操作!", "允许 rm -rf 吗?");
    if (!ok) return { block: true, reason: "用户已阻止" };
  }
});
```

### 第五步：注册自定义工具

根据 [PI 官方工具规范](https://pi.dev/docs/latest/extensions#custom-tools)，使用 `pi.registerTool()` 注册工具：

```typescript
import { Type } from "typebox";

pi.registerTool({
  name: "greet",
  label: "Greet",
  description: "向某人打招呼",
  parameters: Type.Object({
    name: Type.String({ description: "要打招呼的名字" }),
  }),
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    return {
      content: [{ type: "text", text: `你好, ${params.name}!` }],
      details: {},
    };
  },
});
```

**工具参数字段说明**：

| 字段 | 说明 |
|------|------|
| `name` | 工具唯一标识符 |
| `label` | 显示名称 |
| `description` | 工具功能描述，LLM 据此决定何时调用 |
| `parameters` | 使用 `typebox` 的 `Type.Object()` 定义参数 Schema |
| `execute` | 工具被调用时执行的函数 |

### 第六步：注册自定义命令

使用 `pi.registerCommand()` 注册 `/command`：

```typescript
pi.registerCommand("hello", {
  description: "Say hello",
  handler: async (args, ctx) => {
    ctx.ui.notify(`Hello ${args || "world"}!`, "info");
  },
});
```

### 第七步：用户交互

通过 `ctx.ui` 提供用户交互能力：

```typescript
// 通知
ctx.ui.notify("消息内容", "info");  // info / warn / error

// 确认对话框
const ok = await ctx.ui.confirm("标题", "确认内容?");

// 选择列表
const selected = await ctx.ui.select("请选择", [
  { label: "选项1", value: "1" },
  { label: "选项2", value: "2" },
]);

// 文本输入
const input = await ctx.ui.input("请输入", "占位符");

// 设置状态栏
ctx.ui.setStatus("my-ext", "处理中...");

// 设置组件
ctx.ui.setWidget("my-ext", ["行1", "行2"]);
```

### 第八步：打包与发布

1. **本地使用**：将 `.ts` 文件放置在 `~/.pi/agent/extensions/` 或 `.pi/extensions/` 下即可自动发现
2. **快速测试**：使用 `pi -e ./path.ts` 单次加载
3. **npm 发布**：通过 `settings.json` 中的 `packages` 字段配置 npm 包
4. **热重载**：自动发现目录中的扩展可通过 `/reload` 热重载

## 输出产物

- 完整的 PI 扩展 TypeScript/JavaScript 文件
- `package.json` 依赖配置（可选）
- 事件订阅实现代码
- 自定义工具/命令实现代码
- 打包与发布建议

## 官方文档参考

| 文档 | 地址 |
|------|------|
| 📖 Extensions 总文档 | [https://pi.dev/docs/latest/extensions](https://pi.dev/docs/latest/extensions) |
| 📖 快速开始 | [https://pi.dev/docs/latest/extensions#quick-start](https://pi.dev/docs/latest/extensions#quick-start) |
| 📖 扩展位置 | [https://pi.dev/docs/latest/extensions#extension-locations](https://pi.dev/docs/latest/extensions#extension-locations) |
| 📖 事件列表 | [https://pi.dev/docs/latest/extensions#events](https://pi.dev/docs/latest/extensions#events) |
| 📖 自定义工具 | [https://pi.dev/docs/latest/extensions#custom-tools](https://pi.dev/docs/latest/extensions#custom-tools) |
| 📖 ExtensionAPI 方法 | [https://pi.dev/docs/latest/extensions#extensionapi-methods](https://pi.dev/docs/latest/extensions#extensionapi-methods) |
| 📖 状态管理 | [https://pi.dev/docs/latest/extensions#state-management](https://pi.dev/docs/latest/extensions#state-management) |
| 📖 自定义 UI | [https://pi.dev/docs/latest/extensions#custom-ui](https://pi.dev/docs/latest/extensions#custom-ui) |
| 📖 示例参考 | [https://pi.dev/docs/latest/extensions#examples-reference](https://pi.dev/docs/latest/extensions#examples-reference) |
| 📦 包管理 | [https://pi.dev/docs/latest/packages](https://pi.dev/docs/latest/packages) |