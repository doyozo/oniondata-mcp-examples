# 配置模板说明

本目录的模板均使用占位符或环境变量引用，**不含真实令牌**。使用时替换 `<TOKEN>` 或先设置 `ONIONDATA_TOKEN`。

| 文件 | 客户端 | 关键差异 |
| --- | --- | --- |
| `claude-code.mcp.json` | Claude Code（AI 终端） | 有 `url` 必须有 `type: "http"`；支持 `${ONIONDATA_TOKEN}` |
| `codex-cli.config.toml` | Codex CLI（AI 终端） | TOML 格式；`bearer_token_env_var` 填变量名 |
| `gemini-cli.settings.json` | Gemini CLI（AI 终端） | Streamable HTTP 用 `httpUrl`，不能用 `url` |
| `opencode-v2.opencode.json` | OpenCode V2（AI 终端） | 结构为 `mcp.servers.<name>`；必须 `oauth: false`；语法 `{env:ONIONDATA_TOKEN}` |
| `openclaw.json` | OpenClaw（AI 终端） | 结构为 `mcp.servers.<name>`；规范字段 `transport: "streamable-http"`；探测用 `openclaw mcp probe` |
| `cursor.mcp.json` | Cursor（IDE 插件） | 支持 `${env:ONIONDATA_TOKEN}` 插值 |
| `cline.mcp.json` | Cline（IDE 插件） | 必须显式 `type: "streamableHttp"`，省略会回退 legacy `sse`；`autoApprove` 留空 |
| `workbuddy.mcp.json` | WorkBuddy（平台 Connector） | 官方远程类型值 `streamableHttp`，以实际界面为准 |
| `vscode-portable.mcp.json` | VS Code + Copilot（portable 配置） | 写入项目根 `.mcp.json`；顶层 `mcpServers` + `${ONIONDATA_TOKEN}`，无 `inputs` |
| `vscode-editor.mcp.json` | VS Code + Copilot（editor-managed 配置） | 写入 `.vscode/mcp.json`；顶层 `servers` + `inputs` + `${input:oniondataToken}` |
| `postman.md` | Postman（API 工具） | UI MCP request + `postman mcp` CLI；API Network 发布规则待核实 |
| `generic.mcp.json` | 通用客户端 | 仅 `url` + `headers`；不支持变量的客户端替换 `<TOKEN>` |

校验方式：配置完成后让客户端列出 MCP 工具，能看到 12 个工具（`search_companies` / `company_records` / `get_usage` 等 6 个原子工具 + 6 个场景工具）即表示连接成功。
