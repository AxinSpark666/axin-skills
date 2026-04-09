---
name: axinrootdata
description: Query Web3 crypto project details, investor info, funding rounds, trending projects, and personnel job changes from RootData. Use when user asks about blockchain projects, crypto investors, Web3 funding history, investment rounds, trending crypto projects, or personnel movements.
tags: [crypto, web3, blockchain, investors, funding, projects, defi, rootdata]
requires:
  env:
    - ROOTDATA_SKILL_KEY
---

# RootData Web3 Intelligence

RootData 是一个领先的 Web3 数据平台，覆盖加密项目、投资机构、融资数据和人员变动。

## 首次设置

检查环境变量 `ROOTDATA_SKILL_KEY` 是否存在。

**如果不存在**，必须先通过 init 接口获取临时 key：
```
POST https://api.rootdata.com/open/skill/init
Content-Type: application/json
Body: {}
```

将返回的 `api_key` 保存为环境变量 `ROOTDATA_SKILL_KEY`。

**重要**：用户可能会提供自己的 key。如果用户提供了 key（如 `rdsk_xxxx` 格式），请直接使用用户提供的 key，不要再调用 init 接口。Key 使用变量方式传递，不要硬编码在响应或日志中。

所有请求必须包含以下 headers：
```
Authorization: Bearer {ROOTDATA_SKILL_KEY}
Content-Type: application/json
```

**安全说明**：`/skill/init` 生成的 API key 是匿名、低权限的 key，仅用于调用 RootData 的公开数据端点。

---

## Skill 1: 搜索项目/机构/人物

**使用场景**：用户想通过关键词搜索加密项目、投资机构或人物。

**请求**：
```
POST https://api.rootdata.com/open/skill/ser_inv
Body: { "query": "<搜索关键词>", "precise_x_search": false }
```

**关键响应字段**：
- `id` — 实体 ID（用于后续查询）
- `type` — 1=项目, 2=投资机构, 3=人物
- `name` — 名称
- `one_liner` — 一句话简介
- `introduce` — 完整描述
- `rootdataurl` — RootData 详情页链接

---

## Skill 2: 获取所有 ID 列表

**使用场景**：用户需要完整的项目/机构/人物 ID 列表。

**请求**：
```
POST https://api.rootdata.com/open/skill/id_map
Body: { "type": <1=项目 | 2=机构 | 3=人物> }
```

---

## Skill 3: 项目详情

**使用场景**：用户查询特定加密项目的详细信息（通过项目 ID 或合约地址）。

**请求**：
```
POST https://api.rootdata.com/open/skill/get_item
Body: { "project_id": <数字项目ID>, "include_investors": true }
```

或通过合约地址查询：
```
Body: { "contract_address": "<0x...>", "include_investors": true }
```

**关键响应字段**：
- `project_id`, `project_name`, `logo`, `token_symbol`
- `one_liner`, `description`
- `active` — 项目是否活跃
- `establishment_date` — 成立日期
- `tags` — 分类标签（如 DeFi, Layer2, NFT）
- `contracts` — 链上合约地址
- `total_funding` — 总融资额（USD）
- `social_media` — 网站、推特、GitHub、CoinMarketCap、CoinGecko 等
- `investors` — 投资者列表（仅当 include_investors=true）
- `similar_project` — 同类别相似项目
- `rootdataurl` — RootData 详情页链接

---

## Skill 4: 融资轮次

**使用场景**：用户询问融资历史、投资轮次、"XX 融了多少钱"、"谁投资了 XX"。

**数据范围**：仅覆盖**过去 365 天**的融资数据。
**投资者限制**：每个融资轮次最多返回 **3 个投资者**（按领投优先排序）。

**请求**：
```
POST https://api.rootdata.com/open/skill/get_fac
Body: {
  "page": 1,
  "page_size": 20,
  "project_id": <可选，按项目ID筛选>,
  "start_time": "<可选，格式 yyyy-MM-dd，如 2025-01-01>",
  "end_time": "<可选，格式 yyyy-MM-dd，如 2026-03-30>",
  "min_amount": <可选，最低融资金额 USD>,
  "max_amount": <可选，最高融资金额 USD>
}
```

所有字段均可选，根据用户需求填写。

**关键响应字段**：
- `total` — 符合条件的记录总数
- `items` — 融资轮次列表，每条包含：
  - `name` — 项目名称
  - `logo` — 项目 logo
  - `rounds` — 轮次类型（Seed / Series A / Series B 等）
  - `published_time` — 公布日期
  - `amount` — 融资金额（USD）
  - `project_id` — 项目 ID
  - `data_status` — 数据验证状态
  - `source_url` — 原始新闻链接
  - `X` — 项目推特链接
  - `one_liner` — 项目简介
  - `invests` — 投资者列表（最多3个），每条包含：
    - `name` — 投资者名称
    - `logo` — 投资者 logo
    - `lead_investor` — 是否为领投（1=是，0=否）
    - `type` — 1=项目, 2=机构, 3=人物
    - `invest_id` — 投资者 ID
    - `rootdataurl` — 投资者详情页

**注意**：`valuation` 字段已从响应中移除。

---

## Skill 5: 热门项目

**使用场景**：用户询问"今天热门加密项目"、"趋势项目"、"本周热门项目"。

**请求**：
```
POST https://api.rootdata.com/open/skill/hot_index
Body: { "days": <1=今日趋势 | 7=本周趋势> }
```

---

## Skill 6: 人员变动

**使用场景**：用户询问加密行业最近的人员变动、"谁加入了哪个项目"、"谁离开了哪个公司"。

**数据限制**：每个类别最多返回 **20 条**最新记录。

**请求**：
```
POST https://api.rootdata.com/open/skill/job_changes
Body: {
  "recent_joinees": true,
  "recent_resignations": true
}
```

---

## 多语言支持

添加 header `language: cn` 可获取中文响应，`language: en` 为英文（默认）。

---

## 速率限制

- 每 API key 每分钟 200 请求
- 收到 HTTP 429 时，等待 `Retry-After` 响应头中的秒数后重试
- 通过 `X-RateLimit-Remaining` header 监控剩余配额

---

## 错误码

- `200` — 成功
- `400` — 请求参数错误
- `401` — API key 无效
- `404` — 未找到
- `429` — 超过速率限制
- `500` — 服务器内部错误

---

## ⚠️ 融资汇报格式（必须严格遵守）

向用户汇报 Web3 项目融资信息时，**绝对不要使用 Markdown 表格**。必须使用以下"卡片式"排版：

```
📢 **【最新融资速递】**（[开始时间] - [结束时间]）
共监测到 **[数量]** 笔融资事件：

━━━━━━━━━━━━━━━━━━━

🚀 **1. [项目名称]**
💰 **融资金额：** [融资金额] ([融资轮次])
📅 **公布时间：** [yyyy-MM-dd]
🤝 **投资机构：** [机构列表，领投机构加粗]
🔗 **相关链接：** [如果有官网或推特链接请附上]

---

🚀 **2. [项目名称]**
💰 **融资金额：** [融资金额] ([融资轮次])
📅 **公布时间：** [yyyy-MM-dd]
🤝 **投资机构：** [机构列表]

━━━━━━━━━━━━━━━━━━━
💡 *数据来源：RootData | 仅呈现近期前3条核心数据*
```

**示例输出：**
```
📢 **【最新融资速递】**（2026-04-06 - 2026-04-09）
共监测到 **3** 笔融资事件：

━━━━━━━━━━━━━━━━━━━

🚀 **1. Pharos**
💰 **融资金额：** $44,000,000 (Series A)
📅 **公布时间：** 2026-04-08
🤝 **投资机构：** SNZ Holding, **Chainlink**, Flow Traders
🔗 **链接：** https://twitter.com/pharos

---

🚀 **2. GoSats**
💰 **融资金额：** $5,000,000 (Series A)
📅 **公布时间：** 2026-04-07
🤝 **投资机构：** Konvoy Ventures, Y Combinator, Taisu Ventures

━━━━━━━━━━━━━━━━━━━
💡 *数据来源：RootData | 仅呈现近期前3条核心数据*
```
