# Python Requests & HTTP 速查手册
## HTTP 基础概念 · requests 库 · GET / POST · 响应处理

> **核心思路**：`requests` 库是 Python 里发 HTTP 请求最常用的工具。
> 你可以把它理解为"用 Python 代替浏览器"——浏览器能做的事（访问网页、
> 下载图片、提交表单、调用 API），用 `requests` 几行代码都能实现。

---

## 目录

1. [HTTP 基础概念](#1-http-基础概念)
2. [URL 结构拆解](#2-url-结构拆解)
3. [HTTP 方法速查](#3-http-方法速查)
4. [HTTP 状态码速查](#4-http-状态码速查)
5. [安装与导入](#5-安装与导入)
6. [GET 请求](#6-get-请求)
7. [GET 请求携带参数](#7-get-请求携带参数)
8. [POST 请求](#8-post-请求)
9. [响应对象完整速查](#9-响应对象完整速查)
10. [下载文件（图片 / 文本 / 二进制）](#10-下载文件图片--文本--二进制)
11. [请求头与认证](#11-请求头与认证)
12. [错误处理](#12-错误处理)
13. [GET vs POST 对比总结](#13-get-vs-post-对比总结)

---

## 1. HTTP 基础概念

> HTTP（HyperText Transfer Protocol）是浏览器和服务器之间"说话"的语言。
> 每次你访问一个网页，背后都发生了一次 HTTP 请求-响应的往返。

```
你（客户端）                          服务器
    │                                    │
    │  ── HTTP Request（请求）──────────> │  "我要 /index.html"
    │                                    │
    │  <── HTTP Response（响应）──────── │  "给你，这是 HTML 内容"
    │                                    │
```

**三个核心概念：**

| 概念 | 说明 | 例子 |
|------|------|------|
| **客户端（Client）** | 发起请求的一方 | 浏览器、Python 脚本 |
| **服务器（Server）** | 接收请求、返回资源的一方 | ibm.com 的服务器 |
| **资源（Resource）** | 服务器上存储的内容 | HTML 页面、图片、JSON 数据 |

---

## 2. URL 结构拆解

> URL（Uniform Resource Locator）= 资源的"地址"，就像快递地址一样精确定位一个资源。

```
https://www.ibm.com/products/data?category=cloud&page=2
│       │              │            │
│       │              │            └── Query String（查询参数）
│       │              │                ?key=value&key2=value2
│       │              └─────────────── Route（路径）
│       │                               服务器上资源的具体位置
│       └────────────────────────────── Base URL（基础地址）
│                                       服务器的域名
└────────────────────────────────────── Scheme（协议）
                                        http:// 或 https://
```

```python
# 在 Python 中解析 URL（了解用，不常用）
from urllib.parse import urlparse, urlencode, parse_qs

url = 'https://httpbin.org/get?name=Anna&city=London'
parsed = urlparse(url)

print(parsed.scheme)    # 'https'
print(parsed.netloc)    # 'httpbin.org'
print(parsed.path)      # '/get'
print(parsed.query)     # 'name=Anna&city=London'

# 解析查询参数为字典
print(parse_qs(parsed.query))  # {'name': ['Anna'], 'city': ['London']}
```

---

## 3. HTTP 方法速查

> HTTP 方法告诉服务器"我想对这个资源做什么操作"。

| 方法 | 语义 | 数据位置 | 常见场景 |
|------|------|---------|---------|
| **GET** | 获取资源 | URL 参数（可见）| 读网页、查询 API、搜索 |
| **POST** | 提交数据，创建资源 | 请求体（不可见）| 登录表单、提交数据、上传文件 |
| **PUT** | 替换整个资源 | 请求体 | 更新用户全部信息 |
| **PATCH** | 修改资源的一部分 | 请求体 | 只改用户的手机号 |
| **DELETE** | 删除资源 | URL 参数 | 删除一条记录 |
| **HEAD** | 只获取响应头，不要正文 | — | 检查文件是否存在、获取文件大小 |

---

## 4. HTTP 状态码速查

> 状态码是服务器对请求的"回复结果"，三位数字，第一位表示大类。

| 大类 | 范围 | 含义 |
|------|------|------|
| 1xx | 100-199 | 信息性：请求已收到，继续处理 |
| 2xx | 200-299 | ✅ 成功 |
| 3xx | 300-399 | 重定向：资源换地方了 |
| 4xx | 400-499 | ❌ 客户端错误（你的问题）|
| 5xx | 500-599 | ❌ 服务器错误（对方的问题）|

**最常见的状态码：**

| 状态码 | 含义 | 说明 |
|--------|------|------|
| **200** | OK | 请求成功，正常返回 |
| **201** | Created | 创建成功（POST 后常见）|
| **301** | Moved Permanently | 永久重定向 |
| **400** | Bad Request | 请求格式有问题 |
| **401** | Unauthorized | 需要认证（没有登录）|
| **403** | Forbidden | 有认证但没有权限 |
| **404** | Not Found | 资源不存在 |
| **429** | Too Many Requests | 请求频率超限 |
| **500** | Internal Server Error | 服务器内部出错 |
| **503** | Service Unavailable | 服务器暂时不可用 |

```python
# 在代码里判断状态码
r = requests.get(url)

if r.status_code == 200:
    print("成功")
elif r.status_code == 404:
    print("资源不存在")
elif r.status_code >= 500:
    print("服务器出错")

# 更简洁的写法：raise_for_status() 遇到 4xx/5xx 自动抛出异常
r.raise_for_status()   # 200-299 不做任何事；其他状态码抛出 HTTPError
```

---

## 5. 安装与导入

```bash
pip install requests
```

```python
import requests
import os
from PIL import Image           # 处理图片（pip install pillow）
```

---

## 6. GET 请求

> GET = 向服务器"取"一个资源，不改变服务器上的任何内容。
> 就像在图书馆借书——你只是读，不修改原书。

```python
import requests

# ── 最基本的 GET 请求 ────────────────────────
url = 'https://www.ibm.com/'
r = requests.get(url)

# r 是一个 Response 对象，包含服务器返回的所有信息
```

---

## 7. GET 请求携带参数

> 有时候需要告诉服务器"我想要哪种数据"，这时候在 URL 里加查询参数。
> 例如：搜索"python books"就是 `?q=python+books`。

```python
url = 'http://httpbin.org/get'

# ── 方法一：手动拼接 URL（不推荐，容易出错）────
# url_with_params = 'http://httpbin.org/get?name=Anna&city=London'

# ── 方法二：用 params 字典（推荐）──────────────
# requests 会自动把字典转换成 URL 查询字符串，并处理特殊字符编码
payload = {'name': 'Anna', 'city': 'London', 'page': 1}
r = requests.get(url, params=payload)

# 查看最终请求的完整 URL（验证参数是否正确附上）
print(r.url)
# 输出：http://httpbin.org/get?name=Anna&city=London&page=1

# 查看状态码
print(r.status_code)    # 200 = 成功

# 响应内容是 JSON 格式时，用 .json() 转成 Python 字典
data = r.json()
print(data)

# 取出 args 字段（httpbin 会把你发的参数原样返回）
print(data['args'])     # {'city': 'London', 'name': 'Anna', 'page': '1'}

# GET 请求没有请求体（数据在 URL 里，不在 body 里）
print(r.request.body)   # None
```

---

## 8. POST 请求

> POST = 向服务器"发送"数据，数据放在请求体（body）里，URL 上看不到。
> 就像把信封装进信封寄出去——内容对外不可见。
>
> **GET vs POST 核心区别**：
> - GET：数据在 URL 里，可以被书签、历史记录保存，适合查询
> - POST：数据在 body 里，更安全，适合提交密码、上传文件

```python
url = 'http://httpbin.org/post'
payload = {'username': 'Anna', 'password': '123456'}

# ── POST 表单数据（data 参数）────────────────
# 模拟 HTML 表单提交，Content-Type 自动设为 application/x-www-form-urlencoded
r = requests.post(url, data=payload)

print(r.status_code)            # 200
print(r.url)                    # URL 里没有参数（参数在 body 里）
print(r.request.body)           # username=Anna&password=123456（在这里）
print(r.json()['form'])         # {'password': '123456', 'username': 'Anna'}

# ── POST JSON 数据（json 参数）───────────────
# 调用 API 时最常用，Content-Type 自动设为 application/json
r = requests.post(url, json=payload)
print(r.json()['json'])         # {'password': '123456', 'username': 'Anna'}
# 注意：httpbin 返回时，json 参数传入的数据放在 'json' 字段
#       data 参数传入的数据放在 'form' 字段

# ── 对比 GET 和 POST 的请求体 ────────────────
r_get  = requests.get('http://httpbin.org/get', params=payload)
r_post = requests.post('http://httpbin.org/post', data=payload)

print("GET  请求 URL：",  r_get.url)           # URL 里能看到参数
print("POST 请求 URL：",  r_post.url)          # URL 里没有参数
print("GET  请求 body：", r_get.request.body)  # None
print("POST 请求 body：", r_post.request.body) # 参数在 body 里
```

---

## 9. 响应对象完整速查

> `requests.get()` / `requests.post()` 返回的都是 `Response` 对象，
> 下面是它所有常用属性和方法。

```python
r = requests.get('https://www.ibm.com/')

# ── 状态信息 ──────────────────────────────────
r.status_code           # 状态码，如 200、404
r.ok                    # True = 状态码 < 400；False = 有错误（简便判断）
r.reason                # 状态描述，如 'OK'、'Not Found'
r.url                   # 实际请求的完整 URL（可能经过重定向）

# ── 响应头（Headers）─────────────────────────
r.headers               # 全部响应头，是一个类字典对象
r.headers['Content-Type']    # 内容类型，如 'text/html; charset=utf-8'
                              #            'application/json'
                              #            'image/png'
r.headers['Date']            # 服务器响应时间
r.headers.get('Content-Length', '未知')  # 内容长度（字节数）

# ── 请求头（你发出去的）──────────────────────
r.request.headers       # 请求时带的 headers
r.request.body          # 请求体（GET 为 None，POST 有值）
r.request.method        # 'GET' 或 'POST'

# ── 响应内容 ──────────────────────────────────
r.text                  # 文本内容（自动用 encoding 解码），适合 HTML/JSON
r.text[0:200]           # 只看前 200 个字符（内容太长时用）
r.content               # 原始字节内容（bytes），适合图片/PDF 等二进制文件
r.json()                # 把 JSON 响应解析成 Python 字典（Content-Type 是 JSON 时用）

# ── 编码 ──────────────────────────────────────
r.encoding              # 当前解码方式，如 'utf-8'
r.encoding = 'utf-8'    # 可以手动覆盖（网页编码不对时用）
```

---

## 10. 下载文件（图片 / 文本 / 二进制）

### 10.1 下载图片

```python
import requests
import os
from PIL import Image

url = 'https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0101EN-SkillsNetwork/IDSNlogo.png'

r = requests.get(url)

# 验证内容类型（确认是图片）
print(r.headers['Content-Type'])    # 'image/png'

# 图片是二进制数据，用 r.content（不是 r.text）
# 构建保存路径
path = os.path.join(os.getcwd(), 'downloaded_image.png')

# 用 'wb'（write binary）模式写入
with open(path, 'wb') as f:
    f.write(r.content)

print(f"图片已保存到：{path}")

# 用 Pillow 打开查看（Jupyter 里会直接显示）
img = Image.open(path)
img.show()
```

### 10.2 下载文本文件

```python
url = 'https://example.com/data.txt'
r = requests.get(url)

path = os.path.join(os.getcwd(), 'data.txt')

# 文本文件用 'w' 模式，指定编码
with open(path, 'w', encoding='utf-8') as f:
    f.write(r.text)

# 也可以用二进制模式（更通用，不会有编码问题）
with open(path, 'wb') as f:
    f.write(r.content)
```

### 10.3 下载大文件（流式下载，不占内存）

```python
# 普通 get() 会把整个文件读进内存，大文件会撑爆内存
# stream=True：一块一块地下载，边下边写

url = 'https://example.com/large_file.zip'
r = requests.get(url, stream=True)

with open('large_file.zip', 'wb') as f:
    for chunk in r.iter_content(chunk_size=8192):  # 每次读 8KB
        if chunk:                                   # 过滤掉空块
            f.write(chunk)

print("下载完成")
```

---

## 11. 请求头与认证

```python
# ── 自定义请求头 ──────────────────────────────
# 有些 API 要求你发送特定的 headers（如模拟浏览器、携带 token）
headers = {
    'User-Agent': 'Mozilla/5.0',        # 模拟浏览器（防反爬）
    'Accept': 'application/json',        # 告诉服务器我想要 JSON 格式
    'Accept-Language': 'zh-CN,zh;q=0.9'
}
r = requests.get(url, headers=headers)

# ── API Token 认证（最常用）──────────────────
# 方法一：放在 headers 里（Bearer Token）
headers = {'Authorization': 'Bearer your_api_token_here'}
r = requests.get(url, headers=headers)

# 方法二：放在 URL 参数里（部分 API 支持）
r = requests.get(url, params={'api_key': 'your_api_key'})

# ── Basic Auth（用户名密码）───────────────────
r = requests.get(url, auth=('username', 'password'))

# ── 设置超时（重要！防止请求挂起不返回）──────
r = requests.get(url, timeout=10)      # 10秒超时
r = requests.get(url, timeout=(3, 10)) # (连接超时3秒, 读取超时10秒)
```

---

## 12. 错误处理

```python
import requests
from requests.exceptions import (
    RequestException,       # 所有 requests 异常的基类
    ConnectionError,        # 网络连接失败（断网、DNS 解析失败）
    Timeout,                # 请求超时
    HTTPError,              # HTTP 响应错误（4xx/5xx）
    TooManyRedirects        # 重定向次数过多
)

# ── 推荐写法：捕获所有情况 ────────────────────
def safe_get(url, params=None):
    try:
        r = requests.get(url, params=params, timeout=10)
        r.raise_for_status()    # 4xx/5xx 自动抛出 HTTPError
        return r

    except Timeout:
        print(f"请求超时：{url}")
    except ConnectionError:
        print(f"网络连接失败，请检查网络或 URL：{url}")
    except HTTPError as e:
        print(f"HTTP 错误：{e.response.status_code} - {url}")
    except TooManyRedirects:
        print(f"重定向次数过多：{url}")
    except RequestException as e:
        print(f"请求失败：{e}")
    return None

# 使用
r = safe_get('http://httpbin.org/get', params={'name': 'Anna'})
if r:
    print(r.json())

# ── 课程里的写法（简化版）────────────────────
try:
    response = requests.post(url, data=payload)
    if response.status_code == 200:
        print("成功：", response.json())
    else:
        print(f"HTTP 错误：{response.status_code} - {response.reason}")
except requests.exceptions.RequestException as e:
    print(f"请求出错：{e}")
```

---

## 13. GET vs POST 对比总结

| 对比项 | GET | POST |
|--------|-----|------|
| **数据位置** | URL 查询字符串里（可见）| 请求体 body 里（不可见）|
| **使用参数** | `params={}` | `data={}` 或 `json={}` |
| **适合场景** | 查询、读取、搜索 | 提交表单、发送敏感数据、上传 |
| **能否书签** | ✅ 可以（URL 包含完整信息）| ❌ 不行 |
| **数据长度** | 受 URL 长度限制（约 2000 字符）| 无限制 |
| **安全性** | 低（参数暴露在 URL）| 较高（数据在 body 里）|
| **请求体** | `None`（没有 body）| 有 body |
| **幂等性** | ✅ 幂等（多次请求结果相同）| ❌ 非幂等（多次提交会创建多条数据）|

```python
# 一眼看清区别的代码对比

# GET：参数拼在 URL 上
r_get = requests.get('http://httpbin.org/get', params={'name': 'Anna'})
print(r_get.url)            # http://httpbin.org/get?name=Anna  ← 看得到
print(r_get.request.body)   # None                             ← 没有 body

# POST：参数在 body 里
r_post = requests.post('http://httpbin.org/post', data={'name': 'Anna'})
print(r_post.url)            # http://httpbin.org/post          ← URL 干净
print(r_post.request.body)   # name=Anna                       ← 在 body 里

# POST JSON（调 API 最常用）
r_json = requests.post('http://httpbin.org/post', json={'name': 'Anna'})
print(r_json.request.body)   # {"name": "Anna"}
print(r_json.json()['json']) # {'name': 'Anna'}   ← JSON 数据在 'json' 字段
print(r_post.json()['form']) # {'name': 'Anna'}   ← 表单数据在 'form' 字段
```

---

## 附：完整流程速查

```python
import requests
import os

# 1. 发请求
r = requests.get(url)                     # 简单 GET
r = requests.get(url, params={...})       # 带查询参数的 GET
r = requests.post(url, data={...})        # POST 表单
r = requests.post(url, json={...})        # POST JSON（API 常用）
r = requests.get(url, headers={...}, timeout=10)  # 带请求头 + 超时

# 2. 检查结果
r.status_code                             # 200=成功，404=没找到，500=服务器错误
r.ok                                      # True = 成功（<400）
r.raise_for_status()                      # 自动在 4xx/5xx 时抛出异常

# 3. 读取响应
r.text                                    # 文本（HTML / 纯文本）
r.json()                                  # JSON → Python 字典（最常用）
r.content                                 # 二进制（图片 / 文件）
r.headers['Content-Type']                 # 看内容类型，决定用哪个

# 4. 保存文件
with open('file.png', 'wb') as f:         # 图片/二进制 → 'wb' 模式
    f.write(r.content)

with open('file.txt', 'w', encoding='utf-8') as f:  # 文本 → 'w' 模式
    f.write(r.text)
```
