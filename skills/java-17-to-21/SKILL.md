---
name: java-17-to-21
description: Upgrade Java projects from JDK 17 to JDK 21 covering Virtual Threads, Pattern Matching for switch, Record Patterns, Sequenced Collections
---

# GitHub Copilot Instructions: Java 17 to Java 21 Upgrade Guide

These instructions help GitHub Copilot assist developers in upgrading Java projects from JDK 17 to JDK 21, focusing on new language features, API changes, and best practices.

## Major Language Features in JDK 18-21

### Pattern Matching for switch (JEP 441 - Standard in 21)

**Enhanced switch Expressions and Statements**

When working with switch constructs:
- Suggest converting traditional switch to pattern matching where appropriate
- Use pattern matching for type checking and destructuring
- Example upgrade patterns:
```java
// Old approach (Java 17)
public String processObject(Object obj) {
    if (obj instanceof String) {
        String s = (String) obj;
        return s.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return i.toString();
    }
    return "unknown";
}

// New approach (Java 21)
public String processObject(Object obj) {
    return switch (obj) {
        case String s -> s.toUpperCase();
        case Integer i -> i.toString();
        case null -> "null";
        default -> "unknown";
    };
}
```

- Support guarded patterns:
```java
switch (obj) {
    case String s when s.length() > 10 -> "Long string: " + s;
    case String s -> "Short string: " + s;
    case Integer i when i > 100 -> "Large number: " + i;
    case Integer i -> "Small number: " + i;
    default -> "Other";
}
```

### Record Patterns (JEP 440 - Standard in 21)

**Destructuring Records in Pattern Matching**

When working with records:
- Suggest using record patterns for destructuring
- Combine with switch expressions for powerful data processing
- Example usage:
```java
public record Point(int x, int y) {}
public record ColoredPoint(Point point, Color color) {}

// Destructuring in switch
public String describe(Object obj) {
    return switch (obj) {
        case Point(var x, var y) -> "Point at (" + x + ", " + y + ")";
        case ColoredPoint(Point(var x, var y), var color) -> 
            "Colored point at (" + x + ", " + y + ") in " + color;
        default -> "Unknown shape";
    };
}
```

- Use in complex pattern matching:
```java
// Nested record patterns
switch (shape) {
    case Rectangle(ColoredPoint(Point(var x1, var y1), var c1), 
                   ColoredPoint(Point(var x2, var y2), var c2)) 
        when c1 == c2 -> "Monochrome rectangle";
    case Rectangle r -> "Multi-colored rectangle";
}
```

### Virtual Threads (JEP 444 - Standard in 21)

**Lightweight Concurrency**

When working with concurrency:
- Suggest Virtual Threads for high-throughput, concurrent applications
- Use `Thread.ofVirtual()` for creating virtual threads
- Example migration patterns:
```java
// Old platform thread approach
ExecutorService executor = Executors.newFixedThreadPool(100);
executor.submit(() -> {
    // blocking I/O operation
    httpClient.send(request);
});

// New virtual thread approach
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // blocking I/O operation - now scales to millions
        httpClient.send(request);
    });
}
```

### Sequenced Collections (JEP 431 - Standard in 21)

**Enhanced Collection Interfaces**

When working with collections:
- Use new `SequencedCollection`, `SequencedSet`, `SequencedMap` interfaces
- Access first/last elements uniformly across collection types
- Example usage:
```java
// New methods available on Lists, Deques, LinkedHashSet, etc.
List<String> list = List.of("first", "middle", "last");
String first = list.getFirst();  // "first"
String last = list.getLast();    // "last"
List<String> reversed = list.reversed(); // ["last", "middle", "first"]

// Works with any SequencedCollection
SequencedSet<String> set = new LinkedHashSet<>();
set.addFirst("start");
set.addLast("end");
String firstElement = set.getFirst();
```

## API Enhancements and New Features

### UTF-8 by Default (JEP 400 - Standard in 18)

When working with file I/O:
- UTF-8 is now the default charset on all platforms
- Remove explicit charset specifications where UTF-8 was intended
- Example simplification:
```java
// Old explicit UTF-8 specification
Files.readString(path, StandardCharsets.UTF_8);
Files.writeString(path, content, StandardCharsets.UTF_8);

// New default behavior (Java 18+)
Files.readString(path);  // Uses UTF-8 by default
Files.writeString(path, content);  // Uses UTF-8 by default
```

### Simple Web Server (JEP 408 - Standard in 18)

When needing basic HTTP server:
- Use built-in `jwebserver` command or `com.sun.net.httpserver` enhancements
- Great for testing and development
- Example usage:
```java
// Command line
$ jwebserver -p 8080 -d /path/to/files

// Programmatic usage
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
server.createContext("/", new SimpleFileHandler(Path.of("/tmp")));
server.start();
```

### Internet-Address Resolution SPI (JEP 418 - Standard in 19)

When working with custom DNS resolution:
- Implement `InetAddressResolverProvider` for custom address resolution
- Useful for service discovery and testing scenarios

### Key Encapsulation Mechanism API (JEP 452 - Standard in 21)

When working with post-quantum cryptography:
- Use KEM API for key encapsulation mechanisms
- Example usage:
```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("ML-KEM");
KeyPair kp = kpg.generateKeyPair();

KEM kem = KEM.getInstance("ML-KEM");
KEM.Encapsulator encapsulator = kem.newEncapsulator(kp.getPublic());
KEM.Encapsulated encapsulated = encapsulator.encapsulate();
```

## Deprecations and Warnings

### Finalization Deprecation (JEP 421 - Deprecated in 18)

When encountering `finalize()` methods:
- Remove finalize methods and use alternatives
- Suggest Cleaner API or try-with-resources
- Example migration:
```java
// Deprecated finalize approach
@Override
protected void finalize() throws Throwable {
    cleanup();
}

// Modern approach with Cleaner
private static final Cleaner CLEANER = Cleaner.create();

public MyResource() {
    cleaner.register(this, new CleanupTask(nativeResource));
}

private static class CleanupTask implements Runnable {
    private final long nativeResource;
    
    CleanupTask(long nativeResource) {
        this.nativeResource = nativeResource;
    }
    
    public void run() {
        cleanup(nativeResource);
    }
}
```

### Dynamic Agent Loading (JEP 451 - Warnings in 21)

When working with agents or instrumentation:
- Add `-XX:+EnableDynamicAgentLoading` to suppress warnings if needed
- Consider loading agents at startup instead of dynamically
- Update tooling to use startup agent loading

## Build Configuration Updates

### Maven Configuration
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <release>21</release>
    </configuration>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
</plugin>
```

### Gradle Configuration
```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

tasks.withType<JavaCompile> {
    options.release.set(21)
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### Virtual Threads Configuration

For applications using Virtual Threads:
- No special JVM flags required (standard feature in 21)
- Consider these system properties for debugging:
```bash
-Djdk.virtualThreadScheduler.parallelism=N  # Set carrier thread count
-Djdk.virtualThreadScheduler.maxPoolSize=N  # Set max pool size
```

## Runtime and GC Improvements

### Generational ZGC (JEP 439 - Available in 21)

When configuring garbage collection:
- Try Generational ZGC for better performance
- Enable with: `-XX:+UseZGC -XX:+ZGenerational`
- Monitor allocation patterns and GC behavior

## Migration Strategy

### Step-by-Step Upgrade Process

1. **Update Build Tools**: Ensure Maven/Gradle supports JDK 21
2. **Language Feature Adoption**: 
   - Start with pattern matching for switch (standard)
   - Add record patterns where beneficial
   - Consider Virtual Threads for I/O heavy applications
3. **Testing**: Comprehensive testing especially for concurrency changes
4. **Performance**: Benchmark with new GC options

### Code Review Checklist

When reviewing code for Java 21 upgrade:
- [ ] Convert appropriate instanceof chains to switch expressions
- [ ] Use record patterns for data destructuring
- [ ] Replace ThreadLocal with ScopedValues where appropriate (standard in Java 25)
- [ ] Consider Virtual Threads for high-concurrency scenarios
- [ ] Remove explicit UTF-8 charset specifications
- [ ] Replace finalize() methods with Cleaner or try-with-resources
- [ ] Use SequencedCollection methods for first/last access patterns

### Common Migration Patterns

1. **Switch Enhancement**:
   ```java
   // From instanceof chains to switch expressions
   if (obj instanceof String s) return processString(s);
   else if (obj instanceof Integer i) return processInt(i);
   // becomes:
   return switch (obj) {
       case String s -> processString(s);
       case Integer i -> processInt(i);
       default -> processDefault(obj);
   };
   ```

2. **Virtual Thread Adoption**:
   ```java
   // From platform threads to virtual threads
   Executors.newFixedThreadPool(200)
   // becomes:
   Executors.newVirtualThreadPerTaskExecutor()
   ```

3. **Record Pattern Usage**:
   ```java
   // From manual destructuring to record patterns
   if (point instanceof Point p) {
       int x = p.x();
       int y = p.y();
   }
   // becomes:
   if (point instanceof Point(var x, var y)) {
       // use x and y directly
   }
   ```

## Performance Considerations

- Virtual Threads excel with blocking I/O but may not benefit CPU-intensive tasks
- Generational ZGC can reduce GC overhead for most applications
- Pattern matching in switch is generally more efficient than instanceof chains
- SequencedCollection methods provide O(1) access to first/last elements
- Scoped Values have lower overhead than ThreadLocal for virtual threads (standard in Java 25)

## Testing Recommendations

- Test Virtual Thread applications under high concurrency
- Verify pattern matching covers all expected cases
- Performance test with Generational ZGC vs other collectors
- Validate UTF-8 default behavior across different platforms

Remember to test thoroughly in staging environments before deploying to production.