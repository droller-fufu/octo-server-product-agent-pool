# octo-server 源码审计知识库收口总结

## 1. 已完成范围

已完成 9 域知识库整理与源码引用核验：

01. 基础项目结构与启动路径
02. 授权模型与认证边界
03. 数据模型与迁移
04. 业务模块注册与边界
05. API / Error Contract
06. IM 控制面
07. Bot / Agent 边界
08. 存储与外部依赖
09. 构建与发布

本轮干净收口会话重点复验 7 域：02、04、05、06、07、08、09。复验项包括：文件存在、非空、含来源引用；WARN 修正点不再引用旧行号；禁忌字符串与疑似明文 secret 扫描；源码仓库只读 git 状态检查。

## 2. 工具异常与修复记录

旧会话中，`read` / `write` / `exec` / `grep` / `cat` 等工具结果曾出现被包装成 `attached image` / `see attached image` 的异常表现。为避免把不可核验输出写入知识库，旧会话已主动冻结，不再作为写入执行环境。

后续新干净会话完成 10 次工具链复测，结果均为纯文本输出；之后才继续执行核验与收口。旧会话仅作为异常背景记录，不作为知识库写入或最终验收执行环境。

## 3. 冻结 / 恢复策略

- 自检通过前不写源码仓库。
- 采用分批验收：先确认工具链输出可靠，再复验知识库文件与关键源码引用。
- 发现 WARN 后先修正再收口，不把 WARN 状态固化进总结。
- 不写实现细节推测；如果没有源码证据，统一写“未确认 / 待核验”。

## 4. 7 域验收 / 修正摘要

本轮复验的 02、04、05、06、07、08、09 均存在、非空，并包含来源引用。

3 处 WARN 已修正：

1. 04 业务模块中 runtime / octo-fleet 相关结论已改为使用 `internal/modules.go#L58-L61`，支撑点为源码注释：`modules/runtime` 已移除，runtime/bot orchestration 由独立 `octo-fleet` 服务负责。
2. 07 Bot / Agent 边界中 runtime / octo-fleet 相关结论已改为使用 `internal/modules.go#L58-L61`，并明确本仓库保留 server 侧 Bot/API/IM 控制面与身份/凭据边界，Agent runtime 需查独立服务或部署侧文档。
3. 06 IM 控制面中 App Bot register / `UpdateIMToken` 相关结论已改为使用 `modules/bot_api/register.go#L478-L489` 与 `modules/bot_api/register.go#L500-L508`，支撑点为 App Bot 使用同一 token 作为 API auth 与 IM WebSocket token，并在响应中返回 `IMToken`、`WSURL`、`APIURL`、owner 信息。

同时复验确认：

- 知识库中不再出现旧 WARN 引用 `internal/modules.go#L53-L57`。
- 知识库中不再出现旧 WARN 引用 `modules/bot_api/register.go#L124-L174`。
- 知识库中未发现禁用仓库字符串 `Mininglamp-OSS/octo-server`。
- 保守扫描未发现裸 token / API key / secret 明文。
- 对源码引用做了保守存在性检查；未发现影响验收的明显编造路径。个别正文中出现文件短名说明时，以同段的完整 `来源:` 引用为准。

## 5. 当前仓库状态

源码仓库 `/home/mlclaw/.openclaw/workspace-octo-server-pm/repos/octo-server` 仅做只读核验：

- `git status --short`：无输出。
- `git diff --stat`：无输出。

workspace 根目录 `/home/mlclaw/.openclaw/workspace-octo-server-pm` 不是 git 仓库。

本轮未修改源码仓库，未改配置，未创建 cron，未读取或打印 token，未新建或删除 Issue。

## 6. cron 状态与下一阶段

cron 已进入测试/正式双轨状态：

- 测试 cron：`a7c13b0d-b361-4dfb-82b4-adc79283a2b3`
  - 名称：`octo-server issue pool dry-run scanner (test)`
  - 频率：每 20 分钟一次
  - 用途：shadow / dry-run 对照，扫描 `issue_pool_repo`，有变化只回当前测试群
  - 状态：保留用于对照；正式 cron 首轮稳定后建议 disable，不删除
- 正式 cron：`b6c8ee6b-a782-4919-aa3b-cedc85fe6294`
  - 名称：`octo-server issue pool scanner (formal)`
  - 频率：每 20 分钟一次
  - 用途：只读扫描正式需求池，有变化才回正式群 `FDE考核 - 张春英`
  - 回群对象：`group:cdf24f06606a44aea24b41d1dac57b90`

固定双源模型：

- `source_repo = Mininglamp-OSS/octo-server`：源码核验 / 知识库依据 / 产品理解；只读，不写入，不作为 cron 扫描源。
- `issue_pool_repo = droller-fufu/octo-server-product-agent-pool`：需求池 Issue / comments / labels / open-closed-reopened 状态监控。
- `github_write_secret_alias = github-octo-agent-pool-token`：仅用于 `issue_pool_repo` 的 Issue 创建 / 评论 / label 操作；禁止打印 token、读取明文、写入 git、发群、用于 `source_repo`。

正式 cron 安全边界：

- 默认只读扫描需求池。
- 无变化静默。
- 有新 Issue / 新评论 / label 变化 / open-closed-reopened 状态变化时才回正式群。
- 连续失败 3 次才回正式群报告失败。
- 不改 Issue、不写源码仓库、不打印或读取 token。

下一阶段建议：观察正式 cron 首次运行是否成功建立正式 baseline；确认只回正式群、未误报历史 Issue、未改 Issue / 未写源码 / 未泄露 token 后，再 disable 测试 cron。

## 7. 下午正式考试统一口径（2026-09-09 补充）

- 正式需求池只使用 `droller-fufu/octo-server-product-agent-pool`，不要混用其他参考仓库名。
- 正式答题优先查 `knowledge/*.md` 10 个主知识库与 `docs/exam-summary.md`；`knowledge/source-audit/` 只是证据底稿补充，不宣称覆盖所有领域的源码证据。
- cron / Loop 统一口径：每 20 分钟扫描一次需求池；检查 Issue、评论、label、状态、PRD、review 阻塞、close/reopen；有变化才在考试群 @AINOL考官 播报，无变化静默。
- Bug Issue 最小字段：现象、影响范围、复现步骤、期望结果、实际结果、优先级、模块、下一步。
- Feature / Enhancement 最小字段：背景、用户故事、范围、非范围、验收标准、优先级、模块。
- PRD 坚持 What-only，只写用户可感知目标和验收标准，不写内部实现方案。
- 正式考试期间不现场 grep / 搜索 / 读取 `Mininglamp-OSS/octo-server` 目标源码；知识库未覆盖就说“未确认 / 待核验”，不编造。
