# SQL-ROUND (MySQL + Oracle)

A focused SQL reference for interview prep and daily use. Covers table setup, CRUD, joins, aggregations, CASE, UNION, stored procedures, and common interview questions.

---

## 📑 Table of Contents
1. [Create Tables & Insert Data](#1-create-tables--insert-data)
2. [SELECT with Conditions](#2-select-with-conditions)
3. [UPDATE & DELETE](#3-update--delete)
4. [JOIN + GROUP BY + HAVING + ORDER BY](#4-join--group-by--having--order-by)
5. [UNION](#5-union)
6. [CASE Expression](#6-case-expression)
7. [Stored Procedure](#7-stored-procedure)
8. [Second Highest Salary](#8-second-highest-salary)
9. [Find Duplicate Records](#9-find-duplicate-records)

---

## 1. Create Tables & Insert Data

> ✅ **MySQL + Oracle** (minor syntax difference noted)

**MySQL**
```sql
CREATE TABLE departments (
    id   INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

CREATE TABLE employee (
    id            INT AUTO_INCREMENT PRIMARY KEY,
    name          VARCHAR(50) NOT NULL,
    department_id INT,
    salary        DECIMAL(10,2),
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

**Oracle**
```sql
CREATE TABLE departments (
    id   NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR2(50) NOT NULL
);

CREATE TABLE employee (
    id            NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name          VARCHAR2(50) NOT NULL,
    department_id NUMBER,
    salary        NUMBER(10,2),
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

**Insert Data** *(same for both)*
```sql
INSERT INTO departments (name) VALUES ('HR');
INSERT INTO departments (name) VALUES ('Sales');
INSERT INTO departments (name) VALUES ('IT');

INSERT INTO employee (name, department_id, salary) VALUES ('Alice', 1, 60000);
INSERT INTO employee (name, department_id, salary) VALUES ('Bob',   2, 55000);
INSERT INTO employee (name, department_id, salary) VALUES ('Charlie', 3, 70000);
```

> 💡 MySQL allows multi-row `INSERT INTO ... VALUES (...),(...)` in one statement. Oracle requires individual inserts or `INSERT ALL`.

---

## 2. SELECT with Conditions

> ✅ **MySQL + Oracle**

Retrieve employees whose name starts with 'A', OR belongs to Sales (dept 2) with salary > 50000.

```sql
SELECT * FROM employee
WHERE salary > 50000 AND department_id = 2
   OR name LIKE 'A%';
```

> ⚠️ AND has higher precedence than OR. Use parentheses to be explicit:
```sql
WHERE (salary > 50000 AND department_id = 2) OR name LIKE 'A%'
```

Salary range filter:
```sql
SELECT * FROM employee
WHERE salary BETWEEN 80000 AND 100000;
```

---

## 3. UPDATE & DELETE

> ✅ **MySQL + Oracle**

**Update single department's salary by 10%:**
```sql
UPDATE employee
SET salary = salary * 1.1
WHERE department_id = 3;
```

**Increase all employees' salary by 5%:**
```sql
UPDATE employee
SET salary = salary * 1.05;
```

**Delete a specific employee:**
```sql
DELETE FROM employee
WHERE name = 'Bob';
```

---

## 4. JOIN + GROUP BY + HAVING + ORDER BY

> ✅ **MySQL + Oracle**

Average salary per department, only departments with AVG > 50000, sorted descending:

```sql
SELECT d.name, AVG(e.salary) AS avg_salary
FROM departments d
JOIN employee e ON d.id = e.department_id
GROUP BY d.name
HAVING AVG(e.salary) > 50000
ORDER BY avg_salary DESC;
```

> ⚠️ MySQL allows `HAVING avg_salary > 50000` (alias reference). Oracle does **not** — use `HAVING AVG(e.salary) > 50000`.

---

## 5. UNION

> ✅ **MySQL + Oracle**

`UNION` combines results of two queries and removes duplicates. `UNION ALL` keeps duplicates.

Example — list all department names and all employee names in one result:
```sql
SELECT name, 'Department' AS type FROM departments
UNION
SELECT name, 'Employee'   AS type FROM employee;
```

> 💡 Both queries must have the same number of columns and compatible data types.

---

## 6. CASE Expression

> ✅ **MySQL + Oracle**

Categorize employees by salary bracket:

```sql
SELECT name, salary,
    CASE
        WHEN salary > 70000                    THEN 'High'
        WHEN salary BETWEEN 50000 AND 70000    THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employee;
```

---

## 7. Stored Procedure

Get all employees by department ID.

**MySQL**
```sql
DELIMITER //

CREATE PROCEDURE getDeptById(IN deptid INT)
BEGIN
    SELECT * FROM employee
    WHERE department_id = deptid;
END;
//

DELIMITER ;

-- Call it:
CALL getDeptById(2);
```

**Oracle**
```sql
CREATE OR REPLACE PROCEDURE getDeptById(deptid IN NUMBER) AS
BEGIN
    FOR rec IN (SELECT * FROM employee WHERE department_id = deptid) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.name || ' - ' || rec.salary);
    END LOOP;
END;
/

-- Call it:
EXEC getDeptById(2);
```

---

## 8. Second Highest Salary

Three approaches — know all three for interviews.

**Approach 1 — Subquery** *(MySQL + Oracle)*
```sql
SELECT MAX(salary) FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

**Approach 2 — LIMIT + OFFSET** *(MySQL only)*
```sql
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Approach 3 — FETCH NEXT** *(Oracle)*
```sql
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
OFFSET 1 ROWS FETCH NEXT 1 ROWS ONLY;
```

> 💡 Subquery approach is the safest — works on both MySQL and Oracle.

---

## 9. Find Duplicate Records

> ✅ **MySQL + Oracle**

Find names that appear more than once in the employee table:

```sql
SELECT name, COUNT(name) AS count
FROM employee
GROUP BY name
HAVING COUNT(name) > 1;
```
