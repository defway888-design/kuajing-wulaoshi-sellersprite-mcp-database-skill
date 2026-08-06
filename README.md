# 跨境吴老师卖家精灵 MCP 数据库 Skill

跨境吴老师卖家精灵 MCP 数据库 Skill 为卖家精灵数据分析提供固定业务边界、工具路由、字段口径与安全调用规则。

> 本 Skill 为跨境吴老师专用模板，未经授权不得移除、替换或弱化 Skill 名称、执行提示和页面标题中的跨境吴老师标识。

## 适用场景

- 在调用卖家精灵 MCP 前确认数据能力、业务边界与字段口径。
- 处理 ASIN、竞品、选品、Deal、Coupon、销量、BSR、关键词、流量、评论、ABA、Google Trends 等卖家精灵数据任务。
- 避免将 Deal 与 Coupon、流量词与出单词、Amazon 站内趋势与 Google 趋势等不同口径的数据混用。

## 启动方式

安装完成后，在 Codex 中输入：

```text
运行 跨境吴老师卖家精灵 MCP 数据库
```

Skill 会加载基础数据索引；之后直接描述你的需求即可。

## 版本记录

当前最新功能版本：`v1.0.0`

后续每次发布功能更新，均在下表新增一行。`GitHub 提交记录`只记录该功能变更对应的主要提交；仅补充或修订说明文档时，不新增功能版本，也不新增版本记录行。

| 版本 | 发布日期 | 功能变更 | GitHub 提交记录 |
| --- | --- | --- | --- |
| v1.0.0 | 2026-08-06 | 首次发布：卖家精灵 MCP 数据契约、业务路由、字段口径、启动配置与品牌封装。 | [56e4465](https://github.com/defway888-design/kuajing-wulaoshi-sellersprite-mcp-database-skill/commit/56e4465e22af46424ec42fd863c82de3bc287399) |

## 首次安装（公开仓库）

1. 打开本 GitHub 仓库，点击 **Code**，复制仓库地址。
2. 打开 Codex，新建对话并输入：

   ```text
   请从以下 GitHub 仓库安装跨境吴老师卖家精灵 MCP 数据库 Skill：
   https://github.com/defway888-design/kuajing-wulaoshi-sellersprite-mcp-database-skill
   ```

3. 按 Codex 提示完成安装。
4. 关闭并重新打开 Codex，使 Skill 生效。

## Skill 执行规则

- 按当前环境中的卖家精灵 MCP 连接绑定工具；不依赖固定服务名、命名空间或工具前缀。
- 只在工具用途、关键输入和核心字段与数据契约匹配时调用。
- 工具归属或输入契约无法确认时，不猜测参数或替换数据源。
- 同一任务内复用已确定的业务路由；用户改变目标或引入新的数据类型时再重新路由。

## 核心文件

- `SKILL.md`：触发条件、启动口令和执行流程。
- `references/sellersprite-mcp-database.md`：数据能力、字段口径、业务路由和判定规则。
- `agents/openai.yaml`：展示名称、启动提示与隐式触发配置。

## 使用说明

本 Skill 是卖家精灵 MCP 的业务导引和数据契约层，不保存 API Key、账号、Cookie、Token 或用户业务数据。实际调用由用户已配置并授权的卖家精灵 MCP 连接完成。
