# 10 — Mock Interview Questions & Answers

> **วิธีใช้:** อ่านคำถาม ลองตอบออกเสียงก่อน แล้วค่อยเปิดดูคำตอบ  
> **เป้าหมาย:** ฝึก articulate ความรู้ออกมาได้ชัดเจน ไม่ใช่แค่รู้ในใจ

---

## หมวด Java Fundamentals

**Q1: อธิบาย OOP 4 หลักการ พร้อมตัวอย่างจริง**

> **แนวตอบ:** Encapsulation — ซ่อน field เป็น private ใช้ getter/setter ควบคุม access เช่น `User.setAge()` ตรวจสอบ age > 0 ก่อน set / Inheritance — `class Manager extends Employee` ให้ Manager มี method ของ Employee ทั้งหมด / Polymorphism — `List<Animal> animals` เก็บ Dog, Cat ได้ เรียก `sound()` แต่ละตัวทำงานต่างกัน / Abstraction — interface `PaymentGateway` ซ่อน implementation ว่าจะ charge บัตรอย่างไร

---

**Q2: HashMap ทำงานอย่างไร? ทำไม key ต้อง implement hashCode() และ equals()?**

> **แนวตอบ:** HashMap ใช้ array ภายใน เมื่อ put(key, value) จะเรียก key.hashCode() แปลงเป็น index ของ array เพื่อเก็บ value เมื่อ get(key) ก็ hash key เดิมเพื่อหา index ถ้า hash ชนกัน (collision) จะเก็บ linked list หรือ tree ที่ index เดียวกัน จึงต้องใช้ equals() เพื่อหา exact key hashCode() ต้อง consistent กับ equals() — ถ้า a.equals(b) แล้ว a.hashCode() == b.hashCode() ต้องเป็นจริงเสมอ

---

**Q3: Checked Exception กับ Unchecked Exception ต่างกันอย่างไร?**

> **แนวตอบ:** Checked Exception เช่น IOException, SQLException คือ exception ที่ caller ต้องจัดการ (try-catch หรือ throws) เหมาะกับสถานการณ์ที่ recover ได้ เช่น file ไม่พบ / Unchecked Exception เป็น RuntimeException subclass เช่น NullPointerException, IllegalArgumentException มักเป็น programming error ไม่บังคับ catch แต่ควร fix ที่ต้นเหตุ

---

**Q4: Optional ใช้ทำอะไร ดีกว่า null check อย่างไร?**

> **แนวตอบ:** Optional เป็น wrapper ที่บอกชัดว่า value อาจไม่มี บังคับให้ caller จัดการ null case ผ่าน API เช่น orElse(), orElseThrow() แทน if (x != null) ทุกที่ ช่วยป้องกัน NullPointerException และทำให้ intent ของ code ชัดขึ้น เช่น `repo.findById(id).orElseThrow(() -> new NotFoundException(id))`

---

## หมวด Spring Boot

**Q5: Spring IoC Container คืออะไร? อธิบาย Dependency Injection**

> **แนวตอบ:** IoC Container คือ core ของ Spring จัดการ lifecycle และ wiring ของ Bean แทนที่ code จะสร้าง object dependency เอง Spring inject ให้ผ่าน constructor/setter/field ข้อดีคือ loose coupling ทดสอบง่าย เพราะแทน dependency จริงด้วย mock ได้ ตัวอย่าง: UserController ไม่ต้อง new UserService() เอง Spring inject UserService ที่ถูก configure แล้วให้

---

**Q6: ทำไม Constructor Injection ดีกว่า Field Injection?**

> **แนวตอบ:** Constructor injection บังคับให้ระบุ dependency ตอนสร้าง object ทำให้ field เป็น final ได้ (immutable, ไม่มีทางเป็น null หลัง construct) ทดสอบง่ายเพราะ inject mock ผ่าน constructor โดยตรงโดยไม่ต้องพึ่ง reflection ส่วน field injection ใช้ reflection ซ่อน dependency ทำให้รู้ได้แค่ตอน runtime และทดสอบลำบาก

---

**Q7: @Transactional ทำงานอย่างไร จะ rollback เมื่อไหร่?**

> **แนวตอบ:** Spring สร้าง proxy รอบ method ที่มี @Transactional เปิด transaction ก่อน method ทำงาน commit เมื่อ method return สำเร็จ และ rollback อัตโนมัติเมื่อเกิด RuntimeException (unchecked) สำหรับ Checked Exception จะไม่ rollback โดยค่าเริ่มต้น ต้องระบุ `rollbackFor = IOException.class` ถ้าต้องการ

---

**Q8: DTO คืออะไร ทำไมต้องแยก Entity กับ DTO?**

> **แนวตอบ:** DTO (Data Transfer Object) คือ object สำหรับส่งข้อมูลระหว่าง layer เหตุผลที่แยก: (1) Security — ไม่ expose field ที่ไม่ควรส่งออก เช่น password hash (2) Decoupling — เปลี่ยน DB schema ได้โดยไม่กระทบ API contract (3) Validation — ใส่ validation annotation ใน DTO แยกจาก Entity (4) Versioning — API v1 และ v2 ใช้ DTO ต่างกันได้

---

## หมวด Database & SQL

**Q9: อธิบาย ACID ให้ฟัง พร้อมตัวอย่าง**

> **แนวตอบ:** ใช้ตัวอย่างโอนเงิน: Atomicity — ถ้าตัดจากบัญชี A แล้ว server crash ก่อนเพิ่มบัญชี B ต้อง rollback ทั้งคู่ / Consistency — ยอดรวมก่อนและหลังโอนต้องเท่ากัน / Isolation — 2 คนโอนเงินพร้อมกันต้องไม่เห็น intermediate state ของกัน / Durability — commit แล้วข้อมูลไม่หายแม้ server restart

---

**Q10: Index คืออะไร มีข้อดี-ข้อเสียอะไร?**

> **แนวตอบ:** Index คือ data structure (B-Tree) แยกต่างหาก เก็บค่าของ column + pointer ไปยัง row จริง ช่วยลด query time จาก O(n) full scan เป็น O(log n) B-Tree search / ข้อดี: read เร็วขึ้นมาก / ข้อเสีย: ทุก INSERT/UPDATE/DELETE ต้อง update index ด้วย ทำให้ write ช้าลง + ใช้ disk เพิ่ม ควรสร้างเฉพาะ index ที่ใช้จริงบน WHERE/JOIN columns

---

**Q11: เขียน SQL หา top 3 products ที่มียอดขายสูงสุดในแต่ละ category**

> **แนวตอบ:**
```sql
SELECT category, product_name, total_sales
FROM (
    SELECT
        p.category,
        p.name AS product_name,
        SUM(oi.quantity * oi.unit_price) AS total_sales,
        RANK() OVER (
            PARTITION BY p.category
            ORDER BY SUM(oi.quantity * oi.unit_price) DESC
        ) AS rnk
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.category, p.id, p.name
) ranked
WHERE rnk <= 3
ORDER BY category, rnk;
```

---

## หมวด REST API

**Q12: PUT กับ PATCH ต่างกันอย่างไร ให้ยกตัวอย่าง**

> **แนวตอบ:** PUT แทนที่ resource ทั้งหมด ต้องส่งทุก field ถ้า field ไหนไม่ส่งจะเป็น null/default PATCH แก้เฉพาะ field ที่ส่งมา field อื่นไม่เปลี่ยน ตัวอย่าง: อัปเดต email อย่างเดียว ด้วย PATCH ส่งแค่ `{"email": "new@email.com"}` ด้วย PUT ต้องส่ง name, age, phone ทุก field ด้วย ไม่งั้นหาย

---

**Q13: อธิบาย Idempotent คืออะไร HTTP method ไหนบ้างที่ Idempotent?**

> **แนวตอบ:** Idempotent หมายถึงเรียก operation เดิมหลายครั้ง ผลลัพธ์เหมือนกับเรียกครั้งเดียว GET, PUT, DELETE เป็น idempotent เพราะ GET อ่านไม่เปลี่ยน, PUT เขียนค่าเดิมซ้ำๆ ผลเหมือนกัน, DELETE ลบแล้วลบซ้ำก็ไม่มีอะไรเปลี่ยน POST ไม่ idempotent เพราะ POST /orders สองครั้งสร้าง 2 orders สำคัญใน network retry — safe to retry idempotent operations

---

**Q14: ออกแบบ Error Response Format ที่ดีเป็นอย่างไร?**

> **แนวตอบ:** Error response ควรมี: HTTP status code ที่ถูกต้อง, error code (machine-readable), human-readable message, timestamp, request path ตัวอย่าง: `{"status":400, "error":"VALIDATION_FAILED", "message":"...", "errors":[{"field":"email","message":"Invalid"}], "path":"/api/users", "timestamp":"..."}` สิ่งที่ไม่ควรใส่: stack trace (ข้อมูล internal), implementation details

---

## หมวด Testing

**Q15: อธิบาย Test Pyramid และทำไมถึงควรมี unit test มากที่สุด?**

> **แนวตอบ:** Test Pyramid มี 3 ชั้น: Unit (ล่าง) → Integration → E2E (บน) Unit test มากที่สุดเพราะ: (1) เร็วมาก รัน milliseconds ทำให้ feedback loop สั้น (2) ถูก ไม่ต้องการ infrastructure (3) Isolate failure ได้ดี รู้ทันทีว่า method ไหนผิด Integration test ช้าและซับซ้อนกว่า ใช้ confirm ว่า components ทำงานร่วมกันได้ E2E ช้าและ flaky ที่สุด ใช้แค่ critical paths

---

**Q16: อธิบายความแตกต่างระหว่าง @WebMvcTest กับ @SpringBootTest**

> **แนวตอบ:** @WebMvcTest โหลดเฉพาะ web layer (Controller, Filter, HandlerMapping) ไม่โหลด Service/Repository ต้อง @MockBean สำหรับ dependencies เร็วมากเหมาะ unit test Controller @SpringBootTest โหลด application context ทั้งหมดเหมือน run จริง ช้ากว่าแต่ test ว่าทุก component ทำงานร่วมกันจริงๆ ใช้สำหรับ integration test หรือ end-to-end test

---

## หมวด System Design

**Q17: อธิบาย Caching Strategy — Cache-aside คืออะไร?**

> **แนวตอบ:** Cache-aside (Lazy Loading) คือ app จัดการ cache เอง เมื่อ read: ดู cache ก่อน ถ้ามี (hit) return เลย ถ้าไม่มี (miss) query DB แล้วเก็บ cache แล้ว return เมื่อ write: update DB แล้ว invalidate cache (ลบ) เพื่อให้ read ครั้งหน้า fetch ใหม่จาก DB ข้อดี: ง่าย resilient ถ้า cache ล่มก็ query DB แทน ข้อเสีย: cache miss ครั้งแรกช้า, อาจมี stale data ช่วงสั้นๆ

---

**Q18: ทำไม stateless API ถึง scale ได้ดีกว่า?**

> **แนวตอบ:** Stateless หมายถึง server ไม่เก็บ state ของ client ไว้ใน memory แต่ละ request มีข้อมูลครบในตัว (เช่น JWT token) ทำให้สามารถรัน multiple server instances ได้โดย load balancer route request ไปตัวไหนก็ได้ ถ้า stateful request ของ user คนเดียวต้องไปถึง server เดิมเสมอ (sticky session) ทำให้ scale ยาก ถ้า server นั้น crash user ต้อง login ใหม่

---

## หมวด AWS & Deployment

**Q19: อธิบาย EC2, RDS, S3 คืออะไร ใช้ทำอะไร?**

> **แนวตอบ:** EC2 คือ virtual machine บน cloud ใช้รัน application เช่น Spring Boot JAR — เหมือนเช่าเครื่อง server / RDS คือ managed relational database (PostgreSQL, MySQL) AWS ดูแล backup, patching, high availability ให้ เราแค่ connect และ query / S3 คือ object storage เก็บได้ทุกประเภทไฟล์ (รูป, PDF, video, backup) capacity แทบไม่จำกัด เหมาะเก็บ user-uploaded files

---

**Q20: ทำไมต้องใช้ Environment Variables สำหรับ config? อย่าง DB password?**

> **แนวตอบ:** 3 เหตุผลหลัก: (1) Security — ไม่ commit secret ลง git repository ที่อาจ public (2) Flexibility — ใช้ config ต่างกันใน dev/staging/prod โดยไม่ต้องแก้ code หรือ rebuild (3) 12-factor App principle — config เป็นสิ่งที่ต่างกันระหว่าง environments ควรแยกออกจาก code ทีม DevOps จัดการ inject env var ผ่าน deployment pipeline หรือ AWS Secrets Manager

---

## โจทย์เขียนโค้ด

**Q21: เขียน method หา element ที่ซ้ำกันใน List**

```java
public List<Integer> findDuplicates(List<Integer> nums) {
    Set<Integer> seen = new HashSet<>();
    Set<Integer> duplicates = new HashSet<>();
    for (int n : nums) {
        if (!seen.add(n)) { // add() returns false ถ้าซ้ำ
            duplicates.add(n);
        }
    }
    return new ArrayList<>(duplicates);
}
// Time: O(n), Space: O(n)
```

---

**Q22: เขียน REST endpoint + service สำหรับ get user by id พร้อม error handling**

```java
// Controller
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    UserResponse user = userService.getUserById(id);
    return ResponseEntity.ok(user);
}

// Service
public UserResponse getUserById(Long id) {
    return userRepository.findById(id)
        .map(user -> new UserResponse(user.getId(), user.getName(), user.getEmail()))
        .orElseThrow(() -> new UserNotFoundException(id));
}

// Exception — จะถูก catch โดย GlobalExceptionHandler → 404
```

---

**Q23: เขียน Unit Test สำหรับ UserService.getUserById()**

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock UserRepository userRepository;
    @InjectMocks UserService userService;

    @Test
    void shouldReturnUser_whenFound() {
        User user = new User(1L, "Alice", "alice@test.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        UserResponse result = userService.getUserById(1L);

        assertEquals("Alice", result.getName());
        verify(userRepository).findById(1L);
    }

    @Test
    void shouldThrow_whenNotFound() {
        when(userRepository.findById(anyLong())).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class,
            () -> userService.getUserById(99L));
    }
}
```

---

**Q24: อธิบายสิ่งที่ผิดพลาดในโค้ดนี้**

```java
// โค้ดนี้มีปัญหาอะไร?
@RestController
public class UserController {
    private UserService userService = new UserService(new UserRepository());

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return user; // อาจ return null
    }
}
```

> **แนวตอบ:**  
> 1. สร้าง dependency เอง (`new`) → tight coupling, ทดสอบยาก ควรใช้ DI  
> 2. return null แทน 404 → ควรใช้ Optional + throw NotFoundException  
> 3. `new UserRepository()` ข้างใน UserService อีกทีด้วย → cascade tight coupling  
> 4. ไม่มี error handling

---

**Q25: ออกแบบ API สำหรับ Blog system เบื้องต้น (URL + methods)**

> **แนวตอบ:**
```
GET    /api/posts                  → list posts (with pagination)
POST   /api/posts                  → create post (auth required)
GET    /api/posts/{id}             → get one post
PUT    /api/posts/{id}             → update post (author only)
DELETE /api/posts/{id}             → delete post (author only)
GET    /api/posts/{id}/comments    → list comments
POST   /api/posts/{id}/comments    → add comment (auth required)
DELETE /api/comments/{id}          → delete comment

Query params:
GET /api/posts?page=0&size=10&sort=createdAt,desc
GET /api/posts?tag=java&author=alice
```
