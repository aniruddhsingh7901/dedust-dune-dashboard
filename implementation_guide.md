# DeDust Dune Dashboard Implementation Guide

This guide provides step-by-step instructions to implement a winning Dune Analytics dashboard for DeDust on the TON blockchain. Follow these steps to create your dashboard and submit it for the contest.

## Step 1: Create a New Dashboard on Dune Analytics

1. Go to [Dune Analytics](https://dune.com/)
2. Sign in or create an account if you don't have one
3. Click on "New Dashboard" in the top right corner
4. Name your dashboard "DeDust Analytics Dashboard" and add a description
5. Click "Create Dashboard"

## Step 2: Implement the Required Queries

For each section below, create a new query in your dashboard and copy-paste the provided SQL code. Then set up the recommended visualization.

### Overall Metrics (All-Time Totals)

#### Query 1: Overall Trading Volume and Swaps

```sql
-- Overall Trading Volume and Swaps
WITH dedust_swaps AS (
  SELECT
    block_timestamp,
    value AS amount_in_nanotons,
    value / 1e9 AS amount_in_tons,
    -- Using TON price from price feed if available, or use a constant for now
    (value / 1e9) * 3.5 AS amount_in_usd
  FROM ton.messages
  WHERE 
    -- Filter for DeDust contract addresses
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    -- Filter for swap operations
    AND op = 'swap' -- Adjust based on actual operation code used by DeDust
)

SELECT
  SUM(amount_in_tons) AS total_volume_tons,
  SUM(amount_in_usd) AS total_volume_usd,
  COUNT(*) AS total_swaps
FROM dedust_swaps;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create three separate counters:
   - Total Volume (TON): Format as number with 2 decimal places and "TON" suffix
   - Total Volume (USD): Format as currency with "$" prefix
   - Total Swaps: Format as number with no decimal places

#### Query 2: Unique Users and Trading Pairs

```sql
-- Unique Users and Trading Pairs
WITH dedust_users AS (
  SELECT DISTINCT
    source AS user_address
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
),
dedust_pairs AS (
  SELECT DISTINCT
    -- Extract token pair information from the message body or logs
    -- This is a simplified example and needs adjustment based on actual data structure
    CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    ) AS pair
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND (op = 'swap' OR op = 'addLiquidity' OR op = 'removeLiquidity')
)

SELECT
  (SELECT COUNT(*) FROM dedust_users) AS unique_users,
  (SELECT COUNT(*) FROM dedust_pairs) AS trading_pairs;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create two separate counters:
   - Unique Users: Format as number with no decimal places
   - Trading Pairs: Format as number with no decimal places

#### Query 3: Total Collected Fees and Liquidity

```sql
-- Total Collected Fees and Liquidity
WITH dedust_swaps AS (
  SELECT
    block_timestamp,
    value AS amount_in_nanotons,
    value / 1e9 AS amount_in_tons,
    -- Assuming a fee rate of 0.3% (adjust based on actual DeDust fee structure)
    (value / 1e9) * 0.003 AS fee_in_tons,
    (value / 1e9) * 0.003 * 3.5 AS fee_in_usd -- Using $3.5 as TON price
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
),
dedust_liquidity AS (
  SELECT
    block_timestamp,
    -- Extract liquidity information from the message body or logs
    -- This is a simplified example and needs adjustment based on actual data structure
    CASE 
      WHEN op = 'addLiquidity' THEN value / 1e9
      WHEN op = 'removeLiquidity' THEN -value / 1e9
      ELSE 0
    END AS liquidity_change_tons,
    CASE 
      WHEN op = 'addLiquidity' THEN (value / 1e9) * 3.5
      WHEN op = 'removeLiquidity' THEN -(value / 1e9) * 3.5
      ELSE 0
    END AS liquidity_change_usd
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND (op = 'addLiquidity' OR op = 'removeLiquidity')
)

SELECT
  (SELECT SUM(fee_in_tons) FROM dedust_swaps) AS total_fees_tons,
  (SELECT SUM(fee_in_usd) FROM dedust_swaps) AS total_fees_usd,
  (SELECT SUM(liquidity_change_tons) FROM dedust_liquidity) AS total_liquidity_tons,
  (SELECT SUM(liquidity_change_usd) FROM dedust_liquidity) AS total_liquidity_usd;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create four separate counters:
   - Total Fees (TON): Format as number with 2 decimal places and "TON" suffix
   - Total Fees (USD): Format as currency with "$" prefix
   - Total Liquidity (TON): Format as number with 2 decimal places and "TON" suffix
   - Total Liquidity (USD): Format as currency with "$" prefix

### Daily Report (Yesterday's Data)

#### Query 4: Yesterday's Trading Volume by Pair

```sql
-- Yesterday's Trading Volume by Pair
WITH yesterday AS (
  SELECT DATE_TRUNC('day', NOW() - INTERVAL '1 day') AS yesterday_date
),
dedust_swaps AS (
  SELECT
    block_timestamp,
    -- Extract token pair information
    CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    ) AS pair,
    value AS amount_in_nanotons,
    value / 1e9 AS amount_in_tons,
    (value / 1e9) * 3.5 AS amount_in_usd -- Using $3.5 as TON price
  FROM ton.messages
  CROSS JOIN yesterday
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND DATE_TRUNC('day', block_timestamp) = yesterday_date
)

-- Total Volume Yesterday
SELECT
  SUM(amount_in_tons) AS total_volume_tons,
  SUM(amount_in_usd) AS total_volume_usd,
  COUNT(*) AS total_swaps
FROM dedust_swaps;

-- Breakdown by Trading Pairs
SELECT
  pair,
  SUM(amount_in_tons) AS volume_tons,
  SUM(amount_in_usd) AS volume_usd,
  COUNT(*) AS swaps,
  -- Calculate percentage of total volume
  SUM(amount_in_usd) / (SELECT SUM(amount_in_usd) FROM dedust_swaps) * 100 AS volume_percentage
FROM dedust_swaps
GROUP BY pair
ORDER BY volume_usd DESC;
```

**Visualization Setup:**
1. For the first result set:
   - Create a "Counter" visualization showing yesterday's total volume
2. For the second result set:
   - Create a "Bar Chart" visualization
   - X-axis: pair
   - Y-axis: volume_usd
   - Add a secondary Y-axis for volume_percentage
   - Sort by volume_usd in descending order
   - Add a title: "Yesterday's Trading Volume by Pair"

#### Query 5: Yesterday's Fees by Top 10 Pairs

```sql
-- Yesterday's Fees by Top 10 Pairs
WITH yesterday AS (
  SELECT DATE_TRUNC('day', NOW() - INTERVAL '1 day') AS yesterday_date
),
dedust_swaps AS (
  SELECT
    block_timestamp,
    -- Extract token pair information
    CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    ) AS pair,
    value AS amount_in_nanotons,
    -- Assuming a fee rate of 0.3% (adjust based on actual DeDust fee structure)
    (value / 1e9) * 0.003 AS fee_in_tons,
    (value / 1e9) * 0.003 * 3.5 AS fee_in_usd -- Using $3.5 as TON price
  FROM ton.messages
  CROSS JOIN yesterday
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND DATE_TRUNC('day', block_timestamp) = yesterday_date
)

-- Total Fees Yesterday
SELECT
  SUM(fee_in_tons) AS total_fees_tons,
  SUM(fee_in_usd) AS total_fees_usd
FROM dedust_swaps;

-- Breakdown by Top 10 Pairs
SELECT
  pair,
  SUM(fee_in_tons) AS fees_tons,
  SUM(fee_in_usd) AS fees_usd,
  -- Calculate percentage of total fees
  SUM(fee_in_usd) / (SELECT SUM(fee_in_usd) FROM dedust_swaps) * 100 AS fees_percentage
FROM dedust_swaps
GROUP BY pair
ORDER BY fees_usd DESC
LIMIT 10;
```

**Visualization Setup:**
1. For the first result set:
   - Create a "Counter" visualization showing yesterday's total fees
2. For the second result set:
   - Create a "Pie Chart" visualization
   - Values: fees_usd
   - Labels: pair
   - Add a title: "Yesterday's Fees by Top 10 Pairs"

#### Query 6: User Metrics for Yesterday

```sql
-- User Metrics for Yesterday
WITH yesterday AS (
  SELECT DATE_TRUNC('day', NOW() - INTERVAL '1 day') AS yesterday_date
),
user_volumes AS (
  SELECT
    source AS user_address,
    SUM(value / 1e9) AS volume_tons,
    SUM((value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS num_trades
  FROM ton.messages
  CROSS JOIN yesterday
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND DATE_TRUNC('day', block_timestamp) = yesterday_date
  GROUP BY user_address
),
user_pairs AS (
  SELECT
    source AS user_address,
    COUNT(DISTINCT CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    )) AS num_pairs
  FROM ton.messages
  CROSS JOIN yesterday
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND DATE_TRUNC('day', block_timestamp) = yesterday_date
  GROUP BY user_address
)

SELECT
  COUNT(DISTINCT user_address) AS unique_users,
  AVG(volume_usd) AS avg_volume_per_user_usd,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY volume_usd) AS median_volume_per_user_usd,
  AVG(num_trades) AS avg_trades_per_user,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY num_trades) AS median_trades_per_user,
  (SELECT AVG(num_pairs) FROM user_pairs) AS avg_pairs_per_user
FROM user_volumes;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create multiple counters for each metric:
   - Unique Users
   - Average Volume per User (USD)
   - Median Volume per User (USD)
   - Average Trades per User
   - Median Trades per User
   - Average Trading Pairs per User

### Monthly Report (Previous Month's Data)

#### Query 7: Previous Month's Trading Volume by Pair

```sql
-- Previous Month's Trading Volume by Pair
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
dedust_swaps AS (
  SELECT
    block_timestamp,
    -- Extract token pair information
    CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    ) AS pair,
    value AS amount_in_nanotons,
    value / 1e9 AS amount_in_tons,
    (value / 1e9) * 3.5 AS amount_in_usd -- Using $3.5 as TON price
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
)

-- Total Volume Last Month
SELECT
  SUM(amount_in_tons) AS total_volume_tons,
  SUM(amount_in_usd) AS total_volume_usd,
  COUNT(*) AS total_swaps
FROM dedust_swaps;

-- Breakdown by Trading Pairs
SELECT
  pair,
  SUM(amount_in_tons) AS volume_tons,
  SUM(amount_in_usd) AS volume_usd,
  COUNT(*) AS swaps,
  -- Calculate percentage of total volume
  SUM(amount_in_usd) / (SELECT SUM(amount_in_usd) FROM dedust_swaps) * 100 AS volume_percentage
FROM dedust_swaps
GROUP BY pair
ORDER BY volume_usd DESC;
```

**Visualization Setup:**
1. For the first result set:
   - Create a "Counter" visualization showing last month's total volume
2. For the second result set:
   - Create a "Bar Chart" visualization for volume by pair
   - Create a "Pie Chart" visualization for percentage breakdown

#### Query 8: Previous Month's Fees and Liquidity

```sql
-- Previous Month's Fees and Liquidity
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
dedust_swaps AS (
  SELECT
    block_timestamp,
    value AS amount_in_nanotons,
    -- Assuming a fee rate of 0.3% (adjust based on actual DeDust fee structure)
    (value / 1e9) * 0.003 AS fee_in_tons,
    (value / 1e9) * 0.003 * 3.5 AS fee_in_usd -- Using $3.5 as TON price
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
),
dedust_liquidity AS (
  SELECT
    block_timestamp,
    -- Extract liquidity information
    CASE 
      WHEN op = 'addLiquidity' THEN value / 1e9
      WHEN op = 'removeLiquidity' THEN -value / 1e9
      ELSE 0
    END AS liquidity_change_tons,
    CASE 
      WHEN op = 'addLiquidity' THEN (value / 1e9) * 3.5
      WHEN op = 'removeLiquidity' THEN -(value / 1e9) * 3.5
      ELSE 0
    END AS liquidity_change_usd
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND (op = 'addLiquidity' OR op = 'removeLiquidity')
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
)

SELECT
  (SELECT SUM(fee_in_tons) FROM dedust_swaps) AS total_fees_tons,
  (SELECT SUM(fee_in_usd) FROM dedust_swaps) AS total_fees_usd,
  (SELECT SUM(liquidity_change_tons) FROM dedust_liquidity) AS net_liquidity_change_tons,
  (SELECT SUM(liquidity_change_usd) FROM dedust_liquidity) AS net_liquidity_change_usd;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create counters for each metric:
   - Total Fees (TON)
   - Total Fees (USD)
   - Net Liquidity Change (TON)
   - Net Liquidity Change (USD)

#### Query 9: Monthly Trading Volume Trends (Yearly Comparison)

```sql
-- Monthly Trading Volume Trends (Yearly Comparison)
WITH months AS (
  SELECT generate_series(
    DATE_TRUNC('month', NOW() - INTERVAL '12 months'),
    DATE_TRUNC('month', NOW() - INTERVAL '1 month'),
    INTERVAL '1 month'
  ) AS month_start
),
month_ranges AS (
  SELECT
    month_start,
    month_start + INTERVAL '1 month' - INTERVAL '1 day' AS month_end,
    TO_CHAR(month_start, 'Mon YYYY') AS month_label
  FROM months
),
monthly_volumes AS (
  SELECT
    mr.month_label,
    SUM(m.value / 1e9) AS volume_tons,
    SUM((m.value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS num_swaps
  FROM ton.messages m
  JOIN month_ranges mr
    ON m.block_timestamp >= mr.month_start
    AND m.block_timestamp <= mr.month_end
  WHERE 
    m.destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND m.op = 'swap' -- Adjust based on actual operation code
  GROUP BY mr.month_label, mr.month_start
  ORDER BY mr.month_start
)

SELECT
  month_label,
  volume_tons,
  volume_usd,
  num_swaps
FROM monthly_volumes;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Line Chart" as the visualization type
3. X-axis: month_label
4. Y-axis: volume_usd
5. Add a secondary Y-axis for num_swaps
6. Add a title: "Monthly Trading Volume Trends"

#### Query 10: Average Trading Pairs Per User (Monthly)

```sql
-- Average Trading Pairs Per User (Monthly)
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
user_pairs AS (
  SELECT
    source AS user_address,
    COUNT(DISTINCT CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    )) AS num_pairs
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
  GROUP BY user_address
)

SELECT
  AVG(num_pairs) AS avg_pairs_per_user,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY num_pairs) AS median_pairs_per_user,
  MAX(num_pairs) AS max_pairs_per_user
FROM user_pairs;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create counters for each metric:
   - Average Pairs per User
   - Median Pairs per User
   - Maximum Pairs per User

### Historical Monthly Report (Trends Over Time)

#### Query 11: Historical Trading Volume Trends

```sql
-- Historical Trading Volume Trends
WITH months AS (
  SELECT generate_series(
    DATE_TRUNC('month', NOW() - INTERVAL '12 months'),
    DATE_TRUNC('month', NOW()),
    INTERVAL '1 month'
  ) AS month_start
),
month_ranges AS (
  SELECT
    month_start,
    month_start + INTERVAL '1 month' - INTERVAL '1 day' AS month_end,
    TO_CHAR(month_start, 'Mon YYYY') AS month_label
  FROM months
),
monthly_data AS (
  SELECT
    mr.month_label,
    mr.month_start,
    SUM(m.value / 1e9) AS volume_tons,
    SUM((m.value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS num_swaps,
    COUNT(DISTINCT m.source) AS unique_users,
    SUM((m.value / 1e9) * 0.003) AS fees_tons,
    SUM((m.value / 1e9) * 0.003 * 3.5) AS fees_usd
  FROM ton.messages m
  RIGHT JOIN month_ranges mr
    ON m.block_timestamp >= mr.month_start
    AND m.block_timestamp <= mr.month_end
    AND m.destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND m.op = 'swap' -- Adjust based on actual operation code
  GROUP BY mr.month_label, mr.month_start
  ORDER BY mr.month_start
)

SELECT
  month_label,
  volume_tons,
  volume_usd,
  num_swaps,
  unique_users,
  fees_tons,
  fees_usd
FROM monthly_data;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Line Chart" as the visualization type
3. X-axis: month_label
4. Y-axis: volume_usd
5. Add a title: "Historical Trading Volume Trends"

#### Query 12: Historical Collected Fees Trends

```sql
-- Historical Collected Fees Trends
-- This query reuses the monthly_data CTE from Query 11
WITH months AS (
  SELECT generate_series(
    DATE_TRUNC('month', NOW() - INTERVAL '12 months'),
    DATE_TRUNC('month', NOW()),
    INTERVAL '1 month'
  ) AS month_start
),
month_ranges AS (
  SELECT
    month_start,
    month_start + INTERVAL '1 month' - INTERVAL '1 day' AS month_end,
    TO_CHAR(month_start, 'Mon YYYY') AS month_label
  FROM months
),
monthly_data AS (
  SELECT
    mr.month_label,
    mr.month_start,
    SUM((m.value / 1e9) * 0.003) AS fees_tons,
    SUM((m.value / 1e9) * 0.003 * 3.5) AS fees_usd
  FROM ton.messages m
  RIGHT JOIN month_ranges mr
    ON m.block_timestamp >= mr.month_start
    AND m.block_timestamp <= mr.month_end
    AND m.destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND m.op = 'swap' -- Adjust based on actual operation code
  GROUP BY mr.month_label, mr.month_start
  ORDER BY mr.month_start
)

SELECT
  month_label,
  fees_tons,
  fees_usd
FROM monthly_data;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Line Chart" as the visualization type
3. X-axis: month_label
4. Y-axis: fees_usd
5. Add a title: "Historical Collected Fees Trends"

#### Query 13: Historical Unique Users Trends

```sql
-- Historical Unique Users Trends
WITH months AS (
  SELECT generate_series(
    DATE_TRUNC('month', NOW() - INTERVAL '12 months'),
    DATE_TRUNC('month', NOW()),
    INTERVAL '1 month'
  ) AS month_start
),
month_ranges AS (
  SELECT
    month_start,
    month_start + INTERVAL '1 month' - INTERVAL '1 day' AS month_end,
    TO_CHAR(month_start, 'Mon YYYY') AS month_label
  FROM months
),
monthly_users AS (
  SELECT
    mr.month_label,
    mr.month_start,
    COUNT(DISTINCT m.source) AS unique_users
  FROM ton.messages m
  RIGHT JOIN month_ranges mr
    ON m.block_timestamp >= mr.month_start
    AND m.block_timestamp <= mr.month_end
    AND m.destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND m.op = 'swap' -- Adjust based on actual operation code
  GROUP BY mr.month_label, mr.month_start
  ORDER BY mr.month_start
)

SELECT
  month_label,
  unique_users
FROM monthly_users;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Line Chart" as the visualization type
3. X-axis: month_label
4. Y-axis: unique_users
5. Add a title: "Historical Unique Users Trends"

#### Query 14: Historical Average Trading Pairs Per User

```sql
-- Historical Average Trading Pairs Per User
WITH months AS (
  SELECT generate_series(
    DATE_TRUNC('month', NOW() - INTERVAL '12 months'),
    DATE_TRUNC('month', NOW()),
    INTERVAL '1 month'
  ) AS month_start
),
month_ranges AS (
  SELECT
    month_start,
    month_start + INTERVAL '1 month' - INTERVAL '1 day' AS month_end,
    TO_CHAR(month_start, 'Mon YYYY') AS month_label
  FROM months
),
monthly_user_pairs AS (
  SELECT
    mr.month_label,
    mr.month_start,
    m.source AS user_address,
    COUNT(DISTINCT CONCAT(
      SUBSTR(m.body, POSITION('token0=' IN m.body) + 7, 42), 
      '-', 
      SUBSTR(m.body, POSITION('token1=' IN m.body) + 7, 42)
    )) AS num_pairs
  FROM ton.messages m
  JOIN month_ranges mr
    ON m.block_timestamp >= mr.month_start
    AND m.block_timestamp <= mr.month_end
  WHERE 
    m.destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND m.op = 'swap' -- Adjust based on actual operation code
  GROUP BY mr.month_label, mr.month_start, m.source
),
monthly_avg_pairs AS (
  SELECT
    month_label,
    month_start,
    AVG(num_pairs) AS avg_pairs_per_user
  FROM monthly_user_pairs
  GROUP BY month_label, month_start
  ORDER BY month_start
)

SELECT
  month_label,
  avg_pairs_per_user
FROM monthly_avg_pairs;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Line Chart" as the visualization type
3. X-axis: month_label
4. Y-axis: avg_pairs_per_user
5. Add a title: "Historical Average Trading Pairs Per User"

### Additional Insights

#### Query 15: Trading Volume Breakdown by Weekday

```sql
-- Trading Volume Breakdown by Weekday
WITH daily_volumes AS (
  SELECT
    DATE_TRUNC('day', block_timestamp) AS trade_date,
    TO_CHAR(block_timestamp, 'Day') AS weekday,
    EXTRACT(DOW FROM block_timestamp) AS day_number, -- 0 = Sunday, 1 = Monday, etc.
    SUM(value / 1e9) AS volume_tons,
    SUM((value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS num_swaps
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    -- Last 90 days for more relevant data
    AND block_timestamp >= NOW() - INTERVAL '90 days'
  GROUP BY trade_date, weekday, day_number
),
weekday_volumes AS (
  SELECT
    weekday,
    day_number,
    AVG(volume_tons) AS avg_volume_tons,
    AVG(volume_usd) AS avg_volume_usd,
    AVG(num_swaps) AS avg_num_swaps,
    SUM(volume_tons) AS total_volume_tons,
    SUM(volume_usd) AS total_volume_usd,
    SUM(num_swaps) AS total_num_swaps
  FROM daily_volumes
  GROUP BY weekday, day_number
  ORDER BY day_number
)

SELECT
  weekday,
  avg_volume_tons,
  avg_volume_usd,
  avg_num_swaps,
  total_volume_tons,
  total_volume_usd,
  total_num_swaps
FROM weekday_volumes;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Bar Chart" as the visualization type
3. X-axis: weekday
4. Y-axis: avg_volume_usd
5. Add a secondary Y-axis for avg_num_swaps
6. Sort by day_number to ensure correct weekday order
7. Add a title: "Average Trading Volume by Weekday"

#### Query 16: Active Traders Over Different Timeframes

```sql
-- Active Traders Over Different Timeframes
WITH timeframes AS (
  SELECT 
    NOW() - INTERVAL '7 days' AS days_7,
    NOW() - INTERVAL '30 days' AS days_30,
    NOW() - INTERVAL '60 days' AS days_60,
    NOW() - INTERVAL '90 days' AS days_90
),
active_traders AS (
  SELECT
    COUNT(DISTINCT CASE WHEN block_timestamp >= (SELECT days_7 FROM timeframes) THEN source END) AS active_7_days,
    COUNT(DISTINCT CASE WHEN block_timestamp >= (SELECT days_30 FROM timeframes) THEN source END) AS active_30_days,
    COUNT(DISTINCT CASE WHEN block_timestamp >= (SELECT days_60 FROM timeframes) THEN source END) AS active_60_days,
    COUNT(DISTINCT CASE WHEN block_timestamp >= (SELECT days_90 FROM timeframes) THEN source END) AS active_90_days
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= (SELECT days_90 FROM timeframes)
)

SELECT
  active_7_days,
  active_30_days,
  active_60_days,
  active_90_days
FROM active_traders;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Counter" as the visualization type
3. Create four separate counters:
   - Active Traders (7 Days)
   - Active Traders (30 Days)
   - Active Traders (60 Days)
   - Active Traders (90 Days)

#### Query 17: Top 100 Traders (Monthly)

```sql
-- Top 100 Traders (Monthly)
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
trader_stats AS (
  SELECT
    source AS user_address,
    SUM(value / 1e9) AS volume_tons,
    SUM((value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS total_trades,
    COUNT(DISTINCT CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    )) AS trading_pairs_used,
    SUM((value / 1e9) * 0.003) AS fees_paid_tons,
    SUM((value / 1e9) * 0.003 * 3.5) AS fees_paid_usd
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
  GROUP BY user_address
)

SELECT
  user_address,
  volume_tons,
  volume_usd,
  total_trades,
  trading_pairs_used,
  fees_paid_tons,
  fees_paid_usd,
  -- Calculate rank based on volume
  ROW_NUMBER() OVER (ORDER BY volume_usd DESC) AS trader_rank
FROM trader_stats
ORDER BY volume_usd DESC
LIMIT 100;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Table" as the visualization type
3. Configure columns:
   - user_address: Format as text, rename to "Wallet Address"
   - volume_usd: Format as currency, rename to "Volume (USD)"
   - total_trades: Format as number, rename to "Total Trades"
   - trading_pairs_used: Format as number, rename to "Trading Pairs Used"
   - fees_paid_usd: Format as currency, rename to "Fees Paid (USD)"
   - trader_rank: Format as number, rename to "Rank"
4. Add a title: "Top 100 Traders (Last Month)"

#### Query 18: Fee Segments Report

```sql
-- Fee Segments Report
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
user_fees AS (
  SELECT
    source AS user_address,
    SUM((value / 1e9) * 0.003 * 3.5) AS fees_paid_usd
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
  GROUP BY user_address
),
fee_segments AS (
  SELECT
    CASE
      WHEN fees_paid_usd BETWEEN 0.01 AND 10.00 THEN '$0.01-$10'
      WHEN fees_paid_usd BETWEEN 10.01 AND 50.00 THEN '$10.01-$50'
      WHEN fees_paid_usd BETWEEN 50.01 AND 100.00 THEN '$50.01-$100'
      WHEN fees_paid_usd BETWEEN 100.01 AND 250.00 THEN '$100.01-$250'
      WHEN fees_paid_usd > 250.00 THEN '$250.01+'
      ELSE 'Unknown'
    END AS fee_segment,
    COUNT(*) AS user_count,
    SUM(fees_paid_usd) AS total_fees_usd
  FROM user_fees
  GROUP BY fee_segment
),
segment_order AS (
  SELECT
    fee_segment,
    user_count,
    total_fees_usd,
    CASE
      WHEN fee_segment = '$0.01-$10' THEN 1
      WHEN fee_segment = '$10.01-$50' THEN 2
      WHEN fee_segment = '$50.01-$100' THEN 3
      WHEN fee_segment = '$100.01-$250' THEN 4
      WHEN fee_segment = '$250.01+' THEN 5
      ELSE 6
    END AS segment_order
  FROM fee_segments
)

SELECT
  fee_segment,
  user_count,
  total_fees_usd,
  (user_count * 100.0 / (SELECT SUM(user_count) FROM fee_segments)) AS user_percentage,
  (total_fees_usd * 100.0 / (SELECT SUM(total_fees_usd) FROM fee_segments)) AS fees_percentage
FROM segment_order
ORDER BY segment_order;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Pie Chart" as the visualization type for user distribution
   - Values: user_count
   - Labels: fee_segment
   - Add a title: "User Distribution by Fee Segment"
3. Create a second "Pie Chart" visualization for fee distribution
   - Values: total_fees_usd
   - Labels: fee_segment
   - Add a title: "Fee Distribution by Segment"
4. Create a "Table" visualization showing all columns

#### Query 19: Trading Activity Heatmap (Hour of Day vs Day of Week)

```sql
-- Trading Activity Heatmap (Hour of Day vs Day of Week)
WITH trading_activity AS (
  SELECT
    EXTRACT(DOW FROM block_timestamp) AS day_of_week,
    EXTRACT(HOUR FROM block_timestamp) AS hour_of_day,
    COUNT(*) AS num_swaps,
    SUM(value / 1e9) AS volume_tons,
    SUM((value / 1e9) * 3.5) AS volume_usd
  FROM ton.messages
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    -- Last 90 days for more relevant data
    AND block_timestamp >= NOW() - INTERVAL '90 days'
  GROUP BY day_of_week, hour_of_day
)

SELECT
  day_of_week,
  hour_of_day,
  num_swaps,
  volume_tons,
  volume_usd,
  -- Add day name for better readability
  CASE
    WHEN day_of_week = 0 THEN 'Sunday'
    WHEN day_of_week = 1 THEN 'Monday'
    WHEN day_of_week = 2 THEN 'Tuesday'
    WHEN day_of_week = 3 THEN 'Wednesday'
    WHEN day_of_week = 4 THEN 'Thursday'
    WHEN day_of_week = 5 THEN 'Friday'
    WHEN day_of_week = 6 THEN 'Saturday'
  END AS day_name
FROM trading_activity
ORDER BY day_of_week, hour_of_day;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Heatmap" as the visualization type
3. X-axis: hour_of_day
4. Y-axis: day_name
5. Values: num_swaps (for number of trades) or volume_usd (for trading volume)
6. Color scale: Use a gradient from light to dark colors
7. Add a title: "Trading Activity Heatmap"

#### Query 20: Comparison Between High & Low-Volume Traders

```sql
-- Comparison Between High & Low-Volume Traders
WITH prev_month AS (
  SELECT 
    DATE_TRUNC('month', NOW() - INTERVAL '1 month') AS month_start,
    DATE_TRUNC('month', NOW()) - INTERVAL '1 day' AS month_end
),
trader_volumes AS (
  SELECT
    source AS user_address,
    SUM((value / 1e9) * 3.5) AS volume_usd,
    COUNT(*) AS total_trades,
    COUNT(DISTINCT CONCAT(
      SUBSTR(body, POSITION('token0=' IN body) + 7, 42), 
      '-', 
      SUBSTR(body, POSITION('token1=' IN body) + 7, 42)
    )) AS trading_pairs_used,
    SUM((value / 1e9) * 0.003 * 3.5) AS fees_paid_usd,
    AVG((value / 1e9) * 3.5) AS avg_trade_size_usd
  FROM ton.messages
  CROSS JOIN prev_month
  WHERE 
    destination IN (
      '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Replace with actual DeDust router address
      '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Replace with actual DeDust factory address
    )
    AND op = 'swap' -- Adjust based on actual operation code
    AND block_timestamp >= month_start
    AND block_timestamp <= month_end
  GROUP BY user_address
),
trader_segments AS (
  SELECT
    user_address,
    volume_usd,
    total_trades,
    trading_pairs_used,
    fees_paid_usd,
    avg_trade_size_usd,
    NTILE(10) OVER (ORDER BY volume_usd) AS volume_decile
  FROM trader_volumes
),
segment_stats AS (
  SELECT
    CASE
      WHEN volume_decile IN (1, 2, 3) THEN 'Low-Volume Traders (Bottom 30%)'
      WHEN volume_decile IN (8, 9, 10) THEN 'High-Volume Traders (Top 30%)'
      ELSE 'Medium-Volume Traders (Middle 40%)'
    END AS trader_segment,
    COUNT(*) AS trader_count,
    AVG(volume_usd) AS avg_volume_usd,
    AVG(total_trades) AS avg_trades,
    AVG(trading_pairs_used) AS avg_pairs_used,
    AVG(fees_paid_usd) AS avg_fees_paid_usd,
    AVG(avg_trade_size_usd) AS avg_trade_size_usd,
    SUM(volume_usd) AS total_volume_usd,
    SUM(fees_paid_usd) AS total_fees_paid_usd
  FROM trader_segments
  GROUP BY trader_segment
)

SELECT
  trader_segment,
  trader_count,
  avg_volume_usd,
  avg_trades,
  avg_pairs_used,
  avg_fees_paid_usd,
  avg_trade_size_usd,
  total_volume_usd,
  total_fees_paid_usd,
  (total_volume_usd * 100.0 / (SELECT SUM(total_volume_usd) FROM segment_stats)) AS volume_percentage,
  (total_fees_paid_usd * 100.0 / (SELECT SUM(total_fees_paid_usd) FROM segment_stats)) AS fees_percentage
FROM segment_stats
ORDER BY avg_volume_usd;
```

**Visualization Setup:**
1. Click "New Visualization"
2. Select "Bar Chart" as the visualization type for comparing metrics
   - X-axis: trader_segment
   - Y-axis: avg_volume_usd, avg_trades, avg_pairs_used, etc. (create multiple charts)
3. Create a "Pie Chart" visualization for volume distribution
   - Values: total_volume_usd
   - Labels: trader_segment
4. Create a "Table" visualization showing all columns

## Step 3: Organize Your Dashboard Layout

After creating all the queries and visualizations, organize them into a logical layout:

1. Click "Edit" in the top right corner of your dashboard
2. Drag and drop visualizations to arrange them
3. Use the following sections to organize your dashboard:

### Dashboard Sections

1. **Overview Section**
   - Place the Overall Metrics (All-Time Totals) at the top
   - Include counters for Total Volume, Unique Users, Total Swaps, etc.

2. **Daily Report Section**
   - Include Yesterday's Trading Volume, Fees, and User Metrics
   - Add bar charts and pie charts for pair breakdowns

3. **Monthly Report Section**
   - Include Previous Month's Trading Volume, Fees, and Liquidity
   - Add visualizations for monthly trends

4. **Historical Trends Section**
   - Include line charts for Trading Volume, Fees, and Users over time
   - Add visualizations for historical metrics

5. **Trader Analysis Section**
   - Include Top 100 Traders table
   - Add Fee Segments Report
   - Include comparison between high and low-volume traders

6. **Activity Patterns Section**
   - Include Trading Volume by Weekday
   - Add Trading Activity Heatmap
   - Include Active Traders over different timeframes

## Step 4: Add Text Elements and Descriptions

1. Click "Add Text" to add text elements between sections
2. For each section, add a brief description explaining the metrics
3. Include methodology notes where appropriate

Example text for the Overview Section:
```
# DeDust Analytics Dashboard

This dashboard provides comprehensive analytics for DeDust, a decentralized exchange (DEX) on the TON blockchain. The metrics below show all-time totals for key performance indicators.
```

## Step 5: Finalize and Submit Your Dashboard

1. Click "Save" to save your dashboard
2. Test all visualizations to ensure they load correctly
3. Share your dashboard by clicking "Share" in the top right corner
4. Copy the dashboard link
5. Submit your dashboard link using the form: https://forms.gle/qvvSexJJayAAXEB37

## TON Labelling Side Quest

To participate in the TON Labelling side quest:

1. Visit the [TON Labelling Dashboard](https://dune.com/queries/1102089)
2. Identify notable addresses with significant on-chain activity
3. Research these addresses to determine if they belong to TON-based businesses or apps
4. Submit labels by creating a pull request to the [TON Labels Repo](https://github.com/ton-blockchain/ton-labels)
5. Include:
   - Direct confirmation (screenshot from mini app showing the address, or mention in project's social media)
   - OR 2 indirect confirmations proving the project owns the address
   - Your TON wallet address (for the reward)
   - Your Telegram or email for contact

## Additional Tips for Winning

1. **Ensure Data Accuracy**: Double-check all queries and calculations
2. **Create Clear Visualizations**: Use appropriate chart types and colors
3. **Add Insights**: Include brief analysis notes explaining key findings
4. **Optimize Performance**: Ensure queries run efficiently
5. **Test Thoroughly**: Make sure all visualizations load correctly
6. **Submit Early**: Don't wait until the last minute to submit your dashboard
