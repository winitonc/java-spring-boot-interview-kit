# SQL Cheat Sheet

## SELECT
```sql
SELECT col1, col2, COUNT(*), SUM(col), AVG(col), MAX(col), MIN(col)
FROM table_name
WHERE condition
GROUP BY col1
HAVING aggregate_condition
ORDER BY col DESC
LIMIT 10 OFFSET 20;
```

## JOINs
```sql
-- INNER JOIN: ทั้งสอง table match
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: ทุก row จาก left + match จาก right (NULL ถ้าไม่มี)
SELECT u.name, COUNT(o.id) order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- หา users ที่ไม่มี orders
SELECT u.* FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

## Subquery
```sql
-- IN subquery
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- EXISTS
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Scalar subquery
SELECT name, salary,
    (SELECT AVG(salary) FROM employees) as avg_salary
FROM employees;
```

## Window Functions
```sql
SELECT name, dept, salary,
    RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as dept_rank,
    SUM(salary) OVER (PARTITION BY dept) as dept_total,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as overall_rank
FROM employees;
```

## DDL
```sql
CREATE TABLE users (
    id    BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    age   INT CHECK (age >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_composite ON orders(user_id, status);
DROP INDEX idx_users_email;
```

## Transaction
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;   -- หรือ ROLLBACK;
```

## ACID Quick Reminder
| | ความหมาย |
|--|---------|
| **A**tomicity | ทั้งหมดหรือไม่มีเลย |
| **C**onsistency | ข้อมูลอยู่ใน valid state เสมอ |
| **I**solation | transaction ไม่รบกวนกัน |
| **D**urability | commit แล้วข้อมูลถาวร |

## Normalization Quick Reminder
| Form | กำจัด |
|------|------|
| **1NF** | Repeating groups, non-atomic values |
| **2NF** | Partial dependency (non-key → part of composite PK) |
| **3NF** | Transitive dependency (non-key → non-key) |

## Index Guide
| ✅ สร้าง index | ❌ ไม่ต้อง |
|--------------|----------|
| WHERE columns ที่ใช้บ่อย | Table เล็กมาก |
| JOIN columns (FK) | Boolean columns |
| ORDER BY columns | Write-heavy table |
| Unique constraints | Column ที่ cardinality ต่ำ |
