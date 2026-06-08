# 🗄️ SQL 核心功能与高频考点速查表

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
