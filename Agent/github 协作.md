# GPT × Codex × GitHub 协作

> 核心分工：GPT 负责规划与验收，Codex 负责实现与修复，我负责最终确认。

## 一句话流程

```text
GPT 规划并创建 Issue
  → 本地 Codex 实现并创建 Ready PR
  → 云端 Codex 自动 Review
  → 本地 Codex 处理有效意见
  → GPT 最终验收
  → 本地 Codex 收口文档
  → 我确认合并并清理分支
```

## 关键规则

- 一个 Issue 对应一个 PR；一个 PR 可以有多个 commit。
- 实现完成且验证通过：创建 **Ready PR**，自动触发云端 Codex Review。
- 尚未完成、被阻塞或验证失败：使用 **Draft PR**。
- Ready 只表示“可以审核”，不表示验收通过，也不允许自动合并。
- 修复了实质性代码问题后，再用 `@codex review` 请求复审。
- 只回复评论、关闭 thread 或更新收口文档时，通常不用再次 Review。
- GPT 给出验收意见；我确认后，Codex 才能更新正式任务状态。
- 合并、关闭 Issue 和删除分支必须由我明确授权。

## 完整 Demo

### 1. GPT 规划任务

**我对 GPT 说：**

> 请根据当前项目进度规划下一个任务，确认范围和不做事项，并创建 GitHub Issue。

**GPT 回复：**

> 已创建 Issue #12，目标、验收标准、任务边界和学习问题已经写清楚。

### 2. 本地 Codex 实现

**我对 Codex 说：**

> 完成 Issue #12。

**Codex 会：**

1. 读取 Issue 和项目任务文档。
2. 从最新 `master` 创建 `codex/issue-12-xxx` 分支。
3. 实现、测试并记录“已实现、待验收”。
4. commit、push，创建 Ready PR。
5. 停止在等待 Review 的状态，不自行合并。

**Codex 回复示例：**

> Issue #12 已实现，PR #15 已创建并进入 Ready 状态。验证通过，当前等待 Codex Review；未合并、未关闭 Issue。

### 3. 云端 Codex Review

Ready PR 创建后，云端 Codex 自动审核。若发现问题，会在 PR 中留下 Review thread。

**我对本地 Codex 说：**

> 处理 PR #15 的 Review。

**本地 Codex 会：**

1. 拉取尚未解决的审核意见。
2. 先解释问题是否成立、是否属于当前 Issue。
3. 等我确认后修复有效问题。
4. 验证、commit、push 回原 PR。
5. 若改了实质性代码，发送 `@codex review` 请求复审。

**Codex 回复示例：**

> 两条意见中，一条需要修复；另一条技术现象成立，但超出 Issue #12 范围。有效问题已修复并请求复审，PR 仍未合并。

### 4. GPT 最终验收

当 Review 已处理、代码验证通过后：

**我把 PR 结果交给 GPT：**

> 请对 Issue #12 和 PR #15 做最终验收，判断是否通过，并明确收口文档范围。

**GPT 回复示例：**

> 验收通过。代码无阻塞问题，可以更新 Task 状态；暂不授权合并或开始下一任务。

### 5. Codex 收口任务状态

**我对本地 Codex 说：**

> GPT 已确认 Issue #12 验收通过，我也确认通过，请收口。

**Codex 会：**

- 更新对应任务、路线图和工作记录。
- 把状态改为 Completed，但不提前开始下一任务。
- commit、push 到原 PR，并确认 Review threads 已解决。

### 6. 合并与清理

**我对本地 Codex 说：**

> 合并 PR #15 并清理分支。

**Codex 会：**

- 合并 PR，确认 Issue 自动关闭。
- 同步本地 `master`。
- 安全删除远程和本地任务分支。
- 确认工作区干净、与远程一致。

## 小改动怎么处理

| 场景 | 做法 |
| --- | --- |
| typo、文案、普通样式或小型 docs 修正 | 可明确要求直接在 `master` 提交并 push |
| 工作流、AGENTS.md、Skill 等需要审核的小改动 | 不建 Issue，创建轻量分支和 Ready PR |
| 功能、API、数据库、Agent Runtime、Tool Calling 等正式任务 | 必须走完整 Issue 工作流 |

轻量 PR 的指令示例：

> 这是一次不创建 Issue 的小型工作流治理修改。请创建 `codex/update-workflow` 分支，修改并验证后提交、push，创建 Ready PR；不要更新正式任务状态，不要合并。

## 我只需要记住

```text
规划找 GPT → 实现找本地 Codex → Review 由云端 Codex 做
修复仍找本地 Codex → 验收再找 GPT → 最终合并由我确认
```
