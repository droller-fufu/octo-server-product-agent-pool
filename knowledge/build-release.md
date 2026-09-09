# 09 构建与发布

> 引用路径均相对 `octo-server` 仓库根目录。目标源码仓库只读，本文件仅沉淀可核验证据。

## 1. 本地 Quickstart 使用标准 Go 构建并指定配置启动

README 的 Quickstart 给出 `go build -o octo-server .`，随后用 `./octo-server --config ./configs/tsdd.yaml` 启动。

来源: README.md#L45-L58

README 同时说明默认 dev config 期待本地 WuKongIM 实例和 MySQL-compatible database。

来源: README.md#L54-L58

## 2. 本地私有预览构建依赖 sibling repo

BUILDING 说明项目依赖 OCTO 生态的 sibling repositories：octo-lib 与 octo-adapters；pre-release 阶段 `go build ./...` 可能因为 missing go.sum entry 失败。私有预览构建建议 clone sibling repo，并在本地 go.mod 添加 replace。

来源: BUILDING.md#L3-L27

## 3. 完整一键部署不在本仓库

README 与 BUILDING 都指向外部 OOTB deployment；该栈包含 server、admin、web、matter、smart-summary、WuKongIM、MySQL、Redis、MinIO、nginx。本仓库旧 compose 栈已退役。

来源: README.md#L60-L66
来源: BUILDING.md#L33-L41

## 4. Dockerfile 从源码构建静态 Linux 二进制

`Dockerfile` 使用 `golang:1.25` 作为 build stage，先 `go mod download`，再复制源码并执行 `CGO_ENABLED=0 GOOS=linux go build`。

来源: Dockerfile#L11-L32

构建时通过 ldflags 注入 Commit、CommitDate、Version、TreeState；最终 alpine 镜像复制 `/go/release/app`、assets、configs，并以 `/home/app` 为入口。

来源: Dockerfile#L26-L48

## 5. Dockerfile.ghcr 假设二进制已提前构建

`Dockerfile.ghcr` 使用 `debian:bookworm-slim`，安装 ca-certificates 与 tzdata，复制 assets、configs，并把 `linux_${TARGETARCH}` 复制为 `main`，启动命令是 `/app/main`。

来源: Dockerfile.ghcr#L1-L17

## 6. Makefile 包含本地 build、旧 registry push/deploy、env-test 和 i18n lint/extract

`make build` 对应 `docker build -t octo-server .`。Makefile 还包含 push/deploy/deploy-v2 目标、run-dev/stop-dev 退役提示、env-test，以及 i18n-extract/i18n-lint/i18n-extract-check/card-dispatch-lint 等目标。

来源: Makefile#L1-L26
来源: Makefile#L52-L84

BUILDING 明确指出 Makefile 中 `push` / `deploy` / `deploy-v2` 是历史遗留、硬编码团队私有 registry，不是 canonical release surface，不应使用。

来源: BUILDING.md#L57-L62

## 7. 多架构镜像与 release 流程在 GitHub workflow / org 级流程中

BUILDING 说明多架构容器镜像由 `.github/workflows/docker-publish.yml` 在 `v*` Git tag push 时自动发布到 Docker Hub。

来源: BUILDING.md#L49-L55

RELEASING 说明本仓库遵循 org-wide release process；语义化版本 tag 为 `vMAJOR.MINOR.PATCH`，release drafter 自动生成 changelog，发布流程要求选择 main 上 CI green 的 commit、推 tag、再运行 Release Publish workflow。

来源: RELEASING.md#L1-L18
来源: RELEASING.md#L20-L47

## 8. 未确认 / 待核验

- `.github/workflows/docker-publish.yml`、release-drafter、release-publish 的具体 YAML 未在本轮展开。
- 部署 manifests / Helm / compose 属于外部部署仓库，不应编造成当前仓库内能力。
