# 基于用户反馈的技能改进

**日期：** 2025-11-28
**状态：** 草稿
**来源：** 两个在实际开发场景中使用 superpowers 的 Claude 实例

---

## 执行摘要

两个 Claude 实例提供了来自实际开发会话的详细反馈。他们的反馈揭示了**当前技能的系统性差距**，这些差距导致本可以预防的 bug 被发布，尽管遵循了技能。

**关键洞察：** 这些是问题报告，而不仅仅是解决方案提案。问题是真实的；解决方案需要仔细评估。

**关键主题：**
1. **验证差距** - 我们验证操作成功但不验证它们是否达到预期结果
2. **流程卫生** - 后台进程在子代理中积累并干扰
3. **上下文优化** - 子代理获得太多不相关信息
4. **自我反思缺失** - 没有提示在交接前批评自己的工作
5. **模拟安全** - 模拟可能与接口漂移而无法检测
6. **技能激活** - 技能存在但未被阅读/使用

---

## 发现的问题

### 问题 1：配置变更验证差距

**发生了什么：**
- 子代理测试"OpenAI 集成"
- 设置 `OPENAI_API_KEY` 环境变量
- 获得状态码 200 响应
- 报告"OpenAI 集成工作正常"
- **但是** 响应包含 `"model": "claude-sonnet-4-20250514"` - 实际使用的是 Anthropic

**根本原因：**
`verification-before-completion` 检查操作成功，但不检查结果是否反映预期的配置变更。

**影响：** 高 - 集成测试的虚假信心，bug 发布到生产环境

**示例失败模式：**
- 切换 LLM 提供商 → 验证状态码 200 但不检查模型名称
- 启用功能标志 → 验证没有错误但不检查功能是否激活
- 更改环境 → 验证部署成功但不检查环境变量

---

### 问题 2：后台进程积累

**发生了什么：**
- 在会话期间调度了多个子代理
- 每个都启动了后台服务器进程
- 进程积累（4+ 服务器运行）
- 过期进程仍绑定到端口
- 后续 E2E 测试遇到配置错误的过期服务器
- 混淆/错误的测试结果

**根本原因：**
子代理是无状态的——不了解之前子代理的进程。没有清理协议。

**影响：** 中高 - 测试命中错误的服务器，虚假通过/失败，调试混淆

---

### 问题 3：子代理提示中的上下文膨胀

**发生了什么：**
- 标准方法：给子代理完整的计划文件阅读
- 实验：只给任务 + 模式 + 文件 + 验证命令
- 结果：更快、更专注，单次尝试完成更常见

**根本原因：**
子代理在无关的计划部分上浪费 token 和注意力。

**影响：** 中 - 执行更慢，更多失败尝试

**有效的方法：**
```
You are adding a single E2E test to packnplay's test suite.

**Your task:** Add `TestE2E_FeaturePrivilegedMode` to `pkg/runner/e2e_test.go`

**What to test:** A local devcontainer feature that requests `"privileged": true`
in its metadata should result in the container running with `--privileged` flag.

**Follow the exact pattern of TestE2E_FeatureOptionValidation** (at the end of the file)

**After writing, run:** `go test -v ./pkg/runner -run TestE2E_FeaturePrivilegedMode -timeout 5m`
```

---

### 问题 4：交接前无自我反思

**发生了什么：**
- 添加了自我反思提示："用新的眼光看待你的工作——有什么可以改进的？"
- 任务 5 的实施者发现失败的测试是由于实现 bug，而不是测试 bug
- 追溯到第 99 行：`strings.Join(metadata.Entrypoint, " ")` 创建了无效的 Docker 语法
- 没有自我反思，只会报告"测试失败"而没有根本原因

**根本原因：**
实施者不会自然地退后一步，在报告完成之前批评自己的工作。

**影响：** 中 - 本可以由实施者发现的 bug 被交给审查者

---

### 问题 5：模拟-接口漂移

**发生了什么：**
```typescript
// 接口定义了 close()
interface PlatformAdapter {
  close(): Promise<void>;
}

// 代码（有 BUG）调用 cleanup()
await adapter.cleanup();

// 模拟（匹配 BUG）定义了 cleanup()
vi.mock('web-adapter', () => ({
  WebAdapter: vi.fn().mockImplementation(() => ({
    cleanup: vi.fn().mockResolvedValue(undefined),  // 错误！
  })),
}));
```
- 测试通过
- 运行时崩溃："adapter.cleanup is not a function"

**根本原因：**
模拟来自 buggy 代码调用的内容，而不是接口定义。TypeScript 无法捕获具有错误方法名的内联模拟。

**影响：** 高 - 测试给出虚假信心，运行时崩溃

**为什么测试反模式没有防止这个：**
该技能涵盖了测试模拟行为和不理解就模拟，但没有"从接口派生模拟，而不是实现"这个特定模式。

---

### 问题 6：代码审查者文件访问

**发生了什么：**
- 调度了代码审查者子代理
- 找不到测试文件："该文件似乎不在仓库中"
- 文件实际存在
- 审查者不知道需要先显式读取它

**根本原因：**
审查者提示不包含显式文件读取说明。

**影响：** 低-中 - 审查失败或不完整

---

### 问题 7：修复工作流程延迟

**发生了什么：**
- 实施者在自我反思期间发现 bug
- 实施者知道修复方法
- 当前工作流程：报告 → 我调度修复者 → 修复者修复 → 我验证
- 额外的往返增加了延迟而没有增加价值

**根本原因：**
当实施者已经诊断出问题时，实施者和修复者角色之间的刚性分离。

**影响：** 低 - 延迟，但没有正确性问题

---

### 问题 8：技能未被阅读

**发生了什么：**
- `testing-anti-patterns` 技能存在
- 人类和子代理在编写测试之前都没有阅读它
- 本可以防止一些问题（虽然不是全部——见问题 5）

**根本原因：**
没有强制子代理阅读相关技能。没有提示包含技能阅读。

**影响：** 中 - 如果不使用，技能投资就浪费了

---

## 建议的改进

### 1. verification-before-completion：添加配置变更验证

**添加新部分：**

```markdown
## Verifying Configuration Changes

When testing changes to configuration, providers, feature flags, or environment:

**Don't just verify the operation succeeded. Verify the output reflects the intended change.**

### Common Failure Pattern

Operation succeeds because *some* valid config exists, but it's not the config you intended to test.

### Examples

| Change | Insufficient | Required |
|--------|-------------|----------|
| Switch LLM provider | Status 200 | Response contains expected model name |
| Enable feature flag | No errors | Feature behavior actually active |
| Change environment | Deploy succeeds | Logs/vars reference new environment |
| Set credentials | Auth succeeds | Authenticated user/context is correct |

### Gate Function

```
BEFORE claiming configuration change works:

1. IDENTIFY: What should be DIFFERENT after this change?
2. LOCATE: Where is that difference observable?
   - Response field (model name, user ID)
   - Log line (environment, provider)
   - Behavior (feature active/inactive)
3. RUN: Command that shows the observable difference
4. VERIFY: Output contains expected difference
5. ONLY THEN: Claim configuration change works

Red flags:
  - "Request succeeded" without checking content
  - Checking status code but not response body
  - Verifying no errors but not positive confirmation
```

**Why this works:**
Forces verification of INTENT, not just operation success.

---

### 2. subagent-driven-development：添加 E2E 测试的流程卫生

**添加新部分：**

```markdown
## Process Hygiene for E2E Tests

When dispatching subagents that start services (servers, databases, message queues):

### Problem

Subagents are stateless - they don't know about processes started by previous subagents. Background processes persist and can interfere with later tests.

### Solution

**Before dispatching E2E test subagent, include cleanup in prompt:**

```
BEFORE starting any services:
1. Kill existing processes: pkill -f "<service-pattern>" 2>/dev/null || true
2. Wait for cleanup: sleep 1
3. Verify port free: lsof -i :<port> && echo "ERROR: Port still in use" || echo "Port free"

AFTER tests complete:
1. Kill the process you started
2. Verify cleanup: pgrep -f "<service-pattern>" || echo "Cleanup successful"
```

### Example

```
Task: Run E2E test of API server

Prompt includes:
"Before starting the server:
- Kill any existing servers: pkill -f 'node.*server.js' 2>/dev/null || true
- Verify port 3001 is free: lsof -i :3001 && exit 1 || echo 'Port available'

After tests:
- Kill the server you started
- Verify: pgrep -f 'node.*server.js' || echo 'Cleanup verified'"
```

### Why This Matters

- Stale processes serve requests with wrong config
- Port conflicts cause silent failures
- Process accumulation slows system
- Confusing test results (hitting wrong server)
```

**Trade-off analysis:**
- Adds boilerplate to prompts
- But prevents very confusing debugging
- Worth it for E2E test subagents

---

### 3. subagent-driven-development：添加精简上下文选项

**修改第 2 步：使用子代理执行任务**

**之前：**
```
Read that task carefully from [plan-file].
```

**之后：**
```
## Context Approaches

**Full Plan (default):**
Use when tasks are complex or have dependencies:
```
Read Task N from [plan-file] carefully.
```

**Lean Context (for independent tasks):**
Use when task is standalone and pattern-based:
```
You are implementing: [1-2 sentence task description]

File to modify: [exact path]
Pattern to follow: [reference to existing function/test]
What to implement: [specific requirement]
Verification: [exact command to run]

[Do NOT include full plan file]
```

**Use lean context when:**
- Task follows existing pattern (add similar test, implement similar feature)
- Task is self-contained (doesn't need context from other tasks)
- Pattern reference is sufficient (e.g., "follow TestE2E_FeatureOptionValidation")

**Use full plan when:**
- Task has dependencies on other tasks
- Requires understanding of overall architecture
- Complex logic that needs context
```

**Example:**
```
Lean context prompt:

"You are adding a test for privileged mode in devcontainer features.

File: pkg/runner/e2e_test.go
Pattern: Follow TestE2E_FeatureOptionValidation (at end of file)
Test: Feature with `"privileged": true` in metadata results in `--privileged` flag
Verify: go test -v ./pkg/runner -run TestE2E_FeaturePrivilegedMode -timeout 5m

Report: Implementation, test results, any issues."
```

**Why this works:**
Reduces token usage, increases focus, faster completion when appropriate.

---

### 4. subagent-driven-development：添加自我反思步骤

**修改第 2 步：使用子代理执行任务**

**添加到提示模板：**

```
When done, BEFORE reporting back:

Take a step back and review your work with fresh eyes.

Ask yourself:
- Does this actually solve the task as specified?
- Are there edge cases I didn't consider?
- Did I follow the pattern correctly?
- If tests are failing, what's the ROOT CAUSE (implementation bug vs test bug)?
- What could be better about this implementation?

If you identify issues during this reflection, fix them now.

Then report:
- What you implemented
- Self-reflection findings (if any)
- Test results
- Files changed
```

**Why this works:**
Catches bugs implementer can find themselves before handoff. Documented case: identified entrypoint bug through self-reflection.

**Trade-off:**
Adds ~30 seconds per task, but catches issues before review.

---

### 5. requesting-code-review：添加显式文件读取

**修改代码审查者模板：**

**在开头添加：**

```markdown
## Files to Review

BEFORE analyzing, read these files:

1. [List specific files that changed in the diff]
2. [Files referenced by changes but not modified]

Use Read tool to load each file.

If you cannot find a file:
- Check exact path from diff
- Try alternate locations
- Report: "Cannot locate [path] - please verify file exists"

DO NOT proceed with review until you've read the actual code.
```

**Why this works:**
Explicit instruction prevents "file not found" issues.

---

### 6. testing-anti-patterns：添加模拟-接口漂移反模式

**添加新的反模式 6：**

```markdown
## Anti-Pattern 6: Mocks Derived from Implementation

**The violation:**
```typescript
// Code (BUGGY) calls cleanup()
await adapter.cleanup();

// Mock (MATCHES BUG) has cleanup()
const mock = {
  cleanup: vi.fn().mockResolvedValue(undefined)
};

// Interface (CORRECT) defines close()
interface PlatformAdapter {
  close(): Promise<void>;
}
```

**Why this is wrong:**
- Mock encodes the bug into the test
- TypeScript can't catch inline mocks with wrong method names
- Test passes because both code and mock are wrong
- Runtime crashes when real object is used

**The fix:**
```typescript
// ✅ GOOD: Derive mock from interface

// Step 1: Open interface definition (PlatformAdapter)
// Step 2: List methods defined there (close, initialize, etc.)
// Step 3: Mock EXACTLY those methods

const mock = {
  initialize: vi.fn().mockResolvedValue(undefined),
  close: vi.fn().mockResolvedValue(undefined),  // From interface!
};

// Now test FAILS because code calls cleanup() which doesn't exist
// That failure reveals the bug BEFORE runtime
```

### Gate Function

```
BEFORE writing any mock:

  1. STOP - Do NOT look at the code under test yet
  2. FIND: The interface/type definition for the dependency
  3. READ: The interface file
  4. LIST: Methods defined in the interface
  5. MOCK: ONLY those methods with EXACTLY those names
  6. DO NOT: Look at what your code calls

  IF your test fails because code calls something not in mock:
    ✅ GOOD - The test found a bug in your code
    Fix the code to call the correct interface method
    NOT the mock

  Red flags:
    - "I'll mock what the code calls"
    - Copying method names from implementation
    - Mock written without reading interface
    - "The test is failing so I'll add this method to the mock"
```

**Detection:**

When you see runtime error "X is not a function" and tests pass:
1. Check if X is mocked
2. Compare mock methods to interface methods
3. Look for method name mismatches
```

**Why this works:**
Directly addresses the failure pattern from feedback.

---

### 7. subagent-driven-development：要求测试子代理阅读技能

**在涉及测试的任务提示模板中添加：**

```markdown
BEFORE writing any tests:

1. Read testing-anti-patterns skill:
   Use Skill tool: superpowers:testing-anti-patterns

2. Apply gate functions from that skill when:
   - Writing mocks
   - Adding methods to production classes
   - Mocking dependencies

This is NOT optional. Tests that violate anti-patterns will be rejected in review.
```

**Why this works:**
Ensures skills are actually used, not just exist.

**Trade-off:**
Adds time to each task, but prevents entire classes of bugs.

---

### 8. subagent-driven-development：允许实施者修复自我识别的问题

**修改第 2 步：**

**当前：**
```
Subagent reports back with summary of work.
```

**建议：**
```
Subagent performs self-reflection, then:

IF self-reflection identifies fixable issues:
  1. Fix the issues
  2. Re-run verification
  3. Report: "Initial implementation + self-reflection fix"

ELSE:
  Report: "Implementation complete"

Include in report:
- Self-reflection findings
- Whether fixes were applied
- Final verification results
```

**Why this works:**
Reduces latency when implementer already knows the fix. Documented case: would have saved one round-trip for entrypoint bug.

**Trade-off:**
Slightly more complex prompt, but faster end-to-end.

---

## 实现计划

### 阶段 1：高影响、低风险（先做）

1. **verification-before-completion：配置变更验证**
   - 清晰添加，不更改现有内容
   - 解决高影响问题（测试中的虚假信心）
   - 文件：`skills/verification-before-completion/SKILL.md`

2. **testing-anti-patterns：模拟-接口漂移**
   - 添加新的反模式，不修改现有内容
   - 解决高影响问题（运行时崩溃）
   - 文件：`skills/testing-anti-patterns/SKILL.md`

3. **requesting-code-review：显式文件读取**
   - 模板的简单添加
   - 修复具体问题（审查者找不到文件）
   - 文件：`skills/requesting-code-review/SKILL.md`

### 阶段 2：适度变更（仔细测试）

4. **subagent-driven-development：流程卫生**
   - 添加新部分，不更改工作流程
   - 解决中高影响（测试可靠性）
   - 文件：`skills/subagent-driven-development/SKILL.md`

5. **subagent-driven-development：自我反思**
   - 更改提示模板（更高风险）
   - 但有记录表明可以捕获 bug
   - 文件：`skills/subagent-driven-development/SKILL.md`

6. **subagent-driven-development：技能阅读要求**
   - 添加提示开销
   - 但确保技能实际被使用
   - 文件：`skills/subagent-driven-development/SKILL.md`

### 阶段 3：优化（先验证）

7. **subagent-driven-development：精简上下文选项**
   - 添加复杂性（两种方法）
   - 需要验证它不会造成混淆
   - 文件：`skills/subagent-driven-development/SKILL.md`

8. **subagent-driven-development：允许实施者修复**
   - 更改工作流程（更高风险）
   - 优化，不是 bug 修复
   - 文件：`skills/subagent-driven-development/SKILL.md`

---

## 开放问题

1. **精简上下文方法：**
   - 我们应该使其成为基于模式的任务的默认方法吗？
   - 我们如何决定使用哪种方法？
   - 过于精简而错过重要上下文的风险？

2. **自我反思：**
   - 这会显著减慢简单任务吗？
   - 它应该只适用于复杂任务吗？
   - 我们如何防止"反思疲劳"使其变得机械？

3. **流程卫生：**
   - 这应该在 subagent-driven-development 中还是单独的技能中？
   - 它适用于 E2E 测试之外的其他工作流程吗？
   - 我们如何处理进程应该持续存在的情况（开发服务器）？

4. **技能阅读执行：**
   - 我们应该要求所有子代理阅读相关技能吗？
   - 我们如何防止提示变得太长？
   - 过度文档化而失去焦点的风险？

---

## 成功指标

我们如何知道这些改进有效？

1. **配置验证：**
   - "测试通过但使用了错误配置"的实例为零
   - Jesse 不会说"实际上你没有测试你认为的内容"

2. **流程卫生：**
   - "测试命中错误服务器"的实例为零
   - E2E 测试运行期间没有端口冲突错误

3. **模拟-接口漂移：**
   - "测试通过但运行时缺少方法崩溃"的实例为零
   - 模拟和接口之间没有方法名不匹配

4. **自我反思：**
   - 可衡量：实施者报告是否包含自我反思发现？
   - 定性：更少的 bug 进入代码审查？

5. **技能阅读：**
   - 子代理报告引用技能门函数
   - 代码审查中更少的反模式违规

---

## 风险和缓解

### 风险：提示膨胀
**问题：** 添加所有这些要求会使提示势不可挡
**缓解：**
- 分阶段实现（不要一次添加所有内容）
- 使一些添加有条件（E2E 卫生仅适用于 E2E 测试）
- 考虑不同任务类型的模板

### 风险：分析瘫痪
**问题：** 过多的反思/验证减慢执行
**缓解：**
- 保持门函数快速（秒，而不是分钟）
- 最初使精简上下文可选
- 监控任务完成时间

### 风险：虚假安全感的错误感觉
**问题：** 遵循清单不能保证正确性
**缓解：**
- 强调门函数是最少，不是最多
- 在技能中保持"使用判断力"语言
- 记录技能捕获常见失败，不是所有失败

### 风险：技能分歧
**问题：** 不同技能给出相互冲突的建议
**缓解：**
- 审查所有技能的变更以保持一致性
- 记录技能如何交互（集成部分）
- 在部署前用真实场景测试

---

## 建议

**立即执行阶段 1：**
- verification-before-completion：配置变更验证
- testing-anti-patterns：模拟-接口漂移
- requesting-code-review：显式文件读取

**在最终确定之前与 Jesse 一起测试阶段 2：**
- 获取关于自我反思影响的反馈
- 验证流程卫生方法
- 确认技能阅读要求是否值得开销

**持有阶段 3 等待验证：**
- 精简上下文需要真实世界测试
- 实施者-修复工作流程变更需要仔细评估

这些变更解决了用户记录的真实问题，同时最大限度地降低了使技能变得更糟的风险。