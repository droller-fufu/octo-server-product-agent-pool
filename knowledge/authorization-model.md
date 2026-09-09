# 02 权限与授权模型

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. 普通用户 API 与 Bot API 是两条不同鉴权链路

普通用户 API 使用 server 级 `AuthMiddleware`。启动阶段会把 `CacheTokenParser` 注册到路由，并注入 token validator、语言解析器和角色解析器。角色解析器会在 Parse 阶段按 UID 实时解析当前系统角色，避免 admin/superAdmin 只依赖 token 快照。

来源: main.go#L197-L227
来源: pkg/auth/parser.go#L45-L57
来源: pkg/auth/parser.go#L189-L208

Bot API 不走普通用户登录态，而是从 `Authorization: Bearer ...` 读取 token；`app_` 前缀进入 App Bot 分支，否则进入 User Bot 分支。

来源: modules/bot_api/auth.go#L25-L40
来源: modules/bot_api/auth.go#L143-L149

## 2. token payload 有结构化版本，但 v3 token 有更严格校验

`TokenInfo` 是 token cache value 的规范结构，包含 UID、Name、Role、Language、IssuedAt、ExpiresAt、DeviceFlag、DeviceID、SessionGeneration、SessionRevision 等字段；v2/v3 使用前缀区分，旧格式 `uid@name[@role]` 仍可解码。

来源: pkg/auth/tokeninfo.go#L24-L37
来源: pkg/auth/tokeninfo.go#L66-L83
来源: pkg/auth/tokeninfo.go#L122-L148

`TokenValidator` 会读取 token record 并解码；v3 token 要求 Redis TTL 有限、绝对过期时间未到、session generation 可用且匹配，否则按无效 token 拒绝。

来源: pkg/auth/validator.go#L76-L100
来源: pkg/auth/validator.go#L101-L123

## 3. 系统角色存在固定角色与能力映射，但不是通用 RBAC 全表

`manager_roles.go` 定义了两个固定角色：`dashboardReader` 与 `marketAdmin`。注释明确它们不加入 octo-lib 的通用 `CheckLoginRole`，避免自动获得所有 admin endpoint 权限；它们只在特定能力函数中生效。

来源: pkg/auth/manager_roles.go#L5-L24

`IsManagerConsoleRole` 决定哪些角色可进入管理控制台；`CanAdminMarketplace` 与 `CanReadManagerDashboard` 分别表达 marketplace 管理面与运营看板读取面的授权策略。

来源: pkg/auth/manager_roles.go#L63-L90

未确认 / 待核验：完整 org 级 RBAC 权限点表、所有管理端 endpoint 的角色矩阵，当前源码中未在一个中心文件集中声明。

## 4. Space 成员校验是独立的隔离边界

`SpaceMiddleware` 从 query/header 提取 `space_id`，使用当前登录 UID 校验 space membership；无登录 UID 返回 401，非成员返回 403。成员关系结果会以 Redis 缓存，正向 TTL 60s、否定 TTL 30s。

来源: pkg/space/middleware.go#L146-L205

缓存失效路径对 DEL 失败有额外防护：如果正向缓存删不掉，会尝试写入短 TTL 的否定缓存，避免被移除成员在 60s 正向缓存期间继续访问。

来源: pkg/space/middleware.go#L67-L110

## 5. App Bot 管理面区分平台级与 Space 级授权

`app_bot` 管理路由分为平台级 `/v1/admin/app_bot` 与空间级 `/v1/space/:space_id/app_bot`，两者都先过用户 AuthMiddleware。平台创建/列表会调用 `CheckLoginRole`；Space 级路由依赖 `checkSpaceAdmin` 校验当前登录 UID 是目标 Space 的 admin/owner。

来源: modules/app_bot/app_bot.go#L116-L153
来源: modules/app_bot/app_bot.go#L309-L315
来源: modules/app_bot/app_bot.go#L442-L447

Space admin 的源码口径是：`space_member` 中 `status=1`，且 `role >= 1`；注释说明 `0=member, 1=admin, 2=owner`。

来源: modules/app_bot/app_bot.go#L45-L48
来源: modules/app_bot/app_bot.go#L156-L168

`botInRouteScope` 防止跨作用域 IDOR：平台路由只能管理 platform scoped bot；Space 路由只能管理本 Space 的 bot。

来源: modules/app_bot/app_bot.go#L171-L183

## 6. User Bot 与 App Bot 的认证权威不同

User Bot 认证查询 `robot` 表，要求 `bot_token` 非空且 `status=1`；成功后写入 `robot_id`、`bot_kind=user` 和 robot 对象。

来源: modules/bot_api/auth.go#L45-L62
来源: modules/bot_api/db.go#L45-L52

App Bot 认证先查共享 registry/cache，未命中或错误时回落 DB；DB 命中后仍要求 `status=1` 才可服务 API 请求。

来源: modules/bot_api/auth.go#L64-L129

## 7. Bot API 主组中间件顺序是授权安全约束

Bot API 主组顺序是 `authBot → requireBotIdentity → per-bot rate limit`。注释明确 `requireBotIdentity` 只断言 bot 身份存在，不把 `robot_id` 写成普通登录用户的 `uid`，避免 handler 调 `GetLoginUID()` 时产生授权混淆。

来源: modules/bot_api/bot_api.go#L377-L408

## 8. 高风险自愈接口有额外限流边界

`/v1/bot/register` 按 `r.Any` 挂载，先过 per-IP strict，再过 per-token 指纹限流，最后进入 register；注释说明这是为了防止随机 token 放大 Redis keyspace，以及保护 bot 自愈链路。

来源: modules/bot_api/bot_api.go#L299-L339

`/v1/bot/heartbeat` 独立于 Bot API 主组，有 per-IP strict、authBot、requireBotIdentity、per-bot 桶三层顺序；注释说明它不能被业务流量挤掉，也不能让无效 token 无限触发 DB/Redis 查询。

来源: modules/bot_api/bot_api.go#L341-L375

## 未确认 / 待核验

- 普通用户所有业务路由的完整 ACL/owner 检查矩阵未在本轮穷尽。
- org/dept 组织权限、channel ACL 与 Space ACL 的冲突优先级仍需专项核验。
