# 04 业务模块清单

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. 模块注册入口是 `internal/modules.go` 的 blank import

`internal/modules.go` 明确说明：SQL migration 执行顺序由 SQL 文件名时间戳决定，不由 blank import 顺序决定；Go init 顺序也由包依赖图决定。文件中保留历史顺序只是方便扫描，不是 load-bearing。

来源: internal/modules.go#L1-L18

当前被注册到 server 启动链路的模块由 blank import 列出，集中在 `internal/modules.go`。

来源: internal/modules.go#L22-L78

## 2. 注册模块清单

按 `internal/modules.go` 当前 blank import，可确认注册模块包括：

- agentmailgateway
- backup
- base
- robot
- bot_mention
- botfather
- card_template_catalog
- category
- channel
- common
- conversation_ext
- file
- group
- incomingwebhook
- integration
- internal_resolve
- message
- messages_search
- notification
- notify
- oidc
- opanalytics
- openapi
- qrcode
- report
- search
- space
- statistics
- sticker
- thread
- user
- usersecret
- bot_api
- app_bot
- bot_provision
- voice_adapter
- webhook
- workplace

来源: internal/modules.go#L22-L78

注意：本轮只把 `internal/modules.go` 的 blank import 视为“启动注册入口”的证据；`modules/` 目录下存在的辅助包不一定都是 register.Module。

## 3. runtime 模块已移出本仓库

`internal/modules.go` 注释说明 `modules/runtime` 已移除，runtime/bot orchestration 由独立 `octo-fleet` 服务负责；历史 SQL migration 表记录保留在 `gorp_migrations` 中。

来源: internal/modules.go#L53-L57

## 4. thread 模块存在运行时开关

Thread schema 会始终注册，以保证 DB layout 一致；但当 `DM_THREAD_ON` 未开启时，只返回模块名与 SQLDir，API 与 archive worker 不启动。开启后才注册 Start/Stop、API、Swagger 和 IMDatasource/BussDataSource。

来源: modules/thread/1module.go#L24-L43
来源: modules/thread/1module.go#L53-L73

## 5. 核心模块职责速览

| 模块 | 主要职责 | 证据 |
|---|---|---|
| `base` | 基础 app/API 与 SQL | 来源: modules/base/1module.go#L14-L22 |
| `user` | 用户、好友等用户域 API 与 SQL | 来源: modules/user/1module.go#L26-L40 |
| `group` | 群组 API、群成员检查器、群域服务 | 来源: modules/group/1module.go#L22-L45 |
| `channel` | 频道 API、频道服务与 SQL | 来源: modules/channel/1module.go#L16-L25 |
| `message` | 消息、会话、管理 API 与 SQL | 来源: modules/message/1module.go#L26-L54 |
| `file` | 文件上传/下载 API | 来源: modules/file/1module.go#L13-L19 |
| `robot` | User Bot 基础数据与 SQL | 来源: modules/robot/1module.go#L16-L25 |
| `botfather` | User Bot 管理、文档、User API Key、Robot Apply | 来源: modules/botfather/1module.go#L16-L23；来源: modules/botfather/api.go#L83-L108 |
| `bot_api` | Bot 对外 API：register、send、events 等 | 来源: modules/bot_api/1module.go#L13-L20 |
| `app_bot` | 平台/空间 App Bot 管理面、共享 auth registry | 来源: modules/app_bot/1module.go#L13-L20；来源: modules/app_bot/app_bot.go#L73-L101 |
| `bot_provision` | 跨服务/daemon 的 bot mint 与 token lookup | 来源: modules/bot_provision/1module.go#L8-L13；来源: modules/bot_provision/bot_api.go#L1-L19 |
| `bot_mention` | Bot mention ingress/API | 来源: modules/bot_mention/1module.go#L8-L13 |
| `notification` | 通知设置/通知相关 API 与 SQL | 来源: modules/notification/1module.go#L16-L26 |
| `notify` | 通知发送/卡片通知 API 与 SQL | 来源: modules/notify/1module.go#L13-L23 |
| `oidc` | OIDC 登录/绑定等身份集成与 SQL | 来源: modules/oidc/1module.go#L13-L22 |
| `openapi` | OpenAPI 模块 API | 来源: modules/openapi/1module.go#L13-L20 |
| `incomingwebhook` | 入站 webhook API 与 SQL | 来源: modules/incomingwebhook/1module.go#L13-L25 |
| `webhook` | webhook 模块 API 与 SQL | 来源: modules/webhook/1module.go#L13-L22 |
| `space` | Space/组织空间 API 与 SQL | 来源: modules/space/1module.go#L17-L39 |
| `workplace` | workplace 用户侧/管理侧 API 与 SQL | 来源: modules/workplace/1module.go#L16-L33 |
| `messages_search` | 消息搜索 API | 来源: modules/messages_search/1module.go#L13-L17 |
| `search` | 搜索模块 API | 来源: modules/search/1module.go#L13-L20 |
| `report` | report 用户侧/管理侧 API 与 SQL | 来源: modules/report/1module.go#L16-L36 |
| `backup` | 备份管理 API 与 SQL | 来源: modules/backup/1module.go#L13-L20 |
| `usersecret` | 用户外部密钥别名表、write-only CRUD、resolve | 来源: internal/modules.go#L64-L66；来源: modules/usersecret/1module.go#L13-L22 |
| `voice_adapter` | 语音适配器 API 与 SQL | 来源: modules/voice_adapter/1module.go#L14-L28 |

## 6. 未确认 / 待核验

- 每个模块的完整路由表未在本轮展开。
- 哪些模块在生产环境实际启停，需要结合部署配置与环境变量核验。
- `modules/` 目录下未被 `internal/modules.go` blank import 的辅助包，不应直接算作注册业务模块。
