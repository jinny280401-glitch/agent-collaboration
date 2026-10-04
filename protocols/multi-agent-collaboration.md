# Multi-Agent 协作协议

多个 AI agent(Claude Code、Codex、ChatGPT 等)在同一个项目上协作时,最脆弱的不是单个 agent 的能力,而是**它们之间如何传递证据、如何分离判定、如何不互相污染状态**。

这里的协议都来自真实踩坑后沉淀的教训,不是理论推演。核心是一句话:

> **一个 agent 看到了某个产物 ≠ 它有权解释、修改或推进那个产物。**

---

## 1. Builder → Reviewer 回路

角色协议,不是固定身份。下一轮可能 Codex 当 Builder、Claude Code 当 Reviewer。要长期保留的是**角色之间的契约**,不是谁扮演谁。

```
Builder
  ↓ 把全部证据物化成文件
Reviewer
  ↓ 结构化 findings / verdict
Builder
  ↓ 采纳 / 修复 / 继续
Reviewer
  ↓ re-review
Checkpoint
```

规则:

1. **Builder 必须先把证据物化成文件**再交接——不是 stdout,不是对话上下文。跨 session 的对话上下文不保证留存,之前多次交接失败都是因为 Reviewer 试图重跑命令或依赖已经消失的对话。
2. **Reviewer 的 findings 必须结构化**:severity 分级 + 具体修正点。
3. **用一个共享的 plan 文件做协调产物**,而不只是任务清单。回路状态(哪步完成/进行中/待办)必须显式写在 plan 或 tracker 里。
4. 证据文件放在项目专用目录,交接时 pin 住 hash 防漂移。

---

## 2. Verdict 身份分离(最关键的一条)

**修复之后的判定,必须由独立的 Reviewer 回合宣布,Builder 不得自宣通过。**

多 agent 协作最脆弱的环节不是初始审查,而是 **fix 之后的判定**。Builder 天然倾向于认定自己的修复解决了问题。如果没有独立的 re-review 回合,verdict 就是 Builder 自评,整个审查回路退化成普通 code review——

> 这比没有审查更危险,因为它制造了"已经双人验证过"的错觉,实际只过了一个人的判断。

正确流程:

1. fix 之后插入**显式身份切换**:Builder 提交 fix diff + 说明,然后停下,等待独立回合。
2. 给 Reviewer 的 re-review 请求必须是 **Evidence Package 格式**:target artifact hash + 可读文件列表(scope)+ 不相关文件(forbidden scope),防止 Reviewer 会话 drift 到无关文件。
3. 三项同时留存才算数:初始 finding + fix diff + re-review 结果。缺任一项,该 verdict 判为 `NOT PROVEN`。
4. **Reviewer 能力存在 ≠ Reviewer 每次都锁定正确对象**。审查者上下文隔离是独立的下一层问题。

---

## 3. 并行 Session 的权威边界

多个 agent session 可能悄悄收敛到同一个冻结窗口上。共享 git 工作树 + 共享记忆上下文时,**commit 作者身份无法区分 session 归属**。

```
reviewer 看到 artifact
        ≠
reviewer 拥有 window
```

恢复任何冻结的多 session 窗口前:

1. **先确认归属**——通过 peer 询问(ListAgents / SendMessage)。
2. 若另一个 session 声称拥有,把窗口当**只读**。
3. 只向 owner 转发**非行动性**观察(覆盖缺口、不一致),不自己动手。
4. **不要**:改冻结声明或分类 / 跑探针采证 / commit·push·deploy / 声称或暗示对窗口的权威。

证据边界和权威边界是两条正交的线,**两条都跨过,结论才有效**:

```
证据边界              权威边界
artifact 存在          reviewer 看到 artifact
      ≠                      ≠
能力被证明             reviewer 拥有 window
       \                    /
        ↓                  ↓
          有效结论
```

---

## 4. Supersession(状态收口)

当你发现自己在同一个窗口里写**第三份**关于同一主题的治理文档时,停下来收口。

1. 收口文档在顶部声明自己 **AUTHORITATIVE**。
2. **逐一列出被取代的文档名**,带显式反向引用。
3. **说明为什么现在收口**("why now",不只是"what changed")。
4. 在索引(MEMORY.md)里给每条被取代项追加 `SUPERSEDED by [[收口文档]]`。
5. **不要删除被取代的文件**——保留 decision lineage 作审计线索。删了就丢了判断是怎么演化来的。

防的是:多个 session 写下冲突的 HOLD 决策,污染后续判断。

---

## 5. Claim 不得在 agent 之间"洗白"

跨 agent 聚合时,**下游 claim 的强度上限 = min(上游各输入的强度)**。

这不是新公式,是把单 agent 的 `claim_maturity = minimum(...)` 扩到 agent-to-agent 域。防的是 **claim laundering**:没有人撒谎、没有 gate 拒绝,但聚合结论仍然虚高——每个环节都"合理地"提高了一点置信度,汇到最后就超出了证据许可。

配套三条承重结论(来自 Trusted Multi-Agent Research Runtime v0.1 设计):

1. **Gate 必须在 prompt 前,不能靠输出后评审补。** 理由是信息可得性:缺陷如果没 surface 进 prompt,生成的文本里就已经没有痕迹了,评审者无法察觉一个"不存在"。
2. **授权信封必须在外部状态,不能在 agent memory 里。** agent 会用更自信的措辞复述自己的历史,复述即成记录(可篡改);context 压缩又会静默丢溯源。信封在产出时固化(`envelope_at_emission`),之后 `asserted_strength > envelope` 才能成为可机械检测的违规。
3. **辩论不是证据。** 多空辩论在产品形态上充当"可信度放大器"(结论经辩论存活 → 看起来更可靠),但辩论只在既有结论之间重分配置信度,对世界没有新增观测。照搬 = 在 workflow 内部装一个 claim 强度放大器。

**诚实边界**:以上防的是"结论超出证据许可",**不保证结论正确**。authorization ≠ correctness。agent 仍可能编造 bundle 内不存在的事实、claim 强度正确但推理错误、相关证据被当独立确认重复计数——这些不在覆盖范围内。

---

## 6. 产物三分类(避免类别混淆)

设计资产能和 ACTIVE 冻结窗口并存,根因是它们属于不同类别。三者不可混:

| 类别 | 功能 | 权威性 | 特征性失效 |
|---|---|---|---|
| Design Artifact | 知识探索 | 无 | 被当 Governance(未实现的控制被当生效约束);被当 Evidence(架构图被引为能力证明) |
| Governance Artifact | 规范控制 | 有约束力 | 被当 Evidence(规范被引为它自身已实现的证明) |
| Evidence Artifact | 观测事实 | factual | — |

一句话:**冻结的产物 ≠ 执行的产物;写下的规范 ≠ 生效的规范。**
