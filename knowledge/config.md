# 03 配置

> 考前补齐文件。内容来自已整理知识库 `08_storage_dependencies.md`、`09_build_release.md`、`overview.md` 的沉淀口径；本次补齐未读取、未 grep `Mininglamp-OSS/octo-server` 目标源码。引用路径仍沿用已整理知识库中的源码引用路径，均相对 `octo-server` 仓库根目录。

## 1. 本地启动配置入口

Quickstart 使用 `go build -o octo-server .` 构建，再通过 `./octo-server --config ./configs/tsdd.yaml` 指定配置文件启动。默认 dev config 期待本地 WuKongIM 实例和 MySQL-compatible database。

来源: README.md#L45-L58
来源: README.md#L54-L58

## 2. MySQL 配置与迁移行为

MySQL 使用 gocraft/dbr 打开连接，设置连接池参数并执行 `Ping`。如果 migration 开启，则调用 SQL migration。

来源: pkg/db/mysql.go#L15-L39
来源: pkg/db/mysql.go#L41-L50

SQL migration 会递归查找 `.sql` 文件并排序；migration 执行顺序由 SQL 文件名时间戳决定，不由 blank import 顺序决定。

来源: pkg/db/mysql.go#L58-L112
来源: internal/modules.go#L1-L18

## 3. Redis 配置与 TLS

Redis options 来自统一 DB 配置：`RedisAddr`、`RedisPass`、`RedisTLS`、`RedisTLSInsecureSkipVerify`、`RedisTLSCAFile`。需要裸 Redis client 的限流、OIDC、模块级 client、health 探针等场景，应统一使用 instrumented client，以获得 TLS 和指标插桩。

来源: pkg/redis/options.go#L15-L45
来源: pkg/redis/options.go#L58-L70

## 4. 运行期依赖 Redis 的能力

主启动路径构造全局 per-IP 限流 Redis client，并说明限流状态存在 Redis，多副本共享配额。认证会话 store 也依赖 Redis client，并用它构造 `RedisSessionStore`。

来源: main.go#L289-L305
来源: pkg/auth/runtime.go#L46-L96

## 5. Bot / App Bot 相关配置边界

App Bot auth registry 使用 Redis 共享缓存；token 被 SHA-256 后作为 key，避免明文 token 出现在 Redis key 或运维工具里。DB 仍是 source of truth；Redis miss、tombstone 或 Redis error 会回落 DB，不直接放行 stale/revoked spec。

来源: modules/bot_api/registry_redis.go#L17-L23
来源: modules/bot_api/registry_redis.go#L54-L80

## 6. 文件上传与对象存储配置边界

文件上传依赖对象存储预签名直传。客户端必须按返回的 `contentType`、`contentDisposition`、`Content-Length` 等头原样上传；MinIO/COS/S3 SigV4 与 OSS V1 在签名头约束上有差异。

来源: modules/file/api.go#L781-L845

## 7. 部署配置与外部依赖不应从源码臆测

完整 OOTB 栈包括 server、admin、web、matter、smart-summary、WuKongIM、MySQL、Redis、MinIO、nginx，并由外部部署仓库承载；本仓库旧 compose 栈已退役。

来源: README.md#L60-L66
来源: BUILDING.md#L33-L41

## 8. Docker / Release 配置边界

`Dockerfile` 从源码构建静态 Linux 二进制，并通过 ldflags 注入 Commit、CommitDate、Version、TreeState。`Dockerfile.ghcr` 假设二进制已提前构建。多架构容器镜像由 GitHub workflow 在 `v*` tag push 时发布；org-wide release 使用语义化版本 tag。

来源: Dockerfile#L11-L48
来源: Dockerfile.ghcr#L1-L17
来源: BUILDING.md#L49-L55
来源: RELEASING.md#L1-L18
来源: RELEASING.md#L20-L47

## 9. 考试答题边界

可直接回答：配置文件启动入口、MySQL/Redis/TLS、migration 排序、Redis 限流/会话、对象存储预签名、Dockerfile / Dockerfile.ghcr 差异、OOTB 外部依赖边界。

仍应保守回答：生产环境具体地址、密码、TLS CA、部署 manifests / Helm / compose、外部部署仓库细节。本知识库未覆盖时，回答“未确认 / 待核验”，不要编造。
