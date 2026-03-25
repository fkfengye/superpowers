---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans 编写计划

## 概述

编写全面的实施计划，假设工程师对我们的代码库零上下文且品味可疑。记录他们需要知道的一切：每个任务涉及哪些文件、代码、他们可能需要检查的测试和文档、如何测试。把整个计划给他们作为小任务。DRY。YAGNI。TDD。频繁提交。

假设他们是一个熟练的开发者，但几乎不了解我们的工具集或问题领域。假设他们不太了解好的测试设计。

**开头宣布：** "我正在使用 writing-plans 技能来创建实施计划。"

**上下文：** 这应该在专用 worktree 中运行（由 brainstorming 技能创建）。

**保存计划到：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划位置的偏好覆盖此默认）

## 范围检查

如果规范涵盖多个独立子系统，它应该在 brainstorming 期间被分解为子项目规范。如果没有，建议分解为单独的计划——每个子系统一个。每个计划应该独立产生可测试的软件。

## 文件结构

在定义任务之前，映射将创建或修改哪些文件以及每个文件的职责。这是分解决策被锁定的地方。

- 用清晰的边界和定义良好的接口设计单元。每个文件应该有一个清晰的职责。
- 你最好能一次掌握上下文的代码，当你编辑时更可靠，喜欢更小、更专注的文件而非做太多事的大文件。
- 一起变更的文件应该放在一起。按职责拆分，而非按技术层。
- 在现有代码库中，遵循既定模式。如果代码库使用大文件，不要单方面重构——但如果你修改的文件变得笨重，在计划中包含拆分是合理的。

此结构为任务分解提供信息。每个任务应该产生独立的、有意义的变更。

## 小任务粒度

**每步是一个动作（2-5 分钟）：**
- "写失败的测试" - 步骤
- "运行它确保它失败" - 步骤
- "写最少的代码使测试通过" - 步骤
- "运行测试确保它们通过" - 步骤
- "提交" - 步骤

## 计划文档头部

**每个计划必须以此头部开始：**

```markdown
# [功能名称] 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [一句话描述构建内容]

**Architecture:** [2-3 句关于方法的话]

**Tech Stack:** [关键技术和库]

---
```

## 任务结构

````markdown
### Task N: [组件名称]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 记住

- 始终精确的文件路径
- 计划中的完整代码（不是"添加验证"）
- 带预期输出的精确命令
- 使用 @ 语法引用相关 skills
- DRY, YAGNI, TDD, 频繁提交

## 计划审查循环

写完完整计划后：

1. 使用精确制作的审查上下文分派单个计划文档审查 subagent（参见 plan-document-reviewer-prompt.md）——绝不是你的会话历史。这让审查者专注于计划，而非你的思考过程。
   - 提供：计划文档路径，规范文档路径
2. 如果 ❌ 发现问题：修复问题，重新分派审查者审查整个计划
3. 如果 ✅ 批准：继续执行交接

**审查循环指导：**
- 写计划的同一 agent 修复它（保留上下文）
- 如果循环超过 3 次迭代，向人工寻求指导
- 审查者是咨询性质的——如果你认为反馈不正确，解释分歧

## 执行交接

保存计划后，提供执行选择：

**"计划完成并保存到 `docs/superpowers/plans/<filename>.md`。两种执行选项：**

**1. Subagent 驱动（推荐）** - 我为每个任务分派新的 subagent，在任务之间审查，快速迭代

**2. 内联执行** - 使用 executing-plans 在此会话中执行任务，带检查点的批处理执行

**你选择哪个？"**

**如果选择 Subagent 驱动：**
- **必需子技能：** 使用 superpowers:subagent-driven-development
- 每个任务的新 subagent + 两阶段审查

**如果选择内联执行：**
- **必需子技能：** 使用 superpowers:executing-plans
- 带审查检查点的批处理执行
