# 04 — REST API Design

> **เป้าหมาย:** เข้าใจหลักการออกแบบ REST API ที่ดี, HTTP, และ Error handling  
> **หัวข้อ:** HTTP Methods · Status Codes · REST Principles · URL Design · Request/Response · Pagination

---

## 1. HTTP Methods

| Method | ใช้เพื่อ | Idempotent? | Safe? | Body? |
|--------|---------|------------|-------|-------|
| `GET` | ดึงข้อมูล | ✅ | ✅ | ❌ |
| `POST` | สร้างข้อมูล | ❌ | ❌ | ✅ |
| `PUT` | แทนที่ทั้งหมด | ✅ | ❌ | ✅ |
| `PATCH` | แก้บางส่วน | ❌* | ❌ | ✅ |
| `DELETE` | ลบ | ✅ | ❌ | ❌ |

- **Idempotent** = เรียกหลายครั้ง ผลเหมือนกับเรียกครั้งเดียว
- **Safe** = ไม่เปลี่ยนแปลง state บน server

### ตัวอย่าง PUT vs PATCH

```json
// PUT /api/users/1 — ต้องส่งทุก field
{ "name": "Alice", "email": "alice@new.com", "age": 25 }

// PATCH /api/users/1 — ส่งเฉพาะ field ที่จะเปลี่ยน
{ "email": "alice@new.com" }
```

---

## 2. HTTP Status Codes ที่ต้องรู้

### 2xx — Success

| Code | ความหมาย | ใช้เมื่อ |
|------|---------|---------|
| `200 OK` | สำเร็จ | GET, PUT, PATCH ทั่วไป |
| `201 Created` | สร้างสำเร็จ | POST สร้าง resource ใหม่ |
| `204 No Content` | สำเร็จ ไม่มี body | DELETE, หรือ PATCH ที่ไม่ return |

### 4xx — Client Error

| Code | ความหมาย | ใช้เมื่อ |
|------|---------|---------|
| `400 Bad Request` | ข้อมูลที่ส่งมาผิด | Validation error, malformed JSON |
| `401 Unauthorized` | ยังไม่ได้ authenticate | ไม่มี/token หมดอายุ |
| `403 Forbidden` | ไม่มีสิทธิ์ | มี token แต่ไม่มี permission |
| `404 Not Found` | ไม่พบ resource | User/Product ไม่มีในระบบ |
| `409 Conflict` | ข้อมูล conflict | Email ซ้ำ, version conflict |
| `422 Unprocessable Entity` | ข้อมูล valid แต่ logic ผิด | สั่งซื้อเกิน stock |

### 5xx — Server Error

| Code | ความหมาย | ใช้เมื่อ |
|------|---------|---------|
| `500 Internal Server Error` | Error ที่ server | Unexpected exception |
| `503 Service Unavailable` | Service ไม่พร้อม | Overloaded, maintenance |

---

## 3. REST Principles

### 6 Constraints ของ REST

1. **Client-Server** — แยก UI จาก data storage
2. **Stateless** — แต่ละ request ต้องมีข้อมูลครบ server ไม่เก็บ session
3. **Cacheable** — response บอกได้ว่า cache ได้ไหม
4. **Uniform Interface** — ใช้ resource-based URL + HTTP methods
5. **Layered System** — client ไม่รู้ว่ากำลังคุยกับ server จริงหรือ load balancer
6. **Code on Demand** (optional) — server ส่ง executable code ได้

### Stateless คืออะไรในทางปฏิบัติ?

```
❌ Stateful (ไม่ดี):
Client → POST /login        → Server เก็บ session
Client → GET /profile       → Server ค้นหา session

✅ Stateless (REST):
Client → POST /login        → Server ส่ง JWT token
Client → GET /profile       → ส่ง Authorization: Bearer <token> ใน header
                              Server verify token ใน request เดียว
```

---

## 4. URL Design (Resource Naming)

### หลักการ

```
✅ Nouns (resource names)     ❌ Verbs (actions)
/api/users                    /api/getUsers
/api/users/123                /api/getUserById?id=123
/api/users/123/orders         /api/getUserOrders?userId=123
/api/orders/456/items         /api/getOrderItems?orderId=456
```

### Conventions

```
# Collection resource — ใช้ plural
GET    /api/users           → list all users
POST   /api/users           → create user

# Individual resource
GET    /api/users/123       → get user 123
PUT    /api/users/123       → update user 123 (full replace)
PATCH  /api/users/123       → partial update user 123
DELETE /api/users/123       → delete user 123

# Nested resources (sub-resources)
GET    /api/users/123/orders          → orders ของ user 123
POST   /api/users/123/orders          → สร้าง order ให้ user 123
GET    /api/users/123/orders/456      → order 456 ของ user 123

# Actions ที่เป็น verb จริงๆ (ยอมรับได้)
POST   /api/users/123/activate
POST   /api/orders/456/cancel
POST   /api/payments/456/refund
```

### Version

```
/api/v1/users     ← ใน URL path (ชัดเจน แนะนำ)
/api/users        ← Header: Accept: application/vnd.myapp.v1+json
```

---

## 5. Request & Response Design

### Request (Input)

```java
// DTO + Validation
public class CreateUserRequest {
    @NotBlank(message = "Name is required")
    @Size(max = 100)
    private String name;

    @NotBlank
    @Email(message = "Invalid email format")
    private String email;

    @Min(value = 0)
    @Max(value = 150)
    private Integer age;
}

// Controller
@PostMapping
public ResponseEntity<UserResponse> createUser(
    @Valid @RequestBody CreateUserRequest request  // @Valid trigger validation
) {
    ...
}
```

### Response (Output)

```json
// ✅ GET /api/users/123 — 200 OK
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "createdAt": "2026-04-21T10:30:00Z"
}

// ✅ POST /api/users — 201 Created
// Header: Location: /api/users/124
{
  "id": 124,
  "name": "Bob",
  "email": "bob@example.com",
  "createdAt": "2026-04-21T10:31:00Z"
}
```

### Error Response Format

```json
// 400 Bad Request — Validation error
{
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "Invalid email format" },
    { "field": "name",  "message": "Name is required" }
  ],
  "timestamp": "2026-04-21T10:30:00Z",
  "path": "/api/users"
}

// 404 Not Found
{
  "status": 404,
  "error": "NOT_FOUND",
  "message": "User not found with id: 999",
  "timestamp": "2026-04-21T10:30:00Z"
}
```

```java
// Spring implementation
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException ex, HttpServletRequest request
    ) {
        List<FieldError> fieldErrors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .toList();

        ErrorResponse error = ErrorResponse.builder()
            .status(400)
            .error("BAD_REQUEST")
            .message("Validation failed")
            .errors(fieldErrors)
            .path(request.getRequestURI())
            .build();

        return ResponseEntity.badRequest().body(error);
    }
}
```

---

## 6. Pagination

```
GET /api/products?page=0&size=20&sort=price,asc
```

```java
// Controller
@GetMapping
public ResponseEntity<Page<ProductResponse>> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id") String sortBy
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return ResponseEntity.ok(productService.getProducts(pageable));
}
```

```json
// Response ที่มี pagination metadata
{
  "content": [...],
  "page": {
    "number": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8
  }
}
```

---

## 7. Security Basics (รู้ไว้)

```
// ทุก protected endpoint ต้องมี Authorization header
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

// Spring Security filter chain validate token ก่อนถึง Controller
Request → JwtAuthFilter → SecurityContext → Controller
```

**สิ่งที่ไม่ควรทำ:**
- ไม่ expose stack trace ใน error response (ข้อมูล internal)
- ไม่ใส่ sensitive data ใน URL (password, token ใน query param)
- ตรวจสอบ input ทุกครั้งก่อนประมวลผล

---

## คำถามสัมภาษณ์

**Q: REST กับ SOAP ต่างกันอย่างไร?**  
A: REST เป็น architectural style ใช้ HTTP methods + JSON/XML เบากว่า flexible กว่า SOAP เป็น protocol ที่เข้มงวดกว่าใช้ XML เท่านั้น มี WS-* standards REST เป็น standard ใหม่กว่าและใช้กันทั่วไปใน Web API

**Q: 401 กับ 403 ต่างกันอย่างไร?**  
A: 401 Unauthorized = ไม่รู้ว่าคุณคือใคร (ยังไม่ authenticate เช่น ไม่มี token), 403 Forbidden = รู้แล้วว่าคุณคือใครแต่ไม่มีสิทธิ์ทำสิ่งนี้

**Q: Idempotent หมายความว่าอะไร?**  
A: เรียก operation เดิมหลายครั้ง ผลลัพธ์เหมือนกับเรียกครั้งเดียว เช่น DELETE /users/123 เรียกครั้งแรก: ลบ user เรียกครั้งที่ 2: user ไม่มีแล้ว (อาจ 404) ผลสุดท้ายคือ user ถูกลบเหมือนกัน

**Q: ควรออกแบบ API อย่างไรสำหรับการ search ที่ซับซ้อน?**  
A: ใช้ query parameters สำหรับ simple filter (`?status=ACTIVE&minAge=18`) สำหรับ complex search อาจใช้ POST ที่มี body เป็น search criteria (เช่น Elasticsearch-style) หรือใช้ GraphQL
