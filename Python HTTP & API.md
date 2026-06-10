# Python HTTP & API 完整手册
## requests 语法 · API 调用 · 真实案例

> **先搞清楚三个概念的关系，很多人混淆：**
>
> ```
> HTTP        → 网络通信协议，规定了"请求和响应"的格式
>               就像"说话的规则"，所有网络请求都走这套规则
>                  ↓
> requests    → Python 库，让你用几行代码发 HTTP 请求
>               就像"电话机"，帮你拨出去、接回来
>                  ↓
> API         → 服务器专门给程序开的接口，返回干净的 JSON 数据
>               就像"客服热线"，你按规定说，它按规定回
> ```
>
> **爬虫 vs API 调用的本质区别：**
>
> | | 爬虫 | API 调用 |
> |--|------|---------|
> | 目标 | 给人看的网页（HTML）| 给程序用的接口（JSON）|
> | 数据格式 | 混在 HTML 里，需要解析 | 直接返回结构化 JSON |
> | 服务器态度 | 没打算让你这么用，会反爬 | 专门开放的，欢迎调用 |
> | 稳定性 | 网页改版就挂 | 有版本管理，相对稳定 |
> | 认证 | 通常不需要 | 通常需要 API Key / Token |
> | 法律风险 | 灰色地带 | 有明确使用条款 |
>
> **同样是 `requests.get(url)`，一个拿 HTML，一个拿 JSON，仅此而已。**

---

## 目录

1. [安装与导入](#1-安装与导入)
2. [HTTP 基础概念](#2-http-基础概念)
3. [requests 核心语法](#3-requests-核心语法)
4. [响应对象完整速查](#4-响应对象完整速查)
5. [怎么看 API 文档](#5-怎么看-api-文档)
6. [API 认证方式](#6-api-认证方式)
7. [处理 JSON 响应](#7-处理-json-响应)
8. [分页处理](#8-分页处理)
9. [限流与错误处理](#9-限流与错误处理)
10. [实战：ONS 英国统计局 API](#10-实战ons-英国统计局-api)
11. [实战：Open-Meteo 天气 API（免费无需注册）](#11-实战open-meteo-天气-api免费无需注册)
12. [实战：GitHub API](#12-实战github-api)
13. [把 API 数据转成 DataFrame](#13-把-api-数据转成-dataframe)

---

## 1. 安装与导入

```bash
pip install requests pandas
```

```python
import requests
import pandas as pd
import json
import time
import os
```

---

## 2. HTTP 基础概念

### 2.1 URL 结构

```
https://api.ons.gov.uk/v1/datasets/cpih01/editions/time-series/versions/21/observations?time=*&aggregate=cpih1dim1A0&geography=K02000001
│       │                                                                                  │
│       └── 路径（Path）：服务器上具体资源的位置                                            │
│           通常能从路径猜出数据含义：/datasets/cpih01 = CPIH数据集                        │
│                                                                                          └── 查询参数（Query String）
└── 协议 + 域名：https://api.ons.gov.uk                                                       ?key=value&key2=value2
```

### 2.2 HTTP 方法

| 方法 | 语义 | 数据位置 | 数据分析常用场景 |
|------|------|---------|----------------|
| **GET** | 取数据 | URL 参数 | 查询 API、拉数据集 |
| **POST** | 发数据 | 请求体 body | 提交查询条件、上传数据 |
| **PUT** | 替换资源 | 请求体 | 更新整条记录 |
| **PATCH** | 修改部分 | 请求体 | 更新某个字段 |
| **DELETE** | 删除资源 | URL 参数 | 删除记录 |

> 📌 **数据分析师 90% 的情况只用 GET 和 POST。**
> GET 拉数据，POST 提交复杂查询条件（条件太多放不进 URL 时用）。

### 2.3 HTTP 状态码

```python
# 看到状态码的第一位就知道大概情况
# 2xx → 成功    4xx → 你的问题    5xx → 对方的问题

r = requests.get(url)
print(r.status_code)

# 最常见的：
# 200 OK              → 完全成功
# 201 Created         → POST 后创建成功
# 400 Bad Request     → 参数写错了（检查你的 params）
# 401 Unauthorized    → 没有认证（API Key 没带或过期）
# 403 Forbidden       → 有认证但没权限（免费账户访问付费功能）
# 404 Not Found       → URL 写错了，或资源不存在
# 429 Too Many Requests → 超出调用频率限制（等一会儿再试）
# 500 Internal Server Error → 服务器自己出错，不是你的问题

# 一行搞定判断：4xx/5xx 自动抛出异常
r.raise_for_status()
```

---

## 3. requests 核心语法

### 3.1 GET 请求

```python
import requests

# ── 最基本的写法 ──────────────────────────────
r = requests.get('https://api.example.com/data')

# ── 带查询参数（推荐用 params 字典，不要手动拼 URL）──
# 手动拼：'https://api.example.com/data?city=London&year=2024'  ← 容易出错
# 正确写法：
params = {
    'city': 'London',
    'year': 2024,
    'format': 'json'
}
r = requests.get('https://api.example.com/data', params=params)
print(r.url)  # 自动生成：https://api.example.com/data?city=London&year=2024&format=json
              # requests 会自动处理特殊字符编码（如空格变 %20）

# ── 带请求头 ──────────────────────────────────
headers = {
    'Authorization': 'Bearer your_token_here',
    'Accept': 'application/json',          # 告诉服务器我想要 JSON 格式
    'User-Agent': 'MyApp/1.0'
}
r = requests.get(url, headers=headers)

# ── 设置超时（必须养成习惯！）────────────────
# 不设超时的话，请求可能永远挂着不返回
r = requests.get(url, timeout=10)          # 10 秒后放弃
r = requests.get(url, timeout=(3, 30))     # (连接超时3秒, 读取超时30秒)
```

### 3.2 POST 请求

```python
# ── 发送表单数据（data 参数）────────────────
# Content-Type 自动设为 application/x-www-form-urlencoded
r = requests.post(url, data={'username': 'anna', 'password': '123'})

# ── 发送 JSON 数据（json 参数）───────────────
# Content-Type 自动设为 application/json
# 调用 API 时最常用，尤其是查询条件复杂的情况
query = {
    'dataset': 'cpih01',
    'filters': {'geography': 'K02000001', 'year': '2023'},
    'limit': 100
}
r = requests.post(url, json=query)

# ── GET vs POST 核心区别 ─────────────────────
r_get  = requests.get(url, params={'name': 'Anna'})
r_post = requests.post(url, data={'name': 'Anna'})

print(r_get.url)             # URL 里能看到参数：.../get?name=Anna
print(r_post.url)            # URL 干净：没有参数
print(r_get.request.body)    # None（参数在 URL 里，不在 body）
print(r_post.request.body)   # name=Anna（参数在 body 里）
```

### 3.3 Session（复用连接）

```python
# 普通 requests.get() 每次都是独立请求
# Session 复用 TCP 连接，自动保存 cookies，像真实浏览器会话
# 需要登录的 API、或者要发很多请求时用

session = requests.Session()
session.headers.update({
    'Authorization': 'Bearer your_token',
    'Accept': 'application/json'
})
# 之后所有请求都自动带上这些 headers，不用每次重复写
r1 = session.get('https://api.example.com/endpoint1')
r2 = session.get('https://api.example.com/endpoint2')
session.close()              # 用完记得关闭

# 或者用 with 语法自动关闭
with requests.Session() as s:
    s.headers.update({'Authorization': 'Bearer token'})
    r = s.get(url)
```

---

## 4. 响应对象完整速查

```python
r = requests.get(url)

# ── 状态 ──────────────────────────────────────
r.status_code          # 200 / 404 / 500 ...
r.ok                   # True = 状态码 < 400
r.reason               # 'OK' / 'Not Found' ...
r.url                  # 实际请求的 URL（可能经过重定向）

# ── 响应头（服务器返回的元信息）──────────────
r.headers                          # 全部响应头（类字典）
r.headers['Content-Type']          # 内容类型：'application/json' / 'text/html'
r.headers.get('X-RateLimit-Remaining')  # 剩余请求次数（部分 API 有）
r.headers.get('X-RateLimit-Reset')      # 限流重置时间

# ── 响应内容（三选一，根据 Content-Type 决定用哪个）──
r.json()               # JSON → Python 字典/列表（最常用，API 返回 JSON 时用）
r.text                 # 文本字符串（HTML / CSV / XML 时用）
r.content              # 原始字节（图片 / 文件 / 二进制时用）

# ── 编码 ──────────────────────────────────────
r.encoding             # 当前解码编码，如 'utf-8'
r.encoding = 'utf-8'   # 可以手动覆盖（网页乱码时用）

# ── 请求信息（你发出去的）────────────────────
r.request.url          # 发出的完整 URL
r.request.headers      # 发出的请求头
r.request.body         # 发出的请求体（GET 为 None）
r.request.method       # 'GET' / 'POST'

# ── 实用判断 ──────────────────────────────────
# 判断返回的是不是 JSON（避免用 r.json() 解析 HTML 报错）
if 'application/json' in r.headers.get('Content-Type', ''):
    data = r.json()
else:
    print("不是 JSON，内容：", r.text[:200])
```

---

## 5. 怎么看 API 文档

> 拿到一个新 API，文档是第一个要看的东西。
> 下面以 ONS API 为例，解释文档里每个部分是什么意思。

```
# 文档里通常包含这些信息：

─────────────────────────────────────────────
GET /v1/datasets/{dataset_id}/editions/{edition}/versions/{version}/observations
─────────────────────────────────────────────

① 方法：GET（你用 requests.get()）

② 路径：/v1/datasets/{dataset_id}/...
   花括号 {} 是路径参数，需要你替换成真实值
   如：/v1/datasets/cpih01/editions/time-series/versions/21/observations

③ 查询参数（Query Parameters）：
   ┌──────────────┬──────────┬────────────┬────────────────────────────┐
   │ 参数名        │ 类型     │ 必填/选填  │ 说明                        │
   ├──────────────┼──────────┼────────────┼────────────────────────────┤
   │ time         │ string   │ required   │ 时间维度，* 表示全部         │
   │ aggregate    │ string   │ required   │ 聚合维度                    │
   │ geography    │ string   │ required   │ 地区代码                    │
   │ limit        │ integer  │ optional   │ 每页返回条数，默认 10        │
   │ offset       │ integer  │ optional   │ 跳过前 N 条，用于翻页        │
   └──────────────┴──────────┴────────────┴────────────────────────────┘

④ 响应示例：
   {
     "observations": [
       {"time": "2023", "value": "5.1"},
       ...
     ],
     "total_observations": 240,
     "links": {"next": {"href": "/...?offset=10"}}
   }

⑤ 认证：
   Header: Authorization: Bearer <token>
   或
   Query: ?api_key=your_key
```

```python
# 把文档转成代码的过程：

# 文档说的：GET /v1/datasets/{dataset_id}/observations?time=*&geography=K02000001
# Python 写法：
base_url = 'https://api.ons.gov.uk'
dataset_id = 'cpih01'
edition = 'time-series'
version = '21'

url = f'{base_url}/v1/datasets/{dataset_id}/editions/{edition}/versions/{version}/observations'

params = {
    'time': '*',                  # * = 全部时间
    'aggregate': 'cpih1dim1A0',   # 总体 CPIH 指标
    'geography': 'K02000001'      # 英国全国
}

r = requests.get(url, params=params, timeout=30)
```

---

## 6. API 认证方式

> 大多数 API 需要认证，用来识别是谁在调用、控制调用频率。

### 6.1 API Key（最常见）

```python
# ── 方式一：放在查询参数里 ──────────────────
params = {
    'api_key': 'your_key_here',
    'city': 'London'
}
r = requests.get(url, params=params)

# ── 方式二：放在请求头里（更安全，推荐）──────
headers = {'X-Api-Key': 'your_key_here'}
r = requests.get(url, headers=headers)

# ── 保护 API Key（不要硬编码在代码里！）──────
# 正确做法：存在环境变量里
import os
api_key = os.environ.get('MY_API_KEY')   # 从环境变量读取
# 在终端设置：export MY_API_KEY='your_key_here'

# 或者存在 .env 文件里（pip install python-dotenv）
from dotenv import load_dotenv
load_dotenv()                            # 读取 .env 文件
api_key = os.getenv('MY_API_KEY')
# .env 文件内容：MY_API_KEY=your_key_here
# 注意把 .env 加进 .gitignore，绝不上传到 GitHub！
```

### 6.2 Bearer Token（OAuth 2.0，Reddit / GitHub 等用）

```python
# Bearer Token 通常有有效期，需要定期刷新
# Reddit、Spotify、GitHub 等 OAuth 认证返回的就是这种

token = 'eyJhbGciOiJSUzI1NiJ9...'     # 通常是很长的字符串

headers = {
    'Authorization': f'Bearer {token}',  # 注意格式：Bearer + 空格 + token
    'Accept': 'application/json'
}
r = requests.get(url, headers=headers)
```

### 6.3 Basic Auth（用户名密码）

```python
# 最老的认证方式，部分内部 API 还在用
r = requests.get(url, auth=('username', 'password'))

# 等价于手动设置 header（requests 帮你处理了 base64 编码）
import base64
credentials = base64.b64encode(b'username:password').decode()
headers = {'Authorization': f'Basic {credentials}'}
```

### 6.4 无需认证的免费 API

```python
# 有些 API 完全不需要认证，直接调：

# Open-Meteo（天气）
r = requests.get('https://api.open-meteo.com/v1/forecast',
                 params={'latitude': 51.5, 'longitude': -0.12,
                         'daily': 'temperature_2m_max'})

# RestCountries（国家信息）
r = requests.get('https://restcountries.com/v3.1/name/germany')

# ExchangeRate-API（汇率，免费额度）
r = requests.get('https://open.er-api.com/v6/latest/GBP')
```

---

## 7. 处理 JSON 响应

> API 返回的 JSON 经常是嵌套结构，需要一层一层取出来。

```python
r = requests.get(url)
data = r.json()              # 整个响应转成 Python 字典/列表

# ── 嵌套结构取值 ──────────────────────────────
# 假设 data 是这样的：
# {
#   "status": "success",
#   "data": {
#     "observations": [
#       {"time": "2023", "value": "5.1", "geography": {"id": "K02000001"}},
#       {"time": "2022", "value": "9.1", "geography": {"id": "K02000001"}}
#     ],
#     "total": 240
#   }
# }

status = data['status']                           # 'success'
observations = data['data']['observations']       # 列表
first = observations[0]                           # 第一条
value = first['value']                            # '5.1'
geo_id = first['geography']['id']                 # 'K02000001'（多层嵌套）

# ── 安全取值（键不存在时返回默认值，不报 KeyError）──
value = data.get('missing_key', 'default')

# 多层嵌套的安全取值
total = data.get('data', {}).get('total', 0)

# ── 列表推导式展平数据 ────────────────────────
# 把 observations 列表变成平铺的记录列表
records = [
    {
        'time':      obs['time'],
        'value':     float(obs['value']),    # 注意 JSON 里的数字有时是字符串
        'geography': obs['geography']['id']
    }
    for obs in observations
    if obs.get('value') not in ('', None)   # 过滤掉空值
]

# ── 打印 JSON 结构（调试用）──────────────────
import json
print(json.dumps(data, indent=2, ensure_ascii=False))
# ensure_ascii=False = 中文不转义，直接显示

# 只看前两层结构
def show_structure(obj, depth=0, max_depth=2):
    indent = '  ' * depth
    if depth >= max_depth:
        print(f"{indent}...")
        return
    if isinstance(obj, dict):
        for k, v in list(obj.items())[:5]:   # 每层最多显示5个键
            print(f"{indent}{k}: {type(v).__name__}")
            show_structure(v, depth+1, max_depth)
    elif isinstance(obj, list):
        print(f"{indent}[列表，共 {len(obj)} 条，第一条：]")
        if obj:
            show_structure(obj[0], depth+1, max_depth)

show_structure(data)
```

---

## 8. 分页处理

> 大多数 API 不会一次返回所有数据，而是分页返回。
> 翻页有三种主要方式，文档里会说用哪种。

### 8.1 页码分页（page / per_page）

```python
# 文档说：?page=1&per_page=100
# 最直观，知道总页数

def fetch_all_pages(base_url, params, max_pages=50):
    all_records = []
    params = params.copy()

    for page in range(1, max_pages + 1):
        params['page'] = page
        r = requests.get(base_url, params=params, timeout=30)
        r.raise_for_status()
        data = r.json()

        # 取出这页的数据（字段名因 API 不同而异）
        records = data.get('results', data.get('items', data.get('data', [])))

        if not records:             # 空列表 = 没有更多数据了
            print(f"第 {page} 页为空，停止翻页")
            break

        all_records.extend(records)
        print(f"第 {page} 页：{len(records)} 条，累计 {len(all_records)} 条")

        # 判断是否已经到最后一页
        total_pages = data.get('total_pages', data.get('pages'))
        if total_pages and page >= total_pages:
            print("已到最后一页")
            break

        time.sleep(0.5)             # 礼貌等待

    return all_records
```

### 8.2 偏移量分页（offset / limit）

```python
# 文档说：?limit=100&offset=0
# offset = 跳过前 N 条；limit = 每次取 N 条

def fetch_with_offset(base_url, params, limit=100):
    all_records = []
    params = params.copy()
    params['limit'] = limit
    offset = 0

    while True:
        params['offset'] = offset
        r = requests.get(base_url, params=params, timeout=30)
        r.raise_for_status()
        data = r.json()

        records = data.get('observations', data.get('results', []))
        if not records:
            break

        all_records.extend(records)
        print(f"offset={offset}，本次 {len(records)} 条，累计 {len(all_records)} 条")

        # 如果这次返回的条数少于 limit，说明是最后一页
        if len(records) < limit:
            break

        offset += limit
        time.sleep(0.5)

    return all_records
```

### 8.3 游标分页（cursor / next_url）

```python
# 文档说：响应里有 "next": "https://api.example.com/data?cursor=abc123"
# 用上一页返回的 cursor 请求下一页，不能跳页

def fetch_with_cursor(first_url, headers=None):
    all_records = []
    url = first_url

    while url:
        r = requests.get(url, headers=headers, timeout=30)
        r.raise_for_status()
        data = r.json()

        records = data.get('data', [])
        all_records.extend(records)
        print(f"累计 {len(all_records)} 条")

        # 取下一页 URL（不同 API 字段名不同）
        next_info = data.get('links', {}).get('next', {})
        url = next_info.get('href') if isinstance(next_info, dict) else next_info
        # url 为 None 或空字符串时循环结束

        time.sleep(0.5)

    return all_records
```

---

## 9. 限流与错误处理

> 限流（Rate Limit）= 服务器限制你每分钟/每小时最多能调多少次。
> 超了会返回 **429 Too Many Requests**，需要等一会儿再试。

```python
import time
import requests
from requests.exceptions import RequestException, Timeout, ConnectionError, HTTPError

def call_api(url, params=None, headers=None, max_retries=3):
    """
    带重试和限流处理的 API 调用函数。
    遇到 429 自动等待重试，遇到其他错误指数退避重试。
    """
    for attempt in range(max_retries):
        try:
            r = requests.get(url, params=params, headers=headers, timeout=30)

            # ── 处理限流（429）────────────────────
            if r.status_code == 429:
                # 部分 API 会在响应头里告诉你要等多久
                retry_after = int(r.headers.get('Retry-After', 60))
                print(f"触发限流，等待 {retry_after} 秒...")
                time.sleep(retry_after)
                continue                         # 重新发请求（不算入 attempt）

            r.raise_for_status()                 # 其他 4xx/5xx 抛出异常
            return r                             # 成功，直接返回

        except Timeout:
            print(f"第 {attempt+1} 次超时")
        except ConnectionError:
            print(f"第 {attempt+1} 次网络连接失败")
        except HTTPError as e:
            # 4xx 错误通常是参数问题，重试没意义
            if e.response.status_code < 500:
                print(f"客户端错误 {e.response.status_code}，不重试")
                print(f"响应内容：{e.response.text[:300]}")
                return None
            print(f"第 {attempt+1} 次服务器错误 {e.response.status_code}")
        except RequestException as e:
            print(f"第 {attempt+1} 次请求失败：{e}")

        # 指数退避：1s → 2s → 4s
        if attempt < max_retries - 1:
            wait = 2 ** attempt
            print(f"等待 {wait} 秒后重试...")
            time.sleep(wait)

    print(f"已重试 {max_retries} 次，放弃")
    return None


# ── 查看你还剩多少请求次数 ────────────────────
def check_rate_limit(r):
    """从响应头里读取限流信息（不是所有 API 都有）"""
    remaining = r.headers.get('X-RateLimit-Remaining')
    reset_time = r.headers.get('X-RateLimit-Reset')
    limit = r.headers.get('X-RateLimit-Limit')

    if remaining:
        print(f"剩余请求次数：{remaining}/{limit}")
    if reset_time:
        import datetime
        reset = datetime.datetime.fromtimestamp(int(reset_time))
        print(f"限流重置时间：{reset.strftime('%H:%M:%S')}")
```

---

## 10. 实战：ONS 英国统计局 API

> ONS（Office for National Statistics）提供免费 API，无需注册。
> 对你的 UK 就业数据项目直接有用。

```python
import requests
import pandas as pd

BASE_URL = 'https://api.ons.gov.uk/v1'

# ── Step 1：查看有哪些数据集 ─────────────────
r = requests.get(f'{BASE_URL}/datasets', timeout=30)
datasets = r.json()
# 查看前几个数据集的 id 和描述
for ds in datasets.get('items', [])[:10]:
    print(f"{ds['id']:30} {ds.get('title', '')}")

# ── Step 2：以 CPIH（消费者价格指数）为例 ─────
# 数据集 ID 从文档或 Step 1 的结果里找

dataset_id = 'cpih01'
edition    = 'time-series'
version    = '21'

obs_url = (f'{BASE_URL}/datasets/{dataset_id}'
           f'/editions/{edition}/versions/{version}/observations')

params = {
    'time':      '*',              # * = 全部时间点
    'aggregate': 'cpih1dim1A0',   # 总体 CPIH（所有项目）
    'geography': 'K02000001'      # K02000001 = 英国全国
}

r = requests.get(obs_url, params=params, timeout=30)
r.raise_for_status()
data = r.json()

print(f"共 {data['total_observations']} 条数据")

# ── Step 3：转成 DataFrame ────────────────────
records = []
for obs in data['observations']:
    records.append({
        'time':  obs['dimensions']['Time']['id'],
        'value': obs.get('observation', None)
    })

df = pd.DataFrame(records)
df['value'] = pd.to_numeric(df['value'], errors='coerce')
df['time']  = pd.to_datetime(df['time'], format='%b-%y', errors='coerce')
df = df.dropna().sort_values('time').reset_index(drop=True)

print(df.tail())
df.to_csv('ons_cpih.csv', index=False)
```

---

## 11. 实战：Open-Meteo 天气 API（免费无需注册）

> 完全免费、不需要注册、不需要 API Key，适合练手。

```python
import requests
import pandas as pd

# ── 获取伦敦未来 7 天天气预报 ─────────────────
url = 'https://api.open-meteo.com/v1/forecast'

params = {
    'latitude':  51.5074,          # 伦敦纬度
    'longitude': -0.1278,          # 伦敦经度
    'daily': [                     # 想要哪些每日指标（列表）
        'temperature_2m_max',      # 最高气温
        'temperature_2m_min',      # 最低气温
        'precipitation_sum',       # 降水量
        'windspeed_10m_max'        # 最大风速
    ],
    'timezone': 'Europe/London',
    'forecast_days': 7
}

r = requests.get(url, params=params, timeout=15)
r.raise_for_status()
data = r.json()

# ── 转成 DataFrame ──────────────────────────
daily = data['daily']
df = pd.DataFrame({
    'date':        pd.to_datetime(daily['time']),
    'temp_max':    daily['temperature_2m_max'],
    'temp_min':    daily['temperature_2m_min'],
    'rainfall_mm': daily['precipitation_sum'],
    'wind_max':    daily['windspeed_10m_max']
})

print(df)

# ── 获取历史数据（分析用更常见）──────────────
params_hist = {
    'latitude':   51.5074,
    'longitude':  -0.1278,
    'start_date': '2023-01-01',
    'end_date':   '2023-12-31',
    'daily':      ['temperature_2m_max', 'precipitation_sum'],
    'timezone':   'Europe/London'
}

r = requests.get('https://archive-api.open-meteo.com/v1/archive',
                 params=params_hist, timeout=30)
hist = r.json()

df_hist = pd.DataFrame({
    'date':        pd.to_datetime(hist['daily']['time']),
    'temp_max':    hist['daily']['temperature_2m_max'],
    'rainfall_mm': hist['daily']['precipitation_sum']
})

print(f"2023年伦敦平均最高气温：{df_hist['temp_max'].mean():.1f}°C")
print(f"2023年伦敦总降水量：{df_hist['rainfall_mm'].sum():.0f}mm")
```

---

## 12. 实战：GitHub API

> GitHub API 免费额度：未认证 60次/小时，认证后 5000次/小时。
> 不需要 OAuth，用 Personal Access Token 即可。

```python
import requests
import pandas as pd

# ── 申请 Token：GitHub → Settings → Developer settings
# → Personal access tokens → Tokens (classic) → Generate new token
# 权限：只需要勾 public_repo（读取公开仓库）

TOKEN = os.environ.get('GITHUB_TOKEN', '')    # 从环境变量读
headers = {
    'Authorization': f'Bearer {TOKEN}',
    'Accept': 'application/vnd.github+json',
    'X-GitHub-Api-Version': '2022-11-28'
}

# ── 搜索仓库 ──────────────────────────────────
r = requests.get(
    'https://api.github.com/search/repositories',
    params={
        'q': 'data analysis python stars:>1000',  # 搜索条件
        'sort': 'stars',
        'order': 'desc',
        'per_page': 30
    },
    headers=headers,
    timeout=15
)
r.raise_for_status()
data = r.json()

repos = [{
    'name':        repo['full_name'],
    'stars':       repo['stargazers_count'],
    'forks':       repo['forks_count'],
    'language':    repo['language'],
    'description': repo['description'],
    'url':         repo['html_url'],
    'updated_at':  repo['updated_at']
} for repo in data['items']]

df_repos = pd.DataFrame(repos)
print(df_repos[['name', 'stars', 'language']].head(10))

# ── 查看限流状态 ──────────────────────────────
check_rate_limit(r)   # 用前面定义的函数
```

---

## 13. 把 API 数据转成 DataFrame

> API 拿回来的 JSON 经常是嵌套的，这里总结几种常见结构的处理方式。

```python
import pandas as pd

# ── 结构1：直接是列表（最简单）───────────────
# [{"name": "Alice", "age": 25}, {"name": "Bob", "age": 30}]
df = pd.DataFrame(data)

# ── 结构2：数据在某个键下面 ──────────────────
# {"total": 100, "results": [...]}
df = pd.DataFrame(data['results'])

# ── 结构3：嵌套字典（json_normalize 展平）────
# [{"name": "Alice", "address": {"city": "London", "postcode": "SW1"}}]
from pandas import json_normalize
df = json_normalize(data,
                    sep='_')            # 嵌套键用 _ 连接：address_city, address_postcode

# 更复杂的嵌套（有列表嵌套）
# [{"post": "title", "comments": [{"text": "good"}, {"text": "nice"}]}]
df = json_normalize(
    data,
    record_path='comments',            # 要展开的列表字段
    meta=['post'],                     # 保留的父级字段
    meta_prefix='post_'
)

# ── 结构4：时间序列（日期为键）───────────────
# {"2023-01": 5.1, "2023-02": 5.3, "2023-03": 4.9}
df = pd.DataFrame(list(data.items()), columns=['date', 'value'])
df['date'] = pd.to_datetime(df['date'])

# ── 常见清洗 ──────────────────────────────────
# 数字被存成字符串（JSON 里 "value": "5.1"）
df['value'] = pd.to_numeric(df['value'], errors='coerce')
# errors='coerce'：无法转换的变成 NaN，不报错

# 时间戳转日期
df['created_at'] = pd.to_datetime(df['created_utc'], unit='s')   # Unix 时间戳
df['date'] = pd.to_datetime(df['date'], format='%b-%y')           # 如 "Jan-23"

# 保存
df.to_csv('api_data.csv', index=False, encoding='utf-8-sig')
df.to_excel('api_data.xlsx', index=False)
```

---

## 附：流程速查

```
拿到一个新 API
   ↓
1. 读文档
   ├── Base URL 是什么
   ├── 需要什么认证（API Key / Token / 无）
   ├── 参数有哪些（必填 / 选填）
   └── 响应结构长什么样（找 response example）
   ↓
2. 写代码
   ├── headers = {'Authorization': 'Bearer token'}     ← 认证
   ├── params  = {'key': 'value', 'limit': 100}        ← 参数
   └── r = requests.get(url, params=params,
                        headers=headers, timeout=30)   ← 发请求
   ↓
3. 看响应
   ├── print(r.status_code)                            ← 200 才继续
   ├── print(json.dumps(r.json(), indent=2))           ← 看清结构
   └── check_rate_limit(r)                             ← 看剩余次数
   ↓
4. 取数据
   ├── data = r.json()['results']                      ← 按结构取
   └── df = pd.DataFrame(data)                         ← 转 DataFrame
   ↓
5. 翻页（如果有）
   └── 循环 + offset/page/cursor，直到返回空列表
   ↓
6. 清洗存储
   ├── pd.to_numeric() / pd.to_datetime()
   └── df.to_csv('output.csv', index=False)
```

