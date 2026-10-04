# Trust Gate 与准入

> gate 必须在 prompt 前;declarative policy≠enforceable gate;admission 按 producer 枚举

共 24 条。源自 Engram 跨 agent 记忆库,已脱敏。


## 根因：response-dedup 缓存天然不带 gate_decision_id 绑定；Gate 概念引入后无人改造缓存，HIT 路径返回 ungated p…

`2026-09-29`


根因：response-dedup 缓存天然不带 gate_decision_id 绑定；Gate 概念引入后无人改造缓存，HIT 路径返回 ungated prior response。How to apply：任何 Gate 闭合前必须 (a) gate_decision_id 进 cache key；(b) evidence_bundle_id 进 cached payload；(c) HIT 缺绑定时 fail-closed；用 negative test 验证（改动一个原始字段 → HIT 必须重新 admit 或 fail-closed，绝不能返 ungated）。


## 根因：UI 渲染的 verified/success/partial 标签若在模型调用之后才被铸出，是 display contract 而非 admissio…

`2026-09-29`


根因：UI 渲染的 verified/success/partial 标签若在模型调用之后才被铸出，是 display contract 而非 admission decision；minting site 与 rendering site 分居模型两侧。How to apply：审 Gate 主张时先定位 admission 真正 mint 的位置（allowed_use / blocked_fields），再核对 UI label 是来自 mint site 还是 render site；只信 call site 不信 label。


## 根因：fact baseline 与 transport probe 默认与 consumer chain 隔离；gate 闭合 = gate 事实闭合，不等于…

`2026-09-29`


根因：fact baseline 与 transport probe 默认与 consumer chain 隔离；gate 闭合 = gate 事实闭合，不等于线路接好。How to apply：自检 readiness 必须同时存在 producer + consumer + delivery route 三段；任意一环缺位即不可声称 ready；区分 FACT_GATE_CLOSED（事实落档）与 RUNTIME_READY（活链路+pin），禁止跨级宣称。


## 看见渲染器实现不能升级成历史产物由它生成，更不能升级成可绕过 collector→gate→qc 直接喂数据。缺截图库不能推断历史图的 producer。渲染器入参含 gate 说明 gate 必须来自对 run_dir 的 Trust Gate，不能手造。复用前分别证明 binding / producer / gate provenance / schema identity；安全路径未证则不出图、不旁路、不发布。

`2026-09-09`


局部事实（仓库有 renderer、有历史图、venv 无截图库、函数可调用）被连升三级：同一视觉 → 历史 producer 已知 → 可跳流水线。反证形态是内部 preview 指纹与对外发布 HTML 不是同一合同。How to apply：四项分别证明，缺一则停在只读 attribution；手工 HTML 可本地查看但身份保持非 canonical。同类模式：artifact existence ≠ capability；schema ≠ contract。


## When an ImportError names a module the live handler never imports, the producer is a different tree. Copying a historical file into the running directory does not fix it and contaminates identity.

`2026-09-09`


Apply: (1) Hash the live process argv/cwd against every checkout that contains the failing import. (2) If the live tree comments out or never calls that import, do not treat module-absent as the root cause of the live error. (3) If the module exists only on a non-ancestor commit, source wiring can be PARTIAL while remaining non-executable; a fallback path succeeding on the live tree does not complete the other tree. (4) Mutation target stays unproven until an owner chooses drop-the-dead-branch vs formally admit the module through provenance and a trust/QC gate. Never copy from a historical carrier to make the import succeed.


## 同一上游身份被多个本地消费者同时订阅时,「本进程已连上」和「聊天里出现了回复」都不能证明本进程消费了某条入站事件。验端到端必须用当前进程启动时刻之后的入站日志,卡片头才是 producer。发现双订阅先冻结端到端、开隔离门,禁止用修另一路的配置错误来替代归属证明。

`2026-09-06`


根因:投递按身份不按进程;任一同身份存活消费者都可能接到事件。二进制含冲突错误码字符串不等于本次触发了它。How to apply:E2E 四点(收到→spawn→result→delivered)全部取自新 pid 启动横幅之后;UI 卡片不能当归属。双订阅结构存在即可冻结,具体哪条事件归谁未证时不要写成根因。


## 交叉验证必须在 admission 之前，不能放在 QC/allowed_use admission 之后

`2026-09-02`


烯牛 MCP capability audit 里 spec 目标链把 Cross-validation 写在 QC/Trust Gate 之后，与 Trust Gate Contract 的 source-validity-before-admission 冲突，被 reviewer 判 REQUEST_CHANGES。根因：把 provider 级 source-validity 交叉验证（须先隔离验证）和研究级多源交叉（synthesis 阶段）混成一个节点。How to apply：强制链路应为 raw capture → quarantine/provider validation(含独立交叉验证) → normalization → QC → allowed_use admission → admitted_context → synthesis；未通过交叉验证前不得进入 admitted_context。


## 信号名称/规则定义与实际admission的语义错位必须按contract violation处理，不得用盘中结果事后解释

`2026-09-02`


09-02 信号3 名称=大额异动+涨停确认，规则=AND gate(≥5000万 AND 进涨停池 AND 行业涨停≥2)，实际却把未涨停的金牛化工以板块联动写入候选。根因：rule→producer→narrative drift，行业联动被用来代替个股涨停确认。处理：admission contract未严格执行时verdict应为CONTRACT AMBIGUOUS/VALIDATION NOT ADMISSIBLE；禁事后改名信号语义再用结果证明有效（同慧点睛v1时点污染）；large_order_event≠persistent_demand，单笔大额买单推不出持续性大资金承接。How to apply：信号管道应为大额异动→Candidate→价格确认→板块确认→Admission→Observation，而不是大单出现→AI给信号→盘中结果解释。


## Strength Confirmation ≠ Entry Quality：一个候选在 strength 维度强(涨停/连板/新高/高位排名)不构成此刻入场风险可接受的证据；两者是两个独立问题，混为一谈会让强信号直连买入决策。

`2026-08-27`


根因：strength-oriented 候选池 + 数据健康 QC 构建候选，无 pre-decision risk/portfolio/execution admission；UI/prompt 按最强标的排序，把高位/追高后置成叙述。How to apply：任何强候选→推荐管线，把 risk admission 作为独立 veto 层插在 selection 之后、customer-facing 决策之前；veto 输出状态机(PASS/PARTIAL/BLOCK/UNKNOWN)而非分数；PASS 只移除该层 veto 绝不产出买入；UNKNOWN 默认不进决策上下文，禁止 LLM 升级为 BUY；风险审查必须先于 LLM 看到候选。


## Volume expansion itself has no direction: the same volume feature flips semantics with the price structure it accompanies. Volume must always be judged together with frozen price gates, never as a standalone bullish or bearish signal.

`2026-08-26`


Root cause: treating volume growth as intrinsically positive leads to misreading a volume-participating spike-and-fail (breakout failed, support broken) as healthy accumulation. How to apply: define price gates before the session and bind every volume reading to them - volume up plus price holding gains is positive evidence; volume up plus breakout failure and support loss supports distribution instead. If a gate fails once, downgrade the state explicitly and require the market to re-absorb the failure before any upgrade.


## Gate 0 冻结 tree identity 之后，必须显式关闭 Gate 0 并另开 Gate 1 才允许 mutation。IDENTITY FROZEN 是 mutation baseline 的准入条件，不是 mutation 授权本身。

`2026-08-24`


根因：fingerprint 稳定会让人产生"现在可以开始清理"的惯性。hygiene 窗口的目标是分阶段授权：先证明树静止（Gate 0），再决定要不要改、改什么（Gate 1）。如果允许从 Gate 0 自动滑入 mutation，Gate 0 中发现的问题会顺手被修掉，破坏分类的只读原则，也让 mutation scope 模糊。本次实践：Gate 0 观测 5h47m42s / 60 样本 / 无 mutation，给出 IDENTITY FROZEN verdict，但把 MUTATION NOT AUTHORIZED 作为同等重要的冻结条件一并写下，并列出 Gate 1 必须明确的 7 项授权范围。How to apply：(1) Gate 0 verdict 文本必须同时包含 CLOSED/IDENTITY FROZEN 和 MUTATION NOT AUTHORIZED；(2) artifact 中显式写 Gate 1 入口条件，防止自动滑移；(3) 索引行同步更新，避免后续 session 看到旧的 NOT FROZEN 状态；(4) 只有拿到明确的 Gate 1 授权后才执行具体 disposition 动作。


## 规则: 枚举数据准入边界要按 producer(每个产生外部数据的调用点)枚举,不按被消费的字段枚举;只回传标题或链接的路径仍能承载受限域断言,不能给字段级默认豁免

`2026-08-20`


原因: 修复方与评审方都按该路径只回传标题+链接+状态判定其无需 gate,漏掉一条搜索进上下文的旁路,但标题本身就能承载受限域断言(如主力资金净流入居前/高管减持公告)。适用: 任何外部数据源进 prompt 前,对每个 producer 调用点做静态 census,确认全部走同一准入边界;不按回传了哪些字段判断是否豁免。


## 凭证到 consumer 的映射闭合必须分三层：file-path auth、runtime auth、IM event ownership；长驻 daemon 改配置后必须重启，且旧进程 teardown 日志不能算新进程证据

`2026-08-19`


P1 probe 使用 09:28:34 自然事件作为探针，证明 codex-remote 是唯一观测到的消费者，无需发送测试消息。关键纪律：(1) 区分 observed consumer 与 canonical owner，单次 runtime 事件不能自动导出治理裁决；(2) duplicate consumption observed: NO 不等于 duplicate impossible，尤其当其他 consumer 处于崩溃循环或未加载时；(3) runtime defect（lark-channel-bridge agent-version-check-timeout）必须从 ownership chain 剥离，它解释未消费但不能证明无消费能力；(4) root cause 要分级：failure point confirmed 不等于 root cause proven。


## 规则：底层合约闭合不蕴含上层能力授权；wrapper 层契约验证完成，不代表集成层或生产层已获授权，两条线必须独立授权、独立闭合。

`2026-08-17`


根因：wrapper 层（identity contract + normalizer postcondition）验证完成后，有一种惯性是把它当作数据已可进入系统的信号，从而顺势推进集成层接线。这是层级混淆：wrapper 层证明的是协议身份可信，集成层需要证明的是真实 provider session 到 Evidence Object 到 QC/Trust Gate 到 consumer 的全链路，授权边界完全不同。How to apply：每个层级的闭合声明只在该层生效；重新授权某一层不等于上层自动授权；新窗口从 real provider/session → wrapper → Evidence Object → QC/Trust Gate → consumer 重新打开，不在当前 closure 上堆代码。


## 授权阶梯(allowed_use)只在_qc里算出来不算修复，必须在下游消费点(enforce)分支拦截；且多条数据路径要各自单独审计，不能只修一条

`2026-08-14`


根因：集合竞价语义P0的allowed_use阶梯在_qc_auction里计算、在format_*里展示，但(1)/api/analyze仍无条件调LLM，(2)golden-pit走独立词表且从不调_qc_auction，等于两条路径都绕过授权。How to apply：任何admission/authorization/QC信号落地后，grep该数据的所有消费点并加显式enforce分支；把信号已计算与信号已强制执行当作两个独立、分别测试的状态(同feedback_unwired_helper_is_not_a_fix、feedback_wiring_not_enforcement)。本次修复：L1/L2不调LLM(auction_allows_llm判定)+golden-pit统一到_qc_auction阶梯(L1只观察不构造候选,能力上限=market_structure_overview)。


## 抢修窗口收口必须锁定状态机边界：observed 200 OK 不得改写已裁决的 Incident NOT PROVEN，Capability 解冻需要独立 Runtime Validation Gate，最小降级（前端禁用提交）优于后端修补

`2026-08-12`


Leadership Minimum Service Recovery CLOSED。Stock: Incident Correlation CLOSED/PASS 但 Capability FROZEN，解冻唯一入口 Runtime Validation v0.1（三段 Gate）。Auction: Trust Enforcement FAIL，最小降级 = auction.html 前端禁用提交，backend route 未动，其他 skill 不受影响。关键原则：(1) 事故裁决和 Capability 断言是两个独立状态机，禁止用后续 200 洗白；(2) 前端最小降级优于后端修 Trust Gate；(3) 收口后禁止扩边。


## 规则:Consumer自动化≠全链自动化。Brief Engine consumer调度已生产化，但close_handoff producer是人工Codex步骤，从未自动化。系统表现被误解为"自动Producer→自动Handoff→自动Consumer"，实际是"人工Codex→handoff→自动Consumer"。临时/人工执行≠生产安装。

`2026-08-12`


根因:consumer调度(cron+run_once.sh)的存在制造了全链自动化的假象，但upstream handoff producer无任何launchd/cron/pm2调度。How to apply:(1)任何pipeline的自动化声明必须逐层验证——consumer自动化不能推断producer自动化;(2)手工步骤必须在架构文档中显式标注MANUAL/NOT PRODUCTIONIZED，不能靠"昨天跑通了"推断今天会自动跑;(3)Producer生产化时必须解决provenance/freshness/allowed_use/失败降级，不能简单用cron包装Agent调用。


## 规则:生产事故的修复完成≠事故窗口关闭。L1修复(表层Bug)和L2 Gate验证(adversarial)是两条独立证据链，必须由不同Agent执行、不互相背书。关窗需要三条路径裁决：Trust Gate阻断→PASS可关窗; Gate降级→PARTIAL需验证allowed_use严格执行; Gate仍放行低质量evidence→BLOCKED升级为Trust Gate runtime defect。

`2026-08-12`


根因:过去"代码修了→smoke过了→宣布完成"的闭合逻辑缺少Gate层独立验证。L1修复只证明展示层恢复正常，不证明Trust Gate在LLM调用前正确执行了gate决策。How to apply:(1)任何Runtime Incident分L1/L2两条独立修复链，不同Agent执行;(2)L1修复后smoke只验证L1，不为L2背书;(3)L2 adversarial verification必须验证完整链:Evidence→Trust Gate decision→allowed_use→Prompt→LLM→claim，证明Gate在Prompt之前执行而非事后贴标签;(4)关窗裁决需要C输出结构化verdict:Input evidence/ Gate decision(BLOCK|PARTIAL|PASS)/ Allowed use/ Blocked use/ LLM received prohibited evidence(YES|NO)/ Final output exceeded allowed claim(YES|NO)/ Verdict(PASS|BLOCKED|NOT PROVEN);(5)只有BLOCK或严格PARTIAL才允许关窗，PASS=BLOCKED升级。


## 规则: _qc.status=success不能替代Trust Gate gate决策。completeness<100%+sources=[]+source_date_unknown=true时_qc.status仍可为success，但这份evidence不应未经Trust Gate(status/blocked_fields/allowed_use)进入LLM。Trust Gate拦截≠接口故障，不拦截但低质量evidence通过=governance failure。

`2026-08-12`


根因:QC层(status/completeness/sources)与Trust Gate层(blocked_fields/allowed_use/gate_decision)是两层不同职责——QC回答"数据是否成功获取"，Trust Gate回答"evidence是否足够支持业务声明"。当completeness=40%+sources=[]时QC.status=success是正确的(数据获取成功)，但如果Trust Gate没有在此基础上做gate决策就直接放行到LLM，则Trust Gate未履行其职责。How to apply:(1)任何调用LLM的pipeline必须在LLM调用前插入Trust Gate gate决策节点，输入为_qc输出，输出为allowed_use/blocks;(2)adversarial test必须在边界条件(低完整度/无来源)下验证gate行为，不能只在正常条件下验证;(3)Trust Gate拦截时前端展示"当前数据不足以支持完整分析"而非裸JSON。


## 多 producer 共享 consumer pipeline 时，output contract 必须在 schema 层显式定义 + consumer 提供 fallback 路径。Schema 统一不等于 runtime 统一——producer 是 AI agent 时，prompt 须包含 contract 生成指令。

`2026-08-10`


Morning/close 两个 CC loop 各自生成 handoff JSON，被 brief-engine 同一 pipeline 消费。Morning 缺 qc.sources 导致 evidence=0 条。修复：schema v1→v1.1 加可选 qc 块，consumer(evidence.py)加 fallback 路径。根因：contract 只在 close producer prompt 中存在，morning producer prompt 未包含，且 consumer 未防御缺失。How to apply：任何多 producer→单 consumer 的 handoff/artifact pipeline，先在 schema 层定义 unified contract，consumer 对 contract violation 提供降级而非静默空值。


## launchd/service 条目数不等于 runtime consumer 数——判断并发消费者必须先查该 bridge 自己的注册态(如 ps/registry),再看进程存活

`2026-08-09`


根因: launchctl list 有条目只证明 launcher 被登记,不证明进程注册成 consumer。实测 4 个 feishu launchd label 中,1 个从未注册(被 profile 锁挡住即退出),1 个存活但消费未证,只有 2 个确认注册。若按 service count 推断 consumer count,会把"两个 launcher 抢同一把 profile 锁"误读成"两个 Agent 抢消息流",进而删错服务。How to apply: 任何 N 个服务在跑 的断言,先跑 bridge 自带的注册查询拿 registered 列表,再对齐进程;四层要拆开——label 存在 / 进程存活 / 已注册 / 正在消费。附带两条: (1)设计文档里的"短期方案"若本身是 runtime change,必须显式标 NOT AUTHORIZED,否则实施窗口会把 Recommendation 读成 Implementation Authority; (2)Identity Isolation 与 Credential Hygiene 正交,发现明文 secret 不要在验证窗口内顺手轮换,会破坏窗口边界。


## 规则: 共享游标的轮询型消费者,启动动作不幂等 — 多起一个实例会瓜分消息流,且症状与白名单过滤不可区分

`2026-08-07`


原因: 单游标 + 多消费者时,任一条消息只会被其中一个实例取走,另一个永远等不到。对外表象是服务健康、日志无报错、请求不回应 — 与鉴权白名单拒绝的症状完全一致,会把排查引向错误根因。适用: 轮询型消费者 bring-up 前,先按可执行名枚举现役实例再决定是否启动;按命令行文本 grep 会匹配到自己的 shell 与 grep 进程,得到虚高计数,必须换按可执行名统计。发现已有实例时先追父进程链确认归属,不直接终止。


## Trust Gate Phase 1a Enforcement Pilot: wind_query单工具G1+G3边界生效。关键设计决策:(1)Trust Gate作为MCP工具返回的后处理层而非Provider内嵌——不修改Provider/QC;(2)BLOCK时STRIP数据内容仅保留_trust_gate+_qc元数据——被拒绝的证据不进入LLM上下文;(3)每条边界返回独立BoundaryResult含reason_code+evidence——决策记录可独立审计;(4)G3 stock max_age设为48h因为wind_query是EOD快照非实时数据。12个对抗测试中已有T-01/T-03/T-04/T-11可在此Phase验证。How to apply:单工具验证通过后再推广，不要在Proof完成前扩展到22 tools

`2026-08-06`


Phase 1a Enforcement Pilot:新建trust_gate/包(decision.py+BoundaryResult,G1 completeness,G3 freshness,enforcer),修改mcp_server.py wind_query三处return点嵌入Trust Gate,9个验证用例(VC-01~03+LV-01+3 cross-cutting+2 coverage)全部PASS。wind_query返回格式:{_qc,_trust_gate}+content。BLOCK时data stripped。禁止修改Provider/QC/扩展22tools/宣称全链路完成


## Builder修复通过≠Trust Verdict通过。Multi-Agent Trust Gate v1首个完整闭环：CC diagnosis→Codex独立发现→CC Path B fix→Codex re-review→counter-example(F-03: len(stale)<=1无法阻止单关键维度staleness通过success)→verdict从PROVEN降级为NOT PROVEN。Reviewer发现了Builder漏掉的counter-example，验证了身份分离架构的核心价值。

`2026-08-03`


600439 financial freshness bug修复后，CC使用len(stale)<=1作为freshness_ok判据，Codex发现当仅financials单维度stale且completeness>=0.8时，Trust Gate仍返回success——这正是原finding描述的failure mode。关键教训：freshness不能按stale计数判断，必须按关键维度权重判断。Fix effective for original symptom(acquisition truncation)，但Trust Gate closure criteria未满足。决定关闭当前iteration，Trust Gate Freshness Policy v2单独立项。
