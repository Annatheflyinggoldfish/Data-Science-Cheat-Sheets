## Folium地图速查表
#### 避坑
> 经纬度顺序不能反：这是初学者最常犯的错。Matplotlib 或常规坐标系是 (X, Y)，即先横轴后纵轴。但在地理坐标系和 Folium 里，顺序雷打不动是 (纬度 Latitude, 经度 Longitude)。如果你的地图打开之后发现一片漆黑，那就是把经纬度顺序传反了。

> 底图（Tiles）要选对：默认的 "OpenStreetMap" 颜色太花哨、字太多，如果你上面再叠加热力图，会显得杂乱无章。商业分析做热力图，首选 tiles="CartoDB positron"（浅灰色低饱和底图）。

### 动态打点与交互弹窗 (Markers)
```python
import folium
import pandas as pd

# 1. 准备你的现实地理数据（假设你用 Pandas 拿到了 Olist 仓库的经纬度）
# df = pd.read_csv('olist_warehouses.csv')
# 字段必须包含：'lat' (纬度), 'lng' (经度), 'city' (城市), 'latency' (延迟耗时)

# 2. 初始化地图底图：需要传一个初始的中心点坐标（这里以圣保罗市附近为例）
# zoom_start 代表默认的放大倍数
m = folium.Map(location=[-23.55, -46.63], zoom_start=6, tiles="OpenStreetMap")

# 3. 循环把现实里的仓库数据打到地图上
for idx, row in df.iterrows():
    # 🌟 极客技巧：设计一个优雅的 HTML 弹窗，支持换行和粗体
    popup_text = f"""
    <b>城市:</b> {row['city']}<br>
    <b>平均交付延迟:</b> {row['latency']} 小时
    """
    
    # 决定图标颜色：如果延迟超过 48 小时，贴红标警示；否则贴蓝标
    marker_color = 'red' if row['latency'] > 48 else 'blue'
    
    folium.Marker(
        location=[row['lat'], row['lng']],
        popup=folium.Popup(popup_text, max_width=300), # 点击后弹出的文字
        tooltip=f"点击查看 {row['city']} 详情",          # 鼠标悬浮时的提示
        icon=folium.Icon(color=marker_color, icon='info-sign')
    ).add_to(m)
```

### 业务高频订单密集度热力图 (HeatMap)
```python
import folium
from folium.plugins import HeatMap
import pandas as pd

# df = pd.read_csv('olist_customer_locations.csv')

# 1. 初始化底图
m_heatmap = folium.Map(location=[-15.78, -47.92], zoom_start=4, tiles="CartoDB positron") 
# 💡 极客选型：CartoDB positron 是一种非常素雅的淡灰色底图，最能衬托出热力图的颜色！

# 2. 提取经纬度矩阵：热力图插件要求传入一个 [[纬度, 经度], [纬度, 经度]] 的数据列表
# 用 .dropna() 剔除掉没有对齐经纬度的脏数据
heat_data = df[['lat', 'lng']].dropna().values.tolist()

# 3. 注入热力图层
# radius 控制热力红点的辐射半径，blur 控制平滑模糊度
HeatMap(heat_data, radius=10, blur=15, min_opacity=0.4).add_to(m_heatmap)

# 4. 保存成品
m_heatmap.save('olist_order_density_heatmap.html')
```

# 4. 导出为网页（在浏览器双击打开就是震撼的动态地图）
m.save('olist_warehouse_map.html')
