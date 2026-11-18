# 洋葱财报 - A股上市公司财报数据查询 MCP 接入示例

## 项目简介

洋葱财报 OnionData-MCP 提供及时、准确、多口径、多维度的 A 股上市公司财务报表数据，支持单季度同比、单季度环比、逐季度 TTM、年度、最新报告期五大口径近 10 个报告期数据查询。支持上市公司利润表、资产负债表、现金流量表主要财务指标，包括营业收入、净利润、扣非净利润等基础指标，以及毛利率、净利率、扣非净利率、各类周转率等运营指标。

本仓库是洋葱财报 OnionData MCP 服务的公开接入示例：各 AI 客户端配置模板 + 可直接使用的 Skill 示例。

本仓库**只包含配置模板和 Skill 文档**，不含任何服务端代码、真实令牌或账号信息。

## 服务信息

- MCP 端点：`https://api.onionreport.com/mcp`
- 传输：Streamable HTTP（单 POST 端点，无状态）
- 当前鉴权：API 令牌（Bearer）。**OAuth 暂未开通**。
- 官方文档：`https://www.onionreport.com/open/`
- 在线说明：`https://www.onionreport.com/open/mcp-intro`
- 数据服务说明：`https://www.onionreport.com/open/data-intro`
- MCP 契约：`https://api.onionreport.com/mcp/docs`

## MCP 工具

服务提供 6 个工具，其中只有 `company_records` 计积分，其余免费；积分耗尽后 `get_usage` 仍可用。

| 工具 | 功能 | 是否计积分 |
| --- | --- | --- |
| `search_companies` | 按证券代码、简称或拼音首字母搜索公司，最多返回 10 条 `{scode, sname}` | 否 |
| `list_report_periods` | 查询公司可用报告期，确定数据时间范围 | 否 |
| `list_indicators` | 获取指标 key 到中文名的映射及单位字典，与公司无关，无需传 `scode` | 否 |
| `get_dimensions` | 获取报表、口径与指标的机器可读字典（中文名 / 单位 / repotype），取数前先确认可用指标 | 否 |
| `company_records` | 唯一取数工具，读取利润表、资产负债表、现金流量表与业绩预告的底层数值 | 是 |
| `get_usage` | 查询当前档位、积分余额与日调用护栏的上限 / 已用 / 剩余 / 重置时间 | 否 |

推荐调用顺序：用户给出公司名或代码 → `search_companies` 确认 `scode` → `list_report_periods` 确定报告期 → `list_indicators` / `get_dimensions` 确认指标 → `company_records` 取数 → 用户问余额或遇积分报错时用 `get_usage`。

### `company_records` 关键参数

| 参数 | 取值 | 说明 |
| --- | --- | --- |
| `scode` | 6 位证券代码数组，1–10 个，必填 | 1 个为单公司多期；2–10 个为多公司单期横切 |
| `reponame` | `income` / `balance` / `cashflow` / `predict`，必填 | 每次只能传一个报表 |
| `repoterm` | `quarter_t` / `quarter_h` / `ttm` / `year` / `new`，必填 | 决定增长口径，见下表 |
| `repotype` | `basic` / `operation` | 按指标目录细分该报表，可省略 |
| `periods` | 1–10，默认 1 | 仅单公司可用；多公司传会报 `dimension_conflict` |
| `report_date` | 指定单期 | 单公司时与 `periods` 互斥；多公司省略时取最近报告期 |
| `repoitem_key` | 指标 key 数组 | 省略则返回该报表全部指标，建议显式指定以节省积分 |
| `format` | `long` / `wide`，默认 `long` | `long` 一行一指标，`wide` 一行一（公司, 报告期, 报表） |

### 五种口径

| `repoterm` | 含义 | 增长口径 |
| --- | --- | --- |
| `quarter_t` | 单季度 | 同比 |
| `quarter_h` | 单季度 | 环比 |
| `ttm` | 逐季度滚动四季合计 | 环比 |
| `year` | 年度 | 同比 |
| `new` | 最新报告期 | 同比 |

`quarter_t` 与 `quarter_h` 的单季数值相同，只是增长口径不同；`ttm` 是滚动四季合计，不能与单季值直接比较。引用增幅时务必注明口径。完整参数、返回值解读与错误码见 `skills/oniondata/SKILL.md`。

## 获取令牌

1. 打开洋葱财报小程序。
2. 进入「程序化服务 → 用量管理」，绑定手机号并开通服务。
3. 进入「访问令牌」页创建令牌，令牌以 `oda_` 开头。
4. 令牌仅在创建时展示一次，请妥善保存；泄漏后可立即吊销重建。

> 以上菜单路径来自现有产品文档；如与小程序实际界面不一致，请以实际界面为准。

## 令牌安全

- 不要把真实令牌写进会提交到 Git 的配置文件。
- 支持环境变量的客户端使用变量引用；不支持的客户端用 `<TOKEN>` 占位符，在运行时替换。
- 建议先设置环境变量 `ONIONDATA_TOKEN`：
  - macOS / Linux：`export ONIONDATA_TOKEN=oda_...`
  - Windows PowerShell：`$env:ONIONDATA_TOKEN="oda_..."`
- 令牌泄漏请立即在「访问令牌」页吊销并重建。

## 目录结构

```text
mcp-examples/
├── README.md              本说明
├── configs/               各客户端配置模板
└── skills/oniondata/      通用 Skill 示例
```

## 配置模板

| 客户端 | 模板文件 | 配置文件通常位置 |
| --- | --- | --- |
| Claude Code | `configs/claude-code.mcp.json` | 项目 `.mcp.json` |
| Codex CLI | `configs/codex-cli.config.toml` | `~/.codex/config.toml` |
| OpenCode V2 | `configs/opencode-v2.opencode.json` | `opencode.json` |
| WorkBuddy | `configs/workbuddy.mcp.json` | MCP 服务器配置 |
| Cursor | `configs/cursor.mcp.json` | `~/.cursor/mcp.json` |
| Gemini CLI | `configs/gemini-cli.settings.json` | `~/.gemini/settings.json` |
| VS Code + Copilot | `configs/vscode.mcp.json` | 项目 `.mcp.json` |
| 其他客户端 | `configs/generic.mcp.json` | 客户端 MCP 配置 |

## Skill 使用

`skills/oniondata/SKILL.md` 是一个通用 Skill 示例，说明六个工具的用途、调用顺序、参数口径、返回值解读和常见错误。把它复制到你的 AI 工具或 Agent 的 Skill 目录即可按需调整。

## 免责声明

数据来源于公开披露信息，仅供参考，不构成投资建议。使用本服务须遵守日调用护栏与积分规则，不得转售、再分发、公开镜像或批量存储 API 数据。
