# 01. 认证与身份

## 范围

本文件记录 octo-server 中与 token / cookie / WebSocket 握手相关的认证与身份线索。当前样例只覆盖已核验源码证据，不覆盖完整结论。

## 已确认结论

### 1. HTTP 登录态主链路使用 token parser，并由启动阶段注入 route

服务启动后在 `runAPI` 中创建 `CacheTokenParser`，注入 cache、`TokenCachePrefix`、`TokenValidator`、LanguageResolver、RoleResolver，并通过 `route.SetTokenParser(...)` 注册到 HTTP route。

来源: `main.go#L204-L227`

### 2. `Authorization: Bearer` 会被兼容回填到自定义 `token` 头

`BearerTokenCompat` 的注释明确：octo-lib `AuthMiddleware` 只读自定义 `token` 头；该中间件在 `token` 头缺失时，从 `Authorization: Bearer <token>` 提取凭据并写入 `token` 头。它只认 Bearer，不接受 query 参数兜底，且保留原始 Authorization 头。

来源: `pkg/wkhttp/bearer_compat.go#L17-L43`
来源: `pkg/wkhttp/bearer_compat.go#L44-L75`

`main.go` 也说明该中间件必须在各 route group 的 `AuthMiddleware` 前执行，且只做 Bearer → token 头回填。

来源: `main.go#L279-L288`

### 3. CacheTokenParser 的 Parse 会处理 token 缺失、缓存读取和 Decode

`CacheTokenParser.Parse` 在 token 为空时返回 `wkhttp.ErrTokenMissing`；没有 validator 时从 cache 读取 `Prefix + token`；空 payload 视为 token not found；读取到 payload 后调用 `Decode` 解码。

来源: `pkg/auth/parser.go#L130-L160`

### 4. token cache payload 有版本化结构

`TokenInfo` 是 token cache value 的结构化表示，包含 UID、Name、Role、Language、IssuedAt、ExpiresAt、DeviceFlag、DeviceID、SessionGeneration、SessionRevision 等字段。源码定义了 `v2:` 和 `v3:` 前缀。

来源: `pkg/auth/tokeninfo.go#L19-L37`

### 5. Decode 兼容 v2、v3 和 legacy `uid@name[@role]`

`Decode` 会根据 `v2:`、`v3:` 前缀分别进入 v2/v3 解码，否则走 legacy `uid@name[@role]`。legacy 格式要求 2 或 3 段，且 uid 不能为空。

来源: `pkg/auth/tokeninfo.go#L122-L138`
来源: `pkg/auth/tokeninfo.go#L191-L205`

### 6. TokenValidator 对 v3 token 做 TTL 和 session generation 校验

`TokenValidator.Validate` 从 token record reader 读取 `prefix+token`，缺失或空 payload 返回 token not found。v3 token 要求 Redis TTL 为有限正值、payload 未过绝对过期时间，并校验 session generation。

来源: `pkg/auth/validator.go#L76-L123`

### 7. 用户登录成功会签发/复用 token，并同步 UpdateIMToken

登录逻辑中生成 token，使用 session store 签发或复用会话；随后调用 `UpdateIMToken` 同步 IM token。若 IM token 更新失败，已新签发的 token 会尝试补偿撤销。

来源: `modules/user/api.go#L1841-L1947`

## 初步线索

- 多个业务路由通过 `ctx.AuthMiddleware(r)` 挂认证，例如 `/v1`、`/v1/user`、`/v1/user/pinned` 等。
  来源: `modules/user/api.go#L264-L287`
  来源: `modules/user/api.go#L317-L375`
- `/auth/verify`、`/auth/verify-bot`、`/auth/verify-api-key` 供 Gateway 验证用户 token、Bot API Key、daemon API Key，但具体实现还未展开核验。
  来源: `modules/user/api.go#L403-L409`

## 未确认 / 待核验

- 未确认：cookie 是否参与主认证链路；当前样例只找到 token header / Bearer 兼容证据。
- 未确认：WebSocket 握手在哪里校验，需要继续查 `pkg/botutil/ws.go`、WuKongIM 接入路径和相关路由。
- 未确认：bot token、user token、daemon API key 的完整边界，需要继续核验 `modules/bot_api/auth.go`、`modules/bot_provision/`、`modules/usersecret/`。

## 对产品问答的影响

- 可以说：HTTP API 认证主线明确围绕 `token` 头，并兼容 `Authorization: Bearer` 回填。
- 不能说：cookie 登录态或 WebSocket 握手已经完全搞清楚；这两块仍需继续核验。
