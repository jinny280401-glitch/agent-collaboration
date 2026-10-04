# Agent Collaboration

多个 AI agent(Claude Code、Codex、ChatGPT 等)长期在同一批项目上协作,沉淀下来的**跨 agent 记忆机制**和**协作协议**。

这里收的不是某个项目的进展,而是**可复用的教训**——那些"换一个项目、换一个 agent 还会再踩"的坑,以及避免它们的协议。核心是一组反直觉但反复被验证的边界:

> 声明 ≠ 能力 · 测试全绿 ≠ 生产可用 · 看到产物 ≠ 有权推进 · 冻结的产物 ≠ 执行的产物 · 结论不得宽于证据

## 内容

### 协作协议 (`protocols/`)

- **[Multi-Agent 协作协议](protocols/multi-agent-collaboration.md)** — Builder/Reviewer 回路、verdict 身份分离、并行 session 权威边界、supersession 收口、claim 不得跨 agent 洗白、产物三分类。
- **[Engram 跨 Agent 记忆](protocols/engram-cross-agent-memory.md)** — 本地记忆服务架构、各 agent 接入方式、"写教训不写状态"的写入规范、分类器实测规律。

### 教训库 (`lessons/`)

从跨 agent 记忆库导出、按主题归类、已脱敏(去掉生产 IP、本地路径、内部项目代号):

| 主题 | 条数 | 一句话 |
|---|---|---|
| [证据与声明边界](lessons/evidence-and-claim-boundary.md) | 79 | 结论不得宽于证据;声明≠能力;fallback≠真数据 |
| [冻结与状态纪律](lessons/freeze-and-state-discipline.md) | 33 | closed session immutable;SHA pin 证明 identity 非 durability |
| [能力成熟度链](lessons/capability-maturity.md) | 30 | Exists/Invoked/Succeeded/Authorized/Production Proven 五级 |
| [Trust Gate 与准入](lessons/trust-gate-and-admission.md) | 24 | gate 必须在 prompt 前;declarative policy≠enforceable gate |
| [多 Agent 协作](lessons/multi-agent-collaboration.md) | 14 | downstream claim≤min(upstream);授权信封在外部状态 |
| [其他可复用教训](lessons/other-reusable.md) | 14 | 不属以上主题、但跨项目可复用的工程/工作方法 |
| [运维与基础设施](lessons/ops-and-infra.md) | 4 | 跨仓库 review 四元组;代理下 SSH 需 DIRECT |

> 共 198 条(2 条空记录已剔除)。归类按关键词自动路由,少数条目可能落在相邻主题。

## 一句话总览

单个 agent 的能力不是瓶颈,**agent 之间如何传递证据、如何分离判定、如何不互相污染状态**才是。这套协议的目的,是让"已经双人验证过"这句话名副其实,而不是一个错觉。

---

*教训源自真实协作踩坑后的沉淀,非理论推演。已脱敏公开。*
