# 🤖 Machine Learning Modeling & Optimization: Scikit-Learn 数据建模与优化速查表


## 工作表

                    原始数据
                       ↓ df.info() / describe() / corr()
                    EDA 探索 (相关性与异方差初步诊断)
                       ↓ fillna / clip (IQR异常值截断) / astype
                    数据清洗
                       ↓ get_dummies / 特征交叉构造 / Target Log变换
                    特征工程
                       ↓ train_test_split(stratify=y 保持阶层比例)
                  Train/Test 严格划分
                       ↓
         ┌─────────────┴─────────────┐
         ↓ (如果是连续值预测)          ↓ (如果是离散分类任务)
   [回归工作流]                  [分类工作流]
   Pipeline([Scaler, Poly, Ridge])  Pipeline([Scaler, LogisticRegression])
         ↓                             ↓
   GridSearchCV 多维穷举调参       GridSearchCV 样本均衡优化 (C / class_weight)
         ↓                             ↓
   评估指标: R² / RMSE / MAE       评估指标: Confusion Matrix / F1-Score / ROC-AUC
         └─────────────┬─────────────┘
                       ↓ 检查 Data Leakage 泄露红线
                 最终金牌管道模型确定
                       ↓ final_model.fit(X, y) 在全盘上复训
                 持久化上线: joblib.dump(final_model, 'production_model.pkl')

---

## 目录

1. [EDA 探索性分析](#1-eda-探索性分析)
2. [数据清洗](#2-数据清洗)
3. [特征工程](#3-特征工程)
4. [Train/Test Split](#4-traintest-split)
5. [模型训练](#5-模型训练)
6. [交叉验证](#6-交叉验证)
7. [模型评估：R² / MSE](#7-模型评估r²--mse)
8. [识别过拟合与欠拟合](#8-识别过拟合与欠拟合)
9. [正则化：Ridge & Lasso](#9-正则化ridge--lasso)
10. [GridSearchCV 调参](#10-gridsearchcv-调参)
11. [最终模型与输出](#11-最终模型与输出)

---

## 1. EDA 探索性分析

### 基本概览

```python
df.shape                          # 行列数
df.dtypes                         # 各列类型
df.info()                         # 类型 + 非空数量 + 内存
df.describe()                     # 数值列统计摘要（count/mean/std/min/25%/50%/75%/max）
df.describe(include='all')        # 含非数值列
df.head()
df['col'].value_counts()          # 类别列频次
df['col'].value_counts(normalize=True)  # 频率（占比）
df.isna().sum()                   # 各列缺失数量
df.duplicated().sum()             # 重复行数量
```

### 相关性分析

```python
# 全列相关矩阵
df.corr()

# 指定列的相关矩阵
df[['col1', 'col2', 'col3']].corr()

# Pearson 相关系数 + p 值
from scipy import stats
pearson_coef, p_value = stats.pearsonr(df['col1'], df['col2'])
# pearson_coef 接近 1/-1 = 强相关；接近 0 = 弱相关
# p_value < 0.05 = 统计显著
```

### 可视化

```python
import matplotlib.pyplot as plt
import seaborn as sns

# 散点图
plt.scatter(df['x'], df['y'])
plt.xlabel('x') | plt.ylabel('y') | plt.title('title')
plt.show()

# 回归图（散点 + 回归线）
sns.regplot(x='col1', y='col2', data=df)

# 箱线图（查看分布与异常值）
sns.boxplot(x='category_col', y='numeric_col', data=df)

# 分布图（实际值 vs 预测值对比常用）
sns.kdeplot(df['col'], color='r', label='Actual')
sns.kdeplot(y_hat, color='b', label='Fitted')
plt.legend()

# 热力图（相关矩阵可视化）
from matplotlib import pyplot as plt
grouped_pivot = df.groupby(['col1', 'col2'])['value'].mean().unstack()
plt.pcolor(grouped_pivot, cmap='RdBu')
plt.colorbar()
plt.show()

# seaborn 热力图（更常用）
sns.heatmap(df.corr(), annot=True, fmt='.2f', cmap='coolwarm')

# 直方图
df['col'].hist(bins=30)

# 分组对比
df.groupby('category')['value'].mean().plot(kind='bar')
```

### 分组聚合（EDA 常用模式）

```python
# 单列分组
df.groupby('col1', as_index=False).mean()

# 多列分组
df.groupby(['col1', 'col2'], as_index=False).mean()

# 多列、多聚合方式
df.groupby(['col1']).agg({
    'price': 'mean',
    'horsepower': 'max',
    'engine-size': 'count'
})

# 透视表
pivot = df.groupby(['col1', 'col2'])['value'].mean().unstack()
# 等价写法
pd.pivot_table(df, values='value', index='col1', columns='col2', aggfunc='mean', fill_value=0)
```

---

## 2. 数据清洗

### 缺失值

```python
df.isna().sum()                          # 统计各列缺失
df.isna().sum() / len(df)                # 缺失率

# 删除
df.dropna(inplace=True)                  # 删除含空值的行
df.dropna(subset=['col1', 'col2'])       # 只检查指定列
df.dropna(thresh=3)                      # 保留非空值 ≥ 3 的行

# 填充
df['col'].fillna(df['col'].mean())       # 均值填充（数值列）
df['col'].fillna(df['col'].median())     # 中位数填充（有异常值时更稳健）
df['col'].fillna(df['col'].mode()[0])    # 众数填充（类别列）
df['col'].fillna(method='ffill')         # 前向填充（时序数据）
df['col'].fillna(method='bfill')         # 后向填充
df.fillna({'col1': 0, 'col2': 'unknown'})
```

### 重复值

```python
df.duplicated().sum()
df.drop_duplicates(inplace=True)
df.drop_duplicates(subset=['id'], keep='first')
```

### 异常值

```python
# IQR 法则
Q1 = df['col'].quantile(0.25)
Q3 = df['col'].quantile(0.75)
IQR = Q3 - Q1
df = df[(df['col'] >= Q1 - 1.5 * IQR) & (df['col'] <= Q3 + 1.5 * IQR)]

# Z-score 法则
from scipy import stats
df = df[(np.abs(stats.zscore(df['col'])) < 3)]

# 截断而非删除（更保守）
df['col'] = df['col'].clip(lower=Q1 - 1.5*IQR, upper=Q3 + 1.5*IQR)
```

### 类型转换

```python
df['col'].astype(int) | astype(float) | astype(str)
df['col'] = pd.to_numeric(df['col'], errors='coerce')   # 转换失败变 NaN
df['date'] = pd.to_datetime(df['date'])
df['category'] = df['category'].astype('category')      # 节省内存
```

### 列名规范化

```python
df.columns = df.columns.str.lower().str.replace(' ', '_').str.strip()
df.rename(columns={'old_name': 'new_name'}, inplace=True)
```

---

## 3. 特征工程

### 编码类别变量

```python
# One-Hot Encoding（适合无序类别，如城市、颜色）
pd.get_dummies(df, columns=['col1', 'col2'], drop_first=True)
# drop_first=True：删除第一个哑变量，避免多重共线性

# Label Encoding（适合有序类别，如评级）
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
df['col'] = le.fit_transform(df['col'])

# Ordinal Encoding（明确指定顺序）
from sklearn.preprocessing import OrdinalEncoder
oe = OrdinalEncoder(categories=[['low', 'medium', 'high']])
df[['col']] = oe.fit_transform(df[['col']])
```

### 数值缩放（建模前必做）

```python
# StandardScaler：标准化，均值 0，标准差 1（适合线性模型、SVM）
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_train)      # fit 只在训练集上做
X_test_scaled = scaler.transform(X_test)      # 测试集只 transform，不 fit

# MinMaxScaler：归一化到 [0,1]（适合神经网络）
from sklearn.preprocessing import MinMaxScaler
mm = MinMaxScaler()
X_scaled = mm.fit_transform(X_train)

# ⚠️ 重要：scaler 只 fit 训练集，再 transform 测试集
# 否则会造成数据泄露（data leakage）
```

### 多项式特征

```python
# 单变量多项式（numpy）
f = np.polyfit(x, y, n)     # n 为阶数
p = np.poly1d(f)             # 生成多项式模型
y_hat = p(x)

# 多变量多项式特征（sklearn）
from sklearn.preprocessing import PolynomialFeatures
pr = PolynomialFeatures(degree=2, include_bias=False)
X_poly = pr.fit_transform(X)
# degree=2 会生成：x1, x2, x1², x2², x1·x2 等交叉特征
```

### 特征选择

```python
# 基于相关性删除低相关特征
corr = df.corr()['target'].abs().sort_values(ascending=False)
selected = corr[corr > 0.1].index.tolist()

# 删除高度共线的特征（相关性 > 0.9）
corr_matrix = df.corr().abs()
upper = corr_matrix.where(np.triu(np.ones(corr_matrix.shape), k=1).astype(bool))
to_drop = [col for col in upper.columns if any(upper[col] > 0.9)]
df.drop(columns=to_drop, inplace=True)

# 基于方差过滤（方差接近 0 的特征信息量低）
from sklearn.feature_selection import VarianceThreshold
sel = VarianceThreshold(threshold=0.01)
X_filtered = sel.fit_transform(X)
```

### 构造新特征

```python
df['age'] = 2024 - df['birth_year']
df['revenue_per_user'] = df['revenue'] / df['users']
df['log_price'] = np.log1p(df['price'])          # log 变换（处理右偏分布）
df['date'].dt.month | .dt.dayofweek | .dt.quarter
```

---

## 4. Train/Test Split

```python
from sklearn.model_selection import train_test_split

y = df['target']
X = df.drop('target', axis=1)

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,       # 20% 作为测试集
    random_state=42,     # 固定随机种子，保证复现
    stratify=y           # 分类问题加这行：保持各类别比例一致
)

print(X_train.shape, X_test.shape)
```

> ⚠️ **数据泄露（Data Leakage）三大来源**
> 1. 在全集上做 `fit_transform`，再 split → 应该先 split，再 fit
> 2. 测试集的信息进入特征工程（如用全集均值填充缺失值）
> 3. 目标变量的衍生特征混入 X

---

## 5. 模型训练

### 线性回归

```python
from sklearn.linear_model import LinearRegression

lr = LinearRegression()
lr.fit(X_train, y_train)

y_hat = lr.predict(X_test)

print('截距:', lr.intercept_)
print('系数:', lr.coef_)          # 每个特征对应的斜率
```

### 数据管道（Pipeline）

将预处理和模型串联，避免数据泄露，代码更整洁。

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.linear_model import LinearRegression

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('poly', PolynomialFeatures(degree=2, include_bias=False)),
    ('model', LinearRegression())
])

pipe.fit(X_train, y_train)
y_hat = pipe.predict(X_test)

# Pipeline 内部会自动保证：
# fit 阶段：scaler.fit_transform → poly.fit_transform → model.fit
# predict 阶段：scaler.transform → poly.transform → model.predict
```

### 残差图（检验模型假设）

```python
sns.residplot(x=y_hat, y=y_test - y_hat)
plt.axhline(0, color='red', linestyle='--')
plt.xlabel('Fitted Values')
plt.ylabel('Residuals')
# 理想状态：残差随机分布在 0 附近，无规律
# 若有规律（如漏斗形）→ 模型有问题（异方差）
```

---

## 6. 交叉验证

当数据量不足时，单次 train/test split 结果不稳定，用交叉验证得到更可靠的泛化估计。

```python
from sklearn.model_selection import cross_val_score, cross_val_predict
from sklearn.linear_model import LinearRegression

lr = LinearRegression()

# K-Fold 交叉验证（返回每折的 R²）
scores = cross_val_score(lr, X, y, cv=5, scoring='r2')
print('各折 R²:', scores)
print('均值:', scores.mean())
print('标准差:', scores.std())    # 标准差大 → 模型不稳定

# 交叉验证预测（返回每条数据在验证集时的预测值）
y_hat_cv = cross_val_predict(lr, X, y, cv=5)

# 其他 scoring 参数
# 'neg_mean_squared_error'  → MSE（注意取负值）
# 'neg_root_mean_squared_error' → RMSE
# 'r2'                      → R²
mse_scores = -cross_val_score(lr, X, y, cv=5, scoring='neg_mean_squared_error')
```

> **K-Fold 工作原理**：将数据分成 K 份，每次用 K-1 份训练、1 份验证，循环 K 次，取平均。K 通常取 5 或 10。

---

## 7. 模型评估：R² / MSE

| 指标 | 含义 | 范围 | 越高越好？ |
|------|------|------|---------|
| R² | 模型解释了目标变量多少比例的方差 | (-∞, 1] | ✅ 越接近 1 越好 |
| MSE | 预测误差的平方均值 | [0, +∞) | ❌ 越小越好 |
| RMSE | MSE 开方，单位与 y 一致，可解释性更强 | [0, +∞) | ❌ 越小越好 |
| MAE | 预测误差绝对值均值，对异常值更鲁棒 | [0, +∞) | ❌ 越小越好 |

```python
from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error

y_hat = lr.predict(X_test)

r2   = r2_score(y_test, y_hat)
mse  = mean_squared_error(y_test, y_hat)
rmse = np.sqrt(mse)
mae  = mean_absolute_error(y_test, y_hat)

print(f'R²:   {r2:.4f}')
print(f'MSE:  {mse:.4f}')
print(f'RMSE: {rmse:.4f}')
print(f'MAE:  {mae:.4f}')

# 线性回归直接调用 .score() 返回 R²
r2_train = lr.score(X_train, y_train)
r2_test  = lr.score(X_test,  y_test)

# 多项式回归的 R²
from sklearn.metrics import r2_score
f = np.polyfit(x, y, n)
p = np.poly1d(f)
r2 = r2_score(y, p(x))
```

---

## 8. 识别过拟合与欠拟合

```python
# 训练集 vs 测试集 R²
r2_train = lr.score(X_train, y_train)
r2_test  = lr.score(X_test,  y_test)

print(f'Train R²: {r2_train:.4f}')
print(f'Test  R²: {r2_test:.4f}')
```

| 情况 | Train R² | Test R² | 诊断 | 解决方向 |
|------|----------|---------|------|---------|
| 过拟合 | 高（如 0.95）| 明显低（如 0.60）| 模型记住了训练集噪声 | 正则化、减少特征、更多数据 |
| 欠拟合 | 低（如 0.40）| 低（如 0.38）| 模型太简单，捕捉不到规律 | 增加特征、提高多项式阶数、换模型 |
| 正常 | 高 | 接近训练集 | — | — |

```python
# 用学习曲线可视化过拟合/欠拟合
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    lr, X, y, cv=5, scoring='r2',
    train_sizes=np.linspace(0.1, 1.0, 10)
)

plt.plot(train_sizes, train_scores.mean(axis=1), label='Train R²')
plt.plot(train_sizes, val_scores.mean(axis=1), label='Val R²')
plt.xlabel('Training Size')
plt.legend()
# 两线差距大 → 过拟合；两线都低 → 欠拟合

# 用多项式阶数寻找最优复杂度
r2_train_list, r2_test_list = [], []
for d in range(1, 10):
    pr = PolynomialFeatures(degree=d)
    X_tr_p = pr.fit_transform(X_train)
    X_te_p = pr.transform(X_test)
    lr.fit(X_tr_p, y_train)
    r2_train_list.append(lr.score(X_tr_p, y_train))
    r2_test_list.append(lr.score(X_te_p, y_test))

plt.plot(range(1,10), r2_train_list, label='Train')
plt.plot(range(1,10), r2_test_list,  label='Test')
plt.xlabel('Polynomial Degree')
plt.legend()
# Test 曲线先升后降的拐点 = 最优阶数
```

---

## 9. 正则化：Ridge & Lasso

正则化在损失函数中加入惩罚项，限制系数大小，抑制过拟合。

| | Ridge（L2） | Lasso（L1） |
|--|-------------|-------------|
| 惩罚项 | 系数平方和 | 系数绝对值和 |
| 效果 | 系数收缩但不为 0 | 部分系数变为 0（自动特征选择）|
| 适用场景 | 多数特征都有贡献 | 特征多但真正有用的少 |
| 参数 | `alpha`（越大正则越强）| `alpha`（越大正则越强）|

```python
from sklearn.linear_model import Ridge, Lasso
from sklearn.preprocessing import PolynomialFeatures

# 多项式特征（Ridge/Lasso 常配合多项式使用）
pr = PolynomialFeatures(degree=2, include_bias=False)
X_train_pr = pr.fit_transform(X_train)
X_test_pr  = pr.transform(X_test)       # ⚠️ 测试集只 transform

# Ridge 回归
ridge = Ridge(alpha=1)
ridge.fit(X_train_pr, y_train)
y_hat_ridge = ridge.predict(X_test_pr)
print('Ridge R²:', ridge.score(X_test_pr, y_test))

# Lasso 回归
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_pr, y_train)
y_hat_lasso = lasso.predict(X_test_pr)
print('Lasso R²:', lasso.score(X_test_pr, y_test))

# 查看被 Lasso 置零的特征（即被筛掉的）
zero_coef = np.sum(lasso.coef_ == 0)
print(f'Lasso 置零特征数: {zero_coef}')

# Pipeline 写法（推荐）
from sklearn.pipeline import Pipeline
pipe_ridge = Pipeline([
    ('poly', PolynomialFeatures(degree=2, include_bias=False)),
    ('scaler', StandardScaler()),
    ('model', Ridge(alpha=1))
])
pipe_ridge.fit(X_train, y_train)
print('Ridge Pipeline R²:', pipe_ridge.score(X_test, y_test))
```

---

## 10. GridSearchCV 调参

自动遍历所有参数组合，用交叉验证找到最优参数。

```python
from sklearn.model_selection import GridSearchCV
from sklearn.linear_model import Ridge

# 定义参数网格
parameters = {'alpha': [0.001, 0.01, 0.1, 1, 10, 100, 1000]}

ridge = Ridge()
grid = GridSearchCV(ridge, parameters, cv=5, scoring='r2', refit=True)
grid.fit(X_train_pr, y_train)   # 在多项式特征上搜索

print('最优参数:', grid.best_params_)
print('最优 CV R²:', grid.best_score_)

# 直接用最优模型预测
best_model = grid.best_estimator_
y_hat = best_model.predict(X_test_pr)
print('测试集 R²:', r2_score(y_test, y_hat))

# 查看所有参数组合的结果
results = pd.DataFrame(grid.cv_results_)
results[['param_alpha', 'mean_test_score', 'std_test_score']].sort_values('mean_test_score', ascending=False)
```

### 多参数搜索

```python
# Pipeline + GridSearchCV（推荐生产写法）
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('poly', PolynomialFeatures(include_bias=False)),
    ('scaler', StandardScaler()),
    ('model', Ridge())
])

param_grid = {
    'poly__degree': [1, 2, 3],          # Pipeline 中参数用 步骤名__参数名
    'model__alpha': [0.1, 1, 10, 100]
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='r2', refit=True)
grid.fit(X_train, y_train)

print('最优参数:', grid.best_params_)
print('测试集 R²:', grid.score(X_test, y_test))
```

### RandomizedSearchCV（参数空间大时更高效）

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import loguniform

param_dist = {'model__alpha': loguniform(1e-3, 1e3)}

random_search = RandomizedSearchCV(pipe, param_dist, n_iter=50, cv=5, random_state=42)
random_search.fit(X_train, y_train)
print('最优参数:', random_search.best_params_)
```

---

## 11. 最终模型与输出

```python
# 1. 用最优参数重新在全量数据上训练
best_params = grid.best_params_
final_model = Pipeline([
    ('poly', PolynomialFeatures(degree=best_params['poly__degree'], include_bias=False)),
    ('scaler', StandardScaler()),
    ('model', Ridge(alpha=best_params['model__alpha']))
])
final_model.fit(X, y)   # 全量数据（训练集 + 测试集）

# 2. 最终评估
y_final = final_model.predict(X_test)
print('最终 R²:  ', r2_score(y_test, y_final))
print('最终 RMSE:', np.sqrt(mean_squared_error(y_test, y_final)))

# 3. 保存模型
import joblib
joblib.dump(final_model, 'model.pkl')

# 4. 加载模型并预测
loaded_model = joblib.load('model.pkl')
loaded_model.predict(X_new)

# 5. 分布图：实际值 vs 预测值（直观检验）
sns.kdeplot(y_test, color='r', label='Actual')
sns.kdeplot(y_final, color='b', label='Predicted')
plt.legend()
plt.title('Actual vs Predicted Distribution')
plt.show()
```

---

## 附：完整流程速查

```
原始数据
  ↓ df.info() / describe() / corr() / value_counts()
EDA 探索
  ↓ fillna / dropna / drop_duplicates / clip / astype
数据清洗
  ↓ get_dummies / StandardScaler / PolynomialFeatures / log变换
特征工程
  ↓ train_test_split(test_size=0.2, stratify=y)
Train/Test Split
  ↓ Pipeline([scaler, poly, LinearRegression]).fit(X_train, y_train)
模型训练
  ↓ cross_val_score(cv=5, scoring='r2').mean()
交叉验证
  ↓ r2_score / MSE / RMSE / MAE
模型评估
  ↓ train R² >> test R² → 过拟合
识别过拟合
  ↓ Ridge(alpha) / Lasso(alpha)
正则化
  ↓ GridSearchCV(param_grid, cv=5).best_params_
调参
  ↓ final_model.fit(X, y) → joblib.dump(model, 'model.pkl')
最终模型
```
