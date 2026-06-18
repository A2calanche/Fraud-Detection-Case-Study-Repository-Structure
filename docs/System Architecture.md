## Architecture Disclaimer

This document describes a generalized fraud-detection architecture inspired by a real-world e-commerce implementation.

Table names, field names, business rules, workflows, and examples have been anonymized and simplified for documentation purposes. The original production implementation remains confidential and is not reproduced here.

The patterns described in this document are intended to demonstrate architectural approaches that can be adapted to different e-commerce environments and business requirements.

---

## High-Level Flow

```text
Raw Orders
    ↓
Databricks Processing
    ↓
Deduplication & Data Normalization
    ↓
Feature Engineering
    ↓
Business Rule Evaluation
    ↓
Risk Scoring
    ↓
Email Alert Notification
    ↓
Manual Review Threshold
```

---

## Components

### 1. Data Ingestion

**Source**

* E-commerce order platform
* Database exports
* Internal reporting datasets

**Frequency**

* Hourly batch processing

**Typical Fields**

* order_id
* customer_id
* email
* phone_number
* shipping_address
* purchase_amount
* payment_method
* seller_id
* pickup_indicator
* order_timestamp

---

### 2. Data Cleaning & Normalization

Data quality was critical because fraud signals often depend on matching customer information across multiple records.

Key normalization steps included:

#### Deduplication

```sql
ROW_NUMBER() OVER (
    PARTITION BY order_id
    ORDER BY created_at DESC
)
```

Used to eliminate duplicate order records before rule evaluation.

#### Phone Normalization

Multi-step normalization process:

1. Remove punctuation and formatting characters
2. Standardize country and area codes
3. Validate final structure

Purpose:

* Detect shared phone numbers
* Improve customer matching accuracy

#### Address Normalization

Addresses were standardized using:

* `regexp_replace()`
* `translate()`
* upper-case conversion
* whitespace cleanup

Purpose:

* Detect repeated delivery locations
* Match pickup locations despite formatting differences

#### Null Handling

```sql
NULLIF(TRIM(column_name), '')
```

Used extensively to avoid treating blank strings as valid values.

---

### 3. Feature Engineering

The fraud engine generated behavioral indicators for every order.

Examples include:

#### Customer History

* First-time buyer detection
* Previous order count
* Historical address usage
* Historical contact information usage

#### Shared Contact Information

Detection of:

* Shared email addresses across multiple customer accounts
* Shared phone numbers across multiple customer accounts

These indicators frequently revealed account reuse or suspicious registration patterns.

#### Email Analysis

Evaluation of email domains.

Examples:

* Gmail
* Outlook
* Hotmail
* Yahoo

Public email providers were analyzed differently from corporate domains depending on business requirements.

#### Address Analysis

Detection of:

* New delivery addresses
* Repeated addresses across multiple customers
* Known pickup-point addresses
* Known stand locations

Address matching relied on normalized address data rather than raw text comparison.

#### Purchase Behavior

Examples:

* Unusual order activity
* Rapid purchase sequences
* Payment-method changes
* Customer activity anomalies

---

### 4. Rule Engine

The rule engine evaluated each order independently using a collection of business rules.

Rules contributed to an overall risk score.

Example categories:

#### Positive Risk Indicators

* First purchase
* Shared email
* Shared phone number
* New delivery address
* Suspicious purchase velocity
* Repeated customer-identification patterns

#### Exclusion Rules

Certain orders were intentionally excluded from fraud review because they represented legitimate operational scenarios.

Examples:

##### Store Pickup Orders

```text
pickup_indicator = 'YES'
```

Orders marked for store pickup were ignored.

##### Authorized Pickup Locations

Orders shipped to predefined pickup stands or collection points were excluded after address normalization and matching.

##### Assisted Sales

```text
seller_id <> default_self_service_channel
```

Orders associated with a human sales representative were excluded because they had already undergone additional validation during the sales process.

These exclusion rules significantly reduced false positives.

#### Risk Scoring

```text
Rule A + Rule B + Rule C
            ↓
      Risk Score
            ↓
     Above Threshold?
            ↓
          Alert
```

Threshold values were adjusted over time based on operational feedback.

---

### 5. Alert System

When an order exceeded the review threshold:

1. Alert record created
2. Alert stored in Delta Lake
3. Email notification generated
4. Fraud-review team notified

#### Alert Storage

Example table:

```text
historico_alertas_detalle
```

Purpose:

* Historical tracking
* Auditability
* Investigation support
* Operational reporting

#### Notification Method

* SMTP email delivery
* Automated alert generation
* Structured alert content for manual review

---

### 6. Monitoring

Operational monitoring focused on:

* Job execution status
* Processing duration
* Alert volume trends
* Rule-trigger frequency
* Pipeline failures

Fraud-review outcomes could later be used to refine rule effectiveness.

---

## Conceptual Data Model

The following model is illustrative and does not represent the original production schema.

### orders

```text
order_id (PK)
customer_id (FK)
email
phone_number
purchase_amount
shipping_address
payment_method
seller_id
pickup_indicator
created_at
```

### customers

```text
customer_id (PK)
email
phone_number
first_order_date
metadata
```

### historico_alertas_detalle

```text
alert_id (PK)
order_id (FK)
customer_id (FK)
risk_score
flagged_rules
alert_sent_at
created_at
```

---

## Why These Choices

### Hourly Batch Processing Instead of Streaming

Benefits:

* Lower operational complexity
* Easier troubleshooting
* Predictable execution windows

For this use case, near-real-time processing provided sufficient response time.

### Spark SQL + PySpark

Combination chosen because:

* SQL simplifies aggregations
* PySpark handles orchestration and transformations
* Easier maintenance for data teams

### Delta Lake

Provided:

* ACID transactions
* Reliable history retention
* Auditability
* Simplified operational recovery

### GROUP BY Over Heavy Window Usage

Where appropriate, aggregations were performed using GROUP BY instead of complex window-function approaches.

Benefits:

* Reduced row multiplication
* Lower computational overhead
* Simpler query plans
* Easier maintenance
