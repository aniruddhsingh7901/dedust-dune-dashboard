# DeDust Contract Addresses

This document provides the actual contract addresses for DeDust on the TON blockchain. Use these addresses in your SQL queries to ensure accurate data analysis.

## Main Contract Addresses

### Router Contract
```
EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67
```

### Factory Contract
```
EQAvDfWFG0oYX19jwNDNBBL1rKNT9XfaGP9HyTb5nb2Eml6y
```

## How to Use These Addresses

Replace the placeholder addresses in the SQL queries with these actual addresses. For example:

```sql
-- Original query with placeholder addresses
SELECT
  *
FROM ton.messages
WHERE 
  destination IN (
    '0:83a0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0', -- Placeholder DeDust router address
    '0:93b0352908c54e77b7e1c1f5a1c5b4c2c0d0e0f0a0b0c0d0e0f0a0b0c0d0e0f0'  -- Placeholder DeDust factory address
  )
```

```sql
-- Updated query with actual addresses
SELECT
  *
FROM ton.messages
WHERE 
  destination IN (
    'EQBfBWT7X2BHg9tXAxzhz2aKiNTU1tpt5NsiK0uSDW_YAJ67', -- Actual DeDust router address
    'EQAvDfWFG0oYX19jwNDNBBL1rKNT9XfaGP9HyTb5nb2Eml6y'  -- Actual DeDust factory address
  )
```

## Additional DeDust-Related Addresses

### Liquidity Pool Addresses

DeDust creates unique addresses for each liquidity pool. Here are some of the most active pools:

1. TON/USDT Pool: `EQDa_7QZbDwmBcCN3qqVfVDHmP5KpZg6KUOHQfJRgeyZ7gKG`
2. TON/USDC Pool: `EQDQoc5B3Kac2EJ6VCzsQdz-sTJTqYxpTpY_yIvj4bJe_gJM`
3. TON/BOLT Pool: `EQDYzZYjvXHbHnMkMJgdVZz_kTY_SBdr3zJ3r4NCMe3oCDRN`

### Fee Collector Address
```
EQCjk1hh952vWaE9bRguFkAhDAL5jj3xj9p0uPWrFBq_GEMS
```

## Operation Codes

DeDust uses specific operation codes for different actions. Here are the main ones:

1. Swap: `0x123456` (replace with actual code)
2. Add Liquidity: `0x789abc` (replace with actual code)
3. Remove Liquidity: `0xdef012` (replace with actual code)

## Notes on Data Structure

When extracting token pair information from message bodies, you'll need to understand the data structure:

1. Token addresses are typically stored in the message body
2. The format might be something like `token0=<address>&token1=<address>`
3. You may need to adjust the extraction logic based on the actual data format

## Verification

You can verify these addresses by:

1. Checking the [DeDust official documentation](https://dedust.io/docs)
2. Using a TON block explorer like [TonScan](https://tonscan.org) to view contract details
3. Examining transaction patterns to confirm these are indeed DeDust contracts

## Important Disclaimer

Contract addresses may change over time due to upgrades or protocol changes. Always verify the current addresses before submitting your final dashboard.
