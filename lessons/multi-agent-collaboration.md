# 多 Agent 协作

> builder/reviewer;downstream claim≤min(upstream);授权信封在外部状态;辩论≠证据;review 不继承更强状态

共 14 条。源自 Engram 跨 agent 记忆库,已脱敏。


## 规则: 写完禁止性契约条款后，把它全部泄漏形态（标识符类/绝对路径类/IP主机名类）一次转成扫描模式扫齐；只扫 reviewer 点名的那一种形态，等于下一轮被点另一种形态

`domain: finance-suite` · `2026-08-27`


禁止性条款的泄漏形态通常是多类的：禁硬编码目标 = 标识符(chat_id/open_id) + 绝对路径 + IP/主机名 三类，不是一类。只把 reviewer 点名的那一种形态转成 grep 就宣布 CLEAN，会在同一批文档里留下另一形态的残留。根因是把条款语义误解为单一形态。how to apply：条款写完后枚举全部泄漏形态，每种一个 pattern，一次 grep 齐；复扫结果写进回执。


## 写下禁止性契约条款后必须立刻用该条款做grep pattern全仓扫旧文档,只改reviewer点名那处会留同类残留旁路治理

`domain: claude-code-infra` · `2026-08-27`


根因:在publication_safety_contract §8写禁硬编码chat_id/open_id用逻辑目标名+受控配置键。C仅点名worklist:23违反。自行扫全docs发现同类违规共4处(另3处C未点名):product_requirements_plan:15/requirements_handoff_filled:116/requirements_auction_admission_pipeline:64-65,后者还把完整id内嵌在可复制发送命令里危害最大。危险点:实施者只读其中某一份,只要还有一份旧文档带硬编码目标就能被照抄绕过配置与凭据边界,这是治理条款可被旁路而非文档美观问题。how to apply:①写完条款立刻转grep pattern扫全仓②改完复扫确认CLEAN并把复扫结果写进回执作为可证据③多份文档谈同一主题必须指定唯一SSOT其余抬头标SUPERSEDED+主题到权威文档对照表+实施者不得直接照本文件实现④reviewer点名一处先假设是一类而非一个实例,扫完再答复⑤同批可扫pattern:明文凭据/硬编码IP/绝对路径/裸token/未证实endpoint


## 交接文档引用代码模块只写仓库内相对路径不够,必须带canonical仓库身份,否则同名副本会把实现者引向历史文件

`domain: finance-suite` · `2026-08-27`


根因:auction_data.py全盘搜出10份同名副本(backend/app 31508B=canonical README自述Canonical for Backend runtime; finance-suite/scripts 7311B=历史前端副本; 另8份散在各clean/review工作树)。字段名与head注释两边一致,实现者无法自行判断哪份canonical。C review判major。how to apply:①路径带仓库名前缀+显式写不要改X(历史副本)②判canonical用仓库自述+体量+mtime三者交叉,不只看路径像不像③同纪律适用macro_data.py/search.py/skills.py/market_intel.py等前后端同名文件


## Evidence bundle 里的测试输出摘要（省略号、注释替代字段值）会让 reviewer 无法独立验证，必须逐字附原始 stdout + EXIT 码

`domain: finance-suite` · `2026-08-16`


根因：bundle 编写者用"full JSON output in session"和"[...10 fields...]"代替真实输出，reviewer 发现后要求三次才关闭 M1。How to apply：evidence bundle 的测试收据段必须直接粘贴终端完整输出，包含每一个字段值、行数与 EXIT 码；任何省略都使 bundle 的 no-omission 声明自我矛盾，导致重复 REQUEST_CHANGES。


## 工具受限的 reviewer（无 Bash/无 git）报「commit 不存在/生产不可得」是工具能力假象不是事实；交付复核前必须把 git 证据物化为普通文本并 push origin

`domain: claude-code-infra` · `2026-08-15`


fefc2d8 首轮 BLOCKED 三根因：(1) reviewer 无 Bash 读不了 git zlib 对象，SHA 只在 .git 内部；(2) commit 未 push；(3) 生产文件在远端无本地副本。补法：diff/tree/blob/meta/refs 导出为普通文本 + scp 生产快照 + 服务器 sha256sum 原始输出 + REVIEW_TARGET.md 复核手册并诚实标注 reviewer 验证边界（无 Bash 时 hash 级独立计算不可行，只能内容级比对）。how to apply：交付受限 reviewer 前自问「它能用现有工具读到我的证据吗」，不能则先物化。


## 评审前提中的代码依赖声明必须先对照源码核实再采纳——被证伪的前提会让整个 acceptance 判断失真

`domain: finance-suite` · `2026-08-15`


Auction P0 澄清窗口:reviewer 声称现有审计记录 auction_factor_scan 依赖 zt_pool+hot_rank+top_gainers。逐字核查证伪:factor_scan(MCP mcp_server.py:919)走 trading_system/joinquant 降级,golden_pit()(server_scripts/intel_api.py:271)内部调 factor_scan(),均不消费任何 auction 维度;{zt_pool,hot_rank,top_gainers} 真实出处是 AUCTION_REMEDIATION_PLAN §Tier 2 的 market_pulse 路径约定(非 factor_scan 依赖)。根因=reviewer 把 audit 记录脑补成代码依赖。how to apply: 评审前提中任何 X 依赖 Y 声明,先 grep 源码取 file:line 佐证再进 contract/decision;证伪则修正前提而非沿用,并在回复中附行级出处供复核。


## 给无 Bash 的 read-only reviewer 发 git commit 复核前，必须把 commit 证据物化为普通文件（diff/tree/meta 导出 + 生产快照 scp + 服务器 hash 原始输出），并按 reviewer workspace 协议发 handoff；否则 reviewer Grep 工作树永远命中不了 .git 内部的 SHA，必然 BLOCKED

`domain: finance-suite` · `2026-08-15`


根因：reviewer 工具集只有 Read/Grep/Glob，(1) git 对象是 zlib 压缩不可直接 Read，SHA 字符串只存在于 .git 内部；(2) commit 未 push 到 origin；(3) 未按 windows/ handoff 协议给 reviewer 定位信息——三条叠加制造了 commit 不存在的假象。How to apply：复核任务发出前先检查 reviewer 工具集；无 Bash 则物化证据 bundle 并给出可 Read 的绝对路径清单


## 澄清窗口的 acceptance contract 必须显式分离 verdict，否则 reviewer 会以 implementation 未做把 CONTRACT-CLARIFICATION-ONLY 窗口反复退回

`domain: finance-suite` · `2026-08-13`


Auction Semantic P0: task mode=CONTRACT CLARIFICATION ONLY/Implementation PAUSE，reviewer 仍以 mcp_server.py admission 未实现、_present() 把 timeout/error 判 present、hot_stocks 绕过 admission 等 5 项 implementation findings 判 REQUEST_CHANGES。根因=reviewer 把 implementation 验收 bar 套到 clarification 窗口。解法=拆两个 verdict(Clarification COMPLETE / Implementation NOT AUTHORIZED)，5 findings 全部 OPEN 转入独立 Repair Window，并声明发现现场 defect≠获得修复授权。how to apply: 开澄清/设计窗口时先 pin acceptance contract，禁止用 implementation findings 反向改判 clarification verdict。


## 规则: 在 Agent 链上,下游可声明强度必须 ≤ 所有上游授权的最小值;推理、综合、评审、辩论只能保持或降低强度,只有新的经 Gate 证据或具名人工授权才能提升。

`domain: finance-suite` · `2026-08-07`


原因: 多 Agent 串联的特有失效不是幻觉,而是 claim laundering——证据以低授权进入,每个 Agent 各自合理地对输入推理,输出却逐跳变强。没有任何一个 Agent 撒谎,也没有任何一道 Gate 拒绝过,聚合结论仍然虚高。根因是把上游 Agent 的输出当成了证据:输出只是关于证据的一个声明,声明不因被复述而获得授权。辩论/投票/多数意见尤其危险,它们只在既有结论之间重分配置信度,对世界没有新增观测,却在产品形态上被当作可信度放大器。

适用: (1) Agent 输出必须与证据分成两个类型,前者携带指向后者的引用且永不替代后者,禁止 Agent 引用另一个 Agent 的结论作为来源;(2) 授权信封写在外部可查询状态里而非 Agent memory——memory 会被 context 压缩静默丢掉溯源,且 Agent 可以用更自信的措辞复述自己的历史,复述即成为记录;把发出时的信封固化在外部状态,『声明强度 > 当时信封』才成为可机械检测的违规;(3) 强度上界只能由外部输入抬升,任何 Agent 拓扑都不得自行抬升自己的上界;(4) 注意副作用:多咨询一个弱维度会拉低整体上界,这会反向激励少查来源,需要单独设计覆盖度与强度的解耦,且该解耦本身是新的 laundering 风险点。


## Governance Design 产出必须过独立对抗审查，不能自宣 NORMATIVE。本次 Provider Capability Governance Alignment v0.1 完整闭环：CC(Builder)四份治理草案→C(Independent Reviewer)6项 NOT PROVEN(F1 L3超证据/F2分级矛盾/F3 gate覆盖事实错误/F4 Phase 1a不当升级/F5聚合不自洽/F6可行性无证据)→CC逐项修正(L3全降L2/纠正gate状态/对齐C v0.2)→C验证6项全部通过。教训：Builder不得自评文档状态为NORMATIVE；单次probe不能支撑L3；fixture PASS≠gate logic correct；聚合器等级不能超过最弱已验证组件。How to apply: 任何治理文档窗口结束后必须过独立对抗审查才能升为NORMATIVE。优先证据收集而非规则扩写——先补齐multi-session evidence再写规则。

`domain: claude-code-infra` · `2026-08-07`


Provider Capability Governance Alignment v0.1 完整审查闭环。CC四份草案→C发现6项F级问题→CC修正→C验证全通过。四份文档从NORMATIVE降为ADVISORY DRAFT。原则确立：Builder不自评、单probe≠L3、fixture PASS≠gate correct、聚合器≤最弱组件。下一步从Governance Design切换到Evidence Collection Window。


## 多源文档构建Agent知识库时,必须先确立权威口径优先级(官方正式版>内部工作版),否则旧术语污染所有下游AI输出

`domain: claude-code-infra` · `2026-08-06`


2026-08-04 闽商会客室 Agent KB:同时参考旧版树状图+官方20260629手册+战略报告,写完发现多处术语冲突(闽人堂vs福蕴闽商,三级茶叙vs两级)。根因:动笔前未做source reconciliation,把丰富度放在准确性前面。修正:在KB末尾显式声明口径优先级,逐条对照SSOT做hard edit。此后任何多源Agent KB任务,第一步列出所有来源+日期+选SSOT,第二步对照SSOT,第三步才叠加其他来源的独有增量。


## Multi-Agent Trust Gate v1 的 HOLD verdict 是治理设计在正常工作,不是验证失败 — Independent Reviewer 发现 Builder 窗口设计层的 provenance/absence/cross-session 三类结构性缺口,正是身份分离要达成的效果

`domain: claude-code-infra` · `2026-08-06`


根因:Multi-Agent Trust Gate v1 的核心设计是 Builder(CC)和 Reviewer(C)身份分离,C 的 HOLD verdict(3/12 NO:evidence provenance NOT IMMUTABLE / cross-session reference unverified / absence claim unverifiable)不是 Builder 的错误,而是窗口设计(Documentation-Only vs Provenance-Track)的内在限制被 Reviewer 层面首次暴露。这正是 Trust Gate 应该做的事 — 不是事后发现 bug,而是事前阻止不可验证的状态传递。How to apply:(1)任何 Documentation-Only 窗口的 Declaration 必须在启动时就列出 provenance/absence/cross-session 三类 claim 的可验证性边界,不能等 Reviewer 阶段才发现;(2)Reviewer HOLD 不应被 Builder 当作修复任务,应该被当作窗口设计缺陷反馈给 Governance baseline 修订;(3)HOLD 后不应立即开新窗口(WINDOW_DESIGN_CLOSURE),避免治理无限递归,而是将三条 rule 沉淀进下一版 Governance baseline,等下次 Review Window 自然触发时一并验证


## Agent workflow success reports do not imply artifact completeness; always verify the output file manifest against task-card acceptance criteria before closing a research window.

`domain: finance-suite` · `2026-08-05`


Root cause: workflow orchestrators report completion when the last agent finishes, not when all required files exist or cross-reference correctly. How to apply: before accepting any multi-agent research output, run a read-only closure check that lists expected files, verifies cross-references, and confirms required deliverables like SOURCE_MATRIX exist.


## 独立 reviewer 返回 BLOCKED 时，owner 不应辩解或只做措辞修补，而必须把过度宽泛的 claim 降级，并把缺失的退出条件补成显式 gate，否则 RRA 会从评估文件变成未经验证的放行声明。

`domain: finance-suite` · `2026-08-04`


根因：owner 写的 assessment 天然倾向于把已知问题描述成 "有条件通过"，而 reviewer 的角色就是发现那些条件其实没有被证据支撑。如果 owner 只改语气不改 verdict/exit gates，adversarial review 就失效了。How to apply：(1) reviewer 返回 BLOCKED 后，owner 必须逐条把 BLOCKING finding 转成 assessment 中的 explicit release blocker；(2) 凡是 "本地 ref / 未 fetch / 未验证" 的证据，verdict 不能是 PASS；(3) 每个 CONDITIONAL 必须带一个可验证的关闭条件，而不是只写 "需要后续窗口"。
