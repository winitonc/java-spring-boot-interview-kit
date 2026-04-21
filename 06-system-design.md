# 06 — System Design เบื้องต้น

> **เป้าหมาย:** เข้าใจแนวคิด scalability และ system design ในระดับที่ Junior Backend ควรรู้  
> **หัวข้อ:** Scalability · Caching · Load Balancing · Database Scaling · Message Queue · API Design Patterns

---

## 1. Scalability คืออะไร?

ความสามารถของระบบในการรองรับ **load ที่เพิ่มขึ้น** โดยไม่ลดประสิทธิภาพ

### Vertical vs Horizontal Scaling

```
Vertical Scaling (Scale Up)          Horizontal Scaling (Scale Out)
┌─────────────────┐                  ┌──────┐ ┌──────┐ ┌──────┐
│  Big Server     │                  │ App  │ │ App  │ │ App  │
│  CPU: 64 core   │       vs         │  1   │ │  2   │ │  3   │
│  RAM: 512 GB    │                  └──────┘ └──────┘ └──────┘
└─────────────────┘                       ↕ Load Balancer ↕
```

| | Vertical | Horizontal |
|--|---------|-----------|
| วิธี | เพิ่ม CPU/RAM ของ server เดิม | เพิ่ม server ใหม่ |
| ข้อดี | ง่าย, ไม่ต้องเปลี่ยน code | Scale ได้ไม่จำกัด |
| ข้อเสีย | มี limit, downtime ตอน upgrade | ซับซ้อนกว่า (session, state) |
| ใช้กับ | DB server, legacy app | Web servers, stateless services |

---

## 2. Caching

### Cache คืออะไร?

เก็บข้อมูลที่ใช้บ่อยใน storage ที่เร็วกว่า (memory) เพื่อลดเวลาดึงข้อมูลจาก DB

```
Without cache:                    With cache:
Client → App → DB (100ms)         Client → App → Cache (1ms) ✅
                                                 ↓ miss
                                               DB (100ms)
```

### Cache Strategies

```
1. Cache-Aside (Lazy Loading) — แนะนำที่สุด
   Read:  ดู cache ก่อน, ถ้าไม่มี → DB → เก็บ cache → return
   Write: เขียน DB → invalidate/update cache

2. Write-Through
   Write: เขียน cache และ DB พร้อมกัน
   Read:  อ่านจาก cache เสมอ
   ข้อดี: cache always fresh | ข้อเสีย: write ช้าลง

3. Write-Behind (Write-Back)
   Write: เขียน cache ก่อน → async เขียน DB ทีหลัง
   ข้อดี: write เร็วมาก | ข้อเสีย: อาจเสียข้อมูลถ้า crash
```

### Cache ใน Spring Boot (Redis)

```java
// Configuration
@EnableCaching
@SpringBootApplication
public class MyApp { ... }

// Service
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        // เรียกครั้งแรก: query DB, เก็บ cache
        // เรียกครั้งต่อไป: return จาก cache โดยตรง
        return productRepository.findById(id).orElseThrow();
    }

    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
        // ลบ cache เมื่อ delete
    }

    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
        // update cache ทุกครั้ง
    }
}
```

### Cache Considerations

| ปัญหา | คืออะไร | วิธีแก้ |
|------|---------|--------|
| **Cache Miss** | ข้อมูลไม่อยู่ใน cache | ยอมรับได้, cold start |
| **Cache Invalidation** | cache outdated | TTL, explicit evict |
| **Cache Stampede** | หลาย request hit DB พร้อมกันเมื่อ cache expire | Lock หรือ probabilistic refresh |
| **Memory Limit** | cache เต็ม | Eviction policy (LRU, LFU) |

---

## 3. Load Balancing

### Load Balancer คืออะไร?

กระจาย traffic ไปยัง server หลายตัว เพื่อไม่ให้ตัวใดตัวหนึ่ง overload

```
                    ┌──────────────────┐
Clients ──────────→ │  Load Balancer   │
                    └──────────────────┘
                         │    │    │
                    ┌────┘    │    └────┐
                    ↓         ↓         ↓
                 App 1      App 2      App 3
```

### Load Balancing Algorithms

| Algorithm | วิธีทำงาน | ใช้เมื่อ |
|-----------|---------|--------|
| **Round Robin** | เวียนไปทีละตัว | Server มี spec เท่ากัน |
| **Least Connections** | ส่งไปตัวที่ connection น้อยสุด | Request ใช้เวลาต่างกัน |
| **IP Hash** | hash IP → server เดิมเสมอ | ต้องการ session affinity |
| **Weighted** | server แรงกว่า รับมากกว่า | Server ต่าง spec กัน |

### Stateless Application (สำคัญมาก!)

```
❌ Stateful — ทำงานกับ multiple instances ไม่ได้
Request 1 → App 1 (เก็บ session ใน memory)
Request 2 → App 2 (ไม่เห็น session!) → Error

✅ Stateless — ทำงานกับ multiple instances ได้
Session เก็บใน: JWT token / Redis / DB
Request 1 → App 1 (read session from Redis)
Request 2 → App 2 (read session from Redis) ✅
```

---

## 4. Database Scaling

### Read Replicas

```
                         ┌─── Read Replica 1 (async sync)
Primary DB ──────────────┤─── Read Replica 2
(read + write)           └─── Read Replica 3

→ Write → Primary
→ Read  → Any Replica (ลด load บน Primary)
```

### Database Connection Pool

```java
// application.yml — ปรับ HikariCP (default connection pool ของ Spring Boot)
spring:
  datasource:
    hikari:
      maximum-pool-size: 20      # max connections
      minimum-idle: 5
      connection-timeout: 30000  # ms
      idle-timeout: 600000
```

**Q: Connection Pool คืออะไร?**  
A: Pool ของ DB connection ที่เปิดไว้รอ เพื่อไม่ต้องสร้าง connection ใหม่ทุกครั้ง (การ connect ใช้เวลานาน) Application ยืม connection จาก pool ใช้แล้วคืน

---

## 5. Message Queue (แนวคิด)

### ทำไมต้องใช้ Message Queue?

```
❌ Synchronous (tight coupling)
Order Service → Payment Service → Inventory Service
(ถ้า Inventory ล่ม → Order ก็ล้มเหลว)

✅ Asynchronous via Message Queue
Order Service → [Queue] → Payment Service
                        → Inventory Service
                        → Notification Service
(แต่ละ service อิสระต่อกัน)
```

### Use Cases

- **Order processing** — สร้าง order เสร็จแล้วส่ง event ให้ payment, inventory, email notification
- **Email/SMS sending** — ไม่ block user request รอส่ง email
- **Log aggregation** — รวม logs จากหลาย service
- **Retry mechanism** — ถ้า consumer fail, message ยังอยู่ใน queue

### เครื่องมือที่นิยม

| เครื่องมือ | เหมาะกับ |
|---------|---------|
| **RabbitMQ** | Task queues, reliable delivery |
| **Apache Kafka** | High-throughput event streaming, log |
| **AWS SQS** | Managed queue, ง่าย, integrate กับ AWS |

```java
// Spring กับ RabbitMQ — producer
@Service
public class OrderService {
    private final RabbitTemplate rabbitTemplate;

    public Order createOrder(OrderRequest request) {
        Order order = orderRepository.save(buildOrder(request));
        // ส่ง event หลัง save สำเร็จ
        rabbitTemplate.convertAndSend("order.exchange", "order.created", order);
        return order;
    }
}

// Consumer
@Component
public class InventoryConsumer {
    @RabbitListener(queues = "inventory.queue")
    public void handleOrderCreated(Order order) {
        inventoryService.deductStock(order.getItems());
    }
}
```

---

## 6. ออกแบบ System เบื้องต้น — วิธีคิด

### Framework สำหรับตอบ System Design Questions

```
1. Clarify requirements (ถามก่อนเสมอ)
   - DAU / QPS เท่าไหร่?
   - Read heavy หรือ Write heavy?
   - Consistency vs Availability สำคัญกว่า?

2. Back-of-envelope estimation
   - 100K DAU × 10 req/day = 1M req/day ≈ ~12 req/s

3. High-level design
   - Client → Load Balancer → App Servers → DB

4. Deep dive
   - ส่วนที่ขอ: caching strategy, database choice, ฯลฯ

5. Trade-offs
   - ทุก decision มีข้อดีข้อเสีย พูดถึงเสมอ
```

### ตัวอย่าง: ออกแบบ URL Shortener

```
Requirements:
- Shorten URL: POST /shorten → return short code
- Redirect: GET /:code → redirect to original URL
- Read heavy (redirect >> create)

Design:
Client → [Load Balancer]
              ↓
         App Servers (stateless)
              ↓              ↓
          [Cache]          [DB]
        (Redis, TTL 1h)  (code → url mapping)

Key decisions:
- Code generation: base62 encode (random 6 char)
- Cache: popular URLs cached in Redis
- DB: สามารถใช้ PostgreSQL หรือ NoSQL (Cassandra) ถ้า scale มาก
```

---

## คำถามสัมภาษณ์

**Q: Microservices vs Monolith ต่างกันอย่างไร?**  
A: Monolith เป็น single deployable unit ง่ายกว่าในการ develop ตอนต้น แต่ scale ยากเมื่อโตขึ้น Microservices แยกเป็น independent services แต่ละตัว deploy แยก scale แยกได้ แต่ซับซ้อนกว่ามากด้าน ops, network latency, distributed transactions สำหรับ Junior แนะนำให้เข้าใจ concept แต่ไม่จำเป็นต้องลงรายละเอียดลึก

**Q: Caching กับ Database ต่างกันอย่างไร?**  
A: Cache คือ fast temporary storage (มักเป็น in-memory) สำหรับข้อมูลที่ read บ่อย ข้อมูลอาจหายเมื่อ restart DB คือ persistent storage ข้อมูลถาวร Cache เป็น optimization layer ช่วยลด DB load

**Q: ทำไม Stateless API ถึงสำคัญสำหรับ scalability?**  
A: Stateless API ทำให้สามารถรัน multiple instances ได้โดยไม่ต้องกังวลว่า request จาก user คนเดียวกันต้องไปถึง instance เดิม ทำให้ horizontal scaling ง่าย
