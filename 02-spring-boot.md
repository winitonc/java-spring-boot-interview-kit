# 02 — Spring Boot

> **เป้าหมาย:** เข้าใจ Spring Boot core concepts, IoC/DI, MVC layer, และ JPA เบื้องต้น  
> **หัวข้อ:** IoC/DI · Annotations · MVC Architecture · Spring Data JPA · Configuration · Exception Handling

---

## 1. Spring Boot คืออะไร?

Spring Boot คือ framework ที่สร้างบน Spring Framework ช่วยลด configuration ด้วย **Auto-configuration** และ **Convention over Configuration** ทำให้เริ่มต้นสร้าง production-ready app ได้เร็ว

### ข้อดีหลัก
- **Embedded server** — ไม่ต้อง deploy WAR ลง Tomcat แยก, run เป็น JAR ได้เลย
- **Auto-configuration** — detect dependencies และ configure อัตโนมัติ
- **Starter dependencies** — `spring-boot-starter-web`, `spring-boot-starter-data-jpa` รวม dependencies ที่จำเป็น
- **Actuator** — built-in health check, metrics endpoints

---

## 2. IoC และ Dependency Injection

### IoC (Inversion of Control)
แทนที่ code จะสร้าง object เอง ให้ **Spring Container** สร้างและจัดการ lifecycle ให้

```java
// ❌ แบบไม่ใช้ IoC — tight coupling
public class OrderService {
    private UserRepository userRepository = new UserRepository(); // สร้างเอง
}

// ✅ แบบใช้ DI — loose coupling
@Service
public class OrderService {
    private final UserRepository userRepository;

    // Constructor injection (แนะนำ)
    public OrderService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### วิธี Inject (3 แบบ)

```java
// 1. Constructor injection ✅ แนะนำที่สุด
@Service
public class OrderService {
    private final UserRepository userRepository;

    public OrderService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 2. Setter injection — ใช้เมื่อ dependency เป็น optional
@Service
public class NotificationService {
    private EmailSender emailSender;

    @Autowired
    public void setEmailSender(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}

// 3. Field injection ❌ ไม่แนะนำ (ทดสอบยาก, ซ่อน dependency)
@Service
public class BadService {
    @Autowired
    private UserRepository userRepository; // ใช้ reflection, ไม่ดี
}
```

### ทำไม Constructor injection ดีที่สุด?
- `final` field — dependency ไม่สามารถ null หลัง construct
- ทดสอบง่าย — inject mock ผ่าน constructor โดยตรง
- ชัดเจนว่า class ต้องการอะไร

---

## 3. Annotations หลักที่ต้องรู้

### Stereotype Annotations

| Annotation | ใช้ใน Layer | ความหมาย |
|-----------|-----------|---------|
| `@Component` | ทั่วไป | Generic Spring Bean |
| `@Service` | Business Logic | เหมือน @Component + บ่งบอก layer |
| `@Repository` | Data Access | เหมือน @Component + exception translation |
| `@Controller` | Presentation (MVC) | Return view |
| `@RestController` | REST API | = @Controller + @ResponseBody |

### Configuration Annotations

```java
@SpringBootApplication // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

@Configuration
public class AppConfig {
    @Bean // สร้าง bean ให้ Spring Container จัดการ
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

### Request Mapping

```java
@RestController
@RequestMapping("/api/users") // base path
public class UserController {

    @GetMapping("/{id}")           // GET /api/users/{id}
    @PostMapping                   // POST /api/users
    @PutMapping("/{id}")           // PUT /api/users/{id}
    @PatchMapping("/{id}")         // PATCH /api/users/{id}
    @DeleteMapping("/{id}")        // DELETE /api/users/{id}
}
```

---

## 4. MVC Architecture — Project Structure

### Layer Architecture (Layered/N-Tier)

```
Request → Controller → Service → Repository → Database
                ↓
           Response ←────────────────────────
```

### Project Structure แนะนำ

```
src/main/java/com/example/myapp/
├── MyAppApplication.java           ← Entry point
├── controller/
│   └── UserController.java         ← รับ request, return response
├── service/
│   ├── UserService.java            ← Interface (optional)
│   └── UserServiceImpl.java        ← Business logic
├── repository/
│   └── UserRepository.java         ← Data access (extends JpaRepository)
├── model/                          (หรือ entity/)
│   └── User.java                   ← JPA Entity / Domain object
├── dto/
│   ├── UserRequestDto.java         ← Input DTO
│   └── UserResponseDto.java        ← Output DTO
└── exception/
    ├── UserNotFoundException.java
    └── GlobalExceptionHandler.java
```

### ตัวอย่าง: Full CRUD

```java
// Model / Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    // getters, setters (หรือใช้ Lombok @Getter @Setter)
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email); // Spring generates query automatically
    List<User> findByNameContainingIgnoreCase(String name);
}

// Service
@Service
@Transactional
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    public User createUser(UserRequestDto dto) {
        if (userRepository.findByEmail(dto.getEmail()).isPresent()) {
            throw new IllegalArgumentException("Email already exists");
        }
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        return userRepository.save(user);
    }
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserRequestDto dto) {
        User created = userService.createUser(dto);
        URI location = URI.create("/api/users/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 5. Spring Data JPA

### JpaRepository Methods ที่มีให้ฟรี

```java
// CRUD พื้นฐาน
repository.findById(id)          // Optional<T>
repository.findAll()             // List<T>
repository.save(entity)          // T (insert หรือ update)
repository.deleteById(id)        // void
repository.existsById(id)        // boolean
repository.count()               // long

// Paging
Page<User> page = repository.findAll(PageRequest.of(0, 10, Sort.by("name")));
```

### Query Methods (Derived Query)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring แปลงชื่อ method เป็น SQL อัตโนมัติ
    List<User> findByNameAndEmail(String name, String email);
    List<User> findByAgeGreaterThan(int age);
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);

    // Custom JPQL
    @Query("SELECT u FROM User u WHERE u.name LIKE %:keyword%")
    List<User> searchByName(@Param("keyword") String keyword);

    // Native SQL
    @Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
    Optional<User> findByEmailNative(@Param("email") String email);
}
```

### @Transactional

```java
@Service
public class OrderService {

    @Transactional // ถ้า exception เกิดใน method นี้ → rollback ทั้งหมด
    public Order createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(...));
        inventoryService.deductStock(request.getItems()); // ถ้า fail → rollback
        paymentService.charge(request.getPayment());      // ถ้า fail → rollback
        return order;
    }

    @Transactional(readOnly = true) // optimize read-only operations
    public List<Order> getOrders() {
        return orderRepository.findAll();
    }
}
```

---

## 6. Configuration (application.properties / application.yml)

```yaml
# application.yml
spring:
  application:
    name: my-backend-service

  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}       # อ่านจาก environment variable
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: validate        # production: validate (ไม่แก้ schema)
    show-sql: false
    open-in-view: false         # แนะนำให้ปิด

server:
  port: 8080

# Custom properties
app:
  jwt:
    secret: ${JWT_SECRET}
  pagination:
    default-size: 20
```

### อ่าน Custom Properties

```java
@Component
@ConfigurationProperties(prefix = "app.pagination")
public class PaginationConfig {
    private int defaultSize = 10; // default value

    // getter + setter
}

// หรือใช้ @Value
@Value("${app.pagination.default-size:10}") // :10 คือ default
private int defaultPageSize;
```

---

## 7. Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex) {
        ErrorResponse error = new ErrorResponse("BAD_REQUEST", ex.getMessage());
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
        return ResponseEntity.internalServerError().body(error);
    }
}
```

---

## คำถามสัมภาษณ์รวม

**Q: Spring IoC Container คืออะไร?**  
A: Container ที่จัดการ lifecycle ของ Bean ตั้งแต่สร้างจนถึงทำลาย รับผิดชอบ inject dependency ระหว่าง Bean ตาม configuration ที่กำหนด

**Q: @Component กับ @Bean ต่างกันอย่างไร?**  
A: `@Component` ใส่ที่ class และ Spring scan + สร้าง bean อัตโนมัติ ส่วน `@Bean` ใส่ใน method ใน `@Configuration` class ใช้เมื่อต้องการ control การสร้าง object เอง เช่น configure third-party library

**Q: @Service, @Repository, @Controller ต่างจาก @Component อย่างไร?**  
A: ทำงานเหมือน @Component แต่ระบุ layer ให้ชัดขึ้น @Repository มีข้อดีพิเศษคือ Spring จะแปลง DB exception เป็น Spring's DataAccessException

**Q: @Transactional ทำงานอย่างไร?**  
A: Spring สร้าง proxy รอบ method นั้น เปิด transaction ก่อนเรียก และ commit เมื่อ method สำเร็จ หรือ rollback เมื่อเกิด RuntimeException (ค่าเริ่มต้น)

**Q: DTO คืออะไร ทำไมต้องใช้?**  
A: Data Transfer Object คือ object ที่ใช้ส่งข้อมูลระหว่าง layer ป้องกัน expose internal entity structure โดยตรง ช่วยกำหนดว่าจะส่ง field ไหนออกไปและ validate input ก่อนถึง service layer
