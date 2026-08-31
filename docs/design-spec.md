# 详细设计规范文档

> 本文档定义网页的详细设计规范，供 Agent 实施和日常维护使用。

---

## 1. 数据块格式定义

### 1.1 每日数据总结构

`index.html` 中的动态数据以 JSON 格式嵌入在 `<script id="daily-data" type="application/json">` 标签中。Agent 更新时只替换 `<!-- DAILY_DATA_START -->` 和 `<!-- DAILY_DATA_END -->` 之间的内容。

```json
{
  "date": "2026-08-31",
  "reports": [],
  "macroCredit": [],
  "fundFlows": [],
  "marketSentiment": [],
  "commoditiesFx": [],
  "marketJudgement": {},
  "sourceQuality": [],
  "cyclePosition": {},
  "lowValuation": [],
  "recommendations": []
}
```

### 1.2 研报字段

```json
{
  "id": "r001",
  "category": "宏观策略",
  "title": "研报标题",
  "source": "券商名称",
  "rating": "买入/增持/中性/减持",
  "summary": "核心观点摘要，3-5句话",
  "keyPoints": ["要点1", "要点2", "要点3"],
  "tags": ["降息", "社融", "PMI"]
}
```

### 1.3 指标数组字段

`macroCredit`、`fundFlows`、`marketSentiment`、`commoditiesFx` 均使用统一结构，便于页面复用渲染：

```json
{
  "name": "PMI",
  "value": "49.8%",
  "signal": "环比改善",
  "direction": "up",
  "source": "国家统计局"
}
```

- `direction` 可选值：`up`、`down`、`flat`。
- 无法确认的数值使用 `待更新`，不得编造。
- `source` 需要写明来源或口径。

### 1.4 行情判断字段

```json
{
  "stage": "复苏期（早期）",
  "signalScore": 68,
  "signalLevel": "偏积极",
  "conclusion": "综合结论",
  "supportingFactors": ["支持因素1", "支持因素2"],
  "riskFactors": ["风险因素1", "风险因素2"],
  "rules": ["判断规则1", "判断规则2"]
}
```

判断规则应围绕增长、通胀、信用、资金、情绪、估值和商品信号展开，避免只引用单一研报观点。

### 1.5 数据源质量字段

```json
{
  "source": "官方数据",
  "level": "高",
  "usage": "PMI、CPI、PPI、社融、M2、LPR等宏观指标",
  "verified": true,
  "updatedAt": "按官方发布时间"
}
```

来源优先级：官方/交易所数据 > `finance-data` 插件 > 主流财经网站 > 券商公开研报 > 普通新闻。

### 1.6 推荐板块字段

```json
{
  "rank": 1,
  "sector": "有色金属（铜/铝）",
  "logic": "全球复苏+供给约束+估值偏低",
  "watchPoints": "铜价、美元指数、全球PMI、资金流",
  "riskLevel": "中",
  "evidence": {
    "valuation": "PE/PB分位值双低",
    "fund": "资金流待验证",
    "cycle": "复苏期资源品弹性较高",
    "risk": "美元走强或全球需求不及预期"
  }
}
```

`evidence` 为新增可选字段；若旧数据缺失该字段，页面应正常显示。

---

## 2. 页面模块与渲染规则

| 模块 | 数据字段 | 渲染要求 |
|------|----------|----------|
| 每日研报精选 | `reports` | 卡片展示，按宏观策略/行业分析/个股研究筛选 |
| 市场观察面板 | `marketJudgement` | 展示阶段、信号强度、结论、支持因素、风险因素 |
| 宏观信用 | `macroCredit` | 指标卡片，显示名称、值、信号、方向、来源 |
| 资金流向 | `fundFlows` | 指标卡片，显示北向、融资、主力、ETF等 |
| 市场情绪 | `marketSentiment` | 指标卡片，显示成交额、涨跌比、涨跌停、风格 |
| 商品与汇率 | `commoditiesFx` | 指标卡片，显示铜、原油、美元、人民币 |
| 数据源质量 | `sourceQuality` | 卡片展示来源等级、用途、更新口径、验证状态 |
| 低估值筛选 | `lowValuation` | 表格展示并支持排序 |
| 顺周期定位 | `cyclePosition` | 展示周期阶段、指标列表和受益行业 |
| 综合推荐 | `recommendations` | 表格展示推荐逻辑、关注要点、风险等级和证据标签 |

所有新增模块必须支持安全降级：字段缺失时显示“暂无数据”，不能导致脚本报错。

---

## 3. CSS 变量定义

```css
:root {
  --bg-main: #0d1117;
  --bg-card: #161b22;
  --bg-hover: #21262d;
  --text-primary: #c9d1d9;
  --text-secondary: #8b949e;
  --color-up: #3fb950;
  --color-down: #f85149;
  --color-link: #58a6ff;
  --color-accent: #d2a8ff;
  --border-color: #30363d;
  --radius: 8px;
  --font-size: 15px;
  --line-height: 1.7;
}
```

---

## 4. 章节锚点 ID 规范

| 模块 | 锚点 ID | 导航文字 |
|------|---------|---------|
| 模块一 | `#basics` | 基础金融概念库 |
| - 子节 | `#basics-market` | 股票市场基础 |
| - 子节 | `#basics-valuation` | 估值方法论 |
| - 子节 | `#basics-finance` | 财务分析基础 |
| - 子节 | `#basics-tech` | 技术分析基础 |
| - 子节 | `#basics-strategy` | 投资策略 |
| 模块二 | `#cycle` | 周期趋势框架 |
| - 子节 | `#cycle-clock` | 美林投资时钟 |
| - 子节 | `#cycle-china` | 中国特色周期 |
| - 子节 | `#cycle-guide` | 顺周期操作指南 |
| - 子节 | `#cycle-case` | 行业分析范例 |
| 模块三 | `#reports` | 每日研报精选 |
| 模块四 | `#recommend` | 板块推荐 |
| - 子节 | `#recommend-dashboard` | 市场观察面板 |
| - 子节 | `#recommend-sources` | 数据源质量 |
| - 子节 | `#recommend-low` | 低估值筛选 |
| - 子节 | `#recommend-cycle` | 顺周期定位 |
| - 子节 | `#recommend-summary` | 综合推荐 |
