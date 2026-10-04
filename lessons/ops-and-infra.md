# 运维与基础设施

> 跨仓库 review 四元组;代理/TUN 下 SSH 需 DIRECT;明文凭据 scp 后必须清理;哈希前先脱敏 token

共 4 条。源自 Engram 跨 agent 记忆库,已脱敏。


## add-account 脚本内嵌明文密码 scp 到服务器 /tmp,写库后必须 rm 掉,否则服务器残留明文凭据

`domain: finance-suite` · `2026-09-10`


SOP 未含清理步骤;本次 2261/vip 执行零偏差,补做 ssh rm /tmp/add_user.py;schema 确认未漂移(hashed_password 非 password_hash,tier 域为 admin/free/vip)


## 取证期间被第三条路解决的问题，其修复方案的 A/B 选项会同时失效——决策前须重验前提仍然成立

`domain: claude-code-infra` · `2026-09-08`


根因：A/B 决策树建立在『某故障仍在阻塞』这一前提上，但取证耗时期间另一条独立修复（代理注入）已让故障消失，两个选项一起变成 moot，其中一个还会扩张信任面。how to apply：给出修复选项前，重跑一次最小判据确认故障仍复现（读 runtime receipt 的时间序列，而非沿用取证开始时的快照）；发现前提已变，先宣告 MOOT 并重述真实残余风险，不要因为已投入取证成本而硬推原方案。


## 后端 deploy 脚本设计中，rsync manifest 构建时过滤器必须在 sed 转换为相对路径之后才能正确匹配；在绝对路径上 grep -v 会静默漏过所有目标，导致 frontend-owned 文件（如 static/app/）出现在 backend manifest 里。配对 tag 校验必须用精确双字段解析而非 grep -F 子串匹配，否则注释行或多余 token 可误通过 gate。

`domain: finance-suite` · `2026-08-16`


deploy-backend.sh Phase 3 设计稿：三处关键修复——(1) find|sed|grep-v 顺序，过滤在相对路径上操作；(2) release-pairs.txt 用 while read field1 field2 extra 精确双字段解析，拒绝 extra token 和注释行；(3) 第三参数 allowlist 只接受 --dry-run，其他值立即 exit 1 防止误写生产。


## 规则: 平台按 bot/app/tenant 维度分配的对端用户标识不跨实体通用,换实体后照搬白名单会得到'服务在跑但不回话'

`domain: hermes` · `2026-08-07`


原因: 此类标识是(用户,实体)的函数,换实体后同一个人的标识随之变化,旧标识在新实体下被静默过滤。判别方法: 检索该标识在本地状态目录的出现位置,若只出现在旧实体的凭据缓存与旧会话状态里,而在新实体状态目录零出现,即构成陈旧性证据。适用: 换 bot/app/tenant 后,凡存过对端用户标识的白名单、映射表、缓存一律视为失效;必须从真实入站消息重新取标识,禁止猜测、照搬,也不要为了跑通而临时放开为通配。
