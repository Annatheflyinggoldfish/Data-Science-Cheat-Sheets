# 🚀 Plotly Express (px) 速查表

> 💡 Plotly Express 参数命名规律：data_frame=df, x='列名', y='列名', color='分类列名'（相当于 Seaborn 的 hue）

* Plotly 生产环境工程红线与避坑
> 性能死穴：散点图超过 20 万行时，浏览器直接假死卡死
> 原因：Plotly 的底层是用 SVG 渲染每一个数据点的。如果你的 DataFrame 达到几十万行，它就要在网页里强行塞入几十万个 DOM 节点，会让你的 Jupyter 乃至电脑浏览器瞬间内存溢出。
- 解法：当数据量极大时，使用专门优化过的 px.scatter_gl() 代替 px.scatter()。它底层调用的是 WebGL 硬件加速（利用显卡去画点），渲染 100 万个点只需 0.5 秒，纵滑如丝。

* 数据联动陷阱：图表没有跟着 Pandas 链式命名自动改标题。
> 原因：Plotly 默认会直接抓取你的 DataFrame 的列名作为坐标轴标签。如果你清洗出来的列名叫 avg_latency_hours，图表上就会直接大剌剌地打印出这个难看的英文变量名。
- 解法：在任何 px. 函数中，塞入 labels=字典 参数进行一秒翻译，不需要为了画图专门去改原始 DataFrame 的列名：
```python
fig = px.line(
    df, x="month", y="sales",
    labels={"month": "统计月份", "sales": "核心营收额 (元)"} # 👈 动态对齐标签
)
```

---

## 1. 核心开局

```python
import plotly.express as px
import plotly.io as pio
import pandas as pd
import numpy as np

# 🌟 设置为简洁的白色网格皮肤
pio.templates.default = "plotly_white"

# 1. 读入数据
# df = pd.read_csv('your_actual_business_data.csv')
```
## 2. 四大核心流派
### 动态散点图 (px.scatter) —— 带趋势拟合与气泡大小控制
```python
# 业务场景：分析“广告预算”与“销售额”的因果相关性，并自动绘制 OLS 线性回归线
fig = px.scatter(
    df, 
    x="广告预算_元", 
    y="销售额_元", 
    color="品类分部",              # 自动按品类上色
    size="订单总量",              # 🌟 气泡大小代表订单量级别
    hover_name="城市",            # 鼠标悬浮在点上时，顶端加粗显示的标题字段
    trendline="ols",             # 🌟 瞬间注入：线性回归拟合线
    trendline_color_override="#D16666" # 拟合线强制使用莫兰迪高级红
)
# 展现成品（在 Jupyter 里直接交互，或在脚本中导出）
fig.show()
```

### 高阶时序折线图 (px.line) —— 动态悬浮提示
```python
# 业务场景：跨年度、多品类的月度营收走势
fig = px.line(
    df, x="统计月份", y="月营收_万元", color="产品大类",
    markers=True,                # 在折线转折处自动打上数据点样式
    title="2026年度核心品类营收月度走势图"
)
fig.show()
```

## 2.分布流派 (Distribution)：摸清指标扎堆区间
### 统计直方图 (px.histogram) —— 带边缘箱线图的多维透视
```python
# 业务场景：看“交付延迟时间（Latency）”的分布，并在顶部顺手挂一个箱线图抓异常值
fig = px.histogram(
    df, x="交付延迟_小时", color="物流渠道",
    nbins=40,                    # 划分 40 个连续区间
    marginal="box",              # 🌟 降维打击：在直方图顶部无缝叠加箱线图！
    barmode="overlay",           # overlay: 透明重叠；group: 并列
    opacity=0.7
)
fig.show()
```

## 3.分类流派 (Categorical)：分组指标横向对抗
### 条形图 (px.bar) —— 自动顶端贴标签与堆叠控制
```python
# 业务场景：各区域销售额对比。横向条形图（x和y对调）是长文本分类名称的黄金解法。
fig = px.bar(
    df, 
    x="销售额_万元", 
    y="产品品类", 
    color="季度",
    orientation='h',             # 'h' 代表横向条形图，'v' 代表纵向柱状图
    text_auto='.1f',             # 🌟 核心：全自动在柱子末端贴上标签（保留1位小数）
    color_discrete_sequence=['#4E79A7', '#F28E2B', '#E15759'] # 注入大厂配色面板
)
fig.show()
```

### 交互式小提琴图 (px.violin) —— 均值与密度合体
```python
fig = px.violin(
    df, x="所属部门", y="差旅报销_元", color="职级",
    box=True,                    # 在小提琴内部无缝画入标准四分位箱线
    points="all"                 # 🌟 把所有真实的数据点抖动散落在左侧，方便看个体
)
fig.show()
```

## 4.占比流派 (Part-to-Whole)：份额结构拆解
###
```python
fig = px.pie(
    df, values="市场份额", names="品牌名称",
    hole=0.3,                    # 🌟 设为 0.3 秒变更加现代化的“环形图（Donut Chart）”
    color_discrete_sequence=px.colors.qualitative.Muted # 调用 Plotly 内置的低饱和莫兰迪色系
)
# 规整修饰：强制白边隔开，文本显示在外部
fig.update_traces(textposition='inside', textinfo='percent+label', marker=dict(line=dict(color='#FFFFFF', width=2)))
fig.show()
```

## 5.散点气泡地图 (px.scatter_mapbox) —— 查看千万级订单经纬度热力
```python
# df = pd.read_csv('olist_orders_geo.csv') 
# 必须包含列：'lat' (纬度), 'lng' (经度), 'sales' (销售额), 'city' (城市名)

fig = px.scatter_mapbox(
    df, 
    lat="lat", 
    lon="lng", 
    size="sales",                # 圆圈的大小代表销售总额
    color="sales",               # 圆圈的颜色深浅也代表销售额（双重视觉强化）
    color_continuous_scale=px.colors.sequential.Peach, # 高级蜜桃暖色调渐变
    zoom=3,                      # 初始缩放比例（巴西全境选 3）
    hover_name="city",
    mapbox_style="carto-positron" # 🌟 极客红线：使用免 Token 的现代化淡浅灰底图
)
fig.update_layout(title="Olist 巴西全境订单地理气泡分布看板", margin={"r":0,"t":40,"l":0,"b":0})
fig.show()
```

## 一页多图（Subplots）
### 
```python
from plotly.subplots import make_subplots
import plotly.graph_objects as go

# 1. 规划 1 行 2 列的子图画布空间，并指定左边是常规坐标，右边是域坐标（饼图专用）
fig = make_subplots(rows=1, cols=2, specs=[[{"type": "xy"}, {"type": "domain"}]])

# 2. 用 px 快速生成具体的图形组件
sub_fig1 = px.line(df, x="月份", y="销售额")
sub_fig2 = px.pie(df, values="份额", names="分类")

# 3. 🌟 极客提取：把两条流水线上的图形轨迹，定向发射到合并画布的指定格子中
for trace in sub_fig1.data:
    fig.add_trace(trace, row=1, col=1)

for trace in sub_fig2.data:
    fig.add_trace(trace, row=1, col=2)

# 4. 统一修饰大盘大标题
fig.update_layout(title_text="Olist 核心指标多维联动静态大盘", height=500, width=1000)
fig.show()
```

