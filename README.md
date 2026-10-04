# Engram — a shared learning bus for heterogeneous AI agents

> 多个异构 AI agent(Claude Code、Codex、ChatGPT…)协作时,**一个 agent 用真实成本换来的教训,如何被另一个 agent 继承,而不必重新经历整个失败过程。**

Engram 不是聊天记忆。聊天记忆保存"说过什么";Engram 保存"学到了什么",并把它跨 agent、跨任务传播。

---

## 为什么:三类信息,只有一类值得跨 agent 继承

协作中流动的信息有三类,混在一起是大多数记忆系统失败的根因:

| 类型 | 是什么 | 例子 | 寿命 |
|---|---|---|---|
| **Message** | 一次通信 | "这次实验 38/40" | 一次性 |
| **State** | 当前世界/项目状态 | "某 implementation 仍 OPEN" | 到状态改变为止 |
| **Lesson** | 可跨任务、跨 agent 复用的经验 | "必须执行的产品义务,不应交给概率模型重新判断" | 长期 |

Message 过期得最快,State 会被下一次更新推翻,**只有 Lesson 换一个项目、换一个 agent 仍然成立**。

Engram 只重点保存和传播第三类。它会主动拒收 State 快照——这不是限制,是定位:**一个记忆库一旦开始堆项目状态,就退化成了日志,不再是学习系统。**

---

## Lesson 的形状

每条 lesson 固定三段,缺一不可:

```
Lesson      —— 可复用的那句话(换个项目还成立)
Root Cause  —— 为什么会这样(不是"发生了什么",是"为什么")
How to Apply —— 下次遇到同类情况怎么做
```

反例是把 repository 变成项目状态转储:"X 模块已上线""Y 任务 CLOSED"——这些是 State,不是 Lesson,下一个 agent 读了什么也学不到。

---

## 多 Agent 学习回路

单个 agent 的能力不是瓶颈。瓶颈是:A 踩的坑,B 要不要再踩一遍。

```
   Agent / Experiment
          │
          ▼
   Counterexample / Failure        ← 付出真实成本
          │
          ▼
   Lesson Extraction               ← 从失败里蒸馏出可复用内核
          │
          ▼
      ┌─────────┐
      │ Engram  │                  ← 跨 agent 共享总线
      └─────────┘
          │
          ▼
   Another Agent Retrieval         ← 另一个 agent 继承,无需重历失败
          │
          ▼
      New Work
```

核心观点:

> **If agents only share messages, they are a group chat.**
> **When they share distilled lessons, they start to behave like an organization.**
>
> 只共享消息,一群 agent 就是个群聊。
> 当它们共享蒸馏过的教训,才开始像一个组织那样运作。

---

## 仓库内容

### 协作协议 (`protocols/`)

- **[Multi-Agent 协作协议](protocols/multi-agent-collaboration.md)** — Builder/Reviewer 回路、verdict 身份分离、并行 session 权威边界、supersession 收口、claim 不得跨 agent 洗白、产物三分类。
- **[Engram 跨 Agent 记忆](protocols/engram-cross-agent-memory.md)** — 本地记忆服务架构、各 agent 接入方式、"写教训不写状态"的写入规范、分类器实测规律。

### 教训库 (`lessons/`)

从跨 agent 记忆库导出、按主题归类、已脱敏:

| 主题 | 一句话 |
|---|---|
| [证据与声明边界](lessons/evidence-and-claim-boundary.md) | 结论不得宽于证据;声明≠能力;fallback≠真数据 |
| [冻结与状态纪律](lessons/freeze-and-state-discipline.md) | closed session immutable;SHA pin 证明 identity 非 durability |
| [能力成熟度链](lessons/capability-maturity.md) | Exists/Invoked/Succeeded/Authorized/Production Proven 五级 |
| [Trust Gate 与准入](lessons/trust-gate-and-admission.md) | gate 必须在 prompt 前;declarative policy≠enforceable gate |
| [多 Agent 协作](lessons/multi-agent-collaboration.md) | downstream claim≤min(upstream);授权信封在外部状态 |
| [其他可复用教训](lessons/other-reusable.md) | 跨项目可复用的工程/工作方法 |
| [运维与基础设施](lessons/ops-and-infra.md) | 跨仓库 review 四元组;代理下 SSH 需 DIRECT |

> 教训源自真实协作踩坑后的沉淀,非理论推演。已脱敏公开。

---

*A learning bus, not a group chat.*
