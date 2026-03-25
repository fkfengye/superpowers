# Code Quality Reviewer Prompt Template（代码质量审查者提示模板）

分发代码质量审查者子代理时使用此模板。

**目的：** 验证实现构建良好（干净、可测试、可维护）

**仅在规范合规审查通过后分发。**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**除了标准代码质量问题，审查者还应检查：**
- 每个文件是否有一个清晰的职责和定义良好的接口？
- 单元是否分解以便可以独立理解和测试？
- 实现是否遵循计划中的文件结构？
- 此实现是否创建了已经很大的新文件，或显著增长了现有文件？（不要标记已有文件大小——关注此更改贡献了什么。）

**代码审查者返回：** Strengths（优点）、Issues（问题）（Critical/Important/Minor）、Assessment（评估）
