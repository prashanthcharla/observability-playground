# Spring Boot Actuator & Micrometer

## Where Does This Fit?

From the previous notes, we learned that observability relies on:

* Metrics
* Logs
* Traces

In this document, we focus on the **metrics** part.

Our goal is to understand:

```text
Spring Boot Application
        |
        v
     Metrics
```

and how those metrics are generated.

---

# What is Spring Boot Actuator?

Spring Boot Actuator adds production-ready features to a Spring Boot application.

Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Actuator provides operational endpoints such as:

```text
/actuator/health
/actuator/info
/actuator/metrics
```

These endpoints help us understand the current state of the application.

---

# What Comes with Actuator?

When we add:

```xml
spring-boot-starter-actuator
```

Spring Boot automatically pulls:

```text
micrometer-core
```

as a dependency.

Micrometer is the metrics collection library used by Spring Boot.

Think of it like:

```text
Spring Boot
      |
      v
  Micrometer
      |
      v
   Metrics
```

---

# What is Micrometer?

Micrometer is responsible for collecting metrics from the application.

Examples:

* JVM Memory Usage
* CPU Usage
* Thread Count
* Garbage Collection
* HTTP Requests
* Database Metrics

Without Micrometer, Spring Boot would not know how to collect these metrics.

---

# What is a Meter?

A Meter is an object that measures something.

Examples:

```text
JVM Memory Meter
Thread Meter
CPU Meter
HTTP Request Meter
```

Each meter continuously tracks a specific metric.

For example:

```text
Active Threads = 25
Heap Memory = 512 MB
Request Count = 1000
```

---

# What is a Meter Registry?

A Meter Registry is a container that stores all meters.

Think of it as:

```text
Meter Registry
    |
    +-- JVM Meter
    +-- Thread Meter
    +-- CPU Meter
    +-- HTTP Meter
```

When the application starts:

1. Spring Boot creates meters.
2. Meters get registered in the registry.
3. As the application runs, meter values keep changing.
4. The registry always contains the latest metric values.

---

# SimpleMeterRegistry

When only Actuator is added:

```xml
spring-boot-starter-actuator
```

Spring Boot creates:

```text
SimpleMeterRegistry
```

This registry stores metrics in memory.

Example:

```text
Active Threads = 22
Heap Memory = 650 MB
CPU Usage = 35%
```

The values are continuously updated while the application runs.

---

# Why Do We Need Another Registry?

SimpleMeterRegistry stores metrics in memory.

But monitoring systems such as:

* Prometheus
* Datadog
* New Relic
* Dynatrace

need metrics in a format they understand.

For this reason, Micrometer provides different registry implementations.

---

# Prometheus Registry

To integrate with Prometheus, we add:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Important:

```text
micrometer-registry-prometheus
        |
        +-- micrometer-core
```

Since Actuator already brings Micrometer Core, Maven resolves it only once.

---

# What Changes After Adding Prometheus Registry?

After adding:

```xml
micrometer-registry-prometheus
```

Spring Boot auto-configures:

```text
PrometheusMeterRegistry
```

Now the application's meters are registered into:

```text
PrometheusMeterRegistry
```

instead of relying only on SimpleMeterRegistry.

Conceptually:

```text
Before

Micrometer
     |
     v
SimpleMeterRegistry
```

```text
After

Micrometer
     |
     v
PrometheusMeterRegistry
```

---

# What Does PrometheusMeterRegistry Do?

It still stores all metric values.

Example:

```text
Heap Memory = 700 MB
Active Threads = 30
CPU Usage = 40%
```

But it has an additional capability:

```text
Convert metrics into Prometheus format
```

Example:

```text
jvm_memory_used_bytes 734003200
jvm_threads_live_threads 30
```

This is the format expected by Prometheus Server.

---

# How Does /actuator/prometheus Work?

When:

```xml
spring-boot-starter-actuator
```

and

```xml
micrometer-registry-prometheus
```

are both present,

Spring Boot exposes:

```text
/actuator/prometheus
```

Example:

```text
GET /actuator/prometheus
```

Internally:

```text
PrometheusMeterRegistry.scrape()
```

is called.

The registry:

1. Reads all meter values.
2. Converts them into Prometheus format.
3. Returns the response.

Example:

```text
jvm_memory_used_bytes 734003200
jvm_threads_live_threads 30
process_cpu_usage 0.15
```

---

# Can We Have Multiple Registries?

Yes.

This is one of Micrometer's strengths.

Example:

```xml
spring-boot-starter-actuator

micrometer-registry-prometheus

micrometer-registry-datadog
```

Micrometer can publish the same metrics to multiple registries.

Conceptually:

```text
                 Micrometer
                      |
        -----------------------------
        |                           |
        v                           v
Prometheus Registry         Datadog Registry
```

Both registries receive the same metrics.

---

# Pull vs Push Monitoring Systems

Different monitoring systems work differently.

## Pull Model

Example:

```text
Prometheus
```

Prometheus periodically calls the application to collect metrics.

```text
Prometheus
      |
      | GET /actuator/prometheus
      v
Spring Boot Application
```

---

## Push Model

Examples:

```text
Datadog
New Relic
```

The application pushes metrics directly to the monitoring backend.

```text
Application
      |
      v
Monitoring Tool
```

No scrape endpoint is required.

---

# Important Note About Multiple Registries

A registry does not automatically mean an actuator endpoint.

For example:

```text
Prometheus Registry
```

needs:

```text
/actuator/prometheus
```

because Prometheus follows a pull model.

However, a push-based monitoring tool may not expose any actuator endpoint at all.

It may simply push metrics directly from its registry to the monitoring server.

---

# End-to-End Flow

```text
Spring Boot Application
          |
          v
      Micrometer
          |
          v
      Meters
          |
          v
   Meter Registry
          |
          v
PrometheusMeterRegistry
          |
          v
/actuator/prometheus
```

At this point, the application is generating metrics and exposing them in a Prometheus-compatible format.

In the next document, we will learn how Prometheus scrapes these metrics, stores them, and makes them available for querying.
