# 详细设计规范文档

> 本文档定义网页的详细设计规范，供 Agent 实施和日常维护使用。

---

## 1. 数据块格式定义

### 1.1 每日研报数据

```json
{
  "date": "2026-08-31",
  "reports": [
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
  ]
}
```

### 1.2 推荐板块数据

```json
{
  "date": "2026-08-31",
  "cyclePosition": {
    "stage": "复苏期（早期）",
    "indicators": [
      {"name": "PMI", "value": "50.2", "signal": "荣枯线上方", "direction": "up"},
      {"name": "CPI", "value": "0.6%", "signal": "低位温和", "direction": "flat"},
      {"name": "社融", "value": "同比多增", "signal": "信用扩张", "direction": "up"},
      {"name": "LPR(1Y)", "value": "3.35%", "signal": "持续下调", "direction": "down"}
    ],
    "favoredSectors": [
      {"name": "有色金属", "status": "green", "note": "全球复苏+美元走弱"},
      {"name": "可选消费", "status": "green", "note": "政策刺激+消费回暖"}
    ]
  },
  "lowValuation": [
    {
      "sector": "银行",
      "pe": 5.2,
      "pePercentile": 8.5,
      "pb": 0.55,
      "pbPercentile": 3.2,
      "dividendYield": 5.8,
      "reason": "估值历史低位，高股息防御属性突出"
    }
  ],
  "recommendations": [
    {
      "rank": 1,
      "sector": "有色金属（铜/铝）",
      "logic": "全球复苏+美元走弱+供给约束",
      "watchPoints": "PE分位值、大宗商品价格趋势",
      "riskLevel": "中"
    }
  ]
}
```

### 1.3 HTML 中数据块标记

```html
<!-- DAILY_DATA_START -->
<script id="daily-data" type="application/json">
  { ... 上述合并的 JSON ... }
</script>
<!-- DAILY_DATA_END -->
```

Agent 更新时只需替换 `<!-- DAILY_DATA_START -->` 和 `<!-- DAILY_DATA_END -->` 之间的内容。

---

## 2. CSS 变量定义

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

## 3. 章节锚点 ID 规范

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
| - 子节 | `#recommend-low` | 低估值筛选 |
| - 子节 | `#recommend-cycle` | 顺周期定位 |
| - 子节 | `#recommend-summary` | 综合推荐 |
