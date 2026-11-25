# 洋葱财报 - A股上市公司财报数据查询 MCP 接入示例

## 项目简介

洋葱财报 OnionData-Financial-MCP 提供及时、准确、多口径、多维度的 A 股上市公司财务报表数据，支持单季度同比、单季度环比、逐季度 TTM、年度、最新报告期五大口径近 10 个报告期数据查询。支持上市公司利润表、资产负债表、现金流量表主要财务指标，包括营业收入、净利润、扣非净利润等基础指标，以及毛利率、净利率、扣非净利率、各类周转率等运营指标。

本仓库是洋葱财报 OnionData Financial MCP 服务的公开接入示例：各 AI 客户端配置模板 + 可直接使用的 Skill 示例。

本仓库**只包含配置模板和 Skill 文档**，不含任何服务端代码、真实令牌或账号信息。

## 服务信息

- MCP 端点：`https://api.onionreport.com/mcp`
- 传输：Streamable HTTP（单 POST 端点，无状态）
- 鉴权：两种方式任选——① MCP OAuth 2.1 自动授权（推荐，支持该协议的客户端只需填服务地址）；② API 令牌（Bearer，见下方配置模板）。
- 官方文档：`https://www.onionreport.com/open/`
- 在线说明：`https://www.onionreport.com/open/mcp-intro`
- 数据服务说明：`https://www.onionreport.com/open/data-intro`
- MCP 契约：`https://api.onionreport.com/mcp/docs`

## MCP 工具

注册用户可见 12 个工具（6 个原子工具 + 6 个场景工具）；匿名用户可见 6 个工具（4 个发现工具 + `start_trial` + 受限的 `company_records`）。完整清单以 `tools/list` 返回为准。

### 原子工具

| 工具 | 功能 | 是否计积分 |
| --- | --- | --- |
| `search_companies` | 按证券代码、简称或拼音首字母搜索公司，最多返回 10 条 `{scode, sname}` | 否 |
| `list_report_periods` | 查询公司可用报告期，确定数据时间范围 | 否 |
| `list_indicators` | 获取指标 key 到中文名的映射及单位字典，与公司无关，无需传 `scode` | 否 |
| `get_dimensions` | 获取报表、口径与指标的机器可读字典（中文名 / 单位 / repotype），取数前先确认可用指标 | 否 |
| `company_records` | 统一取数工具：1 个 `scode` 为单公司多期，2–10 个 `scode` 为多公司单期横切 | 是 |
| `get_usage` | 查询当前档位、积分余额与日调用护栏的上限 / 已用 / 剩余 / 重置时间 | 否 |

### 场景化工具（仅注册用户可用）

| 工具 | 功能 | 是否计积分 |
| --- | --- | --- |
| `get_company_trend` | 公司业绩诊断：自动探测业绩预告（有则前置）并给出正式数据双口径事实与信号 | 是（数据点） |
| `get_company_compare` | 2–10 家公司对比：经典五项指标双口径横向对齐 | 是（数据点） |
| `get_company_sort` | 业绩榜单：16 个榜单（同比/环比/毛利率/绝对值/连续四期增长）前十 | 是（每榜单固定 20 积分） |
| `get_quarter_yoy_trend` | 单季度同比趋势 | 是（数据点） |
| `get_quarter_qoq_trend` | 单季度环比趋势 | 是（数据点） |
| `get_ttm_qoq_trend` | TTM 环比趋势（无同比，只看环比） | 是（数据点） |

### 匿名试用

未配置令牌时，可先调 `start_trial` 获取匿名试用 handle，再以受限条件调用 `company_records` 体验取数（有效期 24 小时、最多 10 次成功调用或累计 100 积分）。详见 `skills/oniondata/SKILL.md`。

推荐调用顺序：用户给出公司名或代码 → `search_companies` 确认 `scode` → `list_report_periods` 确定报告期 → `list_indicators` / `get_dimensions` 确认指标 → `company_records` 取数（或直接使用场景工具）→ 用户问余额或遇积分报错时用 `get_usage`。

### `company_records` 关键参数

| 参数 | 取值 | 说明 |
| --- | --- | --- |
| `scode` | 6 位证券代码数组，1–10 个，必填 | 1 个为单公司多期；2–10 个为多公司单期横切 |
| `reponame` | `income` / `balance` / `cashflow` / `predict`，必填 | 多张报表用逗号分隔，最多 4 个，如 `income,balance` |
| `repoterm` | `quarter_t` / `quarter_h` / `ttm` / `year` / `new`，必填 | 决定增长口径，见下表 |
| `repotype` | `basic` / `operation` | 按指标目录细分该报表，可省略 |
| `periods` | 1–10，默认 1 | 仅单公司可用；多公司传会报 `dimension_conflict` |
| `report_date` | 指定单期 | 单公司时与 `periods` 互斥；多公司省略时取最近报告期 |
| `repoitem_key` | 指标 key 数组 | 省略则返回该报表全部指标，建议显式指定以节省积分 |
| `format` | `long` / `wide`，默认 `long` | `long` 一行一指标，`wide` 一行一（公司, 报告期, 报表） |
| `trial_handle` | 匿名试用 handle | 无令牌时须先调 `start_trial` 并传回；已配置令牌时忽略 |

### 五种口径

| `repoterm` | 含义 | 增长口径 |
| --- | --- | --- |
| `quarter_t` | 单季度 | 同比 |
| `quarter_h` | 单季度 | 环比 |
| `ttm` | 逐季度滚动四季合计 | 环比 |
| `year` | 年度 | 同比 |
| `new` | 最新报告期 | 同比 |

`quarter_t` 与 `quarter_h` 的单季数值相同，只是增长口径不同；`ttm` 是滚动四季合计，不能与单季值直接比较。引用增幅时务必注明口径。完整参数、返回值解读与错误码见 `skills/oniondata/SKILL.md`。

## OAuth 自动授权（推荐）

支持 MCP OAuth 2.1 的 AI 客户端（Claude、Cursor、Codex、TRAE、CodeBuddy 等）接入时**只需填一个服务地址** `https://api.onionreport.com/mcp`，无需创建或复制令牌；授权发现、令牌换取与续期全部自动完成。首次使用时在【洋葱财报】微信小程序授权确认一次，之后自动续期。

操作流程：

1. 在支持 OAuth 的 AI 客户端填入 MCP 服务地址 `https://api.onionreport.com/mcp`。
2. AI 客户端直接调用该 MCP，浏览器弹出授权页，显示 8 位确认码与授权操作二维码。
3. 用微信扫描二维码，跳转打开【洋葱财报】微信小程序的「MCP 数据服务授权」页（确认码已自动填好）。
4. 若提示尚未开通程序化服务，先在【洋葱财报】微信小程序「我的 → 程序化服务 → 用量管理」完成开通；已开通可跳过。
5. 在小程序页面核对发起方与确认码，点击「确认授权」；请确认是你本人刚发起的授权，不认识请点「拒绝」。
6. 返回 AI 客户端即可正常使用 MCP 工具，令牌由客户端自动续期。

- 前置条件：已登录【洋葱财报】微信小程序、已绑定手机号、已开通程序化服务（未开通会在确认页拦截）。
- 令牌有效期：确认码 10 分钟一次性；access token 1 小时；refresh token 最长 30 天、连续 7 天未使用失效，每次刷新轮换。
- 无法扫码时可手工进入【洋葱财报】微信小程序「我的 → 程序化服务 → MCP 接入管理 → 授权确认（去确认）」输入 8 位确认码。
- 完整流程见在线说明 `https://www.onionreport.com/open/mcp-intro`。

## 获取令牌（TOKEN 方式）

不支持 OAuth 的客户端或 CLI 工具调用用此方式。

1. 打开洋葱财报小程序。
2. 进入「我的 → 程序化服务 → 用量管理」，开通服务。
3. 进入「访问令牌管理」页创建令牌，令牌以 `oda_` 开头。
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

## 接入场景分类

接入方式按宿主形态分为六类。注意：**AI 终端 / CLI Agent 的“CLI”**指终端里运行的编码 Agent；**平台 Connector** 指平台托管的连接器，两者不是一回事。

| 类别 | 含义 | 客户端 |
| --- | --- | --- |
| A. AI 终端 / CLI Agent | 终端里运行的编码 Agent，直连远程 MCP | Claude Code、Codex CLI、Gemini CLI、OpenCode V2、OpenClaw |
| B. IDE / 编辑器插件 | 图形 IDE 内的 Agent，靠插件或 JSON 配置 | Cursor、TRAE、VS Code + Copilot、Cline、WorkBuddy/CodeBuddy |
| C. 平台 Connector | 平台托管的连接器包，由平台管理鉴权 | 依具体平台而定 |
| D. API 工具 / 调试目录 | 非 Agent 宿主，用于连接测试、能力检查、接口发现 | Postman MCP request + `postman mcp` CLI；Postman API Network |
| E. 通用 MCP 客户端 | 无专属模板的客户端，用通用 `url` + `headers` | generic |
| F. Skill / Agent 说明层 | 非连接通道，跨所有宿主复用的工具说明 | `skills/oniondata/SKILL.md` |

配置机制矩阵：

| 配置机制 | 成员 | 关键字段 |
| --- | --- | --- |
| JSON `mcpServers` | Claude Code、Cursor、TRAE、WorkBuddy、Cline、generic | `url` + `headers`；Claude Code 必须 `type: "http"`，Cline 必须 `type: "streamableHttp"` |
| TOML | Codex CLI | `bearer_token_env_var` |
| 专属 JSON 结构 | OpenCode V2、OpenClaw | OpenCode：`mcp.servers.<name>` + `oauth: false`；OpenClaw：`mcp.servers.<name>` + `transport: "streamable-http"` |
| JSON `servers` + `inputs` | VS Code + Copilot | 令牌启动时输入，不落盘 |
| UI 表单 | Cline Remote Servers、Postman MCP request、WorkBuddy Connector | URL + Transport Type / Auth Type |
| CLI 命令 | OpenClaw、Postman | `openclaw mcp set/probe`；`postman mcp tools list` |
| 无连接配置，纯说明 | Skill | 工具用途、调用顺序、错误码 |

## 配置模板

| 类别 | 客户端 | 模板文件 | 配置文件通常位置 |
| --- | --- | --- | --- |
| A. AI 终端 | Claude Code | `configs/claude-code.mcp.json` | 项目 `.mcp.json` |
| A. AI 终端 | Codex CLI | `configs/codex-cli.config.toml` | `~/.codex/config.toml` |
| A. AI 终端 | Gemini CLI | `configs/gemini-cli.settings.json` | `~/.gemini/settings.json` |
| A. AI 终端 | OpenCode V2 | `configs/opencode-v2.opencode.json` | `opencode.json` |
| A. AI 终端 | OpenClaw | `configs/openclaw.json` | `~/.openclaw/openclaw.json` |
| B. IDE 插件 | Cursor | `configs/cursor.mcp.json` | `~/.cursor/mcp.json` |
| B. IDE 插件 | Cline | `configs/cline.mcp.json` | `~/.cline/data/settings/cline_mcp_settings.json` |
| B. IDE 插件 | VS Code + Copilot（portable） | `configs/vscode-portable.mcp.json` | 项目根 `.mcp.json` |
| B. IDE 插件 | VS Code + Copilot（editor） | `configs/vscode-editor.mcp.json` | `.vscode/mcp.json` |
| C. 平台 Connector | WorkBuddy | `configs/workbuddy.mcp.json` | MCP 服务器配置 |
| D. API 工具 | Postman | `configs/postman.md` | UI MCP request / Postman CLI |
| E. 通用 | 其他客户端 | `configs/generic.mcp.json` | 客户端 MCP 配置 |

各模板的关键差异与校验方式见 `configs/README.md`。TRAE 与 VS Code 的既有说明保留在产品内文档中。

## 新增场景配置说明

### OpenClaw（A 类）

配置位置 `~/.openclaw/openclaw.json`，结构为 `mcp.servers.<name>`，**规范字段是 `transport: "streamable-http"`**（`set` 命令接受 CLI 原生 `type: "http"`，但嵌入式运行时使用 `transport`）。模板见 `configs/openclaw.json`。

```bash
openclaw mcp set oniondata '{"url":"https://api.onionreport.com/mcp","transport":"streamable-http","headers":{"Authorization":"Bearer <TOKEN>"}}'
openclaw mcp doctor oniondata --probe
openclaw mcp probe oniondata --json
```

注意：`status` / `list` / `show` / `set` / 不带 `--probe` 的 `doctor` 都**不会**连接服务器；只有 `probe`、`doctor --probe`、`login` 会真正连接。官方会对明文 bearer 令牌发出警告，模板中的 `<TOKEN>` 请替换为私有配置或环境变量引用，不要提交明文。

官方：`https://docs.openclaw.ai/zh-CN/cli/mcp`

### Cursor（B 类）

配置在 `mcp.json`。远程 Server 用 `url` + `headers`，支持 `${env:ONIONDATA_TOKEN}` 插值。模板见 `configs/cursor.mcp.json`。验证：Customize 页面显示已连接、12 个工具可见，调用 `get_usage` 成功。

官方：`https://cursor.com/docs/mcp`

### Cline（B 类）

配置位置：

- CLI：`~/.cline/data/settings/cline_mcp_settings.json`，可用 `CLINE_MCP_SETTINGS_PATH` 覆盖。
- IDE 扩展：Cline 面板 → MCP Servers → Configure → Configure MCP Servers。

远程 Streamable HTTP **必须显式写 `"type": "streamableHttp"`**，省略会回退到 legacy `sse`。模板见 `configs/cline.mcp.json`。UI 方式：Remote Servers 标签页 → Server Name / URL / Transport Type 选 Streamable HTTP → Add Server。

安全：`autoApprove` 保持空数组，令牌用环境变量承载，工具调用前人工审批。验证：MCP Servers 里工具出现，手动跑一次 `get_usage`。

官方：`https://docs.cline.bot/mcp/connecting-to-a-remote-server`、`https://docs.cline.bot/mcp/configuring-mcp-servers`

### Postman API Network（D 类）

Postman 官方支持作为 MCP 客户端连接远程 streamable HTTP 服务，可用来调试和验证本服务。API Network 是接口目录/发现平台，MCP 条目发布规则以实际界面为准，本模板不假设其发布流程。详见 `configs/postman.md`。

UI：左侧 MCP → HTTP → URL `https://api.onionreport.com/mcp` → Authorization tab 填 Bearer Token → Load Capabilities → Tools tab 查看并运行。

CLI：

```bash
postman mcp tools list --url https://api.onionreport.com/mcp -H "Authorization:Bearer ${ONIONDATA_TOKEN}" --no-login
postman mcp tools call get_usage --url https://api.onionreport.com/mcp -H "Authorization:Bearer ${ONIONDATA_TOKEN}" --args '{}' --json
```

官方：`https://learning.postman.com/docs/use/send-requests/protocols/mcp-requests/create/`、`https://learning.postman.com/latest-v-12/docs/postman-cli/postman-cli-mcp`

### 验证三步

无论哪类客户端，配置完成后都按这三步验证：

1. 列出 MCP 工具，能看到 12 个工具（6 个原子工具 + 6 个场景工具）。
2. 调用 `get_usage`，确认令牌、鉴权与积分链路正常。
3. 调用一次 `search_companies` 或 `company_records`，确认取数正常。

## Skill 使用

`skills/oniondata/SKILL.md` 是一个通用 Skill 示例，说明各工具（6 个原子工具 + 6 个场景工具 + 匿名试用）的用途、调用顺序、参数口径、返回值解读和常见错误。把它复制到你的 AI 工具或 Agent 的 Skill 目录即可按需调整。

## 免责声明

数据来源于公开披露信息，仅供参考，不构成投资建议。使用本服务须遵守日调用护栏与积分规则，不得转售、再分发、公开镜像或批量存储 API 数据。
