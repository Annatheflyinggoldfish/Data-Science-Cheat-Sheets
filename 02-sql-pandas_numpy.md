## SQL · Pandas · NumPy 对照参考

---

## 目录

1. [环境准备](#1-环境准备)
2. [数据读取与导出](#2-数据读取与导出)
3. [查看与探索数据](#3-查看与探索数据)
4. [选取与过滤数据](#4-选取与过滤数据)
5. [排序与排名](#5-排序与排名)
6. [新增与修改数据](#6-新增与修改数据)
7. [删除数据](#7-删除数据)
8. [聚合与分组](#8-聚合与分组)
9. [连接与合并](#9-连接与合并)
10. [缺失值处理](#10-缺失值处理)
11. [字符串操作](#11-字符串操作)
12. [正则表达式](#12-正则表达式)
13. [日期操作](#13-日期操作)
14. [窗口函数](#14-窗口函数)
15. [条件逻辑](#15-条件逻辑)
16. [透视与重塑](#16-透视与重塑)
17. [NumPy 数组操作](#17-numpy-数组操作)
18. [类型转换](#18-类型转换)
19. [性能与调试技巧](#19-性能与调试技巧)

---

## 1. 环境准备

```python
import numpy as np
import pandas as pd

pd.set_option('display.max_columns', None)   # 显示所有列
pd.set_option('display.max_rows', 100)       # 显示最多 100 行
pd.set_option('display.float_format', '{:.2f}'.format)  # 浮点数保留 2 位
```

---

## 2. 数据读取与导出

| 操作 | SQL | Pandas |
|------|-----|--------|
| 读取表/文件 | `SELECT * FROM table;` | `pd.read_csv('f.csv')` |
| 写入 | `INSERT INTO ...` | `df.to_csv('f.csv', index=False)` |

```python
# 读取
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv',
    index_col=0,           # 指定索引列
    parse_dates=['date'],  # 自动解析日期
    usecols=['a','b','c'], # 只读指定列
    dtype={'id': str},     # 指定列类型
    na_values=['NA','-']   # 自定义空值标识
)
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')
df = pd.read_json('file.json')
df = pd.read_parquet('file.parquet')   # 大数据集推荐格式

# 导出
df.to_csv('output.csv', index=False)
df.to_excel('output.xlsx', index=False, sheet_name='Result')
df.to_parquet('output.parquet')
```

---

## 3. 查看与探索数据

| 操作 | SQL | Pandas |
|------|-----|--------|
| 前几行 | `SELECT * FROM t LIMIT 5;` | `df.head(5)` |
| 后几行 | — | `df.tail(5)` |
| 行列数 | `SELECT COUNT(*) FROM t;` | `df.shape` |
| 列名类型 | `DESCRIBE table;` | `df.dtypes` / `df.info()` |
| 数值摘要 | — | `df.describe()` |
| 唯一值 | `SELECT DISTINCT col FROM t;` | `df['col'].unique()` |
| 唯一值数量 | `SELECT COUNT(DISTINCT col) FROM t;` | `df['col'].nunique()` |
| 频次统计 | `SELECT col, COUNT(*) FROM t GROUP BY col;` | `df['col'].value_counts()` |
| 频次统计 | `SELECT col, COUNT(*) FROM t GROUP BY col;` | `df['col'].value_counts()` |

```python
df.head()
df.info()                        # 类型 + 非空数量 + 内存
df.describe()                    # count/mean/std/min/25%/50%/75%/max
df.describe(include='all')       # 含非数值列
df['col'].value_counts()         # 频次，默认降序
df['col'].value_counts(normalize=True)  # 频率（占比）
df['col'].value_counts(dropna=False) # 保留空值
df['col'].unique()               # 所有唯一值（数组）
df['col'].nunique()              # 唯一值数量
df.corr()                        # 数值列相关性矩阵
df.memory_usage(deep=True)       # 各列内存占用
```

---

## 4. 选取与过滤数据

| 操作 | SQL | Pandas |
|------|-----|--------|
| 选列 | `SELECT col FROM t;` | `df['col']` / `df[['c1','c2']]` |
| 按标签选行列 | — | `df.loc[行, 列]` |
| 按位置选行列 | — | `df.iloc[行, 列]` |
| 条件过滤 | `WHERE col > 5` | `df[df['col'] > 5]` |
| AND | `WHERE a AND b` | `df[(cond1) & (cond2)]` |
| OR | `WHERE a OR b` | `df[(cond1) \| (cond2)]` |
| NOT | `WHERE NOT cond` | `df[~cond]` |
| IN | `WHERE col IN (a,b)` | `df[df['col'].isin([a,b])]` |
| NOT IN | `WHERE col NOT IN (a,b)` | `df[~df['col'].isin([a,b])]` |
| BETWEEN | `WHERE col BETWEEN a AND b` | `df[df['col'].between(a, b)]` |
| IS NULL | `WHERE col IS NULL` | `df[df['col'].isna()]` |
| IS NOT NULL | `WHERE col IS NOT NULL` | `df[df['col'].notna()]` |
| LIKE '%x%' | `WHERE col LIKE '%x%'` | `df[df['col'].str.contains('x')]` |

```python
# loc：标签索引（含两端）
df.loc[0:5, 'col1':'col3']
df.loc[df['age'] > 25, ['name', 'age']]  # 条件 + 指定列

# iloc：位置索引（不含末端）
df.iloc[0:5, 1:4]
df.iloc[:, -1]     # 最后一列

# query（可读性好，适合复杂条件）
df.query("age > 25 and city == 'London'")
df.query("col in @my_list")   # 引用外部变量用 @

# 随机抽样
df.sample(n=100)              # 抽 100 行
df.sample(frac=0.1)           # 抽 10%
```

---

## 5. 排序与排名

| 操作 | SQL | Pandas |
|------|-----|--------|
| 升序排序 | `ORDER BY col ASC` | `df.sort_values('col')` |
| 降序排序 | `ORDER BY col DESC` | `df.sort_values('col', ascending=False)` |
| 多列排序 | `ORDER BY c1 ASC, c2 DESC` | `df.sort_values(['c1','c2'], ascending=[True,False])` |
| RANK | `RANK() OVER (ORDER BY col DESC)` | `df['col'].rank(method='min', ascending=False)` |
| DENSE_RANK | `DENSE_RANK() OVER (ORDER BY col DESC)` | `df['col'].rank(method='dense', ascending=False)` |
| ROW_NUMBER | `ROW_NUMBER() OVER (ORDER BY col)` | `df['col'].rank(method='first')` |
| PARTITION BY | `DENSE_RANK() OVER (PARTITION BY g ORDER BY col)` | `df.groupby('g')['col'].rank(method='dense')` |

```python
df.sort_values('col', ascending=False, inplace=True)
df.sort_values(['col1','col2'], ascending=[True, False])
df.sort_index()   # 按索引排序

# 取前 N（类似 SQL TOP / LIMIT）
df.nlargest(5, 'salary')    # 最大 5 行
df.nsmallest(5, 'salary')   # 最小 5 行

# 分组排名
df['rank'] = df.groupby('dept')['salary'].rank(method='dense', ascending=False)

# 取前10的最大值
df.sort_values().head(10) 硬性截断，只给你 10 条数据。
pivot.sum().nlargest(10, keep='all')：# 如果有三个一样的90分，那么会把这三个 90 分的全带上，最终返回 12 条数据。
# 注意：.nlargest(10) 只能用于数值型数据（Series 或 DataFrame 的某几列）
```

---

## 6. 新增与修改数据

| 操作 | SQL | Pandas |
|------|-----|--------|
| 新增列 | `ALTER TABLE t ADD col ...` | `df['new'] = ...` |
| 计算列 | — | `df['new'] = df['a'] + df['b']` |
| 修改列名 | `ALTER TABLE t RENAME COLUMN` | `df.rename(columns={'old':'new'})` |
| 更新值 | `UPDATE t SET col=val WHERE cond` | `df.loc[cond, 'col'] = val` |
| 替换值 | — | `df['col'].replace({'a':1, 'b':2})` |
| 类型转换 | `CAST(col AS INT)` | `df['col'].astype(int)` |

```python
# 新增列
df['full_name'] = df['first'] + ' ' + df['last']
df['tax'] = df['salary'] * 0.2
df['label'] = df['score'].apply(lambda x: 'high' if x > 80 else 'low')

# assign（链式写法，不修改原 df）
df = df.assign(
    tax=lambda x: x['salary'] * 0.2,
    seniority=lambda x: 2024 - x['join_year']
)

# 修改
df.rename(columns={'old_name': 'new_name'}, inplace=True)
df.columns = df.columns.str.lower().str.replace(' ', '_')  # 批量规范列名

df.loc[df['status'] == 'inactive', 'score'] = 0  # 条件更新
df['col'].replace({'Yes': 1, 'No': 0}, inplace=True)
df['col'] = df['col'].map({'a': 1, 'b': 2})       # map 替换
```

---

## 7. 删除数据

| 操作 | SQL | Pandas |
|------|-----|--------|
| 删除行 | `DELETE FROM t WHERE cond` | `df.drop(index=...)` / `df[~cond]` |
| 删除列 | `ALTER TABLE t DROP COLUMN col` | `df.drop(columns=['col'])` |
| 去重 | `SELECT DISTINCT ...` | `df.drop_duplicates()` |

```python
df.drop(columns=['col1', 'col2'], inplace=True)
df.drop(index=[0, 1, 2], inplace=True)

df.drop_duplicates()                         # 全列去重
df.drop_duplicates(subset=['col1', 'col2'])  # 按指定列去重
df.drop_duplicates(subset=['id'], keep='last')  # 保留最后出现的

df = df[df['age'] > 0]    # 通过过滤删除不满足条件的行
```

---

## 8. 聚合与分组

| 操作 | SQL | Pandas |
|------|-----|--------|
| COUNT | `COUNT(col)` | `df['col'].count()` |
| COUNT DISTINCT | `COUNT(DISTINCT col)` | `df['col'].nunique()` |
| SUM | `SUM(col)` | `df['col'].sum()` |
| AVG | `AVG(col)` | `df['col'].mean()` |
| 中位数 | — | `df['col'].median()` |
| 众数 | — | `df['col'].mode()[0]` |
| 分位数 | — | `df['col'].quantile(0.25)` |
| MIN/MAX | `MIN(col)` / `MAX(col)` | `df['col'].min()` / `.max()` |
| STD | `STDDEV(col)` | `df['col'].std()` |
| GROUP BY | `GROUP BY col` | `df.groupby('col')` |
| HAVING | `HAVING COUNT(*) > 5` | `.filter(lambda x: len(x) > 5)` |
| 多聚合 | `SELECT col, SUM(a), AVG(b) ...` | `.agg({'a':'sum','b':'mean'})` |
| 行数统计（含NaN） | `COUNT(*)` | `df.groupby('col').size()` |
| 行数统计（非NaN） | `COUNT(col)` | `df.groupby('col').count()` |
| 分组频率统计 | — | `df.groupby(['a','b']).size().reset_index(name='Count')` |
| 分组广播回原行 | 窗口函数 | `df.groupby('col')['val'].transform('mean')` |
| 数值区间分组 | `CASE WHEN` | `pd.cut(df['col'], bins=[...], labels=[...])` |
| 分位数自动分组 | — | `pd.qcut(df['col'], q=4, labels=[...])` |
| 列表拆行 | — | `df.explode('col')` |
| 多值列拆分 | — | `df['col'].str.split(';')` + `.explode()` |
| 交叉计数 | — | `pd.crosstab(df['a'], df['b'])` |
| 长转宽（唯一值） | `PIVOT` | `df.pivot(index, columns, values)` |
| 长转宽（有重复） | `PIVOT` | `df.pivot_table(values, index, columns, aggfunc)` |

```python
# 基本聚合
df['col'].agg(['sum', 'mean', 'min', 'max', 'std', 'count'])

# groupby 单聚合
df.groupby('dept')['salary'].mean()
df.groupby('dept')['salary'].agg(['mean', 'sum', 'count'])

# groupby 多列、多聚合（推荐写法）
df.groupby(['dept', 'gender']).agg(
    avg_salary=('salary', 'mean'),
    total_salary=('salary', 'sum'),
    headcount=('emp_id', 'count'),
    max_salary=('salary', 'max')
).reset_index()

# size() vs count()
# size()：数每组有多少行，包含NaN，返回Series
df.groupby('col').size()
# count()：数每组每列非NaN的值，返回DataFrame（每列都算）
df.groupby('col').count()
# 典型用法：统计频率后重命名
df.groupby(['Country', 'Language']).size().reset_index(name='Count')

# 自定义聚合函数
df.groupby('dept')['salary'].agg(lambda x: x.quantile(0.9))  # 90分位数

# HAVING 等价：过滤分组后的结果
df.groupby('dept').filter(lambda x: x['salary'].mean() > 50000)

# transform：分组聚合结果广播回原始行（不减少行数）
# 类似 SQL 窗口函数
df['dept_avg'] = df.groupby('dept')['salary'].transform('mean')
df['pct_of_dept'] = df['salary'] / df.groupby('dept')['salary'].transform('sum')

# pd.cut：按数值区间分组，创建新列
# right=False → 左闭右开 [0,5), [5,10)；right=True（默认）→ 左开右闭 (0,5]
df['ExperienceLevel'] = pd.cut(
    df['YearsCodePro'],
    bins=[0, 5, 10, 20, 52],
    labels=['0-5', '5-10', '10-20', '>20 years'],
    right=False
)
# pd.qcut：按分位数自动分组（每组人数相等）
df['SalaryQuartile'] = pd.qcut(df['salary'], q=4, labels=['Q1','Q2','Q3','Q4'])

# explode：把列表型单元格拆成多行，其他列值自动复制
# 适用场景：一列里存了多个值，如 "Python;JavaScript;SQL"
df_exploded = df.assign(
    Language=df['LanguageHaveWorkedWith'].str.split(';')
).explode('Language')
# 注意：explode后index会重复，需要时用reset_index(drop=True)重置

# pivot：长格式转宽格式（要求index+columns组合唯一，否则报错）
df.pivot(index='Country', columns='Language', values='Count')

# pivot_table：长格式转宽格式（有重复值时自动聚合，更健壮）
df.pivot_table(
    values='Count',
    index='Country',
    columns='Language',
    aggfunc='sum',
    fill_value=0
)

# crosstab：两个分类变量交叉计数（无需提前groupby）
pd.crosstab(df['Employment'], df['RemoteWork'])
# 按行总数排序
crosstab.loc[crosstab.sum(axis=1).sort_values(ascending=False).index]
```

---

## 9. 连接与合并

| 操作 | SQL | Pandas |
|------|-----|--------|
| INNER JOIN | `INNER JOIN t2 ON t1.id = t2.id` | `pd.merge(df1, df2, on='id')` |
| LEFT JOIN | `LEFT JOIN t2 ON ...` | `pd.merge(df1, df2, on='id', how='left')` |
| RIGHT JOIN | `RIGHT JOIN t2 ON ...` | `pd.merge(df1, df2, on='id', how='right')` |
| FULL OUTER JOIN | `FULL OUTER JOIN t2 ON ...` | `pd.merge(df1, df2, on='id', how='outer')` |
| UNION（去重） | `UNION` | `pd.concat([df1,df2]).drop_duplicates()` |
| UNION ALL | `UNION ALL` | `pd.concat([df1, df2])` |
| 横向拼接 | — | `pd.concat([df1, df2], axis=1)` |

```python
# merge
pd.merge(df1, df2, on='key')
pd.merge(df1, df2, left_on='col1', right_on='col2')   # 列名不同
pd.merge(df1, df2, on='key', how='left', suffixes=('_left','_right'))

# 多键连接
pd.merge(df1, df2, on=['key1', 'key2'])

# concat
pd.concat([df1, df2], axis=0, ignore_index=True)   # 纵向（UNION ALL）
pd.concat([df1, df2], axis=1)                       # 横向

# join（以索引连接，merge 的简写）
df1.join(df2, how='left')
```

---

## 10. 缺失值处理

| 操作 | SQL | Pandas |
|------|-----|--------|
| 检测空值 | `IS NULL` | `df.isna()` / `df.isnull()` |
| 检测非空值 | `IS NOT NULL` | `df.notna()` / `df.notnull()` |
| 统计空值 | — | `df.isna().sum()` |
| 删除空值行 | `WHERE col IS NOT NULL` | `df.dropna()` |
| 填充空值 | `COALESCE(col, 0)` | `df.fillna(0)` |

```python
df.isna().sum()                          # 各列缺失数量
df.isna().sum() / len(df)                # 缺失率
df.isna().any(axis=1)                    # 含任意空值的行（布尔）

df.dropna()                              # 删除含空值的行
df.dropna(axis=1)                        # 删除含空值的列
df.dropna(subset=['col1', 'col2'])       # 只检查指定列
df.dropna(thresh=3)                      # 保留非空值 ≥ 3 的行

df.fillna(0)                             # 填充为固定值
df['col'].fillna(df['col'].mean())       # 填充为均值
df['col'].fillna(df['col'].median())     # 填充为中位数
df['col'].ffill()                        # 前向填充
df['col'].bfill()                        # 后向填充
df.fillna({'col1': 0, 'col2': 'unknown'})  # 各列分别填充

# 检测无穷值（数值运算后可能出现）
df.replace([np.inf, -np.inf], np.nan)
```

---

## 11. 字符串操作

| 操作 | SQL | Pandas |
|------|-----|--------|
| 转大写 | `UPPER(col)` | `df['col'].str.upper()` |
| 转小写 | `LOWER(col)` | `df['col'].str.lower()` |
| 去空格 | `TRIM(col)` | `df['col'].str.strip()` |
| 拼接 | `CONCAT(a, b)` | `df['a'] + df['b']` |
| 截取 | `SUBSTRING(col,1,3)` | `df['col'].str[0:3]` |
| 长度 | `LENGTH(col)` | `df['col'].str.len()` |
| 包含 | `LIKE '%x%'` | `df['col'].str.contains('x')` |
| 替换 | `REPLACE(col,'a','b')` | `df['col'].str.replace('a','b')` |
| 分割 | — | `df['col'].str.split(',')` |

```python
df['col'].str.lower()
df['col'].str.upper()
df['col'].str.strip()                        # 去首尾空格
df['col'].str.lstrip() | .rstrip()           # 去左/右空格
df['col'].str.replace('old', 'new', regex=False)
df['col'].str.contains('pattern', na=False)  # na=False 避免空值报错
df['col'].str.startswith('prefix')
df['col'].str.endswith('suffix')
df['col'].str.split(',', expand=True)        # 分割成多列
df['col'].str.split(',').str[0]              # 取分割后第一个
df['col'].str.len()
df['col'].str[0:3]                           # 切片（等价 SUBSTRING）
df['col'].str.pad(10, side='left', fillchar='0')  # 补零/补位
df['col'].str.zfill(5)                       # 左侧补零到 5 位
df['col'].str.cat(sep=', ')                  # 拼接所有值为一个字符串

# 提取（正则）
df['col'].str.extract(r'(\d{4})')            # 提取第一个匹配
df['col'].str.extractall(r'(\d+)')           # 提取所有匹配
df['col'].str.findall(r'\d+')               # 返回所有匹配的列表
df['col'].str.match(r'^[A-Za-z]+$')         # 是否完整匹配（布尔）
```

---

## 12. 正则表达式

> SQL 用 `REGEXP_LIKE`，Pandas 用 `.str.contains` / `.str.match` / `.str.extract`，底层规则相同。

### 核心语法速查

| 类别 | 表达式 | 含义 |
|------|--------|------|
| **锚点** | `^` | 开头 |
| | `$` | 结尾 |
| | `^...$` | 匹配整个字符串 |
| **字符类** | `[A-Z]` | 大写字母 |
| | `[a-z]` | 小写字母 |
| | `[0-9]` | 数字 |
| | `[A-Za-z0-9]` | 字母或数字 |
| | `[^abc]` | 不是 a、b、c |
| | `[.]` | 真正的点 |
| | `\d` | 数字（等价 `[0-9]`）|
| | `\w` | 字母/数字/下划线 |
| | `\s` | 空白字符 |
| **数量** | `*` | 0 个或多个 |
| | `+` | 1 个或多个 |
| | `?` | 0 个或 1 个 |
| | `{n}` | 恰好 n 个 |
| | `{m,n}` | m 到 n 个 |
| **其他** | `\|` | 或（a\|b） |
| | `()` | 分组/捕获 |
| | `.` | 任意单个字符 |

```sql
-- SQL (REGEXP_LIKE)
WHERE REGEXP_LIKE(col, '^[A-Za-z]')                         -- 以字母开头
WHERE REGEXP_LIKE(col, '^[0-9]+$')                          -- 纯数字
WHERE REGEXP_LIKE(col, '^[A-Za-z][A-Za-z0-9_]*$')          -- 字母开头+字母数字下划线
WHERE REGEXP_LIKE(col, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')  -- 邮箱
WHERE REGEXP_LIKE(col, '^[0-9]{11}$')                       -- 11 位手机号
WHERE REGEXP_LIKE(col, '[0-9]')                             -- 包含数字
-- 第三参数：'c' 大小写敏感，'i' 不敏感（默认）
```

```python
# Pandas 正则
df[df['col'].str.contains(r'^[A-Za-z]', na=False)]           # 以字母开头
df[df['col'].str.match(r'^[0-9]+$')]                         # 纯数字（整列匹配）
df[df['col'].str.contains(r'@.*\.com$', case=False)]         # 邮箱结尾（不敏感）

df['col'].str.extract(r'(\d{4}-\d{2}-\d{2})')               # 提取日期格式
df['col'].str.replace(r'\s+', ' ', regex=True)               # 多个空白替换为单个
df['col'].str.replace(r'[^A-Za-z0-9]', '', regex=True)      # 只保留字母数字
```

---

## 13. 日期操作

| 操作 | SQL | Pandas |
|------|-----|--------|
| 当前日期 | `CURRENT_DATE` | `pd.Timestamp.today()` |
| 转日期类型 | `CAST(col AS DATE)` | `pd.to_datetime(df['col'])` |
| 提取年 | `YEAR(col)` | `df['col'].dt.year` |
| 提取月 | `MONTH(col)` | `df['col'].dt.month` |
| 提取日 | `DAY(col)` | `df['col'].dt.day` |
| 日期差（天） | `DATEDIFF(a, b)` | `(df['a'] - df['b']).dt.days` |
| 加天数 | `DATE_ADD(col, INTERVAL 7 DAY)` | `df['col'] + pd.Timedelta(days=7)` |
| 格式化 | `DATE_FORMAT(col, '%Y-%m')` | `df['col'].dt.strftime('%Y-%m')` |

```python
df['date'] = pd.to_datetime(df['date'])
df['date'] = pd.to_datetime(df['date'], format='%d/%m/%Y')  # 指定格式

# 提取
df['date'].dt.year | .month | .day
df['date'].dt.hour | .minute | .second
df['date'].dt.dayofweek        # 0=周一，6=周日
df['date'].dt.day_name()       # 'Monday', 'Tuesday'...
df['date'].dt.quarter          # 季度 (1-4)
df['date'].dt.is_month_end     # 是否月末（布尔）
df['date'].dt.is_month_start   # 是否月初
df['date'].dt.week             # 第几周

# 计算
(df['end_date'] - df['start_date']).dt.days        # 天数差
df['date'] + pd.Timedelta(days=30)
df['date'] + pd.DateOffset(months=1)               # 加一个月

# 格式化
df['date'].dt.strftime('%Y-%m-%d')
df['date'].dt.strftime('%Y-%m')                    # 年-月

# 按时间分组
df.groupby(df['date'].dt.month)['sales'].sum()
df.groupby(df['date'].dt.to_period('M'))['sales'].sum()  # 按月分组

# 时间范围过滤
df[df['date'] >= '2023-01-01']
df[df['date'].between('2023-01-01', '2023-12-31')]
```

---

## 14. 窗口函数

| 操作 | SQL | Pandas |
|------|-----|--------|
| 累计求和 | `SUM() OVER (ORDER BY date)` | `df['col'].cumsum()` |
| 移动均值（3行） | `AVG() OVER (ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | `df['col'].rolling(3).mean()` |
| RANK | `RANK() OVER (ORDER BY col DESC)` | `df['col'].rank(method='min')` |
| DENSE_RANK | `DENSE_RANK() OVER (...)` | `df['col'].rank(method='dense')` |
| ROW_NUMBER | `ROW_NUMBER() OVER (...)` | `df['col'].rank(method='first')` |
| LAG(n) | `LAG(col, n) OVER (ORDER BY date)` | `df['col'].shift(n)` |
| LEAD(n) | `LEAD(col, n) OVER (ORDER BY date)` | `df['col'].shift(-n)` |
| PARTITION BY | `DENSE_RANK() OVER (PARTITION BY g ORDER BY col)` | `df.groupby('g')['col'].rank(method='dense')` |

```python
# 累计
df['cumsum']   = df['sales'].cumsum()
df['cumprod']  = df['sales'].cumprod()
df['cummax']   = df['sales'].cummax()

# 滚动窗口
df['rolling_avg'] = df['sales'].rolling(window=7).mean()        # 7日均值
df['rolling_sum'] = df['sales'].rolling(window=7, min_periods=1).sum()  # 不足7行也计算

# 扩展窗口（从头累计到当前行）
df['expand_mean'] = df['sales'].expanding().mean()

# 分组窗口（PARTITION BY）
df['dept_avg']   = df.groupby('dept')['salary'].transform('mean')
df['dept_rank']  = df.groupby('dept')['salary'].rank(method='dense', ascending=False)
df['dept_cumsum']= df.groupby('dept')['sales'].transform('cumsum')

# LAG / LEAD（位移）
df['prev_sales'] = df['sales'].shift(1)   # 上一行
df['next_sales'] = df['sales'].shift(-1)  # 下一行

# 环比差 / 增长率
df['diff']       = df['sales'] - df['sales'].shift(1)
df['pct_change'] = df['sales'].pct_change()               # (当前-上一)/上一
df['yoy']        = df['sales'].pct_change(periods=12)     # 同比（12个月前）

# ⚠️ SQL 注意：窗口函数不能直接用在 WHERE，需包子查询
# SELECT * FROM (SELECT col, LAG(col) OVER (...) AS prev FROM t) t WHERE col = prev;
```

---

## 15. 条件逻辑

| 操作 | SQL | Pandas |
|------|-----|--------|
| CASE WHEN | `CASE WHEN cond THEN a ELSE b END` | `np.where(cond, a, b)` |
| 多条件 | `CASE WHEN c1 THEN a WHEN c2 THEN b ...` | `np.select([c1,c2,...], [a,b,...], default)` |
| 条件聚合 | `SUM(CASE WHEN cond THEN col ELSE 0 END)` | `df[cond]['col'].sum()` 或 `.where()` |
| COALESCE | `COALESCE(col, 0)` | `df['col'].fillna(0)` |
| NULLIF | `NULLIF(col, 0)` | `df['col'].replace(0, np.nan)` |

```python
# np.where（二值）
df['label'] = np.where(df['score'] >= 60, 'pass', 'fail')

# np.select（多条件，类似 CASE WHEN）
conditions = [
    df['score'] >= 90,
    df['score'] >= 75,
    df['score'] >= 60
]
choices = ['A', 'B', 'C']
df['grade'] = np.select(conditions, choices, default='F')

# .where 和 .mask
df['col'].where(df['col'] > 0, other=0)   # 不满足条件的替换为 0
df['col'].mask(df['col'] < 0, other=0)    # 满足条件的替换为 0（与 where 相反）

# 条件聚合（类似 SQL CASE WHEN + SUM）
df[df['gender']=='F']['salary'].sum()
(df['gender']=='F').sum()                 # 计数满足条件的行数

# apply 实现复杂逻辑
def classify(row):
    if row['age'] < 18:
        return 'minor'
    elif row['income'] > 50000:
        return 'high_income_adult'
    else:
        return 'adult'
df['category'] = df.apply(classify, axis=1)
```

---

## 16. 透视与重塑

| 操作 | SQL | Pandas |
|------|-----|--------|
| 透视表 | `GROUP BY + CASE WHEN` | `pd.pivot_table(...)` |
| 宽转长 | — | `df.melt(...)` |
| 长转宽 | — | `df.pivot(...)` |

```python
# 透视表（类似 Excel 数据透视表）
pd.pivot_table(df,
    values='sales',
    index='region',
    columns='product',
    aggfunc='sum',
    fill_value=0,
    margins=True    # 加总行/列
)

# pivot（无聚合，要求无重复）
df.pivot(index='date', columns='product', values='sales')

# melt：宽转长（unpivot）
df.melt(id_vars=['id','name'],
        value_vars=['q1','q2','q3','q4'],
        var_name='quarter',
        value_name='sales')

# stack / unstack（多级索引互转）
df.stack()    # 列转行
df.unstack()  # 行转列

# crosstab（交叉频次表）
pd.crosstab(df['gender'], df['dept'])
pd.crosstab(df['gender'], df['dept'], values=df['salary'], aggfunc='mean')
```

---

## 17. NumPy 数组操作

```python
# 创建
np.array([1, 2, 3])
np.zeros((3, 4))              # 全 0
np.ones((2, 3))               # 全 1
np.full((2, 3), 7)            # 全 7
np.arange(0, 10, 2)           # [0,2,4,6,8]
np.linspace(0, 1, 5)          # 0~1 均匀 5 个点
np.eye(3)                     # 单位矩阵
np.random.seed(42)
np.random.rand(3, 4)          # 均匀分布 [0,1)
np.random.randn(3, 4)         # 标准正态
np.random.randint(0, 10, (3, 4))

# 属性
a.shape | a.ndim | a.dtype | a.size

# 索引
a[0] | a[-1] | a[1:4] | a[::2]
a[1, 2]       # 二维：行1列2
a[:, 1]       # 所有行的第 2 列
a[a > 5]      # 布尔索引

# 形状
a.reshape(2, 6)
a.flatten()
a.T                            # 转置
np.expand_dims(a, axis=0)      # 增加维度
np.squeeze(a)                  # 删除长度为 1 的维度

# 拼接
np.concatenate([a, b], axis=0)
np.vstack([a, b])  # 纵向
np.hstack([a, b])  # 横向

# 数学运算
np.sum(a, axis=0)  # 按列
np.sum(a, axis=1)  # 按行
np.mean(a) | np.std(a) | np.var(a)
np.min(a)  | np.max(a)
np.argmin(a) | np.argmax(a)   # 最小/最大值的位置
np.cumsum(a)
np.sort(a)
np.unique(a)
np.clip(a, 0, 1)              # 截断到 [0,1]
np.abs(a)
np.sqrt(a) | np.log(a) | np.exp(a)

# 广播
a = np.array([[1,2,3],[4,5,6]])  # (2,3)
b = np.array([10, 20, 30])       # (3,) 自动广播到每行
a + b

# 线性代数
np.dot(a, b)          # 矩阵乘法（也可 a @ b）
np.linalg.inv(a)      # 逆矩阵
np.linalg.det(a)      # 行列式
np.linalg.norm(a)     # 范数

# 与 Pandas 互转
arr = df['col'].to_numpy()
arr = df.values                # 整个 DataFrame 转 ndarray
df = pd.DataFrame(arr, columns=['a','b','c'])
```

---

## 18. 类型转换

| 操作 | SQL | Pandas |
|------|-----|--------|
| 转整数 | `CAST(col AS INT)` | `df['col'].astype(int)` |
| 转浮点 | `CAST(col AS FLOAT)` | `df['col'].astype(float)` |
| 转字符串 | `CAST(col AS VARCHAR)` | `df['col'].astype(str)` |
| 转日期 | `CAST(col AS DATE)` | `pd.to_datetime(df['col'])` |
| 转数字 | — | `pd.to_numeric(df['col'], errors='coerce')` |

```python
df['col'].astype(int)
df['col'].astype(float)
df['col'].astype(str)
df['col'].astype('category')             # 节省内存，适合低基数列

pd.to_numeric(df['col'], errors='coerce')   # 无法转换的变 NaN
pd.to_datetime(df['col'], errors='coerce')
pd.to_datetime(df['col'], format='%d/%m/%Y')

# 批量转换
df = df.astype({'col1': int, 'col2': float, 'col3': str})
```

---

## 19. 性能与调试技巧

```python
# 查看内存
df.memory_usage(deep=True).sum()

# 降低内存：低基数列转 category
df['status'] = df['status'].astype('category')

# 链式操作（避免多次复制）
df_result = (
    df.copy()
    .dropna(subset=['salary'])
    .query("dept == 'Engineering'")
    .assign(net_salary = lambda x: x['salary'] * 0.8) # 算出净工资
    .groupby('level')
    .agg(avg_net=('net_salary', 'mean'))              # 直接纯净聚合
    .reset_index()
    .sort_values('avg_net', ascending=False)
)

# 🛠️ 管道调试（使用 pipe 在链式调用中途拦截，打印中间形态维度）
def peek_shape(dataframe, stage_msg=''):
    print(f"[{stage_msg}] Shape: {dataframe.shape}")
    return dataframe

df_debug = df.pipe(peek_shape, '原始载入').query("salary > 0").pipe(peek_shape, '清洗过滤后')

# 实战例子
df_result = (
    df.copy()
    # 查看原始数据
    .pipe(peek_shape, '原始数据')

    # 删除空工资
    .dropna(subset=['salary'])
    .pipe(peek_shape, '删除空工资')

    # 删除负工资
    .query("salary > 0")
    .pipe(peek_shape, '删除异常工资')

    # 只看工程部
    .query("dept == 'Engineering'")
    .pipe(peek_shape, '筛选工程部')

    # 计算税后工资
    .assign(net_salary=lambda x: x['salary'] * 0.8)
    .pipe(peek_shape, '新增净工资')

    # 按级别统计平均净工资
    .groupby('level')
    .agg(avg_net_salary=('net_salary', 'mean'))

    .reset_index()
    .sort_values('avg_net_salary', ascending=False)
)

# ⚠️ 速度性能红线：坚决禁止写 for index, row in df.iterrows()！
# 优先进行矢量化直接求和（速度快千倍）；次选调用快速高效的 df.apply(lambda x: ..., axis=1)。

# ✅ 快：向量化
df['new'] = df['a'] + df['b']

# ✅ 次选：apply（比 iterrows 快，比向量化慢）
df['new'] = df.apply(lambda row: row['a'] + row['b'], axis=1)

# 实例：
result = (
    df.copy()
    .assign(                                      # 用asign新增三列net_salary，allowance，final_income
        net_salary=lambda x: x['salary'] * 0.8, 

        allowance=lambda x:
            x['age'].apply(
                lambda age: 500 if age >= 40 else 200   # 对新列allowance逐个apply函数
            ),

        final_income=lambda x:
            x['net_salary'] + x['allowance']
    )
    .sort_values(
        by='final_income',
        ascending=False
    )
)

# 检查重复
df.duplicated().sum()
df.duplicated(subset=['id']).sum()

# 检查数据一致性，语法：assert 条件, "报错信息"
assert df['id'].nunique() == len(df), "id 有重复"
assert df['score'].between(0, 100).all(), "score 存在超范围值" # 检查该列的值是否在闭区间(0,100)以内，是否所有的值都为True，如果不是的话，打印"score 存在超范围值"
```

