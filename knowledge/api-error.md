# 05 API 与错误约定

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. 本仓库使用 localized error facade 统一渲染业务错误

`httperr.ResponseErrorL` 是业务侧本地化错误门面：它校验错误码、分离 params/details，保留 legacy HTTP/body status=400 兼容路径，并委托 wkhttp ErrorRenderer 输出翻译后的 envelope。

来源: pkg/httperr/respond.go#L13-L24
来源: pkg/httperr/respond.go#L57-L81

`ResponseErrorLWithStatus` 用于明确不依赖 fixed-400 的新端点；其 body envelope 与 `ResponseErrorL` 一致，只是 transport status 使用错误码的 canonical HTTPStatus。

来源: pkg/httperr/respond.go#L26-L49

## 2. 兼容期默认 wire status 是 400，语义 HTTP 状态在 body 内

`respondL` 中默认 `transportStatus := http.StatusBadRequest`；只有 `useSemanticStatus` 为 true 时才把 transport status 改成 registered `HTTPStatus`。同时 `SemanticStatus` 会写入 ErrorSpec，最终出现在 `error.http_status`。

来源: pkg/httperr/respond.go#L57-L81
来源: pkg/i18n/renderer.go#L27-L65

## 3. 错误响应 envelope 同时兼容新旧客户端

ErrorRenderer 输出固定结构：新客户端读 `error.code/message/details/http_status`；旧客户端仍可读 `msg` 和 `status`。响应头会设置 `Content-Language`，并对 `Accept-Language`、Octo-Lang、Cookie 等增加 Vary。

来源: pkg/i18n/renderer.go#L27-L65

## 4. 错误码注册表是 user-visible 错误的权威入口

`pkg/i18n/codes` 包注释说明所有 user-visible 错误码都必须通过 Register 登记；Register 会校验 ID 命名空间、DefaultMessage 和 HTTPStatus，重复 ID 会 panic。

来源: pkg/i18n/codes/registry.go#L1-L18
来源: pkg/i18n/codes/registry.go#L74-L118

`Code` 字段包括 ID、HTTPStatus、DefaultMessage、DefaultMessages、SafeDetailKeys、Internal。SafeDetailKeys 用于白名单过滤 details，Internal 用于 5xx 类错误隐藏内部 message。

来源: pkg/i18n/codes/registry.go#L45-L67

## 5. shared 错误码覆盖鉴权、限流、参数、not found 和 internal

shared code 注册包含 auth required/token missing/token invalid/token expired/forbidden、rate limited、param invalid、not found、internal 等。限流只透传 `retry_after`，参数错误只透传 `field`。

来源: pkg/i18n/codes/shared.go#L21-L75
来源: pkg/i18n/codes/shared.go#L77-L106

## 6. 模块级错误码示例：common 模块

`pkg/errcode/common.go` 使用 `register(codes.Code{...})` 定义模块错误码，ID 采用 `err.server.<module>.<reason>`，并为可展示的 details 设置 SafeDetailKeys。

来源: pkg/errcode/common.go#L9-L22
来源: pkg/errcode/common.go#L44-L78

## 7. 直接 raw error response 仍有基线，但新迁移应走 facade

`tools/lint-direct-error-response/baseline.txt` 说明该 lint 统计 `c.AbortWithStatusJSON` / `c.AbortWithStatus`，文件超过基线或新增直接错误响应会失败。部分协议端点如 OIDC/browser redirect、GitHub webhook 签名错误被标注为 EXEMPT。

来源: tools/lint-direct-error-response/baseline.txt#L1-L31

## 8. 未确认 / 待核验

- 成功响应 envelope 的完整规范需要继续追 octo-lib `wkhttp.Context.Response`，不在本仓库单独声明。
- 所有模块错误码分类表可由 `pkg/errcode/*.go` 进一步生成，本轮只确认注册机制和代表性 shared/common 码。
