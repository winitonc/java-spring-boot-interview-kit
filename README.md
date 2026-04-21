# ชุดเตรียมสัมภาษณ์ Junior Backend Engineer
## Java + Spring Boot Interview Kit

> **เป้าหมาย:** เตรียมความพร้อมสำหรับการสัมภาษณ์ตำแหน่ง Junior Backend Software Engineer  
> **Stack:** Java + Spring Boot | ทีม DevOps/CloudOps ดูแล Deployment บน AWS  
> **ระดับ:** Junior (0–2 ปี หรือ Fresh Graduate)

---

## โครงสร้างชุด

```
Interview/
├── README.md                        ← คุณอยู่ที่นี่
├── 01-java-fundamentals.md          ← Java พื้นฐาน (OOP, Collections, ฯลฯ)
├── 02-spring-boot.md                ← Spring Boot core concepts
├── 03-database-sql.md               ← Database + SQL
├── 04-rest-apis.md                  ← REST API design
├── 05-testing.md                    ← Unit & Integration testing
├── 06-system-design.md              ← System Design เบื้องต้น
├── 07-aws-deployment.md             ← AWS overview (เชิงแนวคิด)
├── 08-behavioral-interview.md       ← Behavioral interview + STAR
├── 09-technical-interview-checklist.md ← Checklist ก่อนสัมภาษณ์
├── 10-mock-questions.md             ← Mock Q&A 25+ ข้อ
├── 11-self-assessment.md            ← ประเมินตนเองก่อนสัมภาษณ์จริง
└── cheat-sheets/
    ├── java-cheatsheet.md
    ├── spring-boot-cheatsheet.md
    ├── sql-cheatsheet.md
    └── rest-api-cheatsheet.md
```

---

## ลำดับการศึกษาแนะนำ

### สำหรับผู้มีเวลา 2+ สัปดาห์
| สัปดาห์ | หัวข้อ | ไฟล์ |
|---------|--------|------|
| 1 | Java พื้นฐาน + Spring Boot | `01` → `02` |
| 1 | Database + REST API | `03` → `04` |
| 2 | Testing + System Design | `05` → `06` |
| 2 | AWS + Behavioral | `07` → `08` |
| ก่อนสัมภาษณ์ | Mock Q&A + Self-assessment | `10` → `11` → `09` |

### สำหรับผู้มีเวลา 3–5 วัน (เร่งด่วน)
1. อ่าน Cheat Sheets ทั้งหมดก่อน (`cheat-sheets/`)
2. ทำ Self-assessment (`11`) เพื่อหาจุดอ่อน
3. อ่านเฉพาะหัวข้อที่ยังไม่แน่ใจ
4. ทำ Mock Q&A (`10`) ให้ครบ
5. ทบทวน Checklist (`09`) วันก่อนสัมภาษณ์

---

## สิ่งที่ต้องรู้ vs ไม่ต้องรู้ (สำหรับ Junior ในบริบทนี้)

### ✅ ต้องรู้ดี
- Java OOP, Collections, Exception handling
- Spring Boot: IoC/DI, REST Controller, JPA
- SQL ทั่วไป, Transaction, Index เบื้องต้น
- REST API design principles
- Unit testing ด้วย JUnit + Mockito
- Git workflow เบื้องต้น

### 📖 รู้แนวคิดได้ (ไม่ต้องลึก)
- System design: Caching, Load balancing, Message queue
- AWS services: EC2, RDS, S3 (รู้ว่าคืออะไร ทำอะไรได้)
- Docker (รู้ว่าคืออะไร แต่ DevOps ดูแล)

### ❌ ยังไม่จำเป็นในระดับ Junior
- Kubernetes, Terraform, CI/CD pipeline configuration
- Advanced microservices patterns
- Performance tuning ขั้นสูง

---

## เคล็ดลับก่อนสัมภาษณ์

1. **พูดดังๆ** — ฝึกอธิบายแนวคิดออกเสียงจริงๆ ไม่ใช่แค่คิดในใจ
2. **ยอมรับเมื่อไม่รู้** — "ผมยังไม่เคยทำ แต่ผมเข้าใจว่ามันทำงานอย่างนี้..." ดีกว่าเดา
3. **ถามเมื่อโจทย์ไม่ชัด** — Clarify requirement ก่อนเขียนโค้ดหรือออกแบบ
4. **อธิบาย trade-off** — Junior ที่ดีรู้ว่าทุกการตัดสินใจมีข้อดีข้อเสีย
5. **เตรียม project ของตัวเอง** — พร้อมอธิบาย design choice ที่เคยทำ
