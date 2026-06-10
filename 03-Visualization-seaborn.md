# Seaborn 完整手册


> **核心思路**：Seaborn 直接吃进 Pandas DataFrame，用列名驱动绘图，
> 自动完成分组、聚合、置信区间计算。
>
> - **Seaborn** 负责：统计图表逻辑（分组着色、置信带、回归线）
> - **Matplotlib** 负责：画布大小、标题、坐标轴、字体、细节微调
>
> 两者的分工：先用 Seaborn 一键出图，再用 Matplotlib 精调每一个细节。

```
函数分两类，决定了能不能用 ax= 参数：

Axes-level（画板级）→ 支持 ax=ax，可以嵌进多子图
  sns.scatterplot / lineplot / histplot / kdeplot
  sns.boxplot / violinplot / barplot / stripplot / swarmplot
  sns.regplot / residplot / heatmap

Figure-level（画布级）→ 不支持 ax=，自己管理画布，用 col/row 分面
  sns.relplot / displot / catplot / lmplot / pairplot / jointplot / clustermap
```

---

## 目录

1. [环境配置与主题](#1-环境配置与主题)
2. [关系流派：散点图 & 折线图](#2-关系流派散点图--折线图)
3. [分布流派：直方图 & KDE & ECDF](#3-分布流派直方图--kde--ecdf)
4. [分类流派：箱线图 & 小提琴图 & 条形图 & 点图](#4-分类流派箱线图--小提琴图--条形图--点图)
5. [回归分析流派：regplot & lmplot & residplot](#5-回归分析流派regplot--lmplot--residplot)
6. [矩阵流派：热力图 & 聚类图](#6-矩阵流派热力图--聚类图)
7. [多变量探索：pairplot & jointplot](#7-多变量探索pairplot--jointplot)
8. [分面网格：FacetGrid & catplot & relplot](#8-分面网格facetgrid--catplot--relplot)
9. [样式精调全指南](#9-样式精调全指南)
10. [多子图排版](#10-多子图排版)
11. [实战完整案例](#11-实战完整案例)

---

## 1. 环境配置与主题

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker   # 坐标轴刻度精调
import seaborn as sns
import pandas as pd
import numpy as np

# ── 全局主题设置 ──────────────────────────────
sns.set_theme(
    style='whitegrid',     # 背景风格（见下方对照表）
    context='notebook',    # 元素比例（见下方对照表）
    palette='muted',       # 默认配色
    font_scale=1.1         # 全局字号缩放（>1放大，<1缩小）
)

# style 选项（背景风格）：
# 'white'      → 纯白背景，无网格，最简洁，适合出版/报告
# 'whitegrid'  → 白色背景 + 灰色网格（推荐，商务常用）
# 'darkgrid'   → 深色背景 + 网格，视觉冲击强
# 'ticks'      → 只有坐标轴刻度线，极简风格
# 'dark'       → 深色背景，无网格

# context 选项（控制字体、线条、点的整体比例）：
# 'paper'      → 最小，适合嵌入文章
# 'notebook'   → 默认，适合 Jupyter
# 'talk'       → 较大，适合 PPT 演示
# 'poster'     → 最大，适合海报展示

# ── 只对某一张图临时切换风格 ──────────────────
with sns.axes_style('white'):          # with 块内用 white 风格
    fig, ax = plt.subplots()
    sns.histplot(data=df, x='col', ax=ax)

# ── 中文字体设置（必须加，否则中文变方块）──────
plt.rcParams['font.sans-serif'] = ['SimHei']      # Windows
# plt.rcParams['font.sans-serif'] = ['PingFang SC'] # macOS
plt.rcParams['axes.unicode_minus'] = False         # 负号正常显示
```

### 配色速查

```python
# ── 定性配色（类别变量，颜色间有明显区别）──────
'muted'        # 低饱和度，商务友好（推荐默认）
'pastel'       # 粉嫩系，温和
'deep'         # 饱和度高，对比强
'bright'       # 鲜艳，适合演示
'colorblind'   # 对色盲友好（重要报告推荐）
'Set2'         # Matplotlib 定性配色，8色
'tab10'        # Tableau 经典10色

# ── 连续配色（数值变量，表示大小梯度）──────────
'Blues'        # 蓝色渐变
'YlOrRd'       # 黄橙红（热力图常用）
'viridis'      # 绿黄渐变，对色盲友好
'rocket'       # Seaborn 原生，深红到浅黄
'mako'         # Seaborn 原生，深蓝到浅绿

# ── 发散配色（有中心值的数据，如相关矩阵）──────
'RdBu_r'       # 红蓝对称（_r 表示反转）
'coolwarm'     # 冷暖对称
'vlag'         # Seaborn 原生发散配色

# 手动指定颜色列表
palette = ['#4E79A7', '#F28E2B', '#E15759', '#76B7B2', '#59A14F']
# 上面是 Tableau 经典5色，商务报告常用

# 查看某个配色长什么样
sns.palplot(sns.color_palette('muted', 8))
plt.show()
```

---

## 2. 关系流派：散点图 & 折线图

> **用途**：探索两个连续变量之间的关系、趋势、聚类。

### 2.1 散点图 scatterplot

```python
fig, ax = plt.subplots(figsize=(9, 5))

sns.scatterplot(
    data=df,
    x='在线时长_分钟',
    y='消费金额_元',
    hue='用户性别',             # 按类别分色
    style='用户性别',           # 按类别换形状（配合 hue，黑白打印仍可区分）
    size='历史订单量',          # 点大小代表第三个维度
    sizes=(30, 300),            # 最小点和最大点的面积范围（不是直径）
    alpha=0.7,                  # 透明度：重叠多时调低，稀疏时调高
    palette='muted',
    edgecolor='white',          # 点的描边色（'white'/'none'/具体颜色）
    linewidth=0.5,              # 描边宽度
    ax=ax
)

# ── 精调 ────────────────────────────────────
ax.set_title('用户活跃时长与消费金额多维关联', fontsize=14, fontweight='bold', pad=12)
ax.set_xlabel('在线时长（分钟）', fontsize=11)
ax.set_ylabel('消费金额（元）',   fontsize=11)
ax.legend(
    title='用户性别',
    title_fontsize=10,
    fontsize=9,
    loc='upper left',           # 图例位置
    framealpha=0.8,             # 图例背景透明度
    edgecolor='gray'
)
# 添加参考线（如均值线）
ax.axhline(df['消费金额_元'].mean(), color='red', linestyle='--',
           linewidth=1.2, alpha=0.6, label='消费均值')
ax.axvline(df['在线时长_分钟'].mean(), color='gray', linestyle=':',
           linewidth=1, alpha=0.6)

plt.tight_layout()
plt.show()
```

### 2.2 折线图 lineplot

```python
fig, ax = plt.subplots(figsize=(10, 5))

sns.lineplot(
    data=df,
    x='统计月份',
    y='客流量_人次',
    hue='地区分部',             # 自动按类别画多条线
    style='地区分部',           # 不同线型（实线/虚线/点线）
    marker='o',                 # 数据点标记形状
                                # 'o'=圆 's'=方 '^'=三角 'D'=菱形 'x'=叉
    markersize=7,               # 标记点大小
    linewidth=2,                # 线宽
    dashes=True,                # True=自动分配不同虚线；False=全用实线
    errorbar='ci',              # 置信区间类型：
                                # 'ci'=95%置信区间（默认）
                                # 'sd'=标准差范围
                                # 'se'=标准误差
                                # None=不画置信带
    err_kws={'alpha': 0.15},    # 置信带的透明度（越低越淡）
    palette='deep',
    ax=ax
)

# ── 精调 ────────────────────────────────────
ax.set_title('各地区门店月度客流趋势（含95%置信区间）', fontsize=14, fontweight='bold')

# X轴刻度旋转（月份标签多时防重叠）
ax.tick_params(axis='x', labelrotation=45, labelsize=9)

# Y轴加千分位格式（如 10,000 而不是 10000）
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'{x:,.0f}'))

# 在最后一个数据点旁边加文字标注（避免图例不清楚时）
for region in df['地区分部'].unique():
    last = df[df['地区分部'] == region].iloc[-1]
    ax.text(last['统计月份'], last['客流量_人次'] + 50,
            region, fontsize=9, va='bottom')
ax.get_legend().remove()  # 用了文字标注就可以删掉图例

plt.tight_layout()
plt.show()

# ── 高阶用法：Figure-level relplot（可以用 col 分面）──
g = sns.relplot(
    data=df,
    x='统计月份', y='客流量_人次',
    hue='地区分部', col='年份',    # 按年份横向拆开成多个子图
    kind='line',                   # 'line' 或 'scatter'
    height=4, aspect=1.2,
    col_wrap=3                     # 每行最多3个子图，超过自动换行
)
g.set_titles("{col_name} 年")      # 子图标题格式
g.set_axis_labels("月份", "客流量")
g.fig.suptitle('分年度各地区客流趋势', y=1.02, fontweight='bold')
```

---

## 3. 分布流派：直方图 & KDE & ECDF

> **用途**：了解单个变量的分布形态、集中趋势、长尾情况。

### 3.1 直方图 histplot

```python
fig, ax = plt.subplots(figsize=(9, 5))

sns.histplot(
    data=df,
    x='订单金额_元',
    hue='是否首单',
    multiple='layer',       # 多组叠加方式：
                            # 'layer'   = 重叠透明（默认）
                            # 'dodge'   = 并排
                            # 'stack'   = 堆叠
                            # 'fill'    = 堆叠且归一化到1（看比例）
    bins=40,                # 分桶数量（或用 binwidth=50 指定每桶宽度）
    kde=True,               # 同时叠加 KDE 曲线
    kde_kws={               # KDE 曲线的样式
        'linewidth': 2,
        'cut': 3            # KDE曲线延伸到数据边界外几个标准差（默认3）
    },
    stat='density',         # 纵轴含义：
                            # 'count'   = 频次（默认）
                            # 'density' = 概率密度（面积=1）
                            # 'percent' = 百分比
                            # 'probability' = 概率
    palette='pastel',
    edgecolor='white',      # 柱子描边色（分开柱子用）
    linewidth=0.5,
    alpha=0.7,
    ax=ax
)

ax.set_title('新老客单笔订单金额分布对比', fontsize=14, fontweight='bold')
ax.xaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'¥{x:,.0f}'))

# 在图上标注统计数字
mean_val = df['订单金额_元'].mean()
ax.axvline(mean_val, color='red', linestyle='--', linewidth=1.5)
ax.text(mean_val + 20, ax.get_ylim()[1] * 0.9,
        f'均值\n¥{mean_val:.0f}', color='red', fontsize=10)

plt.tight_layout()
plt.show()
```

### 3.2 核密度图 kdeplot

```python
fig, ax = plt.subplots(figsize=(9, 5))

sns.kdeplot(
    data=df,
    x='配送耗时_小时',
    hue='快递公司',
    fill=True,              # 曲线下方填色
    alpha=0.25,             # 填充透明度（太深会遮挡）
    linewidth=2.5,          # 曲线线宽
    bw_adjust=0.8,          # 带宽调整系数：
                            # <1 = 曲线更"抖"，细节更多
                            # >1 = 曲线更"平滑"
    cut=0,                  # 0 = 曲线在数据范围内截断（不延伸到负值）
    common_norm=False,      # False = 各组独立归一化（各自面积=1）
                            # True  = 全体数据归一化（面积之和=1）
    palette='Set2',
    ax=ax
)

ax.set_title('不同快递公司配送时效密度分布对比', fontsize=14, fontweight='bold')
ax.set_xlabel('配送耗时（小时）')

# 二维 KDE（热力密度图，看两变量联合分布）
fig, ax = plt.subplots(figsize=(7, 6))
sns.kdeplot(
    data=df,
    x='运费_元', y='配送耗时_小时',
    fill=True,
    thresh=0.05,            # 低于此概率密度的区域不填色（去掉边缘噪声）
    levels=10,              # 等高线层数
    cmap='Blues',
    ax=ax
)
sns.scatterplot(data=df, x='运费_元', y='配送耗时_小时',
                color='black', alpha=0.3, s=15, ax=ax)  # 叠加原始点
```

### 3.3 ECDF 图（经验累积分布）

```python
# ECDF = Empirical Cumulative Distribution Function
# 纵轴含义：x 轴某个值以下的数据占总体的比例
# 用途：直观回答"有多少订单在 X 小时内完成配送"

fig, ax = plt.subplots(figsize=(9, 5))

sns.ecdfplot(
    data=df,
    x='配送耗时_小时',
    hue='快递公司',
    linewidth=2,
    ax=ax
)

# 加参考线：读出"80%订单在多少小时内完成"
ax.axhline(0.8, color='red', linestyle='--', linewidth=1, alpha=0.7)
ax.text(ax.get_xlim()[1] * 0.02, 0.82, '80% 订单', color='red', fontsize=10)

ax.set_title('各快递公司配送耗时累积分布曲线（ECDF）', fontsize=14, fontweight='bold')
ax.set_ylabel('累积比例')
ax.yaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))

plt.tight_layout()
plt.show()
```

### 3.4 Figure-level displot

```python
# displot 可以用 col/row 分面，适合同时看多组分布
g = sns.displot(
    data=df,
    x='订单金额_元',
    col='季度',             # 按季度横向排成4个子图
    hue='是否首单',
    kind='kde',             # 'hist' / 'kde' / 'ecdf'
    fill=True,
    height=4, aspect=1.1,
    col_wrap=2              # 每行最多2个，自动换行
)
g.set_titles("Q{col_name}")
g.set_axis_labels("订单金额（元）", "密度")
g.fig.suptitle('各季度新老客订单金额分布', y=1.02, fontweight='bold')
```

---

## 4. 分类流派：箱线图 & 小提琴图 & 条形图 & 点图

> **用途**：对比不同类别之间某个数值指标的差异、分布、离散程度。

### 4.1 箱线图 boxplot

```python
fig, ax = plt.subplots(figsize=(10, 5))

sns.boxplot(
    data=df,
    x='所属部门',
    y='报销金额_元',
    hue='季度',             # 按季度分色，同一部门画多个箱
    order=['销售部', '研发部', '市场部', '运营部'],  # 手动指定 X 轴顺序
    hue_order=['Q1', 'Q2', 'Q3', 'Q4'],
    width=0.6,              # 箱子宽度（0-1）
    linewidth=1.2,          # 箱线宽度
    flierprops={            # 异常值点的样式
        'marker': 'D',      # 'o'圆 'D'菱形 '+'加号 'x'叉号
        'markerfacecolor': 'red',
        'markeredgecolor': 'darkred',
        'markersize': 5,
        'alpha': 0.7
    },
    medianprops={           # 中位数线的样式
        'color': 'red',
        'linewidth': 2
    },
    boxprops={'alpha': 0.8},
    palette='pastel',
    ax=ax
)

ax.set_title('各部门季度差旅报销金额分布（含异常值检测）', fontsize=14, fontweight='bold')
ax.tick_params(axis='x', labelrotation=15)
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'¥{x:,.0f}'))

plt.tight_layout()
plt.show()
```

### 4.2 小提琴图 violinplot

```python
fig, ax = plt.subplots(figsize=(10, 6))

sns.violinplot(
    data=df,
    x='手机系统',
    y='停留时长_分钟',
    hue='是否付费',
    split=True,             # 🌟 对半拼：左半边 vs 右半边，节省空间，对比强
                            # 注意：split=True 时 hue 只能有2个类别
    inner='quart',          # 小提琴内部显示：
                            # 'quart'  = 四分位数虚线（推荐）
                            # 'box'    = 嵌入小箱线图
                            # 'stick'  = 每个数据点一根竖线
                            # 'point'  = 每个数据点一个点
                            # None     = 不显示内部
    density_norm='width',   # 'width' = 所有组最大宽度相同（便于对比形状）
                            # 'area'  = 所有组面积相同（反映样本量）
                            # 'count' = 宽度正比于样本量
    linewidth=1.2,
    palette='muted',
    ax=ax
)

ax.set_title('付费 vs 免费用户在不同手机系统上的停留时长分布', fontsize=13, fontweight='bold')

plt.tight_layout()
plt.show()

# ── 组合技：boxplot + stripplot（箱线图叠原始数据点）──
fig, ax = plt.subplots(figsize=(10, 5))
sns.boxplot(data=df, x='产品品类', y='销售额', palette='pastel',
            width=0.5, ax=ax)
sns.stripplot(data=df, x='产品品类', y='销售额',
              color='black', alpha=0.3, size=3,
              jitter=True,          # 水平随机抖动，防止点重叠
              ax=ax)
ax.set_title('各品类销售额分布（箱线 + 原始数据点叠加）')
```

### 4.3 条形图 barplot

```python
fig, ax = plt.subplots(figsize=(10, 5))

sns.barplot(
    data=df,
    x='产品品类',
    y='利润率_百分比',
    hue='季度',
    order=df.groupby('产品品类')['利润率_百分比'].mean()
            .sort_values(ascending=False).index,  # 按均值降序排列
    errorbar='ci',          # 误差线类型：
                            # 'ci' = 95%置信区间（默认）
                            # 'sd' = 标准差
                            # 'se' = 标准误差
                            # None = 不画误差线
    capsize=0.1,            # 误差线顶端横帽的宽度
    linewidth=1,            # 柱子边框线宽（0=无边框）
    edgecolor='white',
    palette='vlag',
    ax=ax
)

# 在柱子上方标注数值
for container in ax.containers:
    ax.bar_label(container, fmt='%.1f%%', fontsize=8, padding=2)

ax.set_title('各品类季度平均利润率对比（含95%置信区间）', fontsize=14, fontweight='bold')
ax.tick_params(axis='x', labelrotation=15)

plt.tight_layout()
plt.show()
```

### 4.4 点图 pointplot & stripplot & swarmplot

```python
# pointplot：只画均值点和误差线，比 barplot 更简洁，适合对比趋势
fig, ax = plt.subplots(figsize=(9, 5))
sns.pointplot(
    data=df,
    x='季度', y='满意度评分',
    hue='产品线',
    dodge=0.3,              # 分组时横向错开的距离
    markers=['o', 's', '^'], # 各组标记形状
    linestyles=['-', '--', ':'],
    capsize=0.08,
    palette='deep',
    ax=ax
)
ax.set_title('各产品线季度用户满意度均值趋势')

# stripplot：把所有原始数据点画出来，看分布密度
fig, ax = plt.subplots(figsize=(8, 5))
sns.stripplot(
    data=df,
    x='部门', y='绩效分',
    hue='级别',
    jitter=0.2,             # 水平抖动量（0=不抖，0.4=较大抖动）
    alpha=0.6,
    size=5,
    palette='Set2',
    ax=ax
)

# swarmplot：和 stripplot 类似，但用算法避免点重叠（数据量小时更好看）
sns.swarmplot(data=df, x='部门', y='绩效分', color='black',
              alpha=0.5, size=3, ax=ax)

# countplot：计数条形图（不需要 y，自动统计频次）
fig, ax = plt.subplots(figsize=(8, 4))
sns.countplot(
    data=df,
    x='评分',
    hue='是否复购',
    order=[5, 4, 3, 2, 1],  # 手动指定顺序
    palette='pastel',
    ax=ax
)
ax.bar_label(ax.containers[0], fontsize=9)  # 标注第一组的数量
ax.bar_label(ax.containers[1], fontsize=9)
```

### 4.5 Figure-level catplot（分面分类图）

```python
# catplot 是所有分类图的 Figure-level 版本，可以用 col/row 分面
g = sns.catplot(
    data=df,
    x='季度', y='销售额',
    col='地区',             # 按地区横向拆成多个子图
    kind='box',             # 'box'/'violin'/'bar'/'point'/'strip'/'swarm'/'count'
    height=4, aspect=1.2,
    palette='pastel',
    col_wrap=3
)
g.set_titles("{col_name} 地区")
g.set_axis_labels("季度", "销售额")
g.fig.suptitle('各地区季度销售额分布', y=1.02, fontweight='bold')
```

---

## 5. 回归分析流派：regplot & lmplot & residplot

> **用途**：探索两变量的线性/非线性关系，可视化回归结果，检验模型残差。

### 5.1 regplot（基础回归图）

```python
fig, ax = plt.subplots(figsize=(9, 6))

sns.regplot(
    data=df,
    x='折扣力度_折',
    y='门店复购率_百分比',
    order=1,                # 多项式阶数：1=线性，2=二次曲线，3=三次曲线
    ci=95,                  # 置信区间宽度（0-100，None=不画）
    scatter_kws={           # 散点样式字典
        's': 60,            # 点面积
        'alpha': 0.5,
        'color': '#4A7BB0',
        'edgecolor': 'white',
        'linewidth': 0.5
    },
    line_kws={              # 回归线样式字典
        'color': '#D16666',
        'linewidth': 2.5,
        'linestyle': '-'
    },
    ax=ax
)

# 在图上标注相关系数和 p 值
from scipy import stats
r, p = stats.pearsonr(df['折扣力度_折'].dropna(), df['门店复购率_百分比'].dropna())
ax.text(0.05, 0.92, f'r = {r:.3f}，p = {p:.4f}',
        transform=ax.transAxes,    # 坐标系为轴坐标（0-1），不受数据范围影响
        fontsize=11,
        bbox=dict(boxstyle='round,pad=0.4', facecolor='lightyellow',
                  edgecolor='gray', alpha=0.8))

ax.set_title('促销折扣力度与客户复购率回归分析（含95%置信带）',
             fontsize=13, fontweight='bold')
plt.tight_layout()
plt.show()

# ── 非线性回归（order=2 拟合抛物线）──
fig, ax = plt.subplots(figsize=(9, 5))
sns.regplot(data=df, x='广告投入', y='销售额', order=2,
            scatter_kws={'alpha': 0.4, 'color': 'steelblue'},
            line_kws={'color': 'darkred'}, ax=ax)
ax.set_title('广告投入与销售额（二次曲线拟合）')
```

### 5.2 lmplot（分面回归图）

```python
# lmplot = Figure-level，不用 ax=，用 col/row/hue 分面
g = sns.lmplot(
    data=df,
    x='跟进次数',
    y='签约金额_万元',
    col='业务线',           # 横向按业务线拆开，各自独立回归线
    hue='业务线',
    ci=95,
    scatter_kws={'alpha': 0.4, 's': 40},
    line_kws={'linewidth': 2},
    palette='Set1',
    height=5,
    aspect=1.1
)
g.set_titles("{col_name} 业务线", fontsize=11)
g.set_axis_labels("客户跟进次数", "签约金额（万元）")
g.fig.suptitle('不同业务线：跟进频次与签约金额回归对比',
               y=1.04, fontsize=14, fontweight='bold')
plt.show()
```

### 5.3 residplot（残差诊断图）

```python
# 残差图：检验线性回归假设是否成立
# 理想状态：残差随机分布在 y=0 两侧，无规律
# 如果有明显规律（弯曲、漏斗形）→ 模型有问题

fig, ax = plt.subplots(figsize=(9, 5))
sns.residplot(
    data=df,
    x='折扣力度_折',
    y='门店复购率_百分比',
    lowess=True,            # True = 画出 LOWESS 平滑曲线，辅助判断是否有系统性偏差
    scatter_kws={'alpha': 0.4, 'color': 'steelblue', 's': 40},
    line_kws={'color': 'red', 'linewidth': 2},
    ax=ax
)
ax.axhline(0, color='black', linestyle='--', linewidth=1, alpha=0.5)
ax.set_title('回归残差诊断图（LOWESS平滑）', fontsize=13, fontweight='bold')
ax.set_xlabel('折扣力度（折）')
ax.set_ylabel('残差')
plt.tight_layout()
```

---

## 6. 矩阵流派：热力图 & 聚类图

> **用途**：可视化相关矩阵、混淆矩阵、交叉频次表等二维矩阵数据。

### 6.1 heatmap 热力图

```python
# ── 相关矩阵热力图（EDA最常用）──────────────
corr = df.select_dtypes('number').corr()  # 只取数值列

# 生成上三角遮罩（避免重复显示对称信息）
mask = np.triu(np.ones_like(corr, dtype=bool))

fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(
    corr,
    mask=mask,              # 上三角遮罩（只显示下三角）
    annot=True,             # True = 在格子里显示数值
    fmt='.2f',              # 数值格式（保留2位小数）
    cmap='RdBu_r',          # 发散配色，0=白，正值=红，负值=蓝
    vmin=-1, vmax=1,        # 固定色阶范围（相关系数范围就是[-1,1]）
    center=0,               # 色阶中心值（0对应白色）
    square=True,            # True = 每个格子强制为正方形
    linewidths=0.5,         # 格子之间的间隔线宽
    linecolor='white',      # 间隔线颜色
    cbar_kws={              # 颜色条（colorbar）的样式
        'shrink': 0.8,      # 颜色条相对高度（<1缩短）
        'aspect': 20,       # 颜色条宽窄比（越大越细）
        'label': '相关系数'
    },
    annot_kws={'size': 9},  # 格内数值的字号
    ax=ax
)
ax.set_title('特征相关性矩阵（下三角）', fontsize=14, fontweight='bold', pad=15)

# 旋转 X 轴标签防止重叠
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, ha='right', fontsize=9)
ax.set_yticklabels(ax.get_yticklabels(), rotation=0,  fontsize=9)

plt.tight_layout()
plt.show()

# ── 混淆矩阵热力图（分类模型评估）─────────
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
labels = ['负例', '正例']

fig, ax = plt.subplots(figsize=(5, 4))
sns.heatmap(
    cm, annot=True, fmt='d',    # fmt='d' = 整数格式
    cmap='Blues',
    xticklabels=labels,
    yticklabels=labels,
    linewidths=1, linecolor='white',
    ax=ax
)
ax.set_title('混淆矩阵', fontweight='bold')
ax.set_xlabel('预测值')
ax.set_ylabel('实际值')

# ── 数据透视表热力图（业务分析常用）──────────
pivot = df.pivot_table(values='销售额', index='地区', columns='季度', aggfunc='sum')
fig, ax = plt.subplots(figsize=(8, 5))
sns.heatmap(
    pivot,
    annot=True, fmt=',.0f',     # 千分位格式
    cmap='YlOrRd',
    linewidths=0.5,
    ax=ax
)
ax.set_title('各地区季度销售额汇总热力图')
```

### 6.2 clustermap（层次聚类热力图）

```python
# clustermap = heatmap + 层次聚类树状图
# 自动把相似的行/列聚在一起，发现数据中的自然分组

g = sns.clustermap(
    corr,
    cmap='RdBu_r',
    vmin=-1, vmax=1, center=0,
    annot=True, fmt='.2f',
    figsize=(10, 10),
    row_cluster=True,           # 对行做层次聚类
    col_cluster=True,           # 对列做层次聚类
    dendrogram_ratio=0.15,      # 树状图占总宽度的比例
    linewidths=0.3,
    method='ward'               # 聚类算法：'ward'/'complete'/'average'/'single'
)
g.fig.suptitle('特征聚类热力图', y=1.01, fontweight='bold')
```

---

## 7. 多变量探索：pairplot & jointplot

> **用途**：快速鸟瞰多个变量两两之间的关系，是 EDA 阶段的利器。

### 7.1 pairplot

```python
# 对角线 = 每个变量自身的分布
# 非对角线 = 两变量的散点图（或其他图类型）

g = sns.pairplot(
    df[['价格', '销量', '利润率', '用户评分', '品类']],  # 只取关心的列
    hue='品类',                 # 按品类分色
    diag_kind='kde',            # 对角线图类型：'kde'/'hist'/'auto'
    plot_kws={                  # 非对角线散点的样式
        'alpha': 0.5,
        's': 30,
        'edgecolor': 'none'
    },
    diag_kws={                  # 对角线图的样式
        'fill': True,
        'alpha': 0.4
    },
    palette='Set2',
    corner=True                 # True = 只画下三角，避免重复
)
g.fig.suptitle('核心业务指标两两关系矩阵', y=1.02, fontweight='bold')

# 在对角线上叠加 rug（数据密度须线）
g.map_diag(sns.rugplot, color='gray', alpha=0.3)
```

### 7.2 jointplot

```python
# jointplot：单对变量的深度联合分析
# 中间大图 = 两变量关系；上方/右侧边图 = 各自分布

g = sns.jointplot(
    data=df,
    x='广告费用_万元',
    y='新增用户数',
    kind='scatter',             # 中间图类型：
                                # 'scatter'  = 散点（默认）
                                # 'kde'      = 二维密度图
                                # 'hist'     = 二维直方图
                                # 'hex'      = 六边形分箱（数据量大时好用）
                                # 'reg'      = 散点+回归线（推荐）
                                # 'resid'    = 残差图
    marginal_kws={              # 边缘分布图的参数
        'bins': 25,
        'fill': True
    },
    joint_kws={                 # 中间图的参数（针对 scatter）
        'alpha': 0.5,
        's': 40
    },
    height=7,
    ratio=4                     # 中间图占总高度的比例（越大中间图越大）
)

# 在 jointplot 上添加回归线（scatter kind 时）
g.plot_joint(sns.regplot,
             scatter_kws={'alpha': 0.3},
             line_kws={'color': 'red'})

# 标注相关系数
from scipy import stats
r, p = stats.pearsonr(df['广告费用_万元'], df['新增用户数'])
g.ax_joint.text(0.05, 0.92, f'r = {r:.3f}\np = {p:.4f}',
                transform=g.ax_joint.transAxes,
                fontsize=11,
                bbox=dict(boxstyle='round', facecolor='lightyellow', alpha=0.8))

g.fig.suptitle('广告投入与新增用户数联合分布分析', y=1.01, fontweight='bold')
plt.show()
```

---

## 8. 分面网格：FacetGrid & catplot & relplot

> **用途**：把同一类图按某个分类变量拆成多个子图，高效对比多组数据。

```python
# ── FacetGrid：最灵活的分面方式 ───────────────
g = sns.FacetGrid(
    df,
    col='地区',             # 横向分面变量
    row='季度',             # 纵向分面变量（可省略）
    hue='品类',             # 颜色分组
    height=4, aspect=1.2,
    col_wrap=3,             # 每行最多3列（row 和 col_wrap 不能同时用）
    sharey=True,            # 所有子图共享 Y 轴刻度（便于对比绝对值）
    sharex=True,
    margin_titles=True      # 在边缘显示行/列标题
)

# map 方法：对每个子图调用同一个绘图函数
g.map(sns.histplot, '配送耗时_小时', bins=20, kde=True)
# 或用 map_dataframe（可以传入 data 参数的函数）
g.map_dataframe(sns.lineplot, x='月份', y='销售额')

g.add_legend()
g.set_titles("{col_name} | {row_name}")
g.set_axis_labels("配送耗时（小时）", "频次")
g.fig.suptitle('各地区各季度配送耗时分布', y=1.02, fontweight='bold')
plt.show()
```

---

## 9. 样式精调全指南

> 这一节是最实用的部分——大多数教程只讲怎么画图，但真正让图表看起来专业的是这些细节。

### 9.1 标题与标签

```python
fig, ax = plt.subplots(figsize=(10, 5))
# ... 画图 ...

# 标题（pad 控制与图的距离）
ax.set_title('主标题', fontsize=15, fontweight='bold', pad=15, loc='left')
# loc='left'/'center'/'right'

# 副标题（放在主标题下方）
ax.text(0, 1.04, '副标题：补充说明文字', transform=ax.transAxes,
        fontsize=11, color='gray', va='bottom')

# 坐标轴标签
ax.set_xlabel('X轴说明', fontsize=12, labelpad=8)
ax.set_ylabel('Y轴说明', fontsize=12, labelpad=8)

# 数据来源注脚（右下角）
fig.text(0.99, 0.01, '数据来源：Olist 电商数据集',
         ha='right', va='bottom', fontsize=8, color='gray')
```

### 9.2 坐标轴精调

```python
import matplotlib.ticker as mticker

# ── 刻度范围 ──────────────────────────────
ax.set_xlim(0, 100)
ax.set_ylim(0, None)       # None = 自动

# ── 刻度位置与标签 ──────────────────────
ax.set_xticks([0, 25, 50, 75, 100])
ax.set_xticklabels(['0%', '25%', '50%', '75%', '100%'], fontsize=9)
ax.tick_params(axis='x', labelrotation=45, length=4, width=1)
ax.tick_params(axis='y', labelsize=9)

# ── 数字格式化 ───────────────────────────
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'¥{x:,.0f}'))
ax.xaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))  # 0.8 → 80%
ax.yaxis.set_major_formatter(mticker.FormatStrFormatter('%.2f'))

# ── 对数坐标轴 ───────────────────────────
ax.set_yscale('log')       # 数据范围跨越多个数量级时用

# ── 隐藏上/右边框线（更简洁）────────────
sns.despine(ax=ax)                         # 去掉上和右边框
sns.despine(ax=ax, left=True)              # 只保留底部
ax.spines['top'].set_visible(False)        # 单独控制某条边框
ax.spines['right'].set_visible(False)
```

### 9.3 参考线与标注

```python
# ── 水平/垂直参考线 ──────────────────────
ax.axhline(y=60, color='red', linestyle='--', linewidth=1.5,
           alpha=0.7, label='目标线 60%', zorder=2)
ax.axvline(x=df['x'].mean(), color='gray', linestyle=':',
           linewidth=1, alpha=0.6)

# 范围填充（如高亮某个区间）
ax.axhspan(80, 100, alpha=0.1, color='green', label='优秀区间')
ax.axvspan('2024-01', '2024-03', alpha=0.1, color='yellow')  # 时间轴用

# ── 文字标注（text + arrow）────────────
# 简单文字
ax.text(x=50, y=75,
        s='关键节点',
        fontsize=10, color='red',
        ha='center', va='bottom',        # 对齐方式
        bbox=dict(boxstyle='round,pad=0.3',   # 文字背景框
                  facecolor='white',
                  edgecolor='red',
                  alpha=0.9))

# 带箭头的标注（annotate）
ax.annotate(
    '双十一峰值',                        # 标注文字
    xy=(peak_x, peak_y),                 # 箭头指向的坐标（数据坐标）
    xytext=(peak_x + 5, peak_y + 500),   # 文字位置
    fontsize=10,
    arrowprops=dict(
        arrowstyle='->',
        color='red',
        lw=1.5,
        connectionstyle='arc3,rad=0.2'   # 箭头弧度（0=直线）
    ),
    bbox=dict(boxstyle='round', facecolor='lightyellow', alpha=0.9)
)
```

### 9.4 图例精调

```python
# 标准图例
ax.legend(
    title='图例标题',
    title_fontsize=10,
    fontsize=9,
    loc='upper right',           # 'best'/'upper left'/'lower right'...
    ncols=2,                     # 图例分几列排列
    framealpha=0.85,             # 背景透明度
    edgecolor='lightgray',
    borderpad=0.8,               # 图例边框内边距
    handlelength=1.5,            # 图例小图标的长度
    handleheight=1,
    labelspacing=0.4             # 图例各行之间的间距
)

# 把图例放到图外（不遮挡数据区域）
ax.legend(loc='upper left', bbox_to_anchor=(1, 1),
          title='分类', fontsize=9)
plt.tight_layout()              # 自动调整图幅，确保图例不被截断

# 隐藏图例
ax.get_legend().remove()
# 或
ax.legend([], [], frameon=False)
```

### 9.5 颜色与透明度

```python
# 单色覆盖（不用 palette）
sns.histplot(data=df, x='col', color='#4E79A7', alpha=0.7, ax=ax)

# 渐变色手动设置
from matplotlib.colors import LinearSegmentedColormap
custom_cmap = LinearSegmentedColormap.from_list('custom', ['#ffffff', '#4E79A7'])

# 高亮某一组（其余灰化）
palette = {cat: '#4E79A7' if cat == '目标品类' else '#cccccc'
           for cat in df['品类'].unique()}
sns.barplot(data=df, x='品类', y='销售额', palette=palette, ax=ax)
```

### 9.6 保存图片

```python
plt.savefig(
    'output.png',
    dpi=150,                    # 分辨率（屏幕显示用72，打印用300，报告用150）
    bbox_inches='tight',        # 自动裁剪多余白边（必加）
    facecolor='white',          # 背景色（默认透明，保存时变白）
    transparent=False           # True = 透明背景（适合叠加在其他图上）
)
# 也可以保存 svg（矢量图，无限缩放不失真）
plt.savefig('output.svg', format='svg', bbox_inches='tight')
```

---

## 10. 多子图排版

### 10.1 subplots 等分网格

```python
fig, axes = plt.subplots(
    nrows=2, ncols=3,
    figsize=(15, 9),
    dpi=100,
    sharex=False, sharey=False,  # 是否共享坐标轴
    constrained_layout=True      # 自动调整间距（比 tight_layout 更智能）
)

# axes 是二维数组，用 [行][列] 索引
sns.histplot(data=df, x='col1', ax=axes[0][0])
sns.boxplot(data=df, x='cat', y='col2', ax=axes[0][1])
sns.lineplot(data=df, x='date', y='val', ax=axes[0][2])
sns.scatterplot(data=df, x='x', y='y', ax=axes[1][0])
sns.barplot(data=df, x='cat', y='val', ax=axes[1][1])
axes[1][2].set_visible(False)    # 隐藏多余的空子图

fig.suptitle('多维度分析看板', fontsize=16, fontweight='bold', y=1.01)
```

### 10.2 subplot_mosaic 自由排版（推荐）

```python
# 用字符串矩阵描述布局，相同字母自动合并为一个大图
layout = [
    ['趋势', '趋势', '趋势'],    # 第一行：趋势图横跨三列
    ['分布', '对比', '回归']     # 第二行：三张小图各占一列
]

fig, axs = plt.subplot_mosaic(
    layout,
    figsize=(15, 9),
    dpi=100,
    constrained_layout=True
)

fig.suptitle('Olist 电商核心业务多维诊断看板',
             fontsize=16, fontweight='bold')

# 通过字符串 key 访问对应子图
sns.lineplot(data=df, x='月份', y='销售额', hue='品类',
             marker='o', ax=axs['趋势'])
axs['趋势'].set_title('核心品类月度营收趋势', loc='left',
                       fontsize=12, fontweight='bold')

sns.violinplot(data=df, x='地区', y='延迟耗时_小时',
               palette='pastel', ax=axs['分布'])
axs['分布'].set_title('各区域物流延迟特征分布', loc='left',
                        fontsize=12, fontweight='bold')
axs['分布'].tick_params(axis='x', labelrotation=15)

sns.barplot(data=df, x='品类', y='利润率', palette='muted',
            ax=axs['对比'])
axs['对比'].set_title('各品类利润率对比', loc='left',
                       fontsize=12, fontweight='bold')
axs['对比'].tick_params(axis='x', labelrotation=15)
for container in axs['对比'].containers:
    axs['对比'].bar_label(container, fmt='%.1f%%', fontsize=8, padding=2)

sns.regplot(data=df, x='折扣力度', y='复购率',
            scatter_kws={'alpha': 0.4, 'color': '#4A7BB0', 's': 40},
            line_kws={'color': '#D16666', 'linewidth': 2},
            ax=axs['回归'])
axs['回归'].set_title('折扣与复购率回归分析', loc='left',
                       fontsize=12, fontweight='bold')

plt.savefig('dashboard.png', dpi=150, bbox_inches='tight', facecolor='white')
plt.show()
```

---

## 11. 实战完整案例

> 一份完整的 EDA 可视化分析流程，从加载数据到输出多图看板。

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns
import pandas as pd
import numpy as np
from scipy import stats

# ══════════════════════════════════════════════
# STAGE 1: 全局配置
# ══════════════════════════════════════════════
sns.set_theme(style='whitegrid', context='notebook', palette='muted', font_scale=1.05)
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# ══════════════════════════════════════════════
# STAGE 2: 生成模拟数据
# ══════════════════════════════════════════════
np.random.seed(42)
N = 800
df = pd.DataFrame({
    '地区':     np.random.choice(['华东', '华南', '华北', '西南'], N),
    '品类':     np.random.choice(['电子', '服饰', '食品', '家居'], N),
    '季度':     np.random.choice(['Q1', 'Q2', 'Q3', 'Q4'], N),
    '折扣力度': np.random.uniform(0.5, 1.0, N),
    '广告投入': np.random.uniform(1, 50, N),
    '销售额':   np.random.exponential(scale=3000, size=N) + 500,
    '利润率':   np.random.normal(0.18, 0.08, N).clip(0, 0.5),
    '复购率':   np.random.uniform(0.1, 0.9, N),
    '配送耗时': np.random.gamma(shape=3, scale=8, size=N),
    '用户评分': np.random.choice([1,2,3,4,5], N, p=[0.05,0.08,0.15,0.32,0.4])
})
# 制造相关性：折扣力度越大，复购率越高
df['复购率'] = (df['折扣力度'] * 0.6 + df['复购率'] * 0.4).clip(0, 1)

# ══════════════════════════════════════════════
# STAGE 3: 多图看板（subplot_mosaic 自由排版）
# ══════════════════════════════════════════════
layout = [
    ['趋势', '趋势', '相关矩阵'],
    ['分布', '箱线', '回归']
]

fig, axs = plt.subplot_mosaic(layout, figsize=(16, 10), dpi=100,
                               constrained_layout=True)
fig.suptitle('Olist 电商多维业务诊断看板',
             fontsize=16, fontweight='bold', y=1.01)
fig.text(0.99, -0.01, '数据来源：模拟数据 · 仅供演示',
         ha='right', fontsize=8, color='gray')

# ── 图1：各地区品类平均销售额折线趋势 ──────────
monthly = (df.groupby(['季度', '地区'])['销售额'].mean().reset_index())
sns.lineplot(data=monthly, x='季度', y='销售额', hue='地区',
             marker='o', markersize=7, linewidth=2.2,
             palette='deep', ax=axs['趋势'])
axs['趋势'].set_title('各地区季度销售额均值趋势', loc='left',
                        fontsize=12, fontweight='bold')
axs['趋势'].set_xlabel('')
axs['趋势'].yaxis.set_major_formatter(
    mticker.FuncFormatter(lambda x, _: f'¥{x:,.0f}'))
axs['趋势'].legend(title='地区', fontsize=9, title_fontsize=9,
                    loc='upper left', framealpha=0.8)

# ── 图2：数值特征相关矩阵热力图 ──────────────
num_cols = ['折扣力度', '广告投入', '销售额', '利润率', '复购率', '配送耗时']
corr = df[num_cols].corr()
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt='.2f',
            cmap='RdBu_r', vmin=-1, vmax=1, center=0,
            square=True, linewidths=0.5, linecolor='white',
            annot_kws={'size': 8},
            cbar_kws={'shrink': 0.75, 'label': '相关系数'},
            ax=axs['相关矩阵'])
axs['相关矩阵'].set_title('特征相关矩阵', loc='left',
                            fontsize=12, fontweight='bold')
axs['相关矩阵'].set_xticklabels(
    axs['相关矩阵'].get_xticklabels(), rotation=30, ha='right', fontsize=9)
axs['相关矩阵'].set_yticklabels(
    axs['相关矩阵'].get_yticklabels(), rotation=0, fontsize=9)

# ── 图3：配送耗时 KDE 分布对比 ────────────────
sns.kdeplot(data=df, x='配送耗时', hue='地区',
            fill=True, alpha=0.2, linewidth=2,
            bw_adjust=0.9, cut=0, palette='deep', ax=axs['分布'])
axs['分布'].set_title('各地区配送耗时密度分布', loc='left',
                        fontsize=12, fontweight='bold')
axs['分布'].set_xlabel('配送耗时（小时）')
axs['分布'].axvline(df['配送耗时'].mean(), color='red',
                     linestyle='--', linewidth=1.3, alpha=0.7)
axs['分布'].text(df['配送耗时'].mean() + 0.5,
                  axs['分布'].get_ylim()[1] * 0.88,
                  f"均值\n{df['配送耗时'].mean():.1f}h",
                  color='red', fontsize=9)

# ── 图4：各品类利润率箱线图 + 数据点叠加 ────────
order = (df.groupby('品类')['利润率'].median()
          .sort_values(ascending=False).index)
sns.boxplot(data=df, x='品类', y='利润率',
            order=order, palette='pastel',
            width=0.5, linewidth=1.2,
            medianprops={'color': 'red', 'linewidth': 2},
            flierprops={'marker': 'D', 'markerfacecolor': 'red',
                        'markersize': 4, 'alpha': 0.5},
            ax=axs['箱线'])
sns.stripplot(data=df, x='品类', y='利润率',
              order=order, color='black',
              alpha=0.2, size=3, jitter=0.2, ax=axs['箱线'])
axs['箱线'].set_title('各品类利润率分布（含异常值）', loc='left',
                        fontsize=12, fontweight='bold')
axs['箱线'].yaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))

# ── 图5：折扣力度 vs 复购率回归分析 ────────────
sns.regplot(data=df, x='折扣力度', y='复购率',
            ci=95,
            scatter_kws={'alpha': 0.3, 's': 30,
                         'color': '#4A7BB0', 'edgecolor': 'none'},
            line_kws={'color': '#D16666', 'linewidth': 2.5},
            ax=axs['回归'])
r, p = stats.pearsonr(df['折扣力度'], df['复购率'])
axs['回归'].text(0.05, 0.90,
                  f'r = {r:.3f}\np {"< 0.001" if p < 0.001 else f"= {p:.3f}"}',
                  transform=axs['回归'].transAxes, fontsize=10,
                  bbox=dict(boxstyle='round,pad=0.4',
                            facecolor='lightyellow',
                            edgecolor='gray', alpha=0.9))
axs['回归'].set_title('折扣力度与复购率回归分析', loc='left',
                        fontsize=12, fontweight='bold')
axs['回归'].xaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))
axs['回归'].yaxis.set_major_formatter(mticker.PercentFormatter(xmax=1))
sns.despine(ax=axs['回归'])

plt.savefig('seaborn_dashboard.png', dpi=150,
            bbox_inches='tight', facecolor='white')
plt.show()
print("看板已保存为 seaborn_dashboard.png")
```





Claude is AI and can make mi
