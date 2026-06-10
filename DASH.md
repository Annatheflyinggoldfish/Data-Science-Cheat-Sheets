# Plotly Dash 完整 Cheat Sheet + 实战案例

> 适合：刚学完基础，细节还不太清楚的同学

---

## 目录

1. [Dash 是什么？](#1-dash-是什么)
2. [必须导入的包](#2-必须导入的包)
3. [创建 App 及常用配置](#3-创建-app-及常用配置)
4. [Layout 速查 — html 组件](#4-layout-速查--html-组件)
5. [Layout 速查 — dcc 组件（交互组件）](#5-layout-速查--dcc-组件交互组件)
6. [Callback 完整语法速查](#6-callback-完整语法速查)
7. [dcc.Store 实战用法](#7-dccstore-实战用法)
8. [Plotly 图表完整速查](#8-plotly-图表完整速查)
9. [布局技巧 — Flexbox 排列](#9-布局技巧--flexbox-排列)
10. [常见报错速查](#10-常见报错速查)
11. [实战案例：汽车销售 Dashboard](#11-实战案例汽车销售-dashboard)

---

## 1. Dash 是什么？

**Dash = Flask（服务器）+ React（前端）+ Plotly（图表）**

你写的是 Python，Dash 帮你自动转成网页。

```
┌─────────┐    ┌──────────┐    ┌──────────┐
│ Layout  │───>│ Callback │───>│  Output  │
│(页面长啥样)│   │(交互逻辑) │    │(更新内容) │
└─────────┘    └──────────┘    └──────────┘
```

| 概念 | 说明 |
|------|------|
| **Layout** | 静态结构，页面初始长什么样 |
| **Callback** | 动态逻辑，用户操作后发生什么 |
| **id** | 两者通过组件的 `id` 连接 |

---

## 2. 必须导入的包

```python
import dash
from dash import html          # HTML 标签组件，比如 div、h1、p
from dash import dcc           # Dash Core Components，下拉框、图表、滑块等
from dash import no_update     # 告诉 Dash"这次不更新"
from dash.dependencies import Input, Output, State
#   Input  = 触发 callback 的输入（用户操作改变时触发）
#   Output = callback 执行后要更新的目标组件属性
#   State  = 读取某组件的值，但【不触发】callback
import plotly.express as px    # 快速画图（推荐新手用）
import plotly.graph_objects as go  # 更精细的画图控制
import pandas as pd
```

---

## 3. 创建 App 及常用配置

```python
app = dash.Dash(__name__)
# __name__ 告诉 Dash 当前文件在哪，用来找静态资源（CSS/图片等）

app.config.suppress_callback_exceptions = True
# 当 callback 引用的组件还不在 layout 里时不报错
# 动态生成组件时必须开启

app.title = '我的 Dashboard'
# 设置浏览器标签页的标题

if __name__ == '__main__':
    app.run(debug=True)
# debug=True：代码改动后自动重载，报错显示详细信息
# debug=False：生产环境用，不暴露错误细节
```

---

## 4. Layout 速查 — html 组件

```python
html.Div(children=[...], id='my-div', className='box',
         style={'color': 'red', 'fontSize': 16})
# 相当于 HTML 的 <div>，是最常用的容器
```

> ⚠️ **CSS 属性名用驼峰法！**
> - `font-size` → `fontSize`（或字符串 `'font-size'` 也行）
> - `text-align` → `textAlign`
> - `background-color` → `backgroundColor`
> - `border-radius` → `borderRadius`

| 组件 | 说明 |
|------|------|
| `html.H1` / `html.H2` / `html.H3` | 标题（字号依次减小） |
| `html.P` | 段落文字 |
| `html.Div` | 容器，最常用，用来分组和布局 |
| `html.Span` | 行内容器（不换行） |
| `html.Br` | 换行 |
| `html.Hr` | 水平分隔线 |
| `html.Button` | 按钮（配合 `n_clicks` 用） |
| `html.A` | 超链接，`href='url'` |
| `html.Img` | 图片，`src='路径'` |
| `html.Label` | 表单标签（给输入框加说明文字） |
| `html.Table` / `html.Tr` / `html.Td` | 表格 |

**`children` 参数的几种写法：**

```python
html.H1('标题')                          # 字符串
html.Div(html.P('文字'))                 # 单个组件
html.Div([html.H1('标题'), html.P('内容')])  # 组件列表
html.Div([...])                          # 省略 children= 直接传
```

---

## 5. Layout 速查 — dcc 组件（交互组件）

### dcc.Dropdown — 下拉选择框

```python
dcc.Dropdown(
    id='my-dropdown',            # ← 必须有！callback 靠这个找到它
    options=[                    # 选项列表，每项是一个字典
        {'label': '显示的文字', 'value': '传给 callback 的值'},
        {'label': '选项2',     'value': 'option2'},
    ],
    value='option2',             # 默认选中的值（对应 value 字段）
    placeholder='请选择...',      # 没选时的灰色提示文字
    multi=False,                 # True = 可以多选，value 变成列表
    disabled=False,              # True = 禁用（灰色不可点）
    clearable=True,              # 是否显示清除按钮（×）
    searchable=True,             # 是否可以搜索选项
    style={'width': '50%'}
)
```

> ⚠️ `multi=True` 时，`value` 是列表，callback 参数也是列表！

---

### dcc.Graph — 图表容器

```python
dcc.Graph(
    id='my-graph',
    figure=px.line(df, x='Year', y='Sales'),
    # figure 是 plotly 生成的图表对象
    # 也可以先不传 figure，由 callback 动态填充
    style={'height': '400px'}    # 控制图表高度
)
```

---

### dcc.Slider — 滑块

```python
dcc.Slider(
    id='my-slider',
    min=0, max=100, step=10,     # 最小值、最大值、步长
    value=50,                    # 默认值
    marks={
        0: '0%',
        50: {'label': '50%', 'style': {'color': 'red'}},  # 可自定义样式
        100: '100%'
    },
    tooltip={'placement': 'bottom', 'always_visible': True}
    # tooltip：拖动时显示当前值的提示框
)
```

---

### dcc.RangeSlider — 范围滑块

```python
dcc.RangeSlider(
    id='range-slider',
    min=0, max=100, step=5,
    value=[20, 80],              # ← 注意是列表！两个值分别是左右端点
    marks={0: '0', 100: '100'}
)
```

---

### dcc.RadioItems — 单选按钮

```python
dcc.RadioItems(
    id='my-radio',
    options=[{'label': '选项A', 'value': 'a'},
             {'label': '选项B', 'value': 'b'}],
    value='a',                   # 默认选中
    inline=True,                 # True=横排, False=竖排（默认）
    labelStyle={'marginRight': '20px'}  # 调整每个选项的间距
)
```

---

### dcc.Checklist — 复选框

```python
dcc.Checklist(
    id='my-check',
    options=[{'label': '选X', 'value': 'x'},
             {'label': '选Y', 'value': 'y'}],
    value=['x'],                 # 默认勾选的值（是列表！）
    inline=True
)
```

> ⚠️ `value` 永远是列表，即使只选了一个！

---

### dcc.Input — 文本输入框

```python
dcc.Input(
    id='my-input',
    type='text',                 # 'text' / 'number' / 'password' / 'email'
    placeholder='输入内容...',
    value='',                    # 初始值
    debounce=True,               # True=按回车/失焦才触发, False=实时触发
    min=0, max=100,              # type='number' 时可限制范围
    style={'width': '200px'}
)
```

---

### dcc.Textarea — 多行文本框

```python
dcc.Textarea(
    id='my-textarea',
    value='初始文字',
    style={'width': '100%', 'height': 150}
)
```

---

### dcc.DatePickerSingle — 单日期选择

```python
dcc.DatePickerSingle(
    id='my-date',
    date='2023-01-01',
    min_date_allowed='2020-01-01',
    max_date_allowed='2025-12-31',
    display_format='YYYY-MM-DD'
)
```

### dcc.DatePickerRange — 日期范围选择

```python
dcc.DatePickerRange(
    id='date-range',
    start_date='2023-01-01',
    end_date='2023-12-31'
)
```

---

### dcc.Loading — 加载动画

```python
dcc.Loading(
    id='loading',
    type='circle',               # 'circle' / 'dot' / 'cube' / 'default'
    children=[
        dcc.Graph(id='slow-graph')  # 把需要等待的组件放在 children 里
    ]
)
# 效果：当 children 里的组件正在被 callback 更新时，显示加载动画
```

---

### dcc.Store — 客户端数据存储

```python
dcc.Store(
    id='my-store',
    storage_type='session',      # 'memory'=关标签消失
                                 # 'session'=关浏览器消失
                                 # 'local'=永久保存
    data={}                      # 初始数据（任意 JSON 可序列化的对象）
)
# 用途：在多个 callback 之间共享数据，避免重复计算
# 在 callback 里当 Output 写入，当 Input 读取（就像变量一样）
```

---

### dcc.Interval — 定时器

```python
dcc.Interval(
    id='my-interval',
    interval=5000,               # 每隔多少毫秒触发一次（5000=5秒）
    n_intervals=0,               # 已触发次数（从0开始自动累加）
    disabled=False               # True=暂停定时器
)
# 用途：实现自动刷新（比如实时数据更新）
# 在 callback 里用 Input('my-interval', 'n_intervals') 触发
```

---

## 6. Callback 完整语法速查

### 最基础的形式

```python
@app.callback(
    Output('输出组件的id', '要更新的属性'),
    Input('输入组件的id',  '监听的属性')
)
def 函数名(input的当前值):
    # 这里写处理逻辑
    return 返回值   # 返回值会被塞进 Output 指定的属性
```

---

### 常见 component_property 速查

| 组件 | 常用属性 | 说明 |
|------|----------|------|
| `dcc.Dropdown` | `value` | 当前选中的值 |
| `dcc.Dropdown` | `disabled` | 是否禁用（True/False） |
| `dcc.Dropdown` | `options` | 选项列表 |
| `dcc.Graph` | `figure` | 图表对象 |
| `dcc.Slider` | `value` | 当前滑块值 |
| `dcc.Input` | `value` | 输入框内容 |
| `dcc.Store` | `data` | 存储的数据 |
| `dcc.Interval` | `n_intervals` | 已触发次数 |
| `html.Div` | `children` | 内部内容（文字/组件/列表） |
| `html.Div` | `style` | CSS 样式字典 |
| `html.Button` | `n_clicks` | 被点击的次数 |
| `html.H1` | `children` | 标题文字 |
| 任意组件 | `className` | CSS class 名 |

---

### 多个 Input

任意一个变化都会触发 callback：

```python
@app.callback(
    Output('output', 'children'),
    [Input('dd1', 'value'),
     Input('dd2', 'value')]      # ← 多个 Input 放进列表
)
def my_func(val1, val2):         # ← 参数顺序必须和 Input 顺序一致
    return f'{val1} + {val2}'
```

---

### 多个 Output

```python
@app.callback(
    [Output('graph1', 'figure'),
     Output('graph2', 'figure')],  # ← 多个 Output 放进列表
    Input('dropdown', 'value')
)
def my_func(val):
    fig1 = px.line(...)
    fig2 = px.bar(...)
    return fig1, fig2             # ← 返回顺序必须和 Output 顺序一致
```

---

### State — 读值但不触发

```python
@app.callback(
    Output('output', 'children'),
    Input('submit-btn', 'n_clicks'),  # ← 点按钮才触发
    State('input-box', 'value')       # ← 只读值，不触发
)
def on_click(n_clicks, input_value):
    if n_clicks is None:          # 页面刚加载时 n_clicks=None，要防护
        return '还没点击'
    return f'你输入了: {input_value}'
```

> **State 和 Input 的区别：**
> - `Input` = 一旦值变化就触发 callback
> - `State` = 值变化时不触发，只是在 callback 被触发时顺便读一下

---

### no_update — 不更新某个 Output

```python
from dash import no_update

@app.callback(
    [Output('graph', 'figure'),
     Output('error-msg', 'children')],
    Input('dropdown', 'value')
)
def update(val):
    if val is None:
        return no_update, '请先选择一个选项'
        # ↑ graph 不更新，只更新 error-msg
    fig = px.line(...)
    return fig, ''
```

用途：条件性地跳过某些 Output 的更新，避免不必要的重绘。

---

### prevent_initial_call — 阻止首次自动触发

```python
@app.callback(
    Output('output', 'children'),
    Input('button', 'n_clicks'),
    prevent_initial_call=True     # ← 页面加载时不自动触发
)
def on_click(n_clicks):
    return f'点击了 {n_clicks} 次'
```

> **默认行为**：页面加载时所有 callback 会自动触发一次（用初始值）。
> 加上 `prevent_initial_call=True` 可以阻止这个行为。
> 常用场景：按钮点击、避免页面加载时报错。

---

### Callback 链 — 一个 callback 的 Output 触发另一个

```python
# 场景：选了省份 → 自动更新城市列表 → 再选城市 → 更新图表

@app.callback(
    Output('city-dropdown', 'options'),
    Input('province-dropdown', 'value')
)
def update_cities(province):
    cities = get_cities(province)
    return [{'label': c, 'value': c} for c in cities]

@app.callback(
    Output('map', 'figure'),
    Input('city-dropdown', 'value')   # ← 上一个 callback 的 Output 变化时触发
)
def update_map(city):
    return px.scatter_mapbox(...)
```

> ⚠️ 不能形成循环（A→B→A 会报错）

---

## 7. dcc.Store 实战用法

```python
# 场景：过滤数据很慢，想只过滤一次，供多个图表共用

# Layout 里放一个 Store（隐形的，用户看不到）：
dcc.Store(id='filtered-data', storage_type='memory')

# Callback 1：把过滤结果存进 Store
@app.callback(
    Output('filtered-data', 'data'),    # ← 往 Store 里写
    Input('year-dropdown', 'value')
)
def filter_data(year):
    filtered = data[data['Year'] == year]
    return filtered.to_dict('records')  # ← 必须转成 JSON 可序列化的格式

# Callback 2：从 Store 读数据画图（不用重新过滤）
@app.callback(
    Output('my-graph', 'figure'),
    Input('filtered-data', 'data')      # ← 从 Store 里读
)
def update_graph(stored_data):
    df = pd.DataFrame(stored_data)      # ← 从字典列表还原成 DataFrame
    return px.line(df, x='Month', y='Sales')
```

---

## 8. Plotly 图表完整速查

### Plotly Express（px）— 快速画图

#### 折线图

```python
px.line(df, x='col1', y='col2',
    title='标题',
    color='分组列',           # 按某列分颜色画多条线
    line_dash='列名',         # 按某列画不同虚实线
    markers=True,             # 在折线上显示圆点
    labels={'col1': '自定义轴标签'},
    hover_data=['额外列'],    # 鼠标悬浮时额外显示的列
)
```

#### 柱状图

```python
px.bar(df, x='col1', y='col2',
    color='分组列',
    barmode='group',          # 'group'=并排, 'stack'=堆叠, 'overlay'=重叠
    text='col2',              # 在柱子上显示数值
    orientation='h',          # 'h'=水平柱状图，默认是垂直
)
```

#### 饼图

```python
px.pie(df, values='数值列', names='分类列',
    hole=0.3,                 # 0~1，>0 变成甜甜圈图
    color_discrete_map={      # 手动指定每个分类的颜色
        '类别A': '#ff0000',
        '类别B': '#0000ff'
    }
)
```

#### 散点图

```python
px.scatter(df, x='col1', y='col2',
    size='气泡大小列',        # 有这个参数就变成气泡图
    color='分组列',
    trendline='ols',          # 添加趋势线（需要 statsmodels）
    hover_name='标签列',      # 鼠标悬浮时显示的主标签
)
```

#### 箱线图

```python
px.box(df, x='分类列', y='数值列',
    color='分组列',
    points='all',             # 'all'=显示所有点, 'outliers'=只显示异常值
)
```

#### 直方图

```python
px.histogram(df, x='col1',
    nbins=20,                 # 分组数量
    histnorm='probability',   # 'probability'=归一化为概率
    color='分组列',
)
```

#### 热力图

```python
px.imshow(df_matrix,          # 传入二维矩阵或 DataFrame
    color_continuous_scale='RdBu',
    text_auto=True,           # 在格子里显示数值
)
```

#### 地图散点图

```python
px.scatter_mapbox(df, lat='纬度列', lon='经度列',
    size='数值列',
    color='分类列',
    mapbox_style='open-street-map',  # 不需要 token
    zoom=5,
)
```

---

### 图表样式自定义

```python
fig = px.line(df, x='Year', y='Sales')

fig.update_layout(
    title={'text': '标题', 'x': 0.5, 'xanchor': 'center'},  # 标题居中
    xaxis_title='X轴标签',
    yaxis_title='Y轴标签',
    font={'family': 'Arial', 'size': 14, 'color': '#333'},
    plot_bgcolor='white',     # 图表区背景色
    paper_bgcolor='#f8f9fa',  # 整体背景色
    legend={'title': '图例标题', 'x': 1, 'y': 1},
    hovermode='x unified',   # 同一 x 值的点一起显示
    margin={'l': 40, 'r': 40, 't': 60, 'b': 40},
    height=400,
)

fig.update_traces(
    line={'color': '#e74c3c', 'width': 2},
    marker={'size': 8, 'symbol': 'circle'},
    opacity=0.8,
)

fig.update_xaxes(
    tickangle=45,             # 刻度标签旋转角度
    showgrid=True,
    gridcolor='#eee',
    range=[2000, 2020],       # 固定坐标轴范围
)

fig.update_yaxes(
    tickformat=',.0f',        # 千分位不带小数
    tickprefix='$',
    ticksuffix=' 辆',
)
```

---

### Plotly Graph Objects（go）— 精细控制

比 `px` 更底层，适合高度自定义或叠加多种图表类型：

```python
fig = go.Figure()

fig.add_trace(go.Scatter(
    x=df['Year'], y=df['Sales'],
    mode='lines+markers',     # 'lines' / 'markers' / 'lines+markers'
    name='销量',
    line={'color': 'blue', 'width': 2}
))

fig.add_trace(go.Bar(
    x=df['Year'], y=df['Profit'],
    name='利润',
    yaxis='y2',               # 使用第二条 Y 轴
))

fig.update_layout(
    yaxis2={'overlaying': 'y', 'side': 'right'}  # 右侧第二 Y 轴
)
```

---

## 9. 布局技巧 — Flexbox 排列

### 两图并排

```python
html.Div([
    html.Div(dcc.Graph(...), style={'flex': 1}),
    html.Div(dcc.Graph(...), style={'flex': 1}),
], style={'display': 'flex', 'gap': '10px'})
```

### 不等宽（左1右2）

```python
html.Div([
    html.Div(dcc.Graph(...), style={'flex': 1}),  # 占1份
    html.Div(dcc.Graph(...), style={'flex': 2}),  # 占2份（是左边的2倍宽）
], style={'display': 'flex'})
```

### 2行2列网格

```python
html.Div([
    html.Div([图1, 图2], style={'display': 'flex'}),  # 第一行
    html.Div([图3, 图4], style={'display': 'flex'}),  # 第二行
])
```

### 自动换行（响应式）

```python
html.Div([图1, 图2, 图3, 图4],
    style={
        'display': 'flex',
        'flexWrap': 'wrap',    # 放不下就自动换行
        'gap': '10px'
    }
)
```

### Flex 常用属性总结

| 属性 | 说明 |
|------|------|
| `display: flex` | 开启 flex 布局（子元素横排） |
| `flex: 1` | 平均分配剩余空间 |
| `flexWrap: wrap` | 放不下自动换行 |
| `gap: 10px` | 子元素之间的间距 |
| `alignItems: center` | 垂直居中 |
| `justifyContent: center` | 水平居中 |

---

## 10. 常见报错速查

| 报错 | 原因 | 解决方法 |
|------|------|----------|
| `SyntaxError: invalid syntax`（`options=......`） | skeleton 占位符没填 | 换成真实代码 |
| `KeyError: 'Automobile_Sales'` | 列名写错 | `print(df.columns.tolist())` 检查 |
| `Duplicate callback outputs` | 两个 callback 输出了同一个属性 | 合并成一个 callback |
| `nonexistent object with ID "xxx"` | callback 引用了不存在的 id | 加 `suppress_callback_exceptions=True` |
| 页面空白 / callback 不触发 | id 或 property 拼写错误 | 检查大小写是否完全一致 |
| callback 返回 `None` 报错 | 函数某个分支没有 return | 用 `[]` 或 `''` 代替 `None` |
| 图表不更新但没报错 | Input id 拼写错误（Dash 不提示） | 检查 `prevent_initial_call=True` |
| `Circular dependency` 循环依赖 | A→B→A 形成循环 | 用 `dcc.Store` 作为中间层打断循环 |

---

## 11. 实战案例：汽车销售 Dashboard

> 在原作业基础上增加了：统计卡片、`dcc.Loading` 加载动画、图表美化

```python
import dash
from dash import html, dcc, no_update
from dash.dependencies import Input, Output
import plotly.express as px
import pandas as pd

app = dash.Dash(__name__)
app.config.suppress_callback_exceptions = True
app.title = 'Automobile Sales Dashboard'

# ── 读取数据 ──────────────────────────────────────────────
data = pd.read_csv(
    'https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/'
    'IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/'
    'historical_automobile_sales.csv'
)

# 用数据里实际存在的年份，比 range(1980, 2024) 更准确
year_list = sorted(data['Year'].unique())


# ════════════════════════════════════════════════════════════
#  LAYOUT
# ════════════════════════════════════════════════════════════

app.layout = html.Div(
    style={'fontFamily': 'Arial, sans-serif',
           'backgroundColor': '#f8f9fa',
           'padding': '20px',
           'minHeight': '100vh'},
    children=[

        # 标题
        html.H1(
            'Automobile Sales Statistics Dashboard',
            style={'textAlign': 'center', 'color': '#503D36', 'fontSize': 28,
                   'borderBottom': '2px solid #503D36', 'paddingBottom': '10px',
                   'marginBottom': '20px'}
        ),

        # 控制区：两个下拉框横排
        html.Div(
            style={'display': 'flex', 'gap': '40px', 'marginBottom': '20px',
                   'backgroundColor': 'white', 'padding': '15px',
                   'borderRadius': '8px', 'boxShadow': '0 2px 4px rgba(0,0,0,0.1)'},
            children=[

                # 左侧：报告类型
                html.Div(style={'flex': 1}, children=[
                    html.Label('📊 选择报告类型',
                               style={'fontWeight': 'bold', 'marginBottom': '8px',
                                      'display': 'block'}),
                    dcc.Dropdown(
                        id='dropdown-statistics',
                        options=[
                            {'label': '📅 Yearly Statistics',
                             'value': 'Yearly Statistics'},
                            {'label': '📉 Recession Period Statistics',
                             'value': 'Recession Period Statistics'}
                        ],
                        value='Yearly Statistics',
                        style={'fontSize': '16px'}
                    )
                ]),

                # 右侧：年份（disabled 由 callback 控制）
                html.Div(style={'flex': 1}, children=[
                    html.Label('📆 选择年份（仅年度报告可用）',
                               style={'fontWeight': 'bold', 'marginBottom': '8px',
                                      'display': 'block'}),
                    dcc.Dropdown(
                        id='select-year',
                        options=[{'label': str(y), 'value': y} for y in year_list],
                        value=year_list[-1],
                        style={'fontSize': '16px'}
                    )
                ]),
            ]
        ),

        # 统计卡片区（空容器，由 callback 填充）
        html.Div(id='stats-cards',
                 style={'display': 'flex', 'gap': '15px', 'marginBottom': '20px'}),

        # 图表区（dcc.Loading 包裹，加载时显示动画）
        dcc.Loading(
            type='circle',
            children=[html.Div(id='output-container')]
        ),
    ]
)


# ════════════════════════════════════════════════════════════
#  CALLBACK 1：控制年份下拉框启用/禁用
# ════════════════════════════════════════════════════════════

@app.callback(
    Output('select-year', 'disabled'),
    Input('dropdown-statistics', 'value')
)
def update_input_container(selected_statistics):
    # Yearly Statistics → False（可用）
    # Recession         → True（禁用）
    return selected_statistics != 'Yearly Statistics'


# ════════════════════════════════════════════════════════════
#  CALLBACK 2：生成图表和统计卡片
# ════════════════════════════════════════════════════════════

@app.callback(
    [Output('output-container', 'children'),
     Output('stats-cards',      'children')],
    [Input('dropdown-statistics', 'value'),
     Input('select-year',         'value')]
)
def update_output_container(selected_statistics, input_year):

    # 工具函数：生成统计卡片
    def make_card(title, value, color='#503D36'):
        return html.Div(
            style={'backgroundColor': 'white', 'padding': '15px 20px',
                   'borderRadius': '8px', 'borderLeft': f'4px solid {color}',
                   'boxShadow': '0 2px 4px rgba(0,0,0,0.1)', 'flex': 1},
            children=[
                html.P(title,  style={'color': '#666', 'margin': 0, 'fontSize': 13}),
                html.H3(value, style={'color': color, 'margin': '5px 0 0 0'})
            ]
        )

    # 工具函数：给图表加白色卡片背景
    def wrap_graph(graph):
        return html.Div(
            style={'flex': 1, 'backgroundColor': 'white', 'borderRadius': '8px',
                   'boxShadow': '0 2px 4px rgba(0,0,0,0.1)', 'margin': '5px',
                   'overflow': 'hidden'},
            children=[graph]
        )

    # ── 分支一：衰退期统计 ────────────────────────────────
    if selected_statistics == 'Recession Period Statistics':

        recession_data = data[data['Recession'] == 1]

        # 统计卡片
        cards = [
            make_card('衰退期总销售量',
                      f"{int(recession_data['Automobile_Sales'].sum()):,} 辆", '#e74c3c'),
            make_card('衰退期月均销量',
                      f"{round(recession_data['Automobile_Sales'].mean(), 1):,} 辆", '#e67e22'),
            make_card('涉及衰退年份数',
                      f"{recession_data['Year'].nunique()} 年", '#9b59b6'),
        ]

        # 图1：折线图 — 衰退期年均销量
        # groupby('Year') 按年分组，.mean() 求均值，.reset_index() 还原为普通列
        yearly_rec = recession_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        R_chart1 = dcc.Graph(figure=px.line(yearly_rec, x='Year', y='Automobile_Sales',
            title='衰退期各年平均汽车销量', markers=True,
            labels={'Automobile_Sales': '平均销量（辆）'}))

        # 图2：柱状图 — 各车型平均销量
        avg_sales = recession_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        R_chart2 = dcc.Graph(figure=px.bar(avg_sales, x='Vehicle_Type', y='Automobile_Sales',
            title='衰退期各车型平均销量', color='Vehicle_Type',
            labels={'Automobile_Sales': '平均销量（辆）', 'Vehicle_Type': '车型'}))

        # 图3：饼图 — 各车型广告支出占比（用 sum() 看总占比）
        exp_rec = recession_data.groupby('Vehicle_Type')['Advertising_Expenditure'].sum().reset_index()
        R_chart3 = dcc.Graph(figure=px.pie(exp_rec, values='Advertising_Expenditure',
            names='Vehicle_Type', title='衰退期各车型广告支出占比', hole=0.3))

        # 图4：分组柱状图 — 失业率对各车型销量的影响
        # 按 [失业率, 车型] 两个维度分组
        unemp = recession_data.groupby(['unemployment_rate', 'Vehicle_Type'])['Automobile_Sales'].mean().reset_index()
        R_chart4 = dcc.Graph(figure=px.bar(unemp, x='unemployment_rate', y='Automobile_Sales',
            color='Vehicle_Type', barmode='group',
            title='失业率对各车型销量的影响',
            labels={'unemployment_rate': '失业率（%）', 'Automobile_Sales': '平均销量（辆）'}))

        # 2行2列布局
        charts = [
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(R_chart1), wrap_graph(R_chart2)]),
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(R_chart3), wrap_graph(R_chart4)])
        ]
        return charts, cards

    # ── 分支二：年度统计 ──────────────────────────────────
    elif input_year and selected_statistics == 'Yearly Statistics':

        yearly_data = data[data['Year'] == input_year]

        # 统计卡片
        # idxmax() 返回最大值对应的索引（这里是车型名）
        top_vehicle = yearly_data.groupby('Vehicle_Type')['Automobile_Sales'].sum().idxmax()
        cards = [
            make_card(f'{input_year} 年总销量',
                      f"{int(yearly_data['Automobile_Sales'].sum()):,} 辆", '#27ae60'),
            make_card(f'{input_year} 月均销量',
                      f"{round(yearly_data['Automobile_Sales'].mean(), 1):,} 辆", '#2980b9'),
            make_card(f'{input_year} 销量冠军车型', top_vehicle, '#f39c12'),
        ]

        # 图1：折线图 — 全时段年均销量（用全部 data，不是 yearly_data）
        yas = data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        Y_chart1 = dcc.Graph(figure=px.line(yas, x='Year', y='Automobile_Sales',
            title='全时段年均汽车销量趋势', markers=True,
            labels={'Automobile_Sales': '平均销量（辆）'}))

        # 图2：折线图 — 选定年份月度销量
        # sum() 因为同一个月有多条记录（不同车型），需要汇总
        mas = yearly_data.groupby('Month')['Automobile_Sales'].sum().reset_index()
        Y_chart2 = dcc.Graph(figure=px.line(mas, x='Month', y='Automobile_Sales',
            title=f'{input_year} 年月度总销量', markers=True,
            labels={'Automobile_Sales': '总销量（辆）'}))

        # 图3：柱状图 — 该年各车型平均销量
        avr = yearly_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        Y_chart3 = dcc.Graph(figure=px.bar(avr, x='Vehicle_Type', y='Automobile_Sales',
            color='Vehicle_Type', title=f'{input_year} 年各车型平均销量',
            labels={'Automobile_Sales': '平均销量（辆）', 'Vehicle_Type': '车型'}))

        # 图4：饼图 — 该年各车型广告支出
        exp = yearly_data.groupby('Vehicle_Type')['Advertising_Expenditure'].sum().reset_index()
        Y_chart4 = dcc.Graph(figure=px.pie(exp, values='Advertising_Expenditure',
            names='Vehicle_Type', title=f'{input_year} 年各车型广告支出占比', hole=0.3))

        charts = [
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(Y_chart1), wrap_graph(Y_chart2)]),
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(Y_chart3), wrap_graph(Y_chart4)])
        ]
        return charts, cards

    else:
        return [], []


if __name__ == '__main__':
    app.run(debug=True)
```

---

*Made with ❤️ | Licensed under Apache 2.0*
