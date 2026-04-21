# 11 — Self-Assessment

> **วิธีใช้:** ทำก่อนสัมภาษณ์จริง 1-3 วัน ตรงไปตรงมากับตัวเอง  
> **เป้าหมาย:** หาจุดอ่อนที่ต้องทบทวน ไม่ใช่วัดคะแนน

---

## วิธีประเมิน

ให้คะแนนตัวเองแต่ละหัวข้อ:
- **3** — อธิบายได้ชัดเจน ยกตัวอย่างได้ เขียนโค้ดได้ทันที
- **2** — เข้าใจแนวคิด แต่ต้องคิดก่อน หรืออาจติดรายละเอียด
- **1** — รู้ว่าคืออะไรคร่าวๆ แต่อธิบายได้ไม่ชัด
- **0** — ไม่รู้เลย หรือไม่แน่ใจมาก

**เป้าหมาย:** ทุกหัวข้อควรได้ 2+ ก่อนสัมภาษณ์ ถ้าได้ 0-1 ตรงไหน → กลับไปอ่านบทนั้น

---

## Java Fundamentals

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| OOP 4 หลักการ + ตัวอย่าง | | |
| Interface vs Abstract class | | |
| Polymorphism (overriding vs overloading) | | |
| ArrayList vs LinkedList — เมื่อไหร่ใช้อะไร | | |
| HashMap internals (hashing, collision) | | |
| Checked vs Unchecked Exception | | |
| Try-with-resources | | |
| String immutability + StringBuilder | | |
| equals() vs == | | |
| Lambda expression syntax | | |
| Stream API (.filter, .map, .collect) | | |
| Optional — orElse, orElseThrow, map | | |
| Builder pattern | | |

**คะแนนรวม Java:** _____ / 39  
**หัวข้อที่ต้องทบทวน:** _______________

---

## Spring Boot

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| IoC / Dependency Injection concept | | |
| Constructor vs Field injection | | |
| @Component, @Service, @Repository ต่างกันอย่างไร | | |
| @RestController vs @Controller | | |
| @GetMapping, @PostMapping ฯลฯ | | |
| @PathVariable, @RequestParam, @RequestBody | | |
| @Transactional — rollback เมื่อไหร่ | | |
| Spring Data JPA — JpaRepository methods | | |
| Derived query methods (findBy...) | | |
| @Bean vs @Component | | |
| @ConfigurationProperties | | |
| GlobalExceptionHandler (@RestControllerAdvice) | | |
| DTO — คืออะไร ทำไมต้องใช้ | | |
| application.yml — env var injection | | |

**คะแนนรวม Spring Boot:** _____ / 42  
**หัวข้อที่ต้องทบทวน:** _______________

---

## Database & SQL

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| ACID — อธิบายได้ครบ 4 ข้อ | | |
| Primary Key vs Foreign Key vs Unique Key | | |
| 1NF, 2NF, 3NF — ต่างกันอย่างไร | | |
| SELECT + WHERE + ORDER BY + LIMIT | | |
| GROUP BY + HAVING | | |
| INNER JOIN vs LEFT JOIN | | |
| Subquery | | |
| Index — ทำงานอย่างไร ข้อดีข้อเสีย | | |
| เมื่อไหร่ควร/ไม่ควร สร้าง Index | | |
| Transaction Isolation Level (READ COMMITTED) | | |
| @Transactional ใน Spring + rollback | | |
| One-to-Many, Many-to-Many relationships | | |

**คะแนนรวม Database:** _____ / 36  
**หัวข้อที่ต้องทบทวน:** _______________

---

## REST API

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| HTTP Methods — GET/POST/PUT/PATCH/DELETE | | |
| PUT vs PATCH | | |
| Idempotent — คืออะไร ตัวไหนบ้าง | | |
| HTTP Status Codes 2xx | | |
| HTTP Status Codes 4xx (400,401,403,404,409) | | |
| 401 vs 403 ต่างกันอย่างไร | | |
| URL naming — resource-based, plural | | |
| Nested resources URL design | | |
| Stateless — คืออะไร ทำไมสำคัญ | | |
| REST vs SOAP | | |
| Pagination via query params | | |
| Error response format | | |

**คะแนนรวม REST API:** _____ / 36  
**หัวข้อที่ต้องทบทวน:** _______________

---

## Testing

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| Test Pyramid — Unit/Integration/E2E | | |
| JUnit 5 — @Test, @BeforeEach, @ParameterizedTest | | |
| Assertions — assertEquals, assertThrows | | |
| Mockito — @Mock, @InjectMocks | | |
| when().thenReturn() | | |
| verify() — ตรวจสอบ method calls | | |
| Mock vs Stub ต่างกันอย่างไร | | |
| @WebMvcTest — ใช้เมื่อไหร่ | | |
| @SpringBootTest — ใช้เมื่อไหร่ | | |
| @DataJpaTest — ใช้เมื่อไหร่ | | |
| MockMvc — perform/andExpect | | |
| TDD — Red/Green/Refactor cycle | | |

**คะแนนรวม Testing:** _____ / 36  
**หัวข้อที่ต้องทบทวน:** _______________

---

## System Design

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| Vertical vs Horizontal scaling | | |
| Cache — Cache-aside strategy | | |
| Cache TTL และ invalidation | | |
| Load Balancer — role และ algorithms | | |
| Stateless app — ทำไม scale ได้ดีกว่า | | |
| Read Replica — คืออะไร | | |
| Connection Pool — คืออะไร | | |
| Message Queue — use cases | | |
| Monolith vs Microservices (concept) | | |

**คะแนนรวม System Design:** _____ / 27  
**หัวข้อที่ต้องทบทวน:** _______________

---

## AWS (แนวคิด)

| หัวข้อ | คะแนน (0-3) | หมายเหตุ |
|--------|------------|---------|
| EC2 คืออะไร | | |
| RDS คืออะไร ดีกว่า self-managed DB อย่างไร | | |
| S3 คืออะไร ใช้เก็บอะไร | | |
| ElastiCache คืออะไร | | |
| ELB คืออะไร | | |
| CloudWatch คืออะไร | | |
| ทำไมใช้ env var แทน hardcode | | |
| Health check endpoint คืออะไร | | |

**คะแนนรวม AWS:** _____ / 24  
**หัวข้อที่ต้องทบทวน:** _______________

---

## สรุปคะแนนรวม

| หมวด | คะแนน | เต็ม | % |
|------|-------|------|---|
| Java Fundamentals | | 39 | |
| Spring Boot | | 42 | |
| Database & SQL | | 36 | |
| REST API | | 36 | |
| Testing | | 36 | |
| System Design | | 27 | |
| AWS | | 24 | |
| **รวม** | | **240** | |

---

## แผนทบทวน (กรอกเอง)

### หัวข้อที่ได้ 0-1 (ต้องทบทวนก่อนสัมภาษณ์):
```
1. 
2. 
3. 
```

### หัวข้อที่ได้ 2 (ทบทวนเพิ่มเติมถ้ามีเวลา):
```
1. 
2. 
3. 
```

### สิ่งที่มั่นใจแล้ว (ใช้เป็น opening ใน interview ได้):
```
1. 
2. 
3. 
```

---

## ✅ พร้อมสัมภาษณ์เมื่อ...

- [ ] ทุกหัวข้อได้ 2+ หรือมีแผนทบทวนหัวข้อที่ได้ต่ำกว่า
- [ ] ทำ Mock Questions (`10-mock-questions.md`) ครบแล้ว
- [ ] เตรียม behavioral stories 5 เรื่อง (STAR format)
- [ ] มีคำถามถาม interviewer 3 ข้อ
- [ ] ทบทวน project ของตัวเองแล้ว พร้อมอธิบาย design decisions
