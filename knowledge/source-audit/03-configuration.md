# 03. 配置

## 范围

本文件记录 octo-server 配置入口、`configs/tsdd.yaml` 模板段落和启动校验线索。当前样例只覆盖已核验源码证据。

## 已确认结论

### 1. 默认配置文件是 `configs/tsdd.yaml`，可用 `-config` 覆盖

`main.go` 定义 `-config` 参数，默认值为 `configs/tsdd.yaml`；随后调用 `loadConfigFromFile` 读取配置。

来源: `main.go#L138-L145`

`loadConfigFromFile` 使用 viper 读取指定配置文件；读取失败时会 panic。

来源: `main.go#L98-L105`

### 2. 配置支持 `TS_` 环境变量覆盖

读取配置后，启动逻辑设置 viper env prefix 为 `TS`，并用 `strings.NewReplacer(".", "_")` 将点号替换为下划线，然后启用 `AutomaticEnv()`。

来源: `main.go#L141-L145`

### 3. 配置会写入 octo-lib config，并设置 token 过期时间

启动阶段创建 `config.New()`，调用 `cfg.ConfigureWithViper(vp)`，再将 `cfg.Cache.TokenExpire` 设置为 `validateTokenExpireConfig(vp)` 的结果。

来源: `main.go#L152-L156`

### 4. release 模式下禁止配置 smsCode 后门

启动阶段调用 `commonapi.ValidateTestCodeConfig(cfg)`；注释说明 release 模式下禁止配置 smsCode（万能验证码后门），失败会 panic。

来源: `main.go#L158-L161`

### 5. `tsdd.yaml` 基础段声明运行模式、监听地址、appName、rootDir 等配置项

配置模板开头包含 `mode`、`addr`、`grpcAddr`、`appName`、`rootDir`、`messageSaveAcrossDevice`、`welcomeMessage` 等基础配置注释。

来源: `configs/tsdd.yaml#L1-L13`

### 6. `tsdd.yaml` 声明 Webhook HMAC 签名配置

模板说明 `webhookSecretKey` 配置后，入站 webhook 请求必须携带 `X-Signature-256`，格式为 `sha256=<hex(HMAC-SHA256(body, secret_key))>`。

来源: `configs/tsdd.yaml#L15-L19`

### 7. `tsdd.yaml` 声明 WuKongIM 配置段

模板包含 `wukongIM.apiURL` 和 `wukongIM.managerToken`，注释说明它们分别是悟空 IM API 地址和管理者 token。

来源: `configs/tsdd.yaml#L21-L25`

### 8. `tsdd.yaml` 声明数据库与 Redis 配置段

模板包含 MySQL 地址、Redis 地址、Redis 密码、Redis TLS、异步任务 Redis 地址等配置项。

来源: `configs/tsdd.yaml#L26-L35`

### 9. `tsdd.yaml` 声明外网访问配置段

模板包含 `external.ip`、`external.baseURL`、`external.webLoginURL`；注释说明 baseURL 是外网 API 访问地址，webLoginURL 是 web 登录/门户地址，并影响卡片通知 deep-link。

来源: `configs/tsdd.yaml#L36-L44`

### 10. `tsdd.yaml` 配置模板声明对象存储 provider 与 presigned URL 支持矩阵

文件服务段注释列出 `fileService` 可选值和 presigned PUT/GET 矩阵，包括 minio、tencentCOS、aliyunOSS、qiniu、seaweedFS。注意：这只能证明配置模板声明了这些 provider / 矩阵，实际调用链还需继续核验。

来源: `configs/tsdd.yaml#L69-L135`

### 11. `tsdd.yaml` 声明缓存配置段

缓存段包含 `tokenCachePrefix`、`tokenExpire`、`loginDeviceCachePrefix`、`loginDeviceCacheExpire`、`uidTokenCachePrefix`、`friendApplyTokenCachePrefix`、`friendApplyExpire`、`nameCacheExpire` 等。

来源: `configs/tsdd.yaml#L250-L258`

## 初步线索

- 配置字段的完整结构由 `octo-lib/config` 提供，本仓库目前看到的是 `cfg.ConfigureWithViper(vp)` 调用点，不是字段定义本体。
- 运行时 system settings 可能也参与配置，需继续查 `modules/common/db_system_setting.go` 和 manager API。

## 未确认 / 待核验

- 未确认：哪些配置项是启动必填项。
- 未确认：`ConfigureWithViper` 的完整字段映射，需继续核验 `octo-lib` 或依赖源码。
- 未确认：配置是否支持热更新。
- 未确认：对象存储 provider 的实际上传/下载调用链是否与 `tsdd.yaml` 注释完全一致。

## 对产品问答的影响

- 可以说：默认配置入口、环境变量覆盖规则、部分启动校验和 `tsdd.yaml` 模板段落已有源码证据。
- 不能说：所有配置项必填规则、热更新能力、对象存储完整实现已经核验完毕。
