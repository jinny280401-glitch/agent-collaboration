# 能力成熟度链

> Exists/Invoked/Succeeded/Authorized/Production Proven;测试全绿≠能力;commit≠production

共 30 条。源自 Engram 跨 agent 记忆库,已脱敏。


## 手工构造的派生产物，即使外表成功，也不能作为底层 schema 或渲染器能力已就绪的证据——成功来源是人工构造而非引擎能力，二者必须分开声明。

`domain: finance-suite` · `2026-09-17`


根因：派生产物的成功外表会诱导后续 agent 把「当次交付成功」误读为「永久能力就绪」，从而污染能力基线。How to apply：(1) 把「交付成功」与「能力就绪」作为两个独立结论分开记录，禁止由前者推导后者；(2) 临时/一次性派生表达必须标注「非永久定义」，不得回流为永久 schema；(3) 手动复用布局的渲染产物只证明当次交付，底层渲染器是否恢复是独立 gate。


## 规则: 判断一次调用是否真正用了 LLM、报告里的数字是否来自实时数据,只能看数据在生产层准入时的判定与逐字段来源;响应阶段的质检标签由模板或硬编码算出,与生产层是否实际调 LLM 无关,不能用来反推。

`domain: finance-suite` · `2026-09-07`


静态源码审计 + 真实运行时生产层探针共同闭合的治理规律。

根因(4点):
1. 生产层有 fail-closed 准入(运行时探针已实证, 非纯静态结论): 关键实时维度缺失 → bundle 不满足 → 授权降级 → 不调 LLM, 并非在关键维度缺失时无条件生成结论。
2. 响应侧质检对竞价场景存在指标失配: status/completeness/llm 推断占比等字段由股票/宏观风格指标或硬编码生成, 不能解释为生产层实际准入决策或实际 LLM 使用。
3. 生产层质检携带真正与竞价语义相关的字段(授权阶梯/维度结果/不可用维度/阻断字段/市场阶段) 却在响应边界被另一套质检覆盖, 最终回执无法重建当时真实准入决策。
4. provider 溯源有时间戳盲区: 多数维度只有本地观测边界、缺 provider 侧时间戳, 抓取成功不等于时点溯源已证明。

How to apply: 任何能力/证据声明必须分层裁决——provider 结果 → 溯源/时间戳 → 生产层质检 → 授权阶梯 → prompt 前准入 → 实际 LLM 调用 → 响应侧质检 → 发布管控。禁止: 用 HTTP 200 推能力已证明; 用真实运行时模式推实时数据为真; 用响应质检状态推生产层数据可用; 用响应推断占比反推实际是否调 LLM; 用响应质检替代生产层质检; 用不同时点探针倒推历史报告溯源。

特别纪律: 证明某份具体报告里的涨停数/连板数/封单额来自真实生产层证据, 必须在同一次调用内建立 生产层证据 → 准入 → prompt/LLM 输出 → 最终响应 的关联; 不同时点探针不得作前后伪对照。生产层缺失会触发降级+不调 LLM 是当前设计行为, 不等于所有历史运行都必然遵守相同路径。


## 规则: 验证外部数据 provider 时,调用成功只证明 transport+delivery,不证明返回内容的身份或准确性

`domain: finance-suite` · `2026-09-04`


根因: 把非空 provider 返回直接写成真实数据/已验证,是把 transport 成功误当成 content identity 已验证。How to apply: capability 按 配置/discovery/真实调用/运行时/生产 五层独立 gate 逐层取证,禁跨层传染;provider 返回字段一律标 provider-attributed,独立交叉核验后才可升级;修措辞时全仓 sweep 同类词形,同一 claim 常同时藏在 summary table/maturity table/index line 三处,只改被点名一处会漏其余。


## After a reviewer rejects an auction-probe claim, downgrade the plan to read-only capability probe; never package a prior-session snapshot as today's auction evidence.

`domain: finance-suite` · `2026-08-24`


2026-08-24 T+1 observation planned to "record auction evidence and form execution verdict". Reviewer returned REQUEST_CHANGES: Wind returns only 2026-08-21 15:30:54+08:00 close (price 7.08), not today's auction; no proven contemporaneous auction data chain; Channel A hot_up swallowed upstream error, Submit disabled. Initial response instinct was to argue the capability exists somewhere or to find a workaround source. Correct response: adopt the verdict, downgrade 09:15–09:25 from "auction evidence capture" to "capability probe", freeze the nine minimum evidence fields (raw_response / provider / function_or_endpoint / retrieved_at / provider_data_timestamp / market_phase / associated_consumer / qc_status / capability_verdict), and shift the single question to "does Finance Suite's current read-only chain have 'today's auction contemporaneous evidence' capability?" — restricted verdicts PROVEN / PARTIAL / NOT_PROVEN / UNKNOWN. Root cause: Confusing "the data exists somewhere" with "this system can retrieve it contemporaneously". Capability claims require capability evidence, not workaround evidence. How to apply: When a reviewer blocks a capability claim, accept the downgrade before attempting fixes. Do not repackage yesterday's close as today's auction. Do not backfill UNKNOWN checkpoints. Protect the non-replayable real window (09:15–09:25) over engineering tasks.


## Upstream provider capability ≠ Finance Suite integration ≠ fallback/mock behavior。三层必须独立取证，不能因为 upstream 存在就推出 integration 已完成，也不能因为 integration 缺失就把 fallback placeholder 描述成 upstream mock。

`domain: finance-suite` · `2026-08-23`


Why: Wind 4-Mock Attribution Audit 发现：有 wrapper/SDK/CLI 路径（upstream capability PARTIAL），但缺 authenticated provider receipt / provider timestamp / 真实调用日志；Finance Suite 没找到 provider → runtime → normalization → Trust Gate → consumer 实际接线（integration MISSING @ reviewed codebase）；"4-Mock"四个对象连 canonical identity 都未恢复（UNKNOWN / NOT_PROVEN）。How to apply: 任何 provider capability claim 必须分三层独立验证：(1) upstream API/SDK 存在 ≠ (2) 数据质量/receipt/timestamp PROVEN ≠ (3) Finance Suite 已集成。不得跳过中间层直接从"有 API"推出"可以接入生产"。


## 生产文件身份匹配≠运行归属≠canonical ownership，三轴独立取证；production↔repo 比对必须 pin 不可变 ref

`domain: finance-suite` · `2026-08-20`


Phase 4 生产采样 mcp_server.py SHA 匹配 backend e266213，但 systemd 只启 app.main:app，mcp_server.py 运行消费未证；canonical owner 由 G 显式裁决为 FRONTEND。根因1：production presence 被升格为 ownership。根因2：C 把 moving 的 fix/search-trust-gate-p0 分支当 backend HEAD 比对误报 api.py drift，改 pin 不可变 main e266213 后 drift 消失。how to apply：(1) production↔repo 身份比对先固定 immutable anchor；(2) identity/consumption/ownership 三轴各自独立取证不互相推导；(3) 显式 ownership 决策须 repo-materialize 成不可变 artifact，仅聊天/memory 不足构成 PROVEN


## 规则: 测试全绿不等于安全边界完备;bounded fixture 通过只证明列出的用例被覆盖,不能升级为边界完备性证明

`domain: finance-suite` · `2026-08-20`


原因: 正则或关键词分类器对枚举 fixture 命中,却无法穷举零词汇改写,仍会漏放;全绿只证明列出的 case 都过,不证明所有 case 都不会漏。适用: 把结构性覆盖(每条路径都 gate)与语义完备性(分类器穷举)拆成两个独立判据分别评估;不因测试全绿把有限覆盖升级成完备性证明。


## 验证外部 research-context provider 的 PDF 全文能力时，必须先用真实 API 调用确认能拿到 url_info，不能仅凭文档中的 导出原文 描述就假设可机读

`domain: finance-suite` · `2026-08-20`


教训：IMA 官方文档写明 get_media_info 支持 PDF 导出原文，但实际调用对三份不同 PDF（含本人上传控制组）均返回未文档化 220030 且无 url_info，无法进入 GET 阶段。根因：provider 文档/契约与 runtime 能力存在 gap，未文档化错误码使根因不可区分。how to apply：1) 新 provider 先跑最小 bounded probe（1× list + 1× media_info + 0× GET），证明能拿到原文 URL 再写 collector；2) 遇到未文档化错误码时，用控制变量（ownership、batch、media type）尽快缩小假设空间；3) 证据文件脱敏、算 hash、入 SHA256SUMS，NOT_OBSERVED 与 observed negative 严格区分。


## When a spec/contract document references implementation commits, always distinguish the implementation-identity reference from the contract-editing reference. Mechanically replacing all old hashes with HEAD produces wrong provenance and causes repeated reviewer FAIL.

`domain: finance-suite` · `2026-08-17`


Root cause: contract had three normative references (§8 mapping-table header, §10 test reference, §10 merge-authorization clause) still pointing to an old superseded commit instead of the frozen implementation baseline. The contract-canonicalization commit that fixed taxonomy was NOT the right anchor for implementation location — it was the right anchor only for contract-revision history. Fix: rebind all normative implementation references to the implementation baseline; keep any historical mention with an explicit (historical; superseded by <correct-hash>) marker. How to apply: before committing any contract/spec update, run grep for every commit hash and classify each occurrence as (a) normative implementation identity → must point to the frozen implementation baseline, (b) contract revision history → may point to an intermediate commit but must be labeled as such, or (c) pure historical context → keep but annotate superseded. Never treat the most-recent-edit commit as the implementation identity.


## 恢复 worktree 基线前必须先导出并验证可回放资产：patch+测试文件存到仓库外，git apply --check 通过后才允许回退

`domain: finance-suite` · `2026-08-15`


根因：closure 冲突时对方要的是干净 worktree，不是代码消失；未验证可回放就回退等于不可逆删除。How to apply：任何“清理改动/恢复基线”操作前，先证明恢复路径存在且可用（patch 非空可读、apply --check 通过），再执行破坏性动作；原分支 commit 保留作双保险。


## 字段名叫「成交量」不等于「虚拟匹配量」:接线前必须用真实数据验证字段语义,不能从字段名推断。实测 AkShare 盘前分时在 09:15-09:25 竞价窗口成交量全程为0(是实际成交,首笔在09:26),虚拟匹配量/未匹配量根本不在东财分时端点里

`domain: finance-suite` · `2026-08-14`


Auction Pre-Min Semantic Validation v0.1:stock_zh_a_hist_pre_min_em 打东财 trends2 端点(f51时间/f52-55开收高低/f56成交量/f58最新价)。实测竞价窗口 f56成交量=0、f58最新价=昨收恒定,竞价价格轨迹其实在f52-f55区间。裁决:Auction Price=PARTIAL / Virtual Matched Volume=NOT_PROVEN / Provider Timestamp=PARTIAL(分钟级) / Order Book=NOT_PROVEN(实时盘口无历史) / Unmatched Volume=NOT_PROVEN。修正了 Discovery C 方案里「成交量→虚拟匹配量」的错误映射。


## 规则: 时效阈值必须按用途与数据时态双维度定义;单一数值会同时误杀有效数据与放过失效数据,并把时长上限误当成业务规则

`domain: finance-suite` · `2026-08-13`


有效性是条件化判断而非对象永久属性;设计有效期参数先问对谁、在什么条件下失效;时长阈值不等于上一自然日数据天然有效,测试须覆盖跨交易日边界;二维参数化是把生产教训编码进契约而非复杂化


## deployment target 与 source repository 必须分离：生产目录永远不 git init，repo 创建 ≠ canonical SSOT 宣布

`domain: finance-suite` · `2026-08-13`


根因:把运行目录直接变成 git repo 会把 .env/DB/logs/backup/runtime artifact 卷进历史,并让一个对象同时扮演部署落地点和权威来源,审计和回滚都会错位。How to apply: 为运行中系统建版本历史时,先只读 secret/runtime 审计,按排除清单收窄快照到本地 staging,再 init/commit/push;创建 repo 与宣布 canonical source 是两个独立动作,后者要有单独裁决


## Capability closure verdicts need a frozen production-observation chain — automation exit 0, artifact existence, and non-zero result counts can never substitute for it

`domain: finance-suite` · `2026-08-13`


How to apply: when closing a repair card, freeze a numbered production-observation acceptance chain plus evidence-retention requirements (raw execution record, resolver output, final artifact), and keep the verdict at PARTIALLY PROVEN until every chain item holds with real production evidence. Any missing item opens a narrow follow-up card; the closed repair card is NOT reopened. Three non-substitutes that must not individually justify PRODUCTION OBSERVED: exit code 0, presence of a single artifact, non-zero comparison/score counts


## 跨调用比较 provider failure 分类前，必须先证明它们属于同一 invocation。golden-pit 观察到 parse_error、market-context 观察到 timeout，没有 caller-owned trace 关联时只能记为未关联的分类不稳定性，不能推断为同一上游故障的竞态。根因：不同消费路径有不同 timeout boundary（2.5s wait_for vs provider 自身延迟），分类结果不含 invocation identity，观察差异≠根因关联。How to apply：多路径观察到的不同分类只报告各自事实；做根因推断必须先有 caller-owned trace/receipt 把事件关联到同一 invocation；同理 HTTP 200/页面恢复只证明 response-shape，不证明 capability。

`domain: finance-suite` · `2026-08-13`


Auction Channel A Upstream Failure Semantics v0.1 收口：IMPLEMENTATION PASS / DEPLOYMENT SMOKE OBSERVED / FAILURE SEMANTICS OBSERVABLE / CHANNEL A NOT PROVEN / FROZEN。三个裁决边界锁定：(1)无 trace/receipt 关联不做跨调用根因推断；(2)VALID_EMPTY 不自动等于无候选，需算法完成+QC/admission 佐证；(3)HTTP 200≠capability proven。下一证据窗口 2026-08-13 09:15-09:25。


## 并发部署 near-miss：两个 agent 同时执行同一张 Observability 卡、用了两种不同设计（journald-9值taxonomy vs file-sink-3值taxonomy）。C 先部署并重启了生产。我靠 backup 时 md5 与 pull 副本不一致才发现冲突，没有覆盖 C 的版本。How to apply：生产文件替换前必须验证 当前md5 == 本地基准md5，不一致立即停止并排查并发写；生产改动只允许一个 owner。

`domain: finance-suite` · `2026-08-13`


Obs Production Activation: C deployed fetch_receipts.jsonl sink (app/stock_data.py 19796B) at 00:11 + restarted 00:12:42; first receipts show price_history/capital_flow PROVIDER_ERROR ConnectionError RemoteDisconnected (symbol 002594). My journald plan superseded; local MCP observability kept for local probes.


## 生产观测必须保持业务返回契约不变、用 caller-owned token 精确关联、sink 写失败显式暴露(fail-closed)，三者缺一不可；否则 None 永远无法被裁决

`domain: finance-suite` · `2026-08-13`


Observability Enablement v0.1：不改 list[dict]|None 返回契约，仅加可选 invocation_id + JSON-line receipt sink(/var/log/finance-suite/fetch_receipts.jsonl)。7 场景矩阵测试全绿。生产 re-probe 按 trace token 精确取回 receipt，Capital Flow/Technical 均分类为 PROVIDER_ERROR(ConnectionError/RemoteDisconnected)。教训：观测本身会失败，观测失败必须比业务失败更吵。


## except Exception: pass 在 provider fetcher 层会把 JSONDecodeError/timeout 坍缩为 None，下游 QC 无法区分 VALID_EMPTY 与上游故障，导致 completeness=1.0 的假成功。根因：异常被吞在离 provider 最近的一层，而上层 orchestrator 的分类基础设施永远收不到异常。How to apply：fetcher 层不要 try/except 吞异常，让异常传播到既有分类链；QC 必须消费 per-pool dimension status，error pool 不得计入 healthy，上游失败不得包装为 success。

`domain: finance-suite` · `2026-08-13`


Auction Channel A Upstream Failure Semantics v0.1 已部署生产（<server-ip>）。偏离 spec 两处：(1) _safe_auction_dimension 的 reason 保持 provider_error 不变、新增 error_type 细分，因 market_context_layer 按 reason==provider_error 过滤，改 reason 会静默破坏旧契约；(2) forbidden_use 不重复单池维度信息。状态仅 implementation/deployment verified，NOT runtime data capability proven。


## Observability instrumentation works but probe environment matters: local MCP runtime goes through Clash proxy → dominant failure=NETWORK_ERROR (push2his.eastmoney.com blocked). Production (Ubuntu, no Clash) may have different failure signatures. Probe必须从production-equivalent环境执行，否则failure layer归因可能归到proxy而不是real provider failure。How to apply: observability enablement后，Controlled Re-Probe必须先确认probe环境网络栈=production，否则NETWORK_ERROR可能是false positive。

`domain: finance-suite` · `2026-08-12`


Stock Provider Observability v0.1 Controlled Re-Probe: 4/4 NOT ADJUDICABLE domains classified. Dominant pattern=NETWORK_ERROR (Clash proxy blocks eastmoney.com). news=SUCCESS_WITH_DATA (different endpoint). financials=ADAPTER_ERROR (real bug). Environment: macOS local MCP ≠ production Ubuntu. Next gate: Failure Matrix Review (with environment caveat).


## 规则:人工probe证明代码可执行≠Scheduler Production Verified。cron真实运行证明生产调度路径不能完成Consumer才是production evidence。Probe只能把故障从business logic收窄到runtime/environment层，不能替代production gate验收。

`domain: finance-suite` · `2026-08-12`


根因:15:28 cron验收中，2 PASS(cron按时触发+目标路径正确)/3 FAIL(handoff未更新+Consumer未执行+无产物)。人工probe(12:48)已证明brief-engine代码可执行，但生产cron被macOS TCC拦截(Operation not permitted, ~/Documents受隐私保护)。Probe和production observation回答不同问题——前者证明代码逻辑正确，后者暴露runtime/environment缺口。How to apply:(1)production gate验收必须由生产scheduler真实触发，不能手工补跑后宣布VERIFIED;(2)probe只能做故障收窄(排除business logic defect)，不能做能力声明;(3)gate评分逐项独立，已PASS的gate(如cron fires)下一轮不需重证;(4)失败后拆独立卡修复，不混修不重跑冒充首次验收。


## 规则:前端展示的能力名称必须与生产endpoint实际返回的数据源一致。eastmoney_auction返回strong_pool+previous_zt≠Golden Pit factor_scan。UI不因入口关键词而伪造能力声明——如果产品入口保留"黄金坑"触发词,结果区必须展示真实capability name和data source。_qc≠Trust Gate,不能把qc.status=success解释为signal validated。

`domain: finance-suite` · `2026-08-11`


Auction Leadership Demo Final Readiness P0-1~P0-7。根因:auction.html的formatGoldenPit使用旧的{hits,watchlist,market_env} contract,但生产endpoint实际返回{_qc,data,count} from eastmoney_auction。修复后:前端contract与API一致,capability claim从"黄金坑四重门"改为"集合竞价强势候选池",所有legacy labels移除。how to apply:任何展示层变更后验证(1)UI count==API count(2)UI fields==API fields(3)capability name==actual provider(4)QC boundary不膨胀为Trust Gate(5)错误状态≠"今日无命中"。


## 规则：临时一次性cron调度≠生产安装。Brief Engine临时调度(2026-08-12 only)通过crontab + run_once.sh实现，只产出draft不approve不release，phase0.json保持installed:false。四个production blocker未解决前禁止升级为永久调度。风险：cron依赖绝对路径，调度期间不可移动目录或关机。

`domain: finance-suite` · `2026-08-11`


Brief Engine Temporary Schedule INSTALLED FOR 2026-08-12 ONLY；Production Install=FALSE；Permanent Enablement=BLOCKED BY PHASE0 PREREQUISITES。关键边界：08:28 morning draft / 15:28 close draft，draft only，exit code非零fail-loud，明晚删除cron条目。


## 本地代码路径≠生产代码路径。写前端前必须先 curl 生产 endpoint 验证实际响应 shape，不能从本地代码推断。

`domain: finance-suite` · `2026-08-11`


格式契约断裂模式: 本地 intel_api.py 输出格式A，生产 intel.py 输出格式B，前端按格式A写 → 所有数据静默丢弃。根因: 本地代码仓库和部署代码是两份不同的代码，中间没有自动同步机制。How to apply: 任何前后端对接任务，第一件事 curl 生产 endpoint 拿到真实响应 JSON，对着真实 shape 写前端。本地代码只能作为参考，不能作为契约。


## Designed runtime path must not be treated as production execution path. Local source inspection + G3 BLOCK probe on Tushare does not equal runtime capability. Before any runtime capability claim, verify chain in order: (1) Designed Path Identified, (2) Production Path Identified, (3) Path Convergence (designed == production), (4) Runtime Invocation Observed, (5) Decision Evidence Captured. Without step 3, subsequent layers may pass while production remains unguarded. Same meta-rule also applies to: Service architecture exists != Execution capability, API endpoint exposed != Write confirmed.

`domain: engineering-governance` · `2026-08-08`


## 规则: 逐行对照外部项目的功能清单打能力标签时,凡我方从未声明过的能力一律标 OUT OF SCOPE,不得标 NOT PROVEN。

`domain: finance-suite` · `2026-08-07`


原因: NOT PROVEN 的语义前提是『我方存在过能力声明但证据不足』,它在证据链里是一笔待偿的债;OUT OF SCOPE 的语义是『从未进入我方范围』,不产生债务。竞品对照表按对方功能逐行打标时,两个标签被压进同一列,结果是把对方的产品边界当成我方验收标准,凭空生成一份虚假能力债——看起来欠了一批工程,实际从未承诺。虚假债务比漏记更危险,它会牵引后续窗口去补根本不在路线上的东西。

适用: (1) 打 NOT PROVEN 前必须能回答『我方在哪份文档或哪个提交里声明过这项能力』,答不出即 OUT OF SCOPE;(2) 外部参照表与自证清单拆成两份产物,前者不进证据链,后者才进;(3) 架构不同构的项目之间不做逐行功能对照,只对照重叠的那一层能力;(4) 收到他人产出的对照表时,先审标签语义再读结论。


## Login PASS 不能推导 Provider ready——credential→session→consumer→evidence 链必须每步独立验证

`domain: finance-suite` · `2026-08-07`


根因：多个 Provider 的验证记录显示 credential 可用后直接宣布 ready 是常见误判。How to apply：transport 层 PASS 不能推导 capability 层 PASS；验证报告显式标注每步状态 VERIFIED/FAILED/NOT PROVEN，不允许跳跃推导。


## 规则: 任何能力声明前,必须先验证 helper 满足 Capability Chain Separation 五级(Exists / Invoked / Succeeded / Authorized / Production Proven),每一级独立取证,绿测试只证明 Exists 级.

`domain: finance-suite` · `2026-08-06`


写 helper 前先 grep 同义词(freeze/frozen/immutable/validate/sanitize),有更强实现就扩展或委托,不要并行新增。写完 grep 生产调用点(排除自身递归和测试),零调用即死代码,死代码的绿测试不得进入 evidence chain。冻结窗口内发现死代码不要私自删除或接线,标记 finding 交 Reviewer 处置。Why: 死代码加绿测试是最难识破的假证据——测试通过是真的(函数本身逻辑对),但它证明的是函数被调用时行为正确,不是边界被保护了。怎么应用: (1) 写前同义词 grep,若已存在更强实现则扩展而非并行; (2) 写后调用点 grep,零即死代码; (3) 冻结窗口中标记 finding 而非删除; (4) 任何能力声明前过一遍五级链检查。


## 测试通过只能证明 fixture 形状,不能证明生产路径行为 —— 断言必须落在真实函数返回值上,不能是手工构造的相似结构或源码静态解析

`domain: claude-code-infra` · `2026-08-06`


根因: 为避开依赖(import 重/需 mock 环境),写测试时会不自觉地(1)手工构造一个与期望输出同形的 dict,(2)用源码字符串解析代替执行,(3)在测试内联复现生产算法。三者都 PASS,但都不接触被测函数。Trust Gate Phase 1a 四轮对抗审查里 C 每轮发现的都是这同一缺陷: C-02 断言手工 dict 字段;C-03 静态解析 return 缩进;C-04 内联复现 decision_id 生成公式。统一修复手法: import 真实模块 -> try/finally 临时改写模块级开关(如 _TRUST_GATE_AVAILABLE=False)强制进入目标分支 -> 解析真实返回值断言。How to apply: 审查任何测试先问 —— 断言的输入是被测函数返回的,还是测试自己造的? 测试里若出现与生产重复的算法/常量/字段字面量,它证明的是自己而非生产。可迁移推论: 对类型为 Any 的字段做递归冻结,isinstance 分支必须穷举全部可变容器(dict/list/set/tuple-of-dict/bytearray),只覆盖 dict+list 会遗留 tuple 内嵌 dict 与 set 两条变异路径。


## 规则: 任何 Runtime Observation / RCA 在形成根因结论前,必须先把 production 源码拉回版本控制并 diff vs 本地 SOT;否则根因必须标 UNKNOWN,禁止用本地源码形成 PASS 结论。原因: 本地可验证 ≠ production 可验证。已观察到 4 类重复 drift — (a) production 使用 app/ 路径,repo 仅 scripts/ 路径,两边平行实现不一致;(b) deploy.sh 不覆盖 backend,生产代码独立演进不入库;(c) 独立复审环境来自不同 commit;(d) production formatter 与本地 formatter 输出文案不一致,本地源码是 PASS 但生产现场是另一份代码。适用: 对 production 行为做根因推断前;声称某个 module '不是 cause' 前;准备执行 remediation / 改缓存策略 / 改 scheduler 前。How to apply: 1) 任何 Runtime Observation 报告若 Root cause = PROVEN 但未完成 production source reconcile,该 PROVEN 必须降级为 UNKNOWN;2) Remediation gate 的前置条件是 reconcile,不是 deploy 脚本本身;3) Case Registry 中应单独立 R-02 Runtime Source Identity / Deployment Drift 条目,而非散落在各 incident RCA。

`domain: finance-suite` · `2026-08-05`


Auction P0 Observation R1 暴露 — 探针打 market-context 实时返回请求时间,排除 HTTP / Python / 本地 scheduler / 前端缓存五类错误方向,但用户实际路径 POST /api/analyze → app/routers/api.py → app/auction_data.py 完全不在本地 SOT,本地源码 100% PASS 不等于 production 100% PASS。同模式已在 C 独立复审环境、RRA release readiness、deploy.sh 不覆盖 backend 三处重复出现。


## 治理体系的非通过态被包装为通过是比缺失更危险的失败——已设计不等于已实现,已实现不等于已验证,每一项必须独立声明 enforce 状态

`domain: claude-code-infra` · `2026-08-03`


根因: 追求完成度时倾向合并不同成熟度的能力。How to apply: 治理记录中每个条目必须显式声明 enforcement status,文档记录和运行时拦截是两个独立维度的成熟度,同时存在非通过态和通过态是正确状态而非缺陷。
