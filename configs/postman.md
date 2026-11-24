# Postman MCP 接入说明

Postman 可作为 MCP 客户端连接并调试本服务。官方文档确认支持 STDIO 与 streamable HTTP，本服务使用后者。

- MCP request 文档：`https://learning.postman.com/docs/use/send-requests/protocols/mcp-requests/create/`
- Postman CLI 文档：`https://learning.postman.com/latest-v-12/docs/postman-cli/postman-cli-mcp`
- API Network：`https://www.postman.com/api-network/`（目录/发现平台；MCP 条目发布规则请以实际界面为准，本模板不假设）

## 方式 A：Postman UI

1. 左侧栏点击 **MCP**，打开新的 MCP request tab。
2. 通信方式选择 **HTTP**（streamable HTTP）。Postman 会自动检测 stateless / legacy 协议。
3. Server URL 填：`https://api.onionreport.com/mcp`
4. 打开 **Authorization** tab，Auth Type 选择 Bearer Token，填入 `oda_` 开头的令牌。
5. 点击 **Load Capabilities**，确认连接并加载能力。
6. 在 **Tools** tab 查看工具列表；选择 `get_usage`，参数填 `{}`，点击 **Run** 查看响应。

> 不要把真实令牌写进会提交到 Git 的 collection。用 Postman 环境变量（如 `{{oniondata_token}}`）承载令牌。

## 方式 B：Postman CLI

先设置令牌环境变量（示例）：

```powershell
$env:ONIONDATA_TOKEN="oda_你的令牌"
```

列工具：

```bash
postman mcp tools list --url https://api.onionreport.com/mcp -H "Authorization:Bearer ${ONIONDATA_TOKEN}" --no-login
```

调用 `get_usage`：

```bash
postman mcp tools call get_usage --url https://api.onionreport.com/mcp -H "Authorization:Bearer ${ONIONDATA_TOKEN}" --args '{}' --json
```

检查能力：

```bash
postman mcp inspect --url https://api.onionreport.com/mcp -H "Authorization:Bearer ${ONIONDATA_TOKEN}" --json
```

参数说明：

- `-H "key:value"`：添加 HTTP 头；`Authorization` 头会替换保存的 MCP request 中继承的鉴权。
- `--no-login`：禁止触发 OAuth 登录。已配置 Bearer 令牌时可显式加上，避免误触发 OAuth 流程。
- `--protocol-version auto`（默认）：先尝试 `2026-07-28`，失败回退 legacy，与本服务的 dual-era 版本声明匹配。
- 退出码 `3` 表示服务器要求凭据（401/403），优先检查令牌是否有效、受众是否为 `mcp`。

## 验证标准

- `Tools` tab 或 `postman mcp tools list` 能看到 12 个工具：6 个原子工具（`search_companies` / `list_report_periods` / `list_indicators` / `get_dimensions` / `company_records` / `get_usage`）+ 6 个场景工具（`get_company_trend` / `get_company_compare` / `get_company_sort` / `get_quarter_yoy_trend` / `get_quarter_qoq_trend` / `get_ttm_qoq_trend`）。
- `get_usage` 返回档位、积分余额与日调用护栏信息。
