# Observability Fundamentals

## What is Observability?

Observability is the ability to understand what is happening inside a system by looking at the data it produces.

It helps us answer questions such as:

* Is the system healthy?
* What is failing?
* Why is it failing?
* Where is the problem occurring?

The goal of observability is not only to detect problems but also to investigate and find their root cause.

---

# Monitoring vs Observability

A simple way to think about the difference:

## Monitoring

Monitoring tells us:

> What happened?

and

> When did it happen?

Examples:

* CPU usage reached 90%
* Application is down
* Error rate increased
* Response time became slow

Monitoring is good at detecting known problems and generating alerts.

### Car Analogy

Monitoring is like the dashboard in your car.

It can show:

* Check Engine light
* Low fuel warning
* High engine temperature

It tells you something is wrong.

---

## Observability

Observability helps answer:

> How did it happen?

and

> Why did it happen?

It helps engineers investigate the root cause of issues.

### Car Analogy

Observability is like connecting a diagnostic scanner to the car.

It helps determine:

* Which component failed
* What sequence of events caused the failure
* Why the failure occurred

---

# Why Monitoring Alone Is Not Enough

Imagine a monitoring system reports:

```text
Error rate increased to 20%
```

Monitoring successfully detected the problem.

However, it does not tell us:

* Which service failed?
* Which API failed?
* Which user requests are affected?
* Which code path caused the issue?

To answer those questions we need observability.

---

# The Three Pillars of Observability

Observability is commonly built using three types of data:

1. Metrics
2. Logs
3. Traces

---

# 1. Metrics

Metrics are numerical measurements collected over time.

Examples:

* CPU usage
* Memory usage
* Active thread count
* HTTP request count
* Error count

Example:

```text
CPU Usage = 65%
Memory Usage = 1.2 GB
Active Threads = 40
```

Metrics are useful for:

* Monitoring system health
* Creating dashboards
* Triggering alerts

---

# 2. Logs

Logs are timestamped records of events happening inside the application.

Examples:

```text
User login successful
Order created
Payment failed
Database connection timeout
```

Logs provide detailed information about what happened inside the application.

They are useful for investigating specific problems.

---

# 3. Traces

A trace represents the journey of a request through multiple services.

Example:

```text
User Request
      |
      v
Order Service
      |
      v
Payment Service
      |
      v
Inventory Service
```

A trace helps answer:

* Which service was slow?
* Which service failed?
* How long was spent in each service?

Traces are especially useful in microservice architectures.

---

# How Metrics, Logs and Traces Work Together

Suppose a customer reports that order placement is slow.

### Metrics tell us

```text
Response time increased
```

### Logs tell us

```text
Database query timeout occurred
```

### Traces tell us

```text
Order Service -> Payment Service -> Inventory Service

Inventory Service consumed 95% of the total response time.
```

Together they help identify the root cause much faster.

---

# Key Takeaway

Observability is the ability to understand the internal state of a system using:

* Metrics
* Logs
* Traces

Monitoring is an important part of observability.

```text
Observability
    |
    +-- Metrics (Monitoring)
    +-- Logs
    +-- Traces
```

Monitoring tells us that a problem exists.

Observability helps us understand why the problem exists.
