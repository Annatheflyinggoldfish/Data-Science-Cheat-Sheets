# Plotly Express 完整速查手册

> 核心命名规律：`data_frame=df, x='列名', y='列名', color='分类列'`（等同于 Seaborn 的 `hue`）
> 
> 全局建议：所有图表开头加 `pio.templates.default = "plotly_white"` 设置白色底图

```python
import plotly.express as px
import plotly.graph_objects as go
import plotly.io as pio
from plotly.subplots import make_subplots
import pandas as pd
import numpy as np

pio.templates.default = "plotly_white"
```

---

## 目录
1. [折线图 px.line](#1-折线图-pxline)
2. [柱状图 px.bar](#2-柱状图-pxbar)
3. [散点图 px.scatter](#3-散点图-pxscatter)
4. [饼图/环形图 px.pie](#4-饼图环形图-pxpie)
5. [热力图 px.imshow / px.density_heatmap](#5-热力图)
6. [箱线图 px.box](#6-箱线图-pxbox)
7. [小提琴图 px.violin](#7-小提琴图-pxviolin)
8. [直方图 px.histogram](#8-直方图-pxhistogram)
9. [面积图 px.area](#9-面积图-pxarea)
10. [漏斗图 px.funnel](#10-漏斗图-pxfunnel)
11. [地图 px.scatter_mapbox](#11-地图-pxscatter_mapbox)
12. [多图布局 make_subplots](#12-多图布局-make_subplots)
13. [全局样式控制 update_layout](#13-全局样式控制-update_layout)
14. [坐标轴精细控制](#14-坐标轴精细控制)
15. [颜色与配色方案](#15-颜色与配色方案)
16. [Hover Tooltip 定制](#16-hover-tooltip-定制)
17. [图例控制](#17-图例控制)
18. [标注与参考线](#18-标注与参考线)
19. [导出与保存](#19-导出与保存)
20. [性能优化](#20-性能优化)

---

## 1. 折线图 px.line

```python
# 基础折线图
fig = px.line(
    df, 
    x="Year",                          # x轴列名
    y="Value",                          # y轴列名（单列）
    color="Category",                   # 按分类上色，自动生成多条线
    markers=True,                       # 在数据点处显示标记
    line_dash="Category",               # 按分类用不同线型（实线/虚线/点线）
    title="标题",
    labels={"Value": "数值", "Year": "年份", "Category": "分类"}  # 重命名坐标轴和图例
)

# 多列同时画线（wide format）
fig = px.line(
    df, x="Year",
    y=["列A", "列B", "列C"],            # 直接传列名list
    markers=True,
    labels={"value": "数值", "variable": "指标"}  # wide模式下y轴叫value，图例叫variable
)

# 修改图例名称（wide模式列名太丑时用）
fig.for_each_trace(lambda t: t.update(name={
    "列A": "可读名称A",
    "列B": "可读名称B",
}.get(t.name, t.name)))

# 或者提前rename列名（更简洁）
df_plot = df.rename(columns={"列A": "可读名称A", "列B": "可读名称B"})
fig = px.line(df_plot, x="Year", y=["可读名称A", "可读名称B"], markers=True)

# 控制线条样式
fig.update_traces(line=dict(width=2.5))                    # 线宽
fig.update_traces(line=dict(dash="dash"))                  # 线型: solid/dash/dot/dashdot
fig.update_traces(marker=dict(size=8, symbol="circle"))    # 标记大小和形状

# 只改某一条线（用selector筛选）
fig.update_traces(line=dict(color="red"), selector=dict(name="某条线名称"))
```

---

## 2. 柱状图 px.bar

```python
# 基础竖向柱状图
fig = px.bar(
    df,
    x="Category",
    y="Value",
    color="SubGroup",                   # 按子分类上色
    barmode="group",                    # group=并列, overlay=重叠, relative=堆叠
    text_auto=True,                     # 自动在柱子上显示数值
    text_auto=".1f",                    # 保留1位小数
    text_auto=".0%",                    # 显示百分比格式
)

# 横向柱状图（分类名称长时推荐）
fig = px.bar(
    df,
    x="Value",
    y="Category",
    orientation="h",                    # h=横向, v=纵向（默认）
    text_auto=True,
)

# 堆叠柱状图
fig = px.bar(df, x="Year", y="Value", color="Category", barmode="stack")

# 调整柱子上文字位置
fig.update_traces(textposition="outside")    # outside/inside/auto
fig.update_traces(textfont_size=12)

# 调整柱子宽度和间距
fig.update_traces(width=0.6)                 # 0到1，越大越宽

# 排序（按数值从大到小）
df_sorted = df.sort_values("Value", ascending=False)
fig = px.bar(df_sorted, x="Category", y="Value",
             category_orders={"Category": df_sorted["Category"].tolist()})
```

---

## 3. 散点图 px.scatter

```python
# 基础散点图
fig = px.scatter(
    df,
    x="X列",
    y="Y列",
    color="分类列",                      # 按分类上色
    size="数值列",                       # 气泡大小
    size_max=40,                        # 最大气泡直径（像素）
    hover_name="标签列",                 # 悬浮时顶部加粗显示的字段
    hover_data=["额外列1", "额外列2"],   # 悬浮时额外显示的字段
    trendline="ols",                    # 添加线性回归线
    trendline="lowess",                 # 添加局部加权回归线（更平滑）
    trendline_color_override="red",     # 回归线颜色
    opacity=0.7,                        # 透明度（0-1）
    symbol="分类列",                    # 按分类用不同标记形状
)

# 标记形状选项
# circle, square, diamond, cross, x, triangle-up, triangle-down,
# star, hexagon, pentagon

# 添加文字标签
fig = px.scatter(df, x="X", y="Y", text="标签列")
fig.update_traces(textposition="top center")   # 标签位置

# 超大数据量用WebGL加速（>10万行时必用）
fig = px.scatter_gl(df, x="X", y="Y")
```

---

## 4. 饼图/环形图 px.pie

```python
# 基础饼图
fig = px.pie(
    df,
    names="分类列",
    values="数值列",
    title="标题",
    hole=0,                             # 0=饼图，0.3~0.5=环形图
    color_discrete_sequence=px.colors.qualitative.Set2,
)

# 扇区显示内容控制
fig.update_traces(
    textposition="inside",              # inside/outside/auto
    textinfo="label+percent",           # label/percent/value/label+percent/label+value+percent
    pull=[0.1, 0, 0, 0],               # 某个扇区向外拉出（突出显示，按顺序）
    marker=dict(line=dict(color="white", width=2)),  # 白色边框分隔扇区
    rotation=90,                        # 起始角度（顺时针）
)

# 环形图中间加文字
fig.add_annotation(
    text="总计<br>1,234",
    x=0.5, y=0.5,
    font_size=16,
    showarrow=False
)
```

---

## 5. 热力图

```python
# 方式一：矩阵热力图（适合相关性矩阵）
corr_matrix = df.corr()
fig = px.imshow(
    corr_matrix,
    text_auto=".2f",                    # 显示数值，保留2位小数
    color_continuous_scale="RdBu_r",   # 红蓝色阶（负相关红，正相关蓝）
    zmin=-1, zmax=1,                    # 固定色阶范围
    aspect="auto",                      # auto/equal
    title="相关性热力图"
)

# 方式二：二维密度热力图（适合两个连续变量）
fig = px.density_heatmap(
    df,
    x="X列",
    y="Y列",
    nbinsx=30,                          # x方向分箱数
    nbinsy=30,
    color_continuous_scale="Viridis",
    marginal_x="histogram",             # 顶部加边缘直方图
    marginal_y="histogram",
)

# 常用色阶方案
# 连续：Viridis, Plasma, Blues, Reds, Greens, YlOrRd, RdBu_r
# 发散（有中点）：RdBu_r, RdYlGn, Spectral
```

---

## 6. 箱线图 px.box

```python
fig = px.box(
    df,
    x="分类列",
    y="数值列",
    color="分类列",
    notched=True,                       # 显示置信区间缺口
    points="outliers",                  # outliers=只显示异常值, all=显示所有点, False=不显示
    hover_data=["额外信息列"],
)

# 添加均值标记
fig.update_traces(boxmean=True)         # True=显示均值线, "sd"=显示均值±标准差
```

---

## 7. 小提琴图 px.violin

```python
fig = px.violin(
    df,
    x="分类列",
    y="数值列",
    color="分类列",
    box=True,                           # 在小提琴内嵌入箱线图
    points="all",                       # all=显示所有点, outliers=只显示异常值
    violinmode="overlay",               # overlay=重叠, group=并列
)

# 左右对称小提琴（比较两组分布）
fig = px.violin(df, y="数值列", x="分类列", color="对比列",
                violinmode="overlay")
fig.update_traces(side="positive", width=1.5, selector=dict(name="组A"))
fig.update_traces(side="negative", width=1.5, selector=dict(name="组B"))
```

---

## 8. 直方图 px.histogram

```python
fig = px.histogram(
    df,
    x="数值列",
    color="分类列",
    nbins=40,                           # 分箱数量
    barmode="overlay",                  # overlay=透明重叠, group=并列, stack=堆叠
    opacity=0.7,
    marginal="box",                     # 顶部边缘图: box/violin/rug
    histnorm="probability density",     # 归一化: None/percent/probability/density/probability density
    cumulative=True,                    # 累积分布
)
```

---

## 9. 面积图 px.area

```python
fig = px.area(
    df,
    x="Year",
    y="Value",
    color="Category",
    line_group="Category",              # 确保每个分类独立填充
    groupnorm="percent",                # 归一化为百分比堆叠（100%面积图）
)
```

---

## 10. 漏斗图 px.funnel

```python
# 适合展示转化率、治疗路径等层层递减的数据
fig = px.funnel(
    df,
    x="数值列",                          # 漏斗宽度
    y="阶段列",                          # 漏斗各层标签
    color="分类列",
    title="转化漏斗"
)
```

---

## 11. 地图 px.scatter_mapbox

```python
fig = px.scatter_mapbox(
    df,
    lat="纬度列",
    lon="经度列",
    size="数值列",                       # 气泡大小
    color="数值列",                      # 气泡颜色
    color_continuous_scale="Reds",
    zoom=6,                             # 缩放级别（英国全境约5-6）
    hover_name="地名列",
    mapbox_style="carto-positron",      # 免Token底图
    # 其他底图: open-street-map, carto-darkmatter, stamen-terrain
)
fig.update_layout(margin={"r": 0, "t": 40, "l": 0, "b": 0})
fig.show()
```

---

## 12. 多图布局 make_subplots

```python
# 方式一：用px生成后合并（最简洁）
fig1 = px.line(df, x="Year", y="列A")
fig2 = px.bar(df, x="Year", y="列B")

fig = make_subplots(rows=1, cols=2,
                    subplot_titles=["图A标题", "图B标题"],
                    shared_xaxes=False,      # 是否共享x轴
                    shared_yaxes=False,
                    horizontal_spacing=0.1,  # 水平间距（0-1）
                    vertical_spacing=0.1)    # 垂直间距（0-1）

for trace in fig1.data:
    fig.add_trace(trace, row=1, col=1)
for trace in fig2.data:
    fig.add_trace(trace, row=1, col=2)

fig.update_layout(height=500, width=1200, title_text="合并大标题")
fig.show()

# 方式二：含饼图时需要指定type
fig = make_subplots(
    rows=1, cols=2,
    specs=[[{"type": "xy"}, {"type": "domain"}]]   # xy=普通坐标系, domain=饼图专用
)

# 常用specs类型
# {"type": "xy"}       折线/柱/散点
# {"type": "domain"}   饼图/环形图
# {"type": "scene"}    3D图
# {"type": "mapbox"}   地图

# 竖排多图（共享x轴，适合趋势对比）
fig = make_subplots(rows=3, cols=1,
                    shared_xaxes=True,
                    subplot_titles=["转介量", "恢复率", "等待时间"],
                    vertical_spacing=0.05)
```

---

## 13. 全局样式控制 update_layout

```python
fig.update_layout(
    # ── 标题 ──────────────────────────────
    title="图表标题",
    title_x=0.5,                        # 标题水平位置（0=左, 0.5=居中, 1=右）
    title_font_size=20,
    title_font_family="Arial",
    title_font_color="#333333",

    # ── 画布尺寸 ───────────────────────────
    width=1000,
    height=500,
    margin=dict(l=60, r=40, t=80, b=60),  # 左右上下边距（像素）

    # ── 背景颜色 ───────────────────────────
    plot_bgcolor="white",               # 绘图区背景
    paper_bgcolor="white",             # 整个画布背景

    # ── 全局字体 ───────────────────────────
    font=dict(family="Arial", size=13, color="#333333"),
    # 注意：全局font是默认值，局部设置会覆盖它

    # ── 图例 ──────────────────────────────
    showlegend=True,
    legend=dict(
        title="图例标题",
        title_font_size=13,
        font=dict(size=12),
        orientation="h",               # h=水平排列, v=垂直排列（默认）
        x=0.5, y=-0.15,               # 图例位置（图表外下方居中）
        xanchor="center",
        yanchor="top",
        bgcolor="rgba(255,255,255,0.8)",  # 图例背景（半透明白）
        bordercolor="#cccccc",
        borderwidth=1,
    ),

    # ── 网格线 ────────────────────────────
    xaxis_showgrid=True,
    yaxis_showgrid=True,
    xaxis_gridcolor="#eeeeee",
    yaxis_gridcolor="#eeeeee",
    xaxis_gridwidth=1,
)
```

---

## 14. 坐标轴精细控制

```python
fig.update_layout(
    xaxis=dict(
        title="X轴标题",
        title_font_size=14,
        title_font_family="Arial",
        tickfont=dict(size=12, family="Arial"),
        tickangle=-45,                  # 刻度标签旋转角度
        tickformat="%Y/%m",            # 日期格式
        tickformat=".0%",              # 百分比格式
        tickformat=",.0f",             # 千位分隔符
        tickprefix="£",                # 刻度前缀
        ticksuffix="%",                # 刻度后缀
        range=[0, 100],                # 固定轴范围
        nticks=6,                      # 大约显示几个刻度
        showline=True,                 # 显示轴线
        linecolor="#333333",
        linewidth=1.5,
        zeroline=False,                # 隐藏零刻度线
        showgrid=True,
        gridcolor="#eeeeee",
        gridwidth=1,
        type="category",               # 轴类型: linear/log/date/category
    ),
    yaxis=dict(
        title="Y轴标题",
        tickfont=dict(size=12),
        tickformat=".1f",
        rangemode="tozero",            # tozero=从0开始, nonnegative, normal
    )
)

# 双Y轴
fig = make_subplots(specs=[[{"secondary_y": True}]])
fig.add_trace(go.Scatter(x=df["Year"], y=df["列A"], name="列A"), secondary_y=False)
fig.add_trace(go.Scatter(x=df["Year"], y=df["列B"], name="列B"), secondary_y=True)
fig.update_yaxes(title_text="左轴标题", secondary_y=False)
fig.update_yaxes(title_text="右轴标题", secondary_y=True)
```

---

## 15. 颜色与配色方案

```python
# 传入配色：color_continuous_scale
fig = px.scatter(df, x="X", y="Y", color="数值列",
                 color_continuous_scale=px.colors.sequential.Viridis)
# color传的列是数字 → color_continuous_scale（渐变色）
# color传的列是字符串/类别 → color_discrete_sequence（离散色，每个类别一个颜色）

# ── 离散配色（分类数据）─────────────────────
px.colors.qualitative.Plotly          # Plotly默认10色
px.colors.qualitative.Set1            # 高饱和度9色
px.colors.qualitative.Set2            # 低饱和度柔和8色
px.colors.qualitative.Pastel          # 粉彩系
px.colors.qualitative.Safe            # 色盲友好
px.colors.qualitative.Vivid           # 鲜艳10色

# ── 连续配色（数值渐变）─────────────────────
px.colors.sequential.Blues            # 蓝色渐变
px.colors.sequential.Viridis          # 紫→绿→黄（科学可视化标准）
px.colors.sequential.Plasma           # 紫→橙→黄
px.colors.sequential.YlOrRd           # 黄→橙→红
px.colors.sequential.Greens

# ── 发散配色（有正负中心点）──────────────────
px.colors.diverging.RdBu_r            # 红蓝（适合相关性矩阵）
px.colors.diverging.RdYlGn            # 红黄绿
px.colors.diverging.Spectral

# ── 手动指定颜色 ──────────────────────────
fig = px.bar(df, color="Category",
             color_discrete_map={
                 "类别A": "#4E79A7",
                 "类别B": "#F28E2B",
                 "类别C": "#E15759",
             })
color_discrete_sequence=['#2ca02c'] # # 单色：

# ── 常用专业配色（手动）──────────────────────
# Tableau 10：#4E79A7 #F28E2B #E15759 #76B7B2 #59A14F #EDC948 #B07AA1 #FF9DA7 #9C755F #BAB0AC
# NHS蓝：#005EB8
# 莫兰迪系：#B5C4B1 #D4A5A5 #A8C5DA #F2D0A4 #C3B1D6
```
<img width="680" height="126" alt="image" src="https://github.com/user-attachments/assets/ca894d87-622b-4a59-a12d-4554019fe17c" />


---

## 16. Hover Tooltip 定制

```python
# 方式一：用hovertemplate完全自定义
fig.update_traces(
    hovertemplate=(
        "<b>%{x}</b><br>"              # 加粗显示x值
        "数值: %{y:,.0f}<br>"          # y值千位分隔
        "占比: %{customdata[0]:.1%}"   # 自定义数据
        "<extra></extra>"              # 隐藏右侧默认框
    ),
    customdata=df[["占比列"]].values
)

# 方式二：用hover_data参数（简单场景）
fig = px.scatter(df, x="X", y="Y",
                 hover_name="名称列",              # 悬浮框标题
                 hover_data={
                     "X": False,                  # 隐藏某列
                     "Y": ":.2f",                 # 显示并格式化
                     "额外列": True,               # 直接显示
                 })

# 常用格式化符号
# %{y:.2f}       保留2位小数
# %{y:,.0f}      千位分隔符，整数
# %{y:.1%}       百分比格式
# %{y:+.1f}      显示正负号
```

---

## 17. 图例控制

```python
# 隐藏图例
fig.update_layout(showlegend=False)

# 图例位置（图内右上角）
fig.update_layout(legend=dict(x=0.98, y=0.98, xanchor="right", yanchor="top"))

# 图例位置（图表正下方水平排列）
fig.update_layout(legend=dict(
    orientation="h",
    x=0.5, y=-0.15,
    xanchor="center", yanchor="top"
))

# 只隐藏某一条线的图例
fig.update_traces(showlegend=False, selector=dict(name="不想显示的线名"))

# 修改图例顺序（通过重排data）
fig.data = fig.data[::-1]              # 反转图例顺序
```

---

## 18. 标注与参考线

```python
# 添加水平参考线
fig.add_hline(
    y=50,
    line_dash="dash",                  # solid/dash/dot/dashdot
    line_color="red",
    line_width=1.5,
    annotation_text="目标线 50%",
    annotation_position="top right",   # top right/left, bottom right/left
    annotation_font_size=12,
    annotation_font_color="red",
)

# 添加垂直参考线
fig.add_vline(
    x="2021/22",
    line_dash="dot",
    line_color="gray",
    annotation_text="政策变化节点",
    annotation_position="top right",
)

# 添加矩形阴影区域（高亮某个时间段）
fig.add_vrect(
    x0="2020/21", x1="2021/22",
    fillcolor="yellow", opacity=0.15,
    layer="below",                     # below=在数据下方, above=在数据上方
    line_width=0,
    annotation_text="疫情期间",
    annotation_position="top left",
)

# 添加文字标注
fig.add_annotation(
    x="2022/23", y=75,
    text="恢复率高峰",
    showarrow=True,
    arrowhead=2,
    arrowcolor="#333333",
    font=dict(size=12, color="#333333"),
    bgcolor="white",
    bordercolor="#cccccc",
)
```

---

## 19. 导出与保存

```python
# 保存为交互式HTML（完整保留交互功能）
fig.write_html("output.html")
fig.write_html("output.html", include_plotlyjs="cdn")   # 用CDN加载，文件更小

# 保存为静态图片（需要安装kaleido）
# pip install kaleido
fig.write_image("output.png", width=1200, height=600, scale=2)   # scale=2即2倍清晰度
fig.write_image("output.svg")          # 矢量图，可无限放大
fig.write_image("output.pdf")

# 在Notebook中显示
fig.show()

# 导出为JSON（便于复用图表配置）
fig.write_json("output.json")
fig_restored = go.Figure(fig.to_dict())   # 从dict恢复

# GitHub展示方案
# 1. 保存HTML → push到GitHub → 用 https://htmlpreview.github.io 预览
# 2. 保存PNG/SVG作为静态截图放入README
# 3. 把nbviewer链接放在README（notebook里的plotly图可交互）
```

---

## 20. 性能优化

```python
# 散点图超过10万行时用WebGL加速
fig = px.scatter_gl(df, x="X", y="Y")           # 代替 px.scatter
fig = go.Figure(go.Scattergl(x=df["X"], y=df["Y"]))

# 大数据折线图降采样（用resample或每N行取一行）
df_sampled = df.iloc[::5]                        # 每5行取一行
df_sampled = df.set_index("Date").resample("W").mean().reset_index()  # 按周聚合

# 减少动画帧数（animated图表）
# 避免在超大df上用animation_frame参数

# 输出HTML时减小文件体积
fig.write_html("output.html", 
               include_plotlyjs="cdn",            # 不打包plotly.js（减小约3MB）
               full_html=False)                   # 只输出div片段（嵌入网页时用）
```

---

## 附：常用格式化参考

| 场景 | 格式字符串 | 示例输出 |
|------|-----------|---------|
| 整数 | `".0f"` | 1234 |
| 千位分隔 | `",.0f"` | 1,234 |
| 保留2位小数 | `".2f"` | 12.34 |
| 百分比 | `".1%"` | 12.3% |
| 科学计数 | `".2e"` | 1.23e+04 |
| 货币（英镑） | `"£,.0f"` 或 tickprefix="£" | £1,234 |
| 日期 | `"%Y-%m"` | 2024-03 |

## 附：常用模板

```python
# plotly_white    白色背景，灰色网格（最常用）
# plotly_dark     深色背景
# ggplot2         仿ggplot风格
# seaborn         仿seaborn风格
# simple_white    极简白色，无网格
# none            完全空白

pio.templates.default = "plotly_white"
```
