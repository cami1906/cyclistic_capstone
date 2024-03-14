# Combining Divvy Bikes Data: Creating a Unified Table

This section outlines the process of combining data from individual Divvy Bikes CSV files into a single table for analysis. The resulting table, named `model-framing-412705.cyclistic.12month_tripdata`, contains ride records spanning 12 months (January to December 2023).

## SQL Script

```sql
CREATE TABLE `model-framing-412705.cyclistic.12month_tripdata` AS
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202301_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202302_tripdata`
-- ... Repeat for all remaining months ...
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202312_tripdata`;
```

## Result

The combined table contains a total of **5,719,743 rows**.

## Data Integrity Checks

To ensure data quality, we examined specific columns for empty (NULL) fields. The following queries confirmed data integrity:

1. No missing `ride_id` values:
   ```sql
   SELECT COUNT(*)
   FROM `model-framing-412705.cyclistic.12month_tripdata`
   WHERE ride_id IS NULL;
   ```

2. No missing `rideable_type` values:
   ```sql
   SELECT COUNT(*)
   FROM `model-framing-412705.cyclistic.12month_tripdata`
   WHERE rideable_type IS NULL;
   ```