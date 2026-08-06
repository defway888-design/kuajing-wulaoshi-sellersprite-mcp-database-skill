# 卖家精灵 MCP 固定数据契约

本文件只保留已确认的卖家精灵数据能力、规范输入/输出语义和字段判定。不要引用外部 PDF、页面目录、未确认工具或未确认字段。

## 1. 工具绑定与通用输入契约

- 本文件中的 `asin_detail`、`keepa_info` 等名称是规范业务能力标识，不绑定任何固定 MCP 命名空间或工具前缀。
- 先通过 MCP 服务/连接元数据、配置或工具说明确认工具组属于卖家精灵；再绑定用途、关键输入、核心返回字段均匹配该能力标识的工具。不同用户环境可使用不同的服务名、命名空间或工具前缀。
- 无法确认工具组属于卖家精灵、出现多个候选工具、候选工具的用途或字段不明确、或未发现候选工具时，先请用户确认其卖家精灵 MCP 连接；仍无法确认时报告 `blocked: tool_binding_ambiguous`。不得猜测、改用其他数据源或按相近名称替代。
- 本文件中的参数名、字段名和 JSON 块是规范语义模板，不是所有 MCP 实现都必须使用的字面调用格式。绑定工具使用不同参数名、包装层或字段投影参数时，只能在其描述明确表达相同语义后映射；不得凭名称相似自行转换。
- 若某个路由未列出请求模板或关键入参，其内容只定义数据能力与输出口径。调用前必须从已绑定工具的公开输入说明确认所需输入语义；无法确认时报告 `blocked: input_contract_unconfirmed`，不得补造参数。
- 所有站点参数统一使用 Amazon 站点代码：`US`、`JP`、`UK`、`DE`、`FR`、`IT`、`ES`、`CA`、`IN`、`MX`、`BR`、`AU`、`AE`。
- 月度数据的规范时间语义为 `yyyyMM`；若绑定工具使用不同格式，只能按其明确说明转换。
- 时间范围的规范时间语义为毫秒时间戳；若绑定工具使用不同格式，只能按其明确说明转换。
- 批量 ASIN 不得超过绑定工具声明的上限；本契约中上限为 40 的路由需按 40 拆批。
- 若绑定工具支持字段投影能力（常见参数名 `returnFields`），只返回业务验证需要的字段。
- 工具返回成功但核心字段为 `null` 或缺失时，不能自行脑补；该业务路由的结果为 `blocked`。成功响应中的空数组仅在该路由明确表示“无记录”时才可作无记录处理。
- 对 `asin_coupon_trend`、`competitor_lookup`、`traffic_keyword`、`keyword_order`、`traffic_listing`、`review` 等列表路由，成功响应中的空列表表示在当前筛选条件和时间范围内无匹配记录，不表示该商品或数据源在其他条件下不存在。对趋势、统计、预测和详情路由，核心指标或详情字段为空仍为 `blocked`。

## 2. 业务路由与数据契约

### 2.1 ASIN 与商品详情

`asin_detail`

- 用途：查询单个 ASIN 商品详情。
- 典型字段：`asin`、`parent`、`variations`、`variationCount`、`title`、`brand`、`price`、`rating`、`reviews`、`status`、`available`。
- 路由：商品基础详情、父子关系补充、价格与评分核对。
- 禁止：不使用 `status` 或 `available` 单独判定 Listing 异常。

`asin_detail_with_coupon_trend`

- 用途：ASIN 详情 + Coupon 优惠趋势。
- 只用于 Coupon，不用于 Deals。

`asin_coupon_trend`

- 用途：查询 Coupon 优惠价格、优惠类型、优惠金额、最终成交价。
- 只用于 Coupon 促销，不用于 Deals。

`keepa_info`

- 用途：单 ASIN 商品画像和多维历史趋势，不含销量数据。
- 适用：商品状态、Deal 价格、父子体关系补充、Keepa 趋势。
- 常用入参：

以下是规范语义模板；`asin`、`marketplace`、`returnFields` 等键需要在绑定工具的明确映射下使用。

```json
{
  "asin": "<ASIN>",
  "marketplace": "US",
  "startTimestamp": 0,
  "endTimestamp": 0,
  "dailyLatest": false,
  "returnFields": "asin,parentAsin,variationAsins,dealPrice,productStatus"
}
```

- Deals 字段：`dealPrice[]`，每个元素为 `{ "timePoint": 毫秒时间戳, "value": 价格 }`。
- Deal 判断：任一 `dealPrice[].value > 0` 表示该时间点存在 Deal；`-1` 表示没有有效 Deal。
- Listing 状态字段：`productStatus`，实测正常值为 `STANDARD`。

- 路由：查询历史 Deal、Keepa 趋势或 Listing 状态时，唯一使用此工具。
- 禁止：不得使用其价格字段替代 Coupon；不得从此工具推断销量。

### 2.2 竞品、选品、变体

`competitor_lookup`

- 用途：查询 Amazon 商品列表数据，支持市场、月份、品牌、卖家、ASIN、类目、关键词等筛选。
- 适用：竞品列表、ASIN 对比、类目/关键词下商品调研、变体数量。
- 关键入参：

以下是规范语义模板；外层 `request`、字段投影键及分页键不是跨实现固定的字面参数名。

```json
{
  "request": {
    "marketplace": "US",
    "month": "202606",
    "asins": ["<ASIN>"],
    "variation": "N",
    "returnFields": "asin,parent,variations,variationCount,variationList,title,brand",
    "page": 1,
    "size": 40
  }
}
```

- `variation` 枚举：`Y` 表示 exclude，`N` 表示 include。
- 变体数量字段优先级：`variations` -> `variationCount` -> `variationList.length`。
- 路由：竞品列表、ASIN 对比、类目/关键词商品调研、月度变体数量对比。

`product_research`

- 用途：高级商品筛选。
- 适用：按关键词、品牌、卖家、类目、价格区间、销量、销售额、BSR、评分、利润、配送方式等筛选选品。
- 禁止：不作为具体 ASIN 变体数量验证主工具。

### 2.3 销量、销售额、BSR

`asin_sales_trend`

- 用途：查询指定 ASIN 的月度销售趋势。
- 返回：ASIN 详情、父体和子体销量、销售额、历史价格、平均价格等趋势。

`asin_prediction`

- 用途：ASIN 销量和销售额预测。
- 返回：近 14 个月销量数据、日维度和月维度预测指标。

`bsr_prediction`

- 用途：按市场、一级类目节点和大类 BSR 预测日销量与近 30 天销量。
- 入参核心：`marketplace`、`categoryId`、`bsr`。

### 2.4 关键词、流量词、ABA、Google 趋势

`traffic_keyword_stat`

- 用途：ASIN 流量关键词结构概览统计。
- 返回统计级数据，不返回具体关键词列表。
- 适用：判断自然搜索、Amazon 推荐、广告流量词结构。
- 禁止：不返回具体关键词列表，不能替代 `traffic_keyword` 或 `keyword_order`。

`traffic_keyword`

- 用途：ASIN 搜索关键词表现明细。
- 支持近 30 天或指定月份。
- 关键筛选：
  - `badges`: `naturalSearching`、`amazonChoice`、`editorialRecommendations`、`fourStar`、`highlyRated`、`sponsorBrand`、`sponsorVideo`、`ads`
  - `trafficKeywordTypes`: `primary`、`precise`、`preciseLongTail`
  - 排序字段：`rankPosition`、`adPosition`、`searchesRank`、`searches`、`purchases`、`purchaseRate`、`trafficPercentage` 等。

- 路由：查询自然/广告搜索流量词明细时使用；不用于实际出单词验证。

`keyword_order`

- 用途：出单词反查，分析 ASIN 在指定周/月实际参与曝光与转化的关键词表现。
- 关键参数：`asins`、`marketplace`、`reverseType`、`date`、`variation`。
- 周口径 `reverseType=W` 时，`date` 为 `yyyyMMdd`，表示当周周六；月口径 `reverseType=M` 时，`date` 为 `yyyyMM`。

- 路由：查询实际曝光和转化的出单词时，唯一使用此工具。

`keyword_research_trends`

- 用途：关键词选品趋势。
- 返回：搜索量、购买量、购买率、同比、环比、三个月增长率。

`aba_research_trend`

- 用途：ABA 关键词趋势。
- 返回：ABA 排名和搜索量。
- 时间粒度：`W` 周、`M` 月。

`google_trend`

- 用途：Google Trends 搜索热度。
- 注意：这是 Google 搜索行为，不是 Amazon 站内搜索数据。

### 2.5 关联流量

`traffic_listing_stat`

- 用途：ASIN 流量来源结构统计。
- 只返回统计结构，不返回具体关键词或 ASIN 明细。
- 适用：免费流量 vs 付费流量、不同关联类型数量分布。

`traffic_listing`

- 用途：查询与目标 ASIN 存在关联、竞品或同类关系的商品列表。
- 关键参数：`asinList`、`marketplace`、`relations`、`variations`、`page`、`size`。

### 2.6 评论

`review`

- 用途：查询商品评论列表。
- 关键参数：`asin`、`marketplace`、`startTimestamp`、`endTimestamp`、`page`、`size`、`starList`、`typeList`。
- 评论类型：图片评论、视频评论、VP 评论、vine 评论。

## 3. 固定判定规则

- Deal：仅 `keepa_info.dealPrice[]`。任一 `value > 0` 表示该时间点有 Deal；`-1` 表示无有效 Deal。
- Coupon：仅 `asin_coupon_trend` 或 `asin_detail_with_coupon_trend`，不得引用 Deal 价格。
- 变体数量：使用 `competitor_lookup.items[].variations`，其次 `variationCount`，再次 `variationList.length`；三者均为空时不补推。
- Listing 状态：`keepa_info.productStatus=STANDARD` 表示正常；核心字段缺失时不判定。
- 流量：`traffic_keyword_stat` 为结构统计，`traffic_keyword` 为关键词明细，`keyword_order` 为出单/转化词；三者不能按名称互换。
- 趋势：`google_trend` 只代表 Google 搜索行为，不能作为 Amazon 站内搜索趋势。

## 4. 排除范围

以下能力仅存在于历史 PDF 页面目录、但未纳入本 Skill 已确认 MCP 工具契约：市场筛选及集中度分布、关键词挖掘/拓词、关键词流向、独立商品类目查询、图片文字识别、全球商标库。不要将其作为可调用业务路由，也不要根据页面名称猜测工具或字段。
