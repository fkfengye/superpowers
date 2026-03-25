# 文档审查系统设计

## 概述

在 superpowers 工作流程中添加两个新的审查阶段：

1. **规范文档审查** - 头脑风暴之后，writing-plans 之前
2. **计划文档审查** - writing-plans 之后，实施之前

两者都遵循与实施审查相同的迭代循环模式。

## 规范文档审查者

**目的：** 验证规范完整、一致且准备好进行实施规划。

**位置：** `skills/brainstorming/spec-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 检查内容 |
|----------|------------------|
| 完整性 | TODO、占位符、"TBD"、不完整部分 |
| 覆盖范围 | 缺失的错误处理、边缘情况、集成点 |
| 一致性 | 内部矛盾、冲突的需求 |
| 清晰度 | 模糊的需求 |
| YAGNI | 未请求的功能、过度工程 |

**输出格式：**
```
## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [issue] - [why it matters]

**Recommendations (advisory):**
- [suggestions that don't block approval]
```

**审查循环：** 发现问题 -> 头脑风暴代理修复 -> 重新审查 -> 重复直到批准。

**调度机制：** 使用带有 `subagent_type: general-purpose` 的 Task 工具。审查者提示模板提供完整提示。头脑风暴技能的控制器调度审查者。

## 计划文档审查者

**目的：** 验证计划完整、匹配规范且具有正确的任务分解。

**位置：** `skills/writing-plans/plan-document-reviewer-prompt.md`

**检查内容：**

| 类别 | 检查内容 |
|----------|------------------|
| 完整性 | TODO、占位符、不完整的任务、缺失的步骤 |
| 规范对齐 | 计划涵盖相关规范要求，无范围蔓延 |
| 任务分解 | 任务原子化、边界清晰、步骤可操作 |
| 任务语法 | 任务和步骤上的复选框语法（`- [ ]`） |
| 块大小 | 每个块小于 1000 行 |

**块定义：** 块是计划文档中任务的逻辑分组，由 `## Chunk N: <name>` 标题分隔。writing-plans 技能基于逻辑阶段创建这些边界（例如"Foundation"、"Core Features"、"Integration"）。每个块应该足够独立以便独立审查。

**规范对齐验证：** 审查者接收：
1. 计划文档（或当前块）
2. 规范文档路径供参考

审查者读取两者并比较需求覆盖。

**输出格式：** 与规范审查者相同，但范围限定在当前块。

**审查流程（逐块）：**
1. Writing-plans 创建块 N
2. 控制器调度计划文档审查者，提供块 N 内容和规范路径
3. 审查者读取块和规范，返回裁决
4. 如果有问题：writing-plans 代理修复块 N，跳转到步骤 2
5. 如果批准：继续块 N+1
6. 重复直到所有块批准

**调度机制：** 与规范审查者相同 - 带有 `subagent_type: general-purpose` 的 Task 工具。

## 更新的工作流程

```
brainstorming -> spec -> SPEC REVIEW LOOP -> writing-plans -> plan -> PLAN REVIEW LOOP -> implementation
```

**规范审查循环：**
1. 规范完成
2. 调度审查者
3. 如果有问题：修复 -> 跳转到 2
4. 如果批准：继续

**计划审查循环：**
1. 块 N 完成
2. 为块 N 调度审查者
3. 如果有问题：修复 -> 跳转到 2
4. 如果批准：下一个块或实施

## Markdown 任务语法

任务和步骤使用复选框语法：

```markdown
- [ ] ### Task 1: Name

- [ ] **Step 1:** Description
  - File: path
  - Command: cmd
```

## 错误处理

**审查循环终止：**
- 没有硬性迭代限制 - 循环继续直到审查者批准
- 如果循环超过 5 次迭代，控制器应将此问题提交给人工获取指导
- 人工可以选择：继续迭代、带着已知问题批准、或中止

**分歧处理：**
- 审查者是咨询性质的 - 他们标记问题但不阻止
- 如果代理认为审查者反馈不正确，它应该在修复中解释原因
- 如果在同一问题上分歧持续 3 次迭代后仍未解决，提交给人工

**格式错误的审查者输出：**
- 控制器应验证审查者输出包含必填字段（Status、如果适用则有 Issues）
- 如果格式错误，用关于预期格式的说明重新调度审查者
- 2 次格式错误的响应后，提交给人工

## 要变更的文件

**新文件：**
- `skills/brainstorming/spec-document-reviewer-prompt.md`
- `skills/writing-plans/plan-document-reviewer-prompt.md`

**修改的文件：**
- `skills/brainstorming/SKILL.md` - 在规范写完后添加审查循环
- `skills/writing-plans/SKILL.md` - 添加逐块审查循环，更新任务语法示例