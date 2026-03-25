# OpenCode 版 Superpowers

[OpenCode.ai](https://opencode.ai) 使用 Superpowers 的完整指南。

## 安装

将 superpowers 添加到 `opencode.json`（全局或项目级别）的 `plugin` 数组中：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

重启 OpenCode。插件通过 Bun 自动安装并自动注册所有技能。

通过询问来验证："Tell me about your superpowers"

### 从旧的基于符号链接的安装迁移

如果您以前使用 `git clone` 和符号链接安装过 superpowers，请删除旧设置：

```bash
# 删除旧的符号链接
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# 可选删除克隆的仓库
rm -rf ~/.config/opencode/superpowers

# 如果您为 superpowers 添加了 skills.paths，请从 opencode.json 中删除
```

然后按照上面的安装步骤操作。

## 使用方法

### 查找技能

使用 OpenCode 原生的 `skill` 工具列出所有可用技能：

```
use skill tool to list skills
```

### 加载技能

```
use skill tool to load superpowers/brainstorming
```

### 个人技能

在 `~/.config/opencode/skills/` 中创建你自己的技能：

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

创建 `~/.config/opencode/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

### 项目技能

在项目的 `.opencode/skills/` 中创建项目特定的技能。

**技能优先级：** 项目技能 > 个人技能 > Superpowers 技能

## 更新

Superpowers 在您重启 OpenCode 时自动更新。插件在每次启动时从 git 仓库重新安装。

要固定特定版本，请使用分支或标签：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v5.0.3"]
}
```

## 工作原理

插件做两件事：

1. **通过 `experimental.chat.system.transform` 钩子注入引导上下文**，在每次对话中添加 superpowers 感知。
2. **通过 `config` 钩子注册技能目录**，使 OpenCode 无需符号链接或手动配置即可发现所有 superpowers 技能。

### 工具映射

为 Claude Code 编写的技能会自动适配到 OpenCode：

- `TodoWrite` → `todowrite`
- 带子代理的 `Task` → OpenCode 的 `@mention` 系统
- `Skill` 工具 → OpenCode 原生的 `skill` 工具
- 文件操作 → OpenCode 原生工具

## 故障排除

### 插件未加载

1. 检查 OpenCode 日志：`opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. 验证 `opencode.json` 中的插件行是否正确
3. 确保您运行的是最新版本的 OpenCode

### 找不到技能

1. 使用 OpenCode 的 `skill` 工具列出可用技能
2. 检查插件是否正在加载（见上文）
3. 每个技能需要一个带有有效 YAML 前置数据的 `SKILL.md` 文件

### 引导内容未出现

1. 检查 OpenCode 版本是否支持 `experimental.chat.system.transform` 钩子
2. 配置更改后重启 OpenCode

## 获取帮助

- 报告问题：https://github.com/obra/superpowers/issues
- 主文档：https://github.com/obra/superpowers
- OpenCode 文档：https://opencode.ai/docs/