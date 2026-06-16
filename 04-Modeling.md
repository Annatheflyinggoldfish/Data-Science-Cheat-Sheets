# 🤖 Machine Learning Modeling & Optimization: Scikit-Learn 数据建模与优化速查表

## 完整流程速查

```
原始数据
   ↓ info() / describe() / corr() / value_counts() / boxplot / heatmap
EDA 探索（相关性与异方差初步诊断）
   ↓ fillna（均值/中位数/众数）/ clip（IQR截断）/ drop_duplicates / astype
数据清洗
   ↓ get_dummies(drop_first=True) / log1p / 特征交叉 / 删除共线特征
特征工程
   ↓ train_test_split(test_size=0.2, random_state=42, stratify=y)
   ↓ ⚠️ 此后所有 fit 只发生在 X_train 上
Train/Test 严格划分
   ↓
┌──────────────────────────┴──────────────────────────┐
↓ 连续值预测                                           ↓ 离散分类
Pipeline([Scaler, Poly, Ridge])              Pipeline([Scaler, LogisticRegression])
↓                                             ↓
GridSearchCV(alpha/degree, cv=5, r2)         GridSearchCV(C/class_weight, cv=5, roc_auc)
↓                                             ↓
R² / RMSE / MAE                              Confusion Matrix / F1 / ROC-AUC
残差图（检验异方差）                            学习曲线（检验过拟合）
└──────────────────────────┬──────────────────────────┘
   ↓ 检查 Data Leakage（异常高分？某特征相关性>0.99？）
最终金牌管道模型确定
   ↓ final_model.fit(X, y)（全量数据复训）
   ↓ joblib.dump(final_model, 'production_model.pkl')
持久化上线
```

---

## 目录

1. [环境准备](#1-环境准备)
2. [EDA 探索性分析](#2-eda-探索性分析)
3. [数据清洗](#3-数据清洗)
4. [特征工程](#4-特征工程)
5. [Train/Test Split](#5-traintest-split)
6. [🔀 分叉：回归 vs 分类](#6--分叉回归-vs-分类)
   - [6A. 回归工作流](#6a-回归工作流)
   - [6B. 分类工作流](#6b-分类工作流)
7. [交叉验证](#7-交叉验证)
8. [识别过拟合与欠拟合](#8-识别过拟合与欠拟合)
9. [正则化：Ridge / Lasso / LogisticRegression](#9-正则化ridge--lasso--logisticregression)
10. [GridSearchCV 调参](#10-gridsearchcv-调参)
11. [⚠️ Data Leakage 数据泄露](#11-️-data-leakage-数据泄露)
12. [最终模型与持久化](#12-最终模型与持久化)

---

## 1. 环境准备

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.linear_model import LinearRegression, Ridge, Lasso, LogisticRegression
from sklearn.metrics import (r2_score, mean_squared_error, mean_absolute_error,
                              confusion_matrix, classification_report,
                              roc_auc_score, roc_curve)
import joblib

# 显示设置
pd.set_option('display.max_columns', None)
pd.set_option('display.float_format', '{:.2f}'.format)
```

---

## 2. EDA 探索性分析

> **目的**：在动数据之前，先"看清楚"数据长什么样。
> 了解分布、缺失、异常、特征之间的关系。

### 2.1 基本概览

```python
df.shape          # (行数, 列数)，先知道数据规模
df.dtypes         # 每列的数据类型，判断哪些要转换
df.info()         # 类型 + 非空数量 + 内存占用，一次看清
df.describe()     # 数值列：count/mean/std/min/25%/50%/75%/max
                  # std 大说明数据波动大；min/max 反常说明有异常值
df.describe(include='all')    # 含非数值列（类别列的频次、唯一值数）
df.head()                     # 看前几行，了解数据格式
df['col'].value_counts()      # 类别列：每个值出现多少次
df['col'].value_counts(normalize=True)  # 占比（加起来=1）
df.isna().sum()               # 各列缺失数量，决定怎么处理
df.duplicated().sum()         # 重复行数量
```

### 2.2 相关性分析

```python
# 数值列之间的相关矩阵
# 值域 [-1, 1]：接近1=正相关，接近-1=负相关，接近0=无线性关系
df.corr()
df[['col1', 'col2', 'col3']].corr()  # 只看感兴趣的几列

# 相关性热力图例子
cols = ['Age_num', 'YearsCodePro', 'ConvertedCompYearly', 'JobSat']
corr = df[cols].corr()  # pandas默认用pairwise完整数据计算，每对变量各自dropna
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm')

df[cols].corr(method='pearson')   # 默认
df[cols].corr(method='spearman')  # Spearman
df[cols].corr(method='kendall')   # Kendall

# 精确计算：Pearson相关系数 + p值
# p值<0.05：这个相关性在统计上是显著的（不太可能是巧合）
from scipy import stats
coef, p_value = stats.pearsonr(df['col1'], df['col2'])
print(f'相关系数: {coef:.3f}, p值: {p_value:.4f}')

# Spearman相关系数 + p值
coef_s, p_value_s = stats.spearmanr(data['JobSat'], data['YearsCodePro'])
print(f'Spearman 相关系数: {coef_s:.3f}, p值: {p_value_s:.4f}')
```

| 相关系数绝对值范围 ($\|r\|$) | 关联强度 (Strength of Association) | 业务场景解读 |
| :--- | :--- | :--- |
| **0.00 ≤ \|r\| < 0.20** | **极弱相关 / 无相关** | 两者基本没关系。例如：程序员的头发长度与代码质量。 |
| **0.20 ≤ \|r\| < 0.40** | **弱相关** | 有一定苗头，但受到其他大量杂音干扰。例如：日常饮水量与工作满意度。 |
| **0.40 ≤ \|r\| < 0.60** | **中度相关** | 趋势明显，在业务中属于**非常有价值**的特征。例如：学历高低与全职就业倾向。 |
| **0.60 ≤ \|r\| < 0.80** | **强相关** | 关系非常紧密，通常可以作为核心业务预测指标。例如：工作年限与技术熟练度。 |
| **0.80 ≤ \|r\| ≤ 1.00** | **极强相关 / 完全相关** | 极其紧密。*注：如果高达 0.95 以上，需警惕两列数据是否含义重复（数据泄露）。* |

| P 值范围 ($p$) | 统计学结论 (Statistical Significance) | 行业标准称呼 | 报告常用星号 | 业务场景解读 |
| :--- | :--- | :--- | :--- | :--- |
| **p ≥ 0.05** | **不显著 (Not Significant)** | 无法拒绝原假设 | 无星号 (`ns`) | 差异大概率是偶然产生的。不能在报告里吹嘘这两个变量有关系。 |
| **p < 0.05** | **显著 (Significant)** | 达到行业接受边界 | `*` | **只有不到 5% 的概率是靠运气碰上的**。在社会科学、商业分析中，这已经足够证明两者存在真实相关。 |
| **p < 0.01** | **很显著 (Highly Significant)** | 结果非常过硬 | `**` | 只有不到 1% 的概率是巧合，结论非常可靠。 |
| **p < 0.001** | **极显著 (Extremely Significant)** | 铁证如山 | `***` | 只有不到千分之一的概率是巧合。在医学、硬科学或大数据样本中极常见。 |

### 2.3 可视化

```python
# --- 单变量分布 ---
df['col'].hist(bins=30)
# 若分布右偏（长尾在右），考虑后续做 log 变换

sns.boxplot(y=df['col'])
# 箱线图：直观看异常值（箱体外的点）

# --- 双变量关系 ---
plt.scatter(df['x_col'], df['y_col'])
# 散点图：看两变量是否有线性关系

sns.regplot(x='col1', y='col2', data=df)
# 比散点图多一条回归线，直观看趋势方向

sns.boxplot(x='category_col', y='numeric_col', data=df)
# 类别 vs 数值：看不同类别的数值分布差异

# --- 全局相关性热力图 ---
sns.heatmap(df.corr(), annot=True, fmt='.2f', cmap='coolwarm')
plt.title('Correlation Heatmap')
plt.tight_layout()
plt.show()
# 颜色越红=正相关越强，越蓝=负相关越强

# --- 分组聚合（常用于EDA） ---
df.groupby('category')['value'].mean()                 # 每组均值
df.groupby('category')['value'].agg(['mean','std','count'])  # 多统计量

# 透视表：行是col1的类别，列是col2的类别，格子里是value的均值
pivot = df.groupby(['col1', 'col2'])['value'].mean().unstack()
plt.pcolor(pivot, cmap='RdBu')
plt.colorbar()
plt.show()

# 对比实际值 vs 预测值分布（建模后用）
sns.kdeplot(y_test,  color='r', label='Actual')
sns.kdeplot(y_hat,   color='b', label='Predicted')
plt.legend()
```

---

## 3. 数据清洗

> **目的**：处理缺失值、异常值、重复行、类型错误。
> 原则：**先探索，再清洗**，别还没看清楚就开始改数据。

### 3.1 缺失值

```python
df.isna().sum()                    # 各列缺失数量
df.isna().sum() / len(df)          # 缺失率（超过50%的列考虑直接删）

# --- 删除 ---
df.dropna(inplace=True)                     # 删除所有含缺失的行（慎用，会丢数据）
df.dropna(subset=['col1', 'col2'])          # 只有这两列有缺失才删
df.dropna(thresh=3)                         # 保留至少有3个非空值的行

# --- 填充 ---
df['col'].fillna(df['col'].mean())          # 均值填充（适合正态分布的数值列）
df['col'].fillna(df['col'].median())        # 中位数填充（有异常值时更稳健）
df['col'].fillna(df['col'].mode()[0])       # 众数填充（适合类别列）
df['col'].fillna(method='ffill')            # 前向填充（适合时序数据，用上一行值填）
df['col'].fillna(method='bfill')            # 后向填充（用下一行值填）
df.fillna({'col1': 0, 'col2': 'unknown'})   # 各列分别指定填充值
```

### 3.2 异常值

```python
# IQR 法则（最常用）
# IQR = 第75百分位 - 第25百分位，即"中间50%数据的范围"
# 超出 [Q1-1.5*IQR, Q3+1.5*IQR] 的被视为异常值
Q1  = df['col'].quantile(0.25)
Q3  = df['col'].quantile(0.75)
IQR = Q3 - Q1
lower, upper = Q1 - 1.5 * IQR, Q3 + 1.5 * IQR

df = df[(df['col'] >= lower) & (df['col'] <= upper)]  # 删除异常值
df['col'] = df['col'].clip(lower, upper)              # 截断（更保守，不删行）
# clip 更推荐：既处理了极端值，又不丢失这行其他列的信息

# Z-score 法则（适合近似正态分布）
# Z-score = (值 - 均值) / 标准差，绝对值>3说明偏离均值3个标准差
from scipy import stats
mask = np.abs(stats.zscore(df[['col1', 'col2']])) < 3
df = df[mask.all(axis=1)]
```

### 3.3 重复值与类型

```python
df.duplicated().sum()
df.drop_duplicates(inplace=True)
df.drop_duplicates(subset=['id'], keep='first')  # 按 id 去重，保留第一条

df['col'].astype(int) | astype(float) | astype(str)
df['col'] = pd.to_numeric(df['col'], errors='coerce')   # 无法转换的变NaN
df['date'] = pd.to_datetime(df['date'])
df.columns = df.columns.str.lower().str.replace(' ', '_')  # 列名规范化
```

---

## 4. 特征工程

> **目的**：把原始数据变成模型"能看懂"的格式，并创造有预测力的新特征。
> 特征工程是数据分析师和机器学习工程师最核心的技能之一。

### 4.1 编码类别变量

```python
# One-Hot Encoding（最常用，适合无序类别：城市、颜色、品牌）
# 把一列变成多列0/1，每列代表一个类别
df = pd.get_dummies(df, columns=['city', 'gender'], drop_first=True)
# drop_first=True：删掉第一个哑变量
# 原因：3个城市只需2列就能表达，保留3列会造成"多重共线性"（信息冗余）

# Label Encoding（适合有序类别：如评级low/mid/high）
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
df['rating'] = le.fit_transform(df['rating'])
# 注意：LabelEncoder 会给类别分配0,1,2...但顺序是按字母，不是按大小
# 如果顺序重要，用下面的 OrdinalEncoder

# Ordinal Encoding（适合有明确顺序的类别）
from sklearn.preprocessing import OrdinalEncoder
oe = OrdinalEncoder(categories=[['low', 'medium', 'high']])
df[['level']] = oe.fit_transform(df[['level']])
# 结果：low=0, medium=1, high=2，保留了大小关系
```

### 4.2 数值缩放

```python
# ⚠️ 重要原则：scaler 只在训练集上 fit，再 transform 测试集
# 如果在全量数据上 fit，测试集的分布信息就"泄露"给了训练过程

# StandardScaler：标准化，转换成均值0、标准差1
# 适合：线性回归、Ridge、Lasso、Logistic Regression、SVM
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform
X_test_scaled  = scaler.transform(X_test)        # 只 transform，不 fit！

# MinMaxScaler：归一化到 [0,1]
# 适合：神经网络、KNN（对距离敏感的算法）
from sklearn.preprocessing import MinMaxScaler
mm = MinMaxScaler()
X_train_scaled = mm.fit_transform(X_train)
X_test_scaled  = mm.transform(X_test)
```

### 4.3 多项式特征

```python
# 为什么需要多项式特征？
# 线性模型只能拟合直线，但现实中很多关系是曲线的（如面积和价格）
# 加入 x², x³ 或 x1*x2 这样的交叉项，让线性模型能拟合非线性关系

# 单变量多项式（numpy）
f = np.polyfit(x, y, n)   # n=1是直线，n=2是抛物线，n=3是三次曲线
p = np.poly1d(f)           # 生成多项式模型对象
y_hat = p(x)               # 预测

# 多变量多项式特征（sklearn，更常用）
from sklearn.preprocessing import PolynomialFeatures
pr = PolynomialFeatures(degree=2, include_bias=False)
X_poly = pr.fit_transform(X)
# degree=2 时：原本有 x1, x2 两列
# 变换后：x1, x2, x1², x1·x2, x2²（5列）
# 特征数量会急剧增加，degree越高越容易过拟合，通常不超过3
```

### 4.4 目标变量变换（回归专用）

```python
# 当目标变量分布右偏（大多数值小，少数值极大，如收入、房价），
# 直接用原始值建模效果差，log 变换让分布更接近正态

y_log = np.log1p(df['price'])   # log1p = log(1+x)，避免 log(0) 报错
# 建模时用 y_log 作为目标
# 预测后记得反变换：y_pred_original = np.expm1(y_pred_log)
```

### 4.5 构造新特征

```python
df['age']              = 2024 - df['birth_year']
df['revenue_per_user'] = df['revenue'] / df['users']   # 比率特征
df['is_weekend']       = df['date'].dt.dayofweek >= 5  # 布尔特征
df['month']            = df['date'].dt.month
df['log_price']        = np.log1p(df['price'])          # log 变换
df['col1_x_col2']      = df['col1'] * df['col2']        # 特征交叉（手动）
```

### 4.6 特征选择

```python
# 删除与目标变量相关性太低的特征
corr_with_target = df.corr()['target'].abs().sort_values(ascending=False)
print(corr_with_target)
# 相关性<0.1的特征一般贡献不大（但不是绝对，非线性关系时相关性可能失效）

# 删除特征之间高度共线的列（互相关>0.9时保留一个即可）
corr_matrix = df.drop('target', axis=1).corr().abs()
upper = corr_matrix.where(np.triu(np.ones(corr_matrix.shape), k=1).astype(bool))
to_drop = [col for col in upper.columns if any(upper[col] > 0.9)]
df.drop(columns=to_drop, inplace=True)
```

---

## 5. Train/Test Split

> **核心原则**：测试集是"考试卷"，绝对不能在训练过程中看到。
> 所有的 fit 操作只能发生在训练集上。

```python
from sklearn.model_selection import train_test_split

y = df['target']
X = df.drop('target', axis=1)

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,      # 20%作为测试集，80%训练集（常用比例）
    random_state=42,    # 固定随机种子，保证每次运行结果一样（便于复现）
    stratify=y          # ⚠️ 分类任务加这行：保证训练集和测试集中各类别比例相同
                        # 例如原数据90%负例10%正例，split后两个子集都保持这个比例
                        # 回归任务（y是连续值）不能用 stratify=y，会报错
                        # 回归时可以用分箱后的y来stratify：
                        # stratify=pd.cut(y, bins=5)
)

print(f'训练集: {X_train.shape}, 测试集: {X_test.shape}')

# 分类任务：检查类别比例是否保持
print(y_train.value_counts(normalize=True))
print(y_test.value_counts(normalize=True))
```

---

## 6. 🔀 分叉：回归 vs 分类

---

### 6A. 回归工作流

> **适用场景**：预测一个连续数值。
> 例：预测房价、预测销售额、预测用户消费金额。

#### 线性回归（基础）

```python
from sklearn.linear_model import LinearRegression

lr = LinearRegression()
lr.fit(X_train, y_train)
y_hat = lr.predict(X_test)

print('截距 (intercept):', lr.intercept_)
# 截距=当所有特征为0时，预测值是多少

print('系数 (coefficients):', lr.coef_)
# 每个特征对应的斜率：系数=2说明该特征每增加1，预测值增加2
# 系数为负说明该特征与目标变量负相关
```

#### 回归评估指标

```python
from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error

r2   = r2_score(y_test, y_hat)
mse  = mean_squared_error(y_test, y_hat)
rmse = np.sqrt(mse)
mae  = mean_absolute_error(y_test, y_hat)

print(f'R²:   {r2:.4f}')   # 越接近1越好，0.7以上通常认为不错
print(f'RMSE: {rmse:.2f}')  # 单位与y相同，越小越好，可解释性强
print(f'MAE:  {mae:.2f}')   # 对异常值不敏感，越小越好
```

| 指标 | 含义 | 越高/低越好 | 直觉解释 |
|------|------|------------|---------|
| R² | 模型解释了目标变量多少比例的方差 | 越高越好（最大1）| R²=0.8，模型解释了80%的价格波动 |
| RMSE | 预测误差的均方根 | 越低越好 | RMSE=5000，平均预测误差约5000元 |
| MAE | 预测误差的绝对值均值 | 越低越好 | 比RMSE对异常值更鲁棒 |

```python
# 残差图：检验模型假设
sns.residplot(x=y_hat, y=y_test - y_hat)
plt.axhline(0, color='red', linestyle='--')
plt.xlabel('Fitted Values')
plt.ylabel('Residuals')
plt.title('Residual Plot')
# ✅ 理想：残差随机分布在0附近，无规律
# ❌ 漏斗形（方差不均匀）→ 异方差，考虑对y做log变换
# ❌ 有明显弯曲 → 线性假设不成立，考虑加多项式特征
```

#### 回归Pipeline（推荐写法）

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.linear_model import Ridge

# Pipeline把预处理和模型串联成一个对象
# 好处1：代码整洁
# 好处2：自动防止数据泄露（fit时只用训练集信息）
# 好处3：可以直接丢进GridSearchCV

pipe_reg = Pipeline([
    ('scaler', StandardScaler()),                           # 第一步：标准化
    ('poly',   PolynomialFeatures(degree=2,                 # 第二步：生成多项式特征
                                  include_bias=False)),
    ('model',  Ridge(alpha=1))                              # 第三步：带正则化的线性回归
])

pipe_reg.fit(X_train, y_train)
y_hat = pipe_reg.predict(X_test)
print('Pipeline R²:', r2_score(y_test, y_hat))
```

---

### 6B. 分类工作流

> **适用场景**：预测一个离散类别。
> 例：客户是否流失（是/否）、邮件是否垃圾邮件、信用评级（A/B/C）。

#### Logistic 回归（基础分类模型）

```python
# 虽然名字叫"回归"，但实际上是分类模型
# 它预测的是"属于某个类别的概率"（0到1之间）
# 然后用阈值（默认0.5）决定最终类别

from sklearn.linear_model import LogisticRegression

lr_clf = LogisticRegression(
    C=1.0,               # 正则化强度的倒数：C越小正则越强（防过拟合）
    class_weight=None,   # 样本不均衡时改为 'balanced'
    max_iter=1000,       # 最大迭代次数，收敛慢时调大
    random_state=42
)
lr_clf.fit(X_train, y_train)

y_pred       = lr_clf.predict(X_test)          # 预测类别（0或1）
y_pred_proba = lr_clf.predict_proba(X_test)[:, 1]  # 预测为正类的概率
# [:, 1] 表示取第二列（正类概率）；第一列是负类概率
```

#### 分类评估指标

```python
from sklearn.metrics import (confusion_matrix, classification_report,
                              roc_auc_score, roc_curve)

# --- 混淆矩阵 ---
cm = confusion_matrix(y_test, y_pred)
print(cm)
# 输出格式：
#            预测负  预测正
# 实际负  [[  TN    FP  ]]
# 实际正  [[  FN    TP  ]]
# TN=真负例（预测没流失，实际没流失）✅
# TP=真正例（预测流失，实际流失）✅
# FP=假正例（预测流失，实际没流失）❌ 误报
# FN=假负例（预测没流失，实际流失）❌ 漏报

sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.xlabel('Predicted') | plt.ylabel('Actual')
plt.show()

# --- 分类报告（一次看所有指标）---
print(classification_report(y_test, y_pred))
# 输出包含：precision / recall / f1-score / support

# --- 各指标含义 ---
# Accuracy  = (TP+TN) / 总数        → 整体正确率（样本均衡时用）
# Precision = TP / (TP+FP)          → 预测为正的里面，真正是正的比例（精确率）
#             "你说是流失的，有多少真的流失了"
# Recall    = TP / (TP+FN)          → 实际为正的里面，被预测出来的比例（召回率）
#             "真正流失的客户，你找出来了多少"
# F1-Score  = 2 * P*R / (P+R)       → Precision和Recall的调和均值，两者兼顾
# Support   = 该类别的实际样本数

# --- ROC-AUC ---
auc = roc_auc_score(y_test, y_pred_proba)
print(f'AUC: {auc:.4f}')
# AUC = ROC曲线下面积：越接近1越好，0.5等于随机猜，<0.5说明模型有问题
# AUC不受类别不均衡影响，比Accuracy更可靠

# ROC曲线（可视化）
fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)
plt.plot(fpr, tpr, label=f'AUC = {auc:.2f}')
plt.plot([0,1], [0,1], 'k--', label='Random (AUC=0.5)')
plt.xlabel('False Positive Rate (FPR)')   # 误报率
plt.ylabel('True Positive Rate (TPR)')    # 召回率
plt.title('ROC Curve')
plt.legend()
plt.show()
```

#### 处理样本不均衡

```python
# 现实中分类数据经常不均衡，如欺诈检测（99%正常，1%欺诈）
# 直接训练会导致模型偷懒：全部预测"正常"，准确率99%但完全没用

# 方法1：class_weight='balanced'（最简单，推荐先试这个）
lr_clf = LogisticRegression(class_weight='balanced')
# sklearn自动根据类别频次调整权重：少数类权重更大

# 方法2：调整决策阈值（默认0.5，可以降低来提高召回率）
threshold = 0.3
y_pred_adjusted = (y_pred_proba >= threshold).astype(int)
# 降低阈值：更容易预测为正类 → 召回率上升，精确率下降

# 方法3：过采样少数类（SMOTE）
from imblearn.over_sampling import SMOTE
sm = SMOTE(random_state=42)
X_res, y_res = sm.fit_resample(X_train, y_train)
# SMOTE合成新的少数类样本，而不是简单复制
```

#### 分类Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipe_clf = Pipeline([
    ('scaler', StandardScaler()),
    ('model',  LogisticRegression(C=1.0, class_weight='balanced',
                                  max_iter=1000, random_state=42))
])

pipe_clf.fit(X_train, y_train)
y_pred       = pipe_clf.predict(X_test)
y_pred_proba = pipe_clf.predict_proba(X_test)[:, 1]
```

---

## 7. 交叉验证

> **为什么需要交叉验证？**
> 单次 train/test split 结果受随机分割影响很大。
> 交叉验证把数据分成K份，轮流用每份作测试集，得到K个评分，
> 取均值和标准差，结果更可靠，也能看出模型是否稳定。

```python
from sklearn.model_selection import cross_val_score, cross_val_predict

# --- 回归：用 R² 评分 ---
scores = cross_val_score(pipe_reg, X, y, cv=5, scoring='r2')
print('各折 R²:', scores)
print(f'均值: {scores.mean():.4f} ± {scores.std():.4f}')
# 标准差大 → 模型对数据划分敏感，不够稳定

# --- 分类：用 F1 或 AUC 评分 ---
f1_scores  = cross_val_score(pipe_clf, X, y, cv=5, scoring='f1')
auc_scores = cross_val_score(pipe_clf, X, y, cv=5, scoring='roc_auc')
print(f'F1:  {f1_scores.mean():.4f} ± {f1_scores.std():.4f}')
print(f'AUC: {auc_scores.mean():.4f} ± {auc_scores.std():.4f}')

# --- 交叉验证预测（返回每条数据在验证时的预测值）---
y_hat_cv = cross_val_predict(pipe_reg, X, y, cv=5)
# 可以用这个结果算整体R²：r2_score(y, y_hat_cv)

# --- MSE（注意sklearn返回负值，需要取负）---
mse_scores = -cross_val_score(pipe_reg, X, y, cv=5,
                               scoring='neg_mean_squared_error')
rmse_scores = np.sqrt(mse_scores)
print(f'RMSE: {rmse_scores.mean():.4f}')
```

---

## 8. 识别过拟合与欠拟合

> **过拟合**：模型把训练集的噪声也学进去了，在训练集表现很好，
>             但在新数据（测试集）上表现差。像死记答案，换一套题就不行了。
>
> **欠拟合**：模型太简单，连训练集的规律都没学到，
>             训练集和测试集表现都差。

```python
# 最直接的诊断：对比训练集和测试集的分数
r2_train = pipe_reg.score(X_train, y_train)
r2_test  = pipe_reg.score(X_test,  y_test)
print(f'Train R²: {r2_train:.4f}')
print(f'Test  R²: {r2_test:.4f}')
```

| 情况 | Train 分数 | Test 分数 | 诊断 | 解决方向 |
|------|------------|-----------|------|---------|
| 过拟合 | 高（如0.95）| 明显低（如0.60）| 模型记住了噪声 | 正则化↑、减少特征、降低多项式阶数 |
| 欠拟合 | 低（如0.40）| 低（如0.38）| 模型太简单 | 加特征、提高阶数、换更复杂模型 |
| 正常  | 高 | 接近训练集 | — | — |

```python
# 学习曲线：看训练集大小对性能的影响
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    pipe_reg, X, y, cv=5, scoring='r2',
    train_sizes=np.linspace(0.1, 1.0, 10)
)

plt.plot(train_sizes, train_scores.mean(axis=1), label='Train R²')
plt.plot(train_sizes, val_scores.mean(axis=1),   label='Val R²')
plt.fill_between(train_sizes,
                 train_scores.mean(axis=1) - train_scores.std(axis=1),
                 train_scores.mean(axis=1) + train_scores.std(axis=1),
                 alpha=0.1)
plt.xlabel('Training Size')
plt.ylabel('R²')
plt.legend()
# 两线差距很大 → 过拟合
# 两线都低且平 → 欠拟合
# 两线都高且接近 → 正常

# 多项式阶数 vs 性能（找最优复杂度）
r2_train_list, r2_test_list = [], []
for d in range(1, 8):
    pr = PolynomialFeatures(degree=d, include_bias=False)
    X_tr = pr.fit_transform(X_train)
    X_te = pr.transform(X_test)          # ⚠️ transform 不 fit
    lr = LinearRegression().fit(X_tr, y_train)
    r2_train_list.append(lr.score(X_tr, y_train))
    r2_test_list.append(lr.score(X_te, y_test))

plt.plot(range(1,8), r2_train_list, label='Train')
plt.plot(range(1,8), r2_test_list,  label='Test')
plt.xlabel('Polynomial Degree')
plt.ylabel('R²')
plt.legend()
# Test曲线先升后降 → 最高点是最优阶数
```

---

## 9. 正则化：Ridge / Lasso / LogisticRegression

> **为什么需要正则化？**
> 模型系数太大时，对输入数据的小变化极其敏感 → 过拟合。
> 正则化在损失函数里加一个"惩罚项"，限制系数不能太大，强迫模型更简单。

| | Ridge（L2） | Lasso（L1） | LogisticRegression |
|--|-------------|-------------|-------------------|
| 惩罚项 | 系数**平方**和 | 系数**绝对值**和 | L2（默认）或L1 |
| 效果 | 系数收缩但不为0 | 部分系数**变为0**（自动特征选择）| 同Ridge/Lasso |
| 适用 | 多数特征都有贡献 | 特征多但真正有用的少 | 分类任务 |
| 参数 | `alpha`（越大正则越强）| `alpha`（越大正则越强）| `C`（越**小**正则越强，与alpha相反）|

```python
from sklearn.linear_model import Ridge, Lasso

# --- Ridge 回归 ---
ridge = Ridge(alpha=1.0)   # alpha=0等同于普通线性回归，alpha越大越保守
ridge.fit(X_train_poly, y_train)
y_hat = ridge.predict(X_test_poly)
print('Ridge R²:', ridge.score(X_test_poly, y_test))

# --- Lasso 回归 ---
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_poly, y_train)
y_hat = lasso.predict(X_test_poly)

# Lasso的特别之处：会把不重要的特征系数变成精确的0
n_zero = np.sum(lasso.coef_ == 0)
n_total = len(lasso.coef_)
print(f'Lasso 置零了 {n_zero}/{n_total} 个特征（即被筛掉的特征）')

# --- Logistic Regression 的正则化 ---
# C 是正则强度的"倒数"：C=0.1 正则很强，C=100 正则很弱
lr_clf = LogisticRegression(C=1.0, penalty='l2')  # penalty='l1' 也可以
# l2（默认）= Ridge效果，l1 = Lasso效果（稀疏化）

# --- Pipeline 写法（推荐）---
pipe_ridge = Pipeline([
    ('poly',   PolynomialFeatures(degree=2, include_bias=False)),
    ('scaler', StandardScaler()),
    ('model',  Ridge(alpha=1))
])
pipe_ridge.fit(X_train, y_train)
```

---

## 10. GridSearchCV 调参

> **目的**：自动找到最优超参数组合。
> 超参数（如 alpha、degree）不是模型从数据里学到的，而是我们事先设定的。
> GridSearchCV 把所有参数组合都试一遍，用交叉验证评分，找出最好的那个。

```python
from sklearn.model_selection import GridSearchCV

# --- 回归：Ridge + 多项式 ---
pipe_reg = Pipeline([
    ('poly',   PolynomialFeatures(include_bias=False)),
    ('scaler', StandardScaler()),
    ('model',  Ridge())
])

# Pipeline中参数格式：步骤名__参数名（两个下划线）
param_grid_reg = {
    'poly__degree':  [1, 2, 3],          # 尝试3种多项式阶数
    'model__alpha':  [0.01, 0.1, 1, 10, 100]  # 尝试5种正则强度
}
# 总共 3×5=15 种组合，每种做5折交叉验证，共训练75次

grid_reg = GridSearchCV(
    pipe_reg, param_grid_reg,
    cv=5,               # 5折交叉验证
    scoring='r2',       # 用R²评分（回归）
    refit=True,         # 找到最优参数后，用全量训练集重新训练一次（默认True）
    n_jobs=-1           # 并行计算，用所有CPU核心
)
grid_reg.fit(X_train, y_train)

print('最优参数:', grid_reg.best_params_)
print('最优CV R²:', grid_reg.best_score_)
print('测试集R²:', grid_reg.score(X_test, y_test))

# --- 分类：Logistic Regression ---
pipe_clf = Pipeline([
    ('scaler', StandardScaler()),
    ('model',  LogisticRegression(max_iter=1000, random_state=42))
])

param_grid_clf = {
    'model__C':            [0.01, 0.1, 1, 10, 100],
    'model__class_weight': [None, 'balanced']  # 是否处理样本不均衡
}

grid_clf = GridSearchCV(
    pipe_clf, param_grid_clf,
    cv=5,
    scoring='roc_auc',  # 分类用AUC（样本不均衡时比accuracy更可靠）
    refit=True,
    n_jobs=-1
)
grid_clf.fit(X_train, y_train)

print('最优参数:', grid_clf.best_params_)
print('测试集AUC:', roc_auc_score(y_test,
      grid_clf.predict_proba(X_test)[:, 1]))

# --- 查看所有参数组合的结果 ---
results = pd.DataFrame(grid_reg.cv_results_)
results[['param_poly__degree', 'param_model__alpha',
         'mean_test_score', 'std_test_score']].sort_values('mean_test_score', ascending=False)

# --- RandomizedSearchCV（参数空间很大时更高效）---
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import loguniform, randint

param_dist = {
    'poly__degree':  randint(1, 5),              # 从1到4随机整数
    'model__alpha':  loguniform(1e-3, 1e3)       # 对数均匀分布
}

random_search = RandomizedSearchCV(
    pipe_reg, param_dist,
    n_iter=30,          # 只随机试30种组合，比GridSearch快
    cv=5, scoring='r2', random_state=42, n_jobs=-1
)
random_search.fit(X_train, y_train)
print('RandomizedSearch最优参数:', random_search.best_params_)
```

---

## 11. ⚠️ Data Leakage 数据泄露

> **数据泄露**：训练过程中"看到了"不该看到的信息（测试集或未来数据），
> 导致评估分数虚高，但实际上线后表现很差。
> 这是机器学习中最常见、最难察觉的错误之一。

### 三大泄露来源

```python
# ❌ 泄露1：在全量数据上 fit，再 split
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)      # 错！测试集的分布信息已经进入了scaler
X_train, X_test = train_test_split(X_scaled, ...)

# ✅ 正确：先 split，再在训练集上 fit
X_train, X_test, y_train, y_test = train_test_split(X, y, ...)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # 只在训练集上fit
X_test_scaled  = scaler.transform(X_test)       # 测试集只transform

# ❌ 泄露2：用全量数据的均值填充缺失值
df['col'].fillna(df['col'].mean())   # 错！测试集的均值也参与了
# ✅ 正确：用训练集的均值填充
train_mean = X_train['col'].mean()
X_train['col'].fillna(train_mean)
X_test['col'].fillna(train_mean)     # 测试集也用训练集的均值

# ❌ 泄露3：特征中包含目标变量的衍生信息
# 如预测明天的销售额时，特征中包含了"明天是否促销"（未来信息）
```

### Pipeline 是防泄露的最佳实践

```python
# Pipeline 在 fit 时，所有步骤都只见到训练集
# 在 predict 时，所有步骤都只做 transform（不重新 fit）
# 从结构上杜绝了泄露可能

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model',  Ridge())
])
pipe.fit(X_train, y_train)    # scaler只在训练集上fit
pipe.predict(X_test)           # scaler用训练集的参数transform测试集
```

### 泄露的信号

```python
# 如果出现以下情况，高度怀疑数据泄露：
# 1. 模型分数异常高（如R²=0.99，AUC=1.0）
# 2. 训练集和测试集分数几乎完全相同
# 3. 某个特征与目标变量相关性极高（接近1）

# 检查特征相关性
corr_with_target = df.corr()['target'].abs().sort_values(ascending=False)
print(corr_with_target.head(10))
# 如果某特征相关性>0.99，很可能是从target衍生出来的，需要检查
```

---

## 12. 最终模型与持久化

```python
# 1. 确认最优参数
best_params = grid_reg.best_params_
print('最终使用参数:', best_params)

# 2. 在全量数据上重新训练（训练集+测试集合并）
# 原因：之前分出来测试集是为了评估，现在参数确定了，
# 用更多数据训练能让模型更好
final_model = Pipeline([
    ('poly',   PolynomialFeatures(degree=best_params['poly__degree'],
                                  include_bias=False)),
    ('scaler', StandardScaler()),
    ('model',  Ridge(alpha=best_params['model__alpha']))
])
final_model.fit(X, y)   # 全量数据

# 3. 最终评估报告
y_final_pred = final_model.predict(X_test)   # 用之前留出的测试集最终评估

print('=== 最终模型评估（回归）===')
print(f'R²:   {r2_score(y_test, y_final_pred):.4f}')
print(f'RMSE: {np.sqrt(mean_squared_error(y_test, y_final_pred)):.4f}')
print(f'MAE:  {mean_absolute_error(y_test, y_final_pred):.4f}')

# 分类任务的最终评估
# y_final_proba = final_clf.predict_proba(X_test)[:, 1]
# print(classification_report(y_test, final_clf.predict(X_test)))
# print(f'AUC: {roc_auc_score(y_test, y_final_proba):.4f}')

# 4. 分布图：实际值 vs 预测值
sns.kdeplot(y_test,        color='r', label='Actual')
sns.kdeplot(y_final_pred,  color='b', label='Predicted')
plt.legend()
plt.title('Actual vs Predicted Distribution')
plt.show()

# 5. 保存模型
import joblib
joblib.dump(final_model, 'production_model.pkl')
print('模型已保存')

# 6. 加载并使用
loaded_model = joblib.load('production_model.pkl')
y_new = loaded_model.predict(X_new)   # X_new是新来的数据，无需再做缩放（Pipeline会处理）
```
