
# Fraud Detection Pipeline — Case Study

A fraud detection case study inspired by a real-world Brazilian e-commerce environment, documenting the architectural decisions, data processing patterns, and business-rule design used to identify potentially fraudulent orders at scale.

## Important Disclaimer

This repository does **not** contain proprietary source code, notebooks, SQL scripts, datasets, or internal assets from any employer.

The original implementation was developed within a corporate environment between **September 2025 and April 2026**, where I participated in the design, development, operation, and maintenance of the solution. Due to confidentiality obligations and intellectual property restrictions, the actual production code cannot be shared or reproduced.

The content of this repository is therefore presented as a **generic technical case study**, focused on architecture, design decisions, data quality patterns, and fraud detection strategies that can be adapted to other e-commerce businesses.

All examples, diagrams, and rules have been anonymized and generalized.

## Overview

* **Stack**: Databricks, PySpark, Spark SQL, Delta Lake, SMTP Email Alerts
* **Scale**: Hourly batch processing of incoming orders
* **Detection**: Rule-based fraud screening and behavioral pattern analysis
* **Alerting**: Automated email notifications for fraud review teams
* **Status**: Historical case study based on a production environment

## Why This Repository?

This repository is not intended to replicate a production system.

Instead, it documents the engineering decisions, trade-offs, and lessons learned while building and operating a fraud detection workflow under real-world constraints.

The objective is to share reusable architectural patterns that other teams can adapt to their own business requirements.

## Fraud Detection Patterns Documented

Examples of generic fraud indicators discussed in this repository include:

* First-time customer purchases
* Shared phone numbers across multiple accounts
* Shared email addresses across multiple customers
* Recently created delivery addresses
* Order velocity anomalies
* Repeated customer-identification patterns
* Email-domain analysis
* Pickup-location exclusions
* Assisted-sales exclusions
* Address normalization and matching
* Historical transaction correlation

These examples are intentionally generic and should be adapted to each organization's own risk model and business processes.

## Business Rules Covered

The case study discusses patterns such as:

* Detection of shared emails across multiple customer accounts
* Detection of shared phone numbers after normalization
* Filtering customers using common public email providers (Gmail, Outlook, Hotmail, Yahoo, etc.)
* Excluding orders marked as in-store pickup (`PICKUP = 'YES'`)
* Excluding pickup-point or stand locations based on normalized address matching
* Excluding assisted sales where the seller identifier differs from the default self-service channel (for example, seller code different from `9999*`)
* Deduplication and historical correlation logic
* Address and contact-data standardization
  

## Key Decisions Documented

* Why GROUP BY was preferred over certain window-function approaches
* Three-step phone normalization strategy
* Address normalization using `regexp_replace()` and character translation
* Importance of `NULLIF(TRIM(...), '')` for data quality
* Delta Lake for historical traceability
* ROW_NUMBER() deduplication patterns
* Cost-effective alerting architecture using SMTP notifications

## What's Inside

* **Architecture**: System design and data flow
* **ADRs**: Architecture Decision Records
* **Diagrams**: Mermaid diagrams and process flows
* **Examples**: Generic SQL and data-normalization patterns
* **Lessons Learned**: Operational and engineering insights

## How to Use This

Recommended reading order:

1. `docs/architecture.md`
2. `adr/`
3. `diagrams/`
4. `examples/`
5. `docs/lessons-learned.md`

## Built With

* Databricks Runtime
* Apache Spark (PySpark + Spark SQL)
* Delta Lake
* SMTP-based email notifications

## What's NOT Here

* Proprietary source code
* Production notebooks
* Company datasets
* Internal configurations
* Credentials or connection strings
* Exact business-rule implementations
* Employer intellectual property

## License

MIT — The concepts, patterns, and documentation may be reused with attribution.

## Notes

The original system operated in a large Brazilian retail and e-commerce environment between September 2025 and April 2026.

This repository documents generalized engineering patterns derived from that experience and is intended for educational, architectural, and professional portfolio purposes only.


---
