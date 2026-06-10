# Web Scraping 完整手册
## BeautifulSoup · Selenium · Reddit API (PRAW)

> **先读这里：三种方法怎么选？**
>
> ```
> 目标网站
>   ↓
>   ├── 静态页面（HTML 直接包含数据，右键"查看源码"能看到内容）
>   │     → ✅ requests + BeautifulSoup（最简单，首选）
>   │
>   ├── 动态页面（JavaScript 渲染，源码里没有数据，需要等待加载）
>   │     → ✅ Selenium（模拟真实浏览器）
>   │
>   └── 有官方 API 的平台（Reddit、Twitter/X、GitHub 等）
>         → ✅ 官方 API / 专用库（最稳定，不会被封）
> ```
>
> **判断方法**：在浏览器里按 `Ctrl+U`（查看源码），
> 搜索你想要的内容（如帖子标题）。能找到 → 静态；找不到 → 动态。

---

## 目录

1. [环境安装](#1-环境安装)
2. [HTML 基础：看懂网页结构](#2-html-基础看懂网页结构)
3. [BeautifulSoup：静态页面爬取](#3-beautifulsoup静态页面爬取)
4. [BeautifulSoup：实战——抓取表格数据](#4-beautifulsoup实战抓取表格数据)
5. [BeautifulSoup：实战——抓取链接与图片](#5-beautifulsoup实战抓取链接与图片)
6. [Selenium：动态页面爬取](#6-selenium动态页面爬取)
7. [Reddit API (PRAW)：合法抓取 Reddit](#7-reddit-api-praw合法抓取-reddit)
8. [数据清洗与存储](#8-数据清洗与存储)
9. [反爬应对与爬虫礼仪](#9-反爬应对与爬虫礼仪)
10. [完整实战案例](#10-完整实战案例)

---

## 1. 环境安装

```bash
# 核心库
pip install requests beautifulsoup4 lxml pandas

# 动态页面（Selenium）
pip install selenium webdriver-manager

# Reddit API
pip install praw

# 可选（处理图片、存储）
pip install pillow openpyxl
```

---

## 2. HTML 基础：看懂网页结构

> 爬虫的本质是"在 HTML 里找你要的数据"。
> 在浏览器里按 `F12` 打开开发者工具，点"Elements"标签，
> 鼠标悬停在页面元素上就能看到对应的 HTML 代码。

```html
<!-- HTML 结构示例，爬虫里最常见的标签 -->

<a href="https://example.com">这是一个链接</a>
<!-- <a> = 链接标签，href 属性 = 链接地址，标签内文字 = 显示文字 -->

<img src="photo.jpg" alt="图片描述">
<!-- <img> = 图片标签，src 属性 = 图片地址 -->

<p class="description">这是一段文字</p>
<!-- <p> = 段落，class 属性 = CSS 类名（用来定位元素）-->

<div id="main-content">
    <h1>大标题</h1>
    <p>内容</p>
</div>
<!-- <div> = 容器，id 属性 = 唯一标识符 -->

<table>
    <tr>                         <!-- tr = table row（行）-->
        <th>列名1</th>           <!-- th = table header（表头）-->
        <th>列名2</th>
    </tr>
    <tr>
        <td>数据1</td>           <!-- td = table data（数据格）-->
        <td>数据2</td>
    </tr>
</table>

<span class="price">¥99.00</span>
<!-- <span> = 行内容器，常用于包裹价格、标签等小片段文字 -->
```

**定位元素的三种方式（从精确到模糊）：**

| 方式 | HTML 写法 | BeautifulSoup 写法 | 说明 |
|------|----------|-------------------|------|
| id | `id="main"` | `soup.find(id='main')` | 页面唯一，最精确 |
| class | `class="price"` | `soup.find(class_='price')` | 可以有多个同类元素 |
| 标签名 | `<h1>` | `soup.find('h1')` | 最宽泛 |

---

## 3. BeautifulSoup：静态页面爬取

### 3.1 基本流程

```python
import requests
from bs4 import BeautifulSoup

# ── Step 1：下载页面 HTML ────────────────────
url = 'https://example.com'
r = requests.get(url)
# r.text 是整个页面的 HTML 字符串

# ── Step 2：解析 HTML，创建 soup 对象 ────────
soup = BeautifulSoup(r.text, 'html.parser')
# 解析器选项：
# 'html.parser'  → Python 内置，无需安装，够用
# 'lxml'         → 速度更快，需要 pip install lxml（推荐大项目用）

# ── Step 3：用选择器找到你要的元素 ───────────
# （见下方）
```

### 3.2 find 和 find_all

```python
# find()：找到第一个匹配的元素，返回单个 Tag 对象
# find_all()：找到所有匹配的元素，返回列表

# ── 按标签名查找 ──────────────────────────────
soup.find('h1')                         # 第一个 h1 标签
soup.find_all('a')                      # 所有 <a> 链接标签（返回列表）
soup.find_all('p')                      # 所有段落

# ── 按 class 查找（注意：class_ 加下划线，因为 class 是 Python 关键字）──
soup.find('div', class_='product-card')
soup.find_all('span', class_='price')

# ── 按 id 查找（id 是唯一的，用 find 就够）──
soup.find('div', id='main-content')
soup.find(id='search-results')          # 不指定标签名也可以

# ── 同时指定标签 + 属性 ──────────────────────
soup.find_all('a', class_='nav-link')
soup.find_all('img', {'class': 'thumbnail', 'alt': True})  # 有 alt 属性的 img

# ── 限制查找数量 ──────────────────────────────
soup.find_all('div', limit=5)           # 只找前 5 个
```

### 3.3 提取内容

```python
tag = soup.find('p', class_='description')

# ── 获取文字内容 ──────────────────────────────
tag.getText()           # 获取标签内所有文字（包含子标签文字），去掉 HTML 标签
tag.get_text()          # 同上，可以加参数
tag.get_text(strip=True)           # 去掉首尾空白
tag.get_text(separator=' ')        # 子标签之间用空格分隔
tag.string                         # 只有一个文字节点时用（否则返回 None）

# ── 获取属性 ──────────────────────────────────
tag = soup.find('a')
tag.get('href')                    # 获取 href 属性（安全写法，不存在返回 None）
tag['href']                        # 直接取属性（不存在会报 KeyError）
tag.get('class')                   # 返回列表，如 ['nav-link', 'active']

tag = soup.find('img')
tag.get('src')                     # 图片地址
tag.get('alt')                     # 替代文字

# ── 遍历标签内部 ──────────────────────────────
tag.children                       # 直接子节点（生成器）
list(tag.children)                 # 转成列表
tag.descendants                    # 所有后代节点（包括孙子节点）
tag.parent                         # 父节点
tag.find_next_sibling('td')        # 下一个同级兄弟节点

# ── 标签名和属性 ──────────────────────────────
tag.name                           # 标签名，如 'a'、'div'
tag.attrs                          # 所有属性，返回字典
```

### 3.4 CSS 选择器（select）

```python
# select() 用 CSS 选择器语法，比 find_all 更灵活
# 返回列表；select_one() 返回第一个

# ── 基础选择器 ────────────────────────────────
soup.select('h1')                  # 标签名
soup.select('.price')              # class（用 .）
soup.select('#main')               # id（用 #）

# ── 组合选择器 ────────────────────────────────
soup.select('div.product-card')    # div 且 class=product-card
soup.select('div#content p')       # id=content 的 div 里的所有 p（后代）
soup.select('table tr td')         # 表格里的所有数据格

# ── 属性选择器 ────────────────────────────────
soup.select('a[href]')             # 有 href 属性的 a
soup.select('a[href^="https"]')    # href 以 https 开头
soup.select('a[href$=".pdf"]')     # href 以 .pdf 结尾
soup.select('img[src*="thumbnail"]')  # src 包含 thumbnail

# ── 伪类 ──────────────────────────────────────
soup.select('tr:nth-of-type(2)')   # 第二个 tr
soup.select('li:first-child')      # 第一个 li
soup.select('td:last-child')       # 最后一个 td

# 实际使用
prices = [tag.get_text(strip=True) for tag in soup.select('.price')]
links  = [tag.get('href') for tag in soup.select('a.product-link')]
```

---

## 4. BeautifulSoup：实战——抓取表格数据

> 这是数据分析师最常用的爬虫场景：把网页上的 HTML 表格变成 DataFrame。

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

url = 'https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DA0321EN-SkillsNetwork/labs/datasets/HTMLColorCodes.html'

r = requests.get(url)
soup = BeautifulSoup(r.text, 'html.parser')

# ── 方法一：手动解析（灵活，适合复杂表格）────
table = soup.find('table')          # 找第一个 <table>
# 如果有多张表，用 find_all('table')[1] 取第二张

rows_data = []
rows = table.find_all('tr')         # 所有行

for row in rows:
    cols = row.find_all('td')       # 该行所有数据格（<td>）
    # 注意：表头行用的是 <th> 不是 <td>，find_all('td') 会跳过表头行
    if cols:                        # 跳过空行和表头行
        row_data = [col.get_text(strip=True) for col in cols]
        rows_data.append(row_data)

# 单独提取表头
headers = [th.get_text(strip=True)
           for th in table.find_all('th')]

df = pd.DataFrame(rows_data, columns=headers if headers else None)
print(df.head())

# ── 方法二：pd.read_html()（快，适合标准表格）──
# pandas 直接读 HTML 里的 <table>，返回 DataFrame 列表
tables = pd.read_html(url)          # 也可以传 HTML 字符串：pd.read_html(r.text)
df = tables[0]                      # 取第一张表
print(df.head())

# ── 方法三：混合用法（先 soup 定位，再 read_html 解析）──
# 当页面有多张表且 read_html 分不清时
table_html = str(soup.find('table', class_='data-table'))  # 精确找到目标表
df = pd.read_html(table_html)[0]
```

---

## 5. BeautifulSoup：实战——抓取链接与图片

```python
import requests
from bs4 import BeautifulSoup
import os

url = 'https://www.ibm.com/'
r = requests.get(url)
soup = BeautifulSoup(r.text, 'html.parser')

# ── 抓取所有链接 ──────────────────────────────
all_links = []
for a_tag in soup.find_all('a'):
    href = a_tag.get('href')
    text = a_tag.get_text(strip=True)
    if href:                            # 过滤掉没有 href 的 <a> 标签
        all_links.append({'text': text, 'url': href})

# 过滤只保留外部链接（以 http 开头）
external = [l for l in all_links if l['url'].startswith('http')]

# 过滤只保留内部链接（以 / 开头）
internal = [l for l in all_links if l['url'].startswith('/')]

# ── 处理相对路径（把 /about → https://ibm.com/about）──
from urllib.parse import urljoin
base_url = 'https://www.ibm.com'
full_links = [urljoin(base_url, a.get('href'))
              for a in soup.find_all('a') if a.get('href')]

# ── 抓取所有图片 URL ─────────────────────────
img_urls = []
for img in soup.find_all('img'):
    src = img.get('src')
    alt = img.get('alt', '无描述')
    if src:
        full_src = urljoin(base_url, src)   # 处理相对路径
        img_urls.append({'src': full_src, 'alt': alt})

print(f"找到 {len(img_urls)} 张图片")

# ── 批量下载图片 ─────────────────────────────
save_dir = 'downloaded_images'
os.makedirs(save_dir, exist_ok=True)     # 创建保存目录，exist_ok=True 已存在不报错

headers = {'User-Agent': 'Mozilla/5.0'}  # 部分服务器会拒绝没有 User-Agent 的请求

for i, img_info in enumerate(img_urls[:5]):   # 只下载前5张，测试用
    try:
        img_r = requests.get(img_info['src'], headers=headers, timeout=10)
        if img_r.status_code == 200:
            # 从 URL 提取文件名，找不到就用序号命名
            filename = img_info['src'].split('/')[-1].split('?')[0] or f'img_{i}.jpg'
            filepath = os.path.join(save_dir, filename)
            with open(filepath, 'wb') as f:
                f.write(img_r.content)
            print(f"✅ 已下载：{filename}")
    except Exception as e:
        print(f"❌ 下载失败：{img_info['src']} → {e}")
```

---

## 6. Selenium：动态页面爬取

> **为什么需要 Selenium？**
> 很多现代网站（如 Reddit、LinkedIn、小红书）的内容是 JavaScript 执行后
> 才生成的。requests 只能拿到初始 HTML，看不到 JS 渲染后的内容。
> Selenium 控制真实浏览器，等 JS 执行完再抓数据。

### 6.1 安装与启动

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

# ── 启动浏览器（自动下载匹配版本的 ChromeDriver）──
options = webdriver.ChromeOptions()
# options.add_argument('--headless')           # 无头模式：不弹出浏览器窗口
                                               # 正式运行时取消注释，调试时先注释掉
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
options.add_argument('--window-size=1920,1080')

# 伪装成真实浏览器（降低被检测到的概率）
options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                     'AppleWebKit/537.36 (KHTML, like Gecko) '
                     'Chrome/120.0.0.0 Safari/537.36')
options.add_experimental_option('excludeSwitches', ['enable-automation'])
options.add_experimental_option('useAutomationExtension', False)

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)

# 使用完必须关闭，否则浏览器进程会一直占用内存
# driver.quit()
```

### 6.2 基础操作

```python
# ── 打开页面 ──────────────────────────────────
driver.get('https://www.example.com')
print(driver.title)                 # 页面标题
print(driver.current_url)           # 当前 URL

# ── 等待（非常重要！）────────────────────────
# ❌ 不要用 time.sleep()——傻等，不知道页面是否真的加载完
# ✅ 用 WebDriverWait——等到某个元素出现才继续，更可靠

wait = WebDriverWait(driver, timeout=15)  # 最多等 15 秒

# 等到某个元素出现（可见）
element = wait.until(
    EC.presence_of_element_located((By.CSS_SELECTOR, '.post-title'))
)

# 等到某个元素可点击
button = wait.until(
    EC.element_to_be_clickable((By.ID, 'submit-btn'))
)

# ── 定位元素（By 的各种方式）─────────────────
# By.ID              → 最精确，找 id="xxx" 的元素
# By.CLASS_NAME      → 找 class="xxx"（只能写一个类名）
# By.CSS_SELECTOR    → CSS 选择器，最灵活（推荐）
# By.XPATH           → XPath，功能最强但语法复杂
# By.TAG_NAME        → 标签名
# By.NAME            → 表单元素的 name 属性
# By.LINK_TEXT       → 链接的完整文字

driver.find_element(By.ID, 'main')
driver.find_element(By.CSS_SELECTOR, 'div.post-card h2')
driver.find_elements(By.CSS_SELECTOR, 'li.item')  # 返回列表
driver.find_element(By.XPATH, '//div[@class="content"]//p')

# ── 提取内容 ──────────────────────────────────
el = driver.find_element(By.CSS_SELECTOR, 'h1')
el.text                            # 文字内容
el.get_attribute('href')           # 属性值
el.get_attribute('innerHTML')      # 内部 HTML

# ── 操作元素 ──────────────────────────────────
el.click()                         # 点击
el.send_keys('搜索内容')           # 在输入框输入文字
el.send_keys(Keys.RETURN)          # 按回车
el.clear()                         # 清空输入框

# ── 页面滚动 ──────────────────────────────────
# 滚动到页面底部（触发懒加载）
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
time.sleep(2)                      # 等待加载

# 滚动到某个元素
driver.execute_script("arguments[0].scrollIntoView(true);", element)

# 滚动指定像素
driver.execute_script("window.scrollBy(0, 500);")

# ── 页面导航 ──────────────────────────────────
driver.back()                      # 后退
driver.forward()                   # 前进
driver.refresh()                   # 刷新
```

### 6.3 Selenium + BeautifulSoup 组合（推荐）

```python
# Selenium 负责渲染 JS，BeautifulSoup 负责解析 HTML
# 结合两者：Selenium 等页面加载完，把 HTML 交给 soup 解析

driver.get('https://target-website.com')

# 等关键内容加载出来
wait = WebDriverWait(driver, 15)
wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '.post-list')))

# 把当前页面 HTML 交给 BeautifulSoup
soup = BeautifulSoup(driver.page_source, 'html.parser')

# 之后就和普通 BeautifulSoup 一样用
titles = [h2.get_text(strip=True) for h2 in soup.select('h2.post-title')]
```

### 6.4 处理分页和无限滚动

```python
# ── 翻页爬取（点击"下一页"按钮）─────────────
all_data = []

for page in range(1, 6):   # 爬前5页
    driver.get(f'https://example.com/posts?page={page}')
    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '.post-card')))

    soup = BeautifulSoup(driver.page_source, 'html.parser')
    posts = soup.select('.post-card h2')
    all_data.extend([p.get_text(strip=True) for p in posts])

    print(f"第 {page} 页完成，累计 {len(all_data)} 条")
    time.sleep(2)       # 每页之间停 2 秒（礼貌爬虫）

# ── 无限滚动爬取 ─────────────────────────────
# 不断滚动到底部，等新内容加载，重复直到没有新内容

previous_count = 0
scroll_attempts = 0
max_attempts = 10                   # 最多滚动10次

while scroll_attempts < max_attempts:
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(2)                   # 等待新内容加载

    current_items = driver.find_elements(By.CSS_SELECTOR, '.post-item')
    current_count = len(current_items)

    if current_count == previous_count:
        print("没有新内容了，停止滚动")
        break

    print(f"已加载 {current_count} 条")
    previous_count = current_count
    scroll_attempts += 1
```

---

## 7. Reddit API (PRAW)：合法抓取 Reddit

> **为什么用 API 而不是直接爬？**
> Reddit 的页面是 JS 渲染的，Selenium 能用但很慢、容易被封。
> Reddit 提供免费官方 API，数据质量更好，速度更快，法律上也没问题。

### 7.1 申请 API 凭证（一次性操作）

```
1. 登录 Reddit，访问 https://www.reddit.com/prefs/apps
2. 点击 "Create App" 或 "Create Another App"
3. 填写：
   - name：任意名字（如 my_scraper）
   - App type：选 "script"（个人使用）
   - redirect uri：填 http://localhost:8080
4. 点击 Create，记录下：
   - client_id：在应用名称下方的短字符串
   - client_secret：secret 那一栏
```

### 7.2 初始化 PRAW

```python
import praw
import pandas as pd

reddit = praw.Reddit(
    client_id='你的_client_id',
    client_secret='你的_client_secret',
    user_agent='MyBot/1.0 by u/你的用户名',  # 格式：AppName/Version by u/username
    # 只读数据不需要用户名密码，如果要发帖/评论才需要：
    # username='你的用户名',
    # password='你的密码'
)

# 验证是否连接成功
print(reddit.read_only)   # True = 只读模式，能抓数据
```

### 7.3 抓取帖子

```python
# ── 获取某个 subreddit 的帖子 ────────────────
subreddit = reddit.subreddit('datascience')  # 不加 r/

# 排序方式：
# .hot(limit=n)    → 热门（综合热度）
# .new(limit=n)    → 最新
# .top(limit=n, time_filter='week')  → 最高票（时间范围：hour/day/week/month/year/all）
# .rising(limit=n) → 上升中

posts = subreddit.hot(limit=50)   # 获取前 50 条热门帖

posts_data = []
for post in posts:
    posts_data.append({
        'title':       post.title,           # 标题
        'score':       post.score,           # 点赞数（upvotes - downvotes）
        'upvote_ratio':post.upvote_ratio,    # 点赞率（0-1）
        'num_comments':post.num_comments,    # 评论数
        'url':         post.url,             # 帖子链接的 URL
        'permalink':   f"https://reddit.com{post.permalink}",  # Reddit 站内链接
        'author':      str(post.author),     # 作者（可能是 None）
        'created_utc': post.created_utc,     # 发帖时间（UTC 时间戳）
        'selftext':    post.selftext,        # 帖子正文（只有文字帖才有）
        'is_video':    post.is_video,        # 是否是视频帖
        'flair':       post.link_flair_text, # 帖子 flair（标签）
        'subreddit':   str(post.subreddit)   # 所属 subreddit
    })

df = pd.DataFrame(posts_data)

# 转换时间戳为可读时间
df['created_at'] = pd.to_datetime(df['created_utc'], unit='s')
df = df.drop('created_utc', axis=1)

print(df[['title', 'score', 'num_comments']].head())
```

### 7.4 抓取评论

```python
# ── 获取某个帖子的评论 ────────────────────────
post = reddit.submission(url='https://www.reddit.com/r/.../comments/...')
# 或者用 id：post = reddit.submission(id='xxxxxx')

# replace_more() 展开折叠的评论，limit=0 表示不展开（只取顶层评论，速度快）
post.comments.replace_more(limit=0)

comments_data = []
for comment in post.comments.list():    # .list() 展平所有层级评论
    if hasattr(comment, 'body'):        # 过滤掉"更多评论"占位对象
        comments_data.append({
            'body':        comment.body,         # 评论内容
            'score':       comment.score,        # 点赞数
            'author':      str(comment.author),
            'created_utc': comment.created_utc,
            'depth':       comment.depth,        # 评论层级（0=顶层，1=回复）
            'is_op':       comment.is_submitter  # 是否是原帖作者回复
        })

df_comments = pd.DataFrame(comments_data)
df_comments['created_at'] = pd.to_datetime(df_comments['created_utc'], unit='s')
```

### 7.5 搜索与多 subreddit 爬取

```python
# ── 搜索关键词 ────────────────────────────────
results = reddit.subreddit('all').search(
    query='machine learning jobs UK',
    sort='relevance',               # 'relevance'/'hot'/'new'/'top'
    time_filter='month',            # 'hour'/'day'/'week'/'month'/'year'/'all'
    limit=100
)

search_data = [{'title': p.title, 'score': p.score, 'url': p.url}
               for p in results]

# ── 同时爬多个 subreddit ──────────────────────
# 用 + 号连接多个 subreddit（最多可以几十个）
multi_sub = reddit.subreddit('python+datascience+MachineLearning')
posts = multi_sub.hot(limit=100)

# ── 获取 subreddit 的基本信息 ─────────────────
sub = reddit.subreddit('datascience')
print(sub.display_name)             # 名称
print(sub.subscribers)              # 订阅人数
print(sub.public_description)       # 简介
```

---

## 8. 数据清洗与存储

```python
import pandas as pd
import re

# ── 常见文本清洗 ──────────────────────────────
def clean_text(text):
    if not text or text in ['[deleted]', '[removed]']:
        return None
    text = text.strip()                         # 去首尾空白
    text = re.sub(r'\s+', ' ', text)            # 多个空白合并为一个
    text = re.sub(r'http\S+', '', text)         # 去掉 URL
    text = re.sub(r'[^\w\s\u4e00-\u9fff]', '', text)  # 只保留字母数字空格中文
    return text.strip()

df['title_clean'] = df['title'].apply(clean_text)

# ── 处理 None / 缺失值 ────────────────────────
df['author'].replace('None', pd.NA, inplace=True)
df.dropna(subset=['title'], inplace=True)

# ── 保存数据 ──────────────────────────────────
# CSV（最通用）
df.to_csv('reddit_posts.csv', index=False, encoding='utf-8-sig')
# utf-8-sig = 带 BOM 的 UTF-8，Excel 打开中文不会乱码

# Excel（方便分享给别人）
df.to_excel('reddit_posts.xlsx', index=False, sheet_name='Posts')

# JSON（适合保留原始结构）
df.to_json('reddit_posts.json', orient='records',
           force_ascii=False, indent=2)
# force_ascii=False = 中文不转义成 \uXXXX
```

---

## 9. 反爬应对与爬虫礼仪

> **重要原则**：爬虫要有礼貌，不要给服务器造成过大压力。
> 激进爬虫轻则 IP 被封，重则有法律风险（各国数据爬取法律不同）。

### 9.1 常见反爬手段与应对

```python
import requests
import time
import random

# ── 问题1：被识别为爬虫（User-Agent 检测）────
# 服务器看你的 User-Agent 是 'python-requests/2.x.x' 就直接拒绝

# ✅ 伪装成浏览器
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                  'AppleWebKit/537.36 (KHTML, like Gecko) '
                  'Chrome/120.0.0.0 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Referer': 'https://www.google.com/'  # 伪装从 Google 跳转过来
}
r = requests.get(url, headers=headers)

# ── 问题2：请求频率过快（频率检测）──────────
# ✅ 加随机延迟（让请求间隔看起来像人在操作）
time.sleep(random.uniform(1.5, 4.0))   # 每次请求随机等1.5~4秒

# ── 问题3：IP 被封 ──────────────────────────
# ✅ 使用 Session 对象（复用 TCP 连接，更像真实浏览器行为）
session = requests.Session()
session.headers.update(headers)

r1 = session.get('https://example.com/page1')
r2 = session.get('https://example.com/page2')  # 同一个 session，携带 cookies

# ── 检查 robots.txt（爬虫礼仪）──────────────
# robots.txt 规定了哪些页面允许爬，哪些不允许
r = requests.get('https://www.example.com/robots.txt')
print(r.text)
# Disallow: /private/  → 这个路径不允许爬
# Allow: /public/      → 这个路径允许爬

# ── 超时与重试 ────────────────────────────────
def get_with_retry(url, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            r = requests.get(url, timeout=15, **kwargs)
            r.raise_for_status()
            return r
        except requests.RequestException as e:
            print(f"第 {attempt+1} 次失败：{e}")
            if attempt < max_retries - 1:
                wait = 2 ** attempt         # 指数退避：1s → 2s → 4s
                print(f"等待 {wait} 秒后重试...")
                time.sleep(wait)
    return None
```

### 9.2 爬虫礼仪检查清单

```
✅ 爬取前查看 robots.txt，遵守 Disallow 规则
✅ 每次请求之间加延迟（≥ 1 秒）
✅ 不要同时发起大量并发请求
✅ 设置合理的 User-Agent，让对方知道你是谁
✅ 有官方 API 的优先用 API
✅ 爬取数据仅用于个人学习/研究，不商业使用
✅ 不爬取登录才能看的私人内容
✅ 不爬取个人隐私信息（姓名、地址、电话等）
```

---

## 10. 完整实战案例

> 以下是两个可以直接运行的完整案例。

### 案例 A：抓取维基百科表格

```python
"""
目标：抓取维基百科"GDP by country"表格，保存为 DataFrame
使用：requests + BeautifulSoup + pandas
"""
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time

def scrape_wikipedia_table(url, table_class=None):
    headers = {'User-Agent': 'Mozilla/5.0 (compatible; LearningBot/1.0)'}

    r = requests.get(url, headers=headers, timeout=15)
    r.raise_for_status()

    soup = BeautifulSoup(r.text, 'html.parser')

    # 找目标表格
    if table_class:
        table = soup.find('table', class_=table_class)
    else:
        table = soup.find('table', class_='wikitable')

    if not table:
        print("未找到表格")
        return None

    # 提取表头
    headers_row = table.find('tr')
    headers_list = [th.get_text(strip=True)
                    for th in headers_row.find_all(['th', 'td'])]

    # 提取数据行
    rows_data = []
    for row in table.find_all('tr')[1:]:    # 跳过表头行
        cols = row.find_all(['td', 'th'])
        if cols:
            row_data = [col.get_text(strip=True) for col in cols]
            # 对齐列数（有些行因为合并单元格会少列）
            while len(row_data) < len(headers_list):
                row_data.append('')
            rows_data.append(row_data[:len(headers_list)])

    df = pd.DataFrame(rows_data, columns=headers_list)
    return df

# 使用
url = 'https://en.wikipedia.org/wiki/List_of_countries_by_GDP_(nominal)'
df = scrape_wikipedia_table(url)
if df is not None:
    print(df.head(10))
    df.to_csv('gdp_by_country.csv', index=False, encoding='utf-8-sig')
    print("已保存到 gdp_by_country.csv")
```

### 案例 B：抓取 Reddit 帖子并做简单分析

```python
"""
目标：抓取 r/datascience 热门帖，分析关键词和互动数据
使用：PRAW + pandas + 基础文本分析
"""
import praw
import pandas as pd
import re
from collections import Counter

# ── 初始化（填入你的凭证）────────────────────
reddit = praw.Reddit(
    client_id='YOUR_CLIENT_ID',
    client_secret='YOUR_CLIENT_SECRET',
    user_agent='DataScienceAnalyzer/1.0 by u/YOUR_USERNAME'
)

# ── 抓取数据 ──────────────────────────────────
subreddit = reddit.subreddit('datascience')

records = []
print("正在抓取数据...")

for post in subreddit.hot(limit=100):
    records.append({
        'title':        post.title,
        'score':        post.score,
        'num_comments': post.num_comments,
        'upvote_ratio': post.upvote_ratio,
        'created_at':   pd.to_datetime(post.created_utc, unit='s'),
        'flair':        post.link_flair_text or 'No Flair',
        'is_self':      post.is_self,         # True = 文字帖；False = 链接帖
        'url':          post.url
    })

df = pd.DataFrame(records)
print(f"共抓取 {len(df)} 条帖子")

# ── 基础分析 ──────────────────────────────────
print("\n=== 基础统计 ===")
print(df[['score', 'num_comments', 'upvote_ratio']].describe().round(2))

print("\n=== 各 Flair 帖子数量 ===")
print(df['flair'].value_counts().head(10))

print("\n=== 高互动帖子（评论数 Top 5）===")
top_commented = df.nlargest(5, 'num_comments')[['title', 'score', 'num_comments']]
print(top_commented.to_string(index=False))

# ── 标题关键词分析 ────────────────────────────
def extract_keywords(titles):
    # 简单分词：转小写，提取 4 个字母以上的单词
    words = []
    for title in titles:
        words.extend(re.findall(r'\b[a-z]{4,}\b', title.lower()))
    # 过滤停用词
    stopwords = {'that', 'this', 'with', 'from', 'have', 'what',
                 'your', 'about', 'they', 'been', 'will', 'which',
                 'when', 'more', 'some', 'there', 'just', 'into'}
    return Counter(w for w in words if w not in stopwords)

keyword_counts = extract_keywords(df['title'])
print("\n=== 标题高频词 Top 15 ===")
for word, count in keyword_counts.most_common(15):
    print(f"  {word:<20} {count}")

# ── 保存 ──────────────────────────────────────
df.to_csv('reddit_datascience.csv', index=False, encoding='utf-8-sig')
print("\n已保存到 reddit_datascience.csv")
```

---

## 附：方法速查对照

```
静态 HTML 页面（源码能看到内容）
   ↓ requests.get(url).text
   ↓ BeautifulSoup(html, 'html.parser')
   ↓ soup.find() / find_all() / select()
   ↓ tag.get_text() / tag.get('href')
   → pd.DataFrame / df.to_csv()

动态 JS 页面（源码看不到内容）
   ↓ webdriver.Chrome(options)
   ↓ driver.get(url)
   ↓ WebDriverWait(driver, 15).until(EC.presence_of_element_located(...))
   ↓ BeautifulSoup(driver.page_source, 'html.parser')  ← 交给 soup 解析
   → pd.DataFrame / df.to_csv()
   ↓ driver.quit()  ← 记得关闭

Reddit（有官方 API）
   ↓ praw.Reddit(client_id, client_secret, user_agent)
   ↓ reddit.subreddit('name').hot(limit=100)
   ↓ post.title / post.score / post.num_comments
   → pd.DataFrame / df.to_csv()
```
