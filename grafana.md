# Grafana

## Where Does Grafana Fit?

In the previous document, we learned that:

```text
Spring Boot Application
        |
        v
/actuator/prometheus
        |
        v
Prometheus
```

Prometheus collects and stores metrics.

However, Prometheus is mainly a metrics platform and time-series database.

We still need a way to:

* Visualize metrics
* Create dashboards
* Compare metrics
* Monitor system health easily

This is where Grafana comes in.

---

# What is Grafana?

Grafana is a visualization and dashboarding platform.

Its job is to:

* Connect to data sources
* Query data
* Visualize data
* Build dashboards
* Create alerts

Think of it as:

```text
Prometheus stores the data

Grafana displays the data
```

---

# Does Grafana Store Metrics?

No.

Grafana is not a metrics database.

It typically does not store the metrics itself.

Instead, Grafana fetches data from data sources.

Example:

```text
Grafana
    |
    v
Prometheus
```

When a user opens a dashboard:

```text
Grafana
    |
    | Query
    v
Prometheus
```

Grafana requests the required metrics from Prometheus and displays them.

---

# What is a Data Source?

A Data Source is a system that contains data.

Examples:

* Prometheus
* MySQL
* PostgreSQL
* Loki
* Elasticsearch
* CloudWatch

Grafana can connect to many different data sources.

Example:

```text
          Grafana
              |
    -----------------------
    |          |          |
    v          v          v
Prometheus   MySQL      Loki
```

---

# Adding a Prometheus Data Source

In Grafana:

```text
Connections
      |
      v
Data Sources
      |
      v
Prometheus
```

We provide the Prometheus URL.

Example:

```text
http://localhost:9090
```

Grafana then verifies the connection.

After that, Grafana can query Prometheus.

---

# Dashboard

A dashboard is a collection of visualizations.

Example:

```text
Application Dashboard

+----------------------+
| JVM Memory Usage     |
+----------------------+

+----------------------+
| CPU Usage            |
+----------------------+

+----------------------+
| Active Threads       |
+----------------------+
```

A dashboard provides a single place to monitor a system.

---

# Panel

A panel is an individual visualization inside a dashboard.

Examples:

```text
Memory Usage Graph

CPU Usage Graph

Thread Count Graph

Request Count Graph
```

Each panel displays one specific piece of information.

---

# How Does Grafana Get Data?

Suppose we create a panel for:

```text
JVM Thread Count
```

Grafana sends a query to Prometheus.

Example:

```promql
jvm_threads_live_threads
```

Prometheus returns the data.

Grafana then renders a graph.

Flow:

```text
Grafana
     |
     | Query
     v
Prometheus
     |
     | Metric Data
     v
Grafana Panel
```

---

# Why Do Metrics Appear in the Query Builder?

When you configured Prometheus as a datasource and opened the Query Builder, you saw metrics like:

```text
jvm_threads_live_threads

jvm_memory_used_bytes

http_server_requests_seconds_count
```

These metrics come from Prometheus.

Grafana asks Prometheus:

```text
What metric names do you have?
```

Prometheus returns the list.

Grafana displays them in the dropdown.

---

# What Happens When We Select a Metric?

Suppose we select:

```text
jvm_threads_live_threads
```

Grafana generates a PromQL query:

```promql
jvm_threads_live_threads
```

and sends it to Prometheus.

Prometheus executes the query and returns the results.

---

# Label Filters

Prometheus metrics often contain labels.

Example:

```text
jvm_threads_live_threads{
    application="order-service",
    job="order-service",
    instance="localhost:8080"
}
```

Grafana automatically discovers these labels.

This is why the Query Builder shows:

```text
Select Label
Select Value
```

Example:

```text
application = order-service
```

Grafana converts that into:

```promql
jvm_threads_live_threads{application="order-service"}
```

---

# PromQL

PromQL is the query language used by Prometheus.

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

# Does Grafana Only Work With Prometheus?

No.

Grafana supports many data sources.

Examples:

| Data Source   | Query Language |
| ------------- | -------------- |
| Prometheus    | PromQL         |
| MySQL         | SQL            |
| PostgreSQL    | SQL            |
| Loki          | LogQL          |
| Elasticsearch | Query DSL      |

The query builder changes depending on the selected datasource.

---

# Why Can Grafana Work With Different Data Sources?

Every datasource has a Grafana plugin.

Example:

```text
Prometheus Plugin

MySQL Plugin

Loki Plugin
```

Each plugin knows:

* How to connect to the datasource
* How to execute queries
* How to interpret the results

---

# Internal Grafana Data Model

Different data sources return data in different formats.

Examples:

### Prometheus

```text
Time + Metric Values
```

### MySQL

```text
Rows and Columns
```

### Loki

```text
Log Entries
```

Grafana converts all of them into an internal format before rendering charts.

This is why Grafana can support many different backends.

---

# Typical Monitoring Architecture

A common production setup looks like:

```text
Spring Boot Application
          |
          v
      Micrometer
          |
          v
Prometheus Registry
          |
          v
/actuator/prometheus
          |
          | Scrape
          v
      Prometheus
          |
          | Query
          v
       Grafana
          |
          v
      Dashboards
```

---

# Complete End-to-End Flow

## Step 1

Spring Boot generates metrics.

```text
CPU Usage
Memory Usage
Thread Count
Request Count
```

---

## Step 2

Micrometer stores them in PrometheusMeterRegistry.

---

## Step 3

The application exposes:

```text
/actuator/prometheus
```

---

## Step 4

Prometheus scrapes:

```text
GET /actuator/prometheus
```

and stores the metrics.

---

## Step 5

Grafana queries Prometheus.

---

## Step 6

Grafana displays the metrics using dashboards and panels.

---

# Final Summary

### Actuator

Exposes operational endpoints.

### Micrometer

Collects metrics.

### Meter

Measures a specific value.

### Meter Registry

Stores metrics collected by Micrometer.

### PrometheusMeterRegistry

Stores metrics and converts them into Prometheus format.

### /actuator/prometheus

Returns metrics in Prometheus format.

### Prometheus

Scrapes, stores and queries metrics.

### Scraping

Prometheus periodically calling `/actuator/prometheus`.

### Grafana

Reads data from Prometheus and visualizes it.

### Dashboard

Collection of visualizations.

### Panel

Single visualization within a dashboard.

### PromQL

Query language used to query Prometheus metrics.

---

# Reading Order Recap

1. **01-observability-fundamentals.md**
2. **02-spring-boot-actuator-and-micrometer.md**
3. **03-prometheus.md**
4. **04-grafana.md**

Following these four documents in order gives a complete understanding of the monitoring flow from a Spring Boot application all the way to Grafana dashboards.
