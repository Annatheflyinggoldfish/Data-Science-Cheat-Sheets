# 📉 Matplotlib 基础绘图速查表


## 1. 核心概念
* 🏢 **Figure（大底盘/大画布）**：最底层的整张大白纸。它本身不能画图，但它决定了整张画面的总宽高和背景色。
* 🖼️ **Axes（独立的子图/画板）**：底盘上的一个个独立画板。**这是真正干活的图层**。每个 Axes 拥有一套独立的坐标轴、标题、横轴、纵轴。一张大画布（Figure）上可以放一个或多个画板（Axes）。
* ✏️ **Axis（坐标轴）**：画板上的刻度线（Ticks）和数字标签（Tick Labels），控制数据的范围和间距。

---

## 2. 骨架搭建

**永远优先使用 `plt.subplots()` 显式创建画布和画板, 多张子图时禁止混用 plt.plot() 和 ax.plot()**。

```python
import matplotlib.pyplot as plt
import numpy as np

# 1. 准备数据
x = np.linspace(0, 10, 100)
y = np.sin(x)

# 2. 创建画布(fig)与画板(ax) —— 显式指定画布宽高比 (英寸)
fig, ax = plt.subplots(figsize=(8, 4), dpi=100)

# 3. 在指定的画板(ax)上画图
ax.plot(x, y, label='Sine Wave', color='#1f77b4', linewidth=2, linestyle='--')

# 4. 精细修饰画板（坐标轴、标题、图例、网格）
ax.set_title('Standard Line Plot', fontsize=14, fontweight='bold', pad=15)
ax.set_xlabel('Time (s)', fontsize=12)
ax.set_ylabel('Amplitude', fontsize=12)
ax.set_xlim(0, 10)                           # 严格控制横坐标范围
ax.grid(True, linestyle=':', alpha=0.6)      # 开启网格线（虚线，半透明）
ax.legend(loc='upper right', frameon=True)   # 开启图例并显示外框

# 5. 紧凑布局并导出/显示
plt.tight_layout()                           # 自动防止标签、标题被边缘裁剪
plt.show()
```

## 3. 基础图表
# 折线图 (ax.plot) —— 适合看趋势
```python
# 核心参数：color(颜色), linestyle(线型), linewidth(线宽), marker(数据点样式)
ax.plot(x, y, color='crimson', linestyle='-', linewidth=2, marker='o', markersize=4)

# 常见线型(linestyle)：'-' 实线, '--' 虚线, ':' 点线, '-.' 点划线
# 常见标记(marker)：'o' 圆点, 's' 方块, '^' 上三角, '*' 五角星
```

# 条形图 / 柱状图 (ax.bar / ax.barh) —— 适合比大小
```python
categories = ['A', 'B', 'C', 'D']
values = [40, 70, 25, 85]

# 纵向柱状图
ax.bar(categories, values, color='skyblue', edgecolor='black', width=0.6)

# 横向条形图（展示长文本分类名称时的首选，极其优雅）
ax.barh(categories, values, color='salmon', edgecolor='black', height=0.6)
```

# 散点图 (ax.scatter) —— 适合看相关性分布
```python
x_data = np.random.randn(100)
y_data = np.random.randn(100)
sizes = np.random.rand(100) * 100  # 动态控制点的大小

# 核心参数：s(点的大小), c(点的颜色), alpha(透明度，高密度重叠时必设)
ax.scatter(x_data, y_data, s=sizes, c='purple', alpha=0.6, edgecolor='white')
```


# 直方图 (ax.hist) —— 适合看单变量数据分布
```python
data = np.random.normal(100, 15, 1000)

# 核心参数：bins(分箱/划分多少个柱子), density(是否转化为概率密度)
ax.hist(data, bins=30, color='seagreen', edgecolor='black', alpha=0.7)
```

# 饼图 (ax.pie) —— 适合看整体占比（如：各品类市场份额、预算开销结构）
```python
# df = pd.read_csv('market_share.csv')
# 假设数据有两列：'产品品类' (手机, 电脑, 配件...) 和 '市场份额' (35.5, 24.1...)

ax.pie(
    df['市场份额'], 
    labels=df['产品品类'],           # 每一块对应的文字标签
    autopct='%1.1f%%',              # 🌟 最核心：自动计算并显示百分比（保留1位小数）
    startangle=90,                  # 旋转角度：从正上方 12 点方向开始切蛋糕
    colors=['#ff7f0e', '#1f77b4', '#2ca02c', '#d62728'], # 自定义每一块的颜色
    wedgeprops={'edgecolor': 'white', 'linewidth': 1.5}  # 给蛋糕块之间加一条优雅的白线隔开
)
ax.set_title('2026年各产品线市场营收份额占比', fontsize=13, fontweight='bold')
```

## 3. 多子图组合布局
```python
# 创建一个 2 行 2 列，共计 4 个画板的大画布
fig, axes = plt.subplots(2, 2, figsize=(12, 10), sharex=False, sharey=False)

# ⚠️ 注意：此时 axes 是一个 2x2 的 NumPy 矩阵。我们需要通过矩阵坐标定位对应的画板：
axes[0, 0].plot(x, y, color='red')     # 左上角画板
axes[0, 0].set_title('Plot [0,0]')

axes[0, 1].bar(categories, values)     # 右上角画板
axes[0, 1].set_title('Plot [0,1]')

axes[1, 0].scatter(x_data, y_data)     # 左下角画板
axes[1, 0].set_title('Plot [1,0]')

axes[1, 1].hist(data, bins=20)          # 右下角画板
axes[1, 1].set_title('Plot [1,1]')

plt.tight_layout()
```


