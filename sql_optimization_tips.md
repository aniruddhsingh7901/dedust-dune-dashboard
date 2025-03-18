# SQL Optimization Tips for Dune Analytics

This document provides tips and best practices for optimizing your SQL queries on Dune Analytics. Following these guidelines will help you create efficient queries that perform well, even with large datasets.

## General Optimization Tips

### 1. Use Common Table Expressions (CTEs) Effectively

CTEs (WITH clauses) improve readability and can be reused multiple times in a query:

```sql
-- Good practice
WITH swap_events AS (
  SELECT * FROM ton.messages
  WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
  AND op = 'swap'
)

SELECT * FROM swap_events WHERE value > 1000000000;
```

However, be careful with nested CTEs as they can sometimes lead to performance issues.

### 2. Filter Early and Often

Apply filters as early as possible in your query to reduce the amount of data processed:

```sql
-- Less efficient
SELECT
  source,
  SUM(value) as total_value
FROM ton.messages
GROUP BY source
HAVING SUM(value) > 1000000000;

-- More efficient
SELECT
  source,
  SUM(value) as total_value
FROM ton.messages
WHERE value > 0 -- Filter early if possible
GROUP BY source
HAVING SUM(value) > 1000000000;
```

### 3. Use Appropriate Indexes

Dune Analytics has indexes on common columns. Take advantage of them:

- `block_timestamp` is typically indexed
- `source` and `destination` addresses are typically indexed
- `value` might be indexed

Structure your queries to use these indexed columns in WHERE clauses when possible.

### 4. Limit Date Ranges

Always include date ranges in your queries to limit the amount of data scanned:

```sql
-- Good practice
SELECT *
FROM ton.messages
WHERE block_timestamp >= NOW() - INTERVAL '30 days'
AND destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';
```

### 5. Use LIMIT for Development

When developing and testing queries, use LIMIT to restrict the number of rows returned:

```sql
-- During development
SELECT *
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
LIMIT 100;
```

Remove the LIMIT when your query is finalized.

## Dune-Specific Optimizations

### 1. Avoid SELECT *

Specify only the columns you need instead of using SELECT *:

```sql
-- Less efficient
SELECT *
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';

-- More efficient
SELECT
  block_timestamp,
  source,
  value
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';
```

### 2. Use Appropriate Data Types

Be mindful of data types in calculations and comparisons:

```sql
-- Less efficient (implicit conversion)
SELECT *
FROM ton.messages
WHERE block_timestamp >= '2023-01-01';

-- More efficient (explicit date)
SELECT *
FROM ton.messages
WHERE block_timestamp >= DATE '2023-01-01';
```

### 3. Optimize JOIN Operations

Use appropriate JOIN types and join on indexed columns:

```sql
-- Good practice
SELECT
  m.source,
  m.value,
  b.hash
FROM ton.messages m
JOIN ton.blocks b ON m.block = b.number
WHERE m.destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';
```

### 4. Use Subqueries Carefully

Subqueries can sometimes be less efficient than JOINs or CTEs:

```sql
-- Less efficient (correlated subquery)
SELECT
  source,
  value,
  (SELECT hash FROM ton.blocks WHERE number = m.block) as block_hash
FROM ton.messages m
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';

-- More efficient (JOIN)
SELECT
  m.source,
  m.value,
  b.hash as block_hash
FROM ton.messages m
JOIN ton.blocks b ON m.block = b.number
WHERE m.destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';
```

### 5. Cache Intermediate Results

For complex dashboards, consider caching intermediate results in separate queries:

1. Create a query that calculates daily volumes
2. Create another query that references the first query's results
3. This way, the daily volumes are only calculated once

## Performance Considerations for Time-Series Data

### 1. Pre-aggregate Data When Possible

For time-series visualizations, pre-aggregate data to the required granularity:

```sql
-- Good practice for daily chart
SELECT
  DATE_TRUNC('day', block_timestamp) as day,
  SUM(value / 1e9) as daily_volume
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
AND block_timestamp >= NOW() - INTERVAL '90 days'
GROUP BY DATE_TRUNC('day', block_timestamp)
ORDER BY day;
```

### 2. Use Window Functions Efficiently

Window functions are powerful but can be resource-intensive. Use them judiciously:

```sql
-- Efficient use of window function
SELECT
  DATE_TRUNC('day', block_timestamp) as day,
  SUM(value / 1e9) as daily_volume,
  SUM(SUM(value / 1e9)) OVER (ORDER BY DATE_TRUNC('day', block_timestamp)) as cumulative_volume
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
AND block_timestamp >= NOW() - INTERVAL '90 days'
GROUP BY DATE_TRUNC('day', block_timestamp)
ORDER BY day;
```

### 3. Generate Complete Time Series

For visualizations, it's often better to have a complete time series with zeros for missing data:

```sql
-- Generate complete time series
WITH date_series AS (
  SELECT generate_series(
    DATE_TRUNC('day', NOW() - INTERVAL '90 days'),
    DATE_TRUNC('day', NOW()),
    INTERVAL '1 day'
  ) as day
),
daily_volumes AS (
  SELECT
    DATE_TRUNC('day', block_timestamp) as day,
    SUM(value / 1e9) as daily_volume
  FROM ton.messages
  WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
  AND block_timestamp >= NOW() - INTERVAL '90 days'
  GROUP BY DATE_TRUNC('day', block_timestamp)
)

SELECT
  ds.day,
  COALESCE(dv.daily_volume, 0) as daily_volume
FROM date_series ds
LEFT JOIN daily_volumes dv ON ds.day = dv.day
ORDER BY ds.day;
```

## Query Debugging and Testing

### 1. Test with Smaller Date Ranges

Start with a small date range and gradually expand it:

```sql
-- Start with 1 day
SELECT * FROM ton.messages
WHERE block_timestamp >= NOW() - INTERVAL '1 day'
AND destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';

-- Then try 7 days, 30 days, etc.
```

### 2. Validate Results

Always validate your results with alternative calculations:

```sql
-- Two ways to calculate the same metric
SELECT COUNT(*) FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';

SELECT COUNT(1) FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67';
```

### 3. Check for Duplicates

Ensure you're not double-counting data:

```sql
-- Check for potential duplicates
SELECT
  block_timestamp,
  source,
  destination,
  value,
  COUNT(*) as count
FROM ton.messages
WHERE destination = 'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67'
GROUP BY block_timestamp, source, destination, value
HAVING COUNT(*) > 1;
```

## Final Tips

1. **Start Simple**: Begin with simple queries and gradually add complexity
2. **Modularize**: Break complex analyses into multiple queries
3. **Document**: Add comments to explain complex logic
4. **Test Incrementally**: Test each part of your query before combining them
5. **Monitor Performance**: Pay attention to query execution time
6. **Review Query Plans**: If available, review query execution plans to identify bottlenecks
7. **Iterate**: Continuously refine your queries for better performance

By following these optimization tips, you'll create a dashboard that not only looks great but also performs well, providing a smooth experience for users exploring your DeDust analytics.
