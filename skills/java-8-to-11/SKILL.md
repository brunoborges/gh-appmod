---
name: java-8-to-11
description: Upgrade Java projects from JDK 8 to JDK 11 covering module system, removed Java EE modules (JAXB, JAX-WS, CORBA), var keyword, HTTP Client, collection factory methods
---

# GitHub Copilot Instructions: Java 8 to Java 11 Upgrade Guide

## Project Context
This guide provides comprehensive GitHub Copilot instructions for upgrading Java projects from JDK 8 to JDK 11, covering the module system, removed Java EE modules, new language features, API changes, and migration patterns based on JEPs integrated in Java 9, 10, and 11.

## Critical Migration: Removed Java EE and CORBA Modules

### JEP 320: Remove the Java EE and CORBA Modules (Java 11)
**Migration Pattern**: Add explicit dependencies for removed modules
```java
// These packages were bundled in JDK 8 but REMOVED in JDK 11:
// - java.xml.ws (JAX-WS + SAAJ + Web Services Metadata)
// - java.xml.bind (JAXB)
// - java.activation (JAF)
// - java.xml.ws.annotation (Common Annotations)
// - java.corba (CORBA)
// - java.transaction (JTA)
```

**Maven dependencies to add:**
```xml
<!-- JAXB (XML binding) -->
<dependency>
    <groupId>jakarta.xml.bind</groupId>
    <artifactId>jakarta.xml.bind-api</artifactId>
    <version>2.3.3</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>2.3.9</version>
    <scope>runtime</scope>
</dependency>

<!-- JAX-WS (SOAP web services) -->
<dependency>
    <groupId>jakarta.xml.ws</groupId>
    <artifactId>jakarta.xml.ws-api</artifactId>
    <version>2.3.3</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.ws</groupId>
    <artifactId>jaxws-rt</artifactId>
    <version>2.3.7</version>
    <scope>runtime</scope>
</dependency>

<!-- Java Activation Framework (JAF) -->
<dependency>
    <groupId>jakarta.activation</groupId>
    <artifactId>jakarta.activation-api</artifactId>
    <version>1.2.2</version>
</dependency>

<!-- Common Annotations (@PostConstruct, @PreDestroy, @Resource) -->
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>1.3.5</version>
</dependency>

<!-- JTA (Java Transaction API) -->
<dependency>
    <groupId>jakarta.transaction</groupId>
    <artifactId>jakarta.transaction-api</artifactId>
    <version>1.3.3</version>
</dependency>
```

**Gradle dependencies to add:**
```kotlin
// JAXB
implementation("jakarta.xml.bind:jakarta.xml.bind-api:2.3.3")
runtimeOnly("org.glassfish.jaxb:jaxb-runtime:2.3.9")

// JAX-WS
implementation("jakarta.xml.ws:jakarta.xml.ws-api:2.3.3")
runtimeOnly("com.sun.xml.ws:jaxws-rt:2.3.7")

// Java Activation Framework
implementation("jakarta.activation:jakarta.activation-api:1.2.2")

// Common Annotations
implementation("jakarta.annotation:jakarta.annotation-api:1.3.5")

// JTA
implementation("jakarta.transaction:jakarta.transaction-api:1.3.3")
```

## Critical Migration: Module System (JPMS)

### JEP 261: Module System (Java 9)
**Migration Pattern**: Understand and address module encapsulation
```java
// Internal APIs that are NO LONGER ACCESSIBLE by default in Java 9+:
// - sun.misc.Unsafe → use java.lang.invoke.VarHandle or java.util.concurrent.atomic
// - sun.misc.BASE64Encoder/Decoder → use java.util.Base64
// - sun.reflect.ReflectionFactory → use java.lang.invoke.MethodHandles
// - com.sun.* internal APIs → use supported public APIs

// Old: sun.misc.BASE64Encoder (Java 8)
// sun.misc.BASE64Encoder encoder = new sun.misc.BASE64Encoder();
// String encoded = encoder.encode(data);

// New: java.util.Base64 (Java 8+, required in Java 9+)
String encoded = Base64.getEncoder().encodeToString(data);
byte[] decoded = Base64.getDecoder().decode(encoded);

// Old: sun.misc.Unsafe for atomic operations
// Unsafe unsafe = Unsafe.getUnsafe();

// New: VarHandle (Java 9+)
import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;

public class AtomicCounter {
    private volatile int count;
    private static final VarHandle COUNT_HANDLE;
    
    static {
        try {
            COUNT_HANDLE = MethodHandles.lookup()
                .findVarHandle(AtomicCounter.class, "count", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    
    public int incrementAndGet() {
        return (int) COUNT_HANDLE.getAndAdd(this, 1) + 1;
    }
    
    public boolean compareAndSet(int expected, int newValue) {
        return COUNT_HANDLE.compareAndSet(this, expected, newValue);
    }
}
```

**JVM flags for gradual migration:**
```bash
# Allow access to internal APIs during migration (temporary workaround)
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.lang.reflect=ALL-UNNAMED
--add-opens java.base/java.io=ALL-UNNAMED
--add-exports java.base/sun.nio.ch=ALL-UNNAMED

# Common flags needed for frameworks (Hibernate, Spring, etc.)
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.util=ALL-UNNAMED
--add-opens java.base/java.lang.invoke=ALL-UNNAMED
```

**Optional: Adding a module-info.java**
```java
// You do NOT need module-info.java to run on Java 11.
// Without it, your code runs on the "unnamed module" with full classpath access.
// Only add module-info.java if you want explicit module boundaries.

module com.example.myapp {
    requires java.sql;
    requires java.logging;
    requires java.net.http;   // HTTP Client (Java 11)
    
    exports com.example.myapp.api;
    opens com.example.myapp.model to com.fasterxml.jackson.databind;
}
```

## Language Features and API Changes

### JEP 286: Local-Variable Type Inference (Java 10)
**Migration Pattern**: Use `var` for local variable declarations
```java
// Old: Explicit type declarations
Map<String, List<Employee>> departmentEmployees = new HashMap<>();
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
List<String> filteredNames = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());

// New: Local variable type inference with var (Java 10+)
var departmentEmployees = new HashMap<String, List<Employee>>();
var reader = new BufferedReader(new FileReader("data.txt"));
var filteredNames = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());

// Best practices for var:
// DO use var when the type is obvious from the right-hand side
var count = 0;                          // int
var names = List.of("Alice", "Bob");    // List<String>
var stream = names.stream();            // Stream<String>
var path = Path.of("/tmp/data.txt");    // Path
var connection = DriverManager.getConnection(url); // Connection

// DON'T use var when it reduces readability
// Bad: What type is this?
// var result = service.process(data);
// Good: Explicit type makes intent clear
ProcessingResult result = service.process(data);
```

### JEP 323: Local-Variable Syntax for Lambda Parameters (Java 11)
**Migration Pattern**: Use `var` in lambda expressions for annotations
```java
// Old: No type annotations possible on lambda parameters without explicit types
list.stream()
    .map(s -> s.toUpperCase())
    .collect(Collectors.toList());

// New: var allows annotations on lambda parameters (Java 11+)
list.stream()
    .map((@NotNull var s) -> s.toUpperCase())
    .collect(Collectors.toList());

// Consistent use of var in multi-parameter lambdas
BiFunction<String, String, String> concat = (var a, var b) -> a + b;
```

### JEP 269: Convenience Factory Methods for Collections (Java 9)
**Migration Pattern**: Replace verbose collection initialization
```java
// Old: Verbose collection creation (Java 8)
List<String> list = Collections.unmodifiableList(
    Arrays.asList("a", "b", "c")
);
Set<String> set = Collections.unmodifiableSet(
    new HashSet<>(Arrays.asList("a", "b", "c"))
);
Map<String, Integer> map = Collections.unmodifiableMap(
    new HashMap<String, Integer>() {{
        put("one", 1);
        put("two", 2);
        put("three", 3);
    }}
);

// New: Factory methods (Java 9+) — returns immutable collections
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of(
    "one", 1,
    "two", 2,
    "three", 3
);

// For more than 10 entries, use Map.ofEntries
Map<String, Integer> largeMap = Map.ofEntries(
    Map.entry("one", 1),
    Map.entry("two", 2),
    Map.entry("three", 3)
    // ... more entries
);

// Java 10+: Create unmodifiable copies from existing collections
List<String> copy = List.copyOf(existingList);
Set<String> setCopy = Set.copyOf(existingSet);
Map<String, Integer> mapCopy = Map.copyOf(existingMap);

// Java 10+: Collectors for unmodifiable collections
var unmodifiableList = stream.collect(Collectors.toUnmodifiableList());
var unmodifiableSet = stream.collect(Collectors.toUnmodifiableSet());
var unmodifiableMap = stream.collect(
    Collectors.toUnmodifiableMap(Item::key, Item::value)
);
```

**Important**: `List.of()`, `Set.of()`, and `Map.of()` return immutable collections. They do not allow `null` elements and throw `UnsupportedOperationException` on mutation attempts.

### Stream API Improvements (Java 9)
**Migration Pattern**: Use new stream operations
```java
// takeWhile: Take elements while predicate is true (Java 9)
// Old
List<Integer> result = new ArrayList<>();
for (int n : numbers) {
    if (n >= 10) break;
    result.add(n);
}

// New
List<Integer> result = numbers.stream()
    .takeWhile(n -> n < 10)
    .collect(Collectors.toList());

// dropWhile: Skip elements while predicate is true (Java 9)
List<Integer> afterTen = numbers.stream()
    .dropWhile(n -> n < 10)
    .collect(Collectors.toList());

// ofNullable: Create a stream from a nullable element (Java 9)
// Old
Stream<String> stream = value != null ? Stream.of(value) : Stream.empty();

// New
Stream<String> stream = Stream.ofNullable(value);

// iterate with predicate (Java 9) — replaces traditional for loops
// Old
for (int i = 0; i < 10; i++) { /* ... */ }

// New
IntStream.iterate(0, i -> i < 10, i -> i + 1)
    .forEach(i -> { /* ... */ });
```

### Optional Improvements (Java 9-11)
**Migration Pattern**: Use enhanced Optional methods
```java
// ifPresentOrElse (Java 9)
// Old
if (optional.isPresent()) {
    process(optional.get());
} else {
    handleEmpty();
}

// New
optional.ifPresentOrElse(
    this::process,
    this::handleEmpty
);

// or: Provide alternative Optional (Java 9)
// Old
Optional<String> result = primary.isPresent() ? primary : fallback();

// New
Optional<String> result = primary.or(this::fallback);

// stream: Convert Optional to Stream (Java 9)
// Old
List<String> list = optionals.stream()
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());

// New
List<String> list = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());

// isEmpty (Java 11)
if (optional.isEmpty()) {
    // handle absent value — clearer than !optional.isPresent()
}
```

### JEP 321: HTTP Client (Java 11)
**Migration Pattern**: Replace HttpURLConnection with the new HTTP Client
```java
// Old: HttpURLConnection (verbose and error-prone)
URL url = new URL("https://api.example.com/data");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setRequestProperty("Accept", "application/json");
int status = conn.getResponseCode();
BufferedReader reader = new BufferedReader(
    new InputStreamReader(conn.getInputStream()));
StringBuilder response = new StringBuilder();
String line;
while ((line = reader.readLine()) != null) {
    response.append(line);
}
reader.close();
conn.disconnect();

// New: HTTP Client (Java 11+)
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

HttpClient client = HttpClient.newHttpClient();

// Synchronous GET
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(
    request, HttpResponse.BodyHandlers.ofString());
int statusCode = response.statusCode();
String body = response.body();

// Asynchronous GET
CompletableFuture<HttpResponse<String>> future = client.sendAsync(
    request, HttpResponse.BodyHandlers.ofString());
future.thenApply(HttpResponse::body)
      .thenAccept(System.out::println);

// POST with JSON body
HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(
        "{\"name\": \"Alice\", \"email\": \"alice@example.com\"}"))
    .build();

// Configurable client with timeouts and HTTP/2
HttpClient configuredClient = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)
    .build();
```

### String API Enhancements (Java 11)
**Migration Pattern**: Use new String methods
```java
// isBlank: Check if string is empty or contains only whitespace (Java 11)
// Old
boolean blank = str == null || str.trim().isEmpty();

// New
boolean blank = str.isBlank();  // handles whitespace-only strings

// lines: Split string into stream of lines (Java 11)
// Old
String[] lines = text.split("\\R");

// New
text.lines()
    .filter(line -> !line.isBlank())
    .map(String::strip)
    .forEach(System.out::println);

// strip / stripLeading / stripTrailing: Unicode-aware trim (Java 11)
// Old: trim() only removes ASCII whitespace (<= U+0020)
String trimmed = str.trim();

// New: strip() handles all Unicode whitespace
String stripped = str.strip();
String leftStripped = str.stripLeading();
String rightStripped = str.stripTrailing();

// repeat: Repeat a string N times (Java 11)
// Old
String dashes = new String(new char[20]).replace('\0', '-');
// or
String dashes = String.join("", Collections.nCopies(20, "-"));

// New
String dashes = "-".repeat(20);
String indent = " ".repeat(4);
```

### Files API Enhancements (Java 11)
**Migration Pattern**: Simplified file read/write operations
```java
// Old: Reading entire file to string
String content = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);

// New: readString / writeString (Java 11)
String content = Files.readString(Path.of("data.txt"));

Files.writeString(Path.of("output.txt"), content);
Files.writeString(Path.of("log.txt"), "appended line\n",
    StandardOpenOption.APPEND, StandardOpenOption.CREATE);
```

### JEP 213: Milling Project Coin — Small Language Improvements (Java 9)
**Migration Pattern**: Apply small but useful language enhancements
```java
// Private methods in interfaces (Java 9)
public interface Validator {
    default boolean validateName(String name) {
        return isNonEmpty(name) && name.length() <= 100;
    }
    
    default boolean validateEmail(String email) {
        return isNonEmpty(email) && email.contains("@");
    }
    
    // Private helper method — avoids duplicating logic across defaults
    private boolean isNonEmpty(String value) {
        return value != null && !value.isBlank();
    }
}

// Try-with-resources on effectively final variables (Java 9)
// Old: Must declare new variable in try
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
try (BufferedReader r = reader) {
    return r.readLine();
}

// New: Use effectively final variable directly
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
try (reader) {
    return reader.readLine();
}

// Diamond operator with anonymous inner classes (Java 9)
// Old: Required explicit type
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareToIgnoreCase(b);
    }
};

// New: Diamond operator works with anonymous classes
Comparator<String> comp = new Comparator<>() {
    @Override
    public int compare(String a, String b) {
        return a.compareToIgnoreCase(b);
    }
};

// @SafeVarargs on private methods (Java 9)
// Old: Only allowed on final or static methods
// New: Also allowed on private instance methods
@SafeVarargs
private <T> List<T> asList(T... elements) {
    return List.of(elements);
}
```

### JEP 102: Process API Updates (Java 9)
**Migration Pattern**: Use the enhanced Process API
```java
// Old: Limited process information
Process process = Runtime.getRuntime().exec("mycommand");
int exitCode = process.waitFor();

// New: ProcessHandle with rich info (Java 9+)
ProcessHandle current = ProcessHandle.current();
System.out.println("PID: " + current.pid());
System.out.println("Command: " + current.info().command().orElse("unknown"));
System.out.println("Start time: " + current.info().startInstant().orElse(null));
System.out.println("CPU duration: " + current.info().totalCpuDuration().orElse(null));

// List all processes
ProcessHandle.allProcesses()
    .filter(ProcessHandle::isAlive)
    .forEach(ph -> System.out.printf("PID %d: %s%n",
        ph.pid(), ph.info().command().orElse("unknown")));

// Get notified when a process exits
ProcessHandle.current().onExit()
    .thenAccept(ph -> System.out.println("Process exited: " + ph.pid()));

// Destroy process trees
process.toHandle().descendants()
    .forEach(ProcessHandle::destroy);
```

### JEP 266: Reactive Streams — Flow API (Java 9)
**Migration Pattern**: Use built-in reactive streams interfaces
```java
import java.util.concurrent.Flow;
import java.util.concurrent.SubmissionPublisher;

// The Flow API provides standard interfaces:
// Flow.Publisher, Flow.Subscriber, Flow.Subscription, Flow.Processor

// Simple publisher/subscriber example
SubmissionPublisher<String> publisher = new SubmissionPublisher<>();

publisher.subscribe(new Flow.Subscriber<>() {
    private Flow.Subscription subscription;
    
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }
    
    @Override
    public void onNext(String item) {
        System.out.println("Received: " + item);
        subscription.request(1);
    }
    
    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();
    }
    
    @Override
    public void onComplete() {
        System.out.println("Done");
    }
});

publisher.submit("Hello");
publisher.submit("World");
publisher.close();
```

### JEP 259: Stack-Walking API (Java 9)
**Migration Pattern**: Replace Thread.getStackTrace() with StackWalker
```java
// Old: Captures entire stack trace (expensive)
StackTraceElement[] stack = Thread.currentThread().getStackTrace();
String callerClass = stack[2].getClassName();

// New: StackWalker — lazy and efficient (Java 9+)
StackWalker walker = StackWalker.getInstance(StackWalker.Option.RETAIN_CLASS_REFERENCE);

// Get caller class
Class<?> callerClass = walker.getCallerClass();

// Walk the stack lazily
walker.walk(frames -> frames
    .filter(f -> f.getClassName().startsWith("com.example"))
    .map(StackWalker.StackFrame::getMethodName)
    .collect(Collectors.toList()));
```

## I/O and Networking Improvements

### JEP 238: Multi-Release JAR Files (Java 9)
**Migration Pattern**: Support multiple Java versions in a single JAR
```
META-INF/
  MANIFEST.MF              # Must include: Multi-Release: true
  versions/
    9/
      com/example/Helper.class   # Java 9+ version
    11/
      com/example/Helper.class   # Java 11+ version
com/
  example/
    Helper.class               # Base version (Java 8)
    Main.class
```

```xml
<!-- Maven configuration for multi-release JARs -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <executions>
        <execution>
            <id>compile-java-11</id>
            <phase>compile</phase>
            <goals><goal>compile</goal></goals>
            <configuration>
                <release>11</release>
                <compileSourceRoots>
                    <compileSourceRoot>${project.basedir}/src/main/java11</compileSourceRoot>
                </compileSourceRoots>
                <outputDirectory>${project.build.outputDirectory}/META-INF/versions/11</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## Build System Configuration

### Maven Configuration
```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <maven.compiler.release>11</maven.compiler.release>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <release>11</release>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
        </plugin>
    </plugins>
</build>
```

**If code relies on internal APIs during migration:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <release>11</release>
        <compilerArgs>
            <arg>--add-exports</arg>
            <arg>java.base/sun.nio.ch=ALL-UNNAMED</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

### Gradle Configuration
```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(11)
    }
}

tasks.withType<JavaCompile> {
    options.release.set(11)
}

tasks.withType<Test> {
    useJUnitPlatform()
    // Add if frameworks need internal API access
    jvmArgs(
        "--add-opens", "java.base/java.lang=ALL-UNNAMED",
        "--add-opens", "java.base/java.util=ALL-UNNAMED"
    )
}
```

## JVM and Performance Improvements

### JEP 328: Flight Recorder (Java 11)
**Migration Pattern**: Use JFR for production profiling (was commercial in JDK 8)
```bash
# Start recording with the application
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr -jar myapp.jar

# Continuous recording with disk rotation
java -XX:StartFlightRecording=disk=true,maxsize=500m,maxage=1d -jar myapp.jar
```

```java
// Programmatic JFR recording (Java 9+)
import jdk.jfr.*;

@Label("HTTP Request")
@Category({"Application", "HTTP"})
public class HttpRequestEvent extends Event {
    @Label("URL")
    public String url;
    
    @Label("Status Code")
    public int statusCode;
    
    @Label("Duration (ms)")
    @Timespan(Timespan.MILLISECONDS)
    public long duration;
}

// Usage
HttpRequestEvent event = new HttpRequestEvent();
event.begin();
// perform HTTP request
event.url = "https://api.example.com/data";
event.statusCode = 200;
event.end();
event.commit();
```

### JEP 310: Application Class-Data Sharing (Java 10)
**Migration Pattern**: Improved startup time for applications
```bash
# Create shared archive from application classes
java -Xshare:dump -XX:SharedArchiveFile=app-cds.jsa \
     -XX:SharedClassListFile=classlist.txt -cp myapp.jar

# Run with shared archive
java -Xshare:on -XX:SharedArchiveFile=app-cds.jsa -cp myapp.jar com.example.Main
```

### JEP 307: Parallel Full GC for G1 (Java 10)
**Migration Pattern**: G1 now uses parallel threads for full GC
```bash
# G1 is default since Java 9 — full GC is now parallel by default in Java 10+
# No changes needed, but review GC flags:
-XX:+UseG1GC               # default since Java 9
-XX:ParallelGCThreads=4    # tune for your hardware
```

### JEP 332: Transport Layer Security (TLS) 1.3 (Java 11)
**Migration Pattern**: TLS 1.3 is available by default
```java
// TLS 1.3 is supported automatically. To enforce it:
SSLContext sslContext = SSLContext.getInstance("TLSv1.3");
sslContext.init(null, null, null);

// Or via system property
// -Djdk.tls.client.protocols=TLSv1.3
```

## Deprecations and Removals

### Removed: JavaFX
**Migration Pattern**: Use OpenJFX as a separate dependency
```xml
<!-- Add OpenJFX dependencies -->
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-controls</artifactId>
    <version>11.0.2</version>
</dependency>
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-fxml</artifactId>
    <version>11.0.2</version>
</dependency>
```

### JEP 335: Deprecate the Nashorn JavaScript Engine (Java 11)
**Migration Guidance**: Nashorn is deprecated in 11, removed in 15
```java
// Old: Nashorn (still works in 11, but deprecated)
// ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

// Alternatives:
// 1. GraalVM JavaScript engine
// 2. External Node.js process
// 3. Mozilla Rhino
```

### Removed: Java Web Start and Applets
**Migration Pattern**: Convert to standalone applications or use alternatives
```bash
# Java Web Start (javaws) is removed in Java 11.
# Alternatives:
# - jlink to create self-contained application images
# - jpackage (Java 14+) for native installers
# - Install4j, Launch4j, or other packaging tools
```

### JEP 330: Launch Single-File Source-Code Programs (Java 11)
**Usage**: Run .java files directly without explicit compilation
```bash
# Old: Compile then run
javac HelloWorld.java
java HelloWorld

# New: Run directly (Java 11+)
java HelloWorld.java

# With shebang for scripts (Unix)
#!/usr/bin/java --source 11
public class Script {
    public static void main(String[] args) {
        System.out.println("Hello from Java script!");
    }
}
```

## Version Numbering Change

Starting with Java 9, the version scheme changed:
```bash
# Old (Java 8): 1.8.0_292
java -version
# java version "1.8.0_292"

# New (Java 11): 11.0.x
java -version
# java version "11.0.21"

# In code, update version checks:
// Old
String version = System.getProperty("java.version"); // "1.8.0_292"

// New: Use Runtime.Version (Java 9+)
Runtime.Version version = Runtime.version();
int major = version.feature();    // 11
int minor = version.interim();    // 0
int patch = version.update();     // 21
```

## Testing and Migration Strategy

### Phase 1: Compilation and Dependencies (Weeks 1-2)
1. **Update build system**
   - Set source/target/release to 11
   - Update Maven/Gradle plugins
   - Update CI/CD pipelines

2. **Fix removed module dependencies**
   - Add JAXB, JAX-WS, JTA dependencies
   - Add Common Annotations if using @PostConstruct/@PreDestroy
   - Add JavaFX dependencies if needed

3. **Address internal API usage**
   - Replace sun.misc.BASE64Encoder/Decoder with java.util.Base64
   - Replace sun.misc.Unsafe with VarHandle or supported APIs
   - Add --add-opens/--add-exports flags for remaining issues

### Phase 2: Library and Framework Updates (Weeks 3-4)
1. **Update dependencies**
   - Spring Boot 2.1+ for Java 11 support
   - Hibernate 5.4+ for Java 11 support
   - Jackson, Lombok, and annotation processors
   - Testing frameworks (JUnit 5, Mockito 3+)

2. **Fix reflection-based frameworks**
   - Add --add-opens for frameworks using deep reflection
   - Update ORM configurations if needed

### Phase 3: Adopt New Features (Weeks 5-6)
1. **Language features**
   - Use `var` for local variables where it improves readability
   - Replace collection initialization with factory methods
   - Adopt new Stream and Optional methods

2. **API migration**
   - Replace HttpURLConnection with HTTP Client
   - Use new String methods (isBlank, lines, strip, repeat)
   - Use Files.readString/writeString

### Phase 4: Runtime Optimization (Weeks 7-8)
1. **JVM tuning**
   - Review GC settings (G1 is now default)
   - Enable Flight Recorder for profiling
   - Configure Application CDS for faster startup

2. **Testing and validation**
   - Run full test suite
   - Performance benchmarking
   - Verify production deployment

## Best Practices

1. **Use `var` Judiciously**
   - Use when the type is obvious from context
   - Avoid when it reduces readability
   - Never use for method return types or fields (not allowed)

2. **Prefer Immutable Collections**
   - Use `List.of()`, `Set.of()`, `Map.of()` for constants
   - Use `List.copyOf()` for defensive copies
   - Be aware these do not permit null elements

3. **Replace HttpURLConnection**
   - The new HTTP Client supports HTTP/2, async, and WebSocket
   - Use it for all new HTTP code; migrate existing code gradually

4. **Embrace the New String Methods**
   - `isBlank()` over `trim().isEmpty()`
   - `strip()` over `trim()` (Unicode-aware)
   - `lines()` over `split("\\R")`
   - `repeat()` over manual loops

5. **Use Flight Recorder in Production**
   - JFR is now free and open source (was commercial in JDK 8)
   - Low overhead — safe for production use
   - Create custom events for application-level monitoring

This comprehensive guide enables GitHub Copilot to provide contextually appropriate suggestions when upgrading Java 8 projects to Java 11, focusing on module system compatibility, removed Java EE modules, new language features, and modern API patterns.
