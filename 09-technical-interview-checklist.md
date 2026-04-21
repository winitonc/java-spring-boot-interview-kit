# 09 — Technical Interview Checklist

> **ใช้:** ทบทวนก่อนสัมภาษณ์จริง 1-2 วัน  
> **เป้าหมาย:** ไม่ลืมสิ่งสำคัญ, เข้าสัมภาษณ์อย่างมั่นใจ

---

## 1. Checklist ความรู้ (ทบทวน)

### Java
- [ ] OOP 4 หลักการ — อธิบายพร้อมตัวอย่างได้
- [ ] Interface vs Abstract Class — บอกได้ว่าเมื่อไหร่ใช้อะไร
- [ ] Collections — ArrayList vs LinkedList, HashMap internals
- [ ] Exception — Checked vs Unchecked, try-with-resources
- [ ] Java 8+ — Lambda, Stream, Optional ใช้งานได้จริง
- [ ] String — immutable, StringBuilder เมื่อไหร่ใช้
- [ ] `equals()` vs `==` — ต่างกันอย่างไร

### Spring Boot
- [ ] IoC/DI — อธิบายได้, บอกได้ว่า Constructor injection ดีกว่าเพราะอะไร
- [ ] Annotations หลัก — @RestController, @Service, @Repository, @Transactional
- [ ] MVC layer — Controller/Service/Repository แต่ละตัวทำอะไร
- [ ] @Transactional — rollback เมื่อไหร่, readOnly ทำอะไร
- [ ] Spring Data JPA — findById, save, derived query methods
- [ ] @SpringBootApplication = @Configuration + ?
- [ ] DTO — คืออะไร ทำไมต้องใช้

### Database & SQL
- [ ] ACID — อธิบายได้ทุกตัว
- [ ] Normalization — 1NF, 2NF, 3NF ต่างกันอย่างไร
- [ ] SQL — INNER JOIN, LEFT JOIN, GROUP BY, HAVING
- [ ] Index — ทำงานอย่างไร เมื่อไหร่ควรสร้าง
- [ ] Transaction Isolation Level — READ COMMITTED คืออะไร

### REST API
- [ ] HTTP Methods — GET/POST/PUT/PATCH/DELETE ใช้เมื่อไหร่
- [ ] Status Codes — 200, 201, 204, 400, 401, 403, 404, 500
- [ ] Idempotent — หมายความว่าอะไร ตัวไหน idempotent บ้าง
- [ ] 401 vs 403 — ต่างกันอย่างไร
- [ ] URL naming — resource-based, plural, nested resources
- [ ] Stateless — คืออะไร ทำไม REST ต้องเป็น stateless

### Testing
- [ ] Test Pyramid — Unit/Integration/E2E ต่างกันอย่างไร
- [ ] JUnit 5 — @Test, @BeforeEach, @ParameterizedTest
- [ ] Mockito — @Mock, @InjectMocks, when().thenReturn(), verify()
- [ ] @WebMvcTest vs @SpringBootTest — เมื่อไหร่ใช้อะไร
- [ ] Mock vs Stub — ต่างกันอย่างไร

### System Design (แนวคิด)
- [ ] Vertical vs Horizontal scaling — ต่างกันอย่างไร
- [ ] Cache — Cache-aside strategy คืออะไร
- [ ] Load Balancer — กระจาย traffic อย่างไร
- [ ] Stateless app — ทำไม stateless ถึง scale ได้ดีกว่า
- [ ] Message Queue — use case หลักคืออะไร

### AWS (แนวคิด)
- [ ] EC2, RDS, S3, ElastiCache, ELB, CloudWatch — อธิบายสั้นๆ ได้
- [ ] ทำไมใช้ environment variable แทน hardcode
- [ ] Health check endpoint — ทำไม ELB ต้องการ

---

## 2. Checklist เตรียมตัว

### 1 สัปดาห์ก่อน
- [ ] ทบทวน project ที่ทำ — พร้อมอธิบาย technical decision ที่เลือก
- [ ] เตรียม 5-7 behavioral stories (STAR format)
- [ ] ศึกษาข้อมูลบริษัท — product, tech stack, culture
- [ ] เตรียมคำถามถาม interviewer 3-5 ข้อ
- [ ] ทำ mock interview กับเพื่อนหรือตัวเอง

### วันก่อนสัมภาษณ์
- [ ] ทบทวน Cheat Sheets (`cheat-sheets/`)
- [ ] ทำ Mock Questions (`10-mock-questions.md`) ให้ครบ
- [ ] เตรียม code examples ที่เคยเขียน (GitHub profile)
- [ ] ตรวจสอบ link สัมภาษณ์ / address ถ้า onsite
- [ ] นอนหลับพักผ่อนให้เพียงพอ

### วันสัมภาษณ์
- [ ] เข้าถึงก่อน 10-15 นาที
- [ ] เตรียมน้ำ, เงียบสงบ
- [ ] ปิดการแจ้งเตือน device

---

## 3. ระหว่างสัมภาษณ์ — Do's & Don'ts

### ✅ Do's
- **Clarify ก่อน code** — ถามให้ชัดก่อนเริ่ม "ต้องการ input แบบไหน? มี edge case ไหม?"
- **Think aloud** — พูดกระบวนการคิดออกมา interviewer อยากรู้ว่าคุณคิดอย่างไร
- **Trade-offs** — เสนอ solution แล้วพูดถึง trade-off เช่น "ทำแบบนี้เร็วกว่าแต่ใช้ memory มากกว่า"
- **ยอมรับเมื่อไม่รู้** — "ผมไม่แน่ใจเรื่อง X แต่ผมเข้าใจว่ามันทำงานประมาณนี้..." ดีกว่าเดาผิด
- **ถามคำถามกลับ** — แสดงความสนใจใน role และทีม
- **Code อ่านง่าย** — ตั้งชื่อ variable ที่สื่อความหมาย, แบ่ง function ถ้า logic ซับซ้อน

### ❌ Don'ts
- อย่า panic เมื่อโจทย์ยาก — หายใจ, แยก problem ออกเป็นส่วนย่อย
- อย่าเงียบนานโดยไม่บอกว่ากำลังคิดอะไร
- อย่า over-engineer — solution ง่ายที่ work ดีกว่า complex solution ที่ยังไม่ work
- อย่าวิจารณ์บริษัทเก่า/ทีมเก่าในแง่ลบ
- อย่า low-ball ตัวเอง — เชื่อมั่นในความสามารถของตัวเอง
- อย่าลืมถามคำถามกลับ (แสดงว่าไม่สนใจ)

---

## 4. Coding Interview Tips

### เมื่อเจอโจทย์
```
1. อ่านโจทย์ให้ครบ
2. ถาม clarifying questions:
   - Input size เท่าไหร่?
   - มี edge case ไหม? (null, empty, negative)
   - ต้องการ optimal หรือ working solution?
3. พูด approach ก่อน code
4. เริ่ม code พร้อมพูดออกมา
5. Test ด้วย example จากโจทย์ + edge cases
6. วิเคราะห์ time/space complexity
```

### เมื่อติด
```
- พูดว่าติดตรงไหน และกำลังคิดอะไรอยู่
- ลอง brute force ก่อน แล้วค่อย optimize
- ถาม hint ได้ — interviewer ยินดีช่วย
```

---

## 5. ข้อผิดพลาดที่พบบ่อย

| ข้อผิดพลาด | แก้อย่างไร |
|-----------|----------|
| ตอบสั้นเกินไป ไม่มี context | ใช้ STAR, เพิ่ม detail |
| Code โดยไม่ clarify โจทย์ | ถาม 2-3 คำถามก่อนเสมอ |
| เงียบนานเมื่อติด | Think aloud ตลอด |
| ตอบ "เราทำ..." แทน "ฉันทำ..." | ระวัง pronoun ใน behavioral Q |
| ลืม edge cases | check null, empty, boundary |
| ไม่ถามคำถามกลับ | เตรียม 3 ข้อไว้เสมอ |
| Over-explain จนหลุดประเด็น | จับ key point แล้วถาม "ตอบตรงประเด็นไหม?" |
