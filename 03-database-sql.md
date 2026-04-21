# 03 — Database & SQL

> **เป้าหมาย:** เข้าใจ relational database concepts, SQL ที่ใช้บ่อย, Transaction และ Index  
> **หัวข้อ:** Relational Concepts · ACID · Normalization · SQL Queries · Index · Transaction ใน Spring

---

## 1. Relational Database Concepts

### Key Types

| Key | ความหมาย | ตัวอย่าง |
|-----|---------|---------|
| **Primary Key (PK)** | uniquely identify แต่ละ row, ไม่ null | `user_id` |
| **Foreign Key (FK)** | อ้างอิง PK ของ table อื่น, enforce referential integrity | `orders.user_id → users.id` |
| **Unique Key** | ค่าไม่ซ้ำ แต่ยอมให้ null (ต่างจาก PK) | `users.email` |
| **Composite Key** | PK ที่ใช้หลาย column รวมกัน | `(user_id, product_id)` ใน order_items |

### Table Relationships

```
One-to-One:   users (1) ←→ (1) user_profiles
One-to-Many:  users (1) ←→ (N) orders
Many-to-Many: products (N) ←→ (N) orders  → ต้องมี junction table (order_items)
```

---

## 2. ACID Properties

ACID คือคุณสมบัติที่ Transaction ควรมีเพื่อความน่าเชื่อถือของข้อมูล

| Property | ความหมาย | ตัวอย่าง |
|---------|---------|---------|
| **Atomicity** | Transaction ทำทั้งหมดหรือไม่ทำเลย | โอนเงิน: ถ้าตัดยอดสำเร็จแต่เพิ่มล้มเหลว → rollback ทั้งคู่ |
| **Consistency** | Data อยู่ใน valid state เสมอ | balance ไม่ติดลบ (constraint ไม่ถูกละเมิด) |
| **Isolation** | Transaction ไม่เห็นผล intermediate ของ transaction อื่น | 2 user โอนเงินพร้อมกัน ผลไม่ overlap |
| **Durability** | เมื่อ commit แล้ว ข้อมูลถาวร แม้ server crash | บันทึกลง disk / write-ahead log |

### คำถามสัมภาษณ์

**Q: ACID คืออะไร ทำไมสำคัญ?**  
A: ACID คือ properties ที่รับประกันความน่าเชื่อถือของ transaction ใน database ที่ใช้ในระบบการเงินหรือ critical data ถ้าขาด Atomicity เงินอาจหายกลางคัน ถ้าขาด Isolation ข้อมูลอาจ dirty read ทำให้ตัดสินใจผิด

---

## 3. Normalization

จุดประสงค์: ลด **data redundancy** และป้องกัน **update anomalies**

### 1NF — First Normal Form
- ทุก column มีค่าเดียว (atomic values)
- ไม่มี repeating groups

```
❌ ไม่ใช่ 1NF
| user_id | name  | phone_numbers       |
|---------|-------|---------------------|
| 1       | Alice | 081-xxx, 089-xxx    |  ← หลายค่าใน cell เดียว

✅ 1NF
| user_id | name  | phone_number |
|---------|-------|--------------|
| 1       | Alice | 081-xxx      |
| 1       | Alice | 089-xxx      |
```

### 2NF — Second Normal Form
- ต้องเป็น 1NF แล้ว
- Non-key columns ต้องขึ้นกับ **ทั้ง** PK (ไม่มี partial dependency)

```
❌ ไม่ใช่ 2NF (composite PK: order_id + product_id)
| order_id | product_id | quantity | product_name |
|          |            |          | ↑ ขึ้นกับแค่ product_id ไม่ใช่ทั้ง PK

✅ แยกเป็น 2 table
orders_items: (order_id, product_id, quantity)
products:     (product_id, product_name)
```

### 3NF — Third Normal Form
- ต้องเป็น 2NF แล้ว
- Non-key columns ต้องไม่ขึ้นกับ non-key column อื่น (ไม่มี transitive dependency)

```
❌ ไม่ใช่ 3NF
| user_id | zip_code | city    |
|         |          | ↑ city ขึ้นกับ zip_code ไม่ใช่ user_id

✅ แยกเป็น 2 table
users: (user_id, zip_code)
zip_codes: (zip_code, city)
```

---

## 4. SQL Queries

### SELECT พื้นฐาน

```sql
-- Basic SELECT
SELECT name, email FROM users WHERE active = true;

-- ORDER + LIMIT
SELECT * FROM products ORDER BY price DESC LIMIT 10;

-- Aggregate functions
SELECT
    department,
    COUNT(*) AS total_employees,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;  -- HAVING กรอง GROUP, WHERE กรอง rows
```

### JOIN Types

```sql
-- INNER JOIN: เอาเฉพาะที่ match ทั้งสอง table
SELECT u.name, o.order_date
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: เอา users ทั้งหมด + orders ที่ match (ถ้าไม่มีคือ NULL)
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- ตัวอย่าง: หา users ที่ยังไม่เคยสั่งซื้อ
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### Subquery

```sql
-- หา users ที่มียอดซื้อสูงกว่าค่าเฉลี่ย
SELECT name
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
    GROUP BY user_id
    HAVING SUM(total) > (SELECT AVG(total) FROM orders)
);

-- Correlated subquery
SELECT name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e1.department  -- อ้างอิง outer query
);
```

### Window Functions (รู้ไว้ดีมาก)

```sql
-- Rank ใน group
SELECT
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept
FROM employees;

-- Running total
SELECT
    order_date,
    total,
    SUM(total) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

---

## 5. Index

### Index คืออะไร?

โครงสร้างข้อมูล (มักเป็น B-Tree) ที่ช่วยให้ database หาข้อมูลเร็วขึ้น แลกกับ disk space และ write performance ที่ช้าลงเล็กน้อย

```sql
-- สร้าง Index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date); -- Composite index

-- ลบ Index
DROP INDEX idx_users_email;
```

### เมื่อไหร่ควรสร้าง Index?

| ✅ ควรสร้าง | ❌ ไม่ควรสร้าง |
|-----------|--------------|
| Column ที่ใช้ใน WHERE บ่อย | Table เล็กมาก |
| Foreign Key columns | Column ที่ query ไม่บ่อย |
| Column ที่ JOIN กัน | Column ที่มีค่าไม่หลากหลาย (boolean) |
| ORDER BY / GROUP BY | Table ที่มี INSERT/UPDATE หนักมาก |

### คำถามสัมภาษณ์

**Q: Index ทำงานอย่างไร?**  
A: Database สร้าง B-Tree structure แยกสำหรับ column นั้น เก็บค่าและ pointer ไปยัง row จริง เมื่อ query ด้วย column ที่มี index ไม่ต้อง scan ทุก row (full table scan) แต่เดิน B-Tree แทน ทำให้ลด time complexity จาก O(n) เหลือ O(log n)

**Q: มี index เยอะเกินดีไหม?**  
A: ไม่ดี เพราะทุก INSERT/UPDATE/DELETE ต้อง update index ด้วย ทำให้ write performance ช้าลง และใช้ disk space มากขึ้น ควรสร้างเฉพาะ index ที่ใช้จริง

---

## 6. Transaction Isolation Levels

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|-----------|---------------------|-------------|
| READ UNCOMMITTED | ✅ อาจเกิด | ✅ อาจเกิด | ✅ อาจเกิด |
| READ COMMITTED | ❌ ไม่เกิด | ✅ อาจเกิด | ✅ อาจเกิด |
| REPEATABLE READ | ❌ | ❌ | ✅ อาจเกิด |
| SERIALIZABLE | ❌ | ❌ | ❌ |

- **Dirty Read** — อ่านข้อมูลที่ transaction อื่น uncommitted
- **Non-repeatable Read** — อ่านค่าเดิม 2 ครั้ง ได้ต่างกัน (เพราะ transaction อื่น update)
- **Phantom Read** — query เดิม 2 ครั้ง ได้จำนวน row ต่างกัน (เพราะ transaction อื่น insert)

**ค่า default ส่วนใหญ่**: MySQL InnoDB = REPEATABLE READ, PostgreSQL = READ COMMITTED

### Transaction ใน Spring

```java
// Default: rollback เมื่อเกิด RuntimeException
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    Account from = accountRepository.findById(fromId).orElseThrow();
    Account to = accountRepository.findById(toId).orElseThrow();

    from.debit(amount);   // ถ้า throw RuntimeException → rollback ทั้งคู่
    to.credit(amount);

    accountRepository.save(from);
    accountRepository.save(to);
}

// กำหนด isolation level
@Transactional(isolation = Isolation.SERIALIZABLE)
public void criticalOperation() { ... }
```

---

## 7. ตัวอย่าง Schema Design

```sql
-- E-commerce schema เบื้องต้น
CREATE TABLE users (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(200) NOT NULL,
    price       DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock       INT NOT NULL DEFAULT 0,
    category_id BIGINT REFERENCES categories(id)
);

CREATE TABLE orders (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT NOT NULL REFERENCES users(id),
    status     VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    total      DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    order_id   BIGINT REFERENCES orders(id),
    product_id BIGINT REFERENCES products(id),
    quantity   INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, product_id)  -- Composite PK
);

-- Indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

---

## สรุป Key Concepts

- **Normalization** — ลด redundancy: 1NF (atomic), 2NF (full dependency), 3NF (no transitive dependency)
- **ACID** — Atomicity, Consistency, Isolation, Durability
- **JOIN** — INNER (ทั้งคู่ match), LEFT (ซ้ายทั้งหมด), RIGHT, FULL OUTER
- **Index** — เร็วขึ้นใน read แต่ช้าลงใน write ใช้บน WHERE/JOIN columns
- **@Transactional** — Spring จัดการ transaction ให้อัตโนมัติ rollback เมื่อ RuntimeException
