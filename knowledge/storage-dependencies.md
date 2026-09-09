# 08 存储与外部依赖

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. MySQL 使用 gocraft/dbr，启动时会 Ping

`NewMySQL` 使用 dbr 打开 MySQL，设置连接池参数并执行 `Ping`。如果 migration 开启，则调用 `Migration(sqlDir, session)`。

来源: pkg/db/mysql.go#L15-L39

`Migration` 使用 sql-migrate 的 `migrate.Up` 执行迁移，成功后打印迁移数量。

来源: pkg/db/mysql.go#L41-L50

## 2. SQL migration 会递归查找 `.sql` 并排序

`FileDirMigrationSource.FindMigrations` 递归遍历目录，收集 `.sql` 文件并解析为 migration，最后排序返回。

来源: pkg/db/mysql.go#L58-L112

`internal/modules.go` 也说明 migration 执行顺序由 SQL 文件名时间戳决定，不由 blank import 顺序决定。

来源: internal/modules.go#L1-L18

## 3. Redis 配置来源于统一 DB 配置，并支持 TLS

`BuildOptions` 用 `cfg.DB.RedisAddr`、`cfg.DB.RedisPass` 构造 go-redis options。如果 `cfg.DB.RedisTLS` 为 true，会配置 TLS；可设置 `RedisTLSInsecureSkipVerify` 或加载 `RedisTLSCAFile`。

来源: pkg/redis/options.go#L15-L45

需要裸 Redis client 的限流、OIDC、模块级 NewClient、health 探针等场景，应统一使用 `NewInstrumentedClient`，以获得 TLS 和指标插桩。

来源: pkg/redis/options.go#L58-L70

## 4. 主启动路径依赖 Redis 做限流/缓存/会话能力

主启动中构造全局 per-IP 限流 Redis client，并说明限流状态存在 Redis，多副本共享配额；该 client 用 `NewInstrumentedClient` 构造。

来源: main.go#L289-L305

认证会话 store 也依赖 Redis；`SessionStoreAndClientForContext` 为每个 server context 创建共享 Redis client，并用它构造 `RedisSessionStore`。

来源: pkg/auth/runtime.go#L46-L96

## 5. App Bot auth registry 使用 Redis，但 DB 仍是权威

App Bot auth registry 使用 Redis 共享缓存；token 被 SHA-256 后作为 key，避免明文 token 出现在 Redis key 或运维工具里。

来源: modules/bot_api/registry_redis.go#L17-L23

Redis registry 的注释明确 `app_bot` 表是 source of truth；Redis miss、tombstone 或 Redis error 会回落 DB，不直接放行 stale/revoked spec。

来源: modules/bot_api/registry_redis.go#L54-L80

## 6. 文件上传依赖对象存储预签名直传

`getUploadCredentials` 返回预签名 PUT URL，客户端必须按返回的 `contentType`、`contentDisposition`、`Content-Length` 等头原样上传。注释明确区分 MinIO/COS/S3 SigV4 与 OSS V1 在签名头约束上的差异。

来源: modules/file/api.go#L781-L845

## 7. 一键部署外部依赖不在本仓库

README 与 BUILDING 都说明完整 OOTB 栈包括 server、admin、web、matter、smart-summary、WuKongIM、MySQL、Redis、MinIO、nginx，并由外部部署仓库承载；本仓库旧 compose 栈已退役。

来源: README.md#L60-L66
来源: BUILDING.md#L33-L41

## 8. 未确认 / 待核验

- 对象存储各后端（MinIO/COS/OSS/S3）的具体 service 实现需要继续按 file service 展开。
- MySQL/Redis 的生产地址、密码、TLS CA 等属于部署配置，不应从源码仓库推断。
