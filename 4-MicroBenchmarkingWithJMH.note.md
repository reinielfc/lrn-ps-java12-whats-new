# 4. Micro-benchmarking with JMH

## What is Micro-benchmarking?

- measure execution time
- small pieces of code
- compare alternatives
- prevent performance regressions
- don't use `System.currentTimeMilis()` to benchmark code, it ignores mutilple factors about the JVM,
    - like warmup and optimization of code

## Why JMH?

- Java Micro-benchmarking Harness
- reproducibility
- handles JVM warm-up
- consistent reporting
- multi-threading support
- as of Java 12, JMH is used by the JDK for micro-benchmarks

## JMH Basics

annotation-based micro-benchmarks

initialize a benchmark project with maven:
```shell
mvn archetype:generate \
  -DinteractiveMode=false \
  -DarchetypeGroupId=org.openjdk.jmh \
  -DarchetypeArtifactId=jmh-java-benchmark-archetype \
  -DgroupId=org.sample \
  -DartifactId=jmh-number-verification-performance-test \
  -Dversion=1.0
```

the generated code includes the following class:
```java
@BenchmarkMode(Mode.AverageTime) // otherwise it will return throughput (# operations per time unit)
@Fork(1) // 1 JVM (default is 5)
public class MyBenchmark {
    @Benchmark
    public void testMethod() {
        // benchmark code
    }
}
```

- warms up JVM
- does multiple runs
- forks to multiple JVMs
- has `@Setup` and `@Teardown` annotations
- similar to a Test, but for performance of app

## JMH Pitfalls

- dead code elimination
    - if you write statements w/o side effects that don't leave the scope of the test method,
    - the java compiler might conclude that these statements are pointless and won't emit bytecode for it
    - solved by returning a value from the benchmark method
- other compiler optimizations
- assumptions

## Demo: [Writing a Micro-benchmark using JMH](src/main/java/org/sample/MyBenchmark.java)

```shell
mvn clean package # to package our benchmark
```

results in a jar file in the target dir called "benchmarks.jar", run with

```shell
java -jar target/benchmarks.jar
```
output summary

```text
Benchmark              (toParse)  Mode  Cnt   Score   Error  Units
MyBenchmark.parseInt           1  avgt    5   4.642 ± 0.107  ns/op
MyBenchmark.parseInt       12345  avgt    5  10.203 ± 0.562  ns/op
MyBenchmark.parseInt  2147483647  avgt    5  15.060 ± 0.359  ns/op
```

don't assume anything with these numbers, this is only the starting point, investigate further w/ autotools like a profiler

```text
REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.
```
