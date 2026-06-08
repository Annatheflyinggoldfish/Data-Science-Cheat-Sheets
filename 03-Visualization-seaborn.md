# 📊 Seaborn 高阶统计绘图速查表

> Seaborn 的精髓在于直接吃进完整的 Pandas DataFrame，用 `x='列名'`, `y='列名'` 即可极速出图。
> Seaborn 是建立在 Matplotlib 之上的。通常先用 Seaborn 一键换肤，然后用 Matplotlib 控制画布大小。

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np

# 🚀 换上高级皮肤（白色网格背景，自带低饱和度质感）
sns.set_theme(style="whitegrid", context="notebook")

# 1. 读入现实业务数据
df = pd.read_csv('your_actual_business_data.csv')
```

## 1.关系流派 (Relational)：探寻“变量 A 与变量 B 的联动轨迹”
### 高阶散点图 (sns.scatterplot) —— 观察个体分布与聚集聚类
```python
# df = pd.read_csv('user_behavior.csv')
# 业务场景：分析用户的“在线时长”与“消费金额”的关系，并按“性别”分类，用“订单量”控制点的大小
fig, ax = plt.subplots(figsize=(8, 5))
sns.scatterplot(
    data=df, 
    x='在线时长_分钟', 
    y='消费金额_元', 
    hue='用户性别',         # hue：分组上色（冷暖对比）
    size='历史订单量',      # size：让点的大小代表另一个业务维度
    sizes=(20, 200),        # 控制点的最小和最大尺寸范围
    alpha=0.7, 
    palette='muted',
    ax=ax
)
ax.set_title('用户活跃时长与消费金额多维关联散点图')
```

### 统计折线图 (sns.lineplot) —— 自动聚合多店铺走势与 95% 置信区间
```python
# df = pd.read_csv('store_traffic.csv')
# 业务场景：分析各月“门店客流量”趋势。表中同一月份有多个分店数据，Seaborn 会自动算平均值并画出阴影（置信区间）
fig, ax = plt.subplots(figsize=(8, 4))
sns.lineplot(
    data=df, 
    x='统计月份', 
    y='客流量_人次', 
    hue='地区分部',         # 自动按“华东/华南”画出两条趋势线
    style='地区分部',      # 自动为不同线分配“实线/虚线”样式，方便打印黑白报表
    marker='o', 
    ax=ax
)
ax.set_title('2026年度各地区门店月度客流走势及波动范围')
```

## 2.分布流派 (Distribution)：摸清“人群/指标在哪个区间最扎堆”
### 
```python
# df = pd.read_csv('order_details.csv')
# 业务场景：分析单笔“订单金额”的分布。
fig, ax = plt.subplots(figsize=(8, 4))
sns.histplot(
    data=df, 
    x='订单金额_元', 
    hue='是否首单',         # 叠加对比：新老客的客单价分布有什么不同
    multiple='stack',       # stack: 柱状图向上堆叠；dodge: 并列排放
    bins=30,                # 把金额切分成 30 个连续区间
    kde=True,               # 🌟 顺手画出平滑的核密度曲线（KDE）
    palette='pastel',
    ax=ax
)
ax.set_title('新老客单笔订单金额分布直方图')
```

### 核密度估计图 (sns.kdeplot) —— 纯净的概率密度“山峰图”
```python
# df = pd.read_csv('delivery_logs.csv')
# 业务场景：脱离柱子，只看“配送延迟（Delivery Latency）”的绝对密度曲线
fig, ax = plt.subplots(figsize=(8, 4))
sns.kdeplot(
    data=df, 
    x='配送耗时_小时', 
    hue='快递公司', 
    fill=True,              # 给曲线下方填充半透明颜色，更具视觉冲击力
    alpha=0.3,
    linewidth=2,
    ax=ax
)
ax.set_title('不同快递服务商配送时效密度（KDE）对比')
```

## 1.分类流派 (Categorical)：对比“不同分组（品类/地域）的指标差异”
### 置信度条形图 (sns.barplot) —— 带有误差线的高级均值对比
```python
# df = pd.read_csv('product_performance.csv')
# 业务场景：对比各个“产品品类”的“平均利润率”，顶端的小黑线代表误差范围（越短说明数据越稳定）
fig, ax = plt.subplots(figsize=(8, 4))
sns.barplot(
    data=df, 
    x='产品品类', 
    y='利润率_百分比', 
    hue='季度', 
    errorbar='ci',          # 自动计算置信区间误差线
    palette='vlag',
    ax=ax
)
```

### 箱线图 (sns.boxplot) —— 极速抓取各部门的异常高消费/离群点
```python
# df = pd.read_csv('employee_reimbursement.csv')
# 业务场景：看各部门“差旅报销金额”的分布，揪出超出正常范围的“报销异常值”
fig, ax = plt.subplots(figsize=(8, 4))
sns.boxplot(
    data=df, 
    x='所属部门', 
    y='报销金额_元', 
    flierprops={'markerfacecolor': 'red', 'marker': 'D'}, # 把异常值点变成显眼的红方块
    ax=ax
)
```

### 小提琴图 (sns.violinplot) —— 均值与密度的“降维打击”合体
```python
# df = pd.read_csv('app_metrics.csv')
# 业务场景：观察不同“手机系统”用户在 App 内的“停留时长”分布，对半拆分“是否付费”
fig, ax = plt.subplots(figsize=(8, 5))
sns.violinplot(
    data=df, 
    x='手机系统', 
    y='停留时长_分钟', 
    hue='是否付费', 
    split=True,             # 🌟 绝活：把 Paid/Unpaid 对半拼成一个小提琴，节省空间，对比极强
    inner='quart',          # 在小提琴内部画出四分位数虚线
    palette='muted',
    ax=ax
)
```

## 1.回归分析流派 (Regression)：测算“未来趋势与因果预测”
### 基础回归图 (sns.regplot) —— 快速探查两指标的线性回归趋势
```python
# df = pd.read_csv('marketing_campaigns.csv')
# 业务场景：研究“促销打折力度”和“门店复购率”之间是否存在线性因果关系
fig, ax = plt.subplots(figsize=(8, 5))
sns.regplot(
    data=df, 
    x='折扣力度_折', 
    y='门店复购率_百分比',
    scatter_kws={'s': 50, 'alpha': 0.6, 'color': '#4A7BB0'}, # 控制底层散点的样式
    line_kws={'color': '#D16666', 'linewidth': 2.5},        # 控制那条核心拟合线的样式
    ci=95,                                                  # 自动绘制拟合线的 95% 浅色置信区间带
    ax=ax
)
ax.set_title('促销折扣力度与客户复购率回归分析图（含95%置信带）')
```

### 多子图矩阵回归图 (sns.lmplot) —— 拆分维度看“回归线的倾斜度分化”
> 注意：lmplot 是 画布级（Figure-level） 函数，不能写 ax=ax。它的绝活是可以用 col 或 row 参数，把不同的分类直接拆成一排独立的回归图。
```python
# df = pd.read_csv('sales_pipeline.csv')
# 业务场景：看“销售跟进次数”和“最终签约金额”的因果关系，并横向拆开对比“不同业务线”的差异
g = sns.lmplot(
    data=df, 
    x='跟进次数', 
    y='签约金额_万元', 
    col='业务线',           # 🌟 核心：有几个业务线，就横向画几个子图，各自拥有独立的回归线！
    hue='业务线', 
    palette='Set1',
    height=5,               # 控制单个子图的高度（英寸）
    aspect=1.2              # 控制单个子图的宽高比
)
# 画布级函数的标题控制稍微特殊，需要调用 suptitle
g.fig.suptitle('不同业务线：客户跟进频次与签约金额拟合回归矩阵', y=1.05, fontweight='bold')
plt.show()
```
