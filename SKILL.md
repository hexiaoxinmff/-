---
name: data-analysis-reporting
description: >-
  Create professional business data analysis reports with charts. Covers
  Excel/CSV ingestion, multi-dimensional aggregation, dark-themed matplotlib
  charting with Chinese fonts on Windows, and investor-oriented markdown
  report writing with embedded visualizations.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows]
metadata:
  hermes:
    tags: [data-analysis, visualization, matplotlib, report, charts, chinese, excel]
    category: data-science
---

# Data Analysis Reporting (with Charts)

Create structured data analysis reports from Excel/CSV sources, with rich
visualizations and business/investor perspective. Designed for **execute_code**
(run all Python + matplotlib in one shot), not terminal.

## When to Use

- User asks for a **data analysis report** with charts from Excel/CSV
- User wants **investor or business perspective** analysis
- Need **multi-dimensional slicing** (by product, region, customer, price, time)
- Creating a **markdown report** with inline chart images (MEDIA: paths)

## User Preferences (This Project)

- **Output format**: HTML with tab-based layout (no-scroll). The user rejected
  markdown-in-notepad and wanted a browser-openable HTML file.
- **Style**: Professional, restrained. Minimize emoji usage in UI text; keep them
  only in data callouts (e.g. arrows for trends). The user specifically said
  "不要过多的表情符号 专业一点".
- **Depth**: Investment-grade analysis requires more than visualization. The user
  explicitly called out four missing pillars: external data validation, financial
  model, risk quantification, and specific investment targets.
- **Completeness**: Every tab/section must be fully filled. "不要空旷 补充饱满" —
  no empty or sparse areas. Each tab must have charts, tables, or insight boxes.
- **Data directory**: Always save output to `D:/hermes/` (not `C:` drive).

## Pitfalls to Avoid

❌ **Do not use terminal for matplotlib** — the bash shell on Windows (git-bash)
   often fails with path resolution. Always use `execute_code` for Python/plotting.

❌ **Do not install matplotlib at runtime if not needed** — check first with
   `pip list` inside execute_code. Install once, cache it.

❌ **Do not use emoji in matplotlib titles** — Microsoft YaHei font lacks emoji
   glyphs and will emit warnings. Use plain Chinese text or ASCII for chart titles.

❌ **Do not generate charts one at a time** — batch all in one execute_code call
   to avoid repeated pandas reads and matplotlib state resets.

❌ **Do not omit output_dir from chart paths** — always save to D:/hermes/ (the
   user's data directory) so MEDIA: paths work from the markdown report.

❌ **Do not stop at markdown reports** — if the user asks for HTML, build a
   self-contained HTML file with tab navigation and CSS styling that can be
   opened via `file:///` in a browser.

❌ **Do not skip HTML integrity checks** — after writing/replacing sections,
   always verify: (1) all tab buttons map to existing panes, (2) div opens =
   div closes, (3) all chart images are referenced, (4) no broken HTML entities
   like `&display=` instead of `&amp;display=`.

❌ **Do not leave tabs empty or sparse** — each tab must have substantive
   content. If a tab has only one table or chart, add supplemental data
   (cross-analysis, per-segment breakdown, time-series comparison) to fill it.

❌ **Do not skip data sanity checks** — before any analysis, verify that
   profit = sales - cost, identify outliers, and check for duplicate records.
   Report the results as a data quality section in the final output.

❌ **Do not present a "nice dashboard" as an investment report** — a real
   investment analysis needs: external benchmarks vs industry standards,
   financial projections (annualized), ROI scenarios, risk quantification
   (Monte Carlo, HHI index, VaR), and specific investment targets with
   position sizing.

## Workflow

### Step 1: Initial Data Assessment

Read the Excel file to understand structure, dimensions, and key stats:

```python
import pandas as pd
df = pd.read_excel("D:/hermes/your_file.xlsx")
print(df.shape, df.columns.tolist(), df.dtypes)
print(df.describe())
print(df['product_col'].unique())
```

Check for: date range, categorical dimensions, outlier values, missing data.

### Step 2: Multi-Dimensional Analysis

Always slice along these dimensions (at minimum):

| Dimension | Aggregation | Purpose |
|-----------|-------------|---------|
| **Product** | sum(sales), sum(profit), count, avg(price) | Find winners |
| **Category/Price segment** | bins + groupby | Value tier analysis |
| **Customer type** | sum, % share, profit margin | Who brings profit |
| **Region** | sum, margin | Geographic value |
| **Time** | daily groupby | Trend & seasonality |

Key metrics to compute per slice:
- `利润率 = 总利润 / 总销售额 * 100`
- `销售额占比 = 总销售额 / 总销售额.sum() * 100`
- `平均单笔 = 总销售额 / 交易次数`

### Step 3: Chart Setup (Chinese Fonts on Windows)

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm
import numpy as np
import pandas as pd

# Font — Microsoft YaHei ships with Windows 10+
font_path = 'C:/Windows/Fonts/msyh.ttc'
zh_font = fm.FontProperties(fname=font_path, size=12)
zh_title = fm.FontProperties(fname=font_path, size=16, weight='bold')
zh_small = fm.FontProperties(fname=font_path, size=10)
zh_medium = fm.FontProperties(fname=font_path, size=13)
plt.rcParams['font.family'] = 'Microsoft YaHei'
plt.rcParams['axes.unicode_minus'] = False
plt.rcParams['font.sans-serif'] = ['Microsoft YaHei']
```

### Step 4: Dark Theme Boilerplate

```python
BG = '#1a1a2e'
GRID = '#2d2d44'
TEXT = '#e0e0e0'
BLUE = '#00b4d8'
GREEN = '#06d6a0'
RED = '#ef476f'
ORANGE = '#ffd166'
PURPLE = '#9b5de5'
GOLD = '#ffd700'
OUT = 'D:/hermes'  # user data directory

fig, ax = plt.subplots(figsize=(14, 7))
fig.patch.set_facecolor(BG)
ax.set_facecolor(BG)
ax.tick_params(colors=TEXT)
for s in ['bottom','top','left','right']:
    ax.spines[s].set_color(GRID)
ax.grid(axis='y', alpha=0.15, color=GRID)
```

### Step 5: Chart Types by Use Case

| Use Case | Chart Type | Notes |
|----------|-----------|-------|
| **Product comparison** | Horizontal grouped bar (twinx) | Sales on ax1, Profit on ax2 |
| **Ranking** | Horizontal bar (sorted) | Color by tier (green/blue/orange/red) |
| **Time trend** | Line + fill_between | Twinx for sales & profit |
| **Composition** | Pie + bar combo | Share (pie) + margin (bar) |
| **Cross-tabulation** | Heatmap (seaborn) | Region × Customer, annotated % |
| **Comparison of 2 groups** | 2×2 quad of bar charts | 4 metrics in one figure |
| **Top-N ranking** | Horizontal bar | For outliers/high-value items |

### Step 6: Write the Markdown Report

- Use a clear **hierarchy**: H1 → H2 → H3
- **Tables** for structured data (align columns with `|--:|` for right-align numbers)
- **Inline charts** via `MEDIA:/D:/hermes/chart_name.png`
- **Callout blocks** with `> ` for key findings
- **Emoji markers** in text work fine (only matplotlib titles lack emoji)
- End with **investment/actionable recommendations** in a numbered list
- Include a **risk section** for credibility

Save via `execute_code` (not `write_file`, which may have shell quoting issues with Chinese characters):

```python
with open("D:/hermes/report_output.md", "w", encoding="utf-8") as f:
    f.write(content_string)
```

### Step 7: Verify

```python
import os
for f in ["report_output.md", "chart1_xxx.png", ...]:
    path = f"D:/hermes/{f}"
    print(f"{'OK' if os.path.exists(path) else 'MISSING'}: {f}")
```

### Step 8 (Advanced): Investment-Grade Upgrades

When the user expects a **real investment analysis** (not just a data dashboard),  
add these **four pillars** after the basic analysis. The user explicitly validated  
this framework — skipping any pillar will produce a "nice dashboard" instead of  
a decision-grade report.

#### 8a. Data Sanity Verification

Before any analysis, validate data integrity. See `references/financial-model-templates.md` for code.

#### 8b. External Benchmarking (Industry Validation)

Compare against: hardware gross margin (8-15%), enterprise B2B margin (45-65%),  
promo spike (180-250%), avg selling price (¥5,500-7,000), enterprise purchase premium (15-30%).

#### 8c. Financial Model

Build: annual projection (618 = 8-12% of annual), ROI matrix (5 scenarios),  
sensitivity analysis (margin ±15%, ASP ±10%), per-segment P&L.

#### 8d. Risk Quantification

Compute: HHI concentration index, Monte Carlo simulation (10,000 iterations),  
VaR(95%), Sharpe ratio, scenario analysis.

#### 8e. Investment Targets

Map to tickers: Lenovo (00992.HK), Apple (AAPL), BYD Electronic (00285.HK).  
Include execution roadmap with phases.

### Step 9: HTML Tab Report (User Preference)

When the user prefers HTML over markdown scrolling, build a self-contained
tabbed page. This is the user's strongly stated preference: **no-scroll tabs**,
not a long scrolling page.

### HTML Layout Pattern

```html
<html>
<head>
  <style>
    html,body{height:100%;overflow:hidden}
    body{display:flex;flex-direction:column;height:100vh}
    .topbar{height:48px;flex-shrink:0}
    .tab-cw{flex:1;overflow:hidden;position:relative}
    .tab-pane{position:absolute;top:0;left:0;right:0;bottom:0;overflow-y:auto}
  </style>
</head>
<body>
  <div class="topbar">
    <div class="tabs">
      <button class="tab-btn active" data-tab="overview">概览</button>
      <button class="tab-btn" data-tab="products">产品</button>
    </div>
  </div>
  <div class="tab-cw">
    <div class="tab-pane active" id="pane-overview">...</div>
    <div class="tab-pane" id="pane-products">...</div>
  </div>
  <script>
    document.querySelectorAll('.tab-btn').forEach(function(btn){
      btn.addEventListener('click', function(){
        document.querySelectorAll('.tab-btn, .tab-pane').forEach(function(e){e.classList.remove('active')});
        btn.classList.add('active');
        document.getElementById('pane-' + btn.dataset.tab).classList.add('active');
      });
    });
  </script>
</body>
</html>
```

### HTML Integrity Checks (AFTER writing)

```python
import re
with open("report.html", "r", encoding="utf-8") as f:
    html = f.read()

tabs = set(re.findall(r'data-tab="(\w+)"', html))
panes = set(re.findall(r'id="pane-(\w+)"', html))
assert tabs == panes, f"Mismatch: {tabs-panes} / {panes-tabs}"
assert html.count('<div') == html.count('</div>'), "Div imbalance"
# Check all chart .png files referenced
for c in ['chart1','chart2','chart3','chart4','chart5','chart6','chart7','chart8','chart9','chart10']:
    assert c in html, f"Missing: {c}"
# Check for broken entities like &display= without &amp;
import re
for m in re.finditer(r'&([a-z]{2,20})([^a-z0-9;#"])', html, re.I):
    print(f"BROKEN ENTITY near: {html[m.start()-10:m.end()+10]}")
print(f"OK: {len(tabs)} tabs, {len(panes)} panes, balanced divs")
```

### Professional Styling Rules
- **All tabs filled**: "不要空旷 补充饱满" — no sparse panes
- **Tab labels**: Short, single or 2-char Chinese (概览/产品/价格/客户/区域/对标/趋势/财务/风险/标的)

## Usage

### Quick Reference: Typical session flow

```
1. Read Excel → dimension check → describe()
2. Multi-dimension aggregation (product, region, customer, price, time)
3. Data sanity check (profit formula, outliers, duplicates)
4. Revenue/profit projection
5. ROI scenarios + sensitivity analysis
6. Risk quantification (HHI, Monte Carlo)
7. Investment targets + position sizing
8. Generate charts (batch all in one execute_code)
9. Build HTML tab report + integrity verification
10. Save to D:/hermes/output.html
```

## Linked References

- `references/financial-model-templates.md` — Complete code templates for:
  - Data sanity verification
  - External benchmarking
  - Annual revenue projection
  - ROI matrix
  - Sensitivity analysis
  - HHI concentration index
  - Monte Carlo simulation
  - Per-transaction P&L
  - Investment targets table

- `references/617-report-chart-patterns.md` — Chart code templates and
  data findings from the 618 notebook sales analysis