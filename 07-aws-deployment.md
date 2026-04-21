# 07 — AWS & Deployment เบื้องต้น

> **บริบท:** ทีม DevOps/CloudOps ดูแลเรื่อง Deployment บน AWS ให้แล้ว  
> **เป้าหมาย:** รู้แนวคิดว่า service แต่ละตัวคืออะไร ทำอะไรได้ เพื่อสื่อสารกับทีมได้  
> **ไม่ต้องรู้:** Configuration ลึก, Terraform, Kubernetes, CI/CD pipeline setup

---

## 1. AWS Core Services ที่ Backend Developer ควรรู้

### EC2 — Elastic Compute Cloud

**คืออะไร:** Virtual machine (server) บน cloud  
**Backend Developer ควรรู้:**
- App ของเราวิ่งบน EC2 instance
- Instance types: `t3.medium` (2 vCPU, 4 GB RAM) สำหรับ dev, `m5.xlarge` สำหรับ prod
- ทีม DevOps จะ provision และ manage lifecycle ให้

```
EC2 Instance ≈ เครื่อง server เสมือนที่รัน Spring Boot app ของเรา
```

### RDS — Relational Database Service

**คืออะไร:** Managed relational database (PostgreSQL, MySQL, ฯลฯ)  
**Backend Developer ควรรู้:**
- Connection string จะถูก inject ผ่าน environment variable
- RDS handle: backup, patching, multi-AZ failover ให้อัตโนมัติ
- ไม่ต้องจัดการ OS/hardware ของ DB server เอง

```yaml
# application.yml — ใช้ env var ไม่ hardcode credentials
spring:
  datasource:
    url: ${DATABASE_URL}          # เช่น jdbc:postgresql://mydb.rds.amazonaws.com:5432/myapp
    username: ${DATABASE_USER}
    password: ${DATABASE_PASSWORD}
```

### S3 — Simple Storage Service

**คืออะไร:** Object storage สำหรับเก็บไฟล์  
**Backend Developer ควรรู้:**
- ใช้เก็บ: รูปภาพ, PDF, CSV, backup files
- URL format: `https://bucket-name.s3.amazonaws.com/path/to/file`
- ทีม DevOps ตั้ง bucket policy, access control ให้

```java
// ตัวอย่างการ upload file ด้วย AWS SDK (รู้ไว้)
@Service
public class S3Service {
    private final S3Client s3Client;
    private final String bucketName;

    public String uploadFile(String key, byte[] data) {
        s3Client.putObject(PutObjectRequest.builder()
            .bucket(bucketName)
            .key(key)
            .build(), RequestBody.fromBytes(data));
        return "https://" + bucketName + ".s3.amazonaws.com/" + key;
    }
}
```

### ElastiCache — Managed Redis/Memcached

**คืออะไร:** Managed in-memory cache (Redis)  
**Backend Developer ควรรู้:**
- ใช้เป็น cache layer สำหรับ Spring Cache
- Connection จะถูก inject ผ่าน env var เช่นกัน

```yaml
spring:
  redis:
    host: ${REDIS_HOST}
    port: 6379
```

### ELB — Elastic Load Balancer

**คืออะไร:** Load balancer ที่กระจาย traffic ไปยัง EC2 instances  
**Backend Developer ควรรู้:**
- App ของเราต้องเป็น stateless เพื่อ work ข้าม multiple instances
- Health check — ELB จะเรียก `/actuator/health` เพื่อตรวจว่า instance พร้อม
- ต้อง handle `X-Forwarded-For` header ถ้าต้องการ real IP ของ client

### CloudWatch — Monitoring & Logging

**คืออะไร:** Service สำหรับ logs, metrics, alerts  
**Backend Developer ควรรู้:**
- Application logs ของเราจะถูกส่งขึ้น CloudWatch อัตโนมัติ
- ทีม DevOps ตั้ง alert เมื่อ error rate สูงหรือ CPU spike
- ควร log ข้อมูลที่มีประโยชน์ในรูปแบบที่ query ได้ (structured logging)

```java
// ใช้ structured logging ที่ query ใน CloudWatch Logs Insights ได้ง่าย
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class OrderService {
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);

    public Order createOrder(OrderRequest request) {
        log.info("Creating order for user={} items={}", request.getUserId(), request.getItems().size());
        // ...
        log.info("Order created orderId={} userId={}", order.getId(), request.getUserId());
        return order;
    }
}
```

---

## 2. Deployment Flow (ภาพรวม)

```
Developer Push Code
        ↓
  Git Repository (GitHub/GitLab)
        ↓
  CI/CD Pipeline (Jenkins/GitHub Actions) ← ทีม DevOps จัดการ
        ↓
  Build: mvn package → JAR file
        ↓
  Docker Image build + push to ECR
        ↓
  Deploy to EC2 / ECS / EKS
        ↓
  App running behind ELB
        ↓
  CloudWatch collecting logs & metrics
```

**บทบาทของ Backend Developer:**
- เขียน code ที่ environment-agnostic (ใช้ env vars, ไม่ hardcode)
- เขียน health check endpoint (`/actuator/health`)
- Log อย่างมีความหมาย
- Handle graceful shutdown

---

## 3. สิ่งที่ Backend Dev ควรทำให้ Deployment-Ready

### Health Check Endpoint

```java
// Spring Boot Actuator ทำให้อัตโนมัติ
// application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info
  endpoint:
    health:
      show-details: when_authorized
```

### Environment Variables (ไม่ hardcode secrets)

```java
// ❌ อย่าทำ
@Value("secretpassword123") 
private String dbPassword;

// ✅ ถูกต้อง — อ่านจาก env var
@Value("${DATABASE_PASSWORD}")
private String dbPassword;
```

### Graceful Shutdown

```yaml
# application.yml
server:
  shutdown: graceful      # รอ ongoing request ให้เสร็จก่อน shutdown

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

---

## 4. AWS Services Map (สรุป)

| Service | เทียบกับ | ใช้ทำอะไร |
|---------|---------|---------|
| **EC2** | Virtual Machine | รัน application |
| **RDS** | Managed DB Server | PostgreSQL, MySQL |
| **S3** | File System / Google Drive | เก็บไฟล์, static assets |
| **ElastiCache** | Managed Redis | Cache, session |
| **ELB** | Load Balancer | กระจาย traffic |
| **CloudWatch** | Datadog / Grafana | Logs, metrics, alerts |
| **ECR** | Docker Hub (private) | เก็บ Docker images |
| **VPC** | Private Network | Network isolation |
| **IAM** | User/Role Management | Permission management |
| **SQS** | Message Queue | Async task queue |
| **SNS** | Pub/Sub | Notifications, fan-out |

---

## คำถามสัมภาษณ์

**Q: EC2, RDS, S3 คืออะไร อธิบายสั้นๆ?**  
A: EC2 คือ virtual machine สำหรับรัน application, RDS คือ managed database service ที่ AWS ดูแล backup/patching ให้, S3 คือ object storage สำหรับเก็บไฟล์ทุกประเภทได้เกือบไม่จำกัด

**Q: ทำไม App ต้องเป็น Stateless เมื่อ deploy บน AWS?**  
A: เพราะ AWS ELB route request ไปยัง instance ต่างๆ ถ้า app เก็บ state ใน memory instance ใดตัวหนึ่ง request ถัดไปที่ไปถึง instance อื่นจะไม่เห็น state นั้น ต้องเก็บ state ใน external storage เช่น Redis, DB แทน

**Q: ทำไมถึงใช้ environment variable แทนการ hardcode config?**  
A: เพื่อ security (ไม่เก็บ secret ใน code repository), เพื่อ flexibility (ใช้ config ต่างกันใน dev/staging/prod โดยไม่ต้องแก้ code), และ 12-factor app principles
