# REST API Cheat Sheet

## HTTP Methods
| Method | ใช้เมื่อ | Body | Idempotent | Safe |
|--------|---------|------|-----------|------|
| GET | ดึงข้อมูล | ❌ | ✅ | ✅ |
| POST | สร้าง resource ใหม่ | ✅ | ❌ | ❌ |
| PUT | แทนที่ทั้งหมด | ✅ | ✅ | ❌ |
| PATCH | แก้บางส่วน | ✅ | ❌ | ❌ |
| DELETE | ลบ | ❌ | ✅ | ❌ |

## Status Codes
```
2xx Success
200 OK              → GET, PUT, PATCH ทั่วไป
201 Created         → POST สร้างสำเร็จ (+ Location header)
204 No Content      → DELETE, PATCH ที่ไม่ return body

3xx Redirect
301 Moved Permanently
302 Found (temporary redirect)

4xx Client Error
400 Bad Request     → Validation error, malformed
401 Unauthorized    → ยังไม่ login / token หมด
403 Forbidden       → มี token แต่ไม่มีสิทธิ์
404 Not Found       → resource ไม่มี
409 Conflict        → email ซ้ำ, version conflict
422 Unprocessable   → ข้อมูล valid แต่ business logic ผิด
429 Too Many Req    → Rate limit

5xx Server Error
500 Internal Error  → Unexpected exception
503 Unavailable     → Overloaded, maintenance
```

## URL Design
```
Collection:
GET    /api/users          → list
POST   /api/users          → create

Individual:
GET    /api/users/123      → get one
PUT    /api/users/123      → full update
PATCH  /api/users/123      → partial update
DELETE /api/users/123      → delete

Nested:
GET    /api/users/123/orders        → user's orders
POST   /api/users/123/orders        → create order for user

Actions (verbs OK here):
POST   /api/users/123/activate
POST   /api/orders/456/cancel

Versioning:
GET    /api/v1/users
```

## Request / Response Patterns
```json
// POST create — Request
{ "name": "Alice", "email": "alice@example.com" }

// POST create — Response 201
// Header: Location: /api/users/1
{ "id": 1, "name": "Alice", "email": "alice@example.com", "createdAt": "..." }

// Error Response
{
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "errors": [{ "field": "email", "message": "Invalid format" }],
  "timestamp": "...",
  "path": "/api/users"
}
```

## Query Parameters
```
GET /api/products?page=0&size=20        → Pagination
GET /api/products?sort=price,asc        → Sorting
GET /api/products?category=electronics  → Filtering
GET /api/products?q=laptop              → Search
```

## Common Headers
```
Request:
Authorization: Bearer <jwt-token>
Content-Type: application/json
Accept: application/json

Response:
Content-Type: application/json
Location: /api/users/123   (after POST 201)
X-Total-Count: 150         (pagination hint)
```

## Security Basics
```
✅ ใช้ HTTPS เสมอ
✅ Validate input ทุกครั้ง
✅ ไม่ expose stack trace ใน error response
✅ ไม่ใส่ sensitive data ใน URL
✅ ใช้ JWT สำหรับ stateless auth
❌ ไม่ส่ง password ใน response
❌ ไม่ใช้ GET สำหรับ state-changing operations
```
