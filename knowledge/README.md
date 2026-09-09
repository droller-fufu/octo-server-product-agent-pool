# knowledge_base/README.md - octo-server 产品管家知识库组织说明

## 1. 组织原则

本知识库按“考试高频问题域 / 产品机制域”组织，不按源码目录一比一拆分。原因是群内问题通常问的是能力边界、配置决策、鉴权链路、Bot / Agent 行为、存储依赖、发布口径等跨文件主题；按问题域整理，可以更快定位可回答范围和证据来源。

答题时优先使用本目录下的主知识库文件与 `docs/exam-summary.md`。源码仓库 `Mininglamp-OSS/octo-server` 只作为只读证据来源；需求池仓库不能当源码依据。

## 2. 主知识库分块

- `overview.md`：入口索引、双源模型、回答规则、收单规则、cron / 状态输出和安全红线。
- `01-auth-identity.md`：登录、身份、用户 / Bot 身份、token 与身份源边界。
- `02_authorization_model.md`：鉴权链路、AuthMiddleware、Bot API 鉴权顺序、权限边界。
- `03-configuration.md`：启动配置、MySQL / Redis / TLS、对象存储配置、部署配置边界。
- `04_modules.md`：业务模块清单、模块注册、已移除或外置模块、模块启停边界。
- `05_api_error.md`：API 契约、错误码、安全 details、错误响应口径。
- `06_im_control_plane.md`：WuKongIM 控制面、消息发送、IM token、服务端与 IM 内核关系。
- `07_bot_agent.md`：Bot / Agent 边界、Bot register、App Bot、agent runtime、botidentity、octo-fleet 边界。
- `08_storage_dependencies.md`：MySQL、Redis、对象存储、预签名上传、外部依赖边界。
- `09_build_release.md`：构建、Docker、Release、镜像发布与版本口径。

## 3. docs/exam-summary.md 的角色

`docs/exam-summary.md` 是考试收口与统一口径文件：记录已完成范围、已修正 WARN、正式考试答题规则、cron 状态和安全边界。它不是替代 01-09 主知识库，而是在考试场景下给出优先级和统一口径。

当 `docs/exam-summary.md` 与主知识库都覆盖同一问题时，先按 exam-summary 的考试口径约束回答，再回到 01-09 文件取具体证据。

## 4. source-audit 与副本目录

`AINOL_Agent_实操考核_B卷/knowledge/source-audit/` 是源码审计证据底稿或历史审计副本，不是主答题入口。它可以帮助追溯证据，但不能替代当前主知识库和最新源码核验。

`AINOL_Agent_实操考核_B卷/knowledge_base/` 与根目录 `knowledge_base/` 可能存在同名副本；答题默认以当前工作区根目录 `knowledge_base/` 为主，必要时再说明副本差异。

## 5. 为什么不完全按源码目录分块

源码目录适合实现定位，但考试问题常跨越多个包。例如“Bot 注册和 Agent 托管形态”会同时涉及 `modules/bot_api`、`modules/botfather`、SQL migration、用户列表读面和外部 runtime；如果只按源码目录拆，会让一个产品问题散落在多个文件里，增加漏证据风险。

因此本知识库按“可被问到的产品 / 机制主题”聚合证据；每个主题内部再保留相对源码路径与行号，保证既能快速回答，也能回到源码核验。

## 6. 维护与确认人

默认维护者是本 workspace 的 `octo-server 产品管家`。遇到以下不确定项，按类型找对应确认人：

- 生产配置 / 运行态：找运维或部署配置维护人确认 ConfigMap、Helm、环境变量、配置中心和线上状态。
- 下版本计划 / roadmap：找 octo-server PM、PRD owner 或 roadmap 维护人确认。
- OctoPush / 客户端开关：找 OctoPush 客户端负责人或集成负责人确认 UI、默认值和客户端版本行为。
- 知识库分块 / 维护规则：找知识库整理人、考试材料维护人，或由 `octo-server 产品管家` 补充本 README 后再答。

## 7. 不确定性处理模板

考试类回答不得只写“待确认”。必须拆成：

1. 结论。
2. 证据：路径与行号。
3. 已确认范围：当前源码 / 知识库能证明什么。
4. 未确认范围：当前材料不能证明什么。
5. 缺少证据：缺生产配置、roadmap、客户端仓库、运行态数据等哪一类材料。
6. 下一步确认对象：找谁或看哪个系统。
7. 我读了哪些文件。

如果大范围工具输出不可直接引用，应立刻改用窄范围读取；不能把工具输出形态本身作为最终阻断理由。
## 8. GitHub Issue 创建口径

产品管家只把 `droller-fufu/octo-server-product-agent-pool` 作为需求池写入目标。创建 Issue 时，对外只报告结果，不展开工具执行细节：成功给出 `#编号` 和 URL；失败给出可行动原因和需要补齐的信息；处理中只说明正在创建并承诺回贴链接。

创建后必须完成最小验证：仓库名正确、Issue 编号存在、状态为 open、URL 可访问/可见。验证信息可写入 `runtime/status/`，但群内只贴最终结果。

