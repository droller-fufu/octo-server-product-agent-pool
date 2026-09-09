# octo-server 产品管家需求池

本仓库用于 `octo-server` 产品管家的考试需求池与知识库沉淀。

## 双源模型

- `source_repo = Mininglamp-OSS/octo-server`：仅用于考前源码核验 / 知识库依据 / 产品理解；正式考试期间不现场 grep / 搜索 / 读取目标源码。
- `issue_pool_repo = droller-fufu/octo-server-product-agent-pool`：用于 Issue 收单、label、What-only PRD、review 记录与 cron 扫描。

## 目录

- `docs/knowledge_base/overview.md`：知识库总览索引
- `docs/knowledge_base/01-auth-identity.md`：认证与身份
- `docs/knowledge_base/02_authorization_model.md`：鉴权模型
- `docs/knowledge_base/03-configuration.md`：配置
- `docs/knowledge_base/04_modules.md`：业务模块清单
- `docs/knowledge_base/05_api_error.md`：API 与错误约定
- `docs/knowledge_base/06_im_control_plane.md`：IM 控制面
- `docs/knowledge_base/07_bot_agent.md`：Bot 与 Agent
- `docs/knowledge_base/08_storage_dependencies.md`：存储与外部依赖
- `docs/knowledge_base/09_build_release.md`：构建与发布
- `PRD_TEMPLATE.md`：What-only PRD 模板
- `.github/ISSUE_TEMPLATE/`：需求 / bug / 问题收单模板
- `runtime/scan_state.json`：cron 扫描状态占位文件

## 答题纪律

正式考试回答只使用已整理知识库 / summary / issue pool 信息；不能现场 grep `Mininglamp-OSS/octo-server`。不确定时说“未确认 / 待核验”，禁止编造路径、行号、需求、实现细节或凭据。
