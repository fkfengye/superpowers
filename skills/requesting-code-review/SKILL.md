---
name: requesting-code-review
description: 当完成任务、实现主要功能或合并前验证工作时使用
---

# 请求代码审查

分发 superpowers:code-reviewer 代理来在问题扩散前捕获它们。审查者获得精确设计的上下文进行评估——绝不是你的会话历史。这让审查者专注于工作产品，而不是你的思维过程，并保留你自己的上下文以继续工作。

**核心原则：** 尽早审查，经常审查。

## 何时请求审查

**必须：**
- subagent-driven development 中的每个任务之后
- 完成主要功能后
- 合并到 main 前

**可选但有价值：**
- 卡住时（全新视角）
- 重构前（基线检查）
- 修复复杂 bug 后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 分发 code-reviewer 代理：**

使用 Task 工具，类型为 superpowers:code-reviewer，填写 `code-reviewer.md` 中的模板

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 你刚刚构建的内容
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交
- `{DESCRIPTION}` - 简要摘要

**3. 采取行动：**
- 立即修复 Critical 问题
- 继续前修复 Important 问题
- 记录 Minor 问题供稍后
- 如果审查者错了，反驳（带推理）

## 示例

```
[刚完成任务 2：添加验证函数]

你：让我在继续前请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[分发 superpowers:code-reviewer 代理]
  WHAT_WAS_IMPLEMENTED: 对话索引的验证和修复函数
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，4 种问题类型

[代理返回]:
  Strengths: 架构清晰，真正的测试
  Issues:
    Important: 缺少进度指示器
    Minor: 魔法数字 (100) 用于报告间隔
  Assessment: 可以继续

你：[修复进度指示器]
[继续任务 3]
```

## 与工作流集成

**Subagent-Driven Development：**
- 每个任务后审查
- 在问题累积前捕获
- 修复后再继续下一个任务

**Executing Plans：**
- 每批后审查（3个任务）
- 获取反馈，应用，继续

**Ad-Hoc Development：**
- 合并前审查
- 卡住时审查

## 红旗

**永远不要：**
- 因为"很简单"就跳过审查
- 忽略 Critical 问题
- 在 Important 问题未修复时继续
- 用有效的技术反馈争辩

**如果审查者错了：**
- 用技术推理反驳
- 展示证明其有效的代码/测试
- 请求澄清

模板位置：requesting-code-review/code-reviewer.md
