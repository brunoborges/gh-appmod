# Java 8 to Java 11 Upgrade Checklist

## Pre-Upgrade Assessment

- [ ] Identify usage of internal APIs (sun.misc.*, com.sun.*)
- [ ] Locate JAXB usage (javax.xml.bind)
- [ ] Locate JAX-WS usage (javax.xml.ws)
- [ ] Check for Common Annotations (@PostConstruct, @PreDestroy, @Resource)
- [ ] Check for JTA (javax.transaction) usage
- [ ] Check for CORBA usage
- [ ] Identify JavaFX dependencies
- [ ] Identify Java Web Start (JNLP) usage
- [ ] Locate Nashorn JavaScript engine usage
- [ ] Review sun.misc.BASE64Encoder/Decoder usage
- [ ] Assess sun.misc.Unsafe usage
- [ ] Inventory all third-party library versions for Java 11 compatibility

## Build System Updates

### Maven Projects
- [ ] Update `maven.compiler.source` and `maven.compiler.target` to 11
- [ ] Set `maven.compiler.release` to 11
- [ ] Update Maven Compiler Plugin to 3.8.0+
- [ ] Update Surefire Plugin to 2.22.0+ (3.0.0+ recommended)
- [ ] Update Failsafe Plugin to 2.22.0+
- [ ] Remove `<source>1.8</source>` / `<target>1.8</target>` patterns
- [ ] Add `--add-opens` JVM args to Surefire if frameworks require it

### Gradle Projects
- [ ] Update `java.toolchain.languageVersion` to 11
- [ ] Configure `options.release.set(11)` for JavaCompile tasks
- [ ] Update Gradle version to 5.0+ (6.0+ recommended)
- [ ] Add `--add-opens` JVM args to test tasks if frameworks require it

## Removed Java EE and CORBA Modules (JEP 320)

- [ ] Add `jakarta.xml.bind:jakarta.xml.bind-api` + `org.glassfish.jaxb:jaxb-runtime` for JAXB
- [ ] Add `jakarta.xml.ws:jakarta.xml.ws-api` + `com.sun.xml.ws:jaxws-rt` for JAX-WS
- [ ] Add `jakarta.activation:jakarta.activation-api` for JAF
- [ ] Add `jakarta.annotation:jakarta.annotation-api` for Common Annotations
- [ ] Add `jakarta.transaction:jakarta.transaction-api` for JTA
- [ ] Remove any `--add-modules java.xml.bind` workaround flags (from Java 9/10 migration)
- [ ] Verify all JAXB marshalling/unmarshalling still works
- [ ] Verify all SOAP/JAX-WS services still work

## Internal API Encapsulation (Module System)

- [ ] Replace `sun.misc.BASE64Encoder`/`BASE64Decoder` with `java.util.Base64`
- [ ] Replace `sun.misc.Unsafe` field access with `VarHandle`
- [ ] Replace `sun.misc.Unsafe` memory operations with supported APIs
- [ ] Replace other `sun.*` and `com.sun.*` internal API usage
- [ ] Add `--add-opens` flags for third-party libraries that use reflection on internals
- [ ] Add `--add-exports` flags for libraries accessing non-exported packages
- [ ] Document all `--add-opens`/`--add-exports` flags as technical debt to eliminate

## Dependency Management

- [ ] Update all dependencies to Java 11 compatible versions
- [ ] Update Spring Boot to 2.1+ (for Java 11 support)
- [ ] Update Spring Framework to 5.1+ (for Java 11 support)
- [ ] Update Hibernate to 5.4+
- [ ] Update Jackson to 2.10+
- [ ] Update Lombok to a Java 11 compatible version
- [ ] Update annotation processors (MapStruct, Dagger, etc.)
- [ ] Update byte-code libraries (ASM 7+, ByteBuddy, Javassist)
- [ ] Update testing frameworks (JUnit 5.4+, Mockito 3+, AssertJ)
- [ ] Update build plugins (Checkstyle, SpotBugs, PMD, JaCoCo)
- [ ] Replace libraries using removed APIs (if any depend on JAXB, Nashorn, etc.)
- [ ] Add OpenJFX dependencies if using JavaFX

## Language Feature Adoption

### JEP 286: Local-Variable Type Inference (var)
- [ ] Identify verbose local variable declarations for conversion
- [ ] Use `var` where type is obvious from the right-hand side
- [ ] Avoid `var` where it reduces readability
- [ ] Establish team coding standard for `var` usage

### JEP 269: Collection Factory Methods
- [ ] Replace `Collections.unmodifiableList(Arrays.asList(...))` with `List.of(...)`
- [ ] Replace `Collections.unmodifiableSet(new HashSet<>(...))` with `Set.of(...)`
- [ ] Replace verbose map initialization with `Map.of(...)` or `Map.ofEntries(...)`
- [ ] Use `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()` for defensive copies
- [ ] Use `Collectors.toUnmodifiableList()` etc. where applicable
- [ ] Verify code handles `UnsupportedOperationException` from immutable collections
- [ ] Verify code does not insert `null` into `List.of()` / `Set.of()` / `Map.of()`

### JEP 213: Small Language Improvements
- [ ] Add private methods in interfaces to share default method logic
- [ ] Use effectively-final variables in try-with-resources
- [ ] Use diamond operator with anonymous inner classes

### Stream and Optional Enhancements
- [ ] Use `Stream.takeWhile()` / `dropWhile()` where applicable
- [ ] Use `Stream.ofNullable()` to replace null-check-then-stream patterns
- [ ] Use `Stream.iterate()` with predicate for bounded iterations
- [ ] Use `Optional.ifPresentOrElse()` to replace if/else on Optional
- [ ] Use `Optional.or()` for fallback Optional chains
- [ ] Use `Optional.stream()` with `flatMap` for cleaner collection handling
- [ ] Use `Optional.isEmpty()` instead of `!isPresent()`

## API Migration

### JEP 321: HTTP Client
- [ ] Identify `HttpURLConnection` usage for migration
- [ ] Replace with `java.net.http.HttpClient` for new code
- [ ] Migrate existing HTTP code gradually
- [ ] Use async `sendAsync()` where appropriate

### String Methods (Java 11)
- [ ] Replace `str.trim().isEmpty()` with `str.isBlank()`
- [ ] Replace `str.trim()` with `str.strip()` (Unicode-aware)
- [ ] Use `str.lines()` for line-by-line processing
- [ ] Use `str.repeat(n)` for string repetition

### Files Methods (Java 11)
- [ ] Use `Files.readString()` for simple file reads
- [ ] Use `Files.writeString()` for simple file writes

### Process API (Java 9)
- [ ] Use `ProcessHandle` for process information and management
- [ ] Use `ProcessHandle.current().pid()` instead of JMX workarounds

## JVM Configuration Updates

### Garbage Collection
- [ ] Note that G1 is now the default GC (was Parallel GC in Java 8)
- [ ] Review and update GC flags (remove CMS-specific tuning if switching from CMS)
- [ ] Evaluate GC options (G1 is now default)
- [ ] Consider Epsilon GC for benchmarking scenarios
- [ ] Review G1 tuning — full GC is parallel since Java 10

### Performance Tuning
- [ ] Enable Application CDS for faster startup
- [ ] Enable Flight Recorder for production monitoring (now free)
- [ ] Review string deduplication settings (`-XX:+UseStringDeduplication` with G1)
- [ ] Benefit from Compact Strings (enabled by default since Java 9)

### JVM Flags Review
- [ ] Remove flags that reference removed features
- [ ] Replace `-XX:+UseConcMarkSweepGC` if migrating off CMS
- [ ] Remove `-XX:+CMSClassUnloadingEnabled` and other CMS flags
- [ ] Remove `-XX:MaxPermSize` (PermGen was removed in Java 8, flag errors in 9+)
- [ ] Review `-XX:+UseCompressedOops` (default behavior may differ)
- [ ] Remove any Java 8 commercial feature flags (`-XX:+UnlockCommercialFeatures`)

## Testing Strategy

### Functional Testing
- [ ] Run full test suite with Java 11
- [ ] Test JAXB marshalling/unmarshalling with new dependencies
- [ ] Test JAX-WS services with new dependencies
- [ ] Verify reflection-heavy frameworks (Spring, Hibernate) work correctly
- [ ] Test serialization/deserialization compatibility
- [ ] Test any code using `sun.*` internal APIs

### Performance Testing
- [ ] Benchmark startup time (expect improvement from CDS, Compact Strings)
- [ ] Benchmark throughput with G1 (compare to Parallel GC from Java 8)
- [ ] Measure memory footprint (Compact Strings reduces heap for Latin-1)
- [ ] Test GC pause times

### Compatibility Testing
- [ ] Test on all target deployment platforms
- [ ] Verify Docker base image works (switch to eclipse-temurin:11 or similar)
- [ ] Test native library (JNI) compatibility
- [ ] Verify agent libraries (monitoring, APM) are Java 11 compatible

## Migration Implementation

### Phase 1: Build & Compile (Weeks 1-2)
- [ ] Update build configuration for Java 11
- [ ] Add removed Java EE module dependencies
- [ ] Fix internal API usage (`sun.*`, `com.sun.*`)
- [ ] Add `--add-opens`/`--add-exports` flags where needed
- [ ] Get project compiling on Java 11

### Phase 2: Dependencies & Frameworks (Weeks 3-4)
- [ ] Update all third-party dependencies
- [ ] Fix framework-specific issues (Spring, Hibernate, etc.)
- [ ] Run full test suite and fix failures
- [ ] Address all deprecation warnings

### Phase 3: Feature Adoption (Weeks 5-6)
- [ ] Adopt `var` for local variables
- [ ] Use collection factory methods
- [ ] Adopt new String and Optional methods
- [ ] Replace HttpURLConnection with HTTP Client (new code)

### Phase 4: Optimization & Deployment (Weeks 7-8)
- [ ] Tune GC settings for Java 11
- [ ] Enable Flight Recorder
- [ ] Configure Application CDS
- [ ] Update Docker images and deployment scripts
- [ ] Production deployment with monitoring

## Post-Migration Validation

### Performance Monitoring
- [ ] Monitor startup time improvements
- [ ] Track GC behavior (G1 vs previous collector)
- [ ] Measure memory usage (Compact Strings effect)
- [ ] Monitor throughput and latency

### Operational Readiness
- [ ] Update Docker base images to Java 11
- [ ] Update CI/CD pipelines
- [ ] Update monitoring and alerting for new JVM metrics
- [ ] Document all `--add-opens`/`--add-exports` flags in use

### Code Quality
- [ ] Remove all `--add-opens`/`--add-exports` workarounds over time
- [ ] Eliminate remaining internal API usage
- [ ] Update coding standards for Java 11 idioms
- [ ] Review and update Javadoc

## Rollback Plan

- [ ] Maintain Java 8 branch during transition
- [ ] Keep previous build configurations
- [ ] Document rollback procedures
- [ ] Test rollback before production deployment
- [ ] Maintain feature flags for gradual rollout

## Success Criteria

- [ ] All tests passing on Java 11
- [ ] No performance degradation
- [ ] No runtime warnings for illegal reflective access
- [ ] All removed Java EE APIs replaced with explicit dependencies
- [ ] Team comfortable with new language features
- [ ] Successful production deployment
- [ ] Foundation ready for future upgrade to Java 17
