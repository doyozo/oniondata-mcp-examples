---
name: 洋葱财报OnionData-MCP
description: 洋葱财报OnionData-MCP提供及时、准确、多口径、多维度的A股上市公司财务报表数据，支持单季度同比、单季度环比、逐季度TTM、年度、最新报告期五大口径近10个报告期数据查询。支持上市公司利润表、资产负债表、现金流量表主要财务指标，包括营业收入、净利润、扣非净利润等基础指标，以及毛利率、净利率、扣非净利率、各类周转率等运营指标。
---

# 洋葱财报 Skill

通过洋葱财报 MCP 服务（`https://api.onionreport.com/mcp`，Streamable HTTP）读取 A 股上市公司最新财务报告期各财务指标多口径数据。

## 可用工具

注册用户可见 12 个工具（6 个原子工具 + 6 个场景工具）；匿名用户可见 6 个工具（4 个发现工具 + `start_trial` + 受限的 `company_records`）。完整清单以 `tools/list` 返回为准。

### 原子工具

| 工具 | 用途 | 是否计积分 |
| --- | --- | --- |
| `search_companies` | 按证券代码、简称或拼音首字母搜索公司，最多返回 10 条 `{scode, sname}` | 否 |
| `list_report_periods` | 查询公司可用报告期，确定数据时间范围 | 否 |
| `list_indicators` | 获取指标 key 到中文名的映射及单位字典，与公司无关，不需传 `scode` | 否 |
| `get_dimensions` | 获取报表、口径与指标的机器可读字典（中文名 / 单位 / repotype），取数前先确认可用指标 | 否 |
| `company_records` | 统一取数工具：1 个 `scode` 为单公司多期，2–10 个 `scode` 为多公司单期横切 | 是 |
| `get_usage` | 查询当前档位、积分余额与日调用护栏的上限 / 已用 / 剩余 / 重置时间 | 否 |

### 场景化编排工具（仅注册用户可用）

| 工具 | 用途 | 是否计积分 |
| --- | --- | --- |
| `get_company_trend` | 公司业绩诊断：自动探测业绩预告（有则前置）并给出正式数据双口径事实与信号 | 是（数据点） |
| `get_company_compare` | 2–10 家公司对比：经典五项指标双口径横向对齐 | 是（数据点） |
| `get_company_sort` | 业绩榜单：16 个榜单（同比/环比/毛利率/绝对值/连续四期增长）前十 | 是（每榜单固定 20 积分） |
| `get_quarter_yoy_trend` | 单季度同比趋势（`quarter_t`，growth_tz 同比增幅） | 是（数据点） |
| `get_quarter_qoq_trend` | 单季度环比趋势（`quarter_h`，growth_hz 环比增幅） | 是（数据点） |
| `get_ttm_qoq_trend` | TTM 环比趋势（`ttm`，growth_hz；无同比数据） | 是（数据点） |

只有取数类工具计积分；搜索/字典/额度工具免费。积分耗尽后 `get_usage` 仍可用。

### 匿名试用（`start_trial`）

未配置令牌时，先调 `start_trial` 获取匿名试用 handle（有效期 24 小时，最多 10 次成功调用或累计 100 积分，先到为准；每 IP 每 24 小时限领 1 个），再把返回的 handle 作为 `company_records` 的 `trial_handle` 参数传回。匿名取数仅限**单公司、单指标、单口径、`format=long`、最多 2 个报告期**；用尽或到期后完成 OAuth 授权即可转注册额度。

## 场景路由（优先用场景工具）

- 用户问「某公司的业绩怎么样 / 业绩预计怎么样」→ `get_company_trend`（预告自动前置）。
- 问「某公司单季度同比趋势」→ `get_quarter_yoy_trend`；「单季度环比趋势」→ `get_quarter_qoq_trend`；「TTM 趋势」→ `get_ttm_qoq_trend`。
- 问「A 和 B 哪个好 / 对比」→ `get_company_compare`。
- 问「增长最快 / 最赚钱 / 毛利率最高 / 连续增长的公司」→ `get_company_sort`（一次一个榜单；多个指标分别调用）。
- 需要自定义报表/指标/公告日区间等长尾查询 → `company_records`（原子工具）。
- 公司名匹配多家时场景工具会返回候选，改用 6 位证券代码重试。

## 推荐调用顺序（原子工具）

1. 用户给出公司名或代码 → `search_companies` 确认 `scode`。
2. 需要确定有哪些报告期 → `list_report_periods`（`scode`）。
3. 不确定指标字段 → `list_indicators` 或 `get_dimensions` 取指标 key 与单位。
4. 实际取数 → `company_records`。
5. 用户问余额或遇到积分报错 → `get_usage`。

## `company_records` 关键参数

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
| `notice_date_from` / `notice_date_to` | 公告日区间 | 可省略 |
| `trial_handle` | 匿名试用 handle | 无令牌时须先调 `start_trial` 并传回；已配置令牌时忽略 |

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

- 成功响应统一位于 `structuredContent`：`contract_version=2` 与 `resultType=complete` 标识成功契约，顶层固定字段为 `tool` / `kind` / `subject` / `basis` / `period` / `data` / `meta`；计费工具另有 `quota`。
- 场景工具的 `data` 按类型区分：`trend` 为 `data.facts[].points[]` 各期数值与增幅 + `data.signals[].signal` 增长/下滑/持平判断；`company_trend` 另有 `data.official_facts`（正式）与 `data.predict_facts`（预告）；`compare` 为 `data.grouped_facts[].facts[]` 按公司分组；`ranking` 为 `data.items[]` + `data.pagination`；`records` 为 `data.rows` + `data.columns` + `data.quality`。
- `company_records` 的原子行位于 `data.rows`，列定义位于 `data.columns`，分页与截断状态位于 `data.pagination`，数据新鲜度位于 `data.quality`。
- `period` 是结构化对象：`type` 表示 single/list/multi，`values` 是本次报告期列表，`value` 是单值报告期。
- `unit` 是结构化对象：`raw` 用于展示，`dimension` 表示币种/百分比/比率等维度，`scale=hundred_million` 表示亿元。不要再从中文单位字符串中猜测数量级。
- 数值为原始值（金额单位为元）；增幅以百分点表示（`12.3` 即 12.3%），不要再加 `%` 或除以 100。
- `predict_facts` 或 `is_final=false` 的 fact 必须标注「业绩预告，非最终财报，以正式报告为准」。
- 无数据时场景工具返回 `isError=true` 与 `no_data`；`company_records` 返回空 `data.rows`，不是错误，也不消耗积分。
- 错误结果 `resultType=error`，`error.code` 给出稳定错误码，`next_actions` 给出下一步建议。

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
| `trial_handle_required` / `trial_handle_invalid` / `trial_handle_expired` | 匿名试用 handle 缺失 / 无效 / 过期：重新调用 `start_trial` |
| `trial_exhausted` / `trial_budget_exceeded` | 匿名试用次数或积分用尽：完成 OAuth 授权转注册额度 |
| `trial_scope_limited` | 匿名试用超出场景限制：单公司、单指标、单口径、`format=long`、最多 2 期 |
| `rate_limited` / `server_busy` / `shed_build` | 触发限流或容量保护：按提示退避后重试 |
| `query_timeout` | 单查询超过 3 秒：缩小期数或行数 |
| 401 / 403 | 令牌无效、已吊销、过期或受众不符：OAuth 客户端重新发起授权；令牌客户端更新配置中的 API 令牌 |

## 注意事项

- 数据每日增量更新，接口结果可能来自缓存，以返回的报告期与数值为准。
- 数据来源于公开披露信息，仅供参考，不构成投资建议。
- 仅在日调用护栏与积分余额内使用；不得转售、再分发、公开镜像或批量存储 API 数据。
