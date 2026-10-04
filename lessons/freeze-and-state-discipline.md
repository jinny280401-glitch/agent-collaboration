# 冻结与状态纪律

> closed session immutable;SHA pin 证明 identity 非 durability;revision pin 可记录≠可解析

共 33 条。源自 Engram 跨 agent 记忆库,已脱敏。


## 验证一个 artifact 的 SHA 只回答 identity(是不是原来那个),不回答 durability(原来那个还在不在)。宣布 byte-exact unrecoverable 之前,必须先扫所有与 git DAG 正交的 harness 恢复通道,否则会用错误的 unrecoverable 前提污染上游裁决。

`domain: claude-code-infra` · `2026-10-02`


根因:容易把 identity verification 误当成 durability guarantee——SHA pin 过了就以为内容安全,其实它只证明手上这份是真的,不保证别处还留着。Identity/Review/Durability/Recovery 是四个正交属性:review 结论不随文件恢复而失效,文件恢复也不改变 review 结论。How to apply:(1)长期 untracked 的 canonical 文件=durability gap,落地即应进 version control,不要依赖偶然留痕;(2)harness 侧至少有两条与 git 正交的恢复通道——~/.claude/file-history/<session>/<file-id>@vN 的逐版本快照,以及 session transcript 里 Write tool_use.input.content 的原始 payload——两者都能按 sha256 精确检索,宣布不可恢复前必须都扫过;(3)覆盖类事故先把现场产物另存为 NON-CANONICAL 文件留证,再做 restore,恢复出的副本走新路径不盖现场。


## Treat closed agent sessions as immutable historical artifacts; never retro-edit them when new state arrives

`domain: claude-code-infra` · `2026-10-01`


GENERALIZABLE INVARIANT for multi-agent governance. When an agent session ends (terminal state declared), the artifacts it produced become a fixed historical record. New state from other sessions belongs in new artifacts, not in edits to the closed ones. Coupled rule: closing one task never auto-authorizes a downstream dependent task -- each step needs its own authorization. ROOT_CAUSE: retro-editing closed artifacts to mirror newer state destroys chain-of-custody and causes drift between the file content and the claim the file makes about the system, which breaks audit and adversarial review. HOW TO APPLY: (1) at session close, freeze artifacts at their closing state and pin their hash; (2) when a later session needs to convey new state, write new artifacts with new hashes; (3) when reasoning about a closed session, treat its files as immutable historical input; (4) absence claims from a single observation must say ABSENT (this observation), never NO, to keep the scope qualifier bound to the evidence.


## 完整交易计划≠买入授权;事件博弈skill与价值授权skill必须用互斥终态词,BUY AUTHORIZED专属buy-authorization,防"计划完成"被误读为"资本已授权"

`domain: trading-system` · `2026-09-21`


根因:新华传媒600825案(四关到三关、计划优秀仍WATCH,因THESIS是重估博弈非价值来源)证明,任一分析skill做完顺手宣布可买=权限污染。How to apply:非buy-authorization的skill禁输出BUY AUTHORIZED(event-trade用NO PLAN/PLAN INCOMPLETE/SPECULATION PLAN READY/PLAN INVALIDATED);当前仅命名公约非运行时强制(wiring≠enforcement),真强制须加终态词linter。配套不变量TIME STOP IS IMMUTABLE AFTER ENTRY:建仓即冻结T+N,延期只能退出重建新Thesis,与REPLACED THESIS彻底分离。已固化为DEFENSE SCALE V0.2 FROZEN(L1-L6),PORTFOLIO FIT不纳入,演进唯一入口=真实案例打穿V0.2才进V0.3,禁理论补全。


## 发现实现偏离frozen baseline时,先验不变量再决定re-baseline还是rollback,别机械回滚版本号

`domain: hermes` · `2026-09-15`


治理忠诚对象=核心不变量(安全/身份唯一/数据绑定/访问边界),不是版本号。步骤:①先查新状态是否破坏安全假设;②只重验不变量,不重审是否照图施工;③全过则re-baseline把意外漂移转成正典基线,旧frozen降HISTORICAL;④仅当不变量真被破坏才rollback。根因:rollback目标是恢复不变量而非版本字符串;新版本若反而修复旧版缺陷,机械rollback=退回已知更脆弱状态


## 风险接受决策的终态是 OPEN/RISK_ACCEPTED，不是 CLOSED——接受风险不等于消除风险

`domain: claude-code-infra` · `2026-09-08`


根因：把『决策者选择暂不处置』和『问题已不存在』两个不同状态混为一谈。治理/安全 finding 若以 Owner 明确接受已知风险并推迟补救结尾，记录系统里常用的 CLOSED/REMEDIATED 标签会在语义上宣判风险消失，误导后续 agent 把未处置项当成已收口。how to apply：风险接受类收口必须写 RISK_ACCEPTANCE=OWNER_ACCEPTED + EXPOSURE_STATUS=CONFIRMED + REMEDIATION=DEFERRED，并显式禁写 CLOSED/REMEDIATED/NEGLIGIBLE。接受是对『处置决策』的记录，不是对『风险状态』的改变；两者分开，风险状态在处置完成前始终 OPEN。


## 凭证脱敏的安全边界不是自建日志,而是所有落盘的共享日志——SDK/框架自身的 stdout/stderr 不受你的 redaction 控制,会明文写连接凭据,排查和分流必须覆盖 receive-path

`domain: claude-code-infra` · `2026-09-07`


根因:observability 只对自建 receipt 做 redaction,但 SDK/框架(如 Lark WS client)自己的连接日志经共享 stdout/stderr 落盘时,把连接凭据(access_key/ticket)明文写进去,自建 redaction 覆盖不到。how to apply:1) 安全排查范围=所有共享日志,不止自建产出;2) SDK debug log 与 observability receipt 分离,或对框架连接日志脱敏;3) 任何曾写入共享日志的临时连接凭据都要评估轮换。


## 持久化语义内容哈希标记必须先脱敏 token 再 hash,否则输出含 secret 时该哈希仍是 credential-derived material

`domain: claude-code-infra` · `2026-09-07`


R2 review 判 content_marker 直接 sha256(原始输出)可能成为 credential-derived hash、endpoint.declared 可能原样落 userinfo。根因:observability 只记 presence/hash 的纪律不能因是哈希就豁免——hash of raw secret 仍是 credential-derived material(可作 verifier)。修法:sanitize_url 剥 userinfo、content_marker 改为 sha256(redact_text(text))。how to apply:任何对可能含 secret 的文本做派生哈希前,先跑同一套 redaction 再哈希


## version label 单独不足以构成 artifact identity，pin/review target/canonical 声明必须由 immutable hash 承载

`domain: finance-suite` · `2026-08-31`


根因=同一个 label 可被多个 writer 各自赋予不同内容，label 与 bytes 之间无绑定。实证:一份治理设计文档的 "v0.2" 同时指向两个非等价 artifact(一个被裁为 canonical，一个为外来 competing 版本)，而索引文件只登记了 NON-CANONICAL 的那一个 → 按 label 引用会取到错误 artifact 且不报错。How to apply: 凡 pin、review target 指认、canonical 声明、跨文档引用，一律写 hash；label 只能作附注。目录存在并发 writer 时，先冻结 current bytes 为 hash 命名只读副本再 review；核验通过的时点≠消费时点，两者之间存在写入窗口。


## 对 pinned artifact 的 hash 核验只证明核验瞬间，不证明消费瞬间；核验后必须立即取不可变工作副本，并在交付前对原路径重新核验一次以检测窗口内漂移。

`domain: claude-code-infra` · `2026-08-27`


根因：把一次前置 hash 核验当成整段作业期间的持续保证，等价于隐含假设「作业期间无其他 writer 触达该路径」——该假设从未被证明。若 artifact 位于版本控制之外（untracked），覆盖发生后既无 DAG 也无 author 可追溯，producer 只能记 UNKNOWN，无法事后归属。How to apply：(1) 核验通过后立刻 cp 出工作副本，所有修改只作用于副本，绝不在被 pin 的原路径上就地编辑；(2) 交付前对原路径重算 hash，与核验时刻比对，漂移即报 incident 而非静默继续；(3) 若原路径已被覆盖，可用「逆向重演自己的每一次编辑并校验每处匹配唯一」的方式从副本密码学重建原 pin，重建成功同时证明自己的产出确实派生自真 pin；(4) 恢复内容必须落新路径，不得覆盖现场，保留覆盖者的文件供裁决；(5) 出现竞争修订时不得自行选定 canonical，交由 reviewer 裁决。


## 不可撤回外发链路的intent-send-receipt两态记账在崩溃/超时窗口会重复发布,必须durable store+UNCERTAIN三态+禁自动重发

`domain: finance-suite` · `2026-08-27`


根因:send已执行但receipt未落盘之间存在窗口,崩溃或超时后状态停在已发但无记录,账上与未发不可区分,重试即重复发布。C review判major。how to apply:①idempotency store须durable非内存②先写intent再发③三态PENDING->SENT/UNCERTAIN/FAILED,同键UNCERTAIN禁止自动重发转人工④恢复须按业务幂等键查下游实际是否已发⑤去重键用业务键如{report_type,as_of_date}非自增id⑥副作用不同的链(消息发送vs文件发布)不共用一句发送成功当验收


## 规则: 档案以 revision hash 作状态锚点时,必须把"历史曾记录的 reference"与"当前可解析的 revision"分开表述;锚点不可解析不得连带推翻独立取得的证据

`domain: finance-suite` · `2026-08-26`


为什么:某冻结档案写的锚点 hash 事后在仓库中不存在(not a valid object name),原表述让读者误以为该 revision 可复现。怎么做:1) 落档时同时记录锚点的 resolvability;2) 发现不可解析时改写为 reference recorded as X / currently UNRESOLVABLE;3) 独立取得的证据(文件 blob 比对、命令 receipt)不因锚点失效而降级;4) 不重建 replacement commit 去修复历史。


## 「我只读」不等于「树是静止的」。只读保证的是我没写，不保证别人没写。在共享工作目录上做 inventory/classification，必须在快照前后各取一次 fingerprint 证明树未变，否则分类内容可能正确但时效性不成立，且与复核者看到的是不同状态。

`domain: claude-code-infra` · `2026-08-23`


根因：把自身的写权限约束误当成对象的不变性保证。只读窗口只约束了一个 actor，共享目录上的其他 session/脚本仍可自由 mutate；单点快照无法区分「对象本来如此」与「对象正在变化中我恰好看到一帧」。实测：一次全程只读的 repository classification 窗口，inventory 在 16:16 执行；事后发现另一并发 actor 在 16:12→17:37 之间把一个 228M/1650 文件的子树移出该 repo。结果是 inventory 未列出该子树——不是漏扫，而是采样时序问题——但当时完全没有检测到并发写入，直接据此宣布 IDENTITY RECOVERED。复核者随后在另一个树状态上复核，得出「计数不符/该子树消失无法解释」，双方都没错却不可比。How to apply：(1) inventory 前后各取一次 fingerprint(git status 摘要 + untracked 文件 hash 集合)，两次一致才可声明 identity 类结论，不一致就只能声明 classification；(2) 区分两种 verdict——CLASSIFICATION COMPLETE(内容已分类)与 IDENTITY FROZEN(状态已证明未变)，后者需要额外证据，不可由前者推出；(3) 交出快照时附采集时刻，复核者先比 fingerprint 再比内容，避免把时序差异误判为计数错误；(4) 发现并发 actor 后，处置顺序是先确认写入者集合、再决定是否需要 writer fencing，然后才谈 mutation——在树未冻结前授权 mutation，等于在可能过期的分类上操作。


## 冻结一个 allowlist/变更集时，必须冻结「文件内容指纹」而不是「working tree 的 dirty 状态」。dirty 状态可以被后续 session 无痕搬移——文件被移出 tracked 路径后 git status 变干净，冻结记录看起来仍然成立，实际已与磁盘脱节。

`domain: claude-code-infra` · `2026-08-23`


根因：git status 的 dirty/clean 是「路径 × 内容」的函数，不是内容存在性的函数。把变更移到 untracked 目录并改后缀，tracked 路径立刻回到 clean，但内容未丢失也未提交——冻结文档里的「N files modified」于是同时既非真也非假，无法据此判断变更还在不在。实测案例：某 allowlist 冻结为 8 files(frontend 6 + backend 2)，两天后实测 tracked dirty 只剩 1 个；排查发现变更被搬到一个未被任何索引记录的 evidence 目录并加 .deferred 后缀，另有 backend 侧早已 clean 且与 origin 同步。若当时按冻结文档直接 commit，会提交一个与描述完全不同的变更集。How to apply：(1) 冻结变更集时记录内容指纹(git hash-object / diff hash)与完整路径，而不是只记 modified 计数；(2) 恢复 identity 时不要相信任何历史描述，先无条件重测磁盘现实，再解释差异来源；(3) 差异定位要覆盖 relocate 这一类无痕操作——检查 untracked 目录、后缀变体、worktree、stash，而不是只看 reflog 是否有 reset/checkout；(4) 发现描述与磁盘不符时，结论应是「描述已失效」而非「变更已丢失」，两者的处置完全相反；(5) 任何搬移变更集的操作必须留索引记录，否则 owner 永久 UNKNOWN。


## 仓库盘点窗口与 Cleanup 执行窗口必须物理分离：盘点 commit 落地后立即冻结，任何删除/移动/合并须在独立授权窗口中以 frozen candidate matrix 为输入基线重新开始

`domain: finance-suite` · `2026-08-19`


根因：若允许 inventory 窗口顺手执行 cleanup，"已盘点"会被误读为"已授权删除"，导致越界操作难以追溯。CANONICAL_UNKNOWN 项尤其危险：在 blood-line proof 完成前，任何启发式判断（文件较新/较大/生产在用/名字更像）都不能作为删除依据。How to apply：(1) inventory commit 落地即视为只读基线，不再回写；(2) cleanup 独立开窗，起点为 frozen candidate matrix → canonical identity proof → cleanup decision matrix → explicit mutation authorization；(3) git worktree prune 判定 prunable ≠ 删除授权，须单独授权。


## 规则: 多线程共享同一工作树时,任何未提交编辑都会击碎 source freeze;漂移必须先由原作者保存补丁并恢复基线,再由冻结方全量重采样,冻结窗口不得代改。

`domain: finance-suite` · `2026-08-15`


根因:另一线程在同一工作树写入未提交改动,冻结采样时工作树哈希与提交树不一致导致冻结失败。How to apply:发现漂移后先定位写入线程,由其导出 patch 并保存到独立分支、恢复基线文件;冻结采样作废全部旧 PASS 子项,从第一步重新全量采样;冻结/收口窗口本身不修改、不 revert、不 commit、不 deploy。


## 规则：跨仓库 review 提交必须携带 {repo_path, remote, full_sha, role} 四元组——两个独立 git repo（不同 object store、无共享 remote）之间 commit SHA 不可解析，builder repo 的 SHA 拿去权威工作区审必然 BLOCK；复审必须按完整 SHA 而非 branch tip。

`domain: finance-suite` · `2026-08-15`


Auction Price Evidence Repair Source Identity 事件：reviewer 在 ~/finance-suite 解析 91a9f8c 失败返回 INVALID REVIEW TARGET，根因是两个独立 repo（finance-suite-backend=backend canonical vs finance-suite=治理文档仓）对象库互不可见，非 commit 本身有问题。修复=证明 repo 角色+完整 SHA，复审改按完整 SHA 91a9f8c89bb0bc6474b1e4693e738bd769019fac。


## 独立代码复审必须钉死权威绝对路径和不可变 revision；在错误 workspace 上枚举文件得到的"文件不存在"是 observed absence 而非 universal absence，会产出假 REQUEST_CHANGES 压低有效 verdict

`domain: claude-code-infra` · `2026-08-14`


复审误从 sibling 项目 ~/Documents/New project 6/ 枚举文件（该目录也有 research_runtime/ 但内容不同），而 I1 权威工作区是 ~/finance-suite/（immutable revision eeac003）。根因=复审未先 pin 路径和 revision。how to apply=复审前先 git rev-parse HEAD 比对目标 revision；用绝对路径；显式标注 ignore 非权威 workspace。


## 单点反例在 mutable worktree 看似闭合 ≠ 结构性契约成立；冻结 verdict 前必须拿到 immutable revision(commit+sha256)

`domain: finance-suite` · `2026-08-13`


Defect B 独立审阅:auction_in_progress canary 已在工作区修复(未提交),但四个结构 gap(no filtered-payload / allowed_use 未 enforce / binary whole-bundle admission / market_pulse 旁路 mcp_server.py:329)仍证明 Gate-before-Prompt 契约未被证明。根因:审阅锚定工作区路径而非 hash,文件并发被改 4+ 次,复现的 bug 与 bug 已修复同时为真。How to apply:冻结 verdict 前 git rev-parse + shasum pin 住版本;目标在漂移时改报证据不稳定并等作者给 commit+sha256;拿到后同一 probe 同一口径重跑,不换测试方法。


## 审阅未提交且被并发活跃编辑的文件前必须先 pin commit/hash,否则证据随读取漂移,任何 frozen PROVEN/NOT_PROVEN 都是对移动目标的假证

`domain: finance-suite` · `2026-08-13`


Defect B 独立审阅:目标 auction_data.py 在读→写结论间被另一会话改写≥4次,基于旧版复现的 auction_in_progress 时间闸门漏洞在终版已修复,报告的一条活缺陷(577行allowed_use旧二值覆盖)在终版也已不存在。根因:审阅锚定工作区路径而非hash。how to apply:审阅前 git rev-parse HEAD + shasum 锚定版本;发现未提交+持续刷新立即停止冻结结论,改为要求作者停编并提供hash后重跑


## 规则: 语义表存在未定义格时不得宣布契约可验证;机器验证只能覆盖已定义的部分,带未定义格宣布可验证,会把未测范围伪装成已测范围

`domain: finance-suite` · `2026-08-13`


验证用例全部通过只证明已定义语义自洽;任何未定义格都天然在验证覆盖之外。固定路径:先单格裁决未定义语义,再补判别用例,再全量回归,最后才宣布契约可验证。检查已存在但命名粒度不完美的项,登记为分类债而非阻塞项。


## 发现现场异常不等于获得修复授权：证据采集窗口与 Repair Window 必须分离，任何改动都需单独授权

`domain: finance-suite` · `2026-08-13`


根因:同一时刻既观察又修复,会让现场失去作为证据的价值——改动后的状态无法回答改动前发生了什么;而且发现异常带来的行动冲动会自然膨胀成未经裁决的变更。How to apply: 窗口期(复探/验收/事件窗口)内,一切异常只形成 evidence,不形成修复动作;涉及 production/proxy/route/env/service/代码 的修改,一律退出当前窗口、另开单独授权的 Repair Window;窗口纪律以时间点(ex. 09:05 freeze)和清单(只读检查项)两种形式预先锁定


## 不要因为前置条件已讨论清楚就认为可以执行——执行前必须有冻结的单一契约

`domain: finance-suite` · `2026-08-13`


根因:把已达成共识当成已具备执行依据,但共识分散在评审往来记录中,不构成对执行方的约束,执行方可能照旧版本文档行动。How to apply:共识收敛后把结论原样转录为单一契约文档交权威方冻结;冻结前坚持不可执行;复核阶段只查转录忠实度,不重开讨论。


## 真实事件窗口前的 readiness check 必须只读：任何修复/重启/重部署都会污染待观察的现场

`domain: finance-suite` · `2026-08-13`


2026-08-13 09:15-09:25 竞价窗口 Re-Probe 前 G 锁定:08:00-09:00 readiness only(SSH/service/页面auth/receipt sink/rollback artifact),异常只记录不修复不重启不重部署;09:05 change freeze 保护真实窗口。根因:窗口期现场本身就是证据,对它的任何改动都会让失败时留证据失效。How to apply: 任何复探/验收窗口前,先定 readiness-only 边界+freeze 时刻+只读检查项清单;窗口结束前禁止以顺手修一下名义动现场


## 规则: schema 冻结必须与开放参数关闭分离;fixture 物化前先冻结全部开放参数,否则暂定值会被固化成事实标准

`domain: finance-suite` · `2026-08-13`


治理评审中,schema 设计冻结时保留开放参数(阈值数值/受控词表/id 公式),裁决明确要求先关参数再物化 fixture。Why: fixture 会把暂定参数固化成事实标准,后续参数升级时全部 fixture 作废。How to apply: 冻结文档时显式列出 open parameter registry;fixture/测试物化前先完成参数裁决;参数关闭中若发现会改变已冻结语义,触发变更请求而非静默吸收。


## 规则: spec 文档首节固定放冲突登记与归一化清单,任何未定义参数标待裁决,不发明数值

`domain: finance-suite` · `2026-08-13`


跨文档以文本冻结的公式会互相漂移;写 schema spec 前先做 reconciliation,冲突进 ACR register 请求裁决,歧义进归一化清单,绝不静默选一边修正。root cause: 评审以自然语言冻结,章节间公式矛盾不可避免;实现者遇到矛盾会本能选一个顺手的解释。how to apply: 评审先审冲突清单与归一化清单两节。


## 系统代理/TUN 模式下，-o ProxyCommand=none 不能绕过 Clash；SSH 目标 IP 必须配 DIRECT，否则 kex_exchange_identification 阶段会被 reset，易被误判为服务器或防火墙故障

`domain: finance-suite` · `2026-08-10`


排查 <server-ip> SSH 不通：ping 丢包但 nc 22 通，ssh -vvv 在 KEX 阶段被 Connection closed by remote host。服务端 fail2ban/sshd/安全组均正常。根因是 Clash 系统代理拦截了 SSH 流量。修复：IP-CIDR,<server-ip>/32,DIRECT。验证：代理开启状态下 SSH handshake PASS / exit 0


## Parallel Claude Code sessions sharing the same git working directory and memory can silently converge on the same frozen window; always verify window ownership via peer session inquiry before any evidence collection action.

`domain: claude-code-infra` · `2026-08-10`


Root cause: commit authorship and working tree state cannot distinguish which session owns a frozen window. Two sessions (this one and zhuanz-70) both saw the same branch/state and could have proceeded independently, risking baseline drift and duplicate/mutating evidence. Correct action for a non-owner is to stand down immediately, make no collection commits/probes, and route observations to the owner. How to apply: before resuming any frozen multi-session window, send ownership inquiries to peer sessions; if ownership is claimed by another session, treat the window as read-only and forward only non-actionable coverage-gap notes.


## 规则: 设计窗口与冻结窗口对撞时,不要改冻结规范也不要跳过冻结,而是让设计产物显式声明自己在规范链之外、非约束性、能力效应为零。

`domain: finance-suite` · `2026-08-07`


原因: 冻结保护的是规范链的权威性,不是禁止思考。两种常见误处置都有害:一是把设计写进冻结规范或修改它,直接破坏冻结;二是因为有冻结就不做设计,导致下一个窗口在无设计的情况下直接进实现。第三条路是产物自我限定——设计文档在文件头声明:不属于规范链 / 不修改任何冻结资产 / 非约束 / 不计入当前窗口产出 / 成熟度等级为零 / 与冻结资产冲突时一律以冻结资产为准。这样规范链的唯一权威性不受损,设计又不必推迟。

适用: (1) 发现窗口冲突先明说冲突存在,不要静默选一边;(2) 设计产物给出『许可引用措辞』与『禁止引用措辞』两段,防止后续被人当作能力证据引用;(3) 若设计将来要升级为规范,该升级是一次独立的授权行为,需要自己的 gate,不能因为文档已存在就默认生效;(4) 设计文档要接受它自己引用的原则的审查——若文中主张的控制无任何 runtime 实现,按『声明能力≠有效能力』它同样不构成治理能力,这一条要写进文档自己的 open questions。


## 18/18 fixture PASS不能证明runtime boundary闭合——独立对抗审查(C)发现了return-path coverage未闭合和deep immutability未成立两个架构级缺陷，两者都优先级高于fixture结果

`domain: finance-suite` · `2026-08-06`


Phase 1a Closure从AWAITING翻转为FROZEN NOT PROVEN。F-01:缺参/异常return路径未进入Gate。F-03:MappingProxy只冻结外层，嵌套evidence可改写。根因:fixture测试只能证明known paths pass，不能替代runtime boundary proof。How to apply:任何声称boundary closed的结论必须先过独立对抗审查，且审查必须覆盖uncovered paths和deep mutability，不能仅验证happy path fixture。


## 治理验证窗口里,static-reachable 与 live-only 必须二分;否定性证据(某路径不可达 / deploy 脚本不覆盖后端)本身是治理价值,不是验证失败

`domain: claude-code-infra` · `2026-08-05`


根因:Capability Maturity Ladder L0-L6 要求每条能力声明对应到具体 evidence maturity;Validation Only 窗口里,UNKNOWNN 不应该被强行转 RESOLVED,但也不能被忽视。否定性证据(import 找不到模块 / deploy.sh 不覆盖后端)是支撑 Fix exists != Root cause proven 治理区分的关键事实。How to apply:进任何治理验证窗口前,先把 UNKNOWN 列表按 static-reachable / live-only 二分;只对 static-reachable 项分配验证任务;live-only 项如果数量过多,考虑是否需要 Deployment Authorization Window 才能闭合。不要把运行时具体数字(RESOLVED 几条、carried 几条)写进 Engram,那些属于 World State,应进 Memory


## Engram 拒绝存储治理窗口的运行时状态，约定层教训应走 Memory

`domain: finance-suite` · `2026-08-05`


教训：Engram 对 world state / runtime state 有强守卫——窗口 baseline、当前 status、pending、next_owner 这类运行时信息会被拒收。根因：Engram 是跨 agent 跨项目的教训库，不是项目状态库。How to apply：把窗口 handoff 模板这类约定写入项目 Memory，把为什么要这样组织的抽象教训写入 Engram，两者分工不要混。


## 当治理窗口关闭后本地 HEAD 发生漂移时，不要静默选择最新 commit，也不要 reset 回关闭 commit；必须停下来创建一份人类批准的 Baseline Freeze Declaration，显式命名 freeze SHA、列明 inclusions/exclusions，并声明在下一评估完成前后续 commit 不属于输入基线。

`domain: finance-suite` · `2026-08-04`


根因：隐式基线选择把 "commit 存在" 等同于 "已授权输入"，把人类决策藏在 git 拓扑里。本次 987fe8b→5f876cf→12649ca 的漂移正是因为没有显式冻结点，导致 C 任务卡本身又制造了新的漂移。How to apply：以后任何 post-close HEAD drift 都按 baseline-ambiguity incident 处理；下一治理窗口 OPEN 前必须有一份人类批准的 freeze declaration，声明 freeze SHA、工作树未提交修改、未跟踪目录、以及未来 commit 的边界。


## MEMORY.md 治理规则: 161 行接近 200 行读取上限,不在 Phase 2 闭窗后立即压缩——当前冻结窗口的决策链完整性优先于行数优化。下次专门开 Memory Maintenance Window 做分层(MEMORY.md 只保留当前状态/冻结原则/长期规则,细节下沉到 topic files)。How to apply: (1) 不要在功能窗口关闭后立即做记忆压缩——这会断裂 D26→D27→Phase 1→Phase 2 的决策链;(2) 压缩窗口应与功能窗口隔离,目标不是删信息而是分层;(3) 当前边界:不因行数提醒而匆忙操作。

`domain: claude-code-infra` · `2026-08-03`


MEMORY.md 当前 161 行,接近 200 行读取上限。Phase 2 EVIDENCE_GOVERNANCE_v1.0 FROZEN 刚完成,CODE_Thirdparty_Runtime CLOSED,Multi-Agent Trust Gate v1 CLOSED。决策:不在此窗口压缩——当前冻结状态下的历史决策链完整性优先于行数优化。下次独立开 Memory Maintenance Window 做分层迁移。
