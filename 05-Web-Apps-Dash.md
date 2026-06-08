# Dash by Plotly 完整手册

> **核心思路**：Dash 本质上是把 Python 逻辑和网页界面连接起来的桥梁。
> - `dash.html` 负责**页面结构**（标题、容器、布局）
> - `dash.dcc` 负责**交互控件和图表**（下拉框、滑块、图表）
> - `@callback` 负责**数据联动**（用户操作 → Python 计算 → 页面更新）

---

## 目录

1. [安装与初始化](#1-安装与初始化)
2. [页面布局：html 组件](#2-页面布局html-组件)
3. [Bootstrap 栅格排版](#3-bootstrap-栅格排版)
4. [交互控件：dcc 组件](#4-交互控件dcc-组件)
5. [Callback 响应式联动](#5-callback-响应式联动)
6. [Plotly Express 图表速查](#6-plotly-express-图表速查)
7. [实战完整案例（含ML）](#7-实战完整案例含ml)
8. [常见报错与解决](#8-常见报错与解决)

---

## 1. 安装与初始化

```bash
pip install dash dash-bootstrap-components plotly pandas
```

```python
from dash import Dash, html, dcc, Input, Output, State, callback
import dash_bootstrap_components as dbc
import plotly.express as px
import pandas as pd

# 初始化应用
# __name__ 告诉 Dash 从哪里找静态资源文件
# external_stylesheets 加载 Bootstrap CSS，让页面有好看的默认样式
app = Dash(__name__, external_stylesheets=[dbc.themes.FLATLY])

# 常用 Bootstrap 皮肤主题（换名字就换风格）
# FLATLY   → 扁平化、清爽商务感（推荐）
# BOOTSTRAP → 经典 Bootstrap 风格
# CYBORG   → 暗黑主题
# MINTY    → 淡绿清新
# SLATE    → 深灰高级感

# 定义页面内容（必须赋值给 app.layout）
app.layout = html.Div("这里放页面组件")

# 启动本地开发服务器
if __name__ == '__main__':
    app.run(debug=True, port=8050)
    # debug=True：代码修改后网页自动刷新，不用手动重启
    # 访问地址：http://127.0.0.1:8050
```

---

## 2. 页面布局：html 组件

> `dash.html` 模块把 HTML 标签包装成 Python 对象。
> 每个组件都有两个核心参数：
> - `children`：子内容（可以是文字、列表、其他组件），通常作为第一个位置参数传入
> - `className`：直接对应 HTML 的 `class`，可以填任意 Bootstrap 样式名

```python
html.Div([                              # <div> 最常用的容器
    
    # ── 标题系列 ──────────────────────
    html.H1("一级大标题"),               # 最大标题，页面顶部 banner 常用
    html.H2("二级标题"),
    html.H3("卡片标题"),
    html.H4("小节标题"),
    
    # ── 文本系列 ──────────────────────
    html.P("正文段落文字"),              # paragraph，段落
    html.Span("行内文字"),              # 不换行的文字片段
    html.Strong("加粗文字"),
    html.Small("灰色小字说明"),
    html.Pre("代码/等宽字体"),           # 保留空格和换行
    
    # ── 布局辅助 ──────────────────────
    html.Hr(),                          # 水平分割线
    html.Br(),                          # 换行
    
    # ── 链接与图片 ──────────────────────
    html.A("点击跳转", href="https://..."),
    html.Img(src="图片URL", style={"width": "100%"}),
    
    # ── 列表 ──────────────────────────
    html.Ul([                           # 无序列表
        html.Li("第一项"),
        html.Li("第二项"),
    ]),
    
], 
    id="my-div",                        # id：给 callback 引用，全局唯一
    className="container my-4",         # className：Bootstrap 样式类
    style={"color": "red",              # style：行内样式，用字典传入
           "fontSize": "16px"}          # ⚠️ CSS属性名用驼峰写法（font-size → fontSize）
)
```

### Bootstrap 常用样式名速查

```
# 间距（m=margin, p=padding, 数字0-5）
mt-3       → margin-top: 1rem
mb-4       → margin-bottom: 1.5rem
p-3        → padding: 1rem
px-4       → 左右 padding
py-2       → 上下 padding

# 文字
text-center    → 居中
text-muted     → 灰色弱化文字
text-primary   → 蓝色强调
text-success   → 绿色
text-danger    → 红色
fw-bold        → 粗体
fs-5           → 字号（1最大，6最小）

# 背景
bg-primary     → 蓝色背景
bg-light       → 浅灰背景
bg-white       → 白色背景

# 边框与圆角
border         → 加边框
rounded        → 圆角
shadow-sm      → 轻微阴影（卡片感）
shadow         → 明显阴影

# 显示控制
d-flex         → flexbox 容器
d-block        → 块级显示
d-none         → 隐藏
```

---

## 3. Bootstrap 栅格排版

> Bootstrap 把页面横向分成 **12 列**，用 `Row + Col` 控制每个区块占几列。
> 12列分配完就自动换行，是响应式布局的基础。

```python
import dash_bootstrap_components as dbc

app.layout = dbc.Container([      # Container：有最大宽度限制的居中容器
                                  # fluid=True → 全宽撑满屏幕，适合仪表盘

    # ── 单行三等分（每块占4列，4+4+4=12）──
    dbc.Row([
        dbc.Col(html.Div("左侧"),   width=4),
        dbc.Col(html.Div("中间"),   width=4),
        dbc.Col(html.Div("右侧"),   width=4),
    ], className="mb-3"),          # mb-3：这一行底部加间距
    
    # ── 侧边栏 + 主内容区（常用布局）──
    dbc.Row([
        dbc.Col([                  # 左侧控制面板，占3列
            html.Div("筛选器放这里")
        ], width=3),
        dbc.Col([                  # 右侧图表区，占9列
            html.Div("图表放这里")
        ], width=9),
    ]),
    
    # ── 自适应宽度 ──
    dbc.Row([
        dbc.Col(html.Div("自动均分"), width="auto"),
        dbc.Col(html.Div("填满剩余空间")),  # 不写 width → 占满剩余列
    ]),

], fluid=True)

# ── 卡片组件（指标展示常用）──
dbc.Card([
    dbc.CardBody([
        html.H6("卡片标题", className="card-title text-muted"),
        html.H3("核心数字", className="text-primary fw-bold"),
        html.P("补充说明文字", className="text-muted mb-0"),
    ])
], className="shadow-sm border-0 rounded")  # 无边框 + 阴影 = 悬浮卡片感
```

---

## 4. 交互控件：dcc 组件

> `dash.dcc`（Dash Core Components）提供所有可以触发 callback 的控件。
> 每个控件都有一个 `id`，callback 通过这个 id 找到它，读取或修改它的属性。

### 4.1 下拉框 Dropdown

```python
dcc.Dropdown(
    id='my-dropdown',              # ⚠️ id 必填，callback 用这个找到它
    
    # options 写法一：简单列表（label和value相同）
    options=['全部', '上海', '北京', '广州'],
    
    # options 写法二：字典列表（label显示文字，value传给callback的实际值）
    options=[
        {'label': '📍 上海', 'value': 'SH'},
        {'label': '📍 北京', 'value': 'BJ'},
        {'label': '📍 广州', 'value': 'GZ'},
    ],
    
    # 从 DataFrame 动态生成 options（实战中最常用）
    # options=[{'label': v, 'value': v} for v in df['city'].unique()],
    
    value='SH',                    # 默认选中值，要和 options 中的 value 对应
    multi=False,                   # False=单选；True=多选，value变成列表
    clearable=False,               # False=禁止清空；True=允许清空（会传入 None）
                                   # ⚠️ 如果 clearable=True，callback 里要处理 None 的情况
    searchable=True,               # 是否允许搜索过滤选项（默认True）
    placeholder="请选择城市...",    # clearable=True 时的占位提示文字
    className="mb-3"
)
```

### 4.2 单选按钮 RadioItems

```python
dcc.RadioItems(
    id='chart-type-radio',
    options=[
        {'label': ' 折线图', 'value': 'line'},
        {'label': ' 柱状图', 'value': 'bar'},
        {'label': ' 散点图', 'value': 'scatter'},
    ],
    value='line',                  # 默认选中
    inline=True,                   # True=水平排列；False=竖向排列（默认）
    className="mb-2"
)
```

### 4.3 多选框 Checklist

```python
dcc.Checklist(
    id='feature-checklist',
    options=['显示均值线', '显示预测区间', '显示异常点'],
    value=['显示均值线'],           # 默认勾选项（列表）
    inline=True
)
```

### 4.4 滑块 Slider / RangeSlider

```python
# 单滑块
dcc.Slider(
    id='year-slider',
    min=2018, max=2024, step=1,
    value=2022,                    # 默认位置
    marks={                        # 刻度标签（不写则自动生成）
        2018: '2018',
        2020: {'label': '2020', 'style': {'color': 'red'}},  # 带样式的刻度
        2024: '2024'
    },
    tooltip={"placement": "bottom", "always_visible": True}  # 滑动时显示数值
)

# 范围滑块（选一个区间）
dcc.RangeSlider(
    id='price-range',
    min=0, max=1000, step=50,
    value=[200, 800],              # 默认区间，列表形式 [左端, 右端]
    marks={0: '¥0', 500: '¥500', 1000: '¥1000'}
)
```

### 4.5 日期选择器

```python
import datetime

# 单日期
dcc.DatePickerSingle(
    id='date-picker',
    date=datetime.date(2024, 1, 1),      # 默认日期
    min_date_allowed=datetime.date(2020, 1, 1),
    max_date_allowed=datetime.date.today(),
    display_format='YYYY-MM-DD'
)

# 日期范围
dcc.DatePickerRange(
    id='date-range',
    start_date=datetime.date(2024, 1, 1),
    end_date=datetime.date(2024, 12, 31),
    display_format='YYYY-MM-DD'
)
```

### 4.6 输入框与按钮

```python
# 文本输入框
dcc.Input(
    id='search-input',
    type='text',                   # 'text' / 'number' / 'email' / 'password'
    placeholder='输入搜索关键词...',
    debounce=True,                 # True=失焦或回车后才触发callback（防止每输入一个字都触发）
    className="form-control"       # Bootstrap 输入框样式
)

# 按钮（配合 n_clicks 使用）
html.Button(
    '🔄 刷新数据',
    id='refresh-btn',
    n_clicks=0,                    # 记录被点击了多少次，callback 通过判断次数是否变化来触发
    className="btn btn-primary"    # Bootstrap 按钮样式
)
# 也可用 dbc.Button（样式更多）
dbc.Button("确认", id="confirm-btn", color="success", className="mt-2")
```

### 4.7 图表容器 Graph

```python
# Graph 是承载 Plotly 图表的容器
# figure 参数接收 plotly 的 fig 对象
dcc.Graph(
    id='main-chart',
    figure=fig,                    # 初始图表（可以是空的 {}）
    config={
        'displayModeBar': True,    # 是否显示顶部工具栏（缩放/下载等）
        'toImageButtonOptions': {  # 下载图片的设置
            'format': 'png',
            'filename': 'my_chart',
            'scale': 2             # 下载高清图（2倍分辨率）
        }
    },
    style={'height': '400px'}      # 控制图表高度
)
```

### 4.8 数据存储 Store（跨callback共享数据）

```python
# Store 不显示在页面上，只在浏览器内存中存数据
# 用于多个 callback 共享同一份处理过的数据，避免重复计算
dcc.Store(
    id='processed-data-store',
    storage_type='memory'          # 'memory'=刷新清空（默认）
                                   # 'session'=关标签页清空
                                   # 'local'=永久保存在浏览器
)
```

---

## 5. Callback 响应式联动

> **Callback 的工作原理**：
> 1. 用户操作某个控件（Input 触发）
> 2. Dash 自动调用绑定的 Python 函数
> 3. 函数处理数据，return 新的内容
> 4. Dash 把 return 值塞进 Output 指定的组件属性里
> 5. 页面更新

### 5.1 基础单输入单输出

```python
from dash import Input, Output, callback

@callback(
    Output('output-div', 'children'),   # 谁接收结果（组件id，属性名）
    Input('my-dropdown', 'value')       # 谁触发（组件id，属性名）
)
def update_text(selected_value):
    # selected_value 就是用户在下拉框里选的值
    return f"你选择了：{selected_value}"

# 常用的 component_property（属性名）速查：
# Graph  → 'figure'       （图表内容）
# Div    → 'children'     （子内容，可以是文字、组件、列表）
# Input  → 'value'        （输入的文字或数字）
# Dropdown → 'value'      （当前选中值）
# Button → 'n_clicks'     （被点击的次数）
# Store  → 'data'         （存储的数据）
# 任意组件 → 'style'      （行内样式字典）
# 任意组件 → 'className'  （CSS类名）
```

### 5.2 多输入多输出

```python
@callback(
    # 多个 Output：用列表传入，return 时对应顺序返回多个值
    [Output('chart-1', 'figure'),
     Output('chart-2', 'figure'),
     Output('summary-text', 'children')],
    
    # 多个 Input：任意一个改变都会触发整个函数
    [Input('city-dropdown', 'value'),
     Input('year-slider',   'value'),
     Input('metric-radio',  'value')]
)
def update_dashboard(selected_city, selected_year, selected_metric):
    # 三个参数按 Input 顺序接收
    
    filtered = df[
        (df['city'] == selected_city) &
        (df['year'] == selected_year)
    ]
    
    fig1 = px.line(filtered, x='month', y=selected_metric)
    fig2 = px.bar(filtered, x='category', y=selected_metric)
    summary = f"共 {len(filtered)} 条数据，均值 {filtered[selected_metric].mean():.2f}"
    
    return fig1, fig2, summary  # 顺序必须和 Output 列表一致
```

### 5.3 State：不触发但参与计算

```python
from dash import Input, Output, State

# Input：值变化时立即触发 callback
# State：值不触发 callback，但 callback 执行时能读到它的当前值
# 典型场景：点击"确认"按钮时，读取输入框里的内容

@callback(
    Output('result', 'children'),
    Input('submit-btn', 'n_clicks'),    # 按钮点击才触发
    State('name-input', 'value'),       # 读取输入框的值，但不触发
    State('age-input',  'value'),
    prevent_initial_call=True           # 页面加载时不执行（防止空值报错）
)
def on_submit(n_clicks, name, age):
    if not name or not age:
        return "请填写完整信息"
    return f"提交成功：{name}，{age}岁"
```

### 5.4 防止初始调用与空值处理

```python
from dash import callback_context, no_update

@callback(
    Output('chart', 'figure'),
    Input('dropdown', 'value'),
    prevent_initial_call=True           # 页面加载时跳过，等用户操作再执行
)
def update(value):
    if value is None:                   # clearable=True 时可能传入 None
        return no_update                # no_update = 告诉 Dash 不要更新这个 Output
    
    # 正常逻辑
    fig = px.bar(df[df['city'] == value], x='month', y='sales')
    return fig
```

### 5.5 判断是哪个输入触发了 callback

```python
from dash import callback_context

@callback(
    Output('chart', 'figure'),
    [Input('btn-line', 'n_clicks'),
     Input('btn-bar',  'n_clicks')]
)
def switch_chart(n_line, n_bar):
    # callback_context.triggered_id 返回触发这次 callback 的组件 id
    trigger = callback_context.triggered_id
    
    if trigger == 'btn-bar':
        return px.bar(df, x='x', y='y')
    else:
        return px.line(df, x='x', y='y')
```

### 5.6 用 Store 在 callback 间共享数据

```python
# 布局中放一个隐形 Store
dcc.Store(id='filtered-data', storage_type='memory')

# Callback 1：过滤数据后存入 Store
@callback(
    Output('filtered-data', 'data'),    # 存入 Store（data属性）
    Input('city-dropdown', 'value')
)
def filter_data(city):
    filtered = df[df['city'] == city]
    return filtered.to_dict('records')  # Store 只能存 JSON 兼容的数据类型

# Callback 2：从 Store 读取数据画图
@callback(
    Output('chart', 'figure'),
    Input('filtered-data', 'data'),     # 从 Store 读取
    Input('metric-radio',  'value')
)
def draw_chart(stored_data, metric):
    filtered = pd.DataFrame(stored_data)  # 从字典列表还原成 DataFrame
    return px.line(filtered, x='date', y=metric)
```

---

## 6. Plotly Express 图表速查

> Plotly Express（`px`）是 Plotly 的高级接口，一行代码画出交互图表。
> 所有图表都返回 `fig` 对象，可以直接塞进 `dcc.Graph(figure=fig)`。

### 6.1 常用图表

```python
import plotly.express as px

# ── 折线图（趋势）──────────────────────────────
fig = px.line(df,
    x='date', y='sales',
    color='city',               # 按城市分色，自动生成多条线
    line_dash='category',       # 不同类别用不同虚线样式
    title='月度销售趋势',
    labels={'sales': '销售额（元）', 'date': '日期'}  # 重命名轴标签
)

# ── 柱状图（对比）──────────────────────────────
fig = px.bar(df,
    x='category', y='revenue',
    color='region',
    barmode='group',            # 'group'=并排；'stack'=堆叠；'overlay'=重叠
    text_auto=True,             # 柱子上方自动显示数值
    color_discrete_sequence=px.colors.qualitative.Set2  # 配色方案
)

# ── 散点图（相关性）────────────────────────────
fig = px.scatter(df,
    x='price', y='rating',
    size='sales_volume',        # 点的大小代表销量
    color='category',           # 点的颜色代表类别
    hover_name='product_name',  # 悬停时显示的标题
    hover_data=['city', 'date'],# 悬停时额外显示的字段
    trendline='ols'             # 添加线性回归趋势线（需要 statsmodels）
)

# ── 直方图（分布）──────────────────────────────
fig = px.histogram(df,
    x='delivery_hours',
    nbins=30,                   # 分桶数量
    color='city',
    barmode='overlay',          # 多组叠加显示
    opacity=0.6,                # 透明度（叠加时避免遮挡）
    marginal='box'              # 顶部附加箱线图，一图两用
)

# ── 箱线图（分布对比）──────────────────────────
fig = px.box(df,
    x='city', y='price',
    color='city',
    points='outliers'           # 'all'=显示所有点；'outliers'=只显示异常点；False=不显示
)

# ── 热力图（相关矩阵）──────────────────────────
corr_matrix = df.corr()
fig = px.imshow(corr_matrix,
    text_auto='.2f',            # 格子里显示数值，保留2位小数
    color_continuous_scale='RdBu_r',  # 红蓝配色（相关矩阵常用）
    zmin=-1, zmax=1,            # 固定色阶范围
    title='特征相关矩阵'
)

# ── 饼图 / 环形图（占比）──────────────────────
fig = px.pie(df,
    names='category',
    values='revenue',
    hole=0.4,                   # 0=实心饼图；>0=环形图（推荐0.3~0.5）
    title='各类别收入占比'
)
```

### 6.2 图表美化与布局

```python
# update_layout：修改整体布局
fig.update_layout(
    title=dict(text='图表标题', x=0.5, font=dict(size=18)),  # 标题居中
    template='plotly_white',    # 白色背景主题（推荐）
                                # 其他：'plotly'/'ggplot2'/'seaborn'/'simple_white'
    xaxis_title='X轴说明',
    yaxis_title='Y轴说明',
    legend=dict(
        orientation='h',        # 'h'=水平图例；'v'=竖向（默认）
        y=-0.2,                 # 图例位置（负值放到图表下方）
        x=0.5,
        xanchor='center'
    ),
    height=450,                 # 图表高度（px）
    margin=dict(l=40, r=40, t=60, b=40),  # 四周留白
    plot_bgcolor='white',       # 绘图区背景色
    paper_bgcolor='white',      # 整张图背景色
    hovermode='x unified'       # 悬停时显示同一X轴所有系列的值
)

# update_traces：修改数据系列的样式
fig.update_traces(
    line=dict(width=2.5, dash='solid'),  # 线宽和线型
    marker=dict(size=8, opacity=0.8),    # 散点大小和透明度
    textfont=dict(size=11),              # 数值标签字号
    hovertemplate='%{y:.2f}<extra></extra>'  # 自定义悬停格式
)

# update_xaxes / update_yaxes：修改坐标轴
fig.update_xaxes(
    tickformat='%Y-%m',         # 日期格式（时间轴用）
    tickangle=45,               # 刻度标签旋转角度
    showgrid=True,
    gridcolor='#eeeeee'
)
fig.update_yaxes(
    tickprefix='¥',             # 前缀
    ticksuffix='元',            # 后缀
    rangemode='tozero'          # Y轴从0开始
)

# 添加参考线（如均值线、目标线）
fig.add_hline(
    y=df['sales'].mean(),
    line_dash='dash',
    line_color='red',
    annotation_text='均值线'
)
fig.add_vline(x='2024-01-01', line_dash='dot', line_color='gray')

# 添加文字注释
fig.add_annotation(
    x='2024-06', y=9500,
    text='双十一高峰',
    showarrow=True,
    arrowhead=2,
    font=dict(color='red')
)
```

### 6.3 配色速查

```python
# 定性配色（类别变量，颜色间差异明显）
px.colors.qualitative.Set2         # 柔和8色（推荐）
px.colors.qualitative.Pastel       # 粉嫩系
px.colors.qualitative.Bold         # 饱和度高，对比强

# 连续配色（数值变量，表示大小梯度）
px.colors.sequential.Blues         # 蓝色渐变
px.colors.sequential.Viridis       # 绿黄渐变（对色盲友好）
px.colors.sequential.RdYlGn        # 红黄绿（类似交通灯）
px.colors.sequential.Blugrn        # 蓝绿渐变（低饱和度，适合商务）

# 发散配色（有中间值的数据，如相关矩阵）
'RdBu_r'                           # 红蓝（_r 表示反转）
'coolwarm'                         # 冷暖色

# 手动指定颜色列表
color_discrete_sequence=['#4E79A7', '#F28E2B', '#E15759', '#76B7B2']
# 上面是 Tableau 经典配色，商务场合常用
```

---

## 7. 实战完整案例（含ML）

> 下面是一个完整的端到端仪表盘，流程：
> **数据生成 → IQR清洗 → 特征工程 → Ridge回归建模 → Dash可视化 → 双图联动**

```python
import numpy as np
import pandas as pd
import plotly.express as px
from dash import Dash, html, dcc, Input, Output
import dash_bootstrap_components as dbc
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge
from sklearn.metrics import r2_score
import warnings
warnings.filterwarnings('ignore')

# ══════════════════════════════════════════════
# STAGE 1: 数据生成与清洗
# ══════════════════════════════════════════════
np.random.seed(42)
N = 1000

df = pd.DataFrame({
    'state':                np.random.choice(['SP', 'RJ', 'MG', 'BA'], N),
    'freight_value':        np.random.uniform(10, 150, N),
    'product_weight_g':     np.random.uniform(200, 5000, N),
    'review_score':         np.random.choice([1,2,3,4,5], N, p=[0.1,0.05,0.1,0.25,0.5]),
    'delivery_latency_hrs': np.random.uniform(5, 120, N)
})

# IQR 异常值截断（clip 比 drop 更保守：截断而不删行）
for col in ['freight_value', 'delivery_latency_hrs']:
    Q1, Q3 = df[col].quantile(0.25), df[col].quantile(0.75)
    IQR = Q3 - Q1
    df[col] = df[col].clip(Q1 - 1.5*IQR, Q3 + 1.5*IQR)

print(f"清洗后数据量：{len(df)} 行")

# ══════════════════════════════════════════════
# STAGE 2: 特征工程 + Ridge 回归建模
# ══════════════════════════════════════════════
# One-Hot 编码州名（drop_first 避免多重共线性）
df_ml = pd.get_dummies(df, columns=['state'], drop_first=True)

X = df_ml.drop('delivery_latency_hrs', axis=1)
y = df_ml['delivery_latency_hrs']

# 严格按训练集/测试集划分，scaler 只在训练集上 fit
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform
X_test_scaled  = scaler.transform(X_test)         # 只 transform，防泄露

model = Ridge(alpha=10.0)
model.fit(X_train_scaled, y_train)

# 评估
r2  = r2_score(y_test, model.predict(X_test_scaled))
print(f"测试集 R²: {r2:.4f}")

# 把全量预测值写回原 df，供可视化比对
df['predicted_latency'] = model.predict(scaler.transform(X))

# ══════════════════════════════════════════════
# STAGE 3: Dash 应用布局
# ══════════════════════════════════════════════
app = Dash(__name__, external_stylesheets=[dbc.themes.FLATLY])

# 顶部指标卡：把数字包成一个可复用的函数
def make_metric_card(title, value, color="primary"):
    return dbc.Card(
        dbc.CardBody([
            html.H6(title, className="card-title text-muted mb-1"),
            html.H3(value, className=f"text-{color} fw-bold mb-0"),
        ]),
        className="shadow-sm border-0 rounded h-100"
    )

app.layout = dbc.Container([

    # ── 顶部标题栏 ──────────────────────────────
    dbc.Row(
        dbc.Col(
            html.H2(
                "📦 Olist 电商物流延迟智能分析看板",
                className="text-center text-white bg-primary p-3 rounded my-3"
            )
        )
    ),

    # ── 指标卡行 ──────────────────────────────
    dbc.Row([
        dbc.Col(make_metric_card(
            "大盘平均延迟",
            f"{df['delivery_latency_hrs'].mean():.1f} 小时",
            "primary"
        ), width=4),
        dbc.Col(make_metric_card(
            "模型 R² (测试集)",
            f"{r2:.4f}",
            "success"
        ), width=4),
        dbc.Col(make_metric_card(
            "有效订单总量",
            f"{len(df):,} 笔",
            "info"
        ), width=4),
    ], className="mb-4 g-3"),   # g-3：列间距

    # ── 主体：左侧控制面板 + 右侧图表 ──────────
    dbc.Row([

        # 左侧：筛选控件
        dbc.Col([
            html.Div([
                html.H5("🔧 筛选控制", className="text-secondary mb-3"),

                html.Label("选择州 (State)：", className="fw-bold small text-muted"),
                dcc.Dropdown(
                    id='state-dropdown',
                    options=(
                        [{'label': '🌍 全部州', 'value': 'ALL'}] +
                        [{'label': f'📍 {s}', 'value': s}
                         for s in sorted(df['state'].unique())]
                    ),
                    value='ALL',
                    clearable=False,
                    className="mb-4"
                ),

                html.Label("图表主题色：", className="fw-bold small text-muted"),
                dcc.RadioItems(
                    id='color-theme',
                    options=[
                        {'label': ' 蓝绿（冷静）', 'value': 'cool'},
                        {'label': ' 橙红（活跃）', 'value': 'warm'},
                    ],
                    value='cool',
                    className="mb-4"
                ),

                html.Hr(),
                html.Small(
                    "💡 修改上方选项，右侧图表实时联动更新。",
                    className="text-muted"
                ),
            ], className="bg-light p-4 rounded border shadow-sm")
        ], width=3),

        # 右侧：图表区（两图并排）
        dbc.Col([
            dbc.Row([
                dbc.Col(
                    dcc.Graph(id='dist-chart', style={'height': '380px'}),
                    width=6
                ),
                dbc.Col(
                    dcc.Graph(id='scatter-chart', style={'height': '380px'}),
                    width=6
                ),
            ])
        ], width=9),

    ]),

], fluid=True)

# ══════════════════════════════════════════════
# STAGE 4: Callback 数据联动
# ══════════════════════════════════════════════
@app.callback(
    [Output('dist-chart',    'figure'),   # 输出1：分布对比图
     Output('scatter-chart', 'figure')],  # 输出2：散点气泡图
    [Input('state-dropdown', 'value'),    # 输入1：选中的州
     Input('color-theme',    'value')]    # 输入2：颜色主题
)
def update_charts(selected_state, theme):
    
    # ── 步骤1：根据选择过滤数据 ──
    filtered = df if selected_state == 'ALL' else df[df['state'] == selected_state]
    # 如果过滤后数据为空（极端情况），返回空图避免报错
    if filtered.empty:
        empty_fig = px.scatter(title="暂无数据")
        return empty_fig, empty_fig

    # ── 步骤2：决定配色 ──
    color_seq = ['#4E79A7', '#E15759'] if theme == 'cool' else ['#F28E2B', '#E15759']
    scatter_cscale = px.colors.sequential.Blugrn if theme == 'cool' else px.colors.sequential.Oranges

    # ── 步骤3：图一 —— 实际值 vs 预测值分布对比直方图 ──
    # melt 把两列合并成"长表"，方便 px 用 color 区分两条曲线
    df_melt = filtered.melt(
        value_vars=['delivery_latency_hrs', 'predicted_latency'],
        var_name='类型',
        value_name='延迟（小时）'
    )
    df_melt['类型'] = df_melt['类型'].map({
        'delivery_latency_hrs': '📊 实际延迟',
        'predicted_latency':    '🤖 模型预测'
    })

    fig_dist = px.histogram(
        df_melt,
        x='延迟（小时）', color='类型',
        barmode='overlay', opacity=0.65,
        marginal='box',                         # 顶部附加箱线图
        nbins=30,
        color_discrete_sequence=color_seq,
        title=f"实际 vs 预测延迟分布｜{selected_state}"
    )
    fig_dist.update_layout(
        template='plotly_white',
        legend=dict(orientation='h', y=-0.25),
        margin=dict(t=50, b=60)
    )

    # ── 步骤4：图二 —— 运费 × 延迟散点气泡图 ──
    fig_scatter = px.scatter(
        filtered,
        x='freight_value',
        y='delivery_latency_hrs',
        size='product_weight_g',                # 气泡大小 = 商品重量
        color='review_score',                   # 气泡颜色 = 评分
        color_continuous_scale=scatter_cscale,
        labels={
            'freight_value':        '运费（雷亚尔）',
            'delivery_latency_hrs': '交付延迟（小时）',
            'review_score':         '评分',
            'product_weight_g':     '重量（g）'
        },
        title=f"运费 × 延迟 × 评分交叉分析｜{selected_state}"
    )
    fig_scatter.update_layout(
        template='plotly_white',
        margin=dict(t=50)
    )

    return fig_dist, fig_scatter   # 顺序对应上方 Output 列表

# ══════════════════════════════════════════════
# STAGE 5: 启动
# ══════════════════════════════════════════════
if __name__ == '__main__':
    app.run(debug=True, port=8050)
    # 打开浏览器访问 http://127.0.0.1:8050
```

---

## 8. 常见报错与解决

| 报错 / 症状 | 原因 | 解决方法 |
|------------|------|---------|
| `Callback outputs must have unique IDs` | 两个 Output 指向同一个组件的同一个属性 | 检查 callback 里有没有重复的 Output |
| `A nonexistent object was used in an Input` | callback 里引用的 id 在 layout 里不存在 | 检查 id 拼写，或确认组件已经加入 layout |
| `figure` 接收到 `None` | callback 函数在某些条件下没有 return | 所有分支都要有 return，或用 `no_update` |
| 下拉框清空后报错 | `clearable=True` 时 value 变成 `None` | 加 `if value is None: return no_update` |
| 页面加载时 callback 报错 | 初始调用时某些 Input 为空 | 加 `prevent_initial_call=True` |
| 多个 Output 数量对不上 | return 的值数量和 Output 列表长度不一致 | return 的值必须和 Output 数量、顺序完全一致 |
| `dcc.Store` 数据为空 | DataFrame 没有转成 JSON 兼容格式 | 用 `df.to_dict('records')` 存入，`pd.DataFrame(data)` 还原 |
| 图表不更新 | callback 触发了但数据没变 | 用 `print` 调试确认函数被调用，检查过滤逻辑 |
| 端口被占用 | 8050 端口已有进程在跑 | 换端口 `port=8051`，或终止原进程 |

### 调试技巧

```python
# 1. 在 callback 里加 print，查看参数值
def update(value):
    print(f"[DEBUG] callback triggered, value={value}")
    ...

# 2. debug=True 时，浏览器控制台和终端都会显示错误信息
app.run(debug=True)

# 3. 用 raise PreventUpdate 跳过本次更新（比 no_update 更彻底）
from dash.exceptions import PreventUpdate

def update(value):
    if value is None:
        raise PreventUpdate   # 直接跳过，不更新任何 Output
    ...
```


## IBM 教科书案例
```python
import pandas as pd
import dash
from dash import html, dcc
from dash.dependencies import Input, Output, State
import plotly.graph_objects as go
import plotly.express as px
from dash import no_update
import datetime as dt
#Create app
app = dash.Dash(__name__)
#Clear the layout and do not display exception till callback gets executed
app.config.suppress_callback_exceptions = True
# Read the wildfire data into pandas dataframe
df =  pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/Historical_Wildfires.csv')
#Extract year and month from the date column
df['Month'] = pd.to_datetime(df['Date']).dt.month_name() #used for the names of the months
df['Year'] = pd.to_datetime(df['Date']).dt.year
#Layout Section of Dash
#Task 1 Add the Title to the Dashboard
app.layout = html.Div(children=[html.H1('Australia Wildfire Dashboard', 
                                style={'textAlign': 'center', 'color': '#503D36',
                                'font-size': 26}),
# TASK 2: Add the radio items and a dropdown right below the first inner division
     #outer division starts
     html.Div([
                   # First inner divsion for  adding dropdown helper text for Selected Drive wheels
                    html.Div([
                            html.H2('Select Region:', style={'margin-right': '2em'}),

                    #Radio items to select the region
                    #dcc.RadioItems(['NSW','QL','SA','TA','VI','WA'], 'NSW', id='region',inline=True)]),
                    dcc.RadioItems([{"label":"New South Wales","value": "NSW"},
                                    {"label":"Northern Territory","value": "NT"},
                                    {"label":"Queensland","value": "QL"},
                                    {"label":"South Australia","value": "SA"},
                                    {"label":"Tasmania","value": "TA"},
                                    {"label":"Victoria","value": "VI"},
                                    {"label":"Western Australia","value": "WA"}],"NSW", id='region',inline=True)]),
                    #Dropdown to select year
                    html.Div([
                            html.H2('Select Year:', style={'margin-right': '2em'}),
                        dcc.Dropdown(df.Year.unique(), value = 2005,id='year')
                    ]),
#TASK 3: Add two empty divisions for output inside the next inner division. 
         #Second Inner division for adding 2 inner divisions for 2 output graphs
                    html.Div([
                
                        html.Div([ ], id='plot1'),
                        html.Div([ ], id='plot2')
                    ], style={'display': 'flex'}),

    ])
    #outer division ends

])
#layout ends
#TASK 4: Add the Ouput and input components inside the app.callback decorator.
#Place to add @app.callback Decorator
@app.callback([Output(component_id='plot1', component_property='children'),
               Output(component_id='plot2', component_property='children')],
               [Input(component_id='region', component_property='value'),
                Input(component_id='year', component_property='value')])
#TASK 5: Add the callback function.   
#Place to define the callback function .
def reg_year_display(input_region,input_year):  
    #data
   region_data = df[df['Region'] == input_region]
   y_r_data = region_data[region_data['Year']==input_year]
    #Plot one - Monthly Average Estimated Fire Area   
   est_data = y_r_data.groupby('Month')['Estimated_fire_area'].mean().reset_index()
   fig1 = px.pie(est_data, values='Estimated_fire_area', names='Month', title="{} : Monthly Average Estimated Fire Area in year {}".format(input_region,input_year))   
     #Plot two - Monthly Average Count of Pixels for Presumed Vegetation Fires
   veg_data = y_r_data.groupby('Month')['Count'].mean().reset_index()
   fig2 = px.bar(veg_data, x='Month', y='Count', title='{} : Average Count of Pixels for Presumed Vegetation Fires in year {}'.format(input_region,input_year))    
   return [dcc.Graph(figure=fig1),
            dcc.Graph(figure=fig2) ]
if __name__ == '__main__':
    app.run()
    

```
