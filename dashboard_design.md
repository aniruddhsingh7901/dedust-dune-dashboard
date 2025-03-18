# DeDust Dune Dashboard Design

This document provides a visual layout and design recommendations for the DeDust Dune dashboard. Follow this design to create a well-organized, visually appealing dashboard that effectively communicates insights about DeDust's trading data.

## Dashboard Layout

The dashboard should be organized into logical sections, with related metrics grouped together. Here's a recommended layout:

```
+-----------------------------------------------------------------------+
|                                                                       |
|                         DASHBOARD HEADER                              |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|                       OVERALL METRICS (COUNTERS)                      |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|      DAILY REPORT         |           MONTHLY REPORT                  |
|                           |                                           |
+-----------------------------------------------------------------------+
|                                                                       |
|                     HISTORICAL TRENDS SECTION                         |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|                      TRADER ANALYSIS SECTION                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|                    ACTIVITY PATTERNS SECTION                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|                     ADDITIONAL INSIGHTS SECTION                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

## Section Details

### Dashboard Header

Include a title, description, and any relevant links or information about the dashboard.

```
# DeDust Analytics Dashboard

This dashboard provides comprehensive analytics for DeDust, a decentralized exchange (DEX) on the TON blockchain. The metrics are organized into sections covering overall performance, daily and monthly reports, historical trends, trader analysis, and activity patterns.

Data last updated: [Current Date]
```

### Overall Metrics Section

Display key performance indicators as counter visualizations in a single row.

```
+-----------------------------------------------------------------------+
|                                                                       |
|  Total Volume   |  Unique Users  |  Total Swaps  |  Collected Fees    |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|  Trading Pairs  |  Total Liquidity  |  Other Key Metrics              |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Daily Report Section

Focus on yesterday's data with a mix of counters, bar charts, and pie charts.

```
+-----------------------------------------------------------------------+
|                                                                       |
|                  DAILY REPORT (YESTERDAY'S DATA)                      |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Yesterday's Volume       |  Yesterday's Fees by Top 10 Pairs         |
|  (Counter)                |  (Pie Chart)                              |
|                           |                                           |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Volume by Trading Pair   |  User Metrics                             |
|  (Bar Chart)              |  (Multiple Counters)                      |
|                           |                                           |
+-----------------------------------------------------------------------+
```

### Monthly Report Section

Focus on the previous month's data with similar visualizations to the daily report.

```
+-----------------------------------------------------------------------+
|                                                                       |
|                 MONTHLY REPORT (PREVIOUS MONTH)                       |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Monthly Volume           |  Monthly Fees and Liquidity               |
|  (Counter)                |  (Multiple Counters)                      |
|                           |                                           |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Volume by Trading Pair   |  Trading Volume Trends                    |
|  (Bar Chart)              |  (Line Chart)                             |
|                           |                                           |
+-----------------------------------------------------------------------+
```

### Historical Trends Section

Use line charts to show trends over time for key metrics.

```
+-----------------------------------------------------------------------+
|                                                                       |
|                     HISTORICAL TRENDS SECTION                         |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|  Trading Volume Over Time (Line Chart)                                |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Collected Fees Over Time |  Unique Users Over Time                   |
|  (Line Chart)             |  (Line Chart)                             |
|                           |                                           |
+-----------------------------------------------------------------------+
|                                                                       |
|  Average Trading Pairs Per User Over Time (Line Chart)                |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Trader Analysis Section

Focus on trader behavior and segmentation.

```
+-----------------------------------------------------------------------+
|                                                                       |
|                      TRADER ANALYSIS SECTION                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|  Top 100 Traders (Table)                                              |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Fee Segments - Users     |  Fee Segments - Fees                      |
|  (Pie Chart)              |  (Pie Chart)                              |
|                           |                                           |
+-----------------------------------------------------------------------+
|                                                                       |
|  High vs Low-Volume Trader Comparison (Bar Charts)                    |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Activity Patterns Section

Visualize trading patterns by time and day.

```
+-----------------------------------------------------------------------+
|                                                                       |
|                    ACTIVITY PATTERNS SECTION                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                           |                                           |
|  Trading Volume by        |  Active Traders Over Different            |
|  Weekday (Bar Chart)      |  Timeframes (Counters)                    |
|                           |                                           |
+-----------------------------------------------------------------------+
|                                                                       |
|  Trading Activity Heatmap (Hour of Day vs Day of Week)                |
|                                                                       |
+-----------------------------------------------------------------------+
```

## Color Scheme and Visual Design

### Color Palette

Use a consistent color palette throughout the dashboard:

1. Primary Color: #3366CC (Blue) - Use for main metrics and titles
2. Secondary Color: #FF9900 (Orange) - Use for highlighting important data points
3. Tertiary Colors:
   - #66CC66 (Green) - For positive trends
   - #CC3366 (Red) - For negative trends
   - #9966CC (Purple) - For additional categories

### Chart Design Guidelines

1. **Counters**: Use large, bold numbers with appropriate formatting (currency, percentages, etc.)
2. **Bar Charts**: Use consistent coloring, sort bars by value (usually descending), and include value labels
3. **Pie Charts**: Limit to 5-7 segments maximum, use a consistent color scheme, and include percentage labels
4. **Line Charts**: Use smooth lines, include data points, and consider using different line styles for multiple series
5. **Tables**: Use alternating row colors, align numbers right, and highlight important columns

### Text Elements

1. Add descriptive titles to each visualization
2. Include brief explanations of complex metrics
3. Add methodology notes where appropriate
4. Use consistent font sizes and styles:
   - Section Headers: 24px, Bold
   - Visualization Titles: 18px, Bold
   - Descriptions: 14px, Regular
   - Notes: 12px, Italic

## Interactive Elements

Leverage Dune's interactive features:

1. **Date Range Selector**: Add a global date range selector at the top of the dashboard
2. **Filters**: Add filters for trading pairs, user segments, etc.
3. **Tooltips**: Ensure all visualizations have informative tooltips
4. **Drill-downs**: Enable drill-down functionality where appropriate

## Mobile Optimization

Ensure the dashboard is readable on mobile devices:

1. Stack visualizations vertically on smaller screens
2. Ensure text is readable at smaller sizes
3. Simplify complex visualizations for mobile viewing

## Final Checklist

Before submitting your dashboard, ensure:

1. All visualizations load correctly
2. Data is accurate and up-to-date
3. The layout is visually appealing and logical
4. Text elements are clear and informative
5. Interactive elements work as expected
6. The dashboard performs well (queries run efficiently)
