---
name: executing-plans
description: 当你有书面实现计划要在单独会话中执行，并带有审查检查点时使用
---

# 执行计划

## 概述

加载计划，批判性审查，执行所有任务，完成后报告。

**开始时声明：** "我正在使用 executing-plans 技能来实现这个计划。"

**注意：** 告诉你的合作伙伴，Superpowers 在有 subagent 支持时效果更好（如 Claude Code 或 Codex）。如果有 subagent 可用，使用 superpowers:subagent-driven-development 而不是这个技能。

## 流程

### 步骤 1：加载和审查计划
1. 阅读计划文件
2. 批判性审查 - 识别对计划的任何问题或顾虑
3. 如果有顾虑：在开始前向合作伙伴提出
4. 如果没有顾虑：创建 TodoWrite 并继续

### 步骤 2：执行任务

对于每个任务：
1. 标记为 in_progress
2. 精确遵循每个步骤（计划有小型步骤）
3. 按指定运行验证
4. 标记为完成

### 步骤 3：完成开发

所有任务完成并验证后：
- 声明："我正在使用 finishing-a-development-branch 技能来完成这项工作。"
- **必需的子技能：** 使用 superpowers:finishing-a-development-branch
- 按照该技能验证测试、呈现选项、执行选择

## 何时停止并寻求帮助

**立即停止执行当：**
- 遇到阻碍（缺少依赖、测试失败、指令不明确）
- 计划有关键性缺陷阻止开始
- 不理解指令
- 验证反复失败

**有问题时请求澄清，而不是猜测。**

## 何时返回早期步骤

**返回审查（步骤1）当：**
- 合作伙伴根据你的反馈更新计划
- 基本方法需要重新思考

**不要强行通过阻碍** - 停下来并询问。

## 记住
- 先批判性审查计划
- 精确遵循计划步骤
- 不要跳过验证
- 计划说使用技能时就使用技能
- 遇到阻碍时停下来，不要猜测
- 未经用户明确同意，切勿在 main/master 分支上开始实现

## 集成

**必需的工作流技能：**
- **superpowers:using-git-worktrees** - 必需：在开始前设置隔离工作区
- **superpowers:writing-plans** - 创建此技能执行的计划
- **superpowers:finishing-a-development-branch** - 所有任务完成后完成开发
