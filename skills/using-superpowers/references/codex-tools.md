# Codex Tool Mapping Codex 工具映射

Skills use Claude Code tool names. When you encounter these in a skill, use your platform equivalent:
Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些时，使用你的平台等价物：

| Skill references | Codex equivalent |
|-----------------|------------------|
| `Task` tool (dispatch subagent) | `spawn_agent` |
| Multiple `Task` calls (parallel) | Multiple `spawn_agent` calls |
| Task returns result | `wait` |
| Task completes automatically | `close_agent` to free slot |
| `TodoWrite` (task tracking) | `update_plan` |
| `Skill` tool (invoke a skill) | Skills load natively — just follow the instructions |
| `Read`, `Write`, `Edit` (files) | Use your native file tools |
| `Bash` (run commands) | Use your native shell tools |

| Skill 引用 | Codex 等价物 |
|-----------|-------------|
| `Task` 工具（分派 subagent） | `spawn_agent` |
| 多个 `Task` 调用（并行） | 多个 `spawn_agent` 调用 |
| Task 返回结果 | `wait` |
| Task 自动完成 | `close_agent` 释放槽位 |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` 工具（调用 skill） | Skills 本地加载——只需遵循指令 |
| `Read`、`Write`、`Edit`（文件） | 使用你的原生文件工具 |
| `Bash`（运行命令） | 使用你的原生 shell 工具 |

## Subagent dispatch requires multi-agent support Subagent 分派需要多代理支持

Add to your Codex config (`~/.codex/config.toml`):
添加到你的 Codex 配置（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

This enables `spawn_agent`, `wait`, and `close_agent` for skills like `dispatching-parallel-agents` and `subagent-driven-development`.
这为 `dispatching-parallel-agents` 和 `subagent-driven-development` 等 skills 启用 `spawn_agent`、`wait` 和 `close_agent`。
