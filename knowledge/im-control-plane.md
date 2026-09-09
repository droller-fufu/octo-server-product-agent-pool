# 06 IM 控制面

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. server 是业务/API/Agent 编排中心，WuKongIM 是实时 IM core

README 将 server 定位为 Go backend：提供 REST + WebSocket API，编排业务逻辑和 Lobster/Agent 调度，并驱动 WuKongIM IM core 处理实时消息。

来源: README.md#L29-L37

README 还说明本仓库内置 MySQL-compatible SQL migrations 与对象存储适配，WuKongIM 通过 thin control-plane boundary 驱动，使 IM core 保持可替换。

来源: README.md#L39-L43

请求链路中，server 负责认证、授权、业务逻辑/Agent session 启动或恢复，最后把 IM 消息 fan-out 到 WuKongIM。

来源: README.md#L85-L91

## 2. 发送链路由 server 做业务处理，再调用 octo-lib Context 发往 WuKongIM

Bot API 中 `dispatchMsgSendReq` 的注释明确：把构造好的 `MsgSendReq` 发到 WuKongIM；生产路径调用 `ba.ctx.SendMessageWithResult(req)`。

来源: modules/bot_api/bot_api.go#L207-L214

普通消息发送路径在 `modules/message/api.go` 中完成 payload/mention/业务处理后，通过 `m.ctx.SendMessage(&config.MsgSendReq{...})` 发出。

来源: modules/message/api.go#L754-L775

## 3. Bot 注册会给 WuKongIM 更新 IM token，并返回 WS 连接信息

User Bot register 使用 robot 的 bot_token 作为 IM token，调用 `ctx.UpdateIMToken`；响应包含 `IMToken`、`WSURL`、`APIURL` 和 owner 信息。

来源: modules/bot_api/register.go#L47-L76
来源: modules/bot_api/register.go#L104-L121

App Bot register 也会调用 `ctx.UpdateIMToken`；注释明确 App Bot 使用同一个 token 作为 API auth 与 IM WebSocket token。

来源: modules/bot_api/register.go#L124-L174

## 4. WebSocket URL 由 server 配置推导，目标是 WuKongIM WS 入口

`DeriveWSURL` 根据 `external.baseURL` 或 `wukongIM.apiURL` 推导 WS 地址：带端口时走直连 `:5200`，域名模式走 `/ws`。

来源: pkg/botutil/ws.go#L11-L45

## 5. Thread 模块向 WuKongIM 提供社区子区 datasource

Thread 模块开启后注册 `IMDatasource`。对 `ChannelTypeCommunityTopic`，它提供 ChannelInfo、Subscribers、Blacklist、Whitelist 等数据给 IM 层。

来源: modules/thread/1module.go#L53-L73
来源: modules/thread/1module.go#L141-L198

Thread 的 ChannelInfo 中会根据父群解散状态向 WuKongIM 返回 `disband=1`，注释强调 fail-closed：查父群状态失败时不能返回未解散态，否则 WuKongIM 可能缓存错误状态。

来源: modules/thread/1module.go#L122-L139

## 6. 群解散采用业务库权威 + WuKongIM 补偿推送

群解散逻辑注释说明，如果上次 MySQL commit 成功但 WuKongIM 推送失败，后续会重试；推送 WuKongIM disband flag 失败时返回错误，允许客户端重试，因为 MySQL 已提交。

来源: modules/group/api.go#L226-L229
来源: modules/group/api.go#L315-L339

`retryWuKongIMDisbandPush` 注释说明 `IMCreateOrUpdateChannelInfo` 是 upsert 幂等操作，因此重复补偿推送安全。

来源: modules/group/api.go#L351-L355

## 7. 未确认 / 待核验

- WuKongIM 客户端握手、订阅、ack、presence 的完整协议在 octo-lib / WuKongIM 侧，本仓库只看到 server 控制面调用点。
- 群成员变更、频道黑白名单、会话同步等更多 IMDatasource 调用点可继续专项展开。
