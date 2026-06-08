# Streamlit 完整手册
## 从零搭建 → 交互控件 → 数据展示 → 生产部署

> **核心思路**：Streamlit 是比 Dash 更简单直接的数据应用框架。
> 不需要定义 layout，不需要写 callback——**脚本从上到下顺序执行**，
> 用户每次操作控件，整个脚本重新运行一遍，页面自动刷新。
>
> - **Dash** 适合：复杂的多页面仪表盘、需要精细控制布局、团队协作项目
> - **Streamlit** 适合：快速原型、数据探索报告、ML模型演示、个人项目

```
脚本从上到下顺序执行
   ↓ st.title / st.write / st.markdown         → 显示文字内容
   ↓ st.sidebar / st.columns / st.tabs         → 控制页面布局
   ↓ st.selectbox / st.slider / st.button      → 用户交互控件
   ↓ st.dataframe / st.metric / st.plotly_chart → 展示数据和图表
   ↓ @st.cache_data                             → 缓存耗时计算
   ↓ st.session_state                           → 跨次运行保存状态
```

---

## 目录

1. [安装与运行](#1-安装与运行)
2. [文字与内容展示](#2-文字与内容展示)
3. [页面布局](#3-页面布局)
4. [交互控件](#4-交互控件)
5. [数据与图表展示](#5-数据与图表展示)
6. [缓存与性能优化](#6-缓存与性能优化)
7. [Session State 状态管理](#7-session-state-状态管理)
8. [实战完整案例（含ML）](#8-实战完整案例含ml)
9. [常见报错与解决](#9-常见报错与解决)

---

## 1. 安装与运行

```bash
pip install streamlit plotly pandas scikit-learn
```

```python
# 新建一个 Python 文件，比如 app.py
import streamlit as st

st.title("Hello, Streamlit!")
st.write("第一个 Streamlit 应用")
```

```bash
# 在终端运行（不是 python app.py，是 streamlit run）
streamlit run app.py

# 访问地址：http://localhost:8501
# 代码修改后，右上角会出现"Rerun"按钮，或设置自动重跑
```

```python
# ── 全局页面配置（必须是脚本第一个 st 调用）──
st.set_page_config(
    page_title="我的数据看板",          # 浏览器标签页标题
    page_icon="📦",                    # 标签页图标（emoji 或图片路径）
    layout="wide",                     # 'centered'=居中窄版（默认）；'wide'=全宽
    initial_sidebar_state="expanded"   # 侧边栏初始状态：'expanded'/'collapsed'/'auto'
)
```

---

## 2. 文字与内容展示

> Streamlit 用 `st.write()` 当"万能输出"——它能自动判断传入的是文字、
> DataFrame、图表还是其他对象，并选择合适的方式渲染。

### 2.1 文字显示

```python
# ── 标题系列 ──────────────────────────────────
st.title("一级大标题")                  # 最大，页面顶部用
st.header("二级标题")                   # 章节标题
st.subheader("三级标题")                # 小节标题
st.caption("灰色小字说明文字")           # 图表注释、数据来源

# ── 正文 ──────────────────────────────────────
st.write("普通文字，也可以传 DataFrame、fig 等对象")
st.write("支持 **加粗** 和 _斜体_ 的 Markdown 语法")

st.markdown("""
## Markdown 标题
- 列表项 1
- 列表项 2

> 引用块

**加粗** | _斜体_ | `代码`
""")

# ── 代码块 ────────────────────────────────────
st.code("""
SELECT * FROM orders
WHERE status = 'delivered'
LIMIT 100;
""", language='sql')                    # language 可以是 'python'/'sql'/'json' 等

# ── 数学公式（LaTeX）─────────────────────────
st.latex(r"R^2 = 1 - \frac{SS_{res}}{SS_{tot}}")

# ── 分割线 ────────────────────────────────────
st.divider()                            # 比 st.write("---") 更清晰
```

### 2.2 状态与提示

```python
# 四种提示框，颜色和图标不同
st.success("✅ 模型训练完成，R² = 0.87")   # 绿色
st.info("ℹ️ 数据已加载，共 1000 行")       # 蓝色
st.warning("⚠️ 检测到 23 个缺失值")        # 黄色
st.error("❌ 文件读取失败，请检查路径")      # 红色

# 进度条（适合循环任务）
import time
bar = st.progress(0, text="正在处理...")
for i in range(100):
    time.sleep(0.01)
    bar.progress(i + 1, text=f"进度 {i+1}%")
bar.empty()                             # 完成后清除进度条

# 加载动画（适合单次耗时操作）
with st.spinner("模型训练中，请稍候..."):
    time.sleep(2)                       # 这里放耗时代码
st.success("完成！")

# 气球动画（庆祝效果）
st.balloons()                           # 撒气球🎈
st.snow()                               # 下雪❄️
```

---

## 3. 页面布局

> Streamlit 的布局比 Dash 简单很多：
> **侧边栏**放控件，**主区域**放内容；用 `columns` 横向分区，用 `tabs` 切换视图。

### 3.1 侧边栏 Sidebar

```python
# 方法一：with 语法（推荐，结构清晰）
with st.sidebar:
    st.header("🔧 筛选控制")
    selected_city  = st.selectbox("选择城市", ['全部', '上海', '北京'])
    selected_year  = st.slider("选择年份", 2020, 2024, 2022)
    show_raw       = st.checkbox("显示原始数据")

# 方法二：前缀语法（和上面等价）
selected_city = st.sidebar.selectbox("选择城市", ['全部', '上海', '北京'])
```

### 3.2 多列布局 Columns

```python
# 等宽三列
col1, col2, col3 = st.columns(3)
with col1:
    st.metric("平均延迟", "62.3 小时", delta="-3.1")
with col2:
    st.metric("总订单量", "1,000 笔", delta="+120")
with col3:
    st.metric("模型 R²", "0.8731", delta="+0.02")

# 自定义列宽比例（左窄右宽，常用于"控制面板 + 图表"）
left, right = st.columns([1, 3])
with left:
    st.write("控件放这里")
with right:
    st.write("图表放这里")

# 列之间加间距
col1, gap, col2 = st.columns([4, 0.2, 4])  # gap 列留空当间距用
```

### 3.3 标签页 Tabs

```python
tab1, tab2, tab3 = st.tabs(["📈 趋势分析", "🔍 数据详情", "🤖 模型评估"])

with tab1:
    st.subheader("销售趋势")
    st.plotly_chart(fig_trend, use_container_width=True)

with tab2:
    st.subheader("原始数据")
    st.dataframe(df)

with tab3:
    st.subheader("模型指标")
    st.write(f"R² = {r2:.4f}")
```

### 3.4 折叠区块 Expander

```python
# 默认折叠，点击展开——适合放次要信息（数据说明、调试信息等）
with st.expander("📖 查看数据字段说明", expanded=False):
    st.markdown("""
    | 字段 | 说明 |
    |------|------|
    | freight_value | 运费（雷亚尔） |
    | delivery_latency_hrs | 物流延迟（小时） |
    | review_score | 用户评分 1-5 |
    """)

with st.expander("🛠️ 查看原始数据", expanded=False):
    st.dataframe(df.head(50))
```

### 3.5 容器与空位占位

```python
# container：将一组组件打包，方便统一控制
with st.container():
    st.write("这些组件在同一个容器里")
    st.plotly_chart(fig)

# empty：占位符，之后可以动态替换内容
placeholder = st.empty()
placeholder.write("初始内容")
time.sleep(1)
placeholder.write("更新后的内容")   # 替换原来的内容
placeholder.empty()                 # 清空占位符
```

### 3.6 多页面应用

```
# 文件夹结构
my_app/
├── app.py              ← 主页
└── pages/
    ├── 1_趋势分析.py    ← 第二页（数字前缀控制顺序）
    ├── 2_数据详情.py    ← 第三页
    └── 3_模型评估.py    ← 第四页
```

```python
# 每个 pages/ 下的文件都是独立页面
# Streamlit 自动在左侧侧边栏生成导航菜单
# 页面间共享数据用 st.session_state（见第7节）
```

---

## 4. 交互控件

> 所有控件都**直接返回值**，不需要 callback，赋值给变量就能用。
> 用户操作控件后，脚本从头重新执行，控件返回新的值。

### 4.1 选择类

```python
# ── 下拉单选框 ────────────────────────────────
city = st.selectbox(
    "选择城市",                          # 标签文字
    options=['全部', '上海', '北京', '广州'],
    index=0,                            # 默认选中第几个（从0开始）
    help="选择要分析的城市",              # 鼠标悬停提示
    placeholder="请选择..."             # 未选时的提示（Streamlit ≥1.27）
)
# city 就是用户选中的值，直接用

# ── 多选框 ────────────────────────────────────
cities = st.multiselect(
    "选择城市（可多选）",
    options=['上海', '北京', '广州', '深圳'],
    default=['上海', '北京'],            # 默认选中的值（列表）
)
# cities 是列表，如 ['上海', '北京']

# ── 单选按钮 ──────────────────────────────────
chart_type = st.radio(
    "图表类型",
    options=['折线图', '柱状图', '散点图'],
    index=0,
    horizontal=True,                    # True=水平排列；False=竖向（默认）
)
```

### 4.2 数值输入类

```python
# ── 滑块 ──────────────────────────────────────
year = st.slider(
    "选择年份",
    min_value=2018, max_value=2024,
    value=2022,                         # 默认值（单值=单滑块）
    step=1
)

# 范围滑块（value 传元组）
price_range = st.slider(
    "价格区间",
    min_value=0, max_value=1000,
    value=(200, 800),                   # 默认区间，元组形式
    step=50,
    format="¥%d"                        # 显示格式
)
low, high = price_range                 # 解包拿到两端的值

# ── 数字输入框 ────────────────────────────────
alpha = st.number_input(
    "Ridge 正则化强度 (alpha)",
    min_value=0.001, max_value=1000.0,
    value=1.0,
    step=0.1,
    format="%.3f"                       # 显示精度
)
```

### 4.3 文字输入类

```python
# 单行文本
keyword = st.text_input(
    "搜索关键词",
    value="",                           # 默认内容
    max_chars=100,
    placeholder="输入商品名称..."
)

# 多行文本
note = st.text_area(
    "备注",
    value="",
    height=120,                         # 文本框高度（px）
    placeholder="在这里输入说明..."
)
```

### 4.4 勾选与按钮

```python
# 复选框（返回 True/False）
show_raw = st.checkbox("显示原始数据", value=False)
if show_raw:
    st.dataframe(df)

# 开关（比 checkbox 视觉更现代）
dark_mode = st.toggle("深色模式", value=False)

# 普通按钮（点击后返回 True，仅在当次运行中为 True）
if st.button("🔄 重新训练模型", type="primary"):
    # type='primary'=蓝色；'secondary'=灰色（默认）
    st.write("训练中...")
    # 这里放训练代码

# 下载按钮
csv_data = df.to_csv(index=False).encode('utf-8')
st.download_button(
    label="⬇️ 下载数据 CSV",
    data=csv_data,
    file_name="data.csv",
    mime="text/csv"
)
```

### 4.5 日期与时间

```python
import datetime

date = st.date_input(
    "选择日期",
    value=datetime.date(2024, 1, 1),
    min_value=datetime.date(2020, 1, 1),
    max_value=datetime.date.today(),
    format="YYYY-MM-DD"
)

# 日期范围（value 传元组）
date_range = st.date_input(
    "选择日期范围",
    value=(datetime.date(2024, 1, 1), datetime.date(2024, 12, 31))
)
start, end = date_range

time = st.time_input("选择时间", value=datetime.time(9, 0))
```

### 4.6 文件上传

```python
uploaded = st.file_uploader(
    "上传 CSV 文件",
    type=['csv', 'xlsx'],              # 限制文件类型
    accept_multiple_files=False        # True=允许上传多个
)

if uploaded is not None:
    if uploaded.name.endswith('.csv'):
        df = pd.read_csv(uploaded)
    else:
        df = pd.read_excel(uploaded)
    st.success(f"上传成功：{uploaded.name}，共 {len(df)} 行")
    st.dataframe(df.head())
```

### 4.7 表单（批量提交）

```python
# 表单：把多个控件打包，点"提交"才一次性触发，避免每改一个控件就刷新一次
with st.form("filter_form"):
    st.subheader("批量设置参数")
    city   = st.selectbox("城市", ['上海', '北京'])
    year   = st.slider("年份", 2020, 2024, 2022)
    metric = st.radio("指标", ['销售额', '订单量'])
    
    submitted = st.form_submit_button("✅ 应用筛选")  # 提交按钮

if submitted:
    # 只有点了提交按钮，才执行这里的代码
    st.write(f"已选：{city} / {year} / {metric}")
```

---

## 5. 数据与图表展示

### 5.1 数据表格

```python
# ── dataframe：交互式表格（可排序、可全屏）──
st.dataframe(
    df,
    use_container_width=True,          # 宽度撑满父容器
    height=300,                        # 表格高度（px）
    hide_index=True,                   # 隐藏行索引
    column_config={                    # 自定义列的显示方式
        "price": st.column_config.NumberColumn(
            "价格（元）",
            format="¥%.2f",
            min_value=0
        ),
        "rating": st.column_config.ProgressColumn(
            "评分",
            format="%d ⭐",
            min_value=0, max_value=5
        ),
        "url": st.column_config.LinkColumn("链接"),
    }
)

# ── table：静态表格（不可交互，适合小型摘要）──
st.table(df.describe().round(2))
```

### 5.2 指标卡 Metric

```python
# 大数字 + 趋势箭头，仪表盘顶部指标卡标配
col1, col2, col3, col4 = st.columns(4)

col1.metric(
    label="平均延迟",
    value="62.3 小时",
    delta="-3.1 小时",                 # 正数=绿色向上；负数=红色向下
    delta_color="inverse"              # inverse=负数变绿（延迟降低是好事）
                                       # 默认 normal：正绿负红
                                       # 'off'：不显示颜色
)
col2.metric("总订单量", "1,000 笔", "+120")
col3.metric("模型 R²", "0.8731", "+0.02")
col4.metric("RMSE", "8.42", "-0.31", delta_color="inverse")
```

### 5.3 图表

```python
import plotly.express as px

# ── Plotly（推荐，交互式）────────────────────
fig = px.line(df, x='date', y='sales', color='city', title='月度销售趋势')
st.plotly_chart(fig, use_container_width=True)  # use_container_width 让图表自适应宽度

# ── Matplotlib / Seaborn（静态图）────────────
import matplotlib.pyplot as plt
import seaborn as sns

fig, ax = plt.subplots(figsize=(10, 4))
sns.heatmap(df.corr(), annot=True, fmt='.2f', cmap='coolwarm', ax=ax)
st.pyplot(fig)                         # 传 fig 对象，不要用 plt.show()
plt.close(fig)                         # 关闭 fig，释放内存

# ── 原生简易图表（快速用，功能有限）──────────
st.line_chart(df.set_index('date')['sales'])      # 折线图
st.bar_chart(df.set_index('category')['revenue']) # 柱状图
st.area_chart(df.set_index('date')[['a','b']])    # 面积图
st.scatter_chart(df, x='price', y='rating')       # 散点图（≥1.26）
st.map(df[['lat','lon']])                          # 地图（需要 lat/lon 列）
```

### 5.4 图片、音频、视频

```python
from PIL import Image

st.image("path/to/image.png", caption="图片说明", use_column_width=True)
st.image(Image.open("image.png"))      # 也接受 PIL Image 对象

st.audio("audio.mp3")
st.video("video.mp4")
```

### 5.5 JSON 与代码展示

```python
st.json({
    "model": "Ridge",
    "alpha": 10.0,
    "r2_score": 0.8731
})

st.code("""
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model', Ridge(alpha=10))
])
""", language='python')
```

---

## 6. 缓存与性能优化

> **Streamlit 每次用户操作都会重新执行整个脚本**，
> 如果每次都重新读文件、训练模型，会非常慢。
> `@st.cache_data` 把函数结果缓存起来，参数不变时直接返回缓存，不重新计算。

### 6.1 `@st.cache_data`（缓存数据）

```python
# 适合：读取文件、查询数据库、数据清洗、特征工程等
# 返回的是数据对象（DataFrame、列表、字典等）

@st.cache_data
def load_data(filepath):
    df = pd.read_csv(filepath)
    # 清洗...
    return df

# 第一次调用：正常执行，结果存入缓存
# 之后只要 filepath 参数不变：直接返回缓存，不重新读文件
df = load_data("data.csv")

# 带 ttl（缓存过期时间，适合数据库查询）
@st.cache_data(ttl=3600)            # 缓存 1 小时后自动失效（单位：秒）
def fetch_from_db():
    ...

# 手动清除缓存
load_data.clear()                   # 清除这个函数的缓存
st.cache_data.clear()               # 清除所有缓存
```

### 6.2 `@st.cache_resource`（缓存资源）

```python
# 适合：加载 ML 模型、数据库连接等"重型对象"
# 和 cache_data 的区别：
# cache_data    → 每个用户/会话拿到独立的副本（安全）
# cache_resource → 所有用户共享同一个对象（节省内存，适合只读模型）

@st.cache_resource
def load_model():
    model = Ridge(alpha=10.0)
    model.fit(X_train_scaled, y_train)
    return model

model = load_model()    # 全局只训练一次
```

---

## 7. Session State 状态管理

> **核心问题**：Streamlit 每次重跑脚本，普通变量都会重置。
> `st.session_state` 是一个跨次运行持久存在的字典，
> 用来保存需要"记住"的状态，比如按钮点击计数、多步骤表单的中间结果。

```python
# ── 基本用法 ──────────────────────────────────
# 初始化（第一次运行时设置默认值）
if 'count' not in st.session_state:
    st.session_state['count'] = 0
# 也可以用属性语法
if 'count' not in st.session_state:
    st.session_state.count = 0

# 读取
st.write(f"点击次数：{st.session_state.count}")

# 更新
if st.button("点击 +1"):
    st.session_state.count += 1
    st.rerun()                          # 手动触发重跑，立即刷新页面

# ── 实际场景：记录模型训练结果 ──────────────
if 'model_trained' not in st.session_state:
    st.session_state.model_trained = False
    st.session_state.r2 = None
    st.session_state.model = None

if st.button("训练模型"):
    with st.spinner("训练中..."):
        model = Ridge(alpha=10.0)
        model.fit(X_train_scaled, y_train)
        r2 = model.score(X_test_scaled, y_test)
    
    st.session_state.model_trained = True
    st.session_state.r2 = r2
    st.session_state.model = model
    st.success("训练完成！")

# 训练完成后才显示评估结果
if st.session_state.model_trained:
    st.metric("测试集 R²", f"{st.session_state.r2:.4f}")

# ── 多页面间共享数据 ──────────────────────────
# session_state 在整个应用（所有 pages）中共享
# 在 app.py 里存：
st.session_state['shared_df'] = df
# 在 pages/1_趋势分析.py 里读：
df = st.session_state.get('shared_df', None)
if df is None:
    st.warning("请先在主页加载数据")
    st.stop()                           # 停止继续执行当前脚本
```

---

## 8. 实战完整案例（含ML）

> 与 Dash 案例用同一份数据，方便对比两个框架的写法差异。
> **最大区别**：Streamlit 没有 callback，用 `if/else` + 变量直接控制逻辑。

```python
import numpy as np
import pandas as pd
import plotly.express as px
import streamlit as st
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Ridge
from sklearn.metrics import r2_score, mean_squared_error
import warnings
warnings.filterwarnings('ignore')

# ══════════════════════════════════════════════
# 全局配置（必须在最前面）
# ══════════════════════════════════════════════
st.set_page_config(
    page_title="Olist 物流延迟分析看板",
    page_icon="📦",
    layout="wide"
)

# ══════════════════════════════════════════════
# STAGE 1: 数据生成与清洗（用缓存，只执行一次）
# ══════════════════════════════════════════════
@st.cache_data
def load_and_clean_data():
    np.random.seed(42)
    N = 1000
    df = pd.DataFrame({
        'state':                np.random.choice(['SP', 'RJ', 'MG', 'BA'], N),
        'freight_value':        np.random.uniform(10, 150, N),
        'product_weight_g':     np.random.uniform(200, 5000, N),
        'review_score':         np.random.choice([1,2,3,4,5], N,
                                    p=[0.1,0.05,0.1,0.25,0.5]),
        'delivery_latency_hrs': np.random.uniform(5, 120, N)
    })
    # IQR 截断异常值
    for col in ['freight_value', 'delivery_latency_hrs']:
        Q1, Q3 = df[col].quantile(0.25), df[col].quantile(0.75)
        IQR = Q3 - Q1
        df[col] = df[col].clip(Q1 - 1.5*IQR, Q3 + 1.5*IQR)
    return df

@st.cache_resource                      # 模型全局只训练一次
def train_model(df):
    df_ml = pd.get_dummies(df, columns=['state'], drop_first=True)
    X = df_ml.drop('delivery_latency_hrs', axis=1)
    y = df_ml['delivery_latency_hrs']
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled  = scaler.transform(X_test)
    
    model = Ridge(alpha=10.0)
    model.fit(X_train_scaled, y_train)
    
    r2   = r2_score(y_test, model.predict(X_test_scaled))
    rmse = np.sqrt(mean_squared_error(y_test, model.predict(X_test_scaled)))
    
    # 全量预测写回 df
    df_ml_full = pd.get_dummies(df, columns=['state'], drop_first=True)
    X_full = df_ml_full.drop('delivery_latency_hrs', axis=1)
    df = df.copy()
    df['predicted_latency'] = model.predict(scaler.transform(X_full))
    
    return model, scaler, r2, rmse, df

df_raw            = load_and_clean_data()
model, scaler, r2, rmse, df = train_model(df_raw)

# ══════════════════════════════════════════════
# STAGE 2: 侧边栏控件
# ══════════════════════════════════════════════
with st.sidebar:
    st.header("🔧 筛选控制")
    st.divider()
    
    selected_state = st.selectbox(
        "选择州 (State)",
        options=['ALL'] + sorted(df['state'].unique().tolist()),
        format_func=lambda x: '🌍 全部州' if x == 'ALL' else f'📍 {x}'
        # format_func：控制显示文字，但实际值还是原始值
    )
    
    st.divider()
    
    color_theme = st.radio(
        "图表主题色",
        options=['cool', 'warm'],
        format_func=lambda x: '🔵 蓝绿（冷静）' if x == 'cool' else '🟠 橙红（活跃）',
        horizontal=False
    )
    
    show_raw = st.checkbox("显示原始数据", value=False)
    
    st.divider()
    st.caption("💡 修改上方选项，右侧图表实时联动更新。")

# ══════════════════════════════════════════════
# STAGE 3: 主区域 — 标题 + 指标卡
# ══════════════════════════════════════════════
st.title("📦 Olist 电商物流延迟智能分析看板")
st.divider()

# 指标卡行
m1, m2, m3, m4 = st.columns(4)
m1.metric("大盘平均延迟",  f"{df['delivery_latency_hrs'].mean():.1f} 小时")
m2.metric("模型 R²",       f"{r2:.4f}",   delta=f"+{r2:.4f}")
m3.metric("RMSE",          f"{rmse:.2f}", delta_color="inverse")
m4.metric("有效订单总量",  f"{len(df):,} 笔")

st.divider()

# ══════════════════════════════════════════════
# STAGE 4: 根据侧边栏筛选过滤数据（无需 callback）
# ══════════════════════════════════════════════
filtered = df if selected_state == 'ALL' else df[df['state'] == selected_state]

# 数据为空时提前报警
if filtered.empty:
    st.warning("⚠️ 所选条件下没有数据，请重新筛选。")
    st.stop()                           # 停止执行后续代码

st.caption(f"当前筛选：**{selected_state}** | 共 **{len(filtered):,}** 条数据")

# ══════════════════════════════════════════════
# STAGE 5: 标签页 — 图表、数据、模型
# ══════════════════════════════════════════════
tab1, tab2, tab3 = st.tabs(["📊 分布对比", "🔍 散点分析", "📋 原始数据"])

# ── Tab1：实际 vs 预测分布直方图 ──────────────
with tab1:
    color_seq = ['#4E79A7', '#E15759'] if color_theme == 'cool' else ['#F28E2B', '#E15759']
    
    df_melt = filtered.melt(
        value_vars=['delivery_latency_hrs', 'predicted_latency'],
        var_name='类型', value_name='延迟（小时）'
    )
    df_melt['类型'] = df_melt['类型'].map({
        'delivery_latency_hrs': '📊 实际延迟',
        'predicted_latency':    '🤖 模型预测'
    })
    
    fig_dist = px.histogram(
        df_melt,
        x='延迟（小时）', color='类型',
        barmode='overlay', opacity=0.65,
        marginal='box', nbins=30,
        color_discrete_sequence=color_seq,
        title=f"实际 vs 预测延迟分布｜{selected_state}"
    )
    fig_dist.update_layout(template='plotly_white',
                           legend=dict(orientation='h', y=-0.25))
    st.plotly_chart(fig_dist, use_container_width=True)
    
    with st.expander("📖 指标说明"):
        st.markdown("""
        - **实际延迟**：真实物流交付耗时（小时）
        - **模型预测**：Ridge 回归基于运费、重量、评分、州份预测的耗时
        - 两条分布越接近，模型拟合效果越好
        """)

# ── Tab2：散点气泡图 ──────────────────────────
with tab2:
    scatter_cscale = (px.colors.sequential.Blugrn
                      if color_theme == 'cool'
                      else px.colors.sequential.Oranges)
    
    fig_scatter = px.scatter(
        filtered,
        x='freight_value', y='delivery_latency_hrs',
        size='product_weight_g', color='review_score',
        color_continuous_scale=scatter_cscale,
        labels={
            'freight_value':        '运费（雷亚尔）',
            'delivery_latency_hrs': '交付延迟（小时）',
            'review_score':         '评分',
            'product_weight_g':     '重量（g）'
        },
        title=f"运费 × 延迟 × 评分交叉分析｜{selected_state}"
    )
    fig_scatter.update_layout(template='plotly_white')
    st.plotly_chart(fig_scatter, use_container_width=True)

# ── Tab3：原始数据表 ──────────────────────────
with tab3:
    st.subheader(f"原始数据（{len(filtered):,} 行）")
    st.dataframe(
        filtered.round(2),
        use_container_width=True,
        hide_index=True,
        column_config={
            "review_score": st.column_config.ProgressColumn(
                "评分", format="%d ⭐", min_value=0, max_value=5
            ),
            "freight_value": st.column_config.NumberColumn(
                "运费", format="R$%.2f"
            ),
        }
    )
    
    csv = filtered.to_csv(index=False).encode('utf-8')
    st.download_button(
        "⬇️ 下载当前筛选数据",
        data=csv,
        file_name=f"olist_{selected_state}.csv",
        mime="text/csv"
    )
```

---

## 9. 常见报错与解决

| 报错 / 症状 | 原因 | 解决方法 |
|------------|------|---------|
| `StreamlitAPIException: set_page_config() can only be called once` | `set_page_config` 不在脚本最前面，或被调用了两次 | 确保它是第一个 `st.*` 调用 |
| 控件每次操作都重置 | 没有用 `session_state` 保存状态 | 用 `st.session_state` 持久化需要记住的变量 |
| 模型每次都重新训练，很慢 | 训练代码没有缓存 | 把训练函数包在 `@st.cache_resource` 里 |
| `st.pyplot()` 图表重复显示 | `plt.show()` 和 `st.pyplot()` 一起用 | 只用 `st.pyplot(fig)`，删掉 `plt.show()` |
| 上传文件后 `df` 为空 | 没有判断 `uploaded is not None` | 加 `if uploaded is not None:` 保护 |
| 多页面数据无法共享 | 不同 page 的变量互相独立 | 用 `st.session_state` 在页面间传数据 |
| `st.map()` 不显示 | DataFrame 没有 `lat` 和 `lon` 列 | 列名必须是 `lat`/`lon` 或 `latitude`/`longitude` |
| 缓存数据被修改后不更新 | `@st.cache_data` 缓存了旧数据 | 调用 `函数名.clear()` 手动清除，或加 `ttl` 过期时间 |
| `st.stop()` 后面的代码还在运行 | `st.stop()` 只停止当前脚本渲染，不是真的 exit | 确认 `st.stop()` 调用位置正确，它后面的代码确实不会执行 |
| Plotly 图表宽度不满 | 没有传 `use_container_width=True` | `st.plotly_chart(fig, use_container_width=True)` |

### 调试技巧

```python
# 1. 直接 st.write() 打印任何变量（DataFrame、字典、模型都可以）
st.write("当前筛选值：", selected_state)
st.write("filtered shape：", filtered.shape)

# 2. 查看 session_state 的全部内容
st.write(st.session_state)

# 3. 用 st.exception() 捕获并显示报错（不崩溃）
try:
    risky_operation()
except Exception as e:
    st.exception(e)

# 4. 标注代码执行到哪一步了（调试用）
st.toast("✅ 数据加载完成")   # 右下角短暂弹出提示

# 5. 强制重跑脚本
st.rerun()
```

Done

