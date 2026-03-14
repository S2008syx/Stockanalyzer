# Stock Analyzer — 用户手册

## 1. 首页 (Homepage)

### 搜索功能
* 支持股票代码及公司名称的模糊搜索（400ms 防抖）。
* 输入后自动显示下拉候选列表（代码 + 公司名 + 交易所）。
* 回车可直接提交代码。

### Watchlist（自选股）
* 搜索并选择股票后自动加入 Watchlist，持久化存储于 localStorage。
* 每条记录显示：股票代码、公司名称、交易所。
* 点击条目直接加载该股票的详情分析页。
* 每条右侧有删除按钮，可移除单只股票。

### 全局控制
* **Free Key（黄色按钮）**：输入 PIN 码后可解锁内置的免费 OpenRouter API Key，支持一键复制和可用性测试。
* **Clear Storage（红色按钮）**：一键清除 Watchlist、API Keys 及所有 localStorage 缓存数据（操作前有确认弹窗）。
* **API Keys（白色按钮）**：
   * **FMP & Finnhub**：系统内置默认 Key，支持 Accessibility Test（一键测试连通性）。
   * **LLM 配置**：用户可选 OpenRouter 或 Claude AI 作为 AI 服务商。每个 Key 均支持单独测试验证。

### 模式切换
* **"Are you a pro"** 复选框：开启时有彩纸动画。开启后进入 Detail Page 前会强制跳转 Data Checklist Page 进行人工数据审核。

### 底部导航
* 左下角：**Financial Calculator** 链接（跳转至独立的财务计算器工具）。
* 右下角：**Economy Analysis** 链接（跳转至经济分析工具）。


## 2. 数据校验页 (Data Checklist Page)

### 校验项目（共 18 项）

| 类型 | 校验项 |
|------|--------|
| 历史数据 (15Y) | PE Ratio, PS Ratio, EBITDA, Market Cap, FCF, OCF, Net Debt |
| 当前数据 | Stock Price, EPS (TTM), Shares Outstanding, Expected Growth Rate |
| 结构分布 | Revenue by Geography, Revenue by Segment |
| 价格序列 | 1Y Daily Prices, 5Y Crude Oil ETF (USO) |
| 只读/可编辑 | Significant Days >5%, Analyst Target Price, Analyst Rating |

### 功能特性
* **自动化抓取**：显示各项指标的获取状态（✓ 可用 / ⚠ 部分缺失 / ✗ 不可用）。
* **人工补偿**：对缺失的历史数据，提供逐年输入网格；对分布类数据，支持添加/删除行。
* **API 数据保护**：已从 API 获取的数据以绿色高亮显示，不可编辑。
* **自动保存**：编辑过程中自动保存至 localStorage。
* **两种前进方式**：
   * "Continue with available data"：跳过缺失项直接进入详情页。
   * "Confirm & View Analysis"：保存手动输入的数据后进入详情页。


## 3. 详情页 (Detail Page)

### A. 核心数据看板 (Metrics Table)
* **数据源**：FMP + Finnhub。
* **可编辑字段**：Stock Price、EPS (TTM)、Expected Growth Rate、Shares Outstanding。
* **自动联动**：修改 Price 后，Market Cap、PE Ratio、Forward PE 实时重算更新。
* **只读计算字段**：PE Ratio、Forward PE、Market Cap、PS Ratio、EBITDA、Free Cash Flow。
* **分析师数据**：Analyst Target Price、Analyst Rating（Buy/Hold/Sell）来自 Finnhub 共识数据。
* 若未配置 Finnhub Key，支持在表格内直接提交 Key。

### B. AI Insights & 深度分析
* **公司概况**：一句话简介（加粗核心关键词）+ 3–5 个主营业务标签（蓝色胶囊样式）。
* **增长率预测 (AI)**：结构化 3 行分析：
   * Line 1：具名分析师/机构的预期增长率 + 平均值。
   * Line 2：增长预测的核心依据与催化因素。
   * Line 3：潜在下行风险因素。
* **Pros & Cons（折叠式）**：点击展开后 AI 生成，分两列展示：
   * 左列 3 条 Pros（绿色图标）：基本面、AI 影响、宏观经济。
   * 右列 3 条 Cons（红色图标）：基本面、AI 影响、宏观经济。
* **AI 问答框**：
   * 支持自由提问，AI 基于该股票数据回答。
   * 支持 Web Search（Claude Sonnet 模型联网搜索），回答附带来源引用。
   * 单次对话模式（无上下文记忆），节省 Token。
   * 聊天记录持久化存储，支持清空历史。

### C. 财务指标解读 (Financial Metrics Explained)
* **默认折叠**，点击展开。
* 以表格形式对比 5 大核心指标（Revenue/PS、EBITDA、OCF、EPS、FCF）在以下维度的包含关系：
   * 是否包含 COGS（销售成本）
   * 是否包含 Interest（利息）
   * 是否包含 D&A（折旧摊销）
   * 是否包含 CapEx（资本支出）
* **Working Capital 专题**：
   * 公式：Working Capital = Current Assets - Current Liabilities
   * 正值含义：公司应收大于应付，现金被占用，OCF 可能低于净利润。
   * 负值含义：公司应付大于应收，相当于使用供应商资金，OCF 可能高于净利润。

### D. 主图表 (Main Charts)
15 年历史数据可视化（2×N 网格布局）：
* **PE Ratio**（支持负 PE 标记）
* **Market Cap**
* **Free Cash Flow**（柱状图）
* **EBITDA**
* **Operating Cash Flow**（柱状图）
* **1Y Price Trend**（含 >5% 波动日红点标记）

### E. Supplement Charts（默认折叠）

#### Valuation & Profitability
* Net Profit Margin (15Y)
* EPS (15Y)
* EV/EBITDA (15Y)
* PS Ratio (15Y)
* Working Capital（5Y，数据可用时显示）

#### Cash & Equity
* Total Debt (15Y)
* Total Assets (15Y)
* Assets - Debt / Net Worth (15Y)
* Net Cash Liquidity (15Y)

#### Macro Index
* 5Y Crude Oil ETF (USO) 季度价格图（懒加载，展开时才获取数据）。

### F. 结构分析 (Distribution)
* 两个并排饼图：
   * **Geographic Revenue**：按地区分布的营收占比。
   * **Business Segment Revenue**：按业务板块分布的营收占比。


## 4. 波动新闻分析 (Price Alerts)

### 触发机制
* **默认折叠**，标题显示波动日数量。
* 自动筛选过去 1 年中日涨跌幅 >5% 的交易日。

### 波动日列表
| 列 | 内容 |
|----|------|
| Date | 日期 (YYYY-MM-DD) |
| Change | 涨跌幅百分比（绿色=涨，红色=跌）|
| News | AI 生成的新闻摘要（≤30 字），解释该日波动原因 |
| Category | 分类标签（可手动覆盖）|

### 智能归因分类
每条波动新闻自动分类并标记颜色：
* 🟦 **Macro**（宏观）
* 🟩 **Fundamentals**（基本面）
* 🟥 **Tech**（技术/AI 冲击）
* ⬜ **Other**（其他）

用户可通过下拉菜单手动修改分类，覆盖值持久化存储。

### News Briefing
* 点击 **Generate News Briefing** 按钮，AI 生成 4 条总结（每类 1 句，≤25 字）。
* 支持 **Regenerate** 重新生成。
* 按颜色分类展示（蓝=宏观、绿=基本面、红=技术、灰=其他）。


## 5. 底部工具栏

* **Missing Data Footer**：黄色警告框，持续提醒当前页面缺失的数据项。区分 API 错误和数据缺失两种情况，方便回溯补充。


## 6. Financial Calculator（财务计算器）

从首页左下角进入，当前包含一个计算器模块。

### FANG Oil Price → EPS Calculator
专为 Diamondback Energy (FANG) 设计的油价→盈利估算工具，通过 6 种独立方法将 WTI 原油价格映射为年化 EPS。

#### 主要输入
* **WTI 油价**：$40–$120/桶，滑块 + 数字输入双控件。超出 $55–$75 历史回测区间时显示黄色外推风险警告。
* **方法选择**：6 个独立复选框，勾选/取消勾选即时影响统计汇总。至少选 1 个方法。
* **Target EPS**（可选）：输入目标值后，红点在分布曲线上实时移动，显示百分位排名。
* **PE 倍数**：3 个预设场景 + 自定义输入。
   * Bear Case (12x)："油价将下跌"
   * Base Case (15x)："油价持平"
   * Bull Case (18x)："油价将上涨"
   * Custom：用户自定义 PE 值

#### 6 种估算方法
| 方法 | 名称 | 核心逻辑 |
|------|------|----------|
| M1 | Constant ΔWTI | 基于 Q4 2024 基线的线性敏感性外推（~$0.43/桶）|
| M2 | Bottom-Up (Oil Only) | 全成本堆叠季度损益模型，Gas=$1.40/Mcf，偏差修正 0.848 |
| M3 | Bottom-Up (Oil+Gas) | 同 M2 但 Gas 价格可编辑（默认 $3.00/Mcf），偏差修正可编辑（默认 1 = 无修正）。标签随 Gas 价格动态变化 |
| M4 | Regression Calibrated | 对 M3 原始输出做校准回归：-1.464 + 1.290 × rawQ |
| M5 | Historical OLS | 纯经验回归：qEPS = -8.490 + 0.1779 × WTI，R²=0.94 |
| M6 | Barrel→Earnings Chain | 两步年度回归：WTI→Margin→EPS，~$3.30 EPS/每$10油价 |

#### 输出展示
* **结果表格**：6 种方法各自的年化 EPS 估算值（正值绿色，负值红色）。
* **统计汇总面板**（4 卡片）：
   * Mean EPS（均值）
   * Std Dev（标准差，n=1 时显示 N/A）
   * 68% CI（均值 ± 1σ）
   * 95% CI（Student's t 分布，显示自由度 df=n-1）
* **t 分布图**：选中 ≥2 方法后出现，蓝色填充曲线 + 红点标记 Target EPS 位置与百分位。
* **隐含股价**：Mean/Target EPS × 选定 PE = 隐含股价（绿色大字显示）。

#### 特色功能
* **Data Checklist 弹窗**：可编辑全部 62 个底层参数（含数据来源标注：10-K、10-Q、EIA 等），支持一键重置默认值。
* **Documentation 弹窗**：完整方法论文档，含公式、敏感性分析和局限性说明。
* **有效期徽章**：绿色 "Data valid before YYYY/MM/DD" 或红色 "Data expired"，基于下一财报日判断参数是否需要更新。
* **全状态持久化**：油价、方法选择、Target EPS、PE 选择、62 个参数均保存至 localStorage。


## 7. Economy Analysis（宏观经济分析）

从首页右下角进入，当前包含一个分析模块。

### Macro Risk Analyzer（宏观风险分析器）
副标题："Systemic Risk Transmission Loop — A Fragile Equilibrium Under Stress"

#### A. Doom Loop 传导链（交互式流程图）
6 个节点组成的因果传导闭环，点击任一节点展开衍生问题分析：

| 节点 | 颜色 | 含义 | 关键数据 |
|------|------|------|----------|
| Oil Price Surge | 🔴 红 | 霍尔木兹海峡封锁 | 153→13 船/天（仅剩 8% 通行量）|
| Imported Inflation | 🟠 橙 | CPI 推升至 3% 以上 | 通胀传导滞后 3-6 个月 |
| Fed Holds / Hikes | 🟡 黄 | 不降息甚至加息 | CME FedWatch: 3月不降息概率 99% |
| Wall St Valuation Kill | 🔵 蓝 | 估值压缩 | 市场预期 1-2 次降息 → 零次 = 2-4% 估值压缩 |
| K-Shape Upper End | 🟣 紫 | 财富效应逆转 | 白领裁员 9.2 万（2月），全年 117 万（后疫情新高）|
| Stock Price Double Kill | 🔴 红 | 盈利下滑 + 估值压缩 | 消费下降 → 盈利miss → 估值进一步压缩 |

* 每个节点可点击展开弹窗，内含 Q-given（已知问题）和 Q-new（新思考，青色标记）。
* SVG 箭头连接 "Wall St Valuation Kill" 到 "Stock Price Double Kill" 形成反馈环。

#### B. 附带伤害渠道 (Collateral Damage Channels)
3 张风险卡片：
* **Subprime ABS Deterioration**：次贷 ABS 拖欠率 6.9%（历史新高），低收入汽车贷款违约率 +70bps。
* **Credit Market Complacency**：违约率 0.40%，HY OAS 2.9% 接近 2007 低点（2.4%），投机级违约 4.8%。
* **AI Cannot Save the Economy Short-Term**：AI 通缩循环（AI→裁员→消费下降→更多 AI），新岗位创造滞后于岗位消失。

#### C. 多空辩论记分卡 (Argument Scorecard)
7 行对比表格，列出 Bear Case vs Bull Counter + 判定结果：

| 议题 | 判定 |
|------|------|
| Oil & Inflation | Bear Wins |
| Fed & Rates | Lean Bear |
| K-Shape Consumption | Bear Wins（最强论据）|
| Subprime ABS | Draw |
| Credit Spreads | Bear Wins |
| AI & Economy | Lean Bear |
| LTCM Analogy | Bear Wins（框架性）|

#### D. 底层假设 (Underlying Assumptions)
4 张可折叠假设卡片（橙色左边框）：
1. **战争被低估**：伊朗最高领袖遇刺为历史前所未有，霍尔木兹封锁无 30 年先例。
2. **美国经济脆弱平衡（LTCM 类比）**：企业不裁不招，通胀 3% 不上不下，底层消费者已达极限。
3. **多重压力向量同时收敛**：油价冲击 + 就业恶化 + AI 裁员 + 次贷恶化 + Fed 困境 + 高估值，单独可承受，组合尾部风险远超市场定价。
4. **时间是空头最大的敌人**：Doom Loop 可能需 6–18 个月完全显现，储备提供缓冲，Fed 仍有弹药。

#### 交互特性
* 点击节点展开/收起衍生问题弹窗，点击外部关闭。
* 深色主题，按风险等级颜色编码（红/橙/黄/蓝/紫/青）。
* 768px 断点响应式适配移动端。


## 8. 技术与体验细节

### 数据缓存
| localStorage Key | 内容 |
|------------------|------|
| `sa_api_keys` | API Key 配置（FMP, Claude, Finnhub, OpenRouter, AI Provider）|
| `sa_watchlist` | 自选股列表 |
| `sa_ov_<TICKER>` | 单只股票的指标覆盖值 |
| `sa_inputs_<TICKER>` | 用户在 Checklist 中手动输入的数据 |
| `sa_cache_<TICKER>` | 缓存的 API 原始数据 |
| `sa_ai_<TICKER>` | AI Insights 缓存 |
| `sa_pc_<TICKER>` | Pros & Cons 缓存 |
| `sa_qa_<TICKER>` | Q&A 聊天记录 |
| `sa_alerts_<TICKER>` | 波动日新闻数据缓存 |
| `sa_catov_<TICKER>` | 波动新闻分类覆盖值 |

### 响应式设计
* 768px 断点：Metrics 网格 4列→2列，图表 2列→1列，饼图 2列→1列。

### 主题
* 固定深色主题，绿色主色调（#4ADE80），无浅色模式。

### 外部 API
| API | 用途 |
|-----|------|
| FMP | 全部财务数据、公司概况、新闻、股价 |
| Finnhub | 分析师评级、目标价、按日期新闻 |
| Claude (Anthropic) | AI 分析（claude-sonnet-4-6，支持联网搜索）|
| OpenRouter | AI 分析备选通道（claude-3.5-haiku）|
