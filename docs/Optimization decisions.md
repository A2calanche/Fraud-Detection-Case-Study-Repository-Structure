# Optimization Decisions

## Overview

This document explains the most important technical decisions made during the design and operation of the fraud-detection pipeline.

The focus is not on theoretical best practices but on practical trade-offs observed in a production-like e-commerce environment.

---

## Why Hourly Batches Instead of Streaming?

At first glance, streaming appears attractive for fraud detection.

However, hourly batch processing provided several advantages:

### Benefits

* Lower infrastructure complexity
* Easier troubleshooting
* Simpler deployment model
* Predictable execution schedules
* Reduced operational overhead

### Trade-Off

Alerts were generated minutes later than a streaming solution.

For this use case, the delay was acceptable because orders were still reviewed before downstream fulfillment processes.

---

## Why Spark SQL Instead of Pure PySpark?

Most analytical logic was implemented in Spark SQL.

### Benefits

* Easier to read
* Easier to review
* Better adoption by analysts
* Simpler aggregation logic

Example workloads:

* Shared email detection
* Shared phone detection
* Address matching
* Historical customer analysis

PySpark remained useful for orchestration and workflow management.

---

## Why GROUP BY Over Heavy Window Functions?

One of the most important optimization decisions.

### Problem

Window functions can unintentionally multiply rows and increase processing costs.

Example:

```sql
COUNT(*) OVER(PARTITION BY customer_id)
```

on large datasets may produce significantly larger intermediate results.

### Alternative

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
```

### Benefits

* Smaller execution plans
* Lower memory usage
* Reduced shuffle operations
* Faster execution times

---

## Why Three-Step Phone Normalization?

Raw phone numbers were highly inconsistent.

Examples:

```text
(92) 99999-9999
92999999999
+55 92 99999-9999
```

Without normalization, duplicate detection becomes unreliable.

### Strategy

1. Remove formatting characters
2. Standardize structure
3. Validate output

### Result

Improved customer matching and more reliable fraud indicators.

---

## Why Normalize Addresses Before Comparison?

Address quality was one of the largest data-quality challenges.

Examples:

```text
Rua São José
RUA SAO JOSE
Rua Sao Jose
```

All refer to the same location.

### Approach

* Upper-case conversion
* Character translation
* Regular-expression cleanup
* Whitespace normalization

### Result

Higher matching accuracy and fewer missed correlations.

---

## Why Use Delta Lake for Alert Storage?

Fraud alerts should never disappear.

### Requirements

* Historical retention
* Auditability
* Reliability
* Recovery capability

### Benefits of Delta Lake

* ACID transactions
* Time-travel capabilities
* Reliable writes
* Historical traceability

This made it suitable for alert-history storage.

---

## Why Store Alert History Permanently?

Operational teams frequently revisit historical decisions.

Typical scenarios:

* Fraud investigations
* Rule tuning
* Trend analysis
* Audit requests

Permanent history provides valuable context for future analysis.

---

## Why Use SMTP Email Notifications?

The goal was to provide a lightweight alerting mechanism without introducing unnecessary complexity.

Benefits:

* Easy integration
* Broad compatibility
* Minimal operational overhead
* Immediate visibility for reviewers

The objective was reliability rather than sophisticated workflow automation.

---

## Why Exclude Certain Orders Early?

Some transactions consistently generated false positives.

Examples:

* Store pickups
* Authorized pickup locations
* Assisted sales

Evaluating these orders consumed resources while producing limited investigative value.

Excluding them early improved signal quality and reviewer efficiency.

---

## Key Principle

The architecture consistently favored:

* Simplicity over complexity
* Reliability over novelty
* Explainability over sophistication
* Operational efficiency over theoretical optimization

These principles allowed the solution to remain maintainable while processing large volumes of e-commerce transactions.
