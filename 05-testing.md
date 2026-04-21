# 05 — Testing

> **เป้าหมาย:** เข้าใจหลักการ testing, เขียน Unit test และ Integration test ด้วย JUnit + Mockito + Spring  
> **หัวข้อ:** Test Pyramid · JUnit 5 · Mockito · Spring Test Slices · TDD

---

## 1. ทำไม Testing สำคัญ?

- **Regression prevention** — มั่นใจว่าการแก้โค้ดไม่ทำลาย feature เดิม
- **Documentation** — test บอกว่า code ควรทำงานอย่างไร
- **Design feedback** — code ที่ test ยาก มักหมายความว่า design ไม่ดี
- **Confidence to refactor** — เปลี่ยน implementation ได้โดยไม่กลัว

---

## 2. Test Pyramid

```
         /\
        /  \      E2E Tests (น้อย, ช้า, แพง)
       /────\
      / Integ \    Integration Tests (ปานกลาง)
     /──────────\
    /  Unit Tests \ (เยอะ, เร็ว, ถูก)
   /──────────────\
```

| ประเภท | ความเร็ว | ค่าใช้จ่าย | จำนวน | ตัวอย่าง |
|--------|---------|----------|-------|---------|
| Unit | เร็วมาก (ms) | ต่ำ | เยอะ (70%) | test single method, mock dependencies |
| Integration | ปานกลาง (s) | ปานกลาง | ปานกลาง (20%) | test DB, Spring context, REST endpoint |
| E2E | ช้า (min) | สูง | น้อย (10%) | test ทั้ง flow จาก UI ถึง DB |

---

## 3. JUnit 5 พื้นฐาน

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    private Calculator calculator;

    @BeforeEach  // ทำก่อนแต่ละ test
    void setUp() {
        calculator = new Calculator();
    }

    @Test
    @DisplayName("Should add two numbers correctly")
    void shouldAddTwoNumbers() {
        // Arrange
        int a = 5, b = 3;

        // Act
        int result = calculator.add(a, b);

        // Assert
        assertEquals(8, result);
    }

    @Test
    void shouldThrowWhenDivideByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }

    @ParameterizedTest
    @ValueSource(ints = {-1, 0, 1, 100})
    void shouldHandleVariousInputs(int input) {
        assertDoesNotThrow(() -> calculator.abs(input));
    }

    @ParameterizedTest
    @CsvSource({
        "1, 2, 3",
        "5, 5, 10",
        "-1, 1, 0"
    })
    void shouldAddCorrectly(int a, int b, int expected) {
        assertEquals(expected, calculator.add(a, b));
    }
}
```

### Assertions ที่ใช้บ่อย

```java
assertEquals(expected, actual);
assertNotEquals(expected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(value);
assertNotNull(value);
assertThrows(ExceptionType.class, executable);
assertAll(                              // group multiple assertions
    () -> assertEquals(1, result.getId()),
    () -> assertEquals("Alice", result.getName())
);
```

---

## 4. Mockito

### Mock vs Stub vs Spy

| | Mock | Stub | Spy |
|--|------|------|-----|
| คืออะไร | Object จำลองทั้งหมด | Object จำลองที่กำหนด return value | Wrap real object + track calls |
| verify calls | ✅ | ❌ | ✅ |
| ใช้เมื่อ | Test behavior (ถูกเรียกไหม) | Test outcome (ได้ค่าอะไร) | ต้องการ real behavior บางส่วน |

```java
import org.mockito.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;  // Mock dependency

    @InjectMocks
    private UserService userService;        // Class ที่ต้องการ test

    @Test
    void shouldReturnUserWhenFound() {
        // Arrange — กำหนด mock behavior
        User mockUser = new User(1L, "Alice", "alice@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));

        // Act
        User result = userService.getUserById(1L);

        // Assert
        assertEquals("Alice", result.getName());
        verify(userRepository, times(1)).findById(1L);  // verify ว่าถูกเรียก
    }

    @Test
    void shouldThrowWhenUserNotFound() {
        // Arrange
        when(userRepository.findById(anyLong())).thenReturn(Optional.empty());

        // Act & Assert
        assertThrows(UserNotFoundException.class,
            () -> userService.getUserById(999L));
    }

    @Test
    void shouldCreateUser() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("Bob", "bob@example.com");
        when(userRepository.findByEmail("bob@example.com")).thenReturn(Optional.empty());
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> {
            User user = invocation.getArgument(0);
            user.setId(2L);
            return user;
        });

        // Act
        User created = userService.createUser(request);

        // Assert
        assertNotNull(created.getId());
        assertEquals("Bob", created.getName());
        verify(userRepository).save(any(User.class));
    }

    @Test
    void shouldThrowWhenEmailAlreadyExists() {
        when(userRepository.findByEmail("alice@example.com"))
            .thenReturn(Optional.of(new User()));

        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser(new CreateUserRequest("Alice", "alice@example.com")));

        verify(userRepository, never()).save(any()); // ไม่ควรถูก save
    }
}
```

### Mockito Matchers ที่ใช้บ่อย

```java
any()                    // ค่าอะไรก็ได้
any(User.class)          // ค่าที่เป็น User type
anyLong()                // long ค่าอะไรก็ได้
anyString()              // String ค่าอะไรก็ได้
eq("specific value")     // ค่า exact
isNull()
isNotNull()
argThat(user -> user.getAge() > 18)  // custom matcher
```

---

## 5. Spring Test Slices

### @SpringBootTest — Full Integration Test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateAndRetrieveUser() {
        // Create
        CreateUserRequest request = new CreateUserRequest("Alice", "alice@test.com");
        ResponseEntity<User> createResponse = restTemplate
            .postForEntity("/api/users", request, User.class);

        assertEquals(201, createResponse.getStatusCodeValue());
        assertNotNull(createResponse.getBody().getId());

        // Retrieve
        Long id = createResponse.getBody().getId();
        ResponseEntity<User> getResponse = restTemplate
            .getForEntity("/api/users/" + id, User.class);

        assertEquals(200, getResponse.getStatusCodeValue());
        assertEquals("Alice", getResponse.getBody().getName());
    }
}
```

### @WebMvcTest — Controller Layer Only

```java
@WebMvcTest(UserController.class)  // Load เฉพาะ web layer
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean  // Mock service (ไม่ load จริง)
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void shouldReturn200WhenUserFound() throws Exception {
        User mockUser = new User(1L, "Alice", "alice@example.com");
        when(userService.getUserById(1L)).thenReturn(mockUser);

        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"))
            .andExpect(jsonPath("$.email").value("alice@example.com"));
    }

    @Test
    void shouldReturn201WhenUserCreated() throws Exception {
        CreateUserRequest request = new CreateUserRequest("Bob", "bob@example.com");
        User created = new User(2L, "Bob", "bob@example.com");
        when(userService.createUser(any())).thenReturn(created);

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(2));
    }

    @Test
    void shouldReturn400WhenEmailInvalid() throws Exception {
        CreateUserRequest invalid = new CreateUserRequest("Bob", "not-an-email");

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalid)))
            .andExpect(status().isBadRequest());
    }

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.getUserById(999L))
            .thenThrow(new UserNotFoundException(999L));

        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.error").value("NOT_FOUND"));
    }
}
```

### @DataJpaTest — Repository Layer Only

```java
@DataJpaTest  // Load เฉพาะ JPA components, ใช้ in-memory H2 DB
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindByEmail() {
        // Arrange
        User user = new User();
        user.setName("Alice");
        user.setEmail("alice@test.com");
        userRepository.save(user);

        // Act
        Optional<User> found = userRepository.findByEmail("alice@test.com");

        // Assert
        assertTrue(found.isPresent());
        assertEquals("Alice", found.get().getName());
    }

    @Test
    void shouldReturnEmptyWhenEmailNotExists() {
        Optional<User> found = userRepository.findByEmail("nobody@test.com");
        assertFalse(found.isPresent());
    }
}
```

---

## 6. TDD (Test-Driven Development)

### Red → Green → Refactor

```
1. RED    — เขียน test ที่ fail ก่อน (feature ยังไม่มี)
2. GREEN  — เขียน code ขั้นต่ำที่ทำให้ test pass
3. REFACTOR — clean up code โดยไม่ทำให้ test fail
```

### ตัวอย่าง TDD Flow

```java
// Step 1: RED — เขียน test ก่อน
@Test
void shouldCalculateOrderTotal() {
    Order order = new Order();
    order.addItem(new OrderItem("Product A", 100.0, 2));
    order.addItem(new OrderItem("Product B", 50.0, 1));

    assertEquals(250.0, order.getTotal()); // Order.getTotal() ยังไม่มี → FAIL
}

// Step 2: GREEN — เขียน code ที่ทำให้ pass
public class Order {
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) {
        items.add(item);
    }

    public double getTotal() {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}

// Step 3: REFACTOR — ปรับปรุง เช่น ใช้ BigDecimal แทน double
```

---

## สรุป Testing Strategy

| Layer | Test Type | Annotation | Mock ไหม? |
|-------|-----------|-----------|---------|
| Service | Unit | `@ExtendWith(MockitoExtension.class)` | ✅ Mock repo |
| Controller | Unit/Integration | `@WebMvcTest` | ✅ Mock service |
| Repository | Integration | `@DataJpaTest` | ❌ Real H2 DB |
| Full flow | E2E integration | `@SpringBootTest` | ❌ Real components |

### คำถามสัมภาษณ์

**Q: Mock กับ Stub ต่างกันอย่างไร?**  
A: Stub กำหนดแค่ return value เพื่อ test outcome, Mock ทำได้ทั้งนั้นและยังสามารถ verify ว่า method ถูกเรียกหรือไม่, กี่ครั้ง, ด้วย argument อะไร

**Q: ทำไมถึงต้องแยก unit test กับ integration test?**  
A: Unit test เร็วและ isolate failure ได้ชัด ถ้า test fail รู้ทันทีว่าปัญหาอยู่ตรงไหน Integration test ช้ากว่าแต่ test ว่า components ทำงานร่วมกันได้จริง ควรรัน unit test บ่อย และ integration test ใน CI/CD pipeline

**Q: @SpringBootTest กับ @WebMvcTest ต่างกันอย่างไร?**  
A: `@SpringBootTest` โหลด application context ทั้งหมด เหมาะ test แบบ end-to-end แต่ช้า `@WebMvcTest` โหลดเฉพาะ web layer (Controller, Filter, ฯลฯ) เร็วกว่ามาก เหมาะ unit test controller
