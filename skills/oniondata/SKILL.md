---
name: 洋葱财报OnionData-MCP
description: 洋葱财报OnionData-MCP提供及时、准确、多口径、多维度的A股上市公司财务报表数据，支持单季度同比、单季度环比、逐季度TTM、年度、最新报告期五大口径近10个报告期数据查询。支持上市公司利润表、资产负债表、现金流量表主要财务指标，包括营业收入、净利润、扣非净利润等基础指标，以及毛利率、净利率、扣非净利率、各类周转率等运营指标。
---

# 洋葱财报 Skill

通过洋葱财报 MCP 服务（`https://api.onionreport.com/mcp`，Streamable HTTP）读取 A 股上市公司最新财务报告期各财务指标多口径数据。

## 可用工具

| 工具 | 用途 | 是否计积分 |
| --- | --- | --- |
| `search_companies` | 按证券代码、简称或拼音首字母搜索公司，最多返回 10 条 `{scode, sname}` | 否 |
| `list_report_periods` | 查询公司可用报告期，确定数据时间范围 | 否 |
| `list_indicators` | 获取指标 key 到中文名的映射及单位字典，与公司无关，不需传 `scode` | 否 |
| `get_dimensions` | 获取报表、口径与指标的机器可读字典（中文名 / 单位 / repotype），取数前先确认可用指标 | 否 |
| `company_records` | 唯一取数工具，读取利润表、资产负债表、现金流量表与业绩预告的底层数值 | 是 |
| `get_usage` | 查询当前档位、积分余额与日调用护栏的上限 / 已用 / 剩余 / 重置时间 | 否 |

只有 `company_records` 计积分；其余工具免费。积分耗尽后 `get_usage` 仍可用。

## 推荐调用顺序

1. 用户给出公司名或代码 → `search_companies` 确认 `scode`。
2. 需要确定有哪些报告期 → `list_report_periods`（`scode`）。
3. 不确定指标字段 → `list_indicators` 或 `get_dimensions` 取指标 key 与单位。
4. 实际取数 → `company_records`。
5. 用户问余额或遇到积分报错 → `get_usage`。

## `company_records` 关键参数

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
| `notice_date_from` / `notice_date_to` | 公告日区间 | 可省略 |

### 五种口径

| `repoterm` | 含义 | 增长口径 |
| --- | --- | --- |
| `quarter_t` | 单季度 | 同比 |
| `quarter_h` | 单季度 | 环比 |
| `ttm` | 逐季度滚动四季合计 | 环比 |
| `year` | 年度 | 同比 |
| `new` | 最新报告期 | 同比 |

`quarter_t` 与 `quarter_h` 的单季数值相同，只是增长口径不同；`ttm` 是滚动四季合计，不能与单季值直接比较。引用增幅时务必注明口径。

## 返回值解读

- `data` 为记录行，含 `scode` / `sname` / `reponame` / `repoterm` / `report_date` / `repoitem_key` / `repoitem_value` 等字段。
- `page` 含 `companies` / `periods` / `returned` / `truncated`；一次返回，`has_more` 恒为 `false`，`next_cursor` 恒为 `null`。
- 数值为原始值（如金额单位为元），**不做「万 / 亿」格式化**，需要时由你换算。
- 增幅列以百分点表示：`12.3` 表示 12.3%，不要再加 `%` 或除以 100。
- `units` 给出指标单位，`meta.report_dates` 给出本次结果的报告期降序列表。
- 无数据时返回空 `data` 列表，不是错误，也不消耗积分。

行数上界：`long` 250 行、`wide` 10 行。多公司多期趋势需按报告期逐次调用。

## 认证说明

本服务支持两种鉴权方式，任选其一：

- **OAuth 自动授权（推荐）**：支持 MCP OAuth 2.1 的客户端只需填服务地址 `https://api.onionreport.com/mcp`，无需创建令牌。首次调用时浏览器弹出授权页，用微信扫描二维码进入【洋葱财报】微信小程序「MCP 数据服务授权」页，确认后客户端自动换取令牌并续期；需已登录小程序、已绑定手机号、已开通程序化服务（未开通会在确认页引导先开通）。确认码 10 分钟一次性，access token 1 小时，refresh token 最长 30 天且每次刷新轮换。
- **API 令牌（Bearer）**：不支持 OAuth 的客户端或脚本调用，在客户端以 `Authorization: Bearer <令牌>` 请求头调用。令牌获取：【洋葱财报】微信小程序「程序化服务 → 用量管理」绑定手机号并开通服务，再进入「访问令牌」页创建。令牌仅在创建时展示一次，请妥善保存；泄漏后可立即吊销并重建。令牌有效期由创建时选择的有效期决定，过期后需创建新令牌并更新配置。

## 常见错误

| 错误 | 含义与处理 |
| --- | --- |
| `not_activated` | 未开通 API 服务：先在小程序绑定手机号并点击「开通服务」 |
| `quota_exceeded` | 积分不足：等积分发放或升级档位 |
| `daily_limit` | 日调用护栏用尽：次日北京时间 0 点恢复，不扣积分 |
| `result_too_large` | 单次所需积分超过档位上限：缩小 `repoitem_key` 或 `periods` 范围 |
| `rate_limited` / `server_busy` / `shed_build` | 触发限流或容量保护：按提示退避后重试 |
| `query_timeout` | 单查询超过 3 秒：缩小期数或行数 |
| 401 / 403 | 令牌无效、已吊销、过期或受众不符：OAuth 客户端重新发起授权；令牌客户端更新配置中的 API 令牌 |

## 注意事项

- 数据每日增量更新，接口结果可能来自缓存，以返回的报告期与数值为准。
- 数据来源于公开披露信息，仅供参考，不构成投资建议。
- 仅在日调用护栏与积分余额内使用；不得转售、再分发、公开镜像或批量存储 API 数据。
