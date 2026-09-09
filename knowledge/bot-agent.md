# 07 Bot 与 Agent

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. Bot API 区分 User Bot 与 App Bot

Bot API 认证中定义 `BotKindUser = "user"` 与 `BotKindApp = "app"`。鉴权中间件按 token 前缀路由：`app_` 进入 App Bot 认证，否则进入 User Bot 认证。

来源: modules/bot_api/auth.go#L10-L23
来源: modules/bot_api/auth.go#L25-L40

User Bot 认证查 `robot` 表，要求 `bot_token` 非空且 `status=1`；成功后写入 `robot_id`、`bot_kind=user` 和 robot 对象。

来源: modules/bot_api/auth.go#L45-L62
来源: modules/bot_api/db.go#L45-L52

App Bot 认证先查共享 registry，未命中或错误时回落 DB；DB 路径仍要求 `status=1`。

来源: modules/bot_api/auth.go#L64-L129

## 2. `app_bot` 是平台/空间 App Bot 的管理面

`app_bot` 模块注册自己的 API 与 SQL 迁移目录。

来源: modules/app_bot/1module.go#L13-L20

`NewAppBot` 会把 App Bot identity resolver 注册到 user 模块，并初始化共享 Redis auth registry；该 registry 由 app_bot 模块启动时设置给 bot_api。

来源: modules/app_bot/app_bot.go#L73-L101

管理路由分为平台级 `/v1/admin/app_bot` 和空间级 `/v1/space/:space_id/app_bot`，同时提供用户发现与申请入口。

来源: modules/app_bot/app_bot.go#L116-L153

App Bot 写入 DB 前会做跨表 UID 冲突检查：不能与 user.uid 或 robot.robot_id 冲突；同表唯一约束仍是最终兜底。

来源: modules/app_bot/db.go#L43-L85

## 3. App Bot registry：DB 是权威，Redis 是共享 write-through cache

`AppBotRegistryInterface` 注释说明有两类实现：单进程内存 adapter 与共享 Redis write-through cache。FindByToken 在 miss 或 backend error 时返回 nil，让 authAppBot 回落权威 DB；不能 fail-open。

来源: modules/bot_api/registry.go#L15-L24

全局 App Bot registry 用 atomic pointer 存储，由 app_bot 模块设置，bot auth 热路径 lock-free 读取。

来源: modules/bot_api/registry.go#L46-L63

Redis registry key 形式是 `appbot:auth:{sha256hex(token)}`，避免明文 bearer token 出现在 Redis key、RDB、MONITOR 或运维工具中。

来源: modules/bot_api/registry_redis.go#L17-L23

Redis registry 的权威模型是：`app_bot` 表是 source of truth；miss、tombstone 或 Redis error 都回落 DB；Add/Warm/Remove 的不对称写模型用于防止撤销后被 warm-up 复活。

来源: modules/bot_api/registry_redis.go#L54-L80

## 4. `botfather` 是 User Bot 的创建/管理入口，Bot API 已迁出

`botfather` 模块注册 API 与 SQL 迁移目录。

来源: modules/botfather/1module.go#L16-L23

`BotFather.Route` 注释明确 `/v1/bot/*` Bot API endpoints 已迁移到 `modules/bot_api`；BotFather 当前主要处理文档、User Bot 管理、User API Key、Robot Apply。

来源: modules/botfather/api.go#L83-L108

BotFather 启动时会同步所有 bot token 到 WuKongIM，避免 WuKongIM 重启后 token 丢失。

来源: modules/botfather/api.go#L89-L91

## 5. `bot_provision` 是 fleet split 后的跨服务 bot 凭据边界

`bot_provision` 文件头说明它承载 fleet split 后的新 contract surface：`POST /v1/bot/mint` 给 web/session auth 创建 bot；`GET /v1/bot/:uid/token` 给 daemon 使用 `uk_` API key 获取 bot token。

来源: modules/bot_provision/bot_api.go#L1-L19

`mintBot` 复用 session middleware 语义，检查 display name、space_id，并确认调用者属于目标 Space，防止任意登录用户向任意 Space mint bot。

来源: modules/bot_provision/bot_api.go#L48-L91

`botToken` 路径验证 Bearer API key，要求目标 robot status=1、bot.creator_uid 等于 caller UID、bot 是 callerSpace 成员，才返回 bot_token。

来源: modules/bot_provision/bot_api.go#L96-L171

`Route` 挂载 `POST /v1/bot/mint` 与 `GET /v1/bot/:uid/token`；注释说明 JWT exchange/JWKS 已移除。

来源: modules/bot_provision/bot_api.go#L190-L205

## 6. botidentity 是活动 Bot 身份解析器

`botidentity` 包只读取 `robot` 和 `app_bot` 两张生命周期权威表；注释明确 `user.robot` 只是展示元数据，不是授权来源。

来源: modules/botidentity/resolver.go#L1-L4

resolver 使用单条 SQL 同时读取两张权威表及授权元数据，并且不做包级缓存；调用方需要展示缓存时自行负责。

来源: modules/botidentity/resolver.go#L59-L80

## 7. Agent runtime 不在本仓库直接承载

`internal/modules.go` 注释说明 `modules/runtime` 已移除，runtime/bot orchestration 由独立 `octo-fleet` 服务负责。本仓库保留的是 server 侧 Bot/API/IM 控制面与身份/凭据边界。

来源: internal/modules.go#L53-L57

## 8. 未确认 / 待核验

- Agent 会话如何由 octo-fleet 启动/恢复，需要查独立服务或部署侧文档。
- Bot events 与外部 OpenClaw/Lobster adapter 的完整协议需要继续按 `/v1/bot/events` 相关 handler 展开。
