# octo-server 产品管家需求池

本仓库用于 `octo-server` 产品管家的考试需求池、知识库沉淀、Issue 收单、What-only PRD 与长时闭环扫描。

## 固定仓库口径

- 正式需求池 / issue pool：`droller-fufu/octo-server-product-agent-pool`
- 目标源码仓库：`Mininglamp-OSS/octo-server`
  - 用途：考前源码核验 / 知识库依据 / 产品理解。
  - 纪律：正式考试期间优先使用已整理知识库；不现场 grep / 搜索 / 读取目标源码。知识库未覆盖时说“未确认 / 待核验”，不编造。

## 正式答题入口

正式答题优先使用 `knowledge/` 下 10 个主知识库文件：

- `knowledge/overview.md`
- `knowledge/auth-and-identity.md`
- `knowledge/authorization-model.md`
- `knowledge/config.md`
- `knowledge/modules.md`
- `knowledge/api-error.md`
- `knowledge/im-control-plane.md`
- `knowledge/bot-agent.md`
- `knowledge/storage-dependencies.md`
- `knowledge/build-release.md`

补充材料：

- `knowledge/source-audit/`：证据底稿补充，只代表已沉淀的重点证据，不宣称覆盖所有领域的源码证据。
- `docs/exam-summary.md`：考前收口总结、已修正风险与 cron 边界。

## PM 链路

- Bug / Feature / Question 统一进入本仓库 Issues。
- PRD 跟随 Issue 沉淀，使用 `PRD_TEMPLATE.md`，坚持 What-only：只写用户要什么、为什么、怎么验收，不写内部实现方案。
- Issue 模板位于 `.github/ISSUE_TEMPLATE/`。

## 长时闭环

- cron / Loop 口径：每 20 分钟扫描一次需求池。
- 扫描内容：新增 Issue、评论变化、label 变化、状态变化、PRD 更新、review 阻塞、close/reopen。
- 播报规则：有变化才在考试群 @AINOL考官 播报；无变化静默，只更新 `runtime/scan_state.json` / 运行状态。

## 安全红线

- 不写 `Mininglamp-OSS/octo-server` 目标源码仓库。
- 不打印 token、header、env、secret、私密 URL。
- 不把需求池结论伪装成源码证据。
- 不确定就说“未确认 / 待核验”。
