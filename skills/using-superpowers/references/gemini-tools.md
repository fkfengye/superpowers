# Gemini CLI Tool Mapping Gemini CLI 工具映射

Skills use Claude Code tool names. When you encounter these in a skill, use your platform equivalent:
Skills 使用 Claude Code 工具名称。当你在 skill 中遇到这些时，使用你的平台等价物：

| Skill references | Gemini CLI equivalent |
|-----------------|----------------------|
| `Read` (file reading) | `read_file` |
| `Write` (file creation) | `write_file` |
| `Edit` (file editing) | `replace` |
| `Bash` (run commands) | `run_shell_command` |
| `Grep` (search file content) | `grep_search` |
| `Glob` (search files by name) | `glob` |
| `TodoWrite` (task tracking) | `write_todos` |
| `Skill` tool (invoke a skill) | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` tool (dispatch subagent) | No equivalent — Gemini CLI does not support subagents |

| Skill 引用 | Gemini CLI 等价物 |
|-----------|------------------|
| `Read`（文件读取） | `read_file` |
| `Write`（文件创建） | `write_file` |
| `Edit`（文件编辑） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名称搜索文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` 工具（调用 skill） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（分派 subagent） | 无等价物 — Gemini CLI 不支持 subagents |

## No subagent support 无 subagent 支持

Gemini CLI has no equivalent to Claude Code's `Task` tool. Skills that rely on subagent dispatch (`subagent-driven-development`, `dispatching-parallel-agents`) will fall back to single-session execution via `executing-plans`.
Gemini CLI 没有 Claude Code 的 `Task` 工具等价物。依赖 subagent 分派的 skills（`subagent-driven-development`、`dispatching-parallel-agents`）将通过 `executing-plans` 回退到单个会话执行。

## Additional Gemini CLI tools 额外的 Gemini CLI 工具

These tools are available in Gemini CLI but have no Claude Code equivalent:
这些工具在 Gemini CLI 中可用但没有 Claude Code 等价物：

| Tool | Purpose |
|------|---------|
| `list_directory` | List files and subdirectories |
| `save_memory` | Persist facts to GEMINI.md across sessions |
| `ask_user` | Request structured input from the user |
| `tracker_create_task` | Rich task management (create, update, list, visualize) |
| `enter_plan_mode` / `exit_plan_mode` | Switch to read-only research mode before making changes |

| 工具 | 目的 |
|------|------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 跨会话持久化事实到 GEMINI.md |
| `ask_user` | 从用户请求结构化输入 |
| `tracker_create_task` | 丰富的任务管理（创建、更新、列表、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在进行更改之前切换到只读研究模式 |
