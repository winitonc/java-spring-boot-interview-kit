# 08 — Behavioral Interview

> **เป้าหมาย:** เตรียมตัวสำหรับคำถาม behavioral ด้วยเทคนิค STAR  
> **หัวข้อ:** STAR Method · Common Questions · เคล็ดลับการตอบ

---

## 1. STAR Method

STAR คือ framework สำหรับตอบคำถาม behavioral อย่างมีโครงสร้าง

```
S — Situation   บริบทและสถานการณ์ (ที่ไหน, เมื่อไหร่, ทำอะไร)
T — Task        งานหรือความรับผิดชอบที่ต้องทำ
A — Action      สิ่งที่คุณทำ (เน้น "ฉัน" ไม่ใช่ "เรา")
R — Result      ผลลัพธ์ที่ได้ (เป็นตัวเลขได้ยิ่งดี)
```

### ตัวอย่างการใช้ STAR

**Q: "เล่าประสบการณ์ที่คุณต้องแก้ปัญหายาก"**

```
S: ระหว่าง internship ที่บริษัท X เราพัฒนา feature checkout ใหม่
   และพบว่า performance ช้ามากในช่วง peak hours

T: ฉันรับผิดชอบหา root cause และเสนอแนวทางแก้ไขภายใน 3 วัน

A: ฉัน:
   1. เพิ่ม logging และดู slow query log ใน DB
   2. พบว่า query หนึ่งขาด index ทำให้ full table scan
   3. เพิ่ม index บน orders.user_id + status column
   4. วัดผลก่อน-หลัง และทำ load test เพื่อยืนยัน

R: Response time ลดจาก 3 วินาที เหลือ 200ms
   และ error rate ใน peak hour ลดลง 90%
```

---

## 2. คำถาม Behavioral ที่พบบ่อย + แนวทางตอบ

### กลุ่ม: Learning & Growth

**Q: "ทำไมถึงเลือกสาย Software Engineering?"**  
แนวทาง: บอก motivation จริงๆ เชื่อมกับ interest ใน problem-solving หรือ impact ที่อยากสร้าง

**Q: "เล่า project ที่ภูมิใจที่สุด"**  
แนวทาง: เลือก project ที่เรามีบทบาทชัดเจน บอก challenge + สิ่งที่เรียนรู้ ไม่ต้อง project ใหญ่ขอแค่อธิบายได้ดี

**Q: "สิ่งที่ยากที่สุดที่ต้องเรียนรู้ล่าสุดคืออะไร?"**  
แนวทาง: แสดงว่าเราเรียนรู้จาก challenge ได้ เช่น Spring Security, การออกแบบ database schema ที่ scale ได้

**Q: "เมื่อเจอเทคโนโลยีใหม่ที่ไม่รู้จัก คุณรับมืออย่างไร?"**  
แนวทาง: อธิบาย learning process — อ่าน docs, ทำ small experiment, ถามเพื่อนหรือ Stack Overflow

---

### กลุ่ม: Teamwork & Communication

**Q: "เล่าประสบการณ์ทำงานเป็นทีมที่ดีที่สุด"**  
แนวทาง: เน้น collaboration, communication, trust เล่าว่าแต่ละคนมี role อะไร คุณมีส่วนอย่างไร

**Q: "เคย conflict กับเพื่อนร่วมทีมไหม? แก้อย่างไร?"**  
แนวทาง: อย่าบอกว่าไม่เคย conflict เลย (ไม่น่าเชื่อ) เลือกสถานการณ์ที่แก้ได้ด้วยการสื่อสาร เน้น outcome ที่ดี

```
ตัวอย่าง:
S: ทีมมีความเห็นต่างกันเรื่อง database schema design
T: ต้องตัดสินใจและ move forward ภายใน sprint
A: ฉันเสนอให้นัด technical discussion ที่มี evidence
   เตรียม pros/cons ของแต่ละแนวทาง
   เปิดให้ทุกคน input และตัดสินใจร่วมกัน
R: ได้ข้อสรุปที่ทุกคน agree และ implement สำเร็จ
   เพื่อนร่วมทีมบอกว่า appreciate ที่มีโอกาสแสดงความเห็น
```

**Q: "คุณสื่อสารกับ non-technical stakeholder อย่างไร?"**  
แนวทาง: แสดงว่าสามารถอธิบาย technical concept ด้วยภาษาง่าย เช่น analogy, diagram, demo

---

### กลุ่ม: Problem Solving & Initiative

**Q: "เล่าครั้งที่คุณพบ bug ยากๆ และแก้อย่างไร"**  
แนวทาง: อธิบาย debugging process อย่างละเอียด — reproduce, isolate, root cause, fix, test

**Q: "เคยทำ initiative ที่ไม่ได้รับมอบหมายไหม?"**  
แนวทาง: แสดงความ proactive เช่น พบ code ที่ improve ได้ เพิ่ม test coverage, เขียน documentation ที่ขาด

**Q: "เล่าครั้งที่ deadline ตึงมาก คุณจัดการอย่างไร?"**  
แนวทาง: แสดง prioritization, communication กับ manager เมื่อเห็นว่าเสี่ยง ไม่แค่ทำงานหนักขึ้นโดยไม่บอกใคร

---

### กลุ่ม: Career & Motivation

**Q: "ทำไมถึงสมัครงานที่นี่?"**  
แนวทาง: ค้นข้อมูลบริษัทจริง บอก specific — เช่น product ที่น่าสนใจ, tech stack, culture ที่เหมาะ ไม่ใช่แค่ "เงินดี"

**Q: "5 ปีข้างหน้าอยากเป็นอะไร?"**  
แนวทาง: ไม่ต้องวางแผนชัดเจนมาก แต่แสดงว่ามี direction — เช่น อยากเก่งด้าน backend systems design หรือ lead technical project ได้

**Q: "จุดอ่อนของคุณคืออะไร?"**  
แนวทาง: บอกจุดอ่อนจริง + สิ่งที่กำลังพัฒนา อย่าบอกจุดอ่อนที่เป็น core skill ของงาน

```
ตัวอย่างที่ดี:
"ฉันเคยมีปัญหาเรื่อง time estimation — มักประมาณเวลาน้อยกว่าความเป็นจริง
ตอนนี้ฉันเริ่มแยก task เป็นส่วนย่อยก่อน estimate แต่ละส่วน
และบวก buffer เวลา 20% สำหรับสิ่งที่ไม่คาดคิด"
```

---

## 3. คำถามที่ควรถามกลับ

เตรียมคำถาม 2-3 ข้อ แสดงว่าเราสนใจงานจริง:

```
เกี่ยวกับงาน:
- "ทีมใช้ workflow แบบไหน? Agile/Scrum?"
- "งาน Junior ที่นี่จะได้ mentor หรือมี onboarding plan ไหม?"
- "challenge หลักที่ทีม backend เจออยู่ตอนนี้คืออะไร?"

เกี่ยวกับเทคนิค:
- "Tech stack หลักของทีมเป็นอย่างไรบ้าง?"
- "ทีมทำ code review อย่างไร?"
- "มี automated testing ใน pipeline ไหม?"

เกี่ยวกับ culture:
- "คุณชอบอะไรมากที่สุดในการทำงานที่นี่?"
- "ทีมเรียนรู้ เทคโนโลยีใหม่ๆ อย่างไร?"
```

**อย่าถาม:** เงินเดือน, วันลา, สวัสดิการ (เก็บไว้ถามตอน offer stage)

---

## 4. เคล็ดลับการตอบ

- **เตรียม 5-7 stories** ที่ใช้ตอบคำถาม behavioral ได้หลากหลาย
- **เน้น "ฉัน" ไม่ใช่ "เรา"** — interviewer อยากรู้ว่าคุณทำอะไร
- **ตอบเป็น specific** — ตัวเลข, timeline, เครื่องมือที่ใช้
- **ความยาวพอดี** — ประมาณ 2-3 นาทีต่อคำถาม ไม่สั้นเกิน ไม่ยาวเกิน
- **ถ้าคิดไม่ออก** — บอกได้ว่า "ขอคิดสักครู่" ดีกว่าตอบไปแบบไม่มีตัวอย่างจริง
- **ท้ายตอบ** — สรุป learning หรือ impact เสมอ
