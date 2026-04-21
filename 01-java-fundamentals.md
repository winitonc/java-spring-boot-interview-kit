# 01 — Java Fundamentals

> **เป้าหมาย:** เข้าใจ Java core concepts ที่มักถูกถามในสัมภาษณ์ระดับ Junior  
> **หัวข้อ:** OOP · Collections · Exception Handling · String · Java 8+ Features · Design Patterns เบื้องต้น

---

## 1. Object-Oriented Programming (OOP)

### 4 หลักการหลัก

| หลักการ | ความหมาย | ตัวอย่าง |
|---------|----------|---------|
| **Encapsulation** | ซ่อน internal state, เข้าถึงผ่าน method | `private` field + getter/setter |
| **Inheritance** | Class ลูกรับ field/method จาก class แม่ | `class Dog extends Animal` |
| **Polymorphism** | Object ตัวเดียวทำงานได้หลายรูปแบบ | Method overriding, overloading |
| **Abstraction** | ซ่อน implementation, เปิดแค่ interface | `abstract class`, `interface` |

### ตัวอย่าง: Polymorphism

```java
abstract class Animal {
    abstract String sound(); // ทุก subclass ต้อง implement
}

class Dog extends Animal {
    @Override
    public String sound() { return "Woof"; }
}

class Cat extends Animal {
    @Override
    public String sound() { return "Meow"; }
}

// ใช้งาน
Animal a = new Dog();
System.out.println(a.sound()); // "Woof" — Runtime polymorphism
```

### Interface vs Abstract Class

| | `interface` | `abstract class` |
|--|------------|-----------------|
| Multiple inheritance | ✅ ได้ | ❌ ไม่ได้ |
| Constructor | ❌ ไม่มี | ✅ มี |
| Fields | `public static final` only | ทุกประเภท |
| Default method | ✅ (Java 8+) | ✅ |
| เมื่อใช้ | กำหนด contract/behavior | มี shared state/logic |

### คำถามสัมภาษณ์

**Q: Interface กับ Abstract Class ต่างกันอย่างไร?**  
A: Interface กำหนด contract ที่ class ต้อง implement ไม่มี state ใช้เมื่อต้องการ multiple inheritance ของ behavior เช่น `Serializable`, `Comparable` ส่วน Abstract class ใช้เมื่อ subclasses มี logic หรือ state ร่วมกัน เช่น `HttpServlet`

**Q: Overriding vs Overloading ต่างกันอย่างไร?**  
A: Overriding เกิดเมื่อ subclass เขียน method ชื่อเดิมทับ (Runtime polymorphism), Overloading คือ method ชื่อเดิมแต่ parameter ต่างกันใน class เดียว (Compile-time)

---

## 2. Collections Framework

### โครงสร้าง Collections

```
Iterable
└── Collection
    ├── List         → ArrayList, LinkedList
    ├── Set          → HashSet, TreeSet, LinkedHashSet
    └── Queue        → LinkedList, PriorityQueue

Map (แยกต่างหาก)
└── HashMap, TreeMap, LinkedHashMap
```

### เมื่อไหร่ใช้อะไร?

| Collection | ใช้เมื่อ | เวลา get/add | Thread-safe? |
|-----------|---------|-------------|-------------|
| `ArrayList` | ต้องการ index access บ่อย | O(1) get, O(1) add amortized | ❌ |
| `LinkedList` | insert/delete ตรงกลางบ่อย | O(n) get, O(1) add first/last | ❌ |
| `HashSet` | เก็บ unique ค่า, ไม่สนลำดับ | O(1) avg | ❌ |
| `TreeSet` | เก็บ unique ค่า + เรียงลำดับ | O(log n) | ❌ |
| `HashMap` | key-value, ไม่สนลำดับ | O(1) avg | ❌ |
| `LinkedHashMap` | key-value + ลำดับ insertion | O(1) avg | ❌ |
| `ConcurrentHashMap` | key-value + multi-thread | O(1) avg | ✅ |

### ตัวอย่าง: Collection operations

```java
// List
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.get(0);          // "Alice"
names.remove("Bob");
Collections.sort(names);

// Map
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.getOrDefault("Bob", 0); // ไม่ throw exception ถ้าไม่มี key
scores.putIfAbsent("Alice", 100); // ไม่เขียนทับถ้ามีอยู่แล้ว
scores.forEach((k, v) -> System.out.println(k + ": " + v));

// Set
Set<String> unique = new HashSet<>(names); // ลบ duplicates
```

### คำถามสัมภาษณ์

**Q: ArrayList กับ LinkedList ต่างกันอย่างไร?**  
A: ArrayList ใช้ array เก็บข้อมูล get(index) เร็ว O(1) แต่ insert/delete ตรงกลางช้า O(n) เพราะต้อง shift elements LinkedList เป็น doubly linked list insert/delete หัว-ท้ายเร็ว O(1) แต่ get(index) ช้า O(n) ในทางปฏิบัติ ArrayList เร็วกว่าในเกือบทุกกรณีเพราะ cache locality ดีกว่า

**Q: HashMap ทำงานอย่างไร?**  
A: HashMap ใช้ hash function แปลง key เป็น index ของ array ภายใน เมื่อ hash ชนกัน (collision) จะใช้ linked list หรือ balanced tree (Java 8+) เก็บในตำแหน่งเดียวกัน key ต้อง implement `hashCode()` และ `equals()` ให้สอดคล้องกัน

**Q: HashMap vs Hashtable vs ConcurrentHashMap?**  
A: `HashMap` ไม่ thread-safe แต่เร็ว, `Hashtable` เก่า + thread-safe แต่ช้า (synchronized ทุก method), `ConcurrentHashMap` thread-safe ทันสมัย แบ่ง lock เป็น segment ทำให้เร็วกว่า Hashtable มาก

---

## 3. Exception Handling

### Checked vs Unchecked

```
Throwable
├── Error          (OutOfMemoryError, StackOverflowError) — ไม่ catch
└── Exception
    ├── Checked    (IOException, SQLException) — ต้อง handle หรือ throws
    └── Unchecked  (RuntimeException และ subclasses)
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IndexOutOfBoundsException
        └── ...
```

### ตัวอย่าง: Exception handling patterns

```java
// Pattern 1: Try-catch-finally
public String readFile(String path) {
    BufferedReader reader = null;
    try {
        reader = new BufferedReader(new FileReader(path));
        return reader.readLine();
    } catch (IOException e) {
        log.error("Failed to read file: {}", path, e);
        throw new RuntimeException("File read failed", e); // wrap เป็น unchecked
    } finally {
        // ทำงานเสมอ ไม่ว่าจะ exception หรือไม่
        if (reader != null) {
            try { reader.close(); } catch (IOException ignored) {}
        }
    }
}

// Pattern 2: Try-with-resources (แนะนำ)
public String readFileModern(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        return reader.readLine();
    } // auto-close เมื่อออกจาก block
}

// Custom Exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}
```

### คำถามสัมภาษณ์

**Q: ควร catch Exception กว้างๆ หรือ specific exception?**  
A: ควร catch เฉพาะ exception ที่รู้ว่าจะเกิดและมีวิธีจัดการ catch Exception กว้างๆ ซ่อน bug ได้ ถ้าจำเป็นต้อง catch กว้าง ควร log หรือ rethrow ไม่ควร swallow exception โดยไม่ทำอะไร

---

## 4. String

### String, StringBuilder, StringBuffer

| | `String` | `StringBuilder` | `StringBuffer` |
|--|---------|----------------|---------------|
| Mutable | ❌ Immutable | ✅ | ✅ |
| Thread-safe | ✅ (immutable) | ❌ | ✅ |
| Performance | ช้า (new object ทุกครั้ง) | เร็ว | ช้ากว่า StringBuilder |

```java
// ❌ สร้าง String object ใหม่ทุกครั้ง
String s = "";
for (int i = 0; i < 1000; i++) {
    s += i; // inefficient!
}

// ✅ ใช้ StringBuilder ใน single thread
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### คำถาม: `==` vs `.equals()`

```java
String a = new String("hello");
String b = new String("hello");

a == b        // false — compare reference (address ใน memory)
a.equals(b)   // true  — compare value

// String pool (literal)
String c = "hello";
String d = "hello";
c == d        // true — same reference จาก string pool
```

---

## 5. Java 8+ Features ที่ต้องรู้

### Lambda Expression

```java
// แบบเดิม
Comparator<String> old = new Comparator<String>() {
    public int compare(String a, String b) { return a.compareTo(b); }
};

// Lambda
Comparator<String> lambda = (a, b) -> a.compareTo(b);

// Method reference
Comparator<String> ref = String::compareTo;
```

### Stream API

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "Anna");

// กรอง + แปลง + รวม
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))    // ["Alice", "Anna"]
    .map(String::toUpperCase)                // ["ALICE", "ANNA"]
    .sorted()                               // ["ALICE", "ANNA"]
    .collect(Collectors.toList());

// นับ
long count = names.stream().filter(n -> n.length() > 4).count();

// หา max/min
Optional<String> longest = names.stream()
    .max(Comparator.comparingInt(String::length));

// groupBy
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
```

### Optional

```java
// ป้องกัน NullPointerException
Optional<User> user = userRepository.findById(id);

// แบบเดิม (อันตราย)
User u = userRepository.findById(id); // อาจ null!
String name = u.getName();            // NullPointerException!

// ✅ ใช้ Optional
user.map(User::getName)
    .orElse("Unknown");

user.ifPresent(u -> System.out.println(u.getName()));

User u = user.orElseThrow(() -> new UserNotFoundException(id));
```

---

## 6. Design Patterns เบื้องต้น

### Singleton

```java
// Thread-safe Singleton (Bill Pugh / Initialization-on-demand)
public class AppConfig {
    private AppConfig() {}

    private static class Holder {
        static final AppConfig INSTANCE = new AppConfig();
    }

    public static AppConfig getInstance() {
        return Holder.INSTANCE;
    }
}
// หมายเหตุ: ใน Spring ใช้ @Component / @Bean แทน Singleton pattern manual
```

### Builder

```java
public class User {
    private final String name;
    private final String email;
    private final int age;

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
    }

    public static class Builder {
        private String name;
        private String email;
        private int age;

        public Builder name(String name) { this.name = name; return this; }
        public Builder email(String email) { this.email = email; return this; }
        public Builder age(int age) { this.age = age; return this; }
        public User build() { return new User(this); }
    }
}

// ใช้งาน
User user = new User.Builder()
    .name("Alice")
    .email("alice@example.com")
    .age(25)
    .build();
// หมายเหตุ: ใน Spring ใช้ Lombok @Builder แทน
```

### คำถามสัมภาษณ์

**Q: ทำไมถึงใช้ Builder pattern?**  
A: เมื่อ object มี parameter มากและบาง parameter เป็น optional Builder ทำให้ code อ่านง่ายและหลีกเลี่ยงปัญหา telescoping constructor ที่มี constructor หลายตัวหรือต้องส่ง null เพื่อ skip parameter บางตัว

---

## สรุป Key Concepts

- OOP 4 หลักการ — Encapsulation, Inheritance, Polymorphism, Abstraction
- Interface vs Abstract class — ใช้ interface เป็น contract, abstract class สำหรับ shared implementation
- Collections — ArrayList ✓ index access, HashMap ✓ key-value lookup, HashSet ✓ uniqueness
- Exception — Checked ต้อง handle, Unchecked คือ programming error
- Java 8 — Lambda, Stream, Optional คือ standard ในโค้ดสมัยใหม่
