# Java Cheat Sheet

## OOP
```java
// Inheritance + Polymorphism
class Animal { String sound() { return "..."; } }
class Dog extends Animal { @Override String sound() { return "Woof"; } }

// Interface
interface Printable { void print(); default void preview() { /* default impl */ } }

// Abstract class
abstract class Shape { abstract double area(); }
```

## Collections Quick Reference

| ต้องการ | ใช้ |
|--------|-----|
| List ที่ access by index | `ArrayList` |
| Queue / Stack | `LinkedList` / `ArrayDeque` |
| Unique elements, unordered | `HashSet` |
| Unique elements, sorted | `TreeSet` |
| Key-value, fast lookup | `HashMap` |
| Key-value, insertion order | `LinkedHashMap` |
| Key-value, sorted by key | `TreeMap` |
| Thread-safe map | `ConcurrentHashMap` |

```java
List<String>  list = new ArrayList<>();
Set<String>   set  = new HashSet<>();
Map<K,V>      map  = new HashMap<>();
Queue<String> q    = new LinkedList<>();

// Common operations
list.add(e); list.get(0); list.remove(e); list.size();
map.put(k,v); map.get(k); map.getOrDefault(k, default); map.containsKey(k);
set.add(e); set.contains(e); set.remove(e);
```

## Streams
```java
list.stream()
    .filter(x -> x > 0)
    .map(String::valueOf)
    .sorted()
    .distinct()
    .limit(10)
    .collect(Collectors.toList());

// Aggregation
long count   = stream.count();
Optional<T> max = stream.max(Comparator.comparing(...));
int sum      = intStream.sum();
Map<K,List<V>> grouped = stream.collect(Collectors.groupingBy(fn));
```

## Optional
```java
Optional<User> opt = repo.findById(id);
opt.isPresent()
opt.get()                          // throw if empty
opt.orElse(defaultValue)
opt.orElseGet(() -> compute())
opt.orElseThrow(() -> new Ex())
opt.map(User::getName).orElse("?")
opt.ifPresent(u -> log(u))
```

## Exception
```java
// Checked → declare/handle
public void read() throws IOException { ... }

// Unchecked → RuntimeException subclass
throw new IllegalArgumentException("msg");
throw new IllegalStateException("msg");

// Custom
class NotFoundException extends RuntimeException {
    NotFoundException(Long id) { super("Not found: " + id); }
}

// Try-with-resources
try (InputStream is = new FileInputStream(file)) { ... }
```

## Java Basics
```java
// String
s.length(); s.charAt(i); s.substring(i,j);
s.toLowerCase(); s.toUpperCase(); s.trim();
s.contains(sub); s.startsWith(p); s.endsWith(s);
s.split(","); s.replace(old, new);
String.format("Hello %s, age %d", name, age);

// equals vs ==
"abc".equals(other)  // ✅ compare value
"abc" == other       // ❌ compare reference

// StringBuilder
new StringBuilder().append(a).append(b).toString();

// Autoboxing
int i = 5; Integer I = i;  // auto boxing
int j = I;                  // auto unboxing
```
