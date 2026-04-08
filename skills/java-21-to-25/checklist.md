# Java 21 to Java 25 Upgrade Checklist

## Pre-Upgrade Assessment

- [ ] Identify all `System.getSecurityManager()` / `System.setSecurityManager()` usage
- [ ] Inventory `sun.misc.Unsafe` usage in codebase and dependencies
- [ ] Identify JNI usage (direct or through libraries like JNA, JNR)
- [ ] Check for ASM library usage for bytecode manipulation
- [ ] Check for non-generational ZGC flags (`-XX:-ZGenerational`)
- [ ] Review HTML-heavy JavaDoc comments for Markdown conversion
- [ ] Assess `ThreadLocal` usage for ScopedValue migration
- [ ] Identify `synchronized` blocks used with virtual threads
- [ ] Review switch statements for unnamed variable / pattern opportunities
- [ ] Check for 32-bit x86 deployment targets

## Build System Updates

### Maven Projects
- [ ] Update `maven.compiler.release` to 25
- [ ] Update Maven Compiler Plugin to 3.13.0+
- [ ] Update Surefire Plugin to 3.5.0+
- [ ] Add `--enable-native-access=ALL-UNNAMED` to Surefire if using JNI

### Gradle Projects
- [ ] Update `java.toolchain.languageVersion` to 25
- [ ] Update Gradle wrapper to compatible version
- [ ] Configure `options.release.set(25)` for JavaCompile
- [ ] Add `--enable-native-access=ALL-UNNAMED` to test JVM args if using JNI

## Dependency Management

- [ ] Update all dependencies to Java 25 compatible versions
- [ ] Update Spring Boot to 3.4+ / Spring Framework to 6.2+
- [ ] Update Hibernate / JPA provider
- [ ] Update Jackson, Lombok, and annotation processors
- [ ] Update bytecode libraries (ASM → Class-File API, ByteBuddy, Javassist)
- [ ] Update testing frameworks (JUnit 5.11+, Mockito 5+)
- [ ] Update static analysis tools (SpotBugs, PMD, Checkstyle)
- [ ] Verify APM/monitoring agent compatibility

## Breaking Changes

### JEP 486: Security Manager Permanently Disabled
- [ ] Remove all `System.getSecurityManager()` calls
- [ ] Remove all `System.setSecurityManager()` calls
- [ ] Remove `.policy` files and `-Djava.security.policy` JVM flags
- [ ] Remove `-Djava.security.manager` JVM flags
- [ ] Replace with application-level security or container isolation

### JEP 490: ZGC Non-Generational Mode Removed
- [ ] Remove `-XX:-ZGenerational` flag
- [ ] Remove `-XX:+ZGenerational` flag (no longer needed, it's the default)
- [ ] Update GC tuning for generational ZGC behavior
- [ ] Test application with generational-only ZGC

### JEP 479/503: 32-bit x86 Ports Removed
- [ ] Ensure no 32-bit Windows x86 deployment targets
- [ ] Ensure no 32-bit Linux x86 deployment targets
- [ ] Migrate to 64-bit JDK for all environments

## Deprecation Warnings to Address

### JEP 471/498: sun.misc.Unsafe Memory-Access Methods
- [ ] Replace `Unsafe.getInt/putInt` etc. with `VarHandle` for on-heap access
- [ ] Replace `Unsafe.allocateMemory/freeMemory` with `MemorySegment` (FFM API)
- [ ] Replace `Unsafe.compareAndSwap*` with `VarHandle.compareAndSet`
- [ ] Update third-party libraries that use Unsafe
- [ ] Test all memory access patterns thoroughly

### JEP 472: JNI Restrictions
- [ ] Add `--enable-native-access=ALL-UNNAMED` to JVM flags (classpath apps)
- [ ] Add `--enable-native-access=<module>` for modular apps
- [ ] Evaluate migrating JNI code to FFM API (JEP 454)
- [ ] Document all native library dependencies
- [ ] Update deployment scripts with native access flags

## Standard Language Feature Adoption

### JEP 456: Unnamed Variables & Patterns (Java 22)
- [ ] Replace unused catch parameters with `_`
- [ ] Replace unused loop variables with `_`
- [ ] Use `_` in lambda parameters for unused arguments
- [ ] Use unnamed patterns in `instanceof` and `switch` for ignored components

### JEP 513: Flexible Constructor Bodies (Java 25)
- [ ] Identify constructors with factory-method workarounds for pre-super validation
- [ ] Move validation logic before `super()` / `this()` calls
- [ ] Simplify argument preparation for parent constructors

### JEP 511: Module Import Declarations (Java 25)
- [ ] Replace long import lists with `import module java.base` where appropriate
- [ ] Use for scripts, small programs, and utility classes
- [ ] Establish team standard for when to use module imports

### JEP 512: Compact Source Files and Instance Main Methods (Java 25)
- [ ] Consider for scripts, demos, and teaching materials
- [ ] Use instance `main()` for simple programs
- [ ] Note: Not typically needed for production application code

### JEP 467: Markdown Documentation Comments (Java 23)
- [ ] Convert HTML `<b>` / `<i>` to Markdown `**bold**` / `*italic*`
- [ ] Convert `{@code ...}` to backtick `` `...` ``
- [ ] Convert `<ul><li>` lists to Markdown `- ` lists
- [ ] Convert `<pre>` code blocks to fenced code blocks
- [ ] Use `///` prefix for new Markdown-style doc comments
- [ ] Keep `@param`, `@return`, `@throws` tags (still required)

### JEP 485: Stream Gatherers (Java 24)
- [ ] Identify complex stream operations for gatherer conversion
- [ ] Use `Gatherers.windowSliding()` / `windowFixed()` for windowing
- [ ] Use `Gatherers.fold()` / `scan()` for stateful accumulation
- [ ] Use `Gatherers.mapConcurrent()` for parallel mapping
- [ ] Replace custom `Collector` implementations with custom `Gatherer` where appropriate

### JEP 484: Class-File API (Java 24)
- [ ] Replace ASM `ClassReader`/`ClassWriter` with `ClassFile.of().parse()`/`transform()`
- [ ] Migrate `ClassVisitor` patterns to `ClassTransform`
- [ ] Update bytecode generation to use `ClassFile.of().build()`
- [ ] Remove ASM dependency from `pom.xml` / `build.gradle`

### JEP 506: Scoped Values (Java 25)
- [ ] Identify `ThreadLocal` usage suitable for ScopedValue migration
- [ ] Replace `ThreadLocal` with `ScopedValue` for request-scoped data
- [ ] Use `ScopedValue.runWhere()` / `callWhere()` for bounded scoping
- [ ] Prioritize migration in virtual thread codepaths

### JEP 454: Foreign Function & Memory API (Java 22)
- [ ] Evaluate JNI native methods for FFM API migration
- [ ] Use `Arena` for safe memory lifecycle management
- [ ] Use `Linker.nativeLinker()` for calling native functions
- [ ] Replace `ByteBuffer` direct allocation with `MemorySegment` where appropriate

## Runtime Configuration

### JVM Flags Review
- [ ] Remove `-XX:-ZGenerational` (error in 24+)
- [ ] Remove Security Manager flags
- [ ] Add `--enable-native-access=ALL-UNNAMED` if using JNI
- [ ] Remove 32-bit-specific flags
- [ ] Remove `-XX:+UnlockCommercialFeatures` if still present

### AOT Caching (Java 25)
- [ ] Set up AOT cache with `-XX:AOTCacheOutput=app.aot` training run
- [ ] Use `-XX:AOTCache=app.aot` for production runs
- [ ] Benchmark startup time improvements
- [ ] Integrate AOT cache generation into CI/CD pipeline

### Garbage Collection
- [ ] Test with generational ZGC (now the only mode)
- [ ] Evaluate generational Shenandoah (JEP 521)
- [ ] Benefit from compact object headers (JEP 519, automatic)
- [ ] Benefit from G1 region pinning (JEP 423, automatic)

### Module System Updates
- [ ] Add `--enable-native-access` declarations
- [ ] Update service provider configurations if needed

## Security Enhancements

### Quantum-Resistant Cryptography (Java 24)
- [ ] Evaluate ML-KEM (JEP 496) for key encapsulation
- [ ] Evaluate ML-DSA (JEP 497) for digital signatures
- [ ] Plan migration timeline for post-quantum readiness

### JEP 510: Key Derivation Function API (Java 25)
- [ ] Use standard KDF API for key derivation scenarios
- [ ] Replace custom HKDF implementations with `javax.crypto.KDF`

## Testing Strategy

### Functional Testing
- [ ] Run full test suite with Java 25
- [ ] Validate JNI functionality with `--enable-native-access`
- [ ] Test bytecode manipulation changes (ASM → Class-File API)
- [ ] Verify Security Manager removal doesn't affect behavior
- [ ] Test virtual thread workloads (synchronized no longer pins)

### Performance Testing
- [ ] Benchmark startup time (with and without AOT cache)
- [ ] Test GC behavior with generational ZGC
- [ ] Measure memory footprint (compact object headers)
- [ ] Benchmark Stream Gatherer performance vs previous approaches
- [ ] Test virtual thread scalability improvements (JEP 491)

### Compatibility Testing
- [ ] Test on all target deployment platforms (64-bit only)
- [ ] Verify framework compatibility (Spring, Hibernate, etc.)
- [ ] Test APM/monitoring agent compatibility
- [ ] Validate Docker base images

## Migration Implementation

### Phase 1: Build & Breaking Changes (Weeks 1-2)
- [ ] Update build configuration for Java 25
- [ ] Remove Security Manager usage (JEP 486)
- [ ] Remove non-generational ZGC flags (JEP 490)
- [ ] Add `--enable-native-access` for JNI (JEP 472)
- [ ] Update dependencies
- [ ] Get project compiling on Java 25

### Phase 2: Deprecation Warnings (Weeks 3-4)
- [ ] Migrate sun.misc.Unsafe usage (JEP 471/498)
- [ ] Evaluate JNI → FFM API migration (JEP 454)
- [ ] Update third-party libraries with Unsafe/JNI issues

### Phase 3: Feature Adoption (Weeks 5-6)
- [ ] Adopt unnamed variables `_` (JEP 456)
- [ ] Adopt flexible constructor bodies (JEP 513)
- [ ] Use module imports (JEP 511)
- [ ] Convert JavaDoc to Markdown (JEP 467)
- [ ] Adopt Stream Gatherers (JEP 485)
- [ ] Migrate ASM to Class-File API (JEP 484)
- [ ] Adopt Scoped Values (JEP 506)

### Phase 4: Optimization & Deployment (Weeks 7-8)
- [ ] Set up AOT caching (JEP 483/514/515)
- [ ] Tune GC settings
- [ ] Performance benchmarking
- [ ] Update Docker images and deployment scripts
- [ ] Production deployment with monitoring

## Post-Migration Validation

### Performance Monitoring
- [ ] Monitor startup time (expect improvement from AOT, compact headers)
- [ ] Track GC behavior with generational ZGC/Shenandoah
- [ ] Measure memory footprint changes
- [ ] Monitor virtual thread scalability

### Operational Readiness
- [ ] Update Docker base images to Java 25
- [ ] Update CI/CD pipelines
- [ ] Update monitoring and alerting
- [ ] Document all `--enable-native-access` flags

### Code Quality
- [ ] Ensure no runtime warnings for Unsafe usage
- [ ] Ensure no runtime warnings for JNI usage
- [ ] Update coding standards for Java 25 idioms
- [ ] Review JavaDoc for Markdown consistency

## Rollback Plan

- [ ] Maintain Java 21 branch during transition
- [ ] Keep previous build configurations
- [ ] Document rollback procedures
- [ ] Test rollback before production deployment

## Success Criteria

- [ ] All tests passing on Java 25
- [ ] No performance degradation
- [ ] No runtime warnings for Unsafe or JNI
- [ ] Security Manager fully removed
- [ ] AOT caching operational
- [ ] Team comfortable with new language features
- [ ] Successful production deployment
- [ ] Improved startup time and memory footprint