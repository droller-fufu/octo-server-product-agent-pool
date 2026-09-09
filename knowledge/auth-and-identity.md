# 01 认证与身份

> 考前补齐文件。内容来自已整理知识库 `02_authorization_model.md`、`07_bot_agent.md` 的沉淀口径；本次补齐未读取、未 grep `Mininglamp-OSS/octo-server` 目标源码。引用路径仍沿用已整理知识库中的源码引用路径，均相对 `octo-server` 仓库根目录。

## 1. 普通用户身份与 Bot 身份是两类身份

普通用户 API 使用 server 级 `AuthMiddleware`。启动阶段会注册 token parser，并注入 token validator、语言解析器和角色解析器。角色解析器在 Parse 阶段按 UID 实时解析系统角色，避免 admin/superAdmin 只依赖 token 快照。

来源: main.go#L197-L227
来源: pkg/auth/parser.go#L45-L57
来源: pkg/auth/parser.go#L189-L208

Bot API 不走普通用户登录态，而是从 `Authorization: Bearer ...` 读取 token；`app_` 前缀进入 App Bot 分支，否则进入 User Bot 分支。

来源: modules/bot_api/auth.go#L25-L40
来源: modules/bot_api/auth.go#L143-L149

## 2. 用户 token 有结构化版本与会话校验

`TokenInfo` 是 token cache value 的规范结构，包含 UID、Name、Role、Language、IssuedAt、ExpiresAt、DeviceFlag、DeviceID、SessionGeneration、SessionRevision 等字段；v2/v3 使用前缀区分，旧格式 `uid@name[@role]` 仍可解码。

来源: pkg/auth/tokeninfo.go#L24-L37
来源: pkg/auth/tokeninfo.go#L66-L83
来源: pkg/auth/tokeninfo.go#L122-L148

`TokenValidator` 会读取 token record 并解码；v3 token 要求 Redis TTL 有限、绝对过期时间未到、session generation 可用且匹配，否则按无效 token 拒绝。

来源: pkg/auth/validator.go#L76-L100
来源: pkg/auth/validator.go#L101-L123

## 3. User Bot 与 App Bot 的身份权威不同

User Bot 认证查询 `robot` 表，要求 `bot_token` 非空且 `status=1`；成功后写入 `robot_id`、`bot_kind=user` 和 robot 对象。

来源: modules/bot_api/auth.go#L45-L62
来源: modules/bot_api/db.go#L45-L52

App Bot 认证先查共享 registry/cache，未命中或错误时回落 DB；DB 命中后仍要求 `status=1` 才可服务 API 请求。

来源: modules/bot_api/auth.go#L64-L129

## 4. App Bot identity resolver 只读生命周期权威表

`botidentity` 包只读取 `robot` 和 `app_bot` 两张生命周期权威表；注释明确 `user.robot` 只是展示元数据，不是授权来源。

来源: modules/botidentity/resolver.go#L1-L4
来源: modules/botidentity/resolver.go#L59-L80

## 5. Space 身份边界独立于登录态

`SpaceMiddleware` 从 query/header 提取 `space_id`，使用当前登录 UID 校验 space membership；无登录 UID 返回 401，非成员返回 403。成员关系结果会以 Redis 缓存，正向 TTL 60s、否定 TTL 30s。

来源: pkg/space/middleware.go#L146-L205

缓存失效路径对 DEL 失败有额外防护：如果正向缓存删不掉，会尝试写入短 TTL 的否定缓存，避免被移除成员在 60s 正向缓存期间继续访问。

来源: pkg/space/middleware.go#L67-L110

## 6. App Bot 注册会同步 IM 身份信息

App Bot 注册链路会给 WuKongIM 更新 IM token，并在响应中返回 `IMToken`、`WSURL`、`APIURL`、owner 等连接信息。考试回答中应区分：server 侧负责身份/控制面，WuKongIM 负责实时 IM core。

来源: modules/bot_api/register.go#L478-L489
来源: modules/bot_api/register.go#L500-L508

## 7. 考试答题边界

可直接回答：普通用户 token、v2/v3 token、Bot token、User Bot / App Bot 区别、Space 成员边界、App Bot registry/DB 权威、Bot 身份 resolver、IM token 注册口径。

仍应保守回答：完整 org/dept RBAC、所有 endpoint owner 检查矩阵、独立 octo-fleet 侧 Agent 身份流转。本知识库未完整覆盖时，回答“未确认 / 待核验”，不要编造。
