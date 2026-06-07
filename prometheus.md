# Prometheus

## Where Does Prometheus Fit?

In the previous document, we learned that Spring Boot + Micrometer can expose metrics through:

```text
/actuator/prometheus
```

Now we need something that can:

* Collect those metrics
* Store them
* Query them
* Generate alerts

This is where Prometheus comes in.

---

# What is Prometheus?

Prometheus is a monitoring and metrics platform.

Its responsibilities are:

1. Collect metrics
2. Store metrics
3. Query metrics
4. Generate alerts

Think of Prometheus as a central metrics database.

---

# Why Do We Need Prometheus?

Imagine we have multiple applications:

```text
Order Service
Payment Service
Inventory Service
Notification Service
```

Each application exposes:

```text
/actuator/prometheus
```

If every developer has to visit each application separately to check metrics:

```text
Order Service      -> Metrics
Payment Service    -> Metrics
Inventory Service  -> Metrics
Notification Service -> Metrics
```

monitoring becomes difficult.

Instead, Prometheus collects metrics from all applications and stores them in one place.

```text
Order Service
Payment Service
Inventory Service
Notification Service
        |
        v
    Prometheus
```

Now everyone can look at a single system for monitoring.

---

# Prometheus Pull Model

Prometheus follows a:

```text
Pull Model
```

Instead of applications sending metrics to Prometheus,

Prometheus periodically asks applications for metrics.

Example:

```text
Prometheus
      |
      | GET /actuator/prometheus
      v
Spring Boot Application
```

Prometheus collects the response and stores it.

---

# What is Scraping?

Scraping means:

> Prometheus calling an application's metrics endpoint and collecting metrics.

Example:

```text
Prometheus
      |
      | GET /actuator/prometheus
      v
Spring Boot Application
```

The response may look like:

```text
jvm_threads_live_threads 25
jvm_memory_used_bytes 734003200
process_cpu_usage 0.12
```

Prometheus stores these values.

---

# Scrape Interval

Prometheus does not scrape continuously.

Instead it scrapes at fixed intervals.

Example:

```yaml
global:
  scrape_interval: 15s
```

Meaning:

```text
Every 15 seconds:
    Call /actuator/prometheus
    Collect metrics
```

---

# Prometheus Configuration

Example:

```yaml
scrape_configs:
  - job_name: order-service

    metrics_path: /actuator/prometheus

    static_configs:
      - targets:
          - localhost:8080
```

Meaning:

```text
Application:
    localhost:8080

Endpoint:
    /actuator/prometheus

Group Name:
    order-service
```

---

# What is job_name?

A job is a logical group of scrape targets.

Example:

```yaml
job_name: order-service
```

Prometheus automatically adds:

```text
job="order-service"
```

to all metrics scraped from that job.

---

# What is an Instance?

An instance represents a specific application instance.

Example:

```text
order-service-1:8080
order-service-2:8080
order-service-3:8080
```

Prometheus automatically adds:

```text
instance="order-service-1:8080"
```

or

```text
instance="order-service-2:8080"
```

etc.

---

# Job vs Instance

Example:

```text
Job:
    order-service

Instances:
    order-service-1:8080
    order-service-2:8080
    order-service-3:8080
```

Result:

```text
job="order-service"
instance="order-service-1:8080"
```

The job identifies the service.

The instance identifies the specific running application.

---

# Application Tag from Spring Boot

Spring Boot can add:

```yaml
management:
  metrics:
    tags:
      application: ${spring.application.name}
```

Example:

```yaml
spring:
  application:
    name: order-service
```

Result:

```text
application="order-service"
```

being attached to metrics.

---

# job vs application

Many beginners notice:

```text
job="order-service"
application="order-service"
```

and wonder why both exist.

The reason is:

### job

Added by:

```text
Prometheus
```

Represents:

```text
How Prometheus grouped the target.
```

---

### application

Added by:

```text
Spring Boot / Micrometer
```

Represents:

```text
What the application identifies itself as.
```

In small setups both often have the same value.

In larger environments they may differ.

---

# What Happens During Scraping?

Suppose:

### 10:00

```text
http_server_requests_seconds_count 100
```

Prometheus stores:

```text
10:00 -> 100
```

---

### 10:15

```text
http_server_requests_seconds_count 120
```

Prometheus stores:

```text
10:15 -> 120
```

---

### 10:30

```text
http_server_requests_seconds_count 150
```

Prometheus stores:

```text
10:30 -> 150
```

Prometheus keeps building a history of metric values over time.

This is why Prometheus is called a:

```text
Time-Series Database
```

---

# Why Time-Series Data Matters

Prometheus does not only store the latest value.

It stores:

```text
Value + Timestamp
```

Example:

| Time  | Active Threads |
| ----- | -------------- |
| 10:00 | 20             |
| 10:15 | 25             |
| 10:30 | 22             |

This makes it possible to draw graphs and analyze trends.

---

# Querying Metrics

Prometheus provides a query language called:

```text
PromQL
```

Examples:

### Current Thread Count

```promql
jvm_threads_live_threads
```

---

### Memory Usage

```promql
jvm_memory_used_bytes
```

---

### Filter by Application

```promql
jvm_threads_live_threads{application="order-service"}
```

---

### Filter by Job

```promql
jvm_threads_live_threads{job="order-service"}
```

---

# Why Centralize Metrics?

Without Prometheus:

```text
App 1 -> Metrics
App 2 -> Metrics
App 3 -> Metrics
App 4 -> Metrics
```

Everyone must inspect applications separately.

With Prometheus:

```text
App 1
App 2
App 3
App 4
   |
   v
Prometheus
```

Benefits:

* Single source of truth
* Easier monitoring
* Easier alerting
* Easier comparison between services
* Historical analysis

---

# Example Monitoring Flow

```text
Order Service
Payment Service
Inventory Service
        |
        | Scrape
        v
    Prometheus
```

Prometheus now contains metrics from all applications.

Teams can query and analyze everything from one place.

---

# End-to-End Flow So Far

```text
Spring Boot Application
          |
          v
      Micrometer
          |
          v
PrometheusMeterRegistry
          |
          v
/actuator/prometheus
          |
          | Scrape
          v
      Prometheus
          |
          v
      Time-Series DB
```

At this point we have successfully:

1. Generated metrics in Spring Boot.
2. Exposed metrics through `/actuator/prometheus`.
3. Collected metrics using Prometheus.
4. Stored metrics in a central location.

In the next document, we will learn how Grafana connects to Prometheus and turns these metrics into dashboards and visualizations.
