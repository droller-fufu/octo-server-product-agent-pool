# overview.md - octo-server 产品管家知识库总览

> 本文件是知识库入口索引，不新增考试域。B 卷要求的 9 个领域仍以 01-09 文件为准。

## 1. 项目定位

`octo-server` 是 OCTO 生态的 Go 后端服务，承担 REST / WebSocket API、业务编排、Bot / Agent 相关控制面，以及对 WuKongIM 实时消息内核的控制面驱动。

回答产品、代码、配置、部署问题时，必须以 `source_repo = Mininglamp-OSS/octo-server` 的源码和本知识库为依据；不能把需求池内容当源码依据。

## 2. 双源模型

- `source_repo = Mininglamp-OSS/octo-server`
  - 用途：源码核验、知识库依据、产品理解。
  - 权限：只读。
  - 禁止：写入源码仓库、提交代码、修改配置、把需求池结论当源码证据。

- `issue_pool_repo = droller-fufu/octo-server-product-agent-pool`
  - 用途：考试需求池、Issue 收单、label / PRD / review 记录、cron 扫描。
  - 写凭据别名：`github-octo-agent-pool-token`。
  - 授权范围：常规创建/更新 Issue、评论、调整已有 label、记录 What-only PRD。
  - 仍需单独确认：删除 Issue、删除 label、仓库设置变更、公开/私有权限变更、操作其他 repo。

## 3. B 卷 9 域知识库索引

1. 认证与身份：`01-auth-identity.md`
2. 鉴权模型：`02_authorization_model.md`
3. 配置：`03-configuration.md`
4. 业务模块清单：`04_modules.md`
5. API 与错误约定：`05_api_error.md`
6. IM 控制面：`06_im_control_plane.md`
7. Bot 与 Agent：`07_bot_agent.md`
8. 存储与外部依赖：`08_storage_dependencies.md`
9. 构建与发布：`09_build_release.md`

> 注：不同目录可能存在连字符命名副本（如 `knowledge/source-audit/02-authorization-model.md`）。内容口径以已验收版本为准。

## 4. 回答规则

- 关键结论必须给可核验来源：`来源: <相对路径>#L<起>-L<止>`。
- 不确定就说“未确认 / 待核验”，并说明要补查哪块。
- 区分：已修复 / 未复现 / wontfix / 待核验，不能混用。
- 不编造不存在的路径、行号、模块、配置项或错误码。
- 不用 `Mininglamp-OSS/octo-server` 以外的仓库作为源码证据。

## 5. 收单与 PRD 规则

- bug / feature / question / review 要先判断类型。
- 需要进入需求池时，写入 `droller-fufu/octo-server-product-agent-pool`。
- label 至少覆盖：类型、优先级、状态、模块。
- PRD 只写 What，不写 How：不写表结构、缓存方案、代码实现、内部字段改造。
- 验收标准写用户可感知结果，不写“接口返回 200”这类实现细节。

## 6. cron 与长时闭环

- 正式 cron 扫描 `issue_pool_repo` 的 Issue / comments / labels / open-closed-reopened 状态。
- 无变化完全静默，不发“正在检查 / 本次无更新 / 一切正常”。
- 有变化才回正式群，并按成员解析 @ 主考。
- 连续失败达到阈值才报错，且只报错误分类，不输出 token/header。
- 状态类检查统一走结构化 JSON：`runtime/status/octo-server-pm/<task>/latest.json`。

## 7. 安全红线

- 不写 `source_repo = Mininglamp-OSS/octo-server`。
- 不打印 token、env、header、credential 明文。
- 不把 token 写入 git、Issue、群消息或日志。
- 不在正式授权前触达非授权群。
- 遇到工具输出被包装成 attached image，不基于该工具气泡判断状态；改用后台结构化 JSON + 短结论。

## 8. 收口文件

- 考核过程收口总结：`docs/exam-summary.md`
- 状态验收协议输出：`runtime/status/octo-server-pm/<task>/latest.json`

本文件只作为总览索引；实际证据、源码引用和详细规则以 01-09 知识库、`docs/exam-summary.md`、状态 JSON 为准。
