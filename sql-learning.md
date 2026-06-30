# SQL 学习文档（完整体系版）

## 目录
1. [SQL 概述与发展历史](#1-sql-概述与发展历史)
2. [SQL 基础概念](#2-sql-基础概念)
3. [SQL 语法构成详解](#3-sql-语法构成详解)
4. [SQL 数据类型体系](#4-sql-数据类型体系)
5. [SQL 约束（Constraint）体系](#5-sql-约束constraint体系)
6. [SQL 语法分类总览](#6-sql-语法分类总览)
7. [DDL（数据定义语言）详解](#7-ddl数据定义语言详解)
8. [DML（数据操作语言）详解](#8-dml数据操作语言详解)
9. [DQL（数据查询语言）详解](#9-dql数据查询语言详解)
10. [DCL（数据控制语言）详解](#10-dcl数据控制语言详解)
11. [TCL（事务控制语言）详解](#11-tcl事务控制语言详解)
12. [SQL 查询执行顺序](#12-sql-查询执行顺序)
13. [SQL 高级语法](#13-sql-高级语法)
14. [SQL 函数体系](#14-sql-函数体系)
15. [SQL 索引与性能优化](#15-sql-索引与性能优化)
16. [SQL 锁与并发控制](#16-sql-锁与并发控制)
17. [SQL 事务与隔离级别](#17-sql-事务与隔离级别)
18. [SQL 执行计划与调优](#18-sql-执行计划与调优)
19. [SQL 编码规范与最佳实践](#19-sql-编码规范与最佳实践)
20. [SQL 面试与常见问题](#20-sql-面试与常见问题)
21. [SQL 学习路线建议](#21-sql-学习路线建议)

---

## 1. SQL 概述与发展历史

### 1.1 什么是 SQL
**SQL**（*Structured Query Language*，结构化查询语言）是一种用于关系型数据库管理系统（RDBMS）的标准语言。

#### SQL 的核心能力包括：
* **数据定义**：定义和修改数据库结构（库、表、视图、索引等）。
* **数据查询**：从数据库中高效检索所需数据。
* **数据操作**：对数据进行增加、删除、修改操作。
* **权限控制**：管理用户对数据库对象的访问权限。
* **事务控制**：确保多条操作的原子性、一致性、隔离性与持久性。
* **数据完整性约束**：通过规则确保数据的准确性与合法性。

#### 常见关系型数据库（RDBMS）特点：

| 数据库 | 特点 | 适用场景 |
| :--- | :--- | :--- |
| **MySQL** | 开源、免费、社区活跃，互联网场景应用最为广泛。 | 电商、社交平台、中小型网站 |
| **PostgreSQL** | 功能强大，号称最先进的开源数据库，对标准 SQL 支持极其优秀。 | 地理信息系统、复杂数据分析、企业级应用 |
| **Oracle** | 商业级数据库霸主，功能全面，稳定性和安全性极高，价格昂贵。 | 金融、银行、大型传统企业 |
| **SQL Server** | 微软生态核心数据库，图形化管理界面友好，与 Windows Server 集成度高。 | 微软技术栈企业、ERP 系统 |
| **SQLite** | 轻量级、嵌入式数据库，无需独立服务进程，数据存储在单一文件中。 | 移动 App、桌面软件、测试环境 |
| **MariaDB** | MySQL 的开源分支，由原作者主导开发，保持与 MySQL 的高度兼容。 | 替代 MySQL 的开源项目 |




---

### 1.2 SQL 的发展历史

#### （1）关系模型提出
1970 年，IBM 研究员 **Edgar F. Codd** 发表了题为 *《A Relational Model of Data for Large Shared Data Banks》* 的论文，正式提出**关系模型**。
* **核心思想**：
  1. 数据以二维表（Table）的形式存储。
  2. 使用数理逻辑和集合论的方法来操作数据。
  3. 实现数据与程序的物理及逻辑解耦。
* **地位**：这是现代关系型数据库的奠基性理论。

#### （2）SEQUEL 语言
IBM 基于 Codd 的理论开发了实验性关系型数据库系统 System R，并为其设计了语言 **SEQUEL**（*Structured English Query Language*）。后因商标争议，更名为 **SQL**。

#### （3）ANSI SQL 标准

为了打破各数据库厂商的割据状态，国际标准组织对 SQL 进行了标准化：

| 标准版本 | 发布时间 | 重要更新特性 |
| :--- | :--- | :--- |
| **SQL-86** | 1986 年 | 正式成为 ANSI 标准，第一个官方 SQL 标准。 |
| **SQL-89** | 1989 年 | 小幅更新，增强了完整性约束支持。 |
| **SQL-92** | 1992 年 | 奠定了现代 SQL 的基石，定义了 `LEFT/RIGHT JOIN` 等标准联表语法。 |
| **SQL:1999** | 1999 年 | 引入了对象关系特性、递归查询（`WITH RECURSIVE`）和触发器。 |
| **SQL:2003** | 2003 年 | 引入了**窗口函数**（Window Functions）以及对 XML 的支持。 |
| **SQL:2011** | 2011 年 | 增强了对时间段/时序数据的原生支持（Temporal Data）。 |
| **SQL:2016** | 2016 年 | 极大增强了对 **JSON** 数据类型的解析与操作能力。 |

---

### 1.3 SQL 与 NoSQL 的对比

| 特性维度 | SQL (关系型数据库) | NoSQL (非关系型数据库) |
| :--- | :--- | :--- |
| **数据模型** | 关系模型（二维表结构） | 多样化（键值、文档、列族、图形） |
| **一致性** | 强一致性（追求 ACID 强事务） | 最终一致性（追求 BASE 理论，高并发下权衡） |
| **表结构 (Schema)** | 固定且严格，变更成本高 | 动态灵活（Schema-less），可随时调整字段 |
| **横向扩展性** | 较难，通常依赖主从架构或分库分表 | 原生支持分布式，横向扩展（Scale-out）极其简单 |
| **复杂查询** | `JOIN` 联表能力极其强大 | `JOIN` 能力弱或不支持，需在应用层实现 |

---

## 2. SQL 基础概念

* **数据库（Database）**：长期存储在计算机内的、有组织的数据集合。

  ```sql
  CREATE DATABASE company;
  ```
  
* **表（Table）**：关系模型中的核心结构，由行与列组成的二维矩阵。

  ```sql
  CREATE TABLE employee (
      id INT,
      name VARCHAR(50)
  );
  ```
  
* **行（Row / Record）**：表中的一条具体记录，代表一个实体的完整信息。
* **列（Column / Field）**：表中的一个字段，代表实体的一种属性，具有统一的数据类型。
* **主键（Primary Key）**：用于唯一标识表中一条记录的字段。主键必须满足**唯一且非空**。
  ```sql
  id INT PRIMARY KEY
  ```
* **外键（Foreign Key）**：用于在两个表之间建立连接的字段，确保数据的引用的完整性。

---

## 3. SQL 语法构成详解

一条 SQL 语句看似是一句自然的英文，但在数据库引擎内部会被精准拆解。以下面这条语句为例：

```sql
UPDATE employee
SET salary = salary + 1000
WHERE department_id = 1;
```

#### 该语句的语法切片拆解：
| 语法成分 | 对应片段 | 作用说明 |
| :--- | :--- | :--- |
| **关键字 (Keyword)** | `UPDATE`, `SET`, `WHERE` | SQL 语言保留的内置核心命令，不可用作自定义名称。 |
| **标识符 (Identifier)** | `employee`, `salary`, `department_id` | 用户自定义的数据库对象名称（包括库名、表名、字段名等）。 |
| **字面量 (Literal)** | `1000`, `1` | 固定的常量值（数值、字符串、布尔值或 NULL）。 |
| **表达式 (Expression)** | `salary + 1000` | 由字段、常量和运算符组成的计算单元，会计算并产生一个结果值。 |
| **条件表达式 (Predicate)**| `department_id = 1` | 逻辑判断式，返回布尔值（TRUE/FALSE/UNKNOWN），用于过滤数据。 |
| **结束符** | `;` | 标记一条标准 SQL 语句的终结。 |

### 3.1 SQL 基本组成元素

1. **关键字（Keyword）**：如 `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` 等。
2. **标识符（Identifier）**：数据库名、表名、字段名、索引名、视图名等。
3. **字面量（Literal）**：
   * 数值：`100`, `3.14`
   * 字符串：`'hello'`, `'2026-05-22'`
   * 布尔值：`TRUE`, `FALSE`
   * 空值：`NULL`
4. **表达式（Expression）**：如 `salary * 12`, `age > 18`, `NOW()`。
5. **函数（Function）**：
   * 聚合函数：`COUNT(*)`, `SUM(price)`
   * 内置标量函数：`NOW()`, `SUBSTRING(name, 1, 3)`
6. **条件（Predicate）**：评估为布尔值的表达式，如 `id = 1`, `salary > 5000`, `name LIKE 'A%'`。
7. **子句（Clause）**：组成 SQL 语句的核心大模块，如 `WHERE子句`、`GROUP BY子句`。

### 3.2 SQL 语法结构（SELECT 完整结构）
一条标准且完整的 DQL 查询语句其书写结构如下：
```sql
SELECT 字段1, 字段2, 聚合函数()
FROM 表1
INNER JOIN 表2 ON 联表条件
WHERE 基础过滤条件
GROUP BY 分组字段
HAVING 分组后的聚合过滤条件
ORDER BY 排序字段 ASC/DESC
LIMIT 分页数量 OFFSET 偏移量;
```

### 3.3 SQL 中的约束语法组成
```sql
CREATE TABLE user (
    id INT PRIMARY KEY,               -- 主键约束
    name VARCHAR(50) NOT NULL,        -- 非空约束
    age INT CHECK(age > 0),           -- 检查约束
    email VARCHAR(100) UNIQUE         -- 唯一约束
);
```

---

## 4. SQL 数据类型体系

不同数据库类型名称略有差异，但标准核心体系如下：

### 4.1 数值类型
* `INT / INTEGER`：标准整数（通常 4 字节）。
* `BIGINT`：大整数（通常 8 字节），常用于分布式自增 ID、全网唯一标识。
* `DECIMAL(M, D) / NUMERIC`：**高精度定点小数**。M 为总位数，D 为小数位。**金融、货币计算必须使用该类型**，防止浮点数精度丢失。
* `FLOAT / DOUBLE`：近似浮点数，存在精度误差。

### 4.2 字符串类型
* `CHAR(N)`：**定长字符串**。若实际存储长度小于 N，会自动在右侧补空格。适合存储固定长度的数据（如 MD5 摘要、身份证号、状态码）。
* `VARCHAR(N)`：**变长字符串**。根据实际内容大小动态占用空间，节约磁盘，最常用的文本类型。
* `TEXT / BLOB`：长文本/二进制大对象，用于存储超长文章或二进制文件。

### 4.3 日期时间类型
* `DATE`：仅存储日期，格式 `YYYY-MM-DD`。
* `TIME`：仅存储时间，格式 `HH:MM:SS`。
* `DATETIME`：存储日期和时间，格式 `YYYY-MM-DD HH:MM:SS`，与时区无关。
* `TIMESTAMP`：**时间戳**，存储从 1970-01-01 以来的秒数。**受时区影响**，常用于记录创建时间、更新时间。

### 4.4 布尔类型
* `BOOLEAN`：存储 `TRUE` 或 `FALSE`（在 MySQL 等数据库中底层通常映射为 `TINYINT(1)` 的 1 和 0）。

### 4.5 JSON 类型
* `JSON`：现代关系型数据库（如 MySQL 5.7+，PostgreSQL 9.4+）原生支持，允许在关系型表中直接存储、索引和高效查询半结构化 JSON 文档。

---

## 5. SQL 约束（Constraint）体系

约束是定义在表字段上的规则，用以确保数据库数据的**实体完整性、域完整性和参照完整性**。

* **5.1 NOT NULL（非空约束）**：强制列不能存储 `NULL` 值。
* **5.2 UNIQUE（唯一约束）**：确保列中的所有值互不相同，但允许存在多个 `NULL` 值（视数据库具体实现而定）。
* **5.3 PRIMARY KEY（主键约束）**：`NOT NULL` 和 `UNIQUE` 的组合。一张表只能有一个主键，用于全局唯一标识行，数据库会自动为主键创建聚集/主键索引。
* **5.4 FOREIGN KEY（外键约束）**：限制外键列的值必须在另一个表的主键列中存在，维护表与表之间的参照完整性。
* **5.5 CHECK（检查约束）**：确保列中的值必须满足指定的布尔条件（如 `CHECK(age >= 18)`）。
* **5.6 DEFAULT（默认值约束）**：当插入数据未指定该字段值时，自动赋予预设的默认值（如 `status INT DEFAULT 0`）。

---

## 6. SQL 语法分类总览

根据功能侧重点不同，SQL 语言通常被划分为以下五大核心大类：

| 分类简称 | 全称 | 核心关键字 | 核心作用说明 |
| :--- | :--- | :--- | :--- |
| **DDL** | Data Definition Language (数据定义语言) | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | 用于创建、修改和删除数据库的结构（表、索引、视图等）。 |
| **DML** | Data Manipulation Language (数据操作语言) | `INSERT`, `UPDATE`, `DELETE` | 用于对表中的具体数据记录进行增、删、改操作。 |
| **DQL** | Data Query Language (数据查询语言) | `SELECT` | 用于从数据库表中检索、筛选和聚合数据。 |
| **DCL** | Data Control Language (数据控制语言) | `GRANT`, `REVOKE` | 用于控制用户对数据库对象的访问权限与安全级别。 |
| **TCL** | Transaction Control Language (事务控制语言) | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | 用于管理数据库事务，确保多步操作的 ACID 特性。 |

---

## 7. DDL（数据定义语言）详解

### 7.1 CREATE (创建)
```sql
CREATE DATABASE school_db;

USE school_db;

CREATE TABLE student (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT,
    gpa DECIMAL(3,2) DEFAULT 0.00,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
> **DDL `CREATE TABLE` 内部底层执行机制：**
> 1. **语法解析与校验**：解析器校验 SQL 语法正确性，检查当前用户是否具备建表权限。
> 2. **检查元数据冲突**：查询系统表（如 `information_schema`），确保同名表在当前 Schema 下不存在。
> 3. **创建表元数据**：在数据库字典中注册该表的结构信息（字段名、类型、约束等）。
> 4. **分配存储空间**：在磁盘文件系统上为该表创建对应的数据文件与索引文件（如 MySQL InnoDB 的 `.ibd` 文件），并初始化数据页。
> 5. **刷新系统表**：将最新状态同步到系统表，使新表对外界全局可见。

### 7.2 ALTER (修改)
```sql
-- 为表新增一列
ALTER TABLE student ADD COLUMN email VARCHAR(100);

-- 修改已有列的数据类型
ALTER TABLE student MODIFY COLUMN name VARCHAR(100) NOT NULL;

-- 删除某一列
ALTER TABLE student DROP COLUMN age;
```

### 7.3 DROP (删除)
```sql
-- 彻底删除表（表结构与数据全毁，不可逆的高危操作！）
DROP TABLE student;
```

### 7.4 TRUNCATE (截断/清空)
```sql
-- 清空表内的全部数据，但保留表结构
TRUNCATE TABLE student;
```
> **TRUNCATE 与 DELETE 的本质区别：**
> * `TRUNCATE` 的本质是 **DDL 操作**。它是通过直接释放该表所在的数据页来擦除数据，速度极快。
> * 它**不会触发任何行级触发器（Trigger）**，且操作通常**无法在事务中回滚**。
> * 它会**重置**自增 ID（`AUTO_INCREMENT`）回到初始值。

---

## 8. DML（数据操作语言）详解

### 8.1 INSERT (插入)
```sql
INSERT INTO student (id, name, gpa) 
VALUES (1, 'Tom', 3.85);
```
> **`INSERT` 底层执行机制：**
> 1. 权限与约束校验（是否非空、是否主键冲突）。
> 2. 将数据行载入/写入内存中的 Buffer Pool（缓冲池）。
> 3. 顺序写入 **Redo Log（重做日志）**，确保持久化与断电安全性。
> 4. 写入 **Undo Log（回滚日志）** 供事务回滚和 MVCC 使用。
> 5. 异步将内存中的脏页刷入磁盘。
> 6. 更新相关的辅助索引（Index Tree）。

### 8.2 UPDATE (更新)
```sql
UPDATE student
SET gpa = 4.00
WHERE id = 1;
```
> **`UPDATE` 底层执行机制：**
> 1. 结合索引或全表扫描找到 `id = 1` 的目标数据页，若不在内存中则从磁盘读入 Buffer Pool。
> 2. 事务对该行记录加锁（**排他锁 / 写锁**）。
> 3. 记录旧版本数据到 Undo Log（用于回滚和并发读）。
> 4. 在内存中修改该行字段，并将修改操作写入 Redo Log。
> 5. 事务提交后释放锁。

### 8.3 DELETE (删除)
```sql
DELETE FROM student
WHERE id = 1;
```
> **`DELETE` 底层执行机制：**
> 1. 定位目标行，对其加排他锁。
> 2. **并非立刻从物理磁盘上擦除该数据**，而是将该行记录的头信息标记为 **"已删除"（Delete Bit 标记为 1）**。
> 3. 该行原本占用的空间被标记为“可复用”。
> 4. 写入 Redo Log 和 Undo Log。最终由底层的 Purge 线程进行物理物理清理。

---

## 9. DQL（数据查询语言）详解

### 9.1 SELECT 基础语法与条件过滤 (WHERE)
```sql
SELECT name, gpa 
FROM student
WHERE gpa >= 3.50;
```

### 9.2 常用运算符
* **比较运算符**：`=`, `>`, `<`, `>=`, `<=`, `<>` (不等于, 亦可用 `!=`)
* **逻辑运算符**：`AND`, `OR`, `NOT`
* **区间与集合**：`BETWEEN 80 AND 100`, `IN (1, 3, 5, 7)`

### 9.3 模糊查询 (LIKE)
```sql
-- % 代表任意数量的任意字符；_ 代表单个任意字符
SELECT * FROM student WHERE name LIKE 'Tom%';
```

### 9.4 分组与聚合过滤 (GROUP BY & HAVING)
```sql
SELECT department_id, COUNT(*), AVG(salary)
FROM employee
GROUP BY department_id
HAVING AVG(salary) > 5000;
```
> **⚠️ 核心避坑点：** `WHERE` 是在**分组前**对原始行进行数据筛选；`HAVING` 是在**分组后**对聚合出的结果集进行二次筛选。`HAVING` 内部可以使用聚合函数，而 `WHERE` 绝对不行。

### 9.5 联表查询 (JOIN) 详解

假设存在 A 表（左表）和 B 表（右表）：

* **INNER JOIN（内连接）**：仅返回两表满足关联条件（`ON`）的**交集部分**。
  ```sql
  SELECT * FROM employee e INNER JOIN department d ON e.dept_id = d.id;
  ```
* **LEFT JOIN（左外连接）**：返回**左表全部记录**，以及右表中符合条件的记录。若右表无对应匹配，则右表部分显示为 `NULL`。
  ```sql
  SELECT * FROM employee e LEFT JOIN department d ON e.dept_id = d.id;
  ```
* **RIGHT JOIN（右外连接）**：与 LEFT JOIN 相反，返回**右表全部记录**及左表匹配记录。
* **FULL JOIN（全外连接）**：返回两表的**并集**，任何一方不匹配则填 `NULL`（MySQL 不原生支持，需通过 `LEFT JOIN UNION RIGHT JOIN` 实现）。

---

## 10. DCL（数据控制语言）详解

用于 DBA 级别对数据库进行细粒度安全与权限管控。

```sql
-- 10.1 GRANT (授权)
-- 将 school_db 数据库中 student 表的 SELECT 权限赋予用户 user1
GRANT SELECT, INSERT ON school_db.student TO 'user1'@'localhost';

-- 10.2 REVOKE (回收权限)
REVOKE INSERT ON school_db.student FROM 'user1'@'localhost';

-- 刷新权限使其生效
FLUSH PRIVILEGES;
```

---

## 11. TCL（事务控制语言）详解

事务是逻辑上的一组原子操作。

```sql
-- 11.1 显式开启事务
BEGIN; -- 或 START TRANSACTION;

-- 执行一连串数据变更
UPDATE account SET balance = balance - 100 WHERE id = A;
UPDATE account SET balance = balance + 100 WHERE id = B;

-- 11.2 COMMIT (提交事务)
-- 将事务内的所有变更永久刷入磁盘，并释放行锁
COMMIT;

-- 11.3 ROLLBACK (回滚事务)
-- 若中途发生异常，取消整个事务，利用 Undo Log 将数据全部恢复至初始状态
ROLLBACK;
```

---

## 12. SQL 查询执行顺序

在编写 SQL 时，我们按照 `SELECT ... FROM ... WHERE` 的语义规范来写；但在数据库解析引擎内部，执行顺序完全不同。

#### 1. 编写顺序 vs 2. 执行顺序：
| 步骤 | 编写顺序 (Lexical) | 执行顺序 (Logical) | 核心原理作用 |
| :--- | :--- | :--- | :--- |
| **1** | `SELECT` | `FROM` | 确定要查询的主表，读取该数据表。 |
| **2** | `FROM` | `ON` | 应用 JOIN 联表条件，生成多表连接的笛卡尔积/临时表。 |
| **3** | `JOIN` | `JOIN` | 结合联表类型（LEFT/RIGHT）补充不匹配的行。 |
| **4** | `ON` | `WHERE` | **最关键的初筛**，对原始数据行进行条件过滤，极大减少后续计算量。 |
| **5** | `WHERE` | `GROUP BY` | 将初筛后的数据按指定字段划分成独立的分组格。 |
| **6** | `GROUP BY` | `HAVING` | 对分组后的聚合结果（如 `COUNT/AVG`）进行二次条件过滤。 |
| **7** | `HAVING` | `SELECT` | 此时才真正提取并计算最终要输出的字段、表达式或窗口函数。 |
| **8** | `DISTINCT` | `DISTINCT` | 对选出的结果集进行去重。 |
| **9** | `ORDER BY` | `ORDER BY` | 对去重后的最终集合进行排序（升序/降序）。 |
| **10**| `LIMIT / OFFSET`| `LIMIT` | 截取指定偏移量和数量的行数，返回给客户端。 |

> **💡 面试核心常考点：** 为什么在 `WHERE` 子句中不能直接使用 `SELECT` 中定义的字段别名？
> **答案：** 因为 `WHERE` 的执行顺序（第 4 步）远早于 `SELECT`（第 7 步）。当执行 `WHERE` 进行过滤时，别名还没有被数据库计算出来！同理，`ORDER BY` 却可以使用别名，因为它的执行在 `SELECT` 之后。

---

## 13. SQL 高级语法

### 13.1 子查询 (Subquery)

* **标量子查询**（返回单行单列的单一值）：
  ```sql
  SELECT * FROM employee 
  WHERE salary > (SELECT AVG(salary) FROM employee);
  ```
* **IN 子查询**（判断字段是否在子查询返回的集合中）：
  ```sql
  SELECT * FROM student WHERE id IN (SELECT DISTINCT student_id FROM score WHERE num > 90);
  ```
* **EXISTS 子查询**（高效的布尔存在性判断）：
  ```sql
  -- 只要内部子查询有一条数据匹配成功，外部行即刻判定通过，适合大表关联
  SELECT * FROM department d 
  WHERE EXISTS (SELECT 1 FROM employee e WHERE e.dept_id = d.id);
  ```

### 13.2 CTE（公共表表达式）
CTE 比子查询具有更高的可读性，可视为局部作用域内的虚拟临时表。
```sql
WITH high_salary_dept AS (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employee
    GROUP BY dept_id
)
SELECT d.name, h.avg_sal
FROM department d
INNER JOIN high_salary_dept h ON d.id = h.dept_id
WHERE h.avg_sal > 10000;
```

### 13.3 递归 CTE
常用于处理组织架构树、类目菜单树等层级递归结构。
```sql
WITH RECURSIVE org_tree AS (
    -- 锚点成员：根节点
    SELECT id, name, parent_id, 1 AS level FROM employee WHERE parent_id IS NULL
    UNION ALL
    -- 递归成员：联表向下迭代
    SELECT e.id, e.name, e.parent_id, t.level + 1
    FROM employee e
    INNER JOIN org_tree t ON e.parent_id = t.id
)
SELECT * FROM org_tree;
```

### 13.4 窗口函数 (Window Functions)
窗口函数既能进行聚合计算，又能同时保留每一行数据的原始明细，不发生合并。
* **ROW_NUMBER()**：无脑连续唯一排名（如：1, 2, 3）。
* **RANK()**：并列排名，跳过后续名次（如：1, 2, 2, 4）。
* **DENSE_RANK()**：并列排名，不跳过后续名次（如：1, 2, 2, 3）。
```sql
-- 查出每个部门内部薪资最高的前三名
SELECT department_id, name, salary,
       ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) as salary_rank
FROM employee;
```

### 13.5 CASE WHEN 条件分支
```sql
SELECT name, score,
       CASE 
           WHEN score >= 90 THEN '优秀'
           WHEN score >= 80 THEN '良好'
           WHEN score >= 60 THEN '及格'
           ELSE '不及格'
       END AS grade
FROM student;
```

### 13.6 UNION 与 UNION ALL 的本质区别
```sql
SELECT name FROM A UNION SELECT name FROM B;
```
* **UNION**：将两组查询结果合并后进行**去重排序**，性能开销大。
* **UNION ALL**：简单粗暴地将两组结果上下拼接，**保留所有重复行**，不进行排序，**性能极高**。在明确知道没有重复数据或不需要去重时，**必须优先选择 UNION ALL**。

---

## 14. SQL 函数体系

### 14.1 聚合函数
* `COUNT(列名)`：统计非 NULL 行数；`COUNT(*)` 统计包含 NULL 在内的全局总行数。
* `SUM()`, `AVG()`, `MAX()`, `MIN()`

### 14.2 常用字符串函数
* `CONCAT(str1, str2, ...)`：拼接字符串。
* `SUBSTRING(str, pos, len)`：字符串截取（注意：**SQL 中基数索引通常从 1 开始**，而非 0）。
* `LENGTH(str)`：获取字节长度。
* `LOWER() / UPPER()`：大小写转换。

### 14.3 常用日期时间函数
* `NOW()` / `CURRENT_TIMESTAMP`：当前精确日期时间。
* `CURDATE()`：当前日期（不含时间）。
* `DATE_ADD(date, INTERVAL 7 DAY)`：日期递增。
* `DATEDIFF(date1, date2)`：计算两日期天数差。

### 14.4 常用数学函数
* `ROUND(num, d)`：四舍五入保留 d 位小数。
* `CEIL(num)` / `FLOOR(num)`：向上/向下取整。

---

## 15. SQL 索引与性能优化

### 15.1 什么是索引
索引本质上是**帮助数据库管理系统（RDBMS）高效检索数据的数据结构**。
* **通俗比喻**：相当于一本书籍前面的“目录”。如果没有目录，想找某个知识点就必须把整本书从第一页翻到最后一页（全表扫描，Full Table Scan）；有了索引，只需查阅目录直接定位页码。

### 15.2 为什么主流数据库采用 B+Tree 索引？
B+Tree 是一种高度低、扇出大的多路平衡查找树。其核心优势包括：
1. **磁盘 IO 极少**：B+Tree 的非叶子节点仅存储键值（不存具体行 Row 记录），这使得单个节点能容纳海量指针，树的高度通常稳定在 3 ~ 4 层。这意味着找任何一条记录最多只需 3 ~ 4 次磁盘 IO。
2. **范围查询极佳**：B+Tree 的所有**叶子节点之间存在双向双向链表**指针相连。当查出范围起点后，可以直接通过链表向后顺序遍历，极其契合 `BETWEEN`, `>`, `<` 等范围查询。

### 15.3 索引类型划分
* **主键索引 (Primary Key Index)**：建立在主键上的索引，值唯一且不允许为 NULL。在 InnoDB 中即为聚簇索引。
* **唯一索引 (Unique Index)**：限制索引列的值必须唯一，但允许为 NULL。
* **普通索引 (Normal Index)**：没有任何限制，仅用于加速查询。
* **联合索引 (Composite Index)**：由多个字段共同组成的索引（如 `INDEX(a, b, c)`）。
* **全文索引 (Fulltext Index)**：用于在大文本块中快速检索关键词（非精确匹配）。

### 15.4 创建索引语法
```sql
CREATE INDEX idx_user_name ON user(name);
```

### 15.5 联合索引最左匹配原则
假设对字段建立了联合索引 `INDEX(a, b, c)`，查询时条件的匹配规则如下：
* **完全命中**：`WHERE a=1`, `WHERE a=1 AND b=2`, `WHERE a=1 AND b=2 AND c=3` （与 AND 的前后书写顺序无关，优化器会自动调整）。
* **部分命中**：`WHERE a=1 AND c=3`（仅 a 能用到索引，c 无法用到）。
* **完全失效**：`WHERE b=2 AND c=3`（由于缺失最左开端列 `a`，索引彻底失效，走全表扫描）。

### 15.6 导致索引失效的十大高频死穴（核心重点）
1. **在索引列上进行函数操作**：`WHERE YEAR(create_time) = 2026`（应改为 `WHERE create_time >= '2026-01-01'`）。
2. **在索引列上进行算术表达式计算**：`WHERE salary + 1000 > 5000`。
3. **隐式类型转换**：`WHERE phone = 13800000000`（若 phone 字段在设计时是 VARCHAR 字符串类型，而传入了数值，数据库内部会自动调用函数转化，导致索引失效）。
4. **LIKE 前缀模糊查询**：`WHERE name LIKE '%abc'`（前置百分号导致 B+Tree 无法由根向下搜索，但 `LIKE 'abc%'` 可以正常走索引）。
5. **在索引列上使用不等于判断**：`WHERE status <> 1` 或 `NOT IN`（大概率导致引擎放弃索引改为全表扫描）。
6. **OR 链接未加索引的列**：`WHERE id = 1 OR age = 18`（若 age 没有索引，则整体索引失效）。

---

## 16. SQL 锁与并发控制

### 16.1 为什么需要锁
在多用户同时并发读写数据库时，锁是为了保证数据在多线程操作下的**一致性与正确性**。

### 16.2 锁的分类维度

#### 按锁的粒度划分：
* **表级锁 (Table Lock)**：直接锁住整张表。开销小、加锁快；但并发度极低，容易发生锁冲突。
* **行级锁 (Row Lock)**：仅锁住被修改的具体行记录。开销大、加锁慢，可能会产生死锁；但并发度最高，适合高频高并发写入。

#### 按锁的兼容性/意图划分：
* **共享锁 (S 锁 / 读锁)**：允许多个事务同时读取同一行，但任何事务在此期间无法修改。`多个读锁可共存`。
* **排他锁 (X 锁 / 写锁)**：当一个事务对某行加了写锁，其他事务既不能读也不能写该行。`写锁与任何锁都互斥`。

### 16.3 显式行加锁控制
```sql
-- 开启读取时加排他锁（常用于防超卖、金融账户变动）
BEGIN;
SELECT balance FROM account WHERE id = 1 FOR UPDATE;
-- 此时其他事务若执行相同的 FOR UPDATE 或修改此行，将被阻塞挂起，直到本事务 COMMIT
COMMIT;
```

### 16.4 死锁 (Deadlock)
* **成因**：事务 A 锁住了记录 1 并试图去锁定记录 2；与此同时，事务 B 锁住了记录 2 并试图去锁定记录 1。双方互相死等对方释放，形成闭环。
* **数据库防卫机制**：现代 RDBMS（如 InnoDB）具备死锁自动检测机制（Wait-for Graph）。一旦发现死锁环，会自动选择一个代价较小的事务进行强制**回滚**，并释放其持有的锁，让另一个事务顺利走完。

---

## 17. SQL 事务与隔离级别

### 17.1 事务的 ACID 四大特性
1. **Atomicity (原子性)**：事务是不可分割的最小执行单元。要么全部成功，要么全部失败回滚（由 **Undo Log** 保证）。
2. **Consistency (一致性)**：事务执行前后，数据库必须从一个一致性状态转换到另一个一致性状态。
3. **Isolation (隔离性)**：并发执行的各个事务之间相互隔离，互不干扰（由 **锁机制 & MVCC** 保证）。
4. **Durability (持久性)**：一旦事务提交，其对数据的修改必须永久保存到磁盘中，断电绝不丢失（由 **Redo Log** 保证）。

### 17.2 并发事务的三大灾难问题
* **脏读 (Dirty Read)**：事务 A 读取了事务 B **尚未提交**的修改数据。随后事务 B 回滚，事务 A 读到的成了脏数据。
* **不可重复读 (Non-repeatable Read)**：事务 A 在一个事务内多次读取同一条记录。在此期间事务 B **修改并提交**了该记录，导致事务 A 两次读到的数据不一致。
* **幻读 (Phantom Read)**：事务 A 在一个事务内按某个条件读取了一批数据。在此期间事务 B **插入了新的记录并提交**，当事务 A 再次以相同条件读取时，凭空多出了几行，如同产生了幻觉。

### 17.3 四大事务隔离级别
为了解决上述三大并发问题，SQL 标准定义了四个隔离级别（从上往下隔离能力增强，但并发吞吐量降低）：

| 隔离级别 (Isolation Level) | 脏读可能 | 不可重复读可能 | 幻读可能 | 常见数据库默认值 |
| :--- | :---: | :---: | :---: | :--- |
| **Read Uncommitted** (读未提交) | ❌ 有 | ❌ 有 | ❌ 有 | 极少使用 |
| **Read Committed** (读已提交) | ✅ 无 | ❌ 有 | ❌ 有 | Oracle, SQL Server, PostgreSQL |
| **Repeatable Read** (可重复读) | ✅ 无 | ✅ 无 | ❌ 有 | **MySQL InnoDB** (注：InnoDB 在此级别下通过 Next-Key Lock 基本解决了幻读) |
| **Serializable** (串行化) | ✅ 无 | ✅ 无 | ✅ 无 | 纯单线程串行排队，性能极差 |

### 17.4 MVCC（多版本并发控制）
MVCC 是现代高性能数据库（MySQL/PostgreSQL等）实现高并发的核心技术。
* **核心思想**：通过维护数据的多个历史版本，实现**“读写不冲突”**。在读多写少的场景中，读操作完全不需要加锁，从而极大提升吞吐。
* **底层三大基石**：
  1. **Undo Log 版本链**：每次修改数据时，旧版本数据都会记录在 Undo Log 中，通过回滚指针（`roll_ptr`）串联成一条从新到旧的历史版本链。
  2. **隐式字段**：每行数据自带 DB_TRX_ID（最近修改本行的事务 ID）。
  3. **Read View (一致性视图)**：事务启动时生成的一张快照，记录当前全系统所有活跃（未提交）的事务 ID。通过一套严格的判定算法，比对版本链，决定当前事务究竟“有权看见”哪个历史版本的数据。

---

## 18. SQL 执行计划与调优

### 18.1 EXPLAIN 关键字
在任何复杂的查询语句前加上 `EXPLAIN`，可以窥探数据库优化器最终决定的查询执行计划，这是性能调优的“听诊器”。

```sql
EXPLAIN SELECT * FROM employee WHERE department_id = 5 AND name = 'Alice';
```

### 18.2 EXPLAIN 核心字段深度解读
* **id**：查询执行的顺序标识。id 越大越先执行；id 相同则由上至下顺序执行。
* **select_type**：查询类型（SIMPLE 简单查询，PRIMARY 最外层查询，SUBQUERY 子查询等）。
* **type (★极其核心★)**：访问类型，代表找数据的手段。**性能由好到差依次为**：
  * `system` > `const` (主键/唯一索引等值精准命中) > `eq_ref` > `ref` (非唯一索引等值命中) > `range` (索引范围扫描，如 BETWEEN, >, <) > `index` (全索引树扫描) > `ALL` (**全表扫描，灾难级别，必须优化**)。
* **possible_keys**：预测可能会用到的索引。
* **key**：**实际高频命中的索引名称**。若为 NULL 则说明没有用到索引。
* **rows**：估计为了找到目标记录而必须扫描的物理行数。该值越小越好。
* **Extra (额外重要提示)**：
  * `Using index`：应用了**覆盖索引**（查询所需的字段全部在索引树上就能拿到，无需回表去找原始行数据，性能极高）。
  * `Using filesort`：**文件排序**。说明无法利用索引的顺序进行输出，被迫在内存或磁盘中组织排序，**性能较差，需通过建联合索引优化**。
  * `Using temporary`：使用了临时表（常见于不当的 `GROUP BY` 或 `DISTINCT`），**急需优化**。

### 18.3 慢 SQL 优化黄金流程
1. **第一步：精准定位慢 SQL**：开启数据库的 **Slow Query Log（慢查询日志）**，设置阈值（如超过 1 秒），或者通过 APM 系统捕捉生产环境高频变慢的语句。
2. **第二步：分析执行计划**：使用 `EXPLAIN` 查看该语句的 `type`、`key` 和 `Extra` 字段。
3. **第三步：索引靶向优化**：
   * 检查是否缺失索引，针对 `WHERE`、`JOIN ON` 或 `ORDER BY` 后面的高频字段合理建立索引。
   * 调整建立联合索引，尽量追求 `Using index` 覆盖索引状态，杜绝 `Using filesort`。
4. **第四步：SQL 语句重写**：
   * 将高成本的子查询（Subquery）改写为高性能的 `JOIN` 连接。
   * 大范围的 `OR` 条件改写为多条语句的 `UNION ALL`。
   * 杜绝使用 `SELECT *`，只查所需列。
5. **第五步：深分页（Deep Paging）调优**：
   * 错误示范：`LIMIT 1000000, 10` （引发百万级数据的回表扫描，瞬间卡死）。
   * 优化方案：利用主键自增特性转换为范围裁剪：`WHERE id > last_max_id LIMIT 10`。

---

## 19. SQL 编码规范与最佳实践

* **19.1 命名规范**：
  * 数据库对象名（库、表、列、索引）统一使用**小写字母**，单词之间用下划线 `_` 隔离（如 `user_order_detail`），禁止大小写混用，禁止使用中文。
  * 索引命名推荐统一前缀：主键索引 `pk_`，唯一索引 `uk_`，普通联合索引 `idx_`。
* **19.2 SQL 书写美化**：SQL 关键字统一大写，多字段换行对齐，增强可读性。
* **19.3 杜绝使用 `SELECT *`**：
  * 浪费宝贵的网络传输带宽。
  * 破坏“覆盖索引”的优化可能性。
  * 字段变更时极易引发应用端代码反序列化崩溃。
* **19.4 避免 NULL 滥用**：建表字段尽量声明为 `NOT NULL DEFAULT ...`。因为 NULL 值会使数据库的主键、索引统计逻辑复杂化，且在 SQL 判断中容易引发三值逻辑盲区（UNKNOWN）。
* **19.5 小表驱动大表**：在大表与小表进行联表查询（JOIN）时，尽量让数据规模较小的表作为驱动表，以便利用大表上的索引进行快速循环匹配。

---

## 20. SQL 面试与常见问题

#### 20.1 DELETE、TRUNCATE、DROP 的本质区别
| 特性维度 | DELETE | TRUNCATE | DROP |
| :--- | :--- | :--- | :--- |
| **所属分类** | DML (数据操作) | DDL (数据定义) | DDL (数据定义) |
| **删除内容** | 删除指定的行数据（逐行删除） | 清空表内全部数据 | 毁灭性删除整张表（数据+结构全无） |
| **事务回滚** | **支持**（写入 Undo Log，可 ROLLBACK） | **通常不支持** | **绝对不支持** |
| **空间释放** | 不释放磁盘空间，仅做逻辑删除标记 | 立即释放数据页空间，重置自增 | 立即释放该表占用的所有物理磁盘空间 |
| **执行效率** | 慢（需要逐行加锁、记日志） | 极快 | 极快 |

#### 20.2 WHERE 与 HAVING 的本质区别
* `WHERE` 作用于表中的原始行数据，其执行处于 `GROUP BY` 分组之前。其内部绝对不可以使用聚合函数。
* `HAVING` 作用于经由 `GROUP BY` 划分后的聚合虚拟组，其执行处于分组之后。通常配合聚合函数对组结果进行二次筛选。

#### 20.3 为什么 B+Tree 比 B-Tree 更适合做数据库索引？
1. B+Tree 的内层分支节点不包含行数据，所以同样大小的数据页能够塞入更多的路由键，**树的扇出（Fan-out）更大，层高更低，磁盘 IO 次数显著变少**。
2. B+Tree 的所有叶子节点形成了一个有顺的双向链表，对数据库中高频出现的**范围扫描、区间查询和排序输出**具备压倒性的性能优势。

---

## 21. SQL 学习路线建议

```
 [第一阶段：基础CRUD与单表操作]
  └── 掌握 SELECT, INSERT, UPDATE, DELETE 基础四板斧
  └── 玩转 WHERE 条件过滤、LIKE 模糊匹配、ORDER BY 排序
  └── 学习 GROUP BY 与 HAVING 的组合拳

 [第二阶段：多表联查与复杂高级语法]
  └── 烂熟于心：INNER/LEFT/RIGHT JOIN 的连表逻辑与边界
  └── 进阶语法：子查询、CASE WHEN 分支、UNION ALL 合并
  └── 现代高级 SQL 杀手锏：窗口函数(Window Functions)与 CTE 表达式

 [第三阶段：数据库底层原理修炼]
  └── 深入死磕 B+Tree 索引数据结构、深刻理解最左匹配原则
  └── 攻克事务 ACID 理论，研读 MVCC 机制与隔离级别的落地实现
  └── 洞悉行锁、表锁、间隙锁与死锁成因

 [第四阶段：性能调优与架构思维]
  └── 熟练掌握 EXPLAIN 执行计划，诊断慢 SQL 顽疾
  └── 掌握索引失效的避坑指南与重写调优技巧
  └── 扩展学习：读写分离、分库分表、分布式数据库架构
```

---
