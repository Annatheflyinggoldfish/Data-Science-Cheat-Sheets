# PLOTLY DASH 完整 CHEAT SHEET + 实战案例
- Dash = Flask（服务器）+ React（前端）+ Plotly（图表）

- 工作流程：
- Layout(页面长啥样)  ───> Callback(交互逻辑) ───>  Output(更新内容)  
      

## 2. 必须导入的包
```python
import dash
from dash import html        # HTML 标签组件，比如 div、h1、p
from dash import dcc         # Dash Core Components，比如下拉框、图表、滑块
from dash.dependencies import Input, Output, State
#   Input  = 触发 callback 的输入（用户操作）
#   Output = callback 执行后要更新的地方
#   State  = 读取某组件的值，但不触发 callback（不常用但有用）
import plotly.express as px  # 快速画图
import plotly.graph_objects as go  # 更精细的画图控制
import pandas as pd
```

## 3. 创建 APP
```python
#__name__ 告诉 Dash 当前文件在哪，用来找静态资源
app = dash.Dash(__name__)

# suppress_callback_exceptions=True：
# 当 callback 里引用的组件还不存在于 layout 时不报错
# 比如动态生成的组件）
app.config.suppress_callback_exceptions = True
```

## 4. LAYOUT 速查 —— html 组件
```python
html.Div(children=[...], id='my-div', className='box',
         style={'color': 'red', 'fontSize': 16})
#  ↑ 相当于 HTML 的 <div>，是最常用的容器

#  注意：CSS 属性名用驼峰法！
#    HTML:   font-size  →  Python: fontSize 或 'font-size'（字符串形式也行）
#    HTML:   text-align →  Python: textAlign
```

-  常用 html 组件：
>  ┌──────────────────┬────────────────────────────┐
>  │ html.H1/H2/H3    │ 标题                        │
>  │ html.P           │ 段落                        │
>  │ html.Div         │ 容器（最常用）               │
>  │ html.Br          │ 换行                        │
>  │ html.Hr          │ 分隔线                      │
>  │ html.Span        │ 行内容器                    │
>  │ html.Button      │ 按钮                        │
>  │ html.A           │ 超链接                      │
>  └──────────────────┴────────────────────────────┘


# ╔══════════════════════════════════════════════════════════╗
# ║  5. LAYOUT 速查 —— dcc 组件（交互组件）                   ║
# ╚══════════════════════════════════════════════════════════╝

# ── dcc.Dropdown（下拉选择框）──────────────────────────────
# dcc.Dropdown(
#     id='my-dropdown',           # ← 必须有！callback 靠这个找到它
#     options=[                   # 选项列表
#         {'label': '显示的文字', 'value': '传给callback的值'},
#         {'label': '选项2',     'value': 'option2'},
#     ],
#     value='option2',            # 默认选中的值（对应 value 字段）
#     placeholder='请选择...',     # 没选时的灰色提示文字
#     multi=False,                # True = 可以多选
#     disabled=False,             # True = 禁用（灰色不可点）
#     clearable=True,             # 是否显示清除按钮
#     style={'width': '50%'}
# )

# ── dcc.Graph（图表）──────────────────────────────────────
# dcc.Graph(
#     id='my-graph',
#     figure=px.line(df, x='Year', y='Sales')
#     # figure 就是 plotly 生成的图表对象
# )

# ── dcc.Slider（滑块）────────────────────────────────────
# dcc.Slider(
#     id='my-slider',
#     min=0, max=100, step=10,
#     value=50,                   # 默认值
#     marks={0: '0', 50: '50', 100: '100'}  # 刻度标记
# )

# ── dcc.RadioItems（单选按钮）────────────────────────────
# dcc.RadioItems(
#     id='my-radio',
#     options=[{'label': 'A', 'value': 'a'},
#              {'label': 'B', 'value': 'b'}],
#     value='a',                  # 默认选中
#     inline=True                 # True = 横排，False = 竖排
# )

# ── dcc.Checklist（复选框）───────────────────────────────
# dcc.Checklist(
#     id='my-check',
#     options=[{'label': 'X', 'value': 'x'},
#              {'label': 'Y', 'value': 'y'}],
#     value=['x']                 # 默认勾选的值（是列表！）
# )

# ── dcc.Input（文本输入框）───────────────────────────────
# dcc.Input(
#     id='my-input',
#     type='text',                # 'text' / 'number' / 'password'
#     placeholder='输入内容...',
#     debounce=True               # True = 按回车才触发，False = 实时触发
# )

# ── dcc.DatePickerSingle（日期选择）─────────────────────
# dcc.DatePickerSingle(
#     id='my-date',
#     date='2023-01-01'
# )


# ╔══════════════════════════════════════════════════════════╗
# ║  6. CALLBACK 核心语法                                     ║
# ╚══════════════════════════════════════════════════════════╝

# @app.callback(
#     Output('输出组件的id', '要更新的属性'),
#     Input('输入组件的id',  '监听的属性')
# )
# def 函数名(input的值):
#     # 这里写逻辑
#     return 返回值  # 返回值会被放进 Output 指定的属性里

# ── 常见的 component_property（属性）速查 ──────────────────
# ┌────────────────┬─────────────────┬───────────────────────┐
# │ 组件           │ 常用属性         │ 说明                   │
# ├────────────────┼─────────────────┼───────────────────────┤
# │ dcc.Dropdown   │ value           │ 当前选中的值            │
# │ dcc.Dropdown   │ disabled        │ 是否禁用（True/False）  │
# │ dcc.Dropdown   │ options         │ 选项列表               │
# │ dcc.Graph      │ figure          │ 图表内容               │
# │ html.Div       │ children        │ 内部内容               │
# │ html.Div       │ style           │ CSS 样式字典           │
# │ dcc.Input      │ value           │ 输入框的文字            │
# │ html.H1        │ children        │ 标题文字               │
# └────────────────┴─────────────────┴───────────────────────┘

# ── 多个 Input ────────────────────────────────────────────
# @app.callback(
#     Output('output', 'children'),
#     [Input('dd1', 'value'),
#      Input('dd2', 'value')]   # ← 多个 Input 放进列表
# )
# def my_func(val1, val2):      # ← 参数顺序和 Input 顺序一致
#     return f'{val1} + {val2}'

# ── 多个 Output ───────────────────────────────────────────
# @app.callback(
#     [Output('graph1', 'figure'),
#      Output('graph2', 'figure')],
#     Input('dropdown', 'value')
# )
# def my_func(val):
#     fig1 = px.line(...)
#     fig2 = px.bar(...)
#     return fig1, fig2          # ← 返回多个值，顺序对应 Output

# ── State（读值但不触发）─────────────────────────────────
# @app.callback(
#     Output('output', 'children'),
#     Input('button', 'n_clicks'),   # ← 点按钮才触发
#     State('input-box', 'value')    # ← 读取输入框的值，但不触发
# )
# def on_click(n_clicks, input_value):
#     return f'你输入了: {input_value}'


# ╔══════════════════════════════════════════════════════════╗
# ║  7. PLOTLY EXPRESS 常用图表速查                           ║
# ╚══════════════════════════════════════════════════════════╝

# px.line(df, x='col1', y='col2', title='标题',
#         color='分组列',           # 按某列分颜色画多条线
#         labels={'col1': '轴标签'} # 自定义轴标签
# )

# px.bar(df, x='col1', y='col2', title='标题',
#        color='分组列',            # 按某列分颜色（堆叠/并排）
#        barmode='group'            # 'group'=并排, 'stack'=堆叠
# )

# px.pie(df, values='数值列', names='分类列', title='标题',
#        hole=0.3                   # 0.3 = 甜甜圈图
# )

# px.scatter(df, x='col1', y='col2',
#            size='气泡大小列',      # 气泡图
#            color='分组列',
#            hover_data=['额外信息列']  # 鼠标悬浮时显示
# )

# px.histogram(df, x='col1', nbins=20)

# px.box(df, x='分类列', y='数值列')  # 箱线图


# ╔══════════════════════════════════════════════════════════╗
# ║  8. 布局技巧 —— 用 Flexbox 排列图表                       ║
# ╚══════════════════════════════════════════════════════════╝

# 两图并排：
# html.Div([
#     html.Div(dcc.Graph(...), style={'width': '50%'}),
#     html.Div(dcc.Graph(...), style={'width': '50%'}),
# ], style={'display': 'flex'})     # ← 关键！flex 让子元素横排

# 2x2 网格（两行两列）：
# html.Div([
#     html.Div([图1, 图2], style={'display': 'flex'}),  # 第一行
#     html.Div([图3, 图4], style={'display': 'flex'}),  # 第二行
# ])


# ════════════════════════════════════════════════════════════
#         第二部分：实战案例（进阶版汽车销售 Dashboard）
#         在原题基础上增加了：滑块、统计卡片、更好的布局
# ════════════════════════════════════════════════════════════

import dash
from dash import html, dcc
from dash.dependencies import Input, Output
import plotly.express as px
import pandas as pd

app = dash.Dash(__name__)
app.config.suppress_callback_exceptions = True

# ── 读取数据 ──────────────────────────────────────────────
data = pd.read_csv(
    'https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/'
    'IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/'
    'historical_automobile_sales.csv'
)

# ── 预处理 ────────────────────────────────────────────────
year_list = sorted(data['Year'].unique())  # 从数据里取真实年份，比 range 更准确


# ════════════════════════════════════════════════════════════
#  LAYOUT
# ════════════════════════════════════════════════════════════

app.layout = html.Div(
    style={'fontFamily': 'Arial, sans-serif', 'backgroundColor': '#f8f9fa', 'padding': '20px'},
    children=[

        # ── 标题 ──────────────────────────────────────────
        html.H1(
            'Automobile Sales Statistics Dashboard',
            style={'textAlign': 'center', 'color': '#503D36', 'fontSize': 28,
                   'borderBottom': '2px solid #503D36', 'paddingBottom': '10px'}
        ),

        # ── 控制区（两个下拉框横排）───────────────────────
        html.Div(
            style={'display': 'flex', 'gap': '40px', 'marginBottom': '20px',
                   'backgroundColor': 'white', 'padding': '15px', 'borderRadius': '8px',
                   'boxShadow': '0 2px 4px rgba(0,0,0,0.1)'},
            children=[

                # 左侧：报告类型选择
                html.Div(
                    style={'flex': 1},  # flex:1 表示平均分配宽度
                    children=[
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
                            value='Yearly Statistics',   # 默认选中年度统计
                            placeholder='Select a report type',
                            style={'fontSize': '16px'}
                        )
                    ]
                ),

                # 右侧：年份选择（只在 Yearly 模式下可用）
                html.Div(
                    style={'flex': 1},
                    children=[
                        html.Label('📆 选择年份（仅年度报告可用）',
                                   style={'fontWeight': 'bold', 'marginBottom': '8px',
                                          'display': 'block'}),
                        dcc.Dropdown(
                            id='select-year',
                            options=[{'label': str(y), 'value': y} for y in year_list],
                            value=year_list[-1],         # 默认选最新年份
                            placeholder='Select a year',
                            style={'fontSize': '16px'}
                            # disabled 属性会被 callback 动态控制
                        )
                    ]
                ),
            ]
        ),

        # ── 统计卡片区（callback 动态填充）──────────────
        # 这是一个空容器，callback 会往里塞内容
        html.Div(id='stats-cards',
                 style={'display': 'flex', 'gap': '15px', 'marginBottom': '20px'}),

        # ── 图表输出区（callback 动态填充）──────────────
        html.Div(id='output-container', className='chart-grid'),
    ]
)


# ════════════════════════════════════════════════════════════
#  CALLBACK 1：控制年份下拉框的启用/禁用状态
# ════════════════════════════════════════════════════════════

@app.callback(
    # Output：要改的是 select-year 这个组件的 disabled 属性
    Output(component_id='select-year', component_property='disabled'),
    # Input：监听 dropdown-statistics 的 value 属性
    # 只要用户改了报告类型，这个 callback 就会触发
    Input(component_id='dropdown-statistics', component_property='value')
)
def update_input_container(selected_statistics):
    """
    根据报告类型决定年份选择框是否可用。
    - 选了 'Yearly Statistics' → 返回 False → disabled=False → 可以用
    - 其他（Recession）       → 返回 True  → disabled=True  → 灰色不可点
    """
    if selected_statistics == 'Yearly Statistics':
        return False   # False = 不禁用 = 可以选年份
    else:
        return True    # True  = 禁用   = 不需要选年份


# ════════════════════════════════════════════════════════════
#  CALLBACK 2：根据选择动态生成图表和统计卡片
# ════════════════════════════════════════════════════════════

@app.callback(
    # 有两个 Output，所以放在列表里
    [Output(component_id='output-container', component_property='children'),
     Output(component_id='stats-cards',      component_property='children')],
    # 有两个 Input，也放在列表里
    # 注意：两个 Input 任意一个变化都会触发这个 callback
    [Input(component_id='dropdown-statistics', component_property='value'),
     Input(component_id='select-year',         component_property='value')]
)
def update_output_container(selected_statistics, input_year):
    """
    主 callback：负责生成所有图表和顶部统计卡片。

    参数：
        selected_statistics: 用户从第一个下拉框选的值
        input_year:          用户从第二个下拉框选的年份（可能是 None）

    返回：
        (图表区内容, 统计卡片区内容)
        ↑ 顺序必须和 Output 列表的顺序一致！
    """

    # ── 小工具：生成统计卡片 ────────────────────────────
    def make_card(title, value, color='#503D36'):
        """
        生成一个小统计卡片（纯 HTML 组件拼起来的）。
        封装成函数是为了避免重复代码。
        """
        return html.Div(
            style={
                'backgroundColor': 'white',
                'padding': '15px 20px',
                'borderRadius': '8px',
                'borderLeft': f'4px solid {color}',
                'boxShadow': '0 2px 4px rgba(0,0,0,0.1)',
                'flex': 1  # 让每张卡片平均分配宽度
            },
            children=[
                html.P(title, style={'color': '#666', 'margin': 0, 'fontSize': 13}),
                html.H3(value, style={'color': color, 'margin': '5px 0 0 0'})
            ]
        )

    # ── 小工具：包装图表（加白色背景和阴影）────────────
    def wrap_graph(graph_component):
        """给每个图表加一个白色卡片背景，让界面更好看。"""
        return html.Div(
            style={
                'flex': 1,                          # 平均分配宽度
                'backgroundColor': 'white',
                'borderRadius': '8px',
                'boxShadow': '0 2px 4px rgba(0,0,0,0.1)',
                'margin': '5px',
                'overflow': 'hidden'
            },
            children=[graph_component]
        )

    # ════════════════════════════════════════════════════
    #  分支一：衰退期统计
    # ════════════════════════════════════════════════════
    if selected_statistics == 'Recession Period Statistics':

        # 过滤出衰退期数据（Recession == 1 表示衰退期）
        recession_data = data[data['Recession'] == 1]

        # ── 统计卡片数据 ─────────────────────────────
        total_sales   = int(recession_data['Automobile_Sales'].sum())
        avg_sales     = round(recession_data['Automobile_Sales'].mean(), 1)
        recession_yrs = recession_data['Year'].nunique()  # nunique = 不重复的年份数

        cards = [
            make_card('衰退期总销售量',          f'{total_sales:,} 辆', '#e74c3c'),
            make_card('衰退期平均月销量',         f'{avg_sales:,} 辆',  '#e67e22'),
            make_card('涉及衰退年份数',           f'{recession_yrs} 年', '#9b59b6'),
        ]

        # ── 图表 1：折线图 —— 衰退年份的年均销量变化 ──
        # groupby('Year') 按年份分组，['Automobile_Sales'].mean() 求每年平均
        # reset_index() 把分组索引变回普通列，方便 plotly 使用
        yearly_rec = (recession_data
                      .groupby('Year')['Automobile_Sales']
                      .mean()
                      .reset_index())
        R_chart1 = dcc.Graph(
            figure=px.line(
                yearly_rec,
                x='Year',
                y='Automobile_Sales',
                title='衰退期各年平均汽车销量',
                markers=True,           # 在折线上显示圆点标记
                labels={'Automobile_Sales': '平均销量', 'Year': '年份'}
            )
        )

        # ── 图表 2：柱状图 —— 各车型平均销量 ────────
        average_sales = (recession_data
                         .groupby('Vehicle_Type')['Automobile_Sales']
                         .mean()
                         .reset_index())
        R_chart2 = dcc.Graph(
            figure=px.bar(
                average_sales,
                x='Vehicle_Type',
                y='Automobile_Sales',
                title='衰退期各车型平均销量',
                color='Vehicle_Type',   # 按车型着色，更直观
                labels={'Automobile_Sales': '平均销量', 'Vehicle_Type': '车型'}
            )
        )

        # ── 图表 3：饼图 —— 各车型广告支出占比 ──────
        # 用 sum() 而不是 mean()，因为我们想看总占比
        exp_rec = (recession_data
                   .groupby('Vehicle_Type')['Advertising_Expenditure']
                   .sum()
                   .reset_index())
        R_chart3 = dcc.Graph(
            figure=px.pie(
                exp_rec,
                values='Advertising_Expenditure',
                names='Vehicle_Type',
                title='衰退期各车型广告支出占比',
                hole=0.3                # 甜甜圈样式，比普通饼图更现代
            )
        )

        # ── 图表 4：柱状图 —— 失业率对各车型销量的影响
        # 按 [失业率, 车型] 两个维度分组，求平均销量
        unemp_data = (recession_data
                      .groupby(['unemployment_rate', 'Vehicle_Type'])['Automobile_Sales']
                      .mean()
                      .reset_index())
        R_chart4 = dcc.Graph(
            figure=px.bar(
                unemp_data,
                x='unemployment_rate',
                y='Automobile_Sales',
                color='Vehicle_Type',  # 不同车型用不同颜色，形成分组柱状图
                barmode='group',       # 'group'=并排柱, 'stack'=堆叠柱
                title='失业率对各车型销量的影响',
                labels={'unemployment_rate': '失业率(%)',
                        'Automobile_Sales':  '平均销量'}
            )
        )

        # ── 组合图表：2行2列布局 ──────────────────────
        charts = [
            # 第一行：图1 + 图2
            html.Div(
                style={'display': 'flex'},  # flex = 横排
                children=[wrap_graph(R_chart1), wrap_graph(R_chart2)]
            ),
            # 第二行：图3 + 图4
            html.Div(
                style={'display': 'flex'},
                children=[wrap_graph(R_chart3), wrap_graph(R_chart4)]
            )
        ]

        return charts, cards   # ← 返回两个值，对应两个 Output

    # ════════════════════════════════════════════════════
    #  分支二：年度统计
    # ════════════════════════════════════════════════════
    elif input_year and selected_statistics == 'Yearly Statistics':

        # 过滤出选定年份的数据
        yearly_data = data[data['Year'] == input_year]

        # ── 统计卡片 ─────────────────────────────────
        total_sales = int(yearly_data['Automobile_Sales'].sum())
        avg_monthly = round(yearly_data['Automobile_Sales'].mean(), 1)
        top_vehicle = (yearly_data
                       .groupby('Vehicle_Type')['Automobile_Sales']
                       .sum()
                       .idxmax())  # idxmax() 返回最大值对应的索引（车型名）

        cards = [
            make_card(f'{input_year} 年总销量', f'{total_sales:,} 辆', '#27ae60'),
            make_card(f'{input_year} 月均销量', f'{avg_monthly:,} 辆', '#2980b9'),
            make_card(f'{input_year} 销量冠军车型', top_vehicle,         '#f39c12'),
        ]

        # ── 图表 1：折线图 —— 全时段年均销量趋势 ────
        # 注意：用的是全部 data，不是 yearly_data
        # 因为这张图要展示"整个历史"，用来对比当前年份在哪个位置
        yas = data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        Y_chart1 = dcc.Graph(
            figure=px.line(
                yas,
                x='Year',
                y='Automobile_Sales',
                title=f'全时段年均汽车销量（{input_year} 年高亮）',
                markers=True,
                labels={'Automobile_Sales': '平均销量', 'Year': '年份'}
            )
        )

        # ── 图表 2：折线图 —— 选定年份的月度销量 ────
        # 注意：用的是 yearly_data，只看这一年每个月的情况
        # 用 sum() 因为我们想看这一年每个月的总量
        mas = (yearly_data
               .groupby('Month')['Automobile_Sales']
               .sum()
               .reset_index())
        Y_chart2 = dcc.Graph(
            figure=px.line(
                mas,
                x='Month',
                y='Automobile_Sales',
                title=f'{input_year} 年月度总销量',
                markers=True,
                labels={'Automobile_Sales': '总销量', 'Month': '月份'}
            )
        )

        # ── 图表 3：柱状图 —— 该年各车型平均销量 ────
        avr_vdata = (yearly_data
                     .groupby('Vehicle_Type')['Automobile_Sales']
                     .mean()
                     .reset_index())
        Y_chart3 = dcc.Graph(
            figure=px.bar(
                avr_vdata,
                x='Vehicle_Type',
                y='Automobile_Sales',
                color='Vehicle_Type',
                title=f'{input_year} 年各车型平均销量',
                labels={'Automobile_Sales': '平均销量', 'Vehicle_Type': '车型'}
            )
        )

        # ── 图表 4：饼图 —— 该年各车型广告支出 ──────
        exp_data = (yearly_data
                    .groupby('Vehicle_Type')['Advertising_Expenditure']
                    .sum()
                    .reset_index())
        Y_chart4 = dcc.Graph(
            figure=px.pie(
                exp_data,
                values='Advertising_Expenditure',
                names='Vehicle_Type',
                title=f'{input_year} 年各车型广告支出占比',
                hole=0.3
            )
        )

        charts = [
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(Y_chart1), wrap_graph(Y_chart2)]),
            html.Div(style={'display': 'flex'},
                     children=[wrap_graph(Y_chart3), wrap_graph(Y_chart4)])
        ]

        return charts, cards

    # ── 什么都没选时返回空 ────────────────────────────
    else:
        return [], []


# ════════════════════════════════════════════════════════════
#  启动 APP
# ════════════════════════════════════════════════════════════

if __name__ == '__main__':
    # debug=True：
    #   - 代码改动后自动重载，不用手动重启
    #   - 报错时在浏览器里显示详细错误信息
    #   - 生产环境上线时要改成 False！
    app.run(debug=True)


# ════════════════════════════════════════════════════════════
#  常见报错速查
# ════════════════════════════════════════════════════════════
#
#  ❌ KeyError: 'Automobile_Sales'
#     → 列名写错了，用 print(df.columns) 检查实际列名
#
#  ❌ Duplicate callback outputs
#     → 两个 @app.callback 输出了同一个组件的同一个属性，不允许
#
#  ❌ 图表不更新
#     → 检查 Input 里的 id 和 property 是否和 layout 里一致
#
#  ❌ nonexistent object with ID "xxx"
#     → callback 引用了 layout 里没有的 id
#     → 如果是动态生成的组件，加上 suppress_callback_exceptions=True
#
#  ❌ 页面空白
#     → callback 返回了 None，改成返回 [] 或空字符串
