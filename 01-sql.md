# 🗄️ SQL 核心功能速查表

> 💡 本表语法以 **MySQL 8.0+** 为主，兼容主流关系型数据库（如 PostgreSQL）。

---

## 1. 查询数据

### 🔍 基础查询与过滤
```sql
SELECT * FROM table_name;
SELECT DISTINCT column FROM table_name; -- 针对指定列去重

-- 条件过滤
SELECT col1, col2 FROM table WHERE condition;
SELECT * FROM table WHERE cond1 AND cond2;
SELECT * FROM table WHERE cond1 OR cond2;
SELECT * FROM table WHERE NOT condition;

-- 排序
SELECT * FROM table ORDER BY col ASC;                     -- 升序
SELECT * FROM table ORDER BY col1 ASC, col2 DESC;         -- 多列排序

-- 限制行数（MySQL 语法）
SELECT col FROM table LIMIT 5;            -- 取前 5 行
SELECT col FROM table LIMIT 10, 5;        -- 跳过前 10 行，取 5 行 (Offset, Count)

-- 模糊匹配 (LIKE)
SELECT col FROM table WHERE col LIKE 'a%';    -- 以 a 开头
SELECT col FROM table WHERE col LIKE '%a';    -- 以 a 结尾
SELECT col FROM table WHERE col LIKE '%or%';  -- 包含 or
SELECT col FROM table WHERE col LIKE '_r%';   -- 第二位是 r
SELECT col FROM table WHERE col LIKE 'a_%_%'; -- a 开头且长度 ≥ 3

-- 范围 / 多值判断
SELECT col FROM table WHERE col BETWEEN val1 AND val2;  -- 闭区间 [val1, val2]
SELECT col FROM table WHERE col IN (val1, val2, val3);
SELECT col FROM table WHERE col IN (SELECT col FROM table2); -- 子查询集合

-- NULL 判断 (⚠️ 不能用 = NULL)
SELECT * FROM table WHERE col IS NULL;
SELECT * FROM table WHERE col IS NOT NULL;

-- 别名
SELECT col AS alias FROM table;
SELECT CONCAT(col1, ', ', col2) AS full_name FROM table; -- 通用字符串拼接

-- 合并结果集
SELECT col FROM t1 UNION SELECT col FROM t2;      -- 合并并去重（效率较低）
SELECT col FROM t1 UNION ALL SELECT col FROM t2;  -- 合并保留重复（推荐，效率高）

-- 子查询多值条件
SELECT col FROM t1 WHERE col > ANY (SELECT col FROM t2 WHERE cond); -- 大于集合中任意一个（等价于大于最小值）
SELECT col FROM t1 WHERE col > ALL (SELECT col FROM t2 WHERE cond); -- 大于集合中所有个（等价于大于最大值）

-- 分组 + 聚合后过滤
SELECT col, COUNT(col2) 
FROM table
GROUP BY col
HAVING COUNT(col2) > 5                     -- ⚠️ 聚合函数的过滤必须用 HAVING
ORDER BY COUNT(col2) DESC;

-- 空值处理与控制流
COALESCE(col1, col2, '默认值') -- 返回第一个非 NULL 的值（极其强大）
IFNULL(col, 0)                -- 如果是 NULL 则变 0 (MySQL 独有)


-- 修改表结构
ALTER TABLE table ADD column_name datatype;
ALTER TABLE table MODIFY column_name new_type;
ALTER TABLE table DROP COLUMN column_name;
```

---

## 2. 正则表达式（REGEXP_LIKE）

```sql
-- 基本语法
WHERE REGEXP_LIKE(column, 'pattern', 'flags')
-- flags: 'c' 大小写敏感 | 'i' 大小写不敏感（默认）
```

| 类别 | 表达式 | 含义 |
|------|--------|------|
| **锚点** | `^` | 字符串开头 |
| | `$` | 字符串结尾 |
| | `^...$` | 匹配整个字符串 |
| **字符类** | `[A-Z]` | 大写字母 |
| | `[a-z]` | 小写字母 |
| | `[0-9]` | 数字 |
| | `[A-Za-z0-9]` | 字母或数字 |
| | `[^abc]` | 不是 a、b、c |
| | `[.]` | 真正的点（推荐） |
| **数量** | `*` | 0 个或多个 |
| | `+` | 1 个或多个 |
| | `?` | 0 个或 1 个 |
| | `{3}` | 恰好 3 个 |
| | `{2,5}` | 2 到 5 个 |

```sql
-- 以字母开头
WHERE REGEXP_LIKE(col, '^[A-Za-z]')

-- 纯数字
WHERE REGEXP_LIKE(col, '^[0-9]+$')

-- 字母开头，后接字母/数字/下划线
WHERE REGEXP_LIKE(col, '^[A-Za-z][A-Za-z0-9_]*$')

-- 邮箱格式（通用）
WHERE REGEXP_LIKE(col, '^[A-Za-z][A-Za-z0-9_.-]*@[A-Za-z0-9.-]+[.][A-Za-z]{2,}$')

-- 指定域名邮箱（大小写敏感）
WHERE REGEXP_LIKE(col, '^[A-Za-z][A-Za-z0-9_.-]*@leetcode[.]com$', 'c')

-- 11 位手机号
WHERE REGEXP_LIKE(col, '^[0-9]{11}$')

-- 包含至少一个数字
WHERE REGEXP_LIKE(col, '[0-9]')
```

---

## 3. 修改数据

```sql
INSERT INTO table (col1, col2) VALUES (val1, val2);
UPDATE table SET col1 = val1, col2 = val2 WHERE condition;
DELETE FROM table WHERE condition;
```

---

## 4. 聚合函数

```sql
SELECT COUNT(DISTINCT col) FROM table;
SELECT MIN(col), MAX(col), AVG(col), SUM(col) FROM table WHERE condition;
```

---

## 5. 连接（JOIN）

```sql
-- 内连接：只返回匹配行
SELECT cols FROM t1 INNER JOIN t2 ON t1.id = t2.id;

-- 左外连接：左表全部 + 右表匹配
SELECT cols FROM t1 LEFT JOIN t2 ON t1.id = t2.id;

-- 右外连接：右表全部 + 左表匹配
SELECT cols FROM t1 RIGHT JOIN t2 ON t1.id = t2.id;

-- 全外连接：两表所有行，无匹配填 NULL
SELECT cols FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;

-- MySQL 模拟全外连接
SELECT cols FROM t1 LEFT JOIN t2 ON t1.id = t2.id
UNION
SELECT cols FROM t1 RIGHT JOIN t2 ON t1.id = t2.id;

-- 交叉连接：笛卡尔积
SELECT cols FROM t1 CROSS JOIN t2;

-- 自连接
SELECT B.* FROM employees A JOIN employees B ON A.manager_id = B.manager_id
WHERE A.emp_id = 'E1001';
```

---

## 6. 子查询 / CTE / 窗口函数

```sql
-- 子查询（WHERE）
SELECT * FROM employees WHERE salary < (SELECT AVG(salary) FROM employees);

-- 子查询（FROM）
SELECT * FROM (SELECT emp_id, dep_id FROM employees) AS sub;

-- EXISTS
SELECT * FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- 相关子查询：工资高于本部门平均值
SELECT * FROM employees t1
WHERE salary > (SELECT AVG(salary) FROM employees t2 WHERE t1.dep_id = t2.dep_id);

-- CTE（WITH）
WITH ranked AS (
  SELECT *, DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM employees
)
SELECT * FROM ranked WHERE rn <= 3;
```

---

## 7. 窗口函数

```sql
-- 排名：RANK 并列跳号(1,1,3)；DENSE_RANK 不跳号(1,1,2)
RANK()       OVER (PARTITION BY dept ORDER BY salary DESC)
DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)  -- 唯一编号，无并列

-- 前后行
LAG(col, 1)  OVER (ORDER BY date)  -- 上一行
LEAD(col, 1) OVER (ORDER BY date)  -- 下一行

-- 滑动窗口（最近 3 天累计）
SUM(sales) OVER (PARTITION BY store ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)

-- ⚠️ 窗口函数不能直接用在 WHERE，需包子查询
SELECT * FROM (
  SELECT col, LAG(col) OVER (...) AS prev FROM table
) t WHERE col = prev;
```

---

## 8. 条件聚合

```sql
SUM(CASE WHEN condition THEN col ELSE 0 END)
COUNT(CASE WHEN condition THEN 1 END)
```

---

## 9. 常用函数

```sql
-- 聚合
COUNT(*) | AVG(col) | SUM(col) | MIN(col) | MAX(col) | ROUND(col, 2)

-- 字符串
LENGTH(col)
UCASE(col) | LCASE(col)
CONCAT(a, ' ', b)
SUBSTRING(col, 1, 3)
GROUP_CONCAT(DISTINCT col ORDER BY col ASC SEPARATOR ', ')

-- 日期
CURRENT_DATE
DAY(col) | DATEDIFF(date1, date2)
DATE_ADD(date, INTERVAL 7 DAY) | DATE_SUB(date, INTERVAL 7 DAY)
DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s')
```

---

## 10. 视图 / 存储过程 / 事务

```sql
-- 视图
CREATE VIEW v AS SELECT col1, col2 FROM table WHERE condition;
CREATE OR REPLACE VIEW v AS SELECT ...;
DROP VIEW v;

-- 存储过程
DELIMITER //
CREATE PROCEDURE proc_name (IN param datatype)
BEGIN
  UPDATE table SET col = param WHERE condition;
END //
DELIMITER ;
CALL proc_name(value);
DROP PROCEDURE proc_name;

-- 事务
START TRANSACTION;
-- ... SQL 操作 ...
COMMIT;    -- 提交
ROLLBACK;  -- 回滚（需先 SET autocommit = 0;）
```

---

