# 🚀 我的数据科学全栈速查表 (Data Science Cheat Sheets)

## 📂 知识库目录导航

### 1. 🗄️ 数据提取基础
* [01-SQL 基础与进阶](./01-sql.md) - 多表连接(JOIN)、窗口函数、复杂嵌套查询

### 2. 🐼 数据清洗与核心计算
* [02-SQL/Pandas/Numpy 对照](./02-sql-pandas_numpy.md) - 数据过滤、分组聚合(groupby)、矩阵运算与 Polars 对比

### 3. 📉 数据可视化（从静态到交互）
* **参考网页：[The Python Graph Gallery](https://python-graph-gallery.com/)**
* [03-Matplotlib 基础绘图](./03-Visualization_matplotlib.md) - 折线图/柱状图模板、图表元素调优与标注(annotate)
* [04-Seaborn 高级统计可视化](./03-Visualization-seaborn.md) - 分布图(Displot)、相关性热力图(Heatmap)、分类图
* [05-Folium & 地图可视化](./03-Visualization-geo-maps.md) - 地理数据散点图、空间热力图 (含 Plotly Mapbox 推荐)
* [06-Plotly 动态交互图表](./03-Visualization-plotly.md) - 可缩放图表、动态滑块联动

### 4. 🤖 机器学习建模与效果评估
* [07-数据建模与模型测试](./04-Modeling.md) - Scikit-learn 特征工程、回归/分类模型评估 (结合 Seaborn 看模型残差与混淆矩阵)

### 5. 🌐 仪表盘与数据应用部署 (Web Apps)
* [08-Dash 网页应用开发](./05-Web-Apps-Dash.md) - 回调函数(Callbacks)逻辑、Dropdown/Graph 核心组件联动
* [09-Streamlit 快速原型应用](./05-Web-Apps-Streamlit.md) - 轻量级数据看板、一键部署脚本

# 数据分析项目完整 Checklist
## 从数据获取到部署 · 配套已有 Cheat Sheet 速查

> 这份 checklist 和六阶段流程图对应，每一项都标注了该用哪份 cheat sheet。
> 把它当作项目"体检表"——每完成一项打勾，避免漏掉关键步骤。

---

## 阶段 1：数据获取

> 📄 参考：`requests_api_cheatsheet.md`、`web_scraping_cheatsheet.md`

- [ ] 明确数据来源：本地文件 / 数据库 / 网页爬取 / API
- [ ] 如果是 API：先看文档，确认 base URL、认证方式、分页方式
- [ ] 如果是爬虫：先判断静态页面（BeautifulSoup）还是动态页面（Selenium）
- [ ] 检查 `robots.txt`，确认允许爬取
- [ ] API Key / Token 不要硬编码，存到环境变量或 `.env` 文件
- [ ] 写好请求函数：带 `timeout`、`headers`、错误重试
- [ ] 如果数据量大，先用小样本（如 `limit=10`）跑通流程，再放开抓全量
- [ ] 原始数据先完整保存一份（如 `raw_data.csv`），不要边抓边清洗
- [ ] 记录数据获取时间和来源 URL（方便复现和排查问题）

**常见坑**：
- 忘记 `time.sleep()`，被限流或封 IP
- API 返回的数字是字符串（如 `"5.1"`），后面忘记转换类型
- 分页没处理完整，数据少了一截没发现

---

## 阶段 2：数据清洗

> 📄 参考：`data_analyst_cheatsheet.md`（第2、6节）、`ml_workflow_cheatsheet_v2.md`（第3节）

### 基本检查

- [ ] `df.shape` —— 行列数是否符合预期
- [ ] `df.dtypes` —— 每列类型是否正确（数字列是不是被读成了字符串）
- [ ] `df.info()` —— 一次看清类型 + 缺失值情况
- [ ] `df.describe()` —— 数值列的 min/max 是否合理（有没有负数年龄、超大异常值）
- [ ] `df.duplicated().sum()` —— 是否有重复行

### 缺失值

- [ ] `df.isna().sum()` —— 每列缺失数量和缺失率
- [ ] 决定每列的处理方式：删除行 / 删除列 / 均值填充 / 中位数 / 众数 / 前向填充
- [ ] 缺失率超过 50% 的列，考虑是否直接删除
- [ ] 记录填充策略（哪列用了什么方法），后面写报告时要解释

### 异常值

- [ ] 用 IQR 或 Z-score 找出数值列的异常值
- [ ] 决定：删除 还是 `clip()` 截断（推荐 clip，不丢行）
- [ ] 异常值是不是真的"错"，还是业务上合理的极端值（如双十一销量）——别盲目删

### 类型与格式

- [ ] 日期列转换为 `datetime`（`pd.to_datetime`）
- [ ] 数字列确认是 `int`/`float`，不是字符串
- [ ] 类别列考虑转 `category` 类型节省内存
- [ ] 列名统一格式（小写、下划线，去空格）
- [ ] 文本列去除多余空格、统一大小写

**常见坑**：
- 在全量数据上计算均值再填充，造成数据泄露的隐患（虽然清洗阶段还没分训练测试集，但要记得这个原则会在后面用到）
- `clip()` 用错了上下界，把正常值也截断了
- 日期格式混乱（有的是 `2024-01-01`，有的是 `01/01/2024`），`pd.to_datetime` 解析出错

---

## 阶段 3：EDA 与特征工程

> 📄 参考：`data_analyst_cheatsheet.md`（第3、8节）、`seaborn_cheatsheet.md`、`ml_workflow_cheatsheet_v2.md`（第2、4节）

### EDA 探索

- [ ] 单变量分布：直方图 / KDE，看是否右偏（决定要不要 log 变换）
- [ ] 类别变量频次：`value_counts()`，看是否有某个类别占比极端（样本不均衡的信号）
- [ ] 数值列相关矩阵：`df.corr()` + 热力图，找出强相关特征
- [ ] 目标变量与各特征的关系：散点图（回归）/ 箱线图（分类）
- [ ] 用 `pairplot` 或多图看板快速鸟瞰所有变量两两关系
- [ ] 写下 2-3 个初步观察（如"折扣力度与复购率正相关"），后续分析围绕这些展开

### 特征工程

- [ ] 类别变量编码：
  - 无序类别 → `get_dummies(drop_first=True)`
  - 有序类别 → `OrdinalEncoder`
- [ ] 数值特征是否需要变换：
  - 右偏分布 → `log1p`
  - 量纲差异大 → 后面 Pipeline 里用 `StandardScaler`（注意：这一步在 split 之后做）
- [ ] 构造新特征：比率、时间特征（月份/星期/季度）、特征交叉
- [ ] 检查特征之间的共线性，相关性 > 0.9 的考虑删一个
- [ ] 确认目标变量（y）和特征（X）分开

**常见坑**：
- 在做 EDA 时就把 StandardScaler 在全量数据上 fit 了，留下泄露隐患
- One-hot 编码没用 `drop_first=True`，留下多重共线性
- 特征交叉做太多，特征数量爆炸，后面模型跑不动

---

## 阶段 4：模型训练

> 📄 参考：`ml_workflow_cheatsheet_v2.md`（第5节）

### 准备工作

- [ ] `train_test_split`：`test_size=0.2`, `random_state` 固定，分类任务加 `stratify=y`
- [ ] **此刻起，所有 `fit` 只能在 `X_train` 上做**
- [ ] 确认 X_train / X_test / y_train / y_test 的 shape 都对得上

### 选择任务类型

- [ ] 目标变量是连续值 → **回归**：`LinearRegression` / `Ridge` / `Lasso`
- [ ] 目标变量是类别 → **分类**：`LogisticRegression`
- [ ] 用 Pipeline 把 `Scaler` + （可选）`PolynomialFeatures` + 模型串起来

### 训练与初步检查

- [ ] `pipe.fit(X_train, y_train)`
- [ ] 回归：检查 `lr.coef_` 和 `lr.intercept_` 是否合理（系数符号是否符合常识）
- [ ] 分类：检查 `predict_proba` 输出是否合理（概率值在0-1之间）
- [ ] 画残差图（回归）或混淆矩阵（分类），初步判断模型是否work

**常见坑**：
- `scaler.fit_transform(X_test)`——测试集不能 fit，只能 transform
- 分类任务忘记检查样本是否均衡，直接用 accuracy 评估
- 回归目标变量做了 log 变换，预测后忘记 `expm1` 转换回来

---

## 阶段 5：评估与调参

> 📄 参考：`ml_workflow_cheatsheet_v2.md`（第7-11节）

### 基础评估

- [ ] 回归：`R²` / `RMSE` / `MAE`
- [ ] 分类：`混淆矩阵` / `Precision` / `Recall` / `F1` / `ROC-AUC`
- [ ] 对比 Train 和 Test 分数，判断过拟合/欠拟合
  - Train 高 Test 低 → 过拟合 → 加正则化、减少特征
  - Train 和 Test 都低 → 欠拟合 → 加特征、提高复杂度

### 交叉验证

- [ ] `cross_val_score(cv=5)`，看均值和标准差
- [ ] 标准差过大 → 模型对数据划分敏感，考虑增加数据或简化模型

### 正则化与调参

- [ ] 回归：尝试 `Ridge` / `Lasso`，对比效果
- [ ] 分类：调整 `class_weight`（样本不均衡时用 `'balanced'`）
- [ ] `GridSearchCV` 或 `RandomizedSearchCV` 搜索最优超参数
  - 回归常调：`alpha`、`degree`
  - 分类常调：`C`、`class_weight`
- [ ] 用 `grid.best_params_` 确认最优参数，`grid.best_score_` 看交叉验证得分

### ⚠️ Data Leakage 自查

- [ ] 所有 `scaler.fit()` / `encoder.fit()` 是否只在训练集上做了？
- [ ] 缺失值填充用的统计量（均值等）是否来自训练集？
- [ ] 有没有某个特征和目标变量相关性异常高（>0.99）？需要检查是否是目标的衍生变量
- [ ] 测试集分数是不是"好到不真实"？（如 R²=0.99，AUC=1.0）

**如果发现问题，回到阶段 3（特征工程）重新处理。** ← 这就是流程图里的循环箭头。

**常见坑**：
- GridSearchCV 的 `cv` 设太小（如 `cv=2`），结果不稳定
- 只看了一个指标（如只看 accuracy），样本不均衡时被误导
- 调参调到 Test 集上表现完美，但其实是在用测试集"偷偷"调参（应该用交叉验证调，最后才碰一次测试集）

---

## 阶段 6：可视化与部署

> 📄 参考：`dash_cheatsheet.md`、`streamlit_cheatsheet.md`、`seaborn_cheatsheet.md`（第11节）

### 静态报告图表

- [ ] 用 Seaborn 画出关键发现的图（分布对比、相关性、回归拟合）
- [ ] 图表加好标题、坐标轴标签、数据来源标注
- [ ] 配色统一，避免一张图里颜色太杂

### 交互式仪表盘（可选）

- [ ] 快速原型 / 个人项目 → Streamlit（学习曲线平缓，代码量少）
- [ ] 复杂多页面 / 精细布局 → Dash
- [ ] 仪表盘至少包含：核心指标卡、1-2 张交互图表、筛选控件
- [ ] 用 `@st.cache_data` / `@st.cache_resource` 避免重复计算（Streamlit）
- [ ] 测试边界情况：筛选后数据为空时会不会报错

### 模型持久化（如果项目包含建模）

- [ ] `joblib.dump(final_model, 'model.pkl')` 保存最终模型
- [ ] 用全量数据（`X`, `y`）重新训练最终模型
- [ ] 测试 `joblib.load()` 加载后能正常 `predict`

### 最终交付物检查

- [ ] 代码能从头跑到尾，不依赖未保存的中间变量
- [ ] README 写清楚：项目背景、数据来源、主要发现、如何运行
- [ ] 图表和数字在报告里能自洽（没有前后矛盾的结论）
- [ ] 如果是 GitHub portfolio 项目：敏感信息（API Key）不要提交进 repo

---

## 附：完整流程一页速查

```
1. 数据获取
   requests / BeautifulSoup / Selenium / PRAW
   ↓ 保存原始数据 raw_data.csv

2. 数据清洗
   df.info() → fillna / clip / drop_duplicates / astype
   ↓ 保存清洗后数据 clean_data.csv

3. EDA 与特征工程
   df.corr() / heatmap / 分布图
   get_dummies / log1p / 特征交叉
   ↓ X, y 分离

4. 模型训练
   train_test_split(stratify=y if 分类)
   Pipeline([Scaler, (Poly), Model])
   ↓ pipe.fit(X_train, y_train)

5. 评估与调参 ──┐
   R²/RMSE 或 Confusion Matrix/F1/AUC          │
   cross_val_score(cv=5)                       │ 效果不理想
   GridSearchCV(param_grid)                    │ ↻ 回到步骤3
   Data Leakage 自查 ───────────────────────────┘

6. 可视化与部署
   Seaborn 静态图 / Dash或Streamlit仪表盘
   joblib.dump(final_model, 'model.pkl')
```
