# Detection Rules

## Disclaimer

The rules described in this document are generalized fraud-detection patterns inspired by a real-world e-commerce environment.

The original production implementation, scoring model, thresholds, datasets, and business logic remain confidential. The examples presented here are intended to demonstrate common fraud-detection approaches that can be adapted to other organizations.

---

## Rule Engine Philosophy

The system was designed around a rule-based approach rather than a machine learning model.

Reasons included:

* Faster implementation
* Easier explainability
* Lower operational complexity
* Immediate feedback from fraud analysts
* Ability to adjust rules without retraining models

Each rule contributed to an overall risk score.

Orders exceeding a predefined threshold were flagged for manual review.

---

## Rule Categories

### Customer History Rules

#### First-Time Buyer

Customers with no previous completed purchases were considered higher risk than returning customers.

Why it matters:

* Fraud attempts often occur using newly created accounts.
* No historical behavior exists for validation.

Example logic:

```sql
customer_previous_orders = 0
```

---

### Contact Information Rules

#### Shared Email Detection

Detects email addresses associated with multiple customer accounts.

Why it matters:

* Fraudsters frequently reuse email accounts.
* Multiple identities may be created using the same contact information.

Example:

```text
email used by more than one customer
```

---

#### Shared Phone Detection

Detects phone numbers associated with multiple customer accounts after normalization.

Why it matters:

* Phone numbers are generally expected to uniquely identify customers.
* Repeated usage may indicate account farming or identity manipulation.

Example process:

1. Remove punctuation
2. Standardize format
3. Validate structure
4. Compare against customer base

---

#### Email Reputation Analysis

Email addresses were evaluated as part of the overall risk-assessment process.

The objective was not simply to identify the email provider, but to estimate the trustworthiness of the email account being used.

#### Common Email Providers

Examples:

* Gmail
* Outlook
* Hotmail
* Yahoo

For addresses belonging to well-known providers, the domain itself was not considered suspicious.

Instead, the email could be evaluated through an automated reputation or age-assessment process that produced a risk indicator.

Examples of factors that may influence the score:

* Email age
* Reputation signals
* Historical activity patterns
* Deliverability characteristics

The resulting score became one of several inputs to the overall fraud-evaluation process.

#### Uncommon or Suspicious Domains

Addresses associated with uncommon, recently created, or low-reputation domains were treated differently.

Examples:

```text
user@unknown-domain.tld
user@example.ru
user@random-provider.xyz
```

These domains may indicate:

* Disposable email services
* Recently registered domains
* Low-trust email providers
* Increased difficulty in validating customer identity

Because legitimate customers rarely used such domains, these cases typically received a higher risk contribution and were more likely to be reviewed manually.

#### Design Principle

The email domain itself was never considered sufficient evidence of fraud.

Instead, email reputation was combined with other indicators such as:

* First-time buyer
* Shared email
* Shared phone number
* New delivery address
* Purchase velocity

The goal was to improve risk assessment while minimizing unnecessary false positives.

---

### Address Rules

#### New Address Detection

Flags delivery addresses not previously used by the customer.

Why it matters:

* Sudden address changes may indicate account compromise.
* Fraudulent purchases are often shipped to newly introduced locations.

---

#### Shared Address Detection

Detects delivery addresses used by multiple customers.

Why it matters:

* Fraud rings often reuse delivery locations.
* Multiple identities shipping to the same address may warrant review.

Address comparison relied on normalized address values.

---

#### Address Normalization

Before comparison:

* Accents removed
* Special characters removed
* Case standardized
* Extra whitespace removed

Example:

```text
Rua São José, 123
RUA SAO JOSE 123
```

would be treated as the same address.

---

### Purchase Behavior Rules

#### Velocity Analysis

Detects unusual purchase activity within short periods.

Examples:

* Multiple purchases in rapid succession
* Multiple accounts purchasing within a narrow timeframe
* Sudden spikes in activity

Purpose:

* Identify automated or coordinated behavior.

---

#### Historical Correlation

Current transactions were evaluated against historical purchasing behavior.

Examples:

* New contact information
* New delivery location
* Unusual activity patterns

---

## Exclusion Rules

Not every suspicious-looking order should generate an alert.

Several operational scenarios were intentionally excluded.

---

### Store Pickup Orders

Orders marked as store pickup were excluded.

Example:

```text
pickup_indicator = 'YES'
```

Reason:

* These orders involve customer presence at pickup.
* Operational risk differs from standard delivery orders.

---

### Authorized Pickup Locations

The company maintained a list of approved pickup locations and stands.

After address normalization, orders matching those locations were ignored.

Reason:

* Legitimate operational workflow.
* High false-positive potential if evaluated normally.

---

### Assisted Sales

Orders associated with a human sales representative were excluded.

Example:

```text
seller_id != default_self_service_channel
```

Reason:

* Additional validation had already occurred during the sales process.
* Reduced unnecessary fraud-review workload.

---

## Risk Scoring

Rules contributed weighted values to a total risk score.

Illustrative example:

```text
First Purchase         +20
Shared Phone           +30
Shared Email           +30
New Address            +20
Velocity Indicator     +15
---------------------------
Total Score            =115
```

If the score exceeded a predefined threshold, an alert was generated.

Actual production weights and thresholds are not included in this repository.

---

## Design Principles

The rule engine prioritized:

* Explainability
* Reproducibility
* Operational simplicity
* Fast investigation workflows
* Low maintenance overhead

The objective was not to eliminate all fraud automatically, but to surface the highest-risk orders for efficient manual review.
