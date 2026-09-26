# FlowERP Copilot 指南

本仓库是独立客户 ERP 项目，仅维护业务模型、服务、HTTP API、客户页面和业务检查。个人研发工作台、课程讲义和交付治理内容位于独立的 CodexFDE 仓库，不应复制到本仓库。

本项目默认以中文进行沟通、文档和代码注释。后续所有说明、评审、注释与变更说明应保持中文，除非用户显式要求其他语言。

## 仓库边界

- `flowerp/`：业务服务、SQLite 持久化层和 `/api/v1` HTTP API。
- `web/`：客户页面，业务事实必须来自 `/api/v1`，不能写死演示数据。
- `tests/` 和 `eval/`：业务与接口检查，是真正的行为验收来源。
- 使用本仓库 `.venv` 和可编辑安装，不引入第三方运行时依赖。
- 运行数据库、密钥、日志、检查报告等本地运行态数据不能入库。

## 构建、测试与校验命令

在仓库根目录执行，使用本仓库的 Python 环境。

### 启动本地应用

```powershell
.venv\Scripts\python.exe -m pip install -e .
.venv\Scripts\python.exe -X utf8 -m flowerp serve --port 8000 --runtime-dir .runtime
```

### 全量检查

```powershell
.venv\Scripts\python.exe -X utf8 -m unittest discover -s tests -v
.venv\Scripts\python.exe -X utf8 -m eval.harness --suite blocking
node --test tests/flowerp_boot.test.cjs tests/flowerp_inventory_filter.test.cjs tests/flowerp_purchase_recovery.test.cjs
```

### 单项/单文件测试

针对特定模块或用例进行最小验证：

```powershell
.venv\Scripts\python.exe -X utf8 -m unittest tests.test_http_api -v
.venv\Scripts\python.exe -X utf8 -m unittest tests.test_http_api.HTTPAPITests.test_bootstrap_login_and_authenticated_me -v
.venv\Scripts\python.exe -X utf8 -m unittest tests.test_flowerp -v
```

对前端/HTTP API 相关改动，建议在提交前至少运行：

```powershell
.venv\Scripts\python.exe -X utf8 -m unittest tests.test_http_api -v
```

`pyproject.toml` 中没有额外 linter 配置，当前主要依赖 unittest 和 eval harness 进行验证。

## 高层架构

### 服务与持久化结构

- `flowerp/store.py`：创建 SQLite 数据库、应用 schema migration，并提供事务边界。
- `flowerp/schema_v2.py`、`flowerp/schema_extensions.py`：扩展 ERP 领域 schema。
- `flowerp/service.py`：底层 ERP 业务服务，覆盖商品、客户/供应商、库存、订单与采购请求。
- `flowerp/api.py`：主 API 路由，负责鉴权、限流、CSRF 校验、路由分发和业务异常转换。
- `flowerp/server.py`：组装应用、运行时状态和 HTTP 服务，提供静态前端文件并将 `/api/v1` 请求转发到 API 层。
- `flowerp/config.py`：加载环境变量与生产配置保护。
- `flowerp/operations.py`：运行时协调、备份/健康检查、维护模式等。

### 领域服务分层

关键服务模块按业务关注点拆分：

- `flowerp/master_data.py`：主数据、仓库、仓位、商品目录和商品元数据。
- `flowerp/inventory.py`：库存变动、预占、可用性和库存控制。
- `flowerp/sales.py`：销售订单和订单履约流程。
- `flowerp/purchasing.py`：采购申请、审批和入库流程。
- `flowerp/finance.py`、`flowerp/accounting.py`、`flowerp/cash_management.py`：财务、会计和现金流。
- `flowerp/reconciliation.py`、`flowerp/reports.py`、`flowerp/alerts.py`：对账、报表和运营告警。
- `flowerp/identity.py`：认证、授权、会话和初始化。
- `flowerp/channels.py`：渠道订单接收和回调处理。

核心架构原则：业务规则放在服务层，用 SQLite 事务、审计日志和授权检查共同约束；HTTP 层保持薄封装，不替代域内校验。

### 前端边界

`web/` 为静态客户前端，页面通过 `/api/v1` 拉取业务事实，不能写死演示数据或存储服务端权限信息。

## 关键约定

- 变更必须保持在 ERP 项目范围内；课程材料、工作台 API 和训练内容应保持在独立仓库。
- 严格保留以下业务不变量：
  - 可用库存不能为负；
  - 预占必须原子化；
  - 同一入库幂等键只生效一次；
  - 订单状态遵守状态机；
  - 取消时释放预占；
  - 采购必须人工审批后再入库。
- 修改业务规则时，需同时覆盖正常路径和失败路径的测试。
- 保留幂等、鉴权、审计和人工审批职责分离。
- 本地运行态数据放在 `.runtime`，不要提交数据库、日志、密钥或检查报告。
- API 变更优先复用现有领域异常，如 `ValidationError`、`Conflict`、`NotFound`、`PermissionDenied`，不要引入临时绕过逻辑。
- 前端改动遵循 `web/AGENTS.md`：
  - `index.html` 只提供可访问的页面骨架；
  - `app.js` 不保存密码、平台密钥或服务端权限；
  - 新页面需要加载态、空状态、失败提示和高风险操作确认；
  - 不引入 CDN 或需要联网的前端依赖，保持离线执行和容器冷启动。

## 相关仓库指引

- `AGENTS.md` 是仓库顶层指引，视为项目策略文件。
- `web/AGENTS.md` 是前端专用约束说明。
- `README.md` 是启动、检查和项目边界的权威文档。

## MCP 服务器

仅在对当前仓库工作有明确帮助时配置 MCP 服务器。例如，如果需要扩展浏览器端到端测试或前端自动化，可考虑 Playwright；否则不额外增加 MCP 配置。

## 结论

本文件已按 FlowERP 实际结构与约定整理，后续 Copilot 会在中文语境下工作，并优先遵循仓库已有的 `AGENTS.md`、`web/AGENTS.md` 和 `README.md` 约束。

如果你愿意，我可以继续为这个仓库补充更细的模块级指引，例如库存、采购、财务或前端交互区域的专项说明。