# octo-server 源码知识库（source-audit）

本目录用于沉淀 `Mininglamp-OSS/octo-server` 的只读源码调研结论。目标是让 octo-server 产品管家回答产品/代码/配置/部署问题时，能给出可核验证据。

## 使用规则

- 目标源码仓库 `Mininglamp-OSS/octo-server` 只读，不写入。
- 每条关键结论必须带来源：`来源: <相对路径>#L<起>-L<止>`。
- README 与实际目录不一致时，以代码为准。
- 找不到证据时写“未确认”，不编造。
- 配置模板中的说明只能证明“配置模板声明”，不能直接等同于“运行实现已完整支持”。

## 9 域规划

1. `01-auth-identity.md`：认证与身份：token / cookie / WebSocket 握手。
2. `02-authorization-model.md`：鉴权模型：org RBAC、频道 ACL、bot/agent 身份门禁。
3. `03-configuration.md`：配置：`configs/tsdd.yaml` 段落、必填项、启动校验。
4. `04-business-modules.md`：业务模块清单：`modules/` 下模块、职责、是否注册/启用。
5. `05-api-error-contract.md`：API 与错误约定：统一响应体、错误码、HTTP 状态映射。
6. `06-im-control-plane.md`：IM 控制面：server 与 WuKongIM 分工边界。
7. `07-bot-agent.md`：Bot 与 Agent：`app_bot` / `botfather` / `bot_provision` / `botidentity` 关系，agent 会话怎么起。
8. `08-storage-dependencies.md`：存储与外部依赖：表、迁移、Redis、对象存储。
9. `09-build-release.md`：构建与发布：`go build` / `Dockerfile` / `Dockerfile.ghcr` / `octo-deployment`。

## 当前进度

- 已完成样例：`01-auth-identity.md`
- 已完成样例：`03-configuration.md`
- 待补齐：其余 7 域全量调研与核验。

## 待配置项

- cron 扫描主考 uid：待配置，当前不默认张春英就是主考。
- 定时扫描体系：待第二阶段样例验收通过后再进入，不在本次提交中创建。
