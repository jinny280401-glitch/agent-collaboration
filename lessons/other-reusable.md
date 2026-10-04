# 其他可复用教训

> 不属于以上主题、但仍是跨项目可复用的工程/工作方法教训

共 14 条。源自 Engram 跨 agent 记忆库,已脱敏。


## On PEP-668 Python, prefer uv pip --system --break-system-packages over pip install --user.

`domain: claude-code-infra` · `2026-10-01`


pip install --user is refused on externally-managed Python even with the flag. uv pip install --system --break-system-packages works around it and resolves pre-built wheels for newer Python when available. Root cause: PEP 668 protects system Python from package conflicts. How to apply: when pip --user is refused on a PEP-668 Python, switch to uv pip with --system --break-system-packages instead of escalating to venv.


## 低等级证据不得自行授权高等级决策。Signal 是候选资格不是交易授权；BUY AUTHORIZATION = THESIS + INVALIDATION + EXECUTION + RISK，Signal 不在公式里。

`domain: trading-system` · `2026-09-21`


根因：同一强势信号后 −17% 与 +79% 并存，只能否定 SIGNAL→自动BUY，不能外延成 Signal 无预测价值。how to apply：下单前四关全 PASS 才 BUY，任一 STOP 则 WATCH；涨停≠Thesis、竞价强≠Execution、我看好≠Invalidation、跌很多了≠Risk。调度：skill buy-authorization 与 /buy-authorization。


## 把分公司培训做成可复用母版时,产出待执行checklist(按责任方打标)而不是全文纪要或咨询报告;城市专属只隔离两块(产业匹配+竞赛排名),PPT母版不改名。

`domain: hermes` · `2026-09-09`


根因:142页PPT是位图、python-pptx抽不到字,必须用PDF文字层读结构;Finance Suite meeting skill是麦肯锡五段式,跟PPT章节走的学习材料结构冲突,硬套会产出错体裁。How to apply:下一场分公司复制宁波md改城市名,只改定增招商产业匹配和冲刺奖赛况;读PPT先查PDF有无文字层再决定是否看图;用户要上台再压提词卡,未开口不做。路径:Documents/闽商会客室 三件套。


## 高频重复的登记型流程必须固化为slash command，把纪律写成命令约束而非靠会话记忆传承

`domain: claude-code-infra` · `2026-09-02`


盘中观察卡连续4次手工写卡（福建金森/金牛/神奇/键凯）后，模板与七条铁律（只登记不裁决、user-provided≠已核验、禁事后改写、不发明阈值、资金流禁解释出货、不碰冻结索引、收盘帧NONE/UNRESOLVED）只存在于会话上下文，跨会话必然漂移，用户要求建/chicang命令固化。How to apply：同一流程被重复执行≥2次且纪律约束复杂时，主动提议固化为命令文件，把铁律写进命令本体，执行约束与内容分离。


## 用文件名判断是否已装/已存在会失效：同名文件碰撞会被系统改名成 X 2/3/4，文件名不再是稳定身份，判断安装状态必须读身份字段并核对规范目录

`domain: claude-code-infra` · `2026-09-01`


根因：多个skill包解压到同一目录，macOS碰撞改名使文件名与SKILL.md的name脱钩。how to apply：先grep身份字段（如SKILL.md的name），再对照规范目录（~/.claude/skills/<name>）判断是否已装；不要用文件名当身份。


## 同一批数据存在两套ID/序号键空间时，按错误键查表不会报错而是静默错挂——齐备性校验(覆盖率、行数、KeyError)对归属正确性完全不敏感，必须加反例断言证明修复生效

`domain: claude-code-infra` · `2026-08-29`


根因：排序口径变更后(mtime序→日期序)，记录用新序号而上游分批产物仍用旧序号，两边都是零填充三位数这种合法字面形态，查得到、不抛异常，但144/174条会挂到错误对象上。做法：旧键产物降级为只读历史+仅作齐备性核查，另生成re-key产物作为唯一取数口径，且re-key产物自带冗余校验字段(原序号+名称核对)，校验走「键→名称→取值」三段闭合而非「键存在」。关键：必须明确算出「若用旧键会得到什么」并确认与正确值不同(本例旧001=6.1/音乐科普 vs 新001=7.8/文化评论)，否则无法证明修复生效。偏离spec处：无，此为原缺陷修复。


## 规则：验证守卫放在解析层不构成不变量；归一化函数若会被直接调用，必须在自身入口独立检查身份约束，不得依赖调用方曾经过的解析路径。

`domain: finance-suite` · `2026-08-17`


根因：把身份检查只放在解析函数，但归一化函数用 .get(key, "unknown") 默认值读取身份字段，绕过解析层直接构造 parsed dict 的调用方可产出出处字段为占位符的收据。修法：在归一化函数顶部定义模块级预期常量，在读取任何数据字段之前检查，不符则抛出解析错误。How to apply：任何会发出出处或身份字段的归一化函数，都应把这些字段的约束视为该函数自身的后验条件，而非继承自调用路径的前提条件。


## 谈判对方引用新施行法规施压时，不能默认条文真实=适用于我方，必须三查：真实性、引文忠实度、适用射程

`domain: minshang` · `2026-08-15`


根因：物业2026-08-14引用生态环境法典第250条(8-15施行)拒绝22F厨房，条文与罚则引文均真实忠实，但禁令射程是居民住宅楼/商住综合楼，纯商务办公楼不命中——对方用的是真实但对不上的条款。How to apply：收到对方法规论据先三查再定应对姿态；回应时把话题拉回事实(请对方用登记用途证明属于禁令对象)，不争条文真假


## Deployment activation 的 restart receipt 不能靠命令 exit code 或有无输出来判定——systemctl restart 成功时静默无输出，operator 会误判为"没反应"；必须用 systemctl show 的 MainPID/ActiveEnterTimestamp pre/post delta 证明 restart 真实发生

`domain: finance-suite` · `2026-08-14`


harness 拦截提权命令触发 human-operator 交接。第一次声称"命令已执行完成"实际未生效(MainPID 4010635 未变，etime 连续增长)。第二次命令静默返回(exit 0 无输出)，但 post 快照 MainPID 4010635→4063908 + ActiveEnterTimestamp 09:17:48→12:54:48 + 4 worker PID 全换 + etime 归零，才证明 restart 真实发生。教训:静默成功≠no-op，判定 restart 只认 identity delta。


## 引入 per-item key 约定时，命名必须在全部 item 上一致，且 smoke 必须遍历 consumer 的完整 key 集合而非抽样

`domain: finance-suite` · `2026-08-13`


Auction receipt 链部署: Path A 用 top_gainers_receipt 而 _qc_auction 通用 reader 按 {name}_status 读,导致 top_gainers 被误标 unavailable。根因:收据键名约定只对 6 个主维度成立,可选维度走了另一套命名。修复:统一为 top_gainers_status。How to apply: additive observability 变更的 smoke 脚本要让通用 reader 消费全部 item(含 optional/skipped),单个关键 item 通过≠约定成立


## video_extract success=True 不等于 transcript 完整获取。Toutiao adapter 返回 success=True + transcript_status=unavailable_without_asr + content=""。下游必须根据 transcript_status/content_status 判断是否允许执行视频内容总结，不能只看 success 布尔值。

`domain: claude-code-infra` · `2026-08-11`


Toutiao Video Adapter Phase A: SSR metadata + VOD MP4 URLs work perfectly, but no transcript without ASR. Added transcript_status field as the canonical signal. Same pattern applies to any future platform that can fetch video info but not subtitles.


## 规则:Golden Pit四重门AND串联+稀缺性是合理设计,但在冰点/退潮regime下有3类failure:(1)Regime Inversion——正常正向因子在冰点反向(G1涨停基因变退潮遗骸/G2跌破变恶化而非洗盘/G3缩量变流动性枯竭而非蓄力/G4 BBIBOLL反弹在结构性抛售中产生假信号);(2)False Strength——缩量/超跌/独苗被误判为黄金坑;(3)Missing Context——完全缺失Market Regime/Sector Recognition/Tradability/Catalyst Verifiability。Eigenflux独立确认:回测expectancy被regime膨胀+look-ahead泄漏是沉默杀手。

`domain: trading-system` · `2026-08-11`


Track B Golden Pit×Eigenflux Ice-Point Research v0.1。根因:Golden Pit设计时只有单一regime回测(2025-04-07),无regime awareness,无sector context,无tradability filter,无PIT审计。how to apply:不修改Golden Pit代码/参数;先给所有hit附加regime标签;未来condition-al hypothesis:Golden Pit G1-G3+RS Under Stress HIGH+Sector Recognition HIGH+Tradability PASS→G4信号可信度可能高于无条件G4。等Track A PIT Experiment完成后再考虑Golden Pit Regime Adaptation。


## 规则:连板高度不能直接当强度/核心因子——价格限制产生延迟价格发现与短期动量+过度反应反转;必须与板块协同、可交易性和失败路径分开测试。独苗(高RS+低板块)与核心候选(高RS+高板块)必须分桶,其T+1/T+3/T+5/MAE分布可能不同。可交易性排除(连续一字/停牌重组)必须先于胜率统计,否则虚假命中污染结果。

`domain: trading-system` · `2026-08-11`


C+G Ice-Point Factor Candidate Matrix v0.1 研究设计。根因:连板是结果变量非原因变量;一字板=延迟价格发现=不可交易≠强度。how to apply:任何涨停/连板因子必须先过三关:(1)板块协同独立检验(2)可交易性过滤(3)成功/失败配对(PIT observable_at分开09:25/10:00/14:30/EOD,禁止EOD信息污染盘中实验)。


## 规则: 完整性链(manifest → artifact → verify)中,hash 必须取序列化后的文件字节而非序列化前的内存对象;json.dumps 与 write_json 因尾部换行符产生不同字节序列,混用导致同一语义内容 hash 校验失败

`domain: finance-suite` · `2026-08-10`


digest_value(in_memory_dict) != digest_file(written_file) 因为 write_json 末尾自动追加 \n 而 json.dumps 不追加。根因:把内存对象当作规范表示,但实际被分发和校验的是磁盘文件。How to apply:任何内容寻址链必须以序列化产物(磁盘文件)为唯一 hash 源,禁止对内存对象 hash 后再与文件 hash 比对。文件才是 artifact;hash 文件。
