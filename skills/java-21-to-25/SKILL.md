---
name: java-21-to-25
description: Upgrade Java projects from JDK 21 to JDK 25 covering Flexible Constructor Bodies, Stream Gatherers, Class-File API, Scoped Values, Security Manager removal
---

# GitHub Copilot Instructions: Java 21 to Java 25 Upgrade Guide

## Project Context
This guide provides comprehensive GitHub Copilot instructions for upgrading Java projects from JDK 21 to JDK 25, covering new language features, API changes, deprecations, and migration patterns based on 39 JEPs delivered in Java 22, 23, 24, and 25.

## Critical Migration: Security Manager Permanently Disabled

### JEP 486: Permanently Disable the Security Manager (Java 24)
**Migration Pattern**: Remove all Security Manager usage
```java
// These calls now throw UnsupportedOperationException in Java 24+:
// System.setSecurityManager(sm);
// System.getSecurityManager();

// Old: Security Manager checks
SecurityManager sm = System.getSecurityManager();
if (sm != null) {
    sm.checkPermission(new FilePermission("/tmp/data", "read"));
}

// New: Use application-level security, file system permissions,
// or container-based isolation. Remove all SecurityManager references.
// There is no migration path — SecurityManager is gone.
```
- Remove all `.policy` files and `-Djava.security.policy` JVM flags
- Remove `System.setSecurityManager()` and `System.getSecurityManager()` calls
- The `java.security.manager` system property is ignored

## Critical Migration: sun.misc.Unsafe Deprecation and Warnings

### JEP 471: Deprecate Memory-Access Methods in sun.misc.Unsafe (Java 23)
### JEP 498: Warn upon Use of Memory-Access Methods in sun.misc.Unsafe (Java 24)
**Migration Pattern**: Replace Unsafe with VarHandle or Foreign Memory API
```java
// Old: sun.misc.Unsafe for field access (deprecated, warns at runtime in 24+)
Unsafe unsafe = /* obtain Unsafe */;
long offset = unsafe.objectFieldOffset(MyClass.class.getDeclaredField("count"));
unsafe.getInt(obj, offset);
unsafe.putInt(obj, offset, 42);
unsafe.compareAndSwapInt(obj, offset, 0, 1);

// New: VarHandle API (Java 9+) — for on-heap field access
import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;

public class AtomicCounter {
    private volatile int count;
    private static final VarHandle COUNT;

    static {
        try {
            COUNT = MethodHandles.lookup()
                .findVarHandle(AtomicCounter.class, "count", int.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public int get() { return (int) COUNT.get(this); }
    public void set(int val) { COUNT.set(this, val); }
    public boolean cas(int expected, int newVal) {
        return COUNT.compareAndSet(this, expected, newVal);
    }
}

// New: Foreign Memory API (Java 22+) — for off-heap memory access
import java.lang.foreign.*;

// Old: unsafe.allocateMemory / unsafe.getInt / unsafe.freeMemory
// New:
try (Arena arena = Arena.ofConfined()) {
    MemorySegment segment = arena.allocate(ValueLayout.JAVA_INT, 10);
    segment.set(ValueLayout.JAVA_INT, 0, 42);
    int value = segment.get(ValueLayout.JAVA_INT, 0);
}
```

## Critical Migration: JNI Restrictions

### JEP 472: Prepare to Restrict the Use of JNI (Java 24)
**Migration Pattern**: Declare native access and migrate to FFM API
```bash
# JNI usage now produces warnings at runtime in Java 24+.
# Suppress warnings by explicitly granting native access:

# For classpath-based applications
java --enable-native-access=ALL-UNNAMED -jar myapp.jar

# For modular applications
java --enable-native-access=com.example.mymodule -jar myapp.jar
```

```java
// In module-info.java, no special declaration is needed for JNI itself,
// but you must pass --enable-native-access on the command line.

// Preferred: Migrate from JNI to Foreign Function & Memory API (JEP 454)
import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;

// Old: JNI native method
// public native long getProcessId();

// New: FFM API (Java 22+)
public long getProcessId() {
    Linker linker = Linker.nativeLinker();
    SymbolLookup lookup = linker.defaultLookup();
    MethodHandle getpid = linker.downcallHandle(
        lookup.find("getpid").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_LONG)
    );
    try {
        return (long) getpid.invoke();
    } catch (Throwable t) {
        throw new RuntimeException(t);
    }
}
```

## Language Features

### JEP 456: Unnamed Variables & Patterns (Java 22)
**Migration Pattern**: Use `_` for unused variables and patterns
```java
// Old: Unused variable in try-with-resources, catch, or loop
try {
    // ...
} catch (NumberFormatException ex) {  // ex never used
    System.out.println("Invalid number");
}

for (Order order : orders) {  // order never used, just counting
    total++;
}

// New: Unnamed variables (Java 22+)
try {
    // ...
} catch (NumberFormatException _) {
    System.out.println("Invalid number");
}

for (Order _ : orders) {
    total++;
}

// In pattern matching — ignore components you don't need
if (obj instanceof Point(var x, _)) {
    // only care about x coordinate
    System.out.println("x = " + x);
}

switch (shape) {
    case Circle(var radius, _) -> computeCircle(radius);
    case Rectangle(var w, var h) -> computeRect(w, h);
    default -> throw new IllegalArgumentException();
}

// In lambdas
map.forEach((_, value) -> process(value));
```

### JEP 512: Compact Source Files and Instance Main Methods (Java 25)
**Migration Pattern**: Simplified entry points and compact class declarations
```java
// Old: Full class ceremony for simple programs (Java 21)
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// New: Instance main method (Java 25)
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}

// New: Compact source file — implicit class, no class declaration needed
void main() {
    System.out.println("Hello, World!");
}

// Instance main methods can use instance fields and methods
String greeting = "Hello";

void main() {
    println(greeting + ", World!");  // println is available in implicit classes
}

// Priority order for main method selection:
// 1. static void main(String[])     — traditional
// 2. static void main()             — no-args static
// 3. void main(String[])            — instance with args
// 4. void main()                    — instance no-args
```

### JEP 513: Flexible Constructor Bodies (Java 25)
**Migration Pattern**: Statements before super() or this()
```java
// Old: Validation before super() required factory methods or workarounds
public class PositiveInteger extends Number {
    private final int value;

    // Old: Had to validate in a static helper
    public PositiveInteger(int value) {
        super();  // must be first statement in Java 21
        if (value <= 0) throw new IllegalArgumentException("Must be positive");
        this.value = value;
    }
}

// New: Statements allowed before super() (Java 25)
public class PositiveInteger extends Number {
    private final int value;

    public PositiveInteger(int value) {
        // Validation BEFORE calling super
        if (value <= 0) {
            throw new IllegalArgumentException("Must be positive: " + value);
        }
        this.value = value;  // field assignment before super() is allowed
        super();
    }
}

// Useful for preparing arguments for super constructor
public class NamedLogger extends AbstractLogger {
    public NamedLogger(String rawName) {
        // Compute and validate before delegating
        var normalized = rawName.strip().toLowerCase();
        if (normalized.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        super(normalized);  // pass computed value to super
    }
}
```

### JEP 511: Module Import Declarations (Java 25)
**Migration Pattern**: Import all packages exported by a module
```java
// Old: Multiple individual imports
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import java.util.function.Function;

// New: Module import declaration (Java 25)
import module java.base;  // imports all packages exported by java.base

// Brings in java.util.*, java.io.*, java.nio.*, java.time.*, java.lang.*, etc.
// without wildcard-importing each package individually

// Works with any module
import module java.sql;     // imports java.sql.* and javax.sql.*
import module java.net.http; // imports java.net.http.*

// Especially useful for reducing boilerplate in scripts and small programs
import module java.base;

void main() {
    var list = List.of(1, 2, 3);
    var now = Instant.now();
    var path = Path.of("/tmp/test.txt");
}
```

### JEP 454: Foreign Function & Memory API (Java 22)
**Migration Pattern**: Replace JNI with the standard FFM API
```java
import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;

// Call a native C function (e.g., strlen)
public long strlen(String s) {
    Linker linker = Linker.nativeLinker();
    SymbolLookup stdlib = linker.defaultLookup();
    MethodHandle strlenHandle = linker.downcallHandle(
        stdlib.find("strlen").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS)
    );

    try (Arena arena = Arena.ofConfined()) {
        MemorySegment cString = arena.allocateFrom(s);
        return (long) strlenHandle.invoke(cString);
    } catch (Throwable t) {
        throw new RuntimeException(t);
    }
}

// Allocate and use off-heap memory safely
try (Arena arena = Arena.ofConfined()) {
    // Allocate a struct-like segment
    MemorySegment point = arena.allocate(
        MemoryLayout.structLayout(
            ValueLayout.JAVA_INT.withName("x"),
            ValueLayout.JAVA_INT.withName("y")
        )
    );
    point.set(ValueLayout.JAVA_INT, 0, 10);  // x = 10
    point.set(ValueLayout.JAVA_INT, 4, 20);  // y = 20
}
// Memory is automatically freed when arena is closed
```

### JEP 506: Scoped Values (Java 25)
**Migration Pattern**: Replace ThreadLocal with ScopedValue
```java
// Old: ThreadLocal (mutable, easy to leak, problematic with virtual threads)
private static final ThreadLocal<User> CURRENT_USER = new ThreadLocal<>();

public void handleRequest(User user) {
    CURRENT_USER.set(user);
    try {
        processRequest();
    } finally {
        CURRENT_USER.remove();  // must clean up manually
    }
}

public void processRequest() {
    User user = CURRENT_USER.get();
    // ...
}

// New: ScopedValue (Java 25) — immutable, bounded lifetime, virtual-thread-friendly
private static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

public void handleRequest(User user) {
    ScopedValue.runWhere(CURRENT_USER, user, () -> {
        processRequest();  // CURRENT_USER is bound within this scope
    });
    // No cleanup needed — binding ends automatically
}

public void processRequest() {
    User user = CURRENT_USER.get();  // reads the bound value
    // ...
}

// ScopedValue with return value
String result = ScopedValue.callWhere(CURRENT_USER, user, () -> {
    return generateReport();
});
```

### JEP 485: Stream Gatherers (Java 24)
**Migration Pattern**: Custom intermediate stream operations
```java
import java.util.stream.Gatherers;

// Sliding window — groups elements into overlapping windows
List<List<Integer>> windows = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowSliding(3))
    .toList();
// [[1,2,3], [2,3,4], [3,4,5]]

// Fixed-size window (non-overlapping)
List<List<Integer>> chunks = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowFixed(2))
    .toList();
// [[1,2], [3,4], [5]]

// fold — stateful reduction into a stream of accumulated results
Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.fold(() -> 0, Integer::sum))
    .toList();
// [15]

// scan — running accumulation (emits each intermediate result)
Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.scan(() -> 0, Integer::sum))
    .toList();
// [1, 3, 6, 10, 15]

// mapConcurrent — parallel mapping with bounded concurrency
List<String> results = urls.stream()
    .gather(Gatherers.mapConcurrent(10, url -> fetch(url)))
    .toList();

// Custom gatherer — emit elements with distinct consecutive values
Gatherer<String, ?, String> dedup = Gatherer.ofSequential(
    () -> new Object() { String last = null; },
    (state, element, downstream) -> {
        if (!element.equals(state.last)) {
            state.last = element;
            return downstream.push(element);
        }
        return true;
    }
);
Stream.of("a", "a", "b", "b", "b", "c", "a")
    .gather(dedup)
    .toList();
// ["a", "b", "c", "a"]
```

### JEP 484: Class-File API (Java 24)
**Migration Pattern**: Replace ASM with standard Class-File API
```java
import java.lang.classfile.*;
import java.lang.classfile.attribute.*;

// Old: ASM library for bytecode manipulation
// ClassReader reader = new ClassReader(classBytes);
// ClassWriter writer = new ClassWriter(reader, 0);
// ClassVisitor visitor = new MyClassVisitor(writer);
// reader.accept(visitor, 0);
// byte[] result = writer.toByteArray();

// New: Class-File API (Java 24+)
// Parse a class file
ClassModel classModel = ClassFile.of().parse(classBytes);

// Read class information
String className = classModel.thisClass().asInternalName();
for (MethodModel method : classModel.methods()) {
    String name = method.methodName().stringValue();
    String descriptor = method.methodType().stringValue();
}

// Transform a class — add logging to all methods
byte[] transformedBytes = ClassFile.of().transform(classModel,
    ClassTransform.transformingMethods(
        MethodTransform.transformingCode(
            (builder, element) -> {
                if (element instanceof CodeElement ce) {
                    builder.with(ce);
                }
            }
        )
    )
);

// Generate a new class from scratch
byte[] newClass = ClassFile.of().build(
    ClassDesc.of("com.example", "Generated"),
    classBuilder -> classBuilder
        .withFlags(ClassFile.ACC_PUBLIC)
        .withMethod("hello", MethodTypeDesc.of(ClassDesc.ofDescriptor("V")),
            ClassFile.ACC_PUBLIC | ClassFile.ACC_STATIC,
            methodBuilder -> methodBuilder.withCode(codeBuilder -> codeBuilder
                .getstatic(ClassDesc.of("java.lang", "System"), "out",
                    ClassDesc.of("java.io", "PrintStream"))
                .ldc("Hello from generated class!")
                .invokevirtual(ClassDesc.of("java.io", "PrintStream"), "println",
                    MethodTypeDesc.of(ClassDesc.ofDescriptor("V"),
                        ClassDesc.of("java.lang", "String")))
                .return_()
            )
        )
);
```

### JEP 467: Markdown Documentation Comments (Java 23)
**Migration Pattern**: Convert HTML JavaDoc to Markdown
```java
// Old: HTML-based JavaDoc
/**
 * Returns the <b>absolute</b> value of an {@code int} value.
 * <p>
 * If the argument is not negative, the argument is returned.
 * If the argument is negative, the negation of the argument is returned.
 * <ul>
 *   <li>If the argument is {@code Integer.MIN_VALUE}, the result is negative.</li>
 *   <li>If the argument is zero, the result is zero.</li>
 * </ul>
 *
 * @param a the argument whose absolute value is to be determined
 * @return the absolute value of the argument
 * @throws ArithmeticException if the argument is {@code Integer.MIN_VALUE}
 * @see Math#abs(int)
 */

// New: Markdown JavaDoc (Java 23+)
/// Returns the **absolute** value of an `int` value.
///
/// If the argument is not negative, the argument is returned.
/// If the argument is negative, the negation of the argument is returned.
///
/// - If the argument is `Integer.MIN_VALUE`, the result is negative.
/// - If the argument is zero, the result is zero.
///
/// @param a the argument whose absolute value is to be determined
/// @return the absolute value of the argument
/// @throws ArithmeticException if the argument is `Integer.MIN_VALUE`
/// @see Math#abs(int)

// Markdown JavaDoc supports fenced code blocks
/// Example usage:
/// ```java
/// int result = abs(-42);  // returns 42
/// ```

// Links use Markdown syntax with JavaDoc references
/// See [Math#abs(int)] for the standard library version.
/// Uses the algorithm described in [Integer#MIN_VALUE].
```

### JEP 458: Launch Multi-File Source-Code Programs (Java 22)
**Migration Pattern**: Run multi-file programs directly without compilation
```bash
# Java 11 allowed single-file execution: java HelloWorld.java
# Java 22 extends this to multi-file programs

# If Main.java references Helper.java in the same directory:
java Main.java
# The launcher automatically finds and compiles Helper.java

# Works with packages — files must follow package directory structure
java --source 25 com/example/Main.java

# Useful for scripts, prototypes, and educational contexts
```

### JEP 491: Synchronize Virtual Threads without Pinning (Java 24)
**Migration Pattern**: synchronized blocks no longer pin virtual threads
```java
// In Java 21, synchronized blocks pinned virtual threads to platform threads,
// reducing scalability. In Java 24, this is fixed.

// This code now scales properly with virtual threads — no changes needed:
public class SharedResource {
    private final Object lock = new Object();

    public void access() {
        synchronized (lock) {
            // In Java 21: virtual thread was PINNED here (bad for scalability)
            // In Java 24: virtual thread can unmount during blocking (fixed!)
            performBlockingIO();
        }
    }
}

// You no longer need to convert synchronized to ReentrantLock for virtual threads.
// However, ReentrantLock still offers more features (tryLock, fairness, etc.)
```

## I/O and Networking Improvements

### JEP 493: Linking Run-Time Images without JMODs (Java 24)
**Migration Pattern**: Create smaller custom runtime images
```bash
# jlink can now create run-time images without JMOD files,
# reducing JDK installation size by ~25%.

# Standard jlink usage remains the same:
jlink --module-path $JAVA_HOME/jmods \
      --add-modules java.base,java.sql \
      --output custom-runtime

# When JDK is built without JMODs (future distributions may do this):
# jlink uses the run-time image itself as source
jlink --add-modules java.base,java.sql \
      --output custom-runtime
```

## Security Enhancements

### JEP 496: Quantum-Resistant ML-KEM (Java 24)
### JEP 497: Quantum-Resistant ML-DSA (Java 24)
**Migration Pattern**: Use quantum-resistant cryptographic algorithms
```java
// ML-KEM: Module-Lattice-Based Key Encapsulation Mechanism
import javax.crypto.KEM;
import java.security.KeyPairGenerator;

// Generate ML-KEM key pair
KeyPairGenerator kpg = KeyPairGenerator.getInstance("ML-KEM");
kpg.initialize(NamedParameterSpec.ML_KEM_768);
var keyPair = kpg.generateKeyPair();

// Encapsulate (sender side)
KEM kem = KEM.getInstance("ML-KEM");
KEM.Encapsulator enc = kem.newEncapsulator(keyPair.getPublic());
KEM.Encapsulated encapsulated = enc.encapsulate();
byte[] ciphertext = encapsulated.encapsulation();
SecretKey sharedSecret = encapsulated.key();

// Decapsulate (receiver side)
KEM.Decapsulator dec = kem.newDecapsulator(keyPair.getPrivate());
SecretKey receiverSecret = dec.decapsulate(ciphertext);

// ML-DSA: Module-Lattice-Based Digital Signature Algorithm
import java.security.Signature;

KeyPairGenerator sigKpg = KeyPairGenerator.getInstance("ML-DSA");
sigKpg.initialize(NamedParameterSpec.ML_DSA_65);
var sigKeyPair = sigKpg.generateKeyPair();

Signature sig = Signature.getInstance("ML-DSA");
sig.initSign(sigKeyPair.getPrivate());
sig.update(data);
byte[] signature = sig.sign();

// Verify
sig.initVerify(sigKeyPair.getPublic());
sig.update(data);
boolean valid = sig.verify(signature);
```

### JEP 510: Key Derivation Function API (Java 25)
**Migration Pattern**: Standard API for deriving cryptographic keys
```java
import javax.crypto.KDF;

// HKDF (HMAC-based Key Derivation Function)
KDF hkdf = KDF.getInstance("HKDF-SHA256");

// Extract-then-expand
SecretKey derived = hkdf.deriveKey("AES",
    KDF.HKDFParameterSpec.ofExtract()
        .addIKM(inputKeyMaterial)
        .addSalt(salt)
        .thenExpand(info, 32)  // 32 bytes for AES-256
);
```

## JVM and Performance Improvements

### JEP 519: Compact Object Headers (Java 25)
**Migration Pattern**: Reduced object memory overhead (no code changes needed)
```bash
# Compact object headers reduce object header size from 96-128 bits to 64 bits.
# This is now a product feature in Java 25.
# Enabled by default — no flags needed.

# To explicitly disable (not recommended):
# -XX:-UseCompactObjectHeaders
```

### AOT Startup Optimizations

#### JEP 483: Ahead-of-Time Class Loading & Linking (Java 24)
#### JEP 515: Ahead-of-Time Method Profiling (Java 25)
#### JEP 514: Ahead-of-Time Command-Line Ergonomics (Java 25)
**Migration Pattern**: Dramatically improve startup time with AOT caching
```bash
# Java 25 simplifies AOT cache creation with a single flag:

# Step 1: Training run — creates the AOT cache automatically
java -XX:AOTCacheOutput=app.aot -cp myapp.jar com.example.Main

# Step 2: Production run — use the AOT cache for fast startup
java -XX:AOTCache=app.aot -cp myapp.jar com.example.Main

# The AOT cache stores:
# - Pre-loaded and pre-linked classes (JEP 483)
# - Method execution profiles for immediate JIT compilation (JEP 515)
# - Resolved constant pool entries

# For Spring Boot / Quarkus / Micronaut applications, this can cut
# startup time significantly without any code changes.
```

### Garbage Collection Updates

#### JEP 474: ZGC Generational Mode by Default (Java 23)
#### JEP 490: ZGC Remove Non-Generational Mode (Java 24)
```bash
# ZGC is now generational-only. Remove any non-generational flags:

# Old (Java 21): Explicitly choose generational mode
-XX:+UseZGC -XX:+ZGenerational

# New (Java 23+): Generational is default and only mode
-XX:+UseZGC
# -XX:-ZGenerational is no longer accepted in Java 24+

# Non-generational ZGC flags to REMOVE:
# -XX:-ZGenerational        (error in 24+)
# -XX:+ZUncommit            (renamed/removed)
```

#### JEP 423: Region Pinning for G1 (Java 22)
```bash
# G1 GC now supports region pinning for JNI critical sections.
# This eliminates GC pauses caused by JNI GetPrimitiveArrayCritical.
# No configuration needed — automatic improvement.
```

#### JEP 475: Late Barrier Expansion for G1 (Java 24)
```bash
# Internal G1 optimization — reduces C2 JIT compilation overhead.
# No configuration needed — automatic improvement.
```

#### JEP 521: Generational Shenandoah (Java 25)
```bash
# Shenandoah's generational mode is now a product feature.
-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational
```

### JFR Enhancements

#### JEP 518: JFR Cooperative Sampling (Java 25)
#### JEP 520: JFR Method Timing & Tracing (Java 25)
```bash
# JFR is significantly enhanced in Java 25:

# Cooperative sampling — more stable stack sampling at safepoints
# (Enabled by default, no configuration needed)

# Method timing & tracing — measure time spent in specific methods
# Configure via JFR event settings
```

## Build System Configuration

### Maven Configuration
```xml
<properties>
    <maven.compiler.source>25</maven.compiler.source>
    <maven.compiler.target>25</maven.compiler.target>
    <maven.compiler.release>25</maven.compiler.release>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.13.0</version>
            <configuration>
                <release>25</release>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.5.0</version>
            <configuration>
                <!-- If using JNI in tests -->
                <argLine>--enable-native-access=ALL-UNNAMED</argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle Configuration
```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}

tasks.withType<JavaCompile> {
    options.release.set(25)
}

tasks.withType<Test> {
    useJUnitPlatform()
    // If using JNI in tests
    jvmArgs("--enable-native-access=ALL-UNNAMED")
}
```

## Deprecations and Removals

### Removed: 32-bit x86 Ports
- **JEP 479**: Windows 32-bit x86 port removed in Java 24
- **JEP 503**: Linux 32-bit x86 port removed in Java 25
- **Action**: Ensure deployments use 64-bit JDK

### JEP 501: Deprecate 32-bit x86 Port for Removal (Java 24)
- Remaining 32-bit x86 ports deprecated — plan for 64-bit migration

## Testing and Migration Strategy

### Phase 1: Build Compatibility (Weeks 1-2)
1. **Update build system**
   - Set compiler release to 25
   - Update Maven/Gradle plugins
   - Update CI/CD pipelines

2. **Address breaking changes**
   - Remove all SecurityManager usage (JEP 486)
   - Add `--enable-native-access` for JNI usage (JEP 472)
   - Remove `-XX:-ZGenerational` flags (JEP 490)

### Phase 2: Deprecation Warnings (Weeks 3-4)
1. **Fix sun.misc.Unsafe usage** (JEP 471/498)
   - Replace with VarHandle for on-heap access
   - Replace with Foreign Memory API for off-heap access
   - Update third-party libraries

2. **Update JNI code** (JEP 472)
   - Grant native access permissions
   - Evaluate FFM API migration for new code

### Phase 3: Adopt Standard Features (Weeks 5-6)
1. **Language features**
   - Use unnamed variables `_` (JEP 456)
   - Adopt flexible constructor bodies (JEP 513)
   - Use module imports (JEP 511)
   - Adopt compact source files where appropriate (JEP 512)
   - Convert HTML JavaDoc to Markdown (JEP 467)

2. **API features**
   - Use Stream Gatherers (JEP 485)
   - Replace ASM with Class-File API (JEP 484)
   - Adopt Scoped Values (JEP 506)
   - Replace ThreadLocal where appropriate

### Phase 4: Performance Optimization (Weeks 7-8)
1. **Startup optimization**
   - Set up AOT caching (JEP 483/514/515)
   - Benchmark startup improvements

2. **GC tuning**
   - Test generational ZGC (now the only mode)
   - Evaluate generational Shenandoah (JEP 521)
   - Benefit from compact object headers (JEP 519)
   - Leverage virtual thread improvements (JEP 491)

3. **Security updates**
   - Evaluate quantum-resistant algorithms (JEP 496/497)
   - Use KDF API for key derivation (JEP 510)

## Best Practices

1. **Use Unnamed Variables Aggressively**
   - In catch blocks, enhanced for loops, lambdas, and patterns
   - Improves code clarity by signaling intentionally unused values

2. **Adopt Scoped Values over ThreadLocal**
   - Especially important for virtual thread workloads
   - Immutable, bounded lifetime, no leak risk

3. **Embrace Stream Gatherers**
   - Replace complex collector chains with `gather()`
   - Use built-in gatherers: `windowSliding`, `windowFixed`, `fold`, `scan`, `mapConcurrent`

4. **Plan for Post-Quantum Cryptography**
   - Start testing ML-KEM and ML-DSA with non-production workloads
   - Quantum computers that could break RSA/ECC are expected within a decade

5. **Set Up AOT Caching for Production**
   - Single-flag setup in Java 25 makes this easy
   - Significant startup improvements with zero code changes

6. **Migrate Off sun.misc.Unsafe Now**
   - Runtime warnings in Java 24 signal upcoming removal
   - VarHandle and FFM API are the supported replacements

This comprehensive guide enables GitHub Copilot to provide contextually appropriate suggestions when upgrading Java 21 projects to Java 25, focusing on language enhancements, API improvements, security updates, and modern Java development practices.