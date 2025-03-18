# DeDust Dune Dashboard

This project contains SQL queries and design documentation for a comprehensive Dune Analytics dashboard for DeDust, a decentralized exchange (DEX) on the TON blockchain.

## Project Structure

- `README.md` - This file
- `queries/` - Directory containing SQL queries for all required metrics
  - `overall_metrics.sql` - All-time total metrics
  - `daily_report.sql` - Yesterday's data
  - `monthly_report.sql` - Previous month's data
  - `historical_report.sql` - Trends over time
  - `additional_insights.sql` - Unique insights to make the dashboard stand out
- `dashboard_design.md` - Dashboard layout and visualization recommendations
- `implementation_guide.md` - Step-by-step guide for implementing the dashboard in Dune

## Contest Requirements

### Required Metrics

1. **Overall Metrics (All-Time Totals)**
   - Total Trading Volume
   - Unique Wallets (Users) Who Made Swaps
   - Total Swaps
   - Total Collected Fees
   - Number of Trading Pairs
   - Total Liquidity

2. **Daily Report (Yesterday's Data)**
   - Total Trading Volume + Breakdown by Trading Pairs
   - Collected Fees + Breakdown by Top 10 Pairs
   - Volume Per User + Median Volume Per User
   - Trades Per User
   - Unique Users
   - Average Trading Pairs Per User

3. **Monthly Report (Previous Month's Data)**
   - Total Trading Volume + Pair Breakdown
   - Total Collected Fees
   - Monthly Trading Volume Trends (Yearly Comparison)
   - Total Liquidity
   - Average Trading Pairs Per User

4. **Historical Monthly Report (Trends Over Time)**
   - Trading Volume (Total + Pair Breakdown) Over the Past Year
   - Collected Fees Over Time
   - Unique Users Over Time
   - Average Trading Pairs Per User Over Time

5. **Additional Insights**
   - Trading Volume Breakdown by Weekday
   - Active Traders Over Different Timeframes (7, 30, 60, 90 Days)
   - Top 100 Traders (Monthly)
   - Fee Segments Report

### Submission Deadline

March 24, 2023

## TON Labelling Side Quest

The TON Foundation is offering 5 TON per relevant labeled address. Labels should be submitted to the [TON Labels Repo](https://github.com/ton-blockchain/ton-labels).
