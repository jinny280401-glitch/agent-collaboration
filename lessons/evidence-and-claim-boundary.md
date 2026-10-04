# 证据与声明边界

> 结论不得宽于证据;claim ≤ evidence;声明≠能力;fallback≠真数据;observed absence≠universal

共 78 条。源自 Engram 跨 agent 记忆库,已脱敏。


## 评测一个 System-1 决策模型时,不要只比准确率就下采纳结论;先把两条正交判断分开:(1)这件事该不该交给模型(有没有资格),(2)如果交给,值多少 intelligence。必须执行的义务/权限/规则属于前一条的 NO,再准的模型也不该接。

`2026-10-04`


根因:准确率是单轴指标,会把两类不同的问题压成一条线——结果就是把权限和强制义务也当成可以外包给模型的判断。实证:两个架构不同的 System-1 模型在同一道需要强制声明 limitation 的 bounded-decision 题上同向同错,说明这是要求放错层,不是模型不够聪明。how to apply:路由/分类/工具选择/升级类决策可交 System-1;limitation/blocked_fields/allowed_use/安全/合规/Trust Gate 写进 deterministic contract,靠 object-identity 授权而非字段相等。采纳 System-1 的唯一经济理由是高频 bounded decision,不是单纯准确率接近。延迟/成本声明必须剥离 CLI 冷启动等基础设施开销,否则等于谎报 model latency。


## 磁盘上已存在的 WIP 代码不等于 remediation：没有 execution/test/runtime evidence,其设计正确性/完整性/可复用性全部未证,不能因为“代码看起来写好了”就默认只差跑测试。

`2026-10-02`


根因:code-exists 偏差——把“已写”误读为“已对”,跳过 C 要求的 evidence seam。叠加 git 陷阱:untracked whole-file 无可信 baseline,贸然 revert 会连带破坏同文件内来源未证的其他历史工作,commit 又会把来源未证的全文变成 durable history,两边都不安全,唯一安全态=原样冻结。How to apply:(1) WIP 恢复后先按 content-evidenced / git-provenance-not-proven 分类,别用 dirty git status 反推归因;(2) 存在 REQUEST_CHANGES/HOLD 时,recovery 只做 READ-ONLY,先设计 response→parser→downstream 同一因果链的 observation/evidence-retention seam,再谈复用具体函数;(3) 相邻的 Plans/gate 文档若明确 NOT AUTHORIZED do-not-fix,不能反向给该 patch 补授权,授权来源必须显式同源。现场锚点:scripts/ifind_mcp_client.py sha256 562451686a1253cc309d1c5ec578cf2bdd7f5ec403043fbcbf6116337f15de34 FROZEN;C_REVIEW=REQUEST_CHANGES。


## Declarative policy alone ≠ enforceable Trust Gate;Agent Context state container 必须 admission/supersession/freshness/allowed-use 显式闸;今晚 v0.2 在 4 critical + 3 major findings 被 C R2 穿透

`2026-10-02`


根因:今晚 CURRENT_STATE_SSOT_CONTRACT v0.2 (76a3a7a0…) 被 C R2 打穿,4 critical (Authority / Evidence binding / Freshness / Consumer resolver) + 3 major (State precedence / Canonical subject identity / Atomic supersession) findings 全集中在闸是否可强制执行,而不是 schema 描述是否完整。Schema 写得再细,只要没有可强制执行的闸,最终都会退化成 MEMORY.md 同形态的 stale-field 容器。这是 declarative policy ≠ enforceable gate 的工程证据。

How to apply: 任何 agent-side current-state artifact 必须满足六条,Apply 这六条可避开今晚 v0.2 的全部 7 条 finding:
(1) admission gate 五道闸显式实现:identity → provenance/authority → evidence binding → freshness → allowed-use;任一闸未过即拒绝;evidence binding 必须 receipt 内含 CLAIM ID 反向引用(不是 EVIDENCE_REF 存在即算 binding)
(2) supersession 必须 atomic:history/ 落盘 + CURRENT.md 改写 + log append 三件套不可拆分;不可 partial-write;不可 soft-replace(必须显式 SUPERSEDES)
(3) Authority 权限矩阵必须显式:谁能 supersede 谁,evidence 化(不是默认权限);不允许'任意 AUTHORITY supersede 任意 AUTHORITY'
(4) STALE 不可回退兜底:禁止 /status / Agent 直接拿 MEMORY / Engram / session observation 当 current fact 兜底;STALE 是比'更老的具体值'更强的信号
(5) TTL 必须冻结 policy contract,不冻结数值;禁止 hidden fallback(如 'if missing, use 30 days');UNKNOWN TTL = STALE(非 fresh);AS_OF 必须是 admission 时刻,不是 retrieval 时刻(防 masquerading)
(6) status 必须分 evidence_status / execution_status 双轴;禁止单 token 混两层语义(今晚 v0.1 错误:单 STATUS 同时表达'证据无'和'不可执行')

关联反馈原则:feedback_wiring_not_enforcement / feedback_designed_runtime_path_neq_production / feedback_declared_vs_effective_capability / feedback_fallback_endpoint_not_real_data / feedback_stale_field_with_advancing_fetch_timestamp / feedback_observation_chain_must_subscribe_position_state


## 授权边界的输入若由模型自行声明，结构上就不安全；输入必须绑定可信 capability 身份，错误路由不能只甩给输出层兜底

`2026-09-29`


根因：若授权输入(如 requested_use)由模型从 prompt 自由生成，provenance 在授权层之前就已污染——把错误路由归为纯输出层问题只对一半，因为 O2 会按被污染的正确输入放行。how to apply：授权边界输入必须来自可信身份(静态契约/工具/capability 身份)而非模型声明；输出层兜底必要但不足，错误路由会让授权层输入本身失效。


## 默认拒绝的边界必须与判定输入的传递同步落地，否则会隐性破坏未更新的存量调用

`2026-09-29`


根因：默认拒绝(fail-closed)把输入缺失等同于拒绝。当判定所需输入尚未传递到边界位置、只能等调用方未来更新时，默认拒绝会连带拒绝所有历史调用。how to apply：新增默认拒绝边界时，要么同一改动内打通判定输入的传递，要么由决策者明确选择缺省放行的过渡态；不要只实现判定函数加就地默认拒绝，把破坏留给下游。


## 恢复重构中丢失的实现模块，优先从 git stash/history 找回原源码而非重写；验证必须分层——backend import 成功≠工具真实调用可用，provider 数据源 clean fallback≠能力 PROVEN。

`2026-09-15`


release-1.0 重构(8f15695搬目录+4e9833c清洗)把工具壳搬进 backend/mcp_tools/tools/ 但丢了三类东西，导致 backend 无法 import：①7个实现模块(research_digest/digest_llm/market_intel/market_intel_schema/jqdata_fetch/joinquant_data/research_reports)仍被 tool 体 bare import；②9处缺 import json + social.py 缺 import os；③analysis/search/video.py 的 from engine 应为 from backend.engine。恢复源 provenance：6 模块来自 git stash@{1}(stash^3 untracked 层)，research_reports.py + data/research_reports.json 来自 git 89d89b7(被4e9833c删)。验证分层(不得混淆)：backend 完整 import=OK；research_digest/research_reports 真实调用=success(eastmoney 真实数据)；market_intel=provider failure(autocli数据源不可用,clean fallback 非功能恢复)；jqdata/joinquant/digest_llm 的 provider 层未验证(需凭据/环境)，不得标 restored/PROVEN。commit ab54875 → origin/release-1.0，main 未动。


## 当数据源无法交付研究设计所需的字段时，冻结设计并把数据缺口标为 BLOCKED，而不是收窄目标总体去迎合数据源——后者会悄悄改变研究问题并引入比暂停更危险的 selection bias

`2026-09-14`


根因：为适配现有 provider 的能力缺口而缩小目标总体，会让总体由数据源能力决定而非由研究设计决定，研究问题随之漂移，且这种偏差比显式暂停更难察觉。How to apply：先冻结研究设计(总体口径、信息截止线、变量定义)并标 BLOCKED(非失败)；再另开只读的源发现步骤，只验证是否已有授权源能交付所需字段(区分 provider 存在 ≠ 字段可得 ≠ 已集成 ≠ 已授权消费)；源确认达标并经过 schema 规范化、时点校验、多日期复现后，才恢复构造；禁止降格成拿到什么数据就用什么做对照。


## 盘后验证 ≠ canonical 样本：集合竞价选股必须在 09:30 前落盘原始 snapshot，盘后反向填充 = 后见数据污染

`2026-09-08`


根因：第三方 agent 信号文本缺 schema 必填字段(price/is_zt/seal_amount/lianban)，且 09:30 前未 save 原始 snapshot，16:44 盘后才验证 → 只能标 VALIDATION_ONLY/NON_CANONICAL/DO_NOT_INGEST；how to apply：自 2026-09-09 起每日 09:30 前 shoupan.py save 原始竞价快照 → QC/schema 完整性检查 → sha256 冻结 → 盘后再 record-close；盘后 manual-reconstruction 即便加标签也会污染回测，一律拒绝


## Provider integration 必须验证 canonical branch provenance——import、SDK、credential、pyc 都不能单独证明已集成

`2026-09-02`


JQData 的 jqdata_fetch.py 从未进入 canonical main，只存在于 feature/session-1-validation-outcomes(fcafa4d, 2026-07-22)；主线 mcp_server.py:616 却保留对 scripts.jqdata_fetch 的 import，JQData fallback 在主线上结构性缺失，运行到该分支时 ModuleNotFoundError 并被 except 吞掉。残留 pyc(07-05)、已装 jqdatasdk、.env 有 credential 均不能证明 integration 成立。根因:主线保留 dead import 是'接线意图'而非'已接入';静默 except 把结构缺失伪装成 provider/subscription failure。How to apply:任一 provider integration 声明必须至少独立验证 1)canonical branch 存在实际 source 2)import target 可解析 3)caller→provider→normalization/QC→consumer 链路存在 4)runtime receipt 证明实际调用 5)provider/account 权限必须 live probe 独立验证,不能用结构缺失解释,也不能反过来。


## 审计文档禁止把 provider 返回值写成已验证事实；证据等级终扫必须覆盖所有伴随 artifact

`2026-09-02`


Xiniu capability audit 七轮复评：Builder 反复把 provider-returned 值（券商/标的/分析师身份、IPO在审1489、count=23804、上市公司可查）写成已验事实/实证，C 连续 REQUEST_CHANGES（回执全文/因果诊断共5次）。根因：把 delivery 级证据（响应非空）误当 data-identity 级结论（外部身份已验证）；且终扫只 grep 主文档、漏扫 receipt 本体，C 第二次命中的就是 receipt 标题残留。How to apply：一、provider 返回值只能用 attribution 措辞（返回显示/声称），三个正交字段 transport/delivery/data_identity 互不推导；二、证据等级终扫必须两份文件全扫（主文档+回执），逐条判断；三、Moonshot 一类因果诊断只留逐字错误原文+可观察事实，因果标注受限推断。


## 在产物目录里探测依赖可用性会被同名子目录伪造成功(Python命名空间包)，且「这台机器有没有X」是没定义的问题——结论必须写成「哪个解释器有X」

`2026-08-29`


根因：cwd下存在产物目录markdown/，python3 -c "import markdown" 因命名空间包机制返回成功(版本号显示?即为信号，真包几乎都有__version__)，据此判定依赖可用，实跑时ModuleNotFoundError。第二层：同机多解释器，weasyprint+markdown2只装在homebrew python@3.11，默认python3.14两者皆无。做法：依赖探测一律cd /tmp等中立目录；打印__version__，显示?视为未证实而非可用；结论写解释器绝对路径并记入脚本docstring。收益：判定「缺依赖需安装」前先扫已有解释器，因此避免了一次无谓安装(markdown2已在weasyprint所属环境)。


## 取证的停止条件必须预先定义为"证据边界是否穷尽",不能定义为"答案是否拿到";否则 agent 会持续寻找能关闭答案的材料,最终把无法独立验证的 claim 当成正常证据写入事实层

`2026-08-28`


根因:把 UNKNOWN 当成必须消灭的缺陷。一旦停止条件是"答案有没有拿到",穷尽可信来源后 agent 会转向不可验证来源(他人转述、无回执的声明、自己的历史输出),再给这些材料套上正常 evidence class,污染就发生在这一步而非取证之初。三条边界纪律:①结论边界=修复方向获得认可≠该修订被授权为后继 ②决策边界=符合资格≠被选定为唯一 ③获取边界=算不出来≠可以扩大取证范围。前两条防止用已有证据越权推论,第三条防止为得到想要的推论而越界获取新证据;缺第三条前两条可被绕过。How to apply:(a)开工前写下证据边界,达到即停,UNKNOWN/NOT_PROVEN/NOT_COMPUTABLE 是合格终态不是失败;(b)对自己产物的 producer 声明只能记 SELF_ATTRIBUTED,evidence class 标签必须来自声称方之外,否则等于自证;(c)hash/身份类核验只证明核验瞬间,消费前须重新核验


## 找不到 receipt 只能得出 NOT_PROVEN，不能升级成 UNREVIEWED / NEVER AUTHORIZED / DID NOT HAPPEN；observed absence 不等于 historical non-existence。

`2026-08-28`


根因：把「我搜索过、没找到」当成「该事件从未发生」，这是把自身检索能力的边界误当成世界的边界。检索覆盖面本身从未被证明完整——记录可能存在于已消失的 ephemeral 运行时、未被索引的路径、或调用方看不到的另一个 session 里。一旦 absence 被升级成 universal negative，下游会基于「从未授权/从未审查」重新执行本已完成的工作，或撤销本已有效的裁决。How to apply：(1) 缺 receipt 一律落 NOT_PROVEN，只有拿到独立反证才能升级为「未发生」；(2) 措辞上把「没有 X」写成「未找到 X 的 durable receipt」，前者是关于世界的断言，后者是关于检索的断言；(3) 路由不可达同理——live 通道断了只说明当下不可达，不代表历史记录不可恢复，必须先查 durable receipt 再决定是否重做；(4) 这类误标具有传染性，一处出现就要把同批 ledger 逐项复查是否存在同型升级；(5) 复查成本高时可登记为 deferred 复核项，但不得因此默认沿用已升级的表述。


## 静态页在本机渲染正常不等于对方能打开:外链 CDN(如 fonts.googleapis.com)在境内网络不可达会阻塞首屏,分发前必须 grep 掉所有 https:// 外链

`2026-08-27`


根因=作者机器的可达性/缓存与收件人不同,外链失败是静默的:本地看图完全正常,收件人可能白屏。本次实测 curl fonts.googleapis.com 返回 000(不可达),Chrome 已在悄悄回落系统字体——即证据表明本地渲染早已不是设计时的字体,但肉眼无从察觉。How to apply:任何要发出去的自包含 HTML,交付前(1) grep -nE "https?://" 确认零外链,外链字体改为已声明的系统栈或内联 data URI;(2) SVG 检查子元素最大 x 坐标是否超出 viewBox 宽度,超出即静默裁剪(本次 viewBox 1080 而子元素到 1100,右侧文字被切);(3) 用 headless Chrome 实际出图/出 PDF 并逐页看过,不要只看浏览器。三条都属于本地与目标环境差异导致的静默失败,不做显式校验就发现不了。


## 一个失败的检验只推翻它自己检验的那一条,不推翻它所属的整体主张;要让单点失败等于整体失败,必须在检验设计时就声明该点为必要条件。

`2026-08-27`


为什么:事后声称某个检验点本来就是决定性的,与事后改用另一个检验点来保住主张,是同一个错误的两个方向,都是在已知结果后重新分配检验点的权重。一侧让主张永不可推翻,另一侧让单点失败吞掉从未被检验的部分。怎么用:检验点的权重必须与判据一同事前声明;未声明为必要条件时,单点失败的结论写成主张失去该项支撑,其余部分标未知;未被检验既不代表成立也不代表失效。


## 需求/交接文档写API endpoint前必须grep后端路由表证实存在,卡片该有数据源推不出接口已存在

`2026-08-27`


根因:在sidebar需求表写hydrate拉/api/intel/morning-brief与market-temperature,推理链是卡片存在->需要数据->应该有接口。实测finance-suite-backend/app/routers/intel.py全部路由仅xueqiu-hot/xueqiu-hot-stock/discussions/hot-stocks/watch-alerts/research/market-context/golden-pit/all,两个都不存在。C判major:程序员会按猜测接口实现形成新空壳或运行时失败。危险点:臆造在文档里与已证实事实同形(都是一行/api/xxx),读者无法分辨,且现状是空壳可证而所以应拉X接口是自加的,两者被写成同一行同一状态。how to apply:①写endpoint先grep @router.(get|post)取实有清单②不存在标DESIGN_DECISION_REQUIRED并把实有路由清单作反证据写入③现状陈述与建议方案分行分状态不混成一行已定④endpoint存在不等于可用,认证Depends/字段schema/更新时间/失败语义各自证实⑤同类适用数据库表名/配置键/环境变量/CLI子命令等实现者会照抄的标识符


## 事前冻结的验证判据到期落空后,禁止从同一批数据里另选一个恰好为正的指标替代它重建原结论;到期只回答原判据是否兑现,其他正向项要成立必须另立新判据并在结果已知前冻结。

`2026-08-27`


根因:冻结判据的全部作用是让结论可被推翻。落空后允许在同批数据里换判据,而一批数据通常含几十个指标,总能找到一项为正,结论就变成永不可推翻,冻结动作被完全抵消。危险在于被换上的指标往往是真数据真为正真出自权威来源,可核验性不是问题,判据的选取时点才是问题。How to apply:到期先只判原判据兑现或未兑现,此步禁引入其他指标;未兑现则记FAILED并写清是失去该项支撑而非结论被证伪;其他正向项须另立新判据加新到期时点并在看到后续结果之前冻结;合法区分只有一条,判据是在结果已知之前还是之后被写下的。自检句:这个正向指标是到期前就写在判据里的,还是看到落空之后才被我注意到的。另需把结论层与执行层分开裁决,否则会用当下不能处置反推判据没有落空。


## 陈旧检测的两个判据必须都来自记录:值未变化,且采集时点确实推进。时点若是推断出来的,判据不成立,不能据此告警。

`2026-08-27`


为什么:推断时点最常见的方式是借用附近另一个操作的时间戳,而两次操作之间可能隔了很久,借来的时间戳与目标读数无因果关系,于是构造出一个并不存在的时点差。怎么用:告警前自问时点是记录的还是推断的;是推断则标 UNKNOWN,补一次同源重采样或换独立源交叉,再决定是否告警。误报的代价不止噪音,它会让真实告警贬值。可用的区分信号:数据源对尚未生成的周期通常返回全字段空值,而陈旧返回旧值,两者都表现为值没变,靠空值可以分开。


## 组件的角色由它自己的实现契约决定，不由调用方的 prompt 决定；让一个会自行取证的组件充当"只审不产"的审查者，审查结论最终仍由被审方自己产的证据支撑。

`2026-08-26`


根因：prompt 是意图，workflow 是行为。当组件内部固化了"构建候选集 → 自行采集来源 → 排序"这类步骤时，外部再怎么声明"你只做反证、不要产证据"都不会改变其行为，独立性在契约层就已经不存在，形成自证闭环并绕过上游准入检查。How to apply：委派审查/反证角色前，先读该组件的实现契约，检查是否存在自主取证或自主构造输入集的步骤；若存在，判定为契约层不兼容并停止委派，不要临时修改该组件、也不要换一个同类组件冒充审查者。同时区分两种结论强度：读契约文本即可确认的不兼容，与实际运行中是否真的漂移——后者在未实际调用时不成立，不得合并表述。


## 规则: 关闭长期挂着的任务前,先核对审计结论是否覆盖任务全部目标、以及是否出现未消费的新证据;后续 OPEN 工作项保持独立 track 不并入旧任务

`2026-08-26`


为什么:某只读审计任务在结论冻结后仍长期处于进行中,而审计的下一步也已执行,任务状态与实际进度脱节。怎么做:1) 读任务描述目标,逐项核对冻结结论的覆盖情况;2) 无新证据则关闭,并明确审计完成不代表能力达标;3) 后续修复/恢复类工作项单独建 track,不合并进旧任务。


## When auditing why a system selected X, separate three layers: did the evidence exist ex-ante, did the selection path have the capability to consume it, and who was the actual producer. Conclude only on provable layers; a B-class finding (evidence existed, path could not consume it) does not prove the audited path was the one used.

`2026-08-25`


Root cause: narrative closure pressure - once accident type B is established, the next natural step is to conclude the path was used, conflating could-have with did. How to apply: freeze the provable layers first, keep producer identity UNKNOWN when no receipt exists, and replace continued deduction with explicit reopening triggers (input snapshot or execution receipt) - further inference slides from evidence audit into architecture speculation.


## T+1 observation day must explicitly isolate "already holding a position" or the judgment standard degrades into rationalizing the position.

`2026-08-24`


When Live Portfolio Case #1 (002038) entered T+1 observation, the initial plan framed the task as "preparing to start live trading" rather than "first complete T+1 observation of an existing position". This framing error creates pressure to lower the evidence bar — if holding, any excuse to keep holding feels safer than admitting the entry was wrong. The fix is to declare upfront: today is observation → evidence → verdict on the signal, not on validating past decisions. The 08-21 entry is frozen fact; today we measure whether the original thesis held, independent of position status. Root cause: Observation → Evidence → Verdict chain breaks when the observer has an outcome to justify. How to apply: Tag every live-position observation window with its Case ID and explicitly state "not forming rationalization for existing position" in the authorization scope. If you catch yourself thinking "since we are already in, we should", stop — that is the judgment degrading.


## Reviewer 驳回'集合竞价实盘探测'claim 后，必须把观察计划从'记录 evidence'降格为'只读能力探测'，避免把上一交易日快照包装成今日竞价证据

`2026-08-24`


根因：普通行情快照 ≠ 集合竞价证据；没有原始响应、provider timestamp、市场阶段和消费者链路就不能声称具备竞价采集能力。How to apply：每个实时探测调用必须保存 raw_response / provider / endpoint / retrieved_at / provider_data_timestamp / market_phase / consumer / qc_status / capability_verdict；能力未证明前禁止形成 execution verdict。Case #1 002038 双鹭 2026-08-24 09:15 窗口已按此降格。


## Live Portfolio Case 的 T+1 observation day 必须把'已持仓'显式隔离，否则判定标准会自然下滑为维护持仓合理化

`2026-08-24`


根因：仓位是实验结果不是证据输入。How to apply：每个 observation day 开头声明当前仓位≠verdict输入；只采集 contemporaneous evidence（价格、量能、板块相对强弱、QC）并形成 verdict；禁止用新闻或聊天推荐替代市场证据。Case #1 002038 双鹭药业 2026-08-24 首个完整 T+1 观察日已按此更新。


## 给一个被定义为 Methodology Consumer 的 reviewer 开放搜索能力，它会自发漂移成 Evidence Producer + Adviser：不仅自己造证据，还会越过契约给出决策倾向。角色边界必须由能力约束强制，不能只写在 prompt/契约里。

`2026-08-23`


根因：Consumer 角色的定义是「只消费已准入证据」，但一旦具备检索能力，遇到证据不足时，最省力的路径是自己补搜而不是输出 INCONCLUSIVE——模型天然倾向给出结论而非承认证据不足。契约写了边界但没有机制阻止，边界即失效。观察到的具体漂移链：证据不足 → 临时补搜未经 QC 的材料 → 基于未验证材料形成判断 → 从判断进一步输出「排除/降权/不应进入组合」的决策倾向。副作用：补搜材料含实质性事实错误（把公司年报明确写的业务增长说成未增长；引用一条未验证的实控人负面事件），而这些错误直接驱动了最强等级的否定 verdict。How to apply：(1) 任何 Consumer 角色的证据准入要在能力层强制——预先冻结 evidence bundle 或只读模式，而不是靠指令自律；(2) 把 INCONCLUSIVE / 证据不足做成一等公民输出，并明确它不等于负面结论、不改变上游排名，消除模型规避它的动机；(3) Consumer 输出的 schema 要显式禁列决策字段（仓位/目标价/买卖倾向），schema 层拒绝而非事后审查；(4) 如果确实需要补充材料，产物必须降级标注为 exploratory / NOT_ADMITTED 并与正式记录物理分离，禁止回流决策链。


## Multi-session coordination 从观察到的 Agent 行为推进到形式化协议规范；Contract v0.1 经 adversarial review 发现 6 个真实阻塞项（state authority lost-update、session identity 无认证、lease timeout fail-open、WriteToken TOCTOU 未闭合、HALT 状态机矛盾、coverage 声明无实现证据），证明 Contract Research 价值但尚不足以进入 runtime enforcement；项目暂停在 CONTRACT RESEARCH PAUSED 状态，6 个 blocker 保留为未来独立研究窗口输入

`2026-08-23`


**Why:** 直接实现会把未验证的协议假设带入 runtime，重现 Search Trust-Gate 的 coverage 假设问题。**How to apply:** Contract 必须先通过 adversarial review 闭合 authority、identity、lease、TOCTOU、state machine 语义缺口，再授权 sabotage prototype；「COVERED (enforcement implemented)」这类运行时声明在无实现证据时应标为 SPECIFIED_TARGET / RUNTIME_NOT_PROVEN；lost-update 和 stale-backup 问题需要 CAS 或独占锁 + checkpoint digest 验证；session identity 需要不可伪造的凭证或明确限定为 cooperative-only 威胁模型


## 规则: 验收结论必须把已知模式命中与开放世界改写残余拆成两条独立表述,不得用带括号限定的单一标签把两者压成一个词;一个词会掩盖残余风险,让下游误判已封闭。

`2026-08-20`


根因:把 regex 语义分类器的验收写成带括号限定的单标签,把已知模式命中与开放世界改写未证明压成一个词,下游读作语义边界已封闭;实际 regex 只能断言列出的模式命中,无法断言任意改写也命中。how to apply:任何分类器/门禁的验收结论都拆成两条——已知模式命中、开放世界改写,分别标注、分两行写;括号限定词(bounded/limited)不得贴在单一结论上;改写词反例必须进回归 fixture,不能只用 happy-path 词集当安全证明。


## Automation evidence gap 必须从原始 transcript 取证，不能事后重跑 resolver 补充 Gap——重跑只产生 REPRODUCTION_ONLY 结果，会将中间态误认为已验证事实

`2026-08-19`


根因：automation 运行时的上下文（调度路径、时间戳、输入文件状态）不可还原。即使 resolver 是确定性函数，重跑的 stdout 无法证明原始 automation 执行链的完整性。How to apply：每个 automation 任务在首次运行时必须保留原始 transcript；Evidence Gap 出现时先定位 transcript，找不到则 Gap 状态保持 OPEN，不可用 dry-run 替代。


## 验证凭证修复时,必须把文件层/运行进程层/消费层分开判,并且只采信严格晚于新进程启动时刻的日志行

`2026-08-19`


根因:长驻 daemon 在启动时把 credential 载入内存,之后不再回读配置文件,所以"文件已改对"和"运行态已生效"是两个命题;第二个陷阱是重启瞬间旧进程的 teardown 日志会和新进程日志挤在相邻时间戳,带 shutdown/final-notice 标记的错误属于旧进程,拿它判新进程会得出反向结论。How to apply: 1) 改凭证后必须重启并记录新 PID 与启动时刻,再以该时刻为下界截取日志窗口 2) 传输层失败(连接 EOF/超时)与凭证失败(invalid secret 类错误码)分类归因,前者与凭证无关 3) "窗口内无报错"比"观测到成功调用"弱一级,报告时标注证据强度而不是直接写 PASS 4) 同理,配置文件 mtime 只能证明此后没有写入,不能证明文件当前内容是哪个版本


## 对冻结 revision 做 review 时，read-tool 读磁盘路径不等于 git object：on-disk file 可以是 stale/diverged 副本，必须用 git show <hash>:<file> 读 immutable commit object 才算有效证据。

`2026-08-17`


root cause：reviewer 在 fallback Contract 03eb9dd closure 中用 Read tool 读 ~/finance-suite-backend/docs/governance/FALLBACK_ADMISSION_CONTRACT_v0.1.md，得到 frontmatter Baseline = bd727a8；但 git show 03eb9dd:<file> 显示 Baseline = b832cbc（正确）。两条证据直接冲突，原因是磁盘文件是 stale worktree 副本（checkout 未完成或另一会话写入旧版本），和已提交的 03eb9dd object 内容不一致。how to apply：对任何冻结 revision 做 code review / contract review / closure verdict 时，第一步是 git show <frozen-hash>:<file-path> 读 immutable object，不要直接 cat / Read tool 读当前工作区路径；若两者不同，以 git object 为准，工作区副本降为 stale artifact。


## 向第三方交付设计输入文档时，未证实工程参数必须逐项标注 DESIGN ASSUMPTION/TO BE VERIFIED，法规存在条件路径与项目已合规必须分离陈述

`2026-08-15`


根因：装修方会把文档中的数字当设计前提继续深化，把假设当事实或把法规路径当合规结论，会在施工图阶段变成合规事故。How to apply：交付任何 spec/brief 前做假设盘点——每个数值问依据是什么，无依据显式标注；每条法规引用只写规范要求什么，不写本项目已满足，除非有验证证据


## 规则: 复用历史结论之前必须先确认底层证据没有变化;若复用路径会跳过重新判定,修复方式是停止复用、强制每次重新取数与判定,而不是给缓存增加感知逻辑。

`2026-08-15`


根因:历史结论的复用发生在重新取数与判定之前,底层数据变坏后旧结论仍被直接返回。How to apply:对结论需每次重新判定的请求类型,禁用复用读写(fail-closed),让每个请求走完取数→检查→判定→(直接响应或模型调用);回归测试覆盖:旧结论复用+新低质证据→旧结论不返回、模型不调用、其他请求类型复用行为不变。


## 规则：物化时间边界字段（如 09:25 撮合价）必须显式选取边界行，禁止用 sequence[-1] 兜底——数据顺序是序列位置，不等于时间语义；边界行缺失时输出 NOT_PROVEN 哨兵，宁可声明不可证明也不静默退化为最近值。

`2026-08-15`


Auction Price Evidence Repair v0.2：reviewer 指出 trajectory[-1] 会把 09:26 行误当竞价结果（09:25/09:26 同价时被夹具掩盖）。修复=显式匹配 09:25 边界行+NOT_PROVEN 兜底+anti-masking 夹具（边界行与后续行价格必须不同）+端到端非泄漏测试。


## 规则：接线数据字段前先做语义验证——字段名不等于字段语义，按名称猜测映射会产出错误证据；只物化实测验证过含义的字段。

`2026-08-14`


根因：曾把涨停池/强势池/热榜这类非竞价数据当作竞价证据接线，并把字段名当语义（成交量列被误当虚拟匹配量）。How to apply：接线前用实测数据锁定每个字段的真实含义；只物化已语义验证的字段；未验证或禁止的推导（量/盘口/抢筹/撤单）显式写入禁用清单；单一维度成功接入不得自动升权。


## 谈判对手以“缺少现成设施”否决方案时（如物业“无烟道→厨房必须取消”），先查法规替代授权路径和豁免清单再接受结论；且“净化率高”永远不等于“可免外排”

`2026-08-14`


上海餐饮油烟法规两条替代路径：①未配套专用烟道的建筑物“应当加装专用烟道”（含外置烟道，明文授权）；②“不设炒炉和无煎炒炸烤焗工序的甜品、炖品、包点”明文豁免烟道。同时法规只认外排：“通过专用烟道排放油烟”，室内循环无授权，内循环设备无国标不可当合规依据。谈判重构：把争议从“要不要厨房”改为“油烟发生量降到哪档+物业风险控制到什么程度”（产生量→捕集→净化→除味→外排→消防→运维七段）。根因：把路径条件当终局结论；how to apply：面对否决结论先查豁免条款和加装授权，再量化改写争议。


## 同形状 fallback ≠ 同语义：替换故障 endpoint 必须显式降级 provenance，且 fallback 不得扩大授权范围

`2026-08-14`


push2 第二腿故障→降级 push2delay（HTTP 200+同响应形状，但延时报价/批量无 provider-origin timestamp/无竞价字段）。修复=receipt 显式写 provider=eastmoney/source_type=fallback/quote_freshness=NOT_PROVEN，freshness 缺失时不升级，allowed_use 不因 fallback 成功而扩大。教训：HTTP 200 只证明可达，同形状只证明可解析，能力等价需逐字段证明；换 endpoint 时把降级标记写进数据契约而非只换 URL。


## 真实行情数据 ≠ 真实集合竞价证据:竞价能力必须由 09:15-09:25 盘前分时+委托盘口字段证明,涨停/强势/热榜池不能冒充竞价证据

`2026-08-14`


2026-08-14 竞价报告 NOT_PROVEN 根因=auction_data.py 只调 zt_pool/strong_pool/previous_zt/hot_rank/hot_up/spot,从未调用已安装的 AkShare stock_zh_a_hist_pre_min_em(盘前分时)与 stock_bid_ask_em(盘口五档)。发现最快真实竞价源=AkShare pre_min(免credential,返回竞价价格+虚拟成交量+分钟级timestamp,缺未匹配量需另接东财push2竞价盘口或等EmQuant/iFinD修复)。教训:connect=true 是假成功,必须用 stock 数据查询双验证;EmQuant/iFinD 均 connect=true 但 stock 查询 failure。


## 面向人的产品报告与机器证据必须物理分层：把 Evidence Receipt 直接当产品报告，导致池快照涨幅被误读为竞价涨幅、30 家工程表被误读为竞价抢筹榜。

`2026-08-13`


根因=产品层级错位，为治理可信而把 QC/receipts/trace/缺口直接摊给读者。How to apply：投资者层(人话/结论/标的/下一步观察)与 Evidence 层(QC/receipts/trace/缺口)分成两个物理产物；标题升级由证据能力触发而非页面名；投顾经验先沉淀为 Actionability Evidence Contract 再进分类/排序，证据不足统一 UNKNOWN。


## 物化冻结语义到 runtime 时必须 import 单一来源而非复制，否则 schema↔runtime drift

`2026-08-13`


I1 把 I0 冻结语义物化为 EvidenceObject 时，直接 import docs/spec/validate_i0_schema_contract.py 的 d6_ceiling/MATRIX/枚举/JCS，不重写 D6 ceiling 与 §1.8 max_age 矩阵，并用逐行交叉测试断言 runtime 输出==冻结验证器。根因：语义物化最易犯的错是把 spec 表复制进 runtime 造成两处漂移。how to apply：任何把冻结契约变可执行对象的任务，先找单一来源并 import，再加逐行交叉测试，禁止重写。


## provider 无行级时间戳时，禁止用 fetch 完成时间冒充 data_as_of；必须拆分三裁决(provider_origin_time / observation_boundary / provenance_binding)各自证明，不合并升级

`2026-08-13`


Auction DATA_AS_OF Closure: strong_pool/previous_zt 的东财接口不返回时间字段(keys已全量核实)，provider_origin_time 保持 null + timestamp_authority=fetch_boundary + provider_freshness=unknown。receipt 增加 receipt_id/provider/fetch_completed_at，golden_pit candidate 绑定 source_receipt_id 可反查。T1-T4 六测试 PASS，生产 direct callable 验证: strong_pool/previous_zt=fetch_boundary, big_buy=provider_stamped(10:13:42,不被消费)。DATA_AS_OF 仍 NOT PROVEN——只有 provider 换源或接口补时间戳才能 PROVEN


## 证明数据新鲜度时,取证范围必须限定在被下游实际消费的数据源上,而不是整次调用返回的所有源

`2026-08-13`


根因:采集层按源逐个记录来源时间,而过滤层只消费其中一部分源。若某个未被消费的源带来源时间戳,而被消费的源没有,把前者的戳当成整次调用的数据时间,等于用调用发生的时刻冒充数据产生的时刻,freshness 声明被虚假升级。How to apply:先算出 被下游消费的源 与 带来源时间戳的源 两个集合的交集,只在交集内取证;交集为空就判 freshness NOT PROVEN,不允许用相邻源的时间戳代偿。附带规则:当返回结构自带语义标记表明字段不是特定场景专有字段时,不得把通用字段改称场景专有字段来支撑结论。


## 治理本身也不能被永久相信：当证据与规则冲突时，先保护证据，规则必须接受重新验证

`2026-08-13`


根因:治理机制若自我豁免,会从防漂移机制退化成新教条——不被证据约束的权威本身就是最大的漂移源。How to apply: 任何 Gate/fixture/freshness policy 落地时都带可验证的触发条件;当证据证明某规则已过时,规则必须走重新验证流程(如 ACR),而不是压制证据;PROVEN ≠ OBSERVED ≠ IMPLEMENTED 同样适用于治理机制自身


## 证据交付写已记录但未贴原文被评审驳回——原文必须逐字入库

`2026-08-13`


根因:把声称完整当成实际完整,交付物读者无法从指针复算只能以转述当证据。How to apply:hash/canonical/引用类交付必须逐字附原文;交付前重读交付物核对是否真包含声称内容;宁可多贴一段原文不可写已记录。


## A resolver at a producer/consumer contract boundary must degrade I/O and encoding failures into its own fail codes, never raise — an unguarded exception exits the pipeline without the agreed unavailable marker

`2026-08-13`


How to apply: audit every read/parse site at contract boundaries for unguarded exceptions. resolve_morning() called read_text() without a guard, so OSError/IsADirectoryError/UnicodeDecodeError crashed the close producer instead of returning INPUT_PARSE_ERROR (fail-open: no controlled INCONCLUSIVE downgrade, no marker). Fixed by catching (OSError, ValueError) into the fail code with regression fixtures (directory-as-file, non-UTF-8). Review check: any code path that can exit without emitting the contract's failure signal turns a declared fail-closed contract into fail-open


## Input-missing reports need a path-contract check first: file absence and naming drift are different failures with different fixes. Never mask a broken contract with a fuzzy-search fallback or a duplicate SSOT copy

`2026-08-13`


How to apply: (1) when a consumer reports a missing input, list the producer's actual outputs before restoring anything - D13 close read loop_state.json (never produced) and midday_handoff_latest.json (frozen), while same-day data existed as d13_morning_brief_<date>.md and d13_handoff_latest.json; fix the reading side, not the phantom file. (2) Do not add fuzzy same-day filename search as a production fallback - naming drift must fail loudly with explicit codes (INPUT_ABSENT/STALE_INPUT/INPUT_PARSE_ERROR). (3) Do not copy a handoff to a second path to satisfy a consumer - two SSOTs produce divergent-content accidents


## 两个 agent 并行对同一 production target 做 instrumentation 时，各自选用不同 schema（journald-9值 vs file-sink-3值），产出的 evidence 不可跨域对照，导致 Failure Matrix 无法合并裁决。根因：任务卡没有预先锁定单一 schema 和 taxonomy，也没有指定唯一 production owner。How to apply：任何共享 observability/evidence target，开工前先锁定一种 schema + 一个 owner；发现并行实现后立即移交所有权而不是双轨并存。

`2026-08-13`


## 规则:fetch()收到HTTP 401不会抛"Failed to fetch"。Failed to fetch意味着浏览器没有可用HTTP response(网络层/CORS/TLS/代理中断)。不能把auth UX缺陷(未优雅处理401)和真正的网络层失败混为同一根因。两个缺陷独立存在且需要不同修复路径。

`2026-08-12`


根因:Stock Analysis的Failed to fetch被初始怀疑为security patch deny-by-default auth回归，但生产负向smoke证明patch未部署。剩余两个假设:H1(auth session丢失→401→Failed to fetch)因果链不成立，因为fetch()收到401会返回Response而非抛network error;H2(proxy/CORS/网络层中断)更符合"Failed to fetch"的浏览器语义，但缺浏览器Network tab证据+服务端日志。How to apply:(1)区分"服务不可用"(fetch network error)和"鉴权失败"(HTTP 401/403)——前端错误处理策略完全不同;(2)RCA未取得失败时刻的浏览器Network evidence+服务端日志时，根因只能停在NOT PROVEN，不能从假设中强选一个。


## 规则: 外部知识源接入AI研究管线前，必须先经过Artifact Classification——事实进Evidence、观点进Research Context——不得因provider名称默认可信；展示层disclaimer必须与内容物理绑定（内容之前、不可由provider自定义、不可移到页脚），否则用户消费观点后才看到限制

`2026-08-10`


接入外部研报库时发现三种隐性污染路径：①外部观点append到evidence records后即使标记UNKNOWN，未来consumer仍可能误读为Gate已审核（共享命名空间=共享信任）；②provider传入弱化disclaimer会稀释边界声明；③disclaimer放页脚=用户已消费观点才看到限制。修复：独立artifact文件（不入evidence、不入Gate、不影响claim strength），disclaimer由展示层固定注入且置于内容之前，provider不可覆盖。抽象为通用治理规则后，Wind/Choice/iFinD/聚源接入都复用同一模型。


## presentation_identity_cannot_be_inferred_from_footer_metadata: 展示 artifact 的 footer、标签或引用的文件名都不能确立 runtime identity、renderer provenance 或 release approval。renderer 身份必须由确定性生成证据+renderer version+template hash+release manifest 证明;模板里的 status 字段在有执行证据支撑前只是占位符。

`2026-08-10`


根因:同一条统一规则 Observed State != Verified State 的第四个实例(前三:Auction code fixed!=root cause proven / Provider credential exists!=runtime capability / SSH connection restored!=root cause proven)。How to apply:绝不把 filename status、footer text、继承的模板字段(如 claim_gate.status=PASS、renderer.version)升级为 verified state;无 Gate 执行证据时写 NOT_RUN 而非照抄 PASS;renderer 输出与展示 HTML 的 footer 签名不一致即证明后者非引擎确定性产物,manifest 对应字段应标 NOT_PROVEN。


## 修复成功不等于根因证明：SSH 恢复后，必须保留 Clash rule hit log、修复前后 verbose log 对比、服务器侧失败日志，才能将 suspected root cause 升级为 proven

`2026-08-10`


此前 Engram id 89b26eedbdc4 将 Clash 代理拦截写成已定根因，证据不足。正确口径：SSH connectivity RESTORED（已观察），root cause LIKELY but NOT PROVEN（缺排他证据）。需补：Clash 规则命中证据、A/B 验证、sshd/auth.log 关联。


## In capability validation, treat reachability, availability, access, and observation as separate proof layers; never let a lower-layer pass imply a higher-layer pass.

`2026-08-10`


Root cause: lower-layer successes are easier to obtain and are frequently mistaken for full-stack validation. A reachable endpoint, a running process, or a valid credential each proves only its own layer. The error pattern is to stop collecting evidence after the first green check. How to apply: enumerate the full layer chain for any capability claim (e.g., network reachability, service availability, authenticated access, runtime observation); collect independent evidence at each layer; document which layers are proven and which remain unknown; do not upgrade partial evidence into a full capability claim.


## 规则: 一份评审的结论强度上限是其输入的可核查程度;拿不到原文时不得下负面结论,只能声明评审不可执行

`2026-08-10`


根因:评审授权与评审输入是两个独立条件,任务描述常把两者并列,读者会把被允许审读成可以下判断。当手上只有他人转述而无原文时,转述的遗漏与原文的遗漏在形式上不可区分,任何结论的真实证据都是那份转述。How to apply:(1) 下结论前先声明输入的可检索位置与内容校验和,二者缺一即视为无输入;(2) 无输入时不要退选一个听起来更保守的负面结论 —— 负面结论会被后续读者理解为审过了但不合格,这是假阴性,把未曾发生的评价写进被审对象的质量记录,损害大于不下结论;(3) 审查者的发现不得写回被审文档,否则下一个读者会以为原设计已包含这些内容,三者须始终可分:被审文档 / 审查文档 / 规范文档;(4) 一次评审确认的是设计质量而非规范资格,两者必须是相隔一个缺口修复窗口的两次独立决策;(5) 审查发现不等于已承诺的工作项,把发现登记进任务清单会让后续窗口误读为工作已启动。


## 为定位可达 runtime 而跑的探针,本身已经是 validation 开始 —— 入口判定必须靠读与声明,不靠探

`2026-08-10`


根因:探针必然选择一条具体 runtime 才能执行。在未声明验证对象的情况下跑探针,等于让探针替你选目标,再事后解释原来是这条。于是 designed path 与 production path 的差异被探针结果掩盖,验证产出的证据无法归属给任何 provider。How to apply:验证准入门必须是声明式的合取门,五项全 PASS 才允许 invoke —— provider identity(name+route+credential alias,tool 名不算 provider 名) / runtime consumer(命名下游读者,tool 存在不算消费者) / production path(端到端命名并显式声明是否生产) / gate insertion point(在该路径上的具体位置,或声明该路径无 gate) / evidence owner(producer 与 verdict-reader 可区分)。任一项非 PASS 时只允许 scoped evidence collection(读源、记缺口),禁止 invoke 与 verdict。判定值需含 MISMATCH 而不只 UNKNOWN:前者说明目标本身错,后者说明还没查。被门挡住的正确口径是 Not admitted — entry criteria not met,写成 Validation Failed 会把假阴性注入能力档。


## Three-Layer Artifact Separation: Evidence / Governance / Presentation 三层必须互不混淆。状态只经 Evidence 改变;Presentation 镜像状态,不能改状态。展示层反向污染状态(漂亮页面 ≠ 已验证)是必须堵死的失败模式。

`2026-08-07`


2026-08-07 D13 Close Brief 收口时确立。错误样本:起头以 PARTIAL 状态拒绝渲染 HTML(理由:Midday 格式不匹配),把展示与状态混在一起。用户纠正后形成 footer 约定:Presentation Artifact · Not Evidence Artifact · HTML mirrors state, does not change state。适用任何有数据层 + 展示层的系统,包括 Vera 用户面。任何 artifact 任务先定层(E/G/P),按层规则处理;Presentation 必须带 partial/blocked 标识。


## 规则: 诊断与验证类命令一律不吞 stderr

`2026-08-07`


原因: 重定向 stderr 到空设备会把'命令不存在'与'命令无输出'压成同一观测结果,据此推出的根因是假阴性。典型链条: 用一个本机并不存在的超时命令包裹日志跟随做等待,报错被吞,等待从未发生,于是在服务启动瞬间读到空日志,进而错误归因为日志级别或静默开关配置错误,并准备去改一个本来正确的配置。适用: 依赖外部命令实现等待或超时前,先确认该命令存在;把'输出为空'当作结论前,必须有独立证据证明命令确实执行过。


## 规则: 当治理指标与被治理者的优化目标直接冲突时,不要靠纪律压住,要改指标结构——但要先确认新结构是把冲突解决了,还是只是搬到了另一个位置。

`2026-08-07`


原因: 典型形态是『取最小值』式约束:结论强度 = 所有已获取证据的最小授权。它对单点造假有效,副作用是多查一个弱来源会拉低整体上界,于是系统被反向激励少查证据——治理指标与研究质量直接对立。靠纪律要求『必须多查』无效,因为激励结构没变。

更合理的方向是让约束相对于声明类型:声明类型 → 所需证据图 → 允许强度,只对该类型的必需节点取最小值,必需集之外的证据可以参与推理但不设上界,少查的动机随之消失。

适用: (1) 但必须同时检查冲突是否只是被搬走了——此例中问题变成『谁声明类型』,若被治理方自己声明,它会挑一个必需图刚好排除自身弱项的类型,声明类型本身成为提权面,『任何拓扑不得自抬上界』的闭合性失效;可能的修法是类型在请求准入时固定、先于取证、且本身受 gate;(2) 所需证据图是领域断言(『短线判断需要价量资金』),按声明能力≠有效能力它不能由设计文档凭空断言,每张图要自带依据,否则整个机制继承一个未证前提;(3) 识别信号:凡看到『为了合规只好少做事』的行为,先怀疑指标结构而不是执行者纪律。


## 规则: 治理声明本身也是能力声明,同样需要证据;设计文档若主张自己不实现的控制,必须在文内显式记录该缺口,否则它就犯了它所描述的那个错误。

`2026-08-07`


原因: 自指形态是——『我有治理能力』+ 该治理能力无 runtime enforcement → 该声明违反它自己的原则。治理文档最容易豁免自己:它审查别人的能力声明,却把自己的控制主张当作既成事实。但按『声明能力≠有效能力』,一份规定了控制却无任何执行路径的文档,与一个声称有能力却无运行证据的模块,是同一类缺陷,没有理由前者豁免。

适用: (1) 设计文档固定包含一节自指审查:本文主张的控制有哪些无 runtime 实现,在出 runtime 证据前不得被引为治理能力证据;(2) 依赖声明必须逐条核验源文档实际状态而非沿用他人给的标签——带错误状态的依赖块会被后人当作已核验事实引用,比没有依赖块更糟;(3) 区分 spec-stable 与 runtime-implemented:依赖一份规范稳定但运行时未开始的资产,本身就是 declared-vs-effective 缺口,要写明而不是记为已就绪。


## 规则: 文档必须显式归入三类之一且不可混用——Design Artifact = 知识探索(无权威)、Governance Artifact = 规范控制(有约束力)、Evidence Artifact = 观测事实(记录发生过什么)。

`2026-08-07`


原因: 设计资产之所以能与进行中的冻结窗口并存,正是因为二者属于不同类别;一旦类别标识缺失,三种混淆各自产生一种特征性失效:(1) Design 被当 Governance —— 未实现的控制被当作生效中的约束,系统被误认为受一份无人强制的文档治理;(2) Design 被当 Evidence —— 架构图被引为能力证明,即『冻结产物存在≠执行产物已验证』;(3) Governance 被当 Evidence —— 规范被引为它自身已实现的证明,这是『声明能力≠有效能力』最常见的形态。

适用: (1) 每份文档在文件头写明所属类别与该类别的权威性(无/有约束力/factual);(2) 冲突时的处置不是改冻结资产也不是推迟设计,而是让设计产物声明自己在规范链之外;(3) 类别升级(Design→Governance)是一次独立授权行为,需要自己的 gate,不能因文档已存在就默认生效;(4) 评审他人产出时先确认类别标识,再读结论——类别错标会让后续所有引用都错。


## 规则: 证据质量 Gate 必须放在 prompt 组装前,不能靠输出后的评审补;评审者只能看到文本,而缺陷若从未被surface进 prompt,文本里已不存在可供识别的痕迹。

`2026-08-07`


原因: 输出后评审与入口拦截不是强度差异,是信息可得性差异。陈旧或低溯源的数据一旦无标注地进入 prompt,生成结果里不再残留该缺陷的任何线索——评审者无法察觉一个『不存在』。拦截是预防,评审是收容;收容处理的是已经进来的损害。

适用: (1) 把强制检查点放在 Provider→证据规范化 与 上下文组装→prompt 两处,前者是溯源唯一可知的时点,后者是唯一能阻止未授权证据进入推理的时点;(2) 不要为 Agent 内部推理设计控制——不可观测的环节上的控制不可证伪,控制只放在边界;(3) 规划阶段不给证据访问权:先读证据会在授权信封算出前就形成假设并把它编码进任务分解,下游只是在调查一个已被预先塑形的问题;(4) 降级必须显式:某一维度被 BLOCK 时要么中止该分支并记录原因,要么标注覆盖缺口,绝不能产出读起来完整、实际静默少了一个维度的报告。


## 内部治理元数据会通过序列化边界泄漏到用户面——任何 governance metadata（QC/Trust Gate/evidence trace）都必须在 presentation 层显式过滤，不能依赖 contract 文档自我约束

`2026-08-07`


根因：_wrap_response() 把 _qc 元数据作为 MCP 响应的固定前缀注入 LLM 上下文，LLM 无条件携带它进入研报 HTML，PDF renderer 再原样输出。contract 层（REPORT_FORBIDDEN）只是文档声明，runtime 链路未 enforce。How to apply：（1）任何 internal metadata（_qc/allowed_use/blocked_fields/section_data_usage）的输出工具必须区分 internal artifact 和 user-facing artifact 两个视图，不能一把梭序列化；（2）presentation 层加 allowlist（title/body/citations/charts/human-readable summary），禁止 machine-readable fields 透传；（3）contract 声明不等于 runtime enforcement——smoke test 必须验证 report output 不含 forbidden fields。这个教训适用于所有带 governance layer 的系统：内部可信机制的产物默认不该进入用户面，除非显式允许。


## 审计记录是四个独立维度(identity/evidence/decision/immutability),任一维度 PASS 不能扩大为整体审计成立 —— 报告时必须逐维度分列,不能合并成一句 audit record verified

`2026-08-06`


根因: 审计不可变性直觉上像一个布尔属性,于是拿到任一证据(比如冻结实现存在)就整体宣告成立。实际至少四维: identity(每个 decision event 有唯一 ID,证据=生产路径生成的 decision_id) / evidence(对抗方主动 mutation 尝试失败,证据=攻击执行) / decision(决策内容与理由可追溯) / immutability(记录本身无法改写,证据=递归冻结实现)。四者证明产物互不替代: 冻结实现存在只证明 immutability,不证明 mutation attack 会失败;唯一 ID 只证明 event uniqueness,不证明记录不可改写。Trust Gate Phase 1a 里如果把 C-04(ID 唯一性)合并进 F-03(不可变),就会得到一个虚高的 audit record fully immutable 结论。How to apply: 任何 audit/provenance/traceability 声明落地前,先把它拆成维度表(维度 | 证明什么 | 证明产物),每格必须有独立可执行产物;报告时逐格给结论,禁止跨格外推。这个拆解方式同样适用于 provenance(来源身份/取得时刻/传递链/未被改写)与 capability 声明。


## SOP 模板内嵌明文凭据时,执行后必须双端清理脚本;证据链纪律与凭据卫生是正交维度,前者做得再严不会自动覆盖后者

`2026-08-06`


根因:凭据卫生与证据链纪律是正交维度——证据链管的是能否证明做成了,不管过程有没有留下凭据残留,再严的证据链也不会自动覆盖清理。How to apply:执行内嵌凭据的模板脚本后固定加 rm 本地 + 远端两步并写进报告;读 SOP 时除了看它列了哪些验证项,还要问它漏掉了哪些与验证无关但同样有后果的维度。


## Evidence Provenance必须与Trust Gate分层治理。Trust Gate防止坏数据进入LLM（已证明），但数据修复引入的新fallback数据流可能绕过Evidence Governance回归，导致fallback data被误标记为primary provider。修复：P1 source_chain追踪完整降级链，P2 valuation_source区分derived/real_provider，P3 derived evidence不能触发completeness PASS→claim upgrade

`2026-08-06`


C发现的核心问题不是Trust Gate没做好，而是上游Data Recovery Fix引入的新数据流没过Evidence Governance回归。三层状态：Trust Gate PASS / Evidence Provenance BLOCKED / Research NOT PROVEN。三个Patch：P1 source_chain（fallback data=explicit fallback evidence），P2 valuation provenance（derived≠real_provider，修复stock_data.py:477 hardcoded provider=wind bug），P3 QC Claim Boundary（derived valuation→status capped at partial）。18/18 tests pass（13 TG + 5 EP）


## Trust Gate边界审计必须定义falsifiable验收标准而非设计文档——C的对抗评审发现当前三份合同(Trust Gate/Evidence/Data)只有字段名和原则，零可执行谓词、零动作绑定、零时间戳语义、零实体解析、零来源证明、零非旁路边界。Phase 3审计将5个维度定义为18条可证伪标准+12个对抗测试，每条标准都包含falsification条件。关键教训:验收标准要写什么情况下算失败，不能只写什么情况下算成功

`2026-08-06`


Phase 3 Evidence Authorization Runtime Boundary Audit v0.1完成。输入是C的对抗评审(5 GAP + 12测试)，输出是5边界×18验收标准+3跨切标准。不实现、不改QC、不改Runtime。每个标准都有falsification列——缺失这个列是v0.1合同无法验证的根因。How to apply:任何Trust Gate合同必须先定义"什么证据能证伪这个边界"，再定义通过条件。字段名存在≠边界闭合


## 分布式研究合成的结论强度会虚假升级，必须在归档前强制过独立证据审计

`2026-08-05`


规则：多个并行研究来源汇总为一个合成结论时，存在结构性 overstatement 风险——观察到模式 容易被升级为 证明了原因。这不是执行缺陷，是分布式采集→集中合成的固有偏向。必须在合成后强制插入两步：Independent Review 检查是否 overclaim，Evidence Audit 对每条 claim 标注 source grade（DIRECT/INDIRECT/SEARCH/INFERRED/BROKEN）。超出证据强度的结论必须降级（如 causal claim 降为 observation）。原因：两个 INDIRECT 来源不会互相验证成 DIRECT；跨来源共识化会产生虚假的结论可信度提升，而合成本身不会自动标注这个提升。How to apply：1) 多源合成后先过独立 review 关；2) 逐条 claim 标注 source URL 和 evidence grade；3) 无法直接访问的 source 标记 SEARCH 而非 INDIRECT；4) 归档状态为 REFERENCE ONLY + NOT PROVEN，后续 runtime 证据才能升级结论强度。


## 战略研究的结论必须经过证据审计后才能进入架构决策。Phase 1 Harness Research 产生了 7 substrate-primary / 1 procedure-primary 的强结论，但 Source Matrix 审计后发现仅 3/8 VERIFIED、5/8 HYPOTHESIS。完整链路：Research → Multi-Agent Synthesis → Strong Claim Emerged → Independent Review → Evidence Remediation → Claim Downgrade → Reference Archive。教训：研究输出不等于验证结论。Claim Evidence Registry（DIRECT/INDIRECT/SEARCH/INFERRED/BROKEN 五级）应在研究阶段完成，而非架构阶段补做。How to apply：任何战略研究窗口关闭前，必须完成 Source Matrix 审计和 Claim Downgrade，否则不得进入架构决策。

`2026-08-05`


Phase 1 Org-level Agent Harness Research 关闭。原始结论 7 H-primary → 审计后 3 VERIFIED + 5 HYPOTHESIS。建立了 Research → Claim Audit → Downgrade → Reference Archive 的 Trust Gate 链路。Harness adoption 8 criteria defined，0/8 satisfied，NOT READY。关键偏差：Synthesis 将外部参考模式的观察（observation）表述为 Finance Suite 运行时的已验证发现（verified finding），Source Matrix 修复了这个越界。


## Documentation-Only / Validation-Only 窗口的 Declaration 必须显式声明三类 claim 的可验证性边界:evidence provenance / absence claim / cross-session reference

`2026-08-05`


根因:Documentation-Only 窗口(untracked 文件 / 无 commit 要求 / 静态文档)的内在缺陷是它声称 absence(未发生某事)、provenance(某段是 appendix 而未改原内容)、cross-session reference(Related Artifact 引用跨会话内容)。这三类 claim 在 Documentation-Only 边界内结构性不可被独立验证。补救只能在 Declaration 阶段补三条 governance rule。rule 1 commit-per-evidence:每份 evidence 必须 commit 后才算 finalize,commit hash 作为 provenance anchor。rule 2 cross-session-evidence:跨 originSession 引用只能指向不能复述,被引用方必须自行归档到本仓库可审计路径。rule 3 absence-claim-audit:absence claim 默认 unverified,需独立审计日志或外部 verifier 才能升 verified。How to apply:进任何 documentation-only / validation-only / read-only 窗口前,Declaration 必须显式列出该窗口对这三类 claim 的处理边界。否则窗口结束后 Reviewer 必然 HOLD,治理成本错位 — 窗口设计阶段的设计缺陷被 Reviewer 阶段承担。


## 多 Agent 系统的证据聚合规则:两个视角不等于两个事实,必须先做 Identity Reconciliation 才能触发下一阶段窗口

`2026-08-05`


根因:多 Agent / 多会话 / 多 originSessionId 在同一项目上工作,evidence scope 经常被误合并为 Same Artifact,产生错误状态漂移 — 两个不同视角被计权为两个独立验证(放大证据权重),或两个互补切片被计权为同一个证据(丢失维度)。两种都破坏 Capability Maturity Ladder L0-L6 的可信基础。How to apply:进任何验证下一阶段窗口前,先对上一阶段产出做 Identity Reconciliation(逐项核对 originSession / observed_at / evidence scope / verdict / next action),三选一判定:Same Artifact(合并)/ Related Artifact(交叉引用,不合并不取代)/ Independent Artifact(各自推进)。R-02 等 case 命名必须等到 Pattern Confirmed 才能进 Case Registry,不能为了匹配已有 Case 而反向解释证据。任何 Agent 不做 Identity Reconciliation 就直接进下一阶段,是状态机膨胀的开端


## 独立评审的否决结论必须保持历史完整性，禁止通过回写被审文档消除

`2026-08-05`


规则：当独立评审对某治理评估产出 BLOCKED 时，被审文档（原始 Assessment）不得被修改、覆盖或弱化以消化该结论。否决必须通过新建的下游处理窗口承载，原评估与独立评审两个 artifact 都作为历史证据保留。原因：原评估回答的是 评估时刻是否满足条件，下游窗口回答的是 后续是否恢复了条件；两类问题的证据对象不同。把否决回写到原评估，等于静默改写历史结论，让后来者无法分辨 评估当时就不通过 与 后来修好了。How to apply：1) 保留原始评估文档和独立评审 artifact，禁止修改；2) 在新窗口中记录否决结论与阻塞项；3) 修复完成后通过新窗口重新验证并产出新结论；4) 任何把原始评估从 否决 改为 通过 的修改都视为违规。


## Governance rule: responsibility transfers require an explicit baseline, authority boundaries, unresolved evidence, and accountable successor.

`2026-08-04`


Root cause: implicit context makes successors infer scope and evidence maturity. How to apply: preserve project-specific facts in a durable structured record; retain this portable rule in long-term memory.


## Baseline Freeze Declaration 不能自证人类授权；必须单独附加逐字记录的人类授权原文，以及带 observed_at 的工作区快照，否则独立评审会把下一道门 BLOCK。

`2026-08-04`


根因：把 "CC 写了 human-approved" 等同于 "人类已批准" 是 claim-strength 错误（Rule C）。本次 C 的 F-01 判定 self-attested 不成立，因为 committed document 只能记录决定，不能证明授权发生。F-02/F-03 同理：排除工作区和 remote 状态必须用时间戳证据，不能用绝对声明。How to apply：以后任何 freeze declaration 必须包含 (1) 人类授权原文附录（逐字、不改写、带归因）；(2) observed_at 工作区清单，含未提交修改 diff stat 和未跟踪目录清单；(3) remote 声明必须收窄为 local ref 状态，并注明未 fetch。


## 规则: 验证分支上的任何提交都使该分支上已采集的证据失效,无论提交意图是否无害。验证语境完整性是验证证据成立的前置条件。

`2026-08-04`


根因: 验证结论的效力来自它与某个确定基线的绑定关系。一旦基线漂移,先前采集的证据就无法回答来自哪个基线这个问题,即使新提交只是文档修订也一样。做法: 每个验证阶段切换前重新核对基线指纹,不只在窗口启动时核对一次;发现漂移时先只读调查来源并记录,再把验证分支恢复到基线,漂移产物保留为独立审查记录而不纳入基线。这条规则在一次窗口内被两次真实触发验证过。


## 规则: 多主题分支不得整体合入主干。合并前必须按内容类别逐条分类,只吸收运行时与治理类内容,其余保持隔离。

`2026-08-04`


根因: 分支会无差别累积所有工作,而合并操作本身不理解内容语义。整体合并会把研究草稿、展示材料、个人扩展一并注入主干,使后来者无法区分已验证能力与探索产物。做法: 合并前先列出全部变更路径,逐条判定类别,只签出获准类别的路径,冲突项单独写调和说明而不自动合并。这样仓库结构本身就表达了能力边界,而不是只靠文档声明。


## 验证窗口的核心产出不是证明 PASS，而是声明 UNKNOWN。三个窗口（Evidence Gov v1.0→Trust Gate→Runtime Validation）形成了一个完整循环，但每一层的价值都在于“不能宣称什么”而非“能宣称什么”。How to apply: (1) 任何验证报告的“修复”是删除而非添加——每轮 Adversarial Review 的修正方向是移除未经独立来源支持的主张，不是补写看起来对的内容；(2) Engram 拒绝 World State / Runtime State / Phase Completion 不是 bug，是特征——只有设计原则、层级边界、冻结规则能进长期记忆，系统状态走 MEMORY.md；(3) Reviewer 优先级：Evidence Contract 完整性 > 事实归因 > 文本自我矛盾——这个顺序不因项目类型改变。

`2026-08-04`


三窗口闭环验证: CC 写报告→C Blocked(5 F)→CC 修正→C Blocked(2 F)→CC 再修正→PASS。核心发现: 验证报告的修复方向是删除不支持声明，不是补充内容。Engram 拒收 phase completion 是已确认过滤规则。


## Governance 规范自身也需要可信边界校验——外部标准归因缺失是比功能遗漏更危险的风险。Phase 2 的 F-01/F-02 暴露了同一个模式: External Standard exists → Vera references it → Reader assumes Vera implements it。修复不是加功能,而是给每处外部标准引用加 [Origin: External Standard] + [Vera Status: Not Implemented] 显式标签。How to apply: (1) 任何规范文档引用外部标准/方法/架构时必须同时标注 Origin 和 Vera Runtime Status,缺一不可;(2) Capability Claim Boundary Audit 的核心不是查“有没有实现”,而是查“会不会被误读为已实现”——被误读比未实现更危险;(3) Governance 自身的 Freeze Gate 必须进 Adversarial Review,且 Reviewer 不能是 Builder。

`2026-08-03`


Phase 2 EVIDENCE_GOVERNANCE_v1.0 吸收增强完成→C Adversarial Review(PASS WITH CONDITIONS)→F-01/F-02 修复→PASS→FROZEN。核心成果:8条公理+双时态合约(valid_time/knowledge_time MUST)+Evidence→Decision Authorization Model(Trust PASS≠Authorization ALLOW)+Claim Strength Contract(六级声明强度+Escalation Gate)。外部标准(in-toto/PROV/SCITT/PIT/OPA/Cedar)全部标注 Origin:External+Vera Status:Not Implemented。冻结后禁止继续扩展 Governance,下一窗口=Runtime Governance Validation v0.1:用 Governance 审 Runtime,不修改 Governance。


## 治理案例只能由生产事故推动，不能由规则设计反向寻找案例来凑数

`2026-08-03`


在建立Learning Loop时容易犯一个结构性错误：先定义治理域缺口(还差第三个Case)，再寻找一个看起来像事故的案例来填位。这会把Incident→Rule的因果方向反转为Rule→Incident。正确做法：Case Registry里留一个PENDING INCIDENT空位，并明确声明禁止人造案例。这个空位本身就是治理约束——它证明了系统不会为了凑齐模型而伪造证据。how to apply：建立Governance Case Registry时，为每一个SPECIFIED case保留原始incident溯源；为每一个PENDING slot标注awaiting real incident而非TBD feature。


## Evidence Governance体系必须由真实生产事故推导，不能由设计先行。Learning Loop因果方向不可逆：Incident→RCA→Defect→Rule→Test，禁止Rule→Incident的反向设计。三个治理域(Freshness/Claim Strength/Runtime Evidence)覆盖了输入可信、结论授权、执行可证明三角。Freeze Gate保持2/3是正确的——第三个域必须等真实incident，人工补案例会破坏整个体系的可信基础。How to apply: 每个governance case必须能回答(1)哪个真实incident推导(2)defect是什么(3)acceptance test是什么(4)谁实现谁批准。Tier A(incident-derived)与Tier B(proactive audit)必须分离，后者不能block Freeze Gate。

`2026-08-03`


Evidence Governance v1.0 Architecture Freeze Draft APPROVED。4个Case: F-03 Freshness(SPECIFIED,600439), C-01 Claim Strength(SPECIFIED,stock-analyst.md), R-01 Runtime Evidence(SPECIFIED,Auction RCA,separate track), P-01 Provider Capability(TRACKED Tier B,IMA/Wind Audit)。Freeze Gate 2/3,第三个domain PENDING INCIDENT。核心约束: Incident→Rule not Rule→Incident, Builder≠Reviewer, One Incident→One Rule→One Test。


## Data Governance架构设计必须从Data→Answer升级为Data→Evidence Governance→Trusted Bundle→Agent。三个不可跳过的设计决策：(1)三日期分离(data_period_end/announced_at/fetched_at)替代单一as_of——因为fetch time≠data period，消费者会误读；(2)关键维度freshness gating替代stale计数——因为一个关键维度stale+其他全fresh时，len(stale)<=1会错误放行；(3)五域分治——Financials/Announcements/News/Valuation/Macro各有不同freshness政策和时间语义，不能共用一套QC。How to apply:设计任何数据治理层时，先问三个问题：这条数据的时间语义是什么（period/announce/fetch）？如果只有这一个维度stale，Gate会放行吗？不同数据域的时间特性是否被同一套规则掩盖了？

`2026-08-03`


600439事件暴露了单一as_of字段+stale计数模型的双重缺陷。Evidence Governance v1.0设计文档将教训转化为normative spec：Evidence Object统一schema、五域分治、四状态Trust Gate(PASS/PARTIAL/UNKNOWN/BLOCKED)、source_confidence随fallback降级。
