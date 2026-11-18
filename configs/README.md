# 配置模板说明

本目录的模板均使用占位符或环境变量引用，**不含真实令牌**。使用时替换 `<TOKEN>` 或先设置 `ONIONDATA_TOKEN`。

| 文件 | 客户端 | 关键差异 |
| --- | --- | --- |
| `claude-code.mcp.json` | Claude Code | 有 `url` 必须有 `type: "http"`；支持 `${ONIONDATA_TOKEN}` |
| `codex-cli.config.toml` | Codex CLI | TOML 格式；`bearer_token_env_var` 填变量名 |
| `opencode-v2.opencode.json` | OpenCode V2 | 结构为 `mcp.servers.<name>`；必须 `oauth: false`；语法 `{env:ONIONDATA_TOKEN}` |
| `workbuddy.mcp.json` | WorkBuddy | 官方远程类型值 `streamableHttp`，以实际界面为准 |
| `cursor.mcp.json` | Cursor | 支持 `${env:ONIONDATA_TOKEN}` 插值 |
| `gemini-cli.settings.json` | Gemini CLI | Streamable HTTP 用 `httpUrl`，不能用 `url` |
| `vscode.mcp.json` | VS Code + Copilot | 用 `inputs` 让令牌启动时输入，不落盘 |
| `generic.mcp.json` | 通用客户端 | 仅 `url` + `headers`；不支持变量的客户端替换 `<TOKEN>` |

校验方式：配置完成后让客户端列出 MCP 工具，能看到 `search_companies` / `company_records` / `get_usage` 六个工具即表示连接成功。
