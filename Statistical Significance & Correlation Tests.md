# 统计显著性与相关性检验速查表 (Statistical Significance & Correlation Tests Cheat Sheet)

一份实用参考:怎么选检验方法、怎么在Python里跑、怎么正确解读结果——包括每个p-value该搭配哪个effect size(效应量)。术语全部中英对照,方便你在英国求职语境下直接用英文表达。

---

## 核心原则 (Core Principle)

**p-value(p值)回答的是"这个关联是不是真实存在的,不是随机噪音造成的"。它不回答"这个关联有多大、有多重要"。**

样本量越大,p-value越容易显著(significant),哪怕实际差异小到没有意义。**Effect size(效应量)** 才是真正告诉你这个结果重不重要的指标。任何时候都要两个一起看,判断"实际意义"时以effect size为准,不是p-value。

| | 回答的问题 (Answers) | 受样本量影响吗 (Sample-size sensitive?) |
|---|---|---|
| **p-value / p值** | 关联是否真实存在? | 是——样本越大越容易显著,跟效应大小无关 |
| **Effect size / 效应量** | 关联有多强/多大? | 否——是标准化后的指标,可跨研究比较 |


---

## 一、分类变量 vs 分类变量 (Categorical vs Categorical) —— 比较比例/计数

### 卡方独立性检验 (Chi-square test of independence)
- **什么时候用:** 两个分类变量,测它们是否相互独立(比如ethnicity vs. dropout status,种族 vs. 是否中途脱落)
- **前提假设 (assumptions):** 每个格子的期望频数(expected frequency)最好≥5,太小的话结果不可靠
- **搭配的effect size:** Cramér's V(适合行列都大于2的表);Phi coefficient(适合2×2表)

```python
from scipy.stats import chi2_contingency
from scipy.stats.contingency import association

chi2, p, dof, expected = chi2_contingency(contingency_table)
cramers_v = association(contingency_table.astype(int), method="cramer")
```

**Cramér's V怎么读:** <0.1 可忽略(negligible) · 0.1–0.3 弱到中等(weak–moderate) · >0.3 较强(strong)

**英文怎么说(面试/写作用语):**
"I ran a chi-square test of independence to check whether [variable A] and [variable B] are statistically associated, then used Cramér's V to assess the strength of that association."

### Fisher精确检验 (Fisher's exact test)
- **什么时候用:** 跟卡方检验问的是同一个问题,但适用于样本量小、期望频数<5的情况(卡方在小样本下不准,Fisher's是精确解,不是近似解)
- **典型场景:** 2×2表(scipy也支持更大的表,但计算量会明显增加)

```python
from scipy.stats import fisher_exact
odds_ratio, p = fisher_exact(table_2x2)
```

**英文怎么说:** "Given the small expected cell counts, I used Fisher's exact test instead of chi-square."

### 双比例z检验 (Two-proportion z-test)
- **什么时候用:** 只想比较两个具体组的比例是否有显著差异(比如"A族裔的dropout率是否显著高于B族裔"),通常是卡方检验整体显著之后,拿来做post-hoc(事后)两两比较的工具

```python
from statsmodels.stats.proportion import proportions_ztest

count = [n_events_group1, n_events_group2]
nobs = [n_total_group1, n_total_group2]
z_stat, p = proportions_ztest(count, nobs)
```

**英文怎么说:** "As a post-hoc follow-up, I used a two-proportion z-test to compare the two specific groups directly."

---

## 二、连续变量 vs 分类变量 (Continuous vs Categorical) —— 比较均值

### 独立样本t检验 (Independent samples t-test)
- **什么时候用:** 比较一个连续变量在恰好2组之间的均值差异
- **搭配的effect size:** Cohen's d

```python
from scipy.stats import ttest_ind
t_stat, p = ttest_ind(group1, group2)

import numpy as np
pooled_std = np.sqrt((group1.std()**2 + group2.std()**2) / 2)
cohens_d = (group1.mean() - group2.mean()) / pooled_std
```

**Cohen's d怎么读:** ~0.2 小(small) · ~0.5 中(medium) · ~0.8 大(large)

**英文怎么说:** "I ran an independent samples t-test to compare the means between the two groups, and reported Cohen's d alongside it to quantify the size of the difference."

### 配对t检验 (Paired t-test)
- **什么时候用:** 比较同一批对象在两个时间点/两种条件下的测量值(比如治疗前 vs. 治疗后)

```python
from scipy.stats import ttest_rel
t_stat, p = ttest_rel(before, after)
```

**英文怎么说:** "Since the same individuals were measured before and after treatment, I used a paired t-test rather than an independent samples test."

### 单因素方差分析 (One-way ANOVA)
- **什么时候用:** 比较一个连续变量在3组或以上之间的均值(比如你的16个族裔各自的average improvement score)
- **注意:** ANOVA只告诉你"至少有一组和别的不一样",不会告诉你具体是哪一组——如果显著了,想知道具体哪两组不同,需要再做post-hoc检验(比如Tukey HSD)
- **搭配的effect size:** Eta-squared (η²,eta平方)

```python
from scipy.stats import f_oneway
f_stat, p = f_oneway(group1, group2, group3)  # 每组一个array

# Eta-squared 用 statsmodels 算
import statsmodels.api as sm
from statsmodels.formula.api import ols

model = ols('score ~ C(group_column)', data=df).fit()
anova_table = sm.stats.anova_lm(model, typ=2)
eta_sq = anova_table['sum_sq'].iloc[0] / anova_table['sum_sq'].sum()
```

**Eta-squared怎么读:** ~0.01 小(small) · ~0.06 中(medium) · ~0.14 大(large)

**英文怎么说:** "I used a one-way ANOVA to test whether the group means differed significantly across the 16 categories, and reported eta-squared as the effect size."

### Tukey事后检验 (Tukey HSD, post-hoc pairwise test)
- **什么时候用:** ANOVA结果显著之后,想知道具体是哪几组之间存在差异

```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd
result = pairwise_tukeyhsd(endog=df['score'], groups=df['group_column'], alpha=0.05)
print(result)
```

**英文怎么说:** "Following a significant ANOVA result, I ran Tukey's HSD test to identify which specific pairs of groups differed."

### Kruskal-Wallis检验
- **什么时候用:** 跟ANOVA问的是同一个问题(3组以上的均值比较),但数据不满足正态分布假设,或者有明显离群值——是ANOVA的非参数(non-parametric)替代方法

```python
from scipy.stats import kruskal
h_stat, p = kruskal(group1, group2, group3)
```

**英文怎么说:** "Since the data didn't meet ANOVA's normality assumption, I used the Kruskal-Wallis test as a non-parametric alternative."

### Mann-Whitney U检验
- **什么时候用:** 跟t-test问的是同一个问题(2组比较),但不假设数据服从正态分布——是t-test的非参数替代方法

```python
from scipy.stats import mannwhitneyu
u_stat, p = mannwhitneyu(group1, group2)
```

**英文怎么说:** "I used the Mann-Whitney U test as a non-parametric alternative to the t-test."

---

## 三、连续变量 vs 连续变量 (Continuous vs Continuous) —— 相关性

### 皮尔逊相关系数 (Pearson correlation)
- **什么时候用:** 测两个连续变量之间是否存在线性关系(比如deprivation index vs. dropout率)
- **前提假设:** 大致呈线性关系、大致服从正态分布
- **effect size:** r本身就是,不需要另外计算

```python
from scipy.stats import pearsonr
r, p = pearsonr(x, y)
```

**r怎么读:** |r|<0.3 弱(weak) · 0.3–0.5 中(moderate) · >0.5 强(strong)

**英文怎么说:** "I calculated Pearson's correlation coefficient to assess the linear relationship between the two variables."

### 斯皮尔曼等级相关 (Spearman correlation)
- **什么时候用:** 跟Pearson问的是同一个问题,但不要求线性关系或正态分布,测的是"单调关系"(monotonic relationship)——一个变量增大,另一个是否倾向增大或减小,不要求是直线。数据有明显离群值,或者是等级(ordinal)数据时更适合

```python
from scipy.stats import spearmanr
rho, p = spearmanr(x, y)
```

**英文怎么说:** "Given the presence of outliers, I used Spearman's rank correlation instead of Pearson's."

### 肯德尔tau (Kendall's tau)
- **什么时候用:** 跟Spearman类似,但在样本量小或存在大量并列排名(tied ranks)时更稳健

```python
from scipy.stats import kendalltau
tau, p = kendalltau(x, y)
```

---

## 四、多变量情况 (Regression-based, 回归分析)

### 线性回归 (Linear regression)
- **什么时候用:** 想同时量化多个预测变量(连续或分类)对一个连续结果变量的解释力,并且控制其他变量的影响
- **effect size:** R²(解释的方差比例,proportion of variance explained)

```python
import statsmodels.api as sm
from statsmodels.formula.api import ols

model = ols('outcome ~ predictor1 + C(categorical_predictor)', data=df).fit()
print(model.summary())
```

**英文怎么说:** "I ran a linear regression to isolate the effect of [variable] while controlling for [other variables]."

### 逻辑回归 (Logistic regression)
- **什么时候用:** 结果变量是二元的(比如dropout: yes/no),想在控制多个预测变量的情况下看某个变量的独立效应——这通常是卡方检验显著之后,想进一步"控制混杂因素"时该走的下一步

```python
import statsmodels.formula.api as smf

model = smf.logit('dropout ~ C(ethnicity) + deprivation_index + age', data=df).fit()
print(model.summary())
```

**英文怎么说:** "To account for potential confounders like deprivation and age, I followed up the chi-square test with a logistic regression."

---

## 快速对照表 (Quick Decision Table)

| 比较什么 | 变量类型 | 检验方法 (中英对照) | Effect size |
|---|---|---|---|
| 多组之间的比例 | 分类 vs 分类 | 卡方检验 Chi-square test | Cramér's V |
| 2×2、小样本的比例 | 分类 vs 分类 | Fisher精确检验 Fisher's exact test | Odds ratio |
| 恰好两组的比例 | 分类 vs 分类 | 双比例z检验 Two-proportion z-test | — |
| 两组均值 | 连续 vs 二分类 | t检验 t-test | Cohen's d |
| 同一批对象前后测量 | 连续 vs 连续(配对) | 配对t检验 Paired t-test | Cohen's d |
| 三组以上均值 | 连续 vs 分类 | 方差分析 ANOVA | Eta-squared |
| 三组以上均值,数据非正态 | 连续 vs 分类 | Kruskal-Wallis检验 | — |
| 两组均值,数据非正态 | 连续 vs 二分类 | Mann-Whitney U检验 | — |
| 两个连续变量,线性关系 | 连续 vs 连续 | 皮尔逊相关 Pearson correlation | r |
| 两个连续变量,单调关系 | 连续 vs 连续 | 斯皮尔曼相关 Spearman correlation | rho |
| 结果变量~多个预测变量(连续) | 混合 | 线性回归 Linear regression | R² |
| 结果变量~多个预测变量(二元结果) | 混合 | 逻辑回归 Logistic regression | Pseudo-R² / odds ratios |

# Effect Size 取值范围与解读方法 (Effect Size Ranges & Interpretation)

补充笔记:不同的effect size(效应量),取值范围和阈值并不统一,不能都按0-1去理解。读法逻辑一致(数值越大代表关联/差异越强),但量纲和上限不同,不能跨类型直接比较数字大小。

---

## 一、有界在 0–1 之间的(接近"百分比"直觉)

- **Cramér's V** —— 0到1。1表示完全关联,0表示完全独立。用于列联表(contingency table),行列数都>2时。
- **Phi coefficient** —— 0到1。2×2表专用,本质是Cramér's V的特例。
- **R² (R-squared / 决定系数)** —— 0到1,表示模型解释了多少比例的方差(variance)。最接近"百分比"直觉:R²=0.3就是"这个模型解释了30%的变异"。
- **Eta-squared (η² / eta平方)** —— 0到1,逻辑跟R²很像(ANOVA本身是回归的特殊形式),表示组间差异占总变异的比例。

**r (Pearson/Spearman correlation coefficient)** —— 严格说是 **-1到1**,不是0到1,因为相关有方向(正相关/负相关)。看强弱时通常取绝对值 |r|。

---

## 二、没有固定上限的(不能套0-1去读)

- **Cohen's d** —— 没有固定上限,理论上可以是任意正数(实际研究里很少超过2)。衡量的是"两组均值差,以标准差为单位算出的距离",不是比例。0.2/0.5/0.8这几个阈值是经验上常引用的分界,不是理论边界。
- **Odds ratio (优势比)** —— 范围是0到正无穷,**1代表"没有差异"**(不是0)。odds ratio=2意味着某事件发生几率是对照组的2倍;odds ratio=0.5意味着只有一半。判断标准是"离1有多远",不是"离0有多远"。
- **卡方统计量本身 (chi-square value)** —— 完全没有固定范围,大小取决于自由度(degrees of freedom)和样本量,**不能单独当effect size使用**。

---

## 三、速查表

| Effect Size | 取值范围 | "无效应"对应值 | 常见阈值(小/中/大) |
|---|---|---|---|
| Cramér's V | 0 ~ 1 | 0 | 0.1 / 0.3 / 0.5 |
| Phi coefficient | 0 ~ 1 | 0 | 同Cramér's V |
| r (Pearson / Spearman) | -1 ~ 1 | 0 | 0.1 / 0.3 / 0.5 |
| R² | 0 ~ 1 | 0 | 0.01 / 0.09 / 0.25 |
| Eta-squared (η²) | 0 ~ 1 | 0 | 0.01 / 0.06 / 0.14 |
| Cohen's d | 0 ~ 无上限 | 0 | 0.2 / 0.5 / 0.8 |
| Odds ratio | 0 ~ 无上限 | **1**(不是0) | 无统一标准,看离1有多远 |

---

## 需要注意的一点 (a caveat worth remembering)

以上"小/中/大"这些阈值,本质上是**经验性的、领域惯例性的分界(empirical, field-specific conventions)**,不是数学上严格推导出来的规则。不同学科、不同应用场景,对"多大算大"的容忍度并不一样——比如社会科学和医学临床试验,对同一个Cohen's d值,判断"重不重要"的标准经常不同。

**如果被问到"这个阈值哪来的",可以这样回答(英文可直接用):**
"These thresholds are empirical conventions — most commonly attributed to Cohen — rather than fixed statistical rules. The right interpretation still depends on the context and field norms, not just the raw number."

---

## 贯穿所有方法的一条原则

**永远不要单独报告p-value。** 样本量大的时候,几乎任何非零差异都会统计显著,这是预期之内的正常现象,不是什么了不起的发现。永远要把显著性检验和对应的effect size放在一起看,用effect size(而不是p-value)来判断这个结果在实际中到底重不重要。

"Statistical significance tells you whether an effect exists; effect size tells you whether it matters. With large administrative datasets, p-values become significant very easily, so I always pair them with an effect size before drawing conclusions."
