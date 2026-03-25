# 视觉头脑风暴重构：浏览器显示，终端命令

**日期：** 2026-02-19
**状态：** 已批准
**范围：** `lib/brainstorm-server/`、`skills/brainstorming/visual-companion.md`、`tests/brainstorm-server/`

## 问题

在视觉头脑风暴期间，Claude 运行 `wait-for-feedback.sh` 作为后台任务并阻塞在 `TaskOutput(block=true, timeout=600s)` 上。这完全占据了 TUI——当视觉头脑风暴运行时，用户无法向 Claude 输入。浏览器成为唯一的输入通道。

Claude Code 的执行模型是基于轮次的。在单个轮次内没有办法让 Claude 同时监听两个通道。阻塞式 `TaskOutput` 模式是错误的原语——它模拟了平台不支持的事件驱动行为。

## 设计

### 核心模型

**浏览器 = 交互式显示器。** 显示模拟，让用户点击选择选项。选择在服务器端记录。

**终端 = 对话通道。** 始终不阻塞，始终可用。用户在这里与 Claude 对话。

### 循环

1. Claude 将 HTML 文件写入会话目录
2. 服务器通过 chokidar 检测到它，向浏览器推送 WebSocket 重新加载（不变）
3. Claude 结束其轮次 — 告诉用户检查浏览器并在终端中响应
4. 用户查看浏览器，可选点击选择选项，然后在终端中输入反馈
5. 在下一轮，Claude 读取 `$SCREEN_DIR/.events` 获取浏览器交互流（点击、选择），与终端文本合并
6. 迭代或推进

无后台任务。无 `TaskOutput` 阻塞。无轮询脚本。

### 关键删除：`wait-for-feedback.sh`

完全删除。它的目的是桥接"服务器将事件记录到 stdout"和"Claude 需要接收这些事件"。`.events` 文件取代了它 — 服务器直接写入用户交互事件，Claude 使用平台提供的任何文件读取机制读取它们。

### 关键添加：`.events` 文件（每个屏幕的事件流）

服务器将所有用户交互事件写入 `$SCREEN_DIR/.events`，每行一个 JSON 对象。这为 Claude 提供了当前屏幕的完整交互流 — 不仅包括最终选择，还包括用户的探索路径（点击了 A，然后 B，最后确定 C）。

用户探索选项后的示例内容：

```jsonl
{"type":"click","choice":"a","text":"Option A - Preset-First Wizard","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Manual Config","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid Approach","timestamp":1706000115}
```

- 在屏幕内仅追加。每个用户事件作为新行追加。
- 当 chokidar 检测到新的 HTML 文件（新屏幕推送）时，文件被清除（删除）。防止过时事件携带。
- 当 Claude 读取时文件不存在，则没有浏览器交互发生 — Claude 仅使用终端文本。
- 文件仅包含用户事件（`click` 等）— 不包含服务器生命周期事件（`server-started`、`screen-added`）。这使其保持小规模和专注。
- Claude 可以读取完整流以了解用户的探索模式，或者只查看最后的 `choice` 事件作为最终选择。

## 按文件的变更

### `index.js`（服务器）

**A. 将用户事件写入 `.events` 文件。**

在 WebSocket `message` 处理器中，在将事件记录到 stdout 之后：通过 `fs.appendFileSync` 将事件作为 JSON 行追加到 `$SCREEN_DIR/.events`。仅写入用户交互事件（那些有 `source: 'user-event'` 的），不写入服务器生命周期事件。

**B. 在新屏幕上清除 `.events`。**

在 chokidar `add` 处理器中（检测到新的 `.html` 文件），删除 `$SCREEN_DIR/.events`（如果存在）。这是确定的"新屏幕"信号 — 比在每次重新加载时清除的 GET `/` 更好。

**C. 替换 `wrapInFrame` 内容注入。**

当前正则表达式锚点在 `<div class="feedback-footer">` 上，正被删除。替换为注释占位符：删除 `#claude-content` 内的默认内容（`<h2>Visual Brainstorming</h2>` 和副标题段落）并替换为单个 `<!-- CONTENT -->` 标记。内容注入变为 `frameTemplate.replace('<!-- CONTENT -->', content)`。更简单，如果模板格式更改不会破坏。

### `frame-template.html`（UI 框架）

**删除：**
- `feedback-footer` div（textarea、发送按钮、标签、`.feedback-row`）
- 相关 CSS（`.feedback-footer`、`.feedback-footer label`、`.feedback-row`、其中的 textarea 和 button 样式）

**添加：**
- 在 `#claude-content` 内部添加 `<!-- CONTENT -->` 占位符，替换默认文本
- 在页脚位置添加选择指示器栏，具有两种状态：
  - 默认："Click an option above, then return to the terminal"
  - 选择后："Option B selected — return to terminal to continue"
- 指示器栏的 CSS（微妙，类似于现有页眉的视觉权重）

**保持不变：**
- 带"Brainstorm Companion"标题和连接状态的页眉栏
- `.main` 包装器和 `#claude-content` 容器
- 所有组件 CSS（`.options`、`.cards`、`.mockup`、`.split`、`.pros-cons`、占位符、模拟元素）
- 深色/浅色主题变量和媒体查询

### `helper.js`（客户端脚本）

**删除：**
- `sendToClaude()` 函数和"Sent to Claude"页面接管
- `window.send()` 函数（绑定到已删除的发送按钮）
- 表单提交处理器 — 没有反馈 textarea 就没有用途，添加日志噪音
- 输入更改处理器 — 同样原因
- `pageshow` 事件监听器（添加用于修复 textarea 持久化 — 不再有 textarea）

**保留：**
- WebSocket 连接、重连逻辑、事件队列
- 重新加载处理器（服务器推送时 `window.location.reload()`）
- `window.toggleSelect()` 用于选择高亮
- `window.selectedChoice` 跟踪
- `window.brainstorm.send()` 和 `window.brainstorm.choice()` — 这些与已删除的 `window.send()` 不同。它们调用 `sendEvent`，通过 WebSocket 记录到服务器。对自定义完整文档页面有用。

**缩小：**
- 点击处理器：仅捕获 `[data-choice]` 点击，不是所有按钮/链接。当浏览器是反馈通道时需要广泛捕获；现在只是选择跟踪。

**添加：**
- 在 `data-choice` 点击时，更新选择指示器栏文本以显示选择了哪个选项。

**从 `window.brainstorm` API 中删除：**
- `brainstorm.sendToClaude` — 不再存在

### `visual-companion.md`（技能说明）

**重写"The Loop"部分**为上述非阻塞流。删除所有对以下内容的引用：
- `wait-for-feedback.sh`
- `TaskOutput` 阻塞
- 超时/重试逻辑（600 秒超时、30 分钟上限）
- 描述 `send-to-claude` JSON 的"用户反馈格式"部分

**替换为：**
- 新循环（写 HTML → 结束轮次 → 用户在终端响应 → 读取 `.events` → 迭代）
- `.events` 文件格式文档
- 关于终端消息是主要反馈；`.events` 提供完整浏览器交互流以获取额外上下文的指导

**保留：**
- 服务器启动/关闭说明
- 内容片段 vs 完整文档指导
- CSS 类参考和可用组件
- 设计提示（根据问题扩展保真度、每个屏幕 2-4 个选项等）

### `wait-for-feedback.sh`

**完全删除。**

### `tests/brainstorm-server/server.test.js`

需要更新的测试：
- 断言片段响应中存在 `feedback-footer` 的测试 — 更新为断言选择指示器栏或 `<!-- CONTENT -->` 替换
- 断言 `helper.js` 包含 `send` 的测试 — 更新以反映缩小的 API
- 断言 `sendToClaude` CSS 变量使用的测试 — 删除（函数不再存在）

## 平台兼容性

服务器代码（`index.js`、`helper.js`、`frame-template.html`）完全平台无关 — 纯 Node.js 和浏览器 JavaScript。没有 Claude Code 特定引用。已经通过后台终端交互在 Codex 上证明可以工作。

技能说明（`visual-companion.md`）是平台适配层。每个平台的 Claude 使用自己的工具启动服务器、读取 `.events` 等。非阻塞模型自然跨平台工作，因为它不依赖任何平台特定的阻塞原语。

## 这实现了什么

- **TUI 在视觉头脑风暴期间始终响应**
- **混合输入** — 在浏览器中点击 + 在终端中输入，自然合并
- **优雅降级** — 浏览器宕机或用户未打开？终端仍然工作
- **更简单的架构** — 无后台任务、无轮询脚本、无超时管理
- **跨平台** — 相同的服务器代码在 Claude Code、Codex 和任何未来平台上都能工作

## 这放弃了什么

- **纯浏览器反馈工作流程** — 用户必须返回终端继续。选择指示器栏引导他们，但与旧的点击-发送-等待流程相比多了一步。
- **来自浏览器的内联文本反馈** — textarea 消失了。所有文本反馈通过终端。这是有意的 — 终端比框架中小 textarea 是更好的文本输入通道。
- **浏览器发送后立即响应** — 旧系统在用户点击发送时 Claude 立即响应。现在在用户切换到终端时有一个间隙。实际上这是几秒钟，用户可以在终端消息中添加上下文。