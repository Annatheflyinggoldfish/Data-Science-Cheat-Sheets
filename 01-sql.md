
# SQL Cheat Sheet

---

## 聚合与标量函数

| 函数 | 语法 | 说明 |
|------|------|------|
| `COUNT` | `SELECT COUNT(column) FROM table WHERE condition;` | 返回符合条件的行数 |
| `AVG` | `SELECT AVG(column) FROM table WHERE condition;` | 返回数值列的平均值 |
| `SUM` | `SELECT SUM(column) FROM table WHERE condition;` | 返回数值列的总和 |
| `MIN` | `SELECT MIN(column) FROM table WHERE condition;` | 返回列的最小值 |
| `MAX` | `SELECT MAX(column) FROM table WHERE condition;` | 返回列的最大值 |
| `ROUND` | `SELECT ROUND(number, decimals);` | 将数字四舍五入到指定小数位 |
| `LENGTH` | `SELECT LENGTH(column) FROM table;` | 返回字符串的字节长度 |
| `UCASE` | `SELECT UCASE(column) FROM table;` | 转换为大写 |
| `LCASE` | `SELECT LCASE(column) FROM table;` | 转换为小写 |
| `DISTINCT` | `SELECT DISTINCT column FROM table;` | 去重显示数据 |

---

## 日期函数

| 函数 | 语法 | 说明 |
|------|------|------|
| `DAY` | `SELECT DAY(column) FROM table;` | 返回日期中的天数 |
| `CURRENT_DATE` | `SELECT CURRENT_DATE;` | 显示当前日期 |
| `DATEDIFF` | `SELECT DATEDIFF(date1, date2);` | 计算两个日期之间的天数差 |
| `FROM_DAYS` | `SELECT FROM_DAYS(n);` | 将天数转换为 YYYY-MM-DD 格式 |
| `DATE_ADD` | `SELECT DATE_ADD(date, INTERVAL n DAY/MONTH/YEAR);` | 在日期上加 n 个时间单位 |
| `DATE_SUB` | `SELECT DATE_SUB(date, INTERVAL n DAY/MONTH/YEAR);` | 在日期上减 n 个时间单位 |
| `DATE_FORMAT` | `SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');` | 格式化日期，例：`'2026-04-11 14:45:00'` |

---

## 字符串函数

```sql
-- 拼接字符串
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- 分组拼接（去重、排序、指定分隔符）
SELECT GROUP_CONCAT(DISTINCT col ORDER BY col ASC SEPARATOR ', ') FROM table;

-- 截取子字符串（从第1位起，取3个字符）
SELECT SUBSTRING('abcdef', 1, 3);  -- 结果：abc

-- 字符串长度
SELECT LENGTH(col) FROM table;
```

---

## 连接（JOIN）

### 内连接（INNER JOIN）
只返回两表中匹配的行。

```sql
SELECT E.F_NAME, E.L_NAME, JH.START_DATE
FROM EMPLOYEES AS E
INNER JOIN JOB_HISTORY AS JH ON E.EMP_ID = JH.EMPL_ID
WHERE E.DEP_ID = '5';
```

### 左外连接（LEFT OUTER JOIN）
返回左表所有行，右表无匹配时填 NULL。

```sql
SELECT E.EMP_ID, E.L_NAME, E.DEP_ID, D.DEP_NAME
FROM EMPLOYEES AS E
LEFT OUTER JOIN DEPARTMENTS AS D ON E.DEP_ID = D.DEPT_ID_DEP;
```

### 右外连接（RIGHT OUTER JOIN）
返回右表所有行，左表无匹配时填 NULL。

```sql
SELECT E.EMP_ID, E.L_NAME, E.DEP_ID, D.DEP_NAME
FROM EMPLOYEES AS E
RIGHT OUTER JOIN DEPARTMENTS AS D ON E.DEP_ID = D.DEPT_ID_DEP;
```

### 全外连接（FULL OUTER JOIN）
返回两表所有行，无匹配时填 NULL。MySQL 不直接支持，用 LEFT JOIN UNION RIGHT JOIN 模拟。

```sql
-- 标准语法（DB2、PostgreSQL 等支持）
SELECT E.F_NAME, E.L_NAME, D.DEP_NAME
FROM EMPLOYEES AS E
FULL OUTER JOIN DEPARTMENTS AS D ON E.DEP_ID = D.DEPT_ID_DEP;

-- MySQL 模拟写法
SELECT E.F_NAME, E.L_NAME, D.DEP_NAME
FROM EMPLOYEES AS E
LEFT OUTER JOIN DEPARTMENTS AS D ON E.DEP_ID = D.DEPT_ID_DEP
UNION
SELECT E.F_NAME, E.L_NAME, D.DEP_NAME
FROM EMPLOYEES AS E
RIGHT OUTER JOIN DEPARTMENTS AS D ON E.DEP_ID = D.DEPT_ID_DEP;
```

### 交叉连接（CROSS JOIN）
笛卡尔积：左表每行与右表每行配对。

```sql
SELECT DEPT_ID_DEP, LOCT_ID
FROM DEPARTMENTS
CROSS JOIN LOCATIONS;
```

### 隐式连接（Implicit JOIN）

```sql
-- 隐式内连接
SELECT * FROM employees, jobs
WHERE employees.job_id = jobs.job_ident;

-- 隐式交叉连接（无 WHERE 条件）
SELECT * FROM employees, jobs;
```

### 自连接（Self JOIN）
表与自身连接。

```sql
SELECT B.*
FROM EMPLOYEES A
JOIN EMPLOYEES B ON A.MANAGER_ID = B.MANAGER_ID
WHERE A.EMP_ID = 'E1001';
```

---

## 子查询（Subquery）

嵌套在另一个 SQL 语句中的查询，通常用于 WHERE 或 FROM 子句。

```sql
-- WHERE 子句中：工资低于平均工资的员工
SELECT emp_id, f_name, l_name, salary
FROM employees
WHERE salary < (SELECT AVG(salary) FROM employees);

-- FROM 子句中：将子查询结果作为虚拟表
SELECT * FROM (
  SELECT emp_id, f_name, l_name, dep_id FROM employees
) AS emp4all;

-- IN 子句中：匹配另一张表的值
SELECT * FROM employees
WHERE job_id IN (SELECT job_ident FROM jobs);
```

---

## EXISTS（存在判断）

判断子查询是否返回任何行，常用于关联查询。

```sql
-- 找出至少下过一次订单的客户
SELECT * FROM customers c
WHERE EXISTS (
  SELECT 1  -- 1/0/* 写法等价，只判断有无结果
  FROM orders o
  WHERE o.customer_id = c.id
);
```

---

## 窗口函数（Window Functions）

> 与 GROUP BY 不同：窗口函数**不合并行**，每行保留，同时能"看"到其他行的值。

### 排名函数

```sql
-- RANK：并列时跳号（1,1,3）
-- DENSE_RANK：并列时不跳号（1,1,2）
SELECT name, dept, salary,
  RANK()       OVER (PARTITION BY dept ORDER BY salary DESC) AS rnk,
  DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS dense_rnk
FROM employees;

-- 取各部门工资前 3 名（常用模式）
SELECT * FROM (
  SELECT *, DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM employees
) t
WHERE rn <= 3;
```

### 行编号

```sql
-- ROW_NUMBER：每行唯一编号，无并列
ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY record_date)
```

### 前后行函数

```sql
-- LAG：取上一行的值；LEAD：取下一行的值
SELECT date, sales,
  LAG(sales, 1)  OVER (ORDER BY date) AS prev_sales,
  LEAD(sales, 1) OVER (ORDER BY date) AS next_sales,
  sales - LAG(sales, 1) OVER (ORDER BY date) AS diff
FROM t;

-- 增长率：(sales - prev_sales) / prev_sales
```

### 滑动窗口聚合

```sql
-- 最近 3 天累计销售额（含当天）
SUM(sales) OVER (
  PARTITION BY store
  ORDER BY date
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)

-- 滑动平均（最近 3 天）
AVG(sales) OVER (
  ORDER BY date
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)

-- 分组累计趋势（每个店从头累计）
SUM(sales) OVER (
  PARTITION BY store
  ORDER BY date
)
```

> ⚠️ **易错点**：窗口函数不能直接用在 WHERE 里，需包一层子查询。
>
> ```sql
> -- ❌ 错误
> WHERE LAG(col) = col
>
> -- ✅ 正确
> SELECT * FROM (
>   SELECT col, LAG(col) OVER (...) AS prev
>   FROM table
> ) t
> WHERE col = prev;
> ```

---

## 条件聚合（CASE WHEN）

```sql
-- 条件求和
SUM(CASE WHEN condition THEN col ELSE 0 END)

-- 条件计数
COUNT(CASE WHEN condition THEN 1 END)

-- 示例：分别统计男女员工数
SELECT
  COUNT(CASE WHEN sex = 'M' THEN 1 END) AS male_count,
  COUNT(CASE WHEN sex = 'F' THEN 1 END) AS female_count
FROM employees;
```

---

## CTE（公用表表达式 / WITH）

将复杂子查询命名，使主查询更清晰。可定义多个 CTE。

```sql
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM employees
)
SELECT * FROM ranked
WHERE rn <= 3;

-- 多个 CTE
WITH
  cte1 AS (SELECT ...),
  cte2 AS (SELECT ... FROM cte1)
SELECT * FROM cte2;
```

---

## 视图（Views）

```sql
-- 创建视图
CREATE VIEW EMPSALARY AS
SELECT EMP_ID, F_NAME, L_NAME, B_DATE, SEX, SALARY
FROM EMPLOYEES;

-- 更新视图
CREATE OR REPLACE VIEW EMPSALARY AS
SELECT EMP_ID, F_NAME, L_NAME, B_DATE, SEX, JOB_TITLE, MIN_SALARY, MAX_SALARY
FROM EMPLOYEES, JOBS
WHERE EMPLOYEES.JOB_ID = JOBS.JOB_IDENT;

-- 删除视图
DROP VIEW EMPSALARY;
```

---

## 存储过程（Stored Procedures）

```sql
-- 创建存储过程（MySQL）
DELIMITER //
CREATE PROCEDURE RETRIEVE_ALL()
BEGIN
  SELECT * FROM PETSALE;
END //
DELIMITER ;

-- 调用
CALL RETRIEVE_ALL();
```

---

## 事务控制（TCL）

### 基本 COMMIT / ROLLBACK

```sql
-- 开启事务并提交
START TRANSACTION;
INSERT INTO employee VALUES (1, 'Alice', 'London', 50000, 30);
COMMIT;

-- 回滚（需先关闭自动提交）
SET autocommit = 0;
INSERT INTO employee VALUES (2, 'Bob', 'Manchester', 45000, 25);
ROLLBACK;  -- 上面的 INSERT 不会保存
```

### 存储过程中的事务（含异常处理）

```sql
DELIMITER //
CREATE PROCEDURE TRANSACTION_ROSE()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    ROLLBACK;
    RESIGNAL;
  END;

  START TRANSACTION;

  UPDATE BankAccounts SET Balance = Balance - 200 WHERE AccountName = 'Rose';
  UPDATE BankAccounts SET Balance = Balance - 300 WHERE AccountName = 'Rose';

  COMMIT;
END //
DELIMITER ;
```

---

## 相关子查询（Correlated Subquery）

子查询引用外层查询的列，逐行执行。

```sql
-- 工资高于本部门平均水平的员工
SELECT * FROM employees t1
WHERE salary > (
  SELECT AVG(salary) FROM employees t2
  WHERE t1.dep_id = t2.dep_id
);
```
