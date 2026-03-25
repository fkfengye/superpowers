# Plan Document Reviewer Prompt Template 计划文档审查者提示模板

Use this template when dispatching a plan document reviewer subagent.
在分派计划文档审查者 subagent 时使用此模板。

**Purpose:** Verify the plan is complete, matches the spec, and has proper task decomposition.
**目的：** 验证计划完整、符合规范，并有适当的任务分解。

**Dispatch after:** The complete plan is written.
**分派时机：** 完整计划写完之后。

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Verify this plan is complete and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers spec requirements, no major scope creep |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    contradictory steps, placeholder content, or tasks so vague they can't be acted on.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
**审查者返回：** 状态、问题（如果有）、建议
