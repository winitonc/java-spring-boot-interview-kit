# Spring Boot Cheat Sheet

## Annotations Quick Reference

### Stereotype
```java
@SpringBootApplication    // Entry point = @Configuration + @EnableAutoConfiguration + @ComponentScan
@Component                // Generic bean
@Service                  // Business logic layer
@Repository               // Data access layer (+ exception translation)
@RestController           // REST API = @Controller + @ResponseBody
@Controller               // MVC controller (return view name)
```

### Request Mapping
```java
@RequestMapping("/api")       // Base path
@GetMapping("/{id}")
@PostMapping
@PutMapping("/{id}")
@PatchMapping("/{id}")
@DeleteMapping("/{id}")

// Parameters
@PathVariable Long id
@RequestParam(defaultValue="0") int page
@RequestBody @Valid CreateRequest request
@RequestHeader("Authorization") String token
```

### Configuration
```java
@Configuration            // Define beans
@Bean                     // Method returns a bean
@Value("${prop.key:default}") // Inject property
@ConfigurationProperties(prefix = "app")  // Bind group of properties
@EnableCaching            // Enable @Cacheable
@Transactional            // Manage transaction
@Transactional(readOnly = true)
```

### Validation
```java
@NotNull @NotBlank @NotEmpty
@Min(0) @Max(100)
@Size(min=2, max=50)
@Email
@Pattern(regexp = "...")
@Valid  // trigger validation on @RequestBody
```

## Project Structure
```
src/main/java/com/example/
├── Application.java
├── controller/     ← HTTP layer
├── service/        ← Business logic
├── repository/     ← Data access (JpaRepository)
├── model/          ← JPA Entities
├── dto/            ← Request/Response DTOs
└── exception/      ← Custom exceptions + GlobalExceptionHandler
```

## Spring Data JPA
```java
// Repository
interface UserRepo extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThan(int age);
    boolean existsByEmail(String email);
    @Query("SELECT u FROM User u WHERE u.name LIKE %:kw%")
    List<User> search(@Param("kw") String kw);
}

// Entity
@Entity @Table(name = "users")
class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    @Column(nullable = false, unique = true)
    String email;
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    List<Order> orders;
}

// JPA operations
repo.findById(id)        // Optional<T>
repo.findAll()           // List<T>
repo.save(entity)        // insert or update
repo.deleteById(id)
repo.count()
repo.findAll(PageRequest.of(page, size, Sort.by("name")))
```

## Exception Handling
```java
@RestControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(NotFoundException.class)
    ResponseEntity<Error> handle(NotFoundException ex) {
        return ResponseEntity.status(404).body(new Error(ex.getMessage()));
    }
    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<Error> handleValidation(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest().body(...);
    }
}
```

## application.yml Quick Config
```yaml
server:
  port: 8080
  shutdown: graceful

spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DB_USER}
    password: ${DB_PASS}
  jpa:
    hibernate.ddl-auto: validate
    show-sql: false
    open-in-view: false
  redis:
    host: ${REDIS_HOST}

management:
  endpoints.web.exposure.include: health,info
```
