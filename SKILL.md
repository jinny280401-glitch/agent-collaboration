---
name: agent-collaboration
description: 多个 AI agent 协作时的治理协议与跨 agent 可复用教训库。当需要协调多个 agent、通过共享记忆(Engram)传递经验、运行 Builder/Reviewer 审查回路、做公开发布/披露决策,或判断"声明 vs 能力""证据 vs 结论"边界时使用。内含协作协议 2 篇 + 195 条已蒸馏的跨 agent 教训(按 7 主题归类)。
---

# Agent Collaboration

多个异构 AI agent(Claude Code、Codex、ChatGPT…)协作时,一个 agent 用真实成本换来的教训,如何被另一个 agent 继承,而不必重新经历整个失败过程。

## 何时用这个 skill

- 协调多个 agent 分工(谁 build、谁 review、谁有权下 verdict)
- 通过共享记忆总线(Engram)写入/读取跨 agent 教训
- 跑 Builder → Reviewer 审查回路,或做跨 session 交接
- 做公开发布 / 披露审计 / release gate 决策
- 判断能力边界:声明 ≠ 能力、测试全绿 ≠ 生产可用、看到产物 ≠ 有权推进、冻结的产物 ≠ 执行的产物

## 核心原则(先读这几条)

1. **结论不得宽于证据** —— downstream claim ≤ min(upstream)。
2. **Gate 必须在 prompt 前**,不能靠输出后评审补——缺陷没 surface 进 prompt,生成文本里就没痕迹了。
3. **授权绑定到确切 artifact,不是容器** —— 批准的是"这一份被审计过的内容",不是"这个仓库/分支可以发布"。
4. **verdict 身份分离** —— Builder 修复后不得自宣通过,必须独立 Reviewer 回合。
5. **只共享 message 是群聊;共享蒸馏过的 lesson 才像一个组织。**

## 如何导航内容

### 协作协议 `protocols/`
- `multi-agent-collaboration.md` — Builder/Reviewer 回路、verdict 分离、并行 session 权威边界、supersession、claim 不洗白、产物三分类
- `engram-cross-agent-memory.md` — 共享记忆架构、各 agent 接入、"写教训不写状态"规范、分类器实测规律

### 教训库 `lessons/`(195 条,7 主题)
按当前任务选对应主题文件读:

| 主题文件 | 什么时候读 |
|---|---|
| `evidence-and-claim-boundary.md` | 下结论、引用数据、判断声明是否成立时 |
| `freeze-and-state-discipline.md` | 处理冻结窗口、SHA pin、不可变 session 时 |
| `capability-maturity.md` | 判断"做完了没""能力到哪一级"时 |
| `trust-gate-and-admission.md` | 设计准入闸、判断 policy 是否可强制执行时 |
| `multi-agent-collaboration.md` | 多 agent 分工、聚合结论时 |
| `ops-and-infra.md` | 跨仓库 review、SSH/代理、凭据卫生时 |
| `other-reusable.md` | 其他工程/工作方法 |

每条 lesson 固定三段:**Lesson(可复用的那句话)+ Root Cause(为什么)+ How to Apply(下次怎么做)**。

## 用法

遇到上述场景时,先读 `protocols/` 对应协议定边界,再查 `lessons/` 对应主题找有没有同类坑已经被别的 agent 踩过并蒸馏成教训。应用教训时,保持它的三段结构;若本次又踩出新坑,按 `engram-cross-agent-memory.md` 的写入规范把新教训写回共享总线。
