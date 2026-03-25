# Persuasion Principles for Skill Design Skill 设计的说服原则

## 概述

LLMs 像人类一样对相同的说服原则做出反应。理解这种心理学帮助你设计更有效的 skills——不是为了操纵，而是确保即使在压力下也能遵循关键实践。

**研究基础：** Meincke et al. (2025) 用 N=28,000 次 AI 对话测试了 7 个说服原则。说服技术使合规率增加一倍以上（33% → 72%，p < .001）。

## 七个原则

### 1. 权威

**它是什么：** 对专业知识、资质或官方来源的尊重。

**它在 skills 中如何工作：**
- 命令式语言："YOU MUST"、"Never"、"Always"
- 不可协商的框架："No exceptions"
- 消除决策疲劳和合理化

**何时使用：**
- 纪律强制 skills（TDD、验证要求）
- 安全关键实践
- 既定最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```
✅ 在测试前写代码？删除它。重新开始。无例外。
❌ 考虑在可行时先写测试。

### 2. 承诺

**它是什么：** 与先前行为、声明或公开宣言的一致性。

**它在 skills 中如何工作：**
- 要求宣布："Announce skill usage"
- 强制明确选择："Choose A, B, or C"
- 使用跟踪：TodoWrite 用于检查清单

**何时使用：**
- 确保 skills 实际被遵循
- 多步骤流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```
✅ 当你找到 skill，你必须宣布："我正在使用 [Skill 名称]"
❌ 考虑让你的伙伴知道你正在使用哪个 skill。

### 3. 稀缺

**它是什么：** 时间限制或有限可用性带来的紧迫感。

**它在 skills 中如何工作：**
- 时间绑定要求："Before proceeding"
- 顺序依赖："Immediately after X"
- 防止拖延

**何时使用：**
- 立即验证要求
- 时间敏感工作流
- 防止"我稍后会做"

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```
✅ 完成任务后，立即请求代码审查再继续。
❌ 你可以在方便时审查代码。

### 4. 社会证明

**它是什么：** 对他人做什么或什么是正常的从众心理。

**它在 skills 中如何工作：**
- 普遍模式："Every time"、"Always"
- 失败模式："X without Y = failure"
- 建立规范

**何时使用：**
- 记录普遍实践
- 警告常见失败
- 加强标准

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```
✅ 没有 TodoWrite 跟踪的检查清单 = 步骤被跳过。每次都是。
❌ 有些人觉得 TodoWrite 对检查清单有帮助。

### 5. 一致

**它是什么：** 共同身份、"我们"感、圈内归属。

**它在 skills 中如何工作：**
- 协作语言："our codebase"、"we're colleagues"
- 共同目标："we both want quality"

**何时使用：**
- 协作工作流
- 建立团队文化
- 非等级实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```
✅ 我们是一起工作的同事。我需要你诚实的技术判断。
❌ 如果我错了你可能应该告诉我。

### 6. 互惠

**它是什么：** 返回所获利益的义务。

**它如何工作：**
- 谨慎使用——可能感觉像操纵
- Skills 中很少需要

**何时避免：**
- 几乎总是（其他原则更有效）

### 7. 喜好

**它是什么：** 对我们喜欢的人合作的偏好。

**它如何工作：**
- **不要用于合规**
- 与诚实反馈文化冲突
- 产生谄媚

**何时避免：**
- 始终用于纪律强制

## 按 Skill 类型的原则组合

| Skill 类型 | 使用 | 避免 |
|------------|-----|-------|
| 纪律强制 | 权威 + 承诺 + 社会证明 | 喜好、互惠 |
| 指导/技术 | 中等权威 + 一致 | 重度权威 |
| 协作 | 一致 + 承诺 | 权威、喜好 |
| 参考 | 仅清晰度 | 所有说服 |

## 为什么这有效：心理学

**明线规则减少合理化：**
- "YOU MUST" 消除决策疲劳
- 绝对语言消除"这是例外吗？"问题
- 明确的反合理化计数器关闭特定漏洞

**实施意图创造自动行为：**
- 明确触发器 + 要求的行动 = 自动执行
- "当 X，做 Y"比" generally do Y"更有效
- 减少合规的认知负担

**LLMs 是准人类：**
- 在包含这些模式的训练数据上训练
- 权威语言在训练数据中先于合规
- 承诺序列（声明 → 行动）经常被建模
- 社会证明模式（每个人都做 X）建立规范

## 伦理使用

**合法：**
- 确保关键实践被遵循
- 创建有效文档
- 防止可预测的失败

**非法：**
- 为个人利益操纵
- 制造虚假紧迫感
- 基于内疚的合规

**测试：** 如果用户完全理解这项技术会为他们的真正利益服务吗？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 七大说服原则
- 影响研究的经验基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 用 N=28,000 次 LLM 对话测试了 7 个原则
- 说服技术使合规从 33% 增加到 72%
- 权威、承诺、稀缺最有效
- 验证 LLM 行为的准人类模型

## 快速参考

设计 skill 时问：

1. **它是什么类型？**（纪律 vs 指导 vs 参考）
2. **我试图改变什么行为？**
3. **哪个原则适用？**（通常是纪律的权威 + 承诺）
4. **我组合得太多了吗？**（不要用所有七个）
5. **这是伦理的吗？**（为用户的真正利益服务？）
