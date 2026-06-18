
# Spark SQL Patterns Used

## Pattern 1: Deduplication with ROW_NUMBER()

```sql
WITH ranked_orders AS (
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY created_at DESC) as rn
  FROM raw_orders
)
SELECT * FROM ranked_orders
WHERE rn = 1
```

Why: Fast, distributed, scales to billions of rows. Window function on single column.

---

## Pattern 2: Shared Phone Detection via GROUP BY

```sql
WITH phone_stats AS (
  SELECT 
    phone_number,
    COUNT(DISTINCT customer_id) as num_customers,
    MAX(created_at) as last_use_date
  FROM clean_orders
  WHERE phone_number IS NOT NULL
  GROUP BY phone_number
  HAVING COUNT(DISTINCT customer_id) > 1
)
SELECT 
  o.order_id,
  o.customer_id,
  ps.num_customers,
  'SHARED_PHONE' as risk_flag
FROM clean_orders o
INNER JOIN phone_stats ps ON o.phone_number = ps.phone_number
```

Why: Efficient aggregation, clear intent, easy to tune the HAVING clause.

---

## Pattern 3: First-Time Buyer Detection

```sql
WITH customer_first_order AS (
  SELECT 
    customer_id,
    MIN(created_at) as first_order_date
  FROM clean_orders
  GROUP BY customer_id
)
SELECT 
  o.order_id,
  o.customer_id,
  CASE 
    WHEN cfo.first_order_date = o.created_at THEN 'FIRST_TIME'
    ELSE 'RETURNING'
  END as buyer_type
FROM clean_orders o
LEFT JOIN customer_first_order cfo ON o.customer_id = cfo.customer_id
```

Why: Simple temporal join, avoids complex window function logic.
