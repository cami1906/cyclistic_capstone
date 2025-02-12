# Cyclistic Case Study: Data Analysis with Divvy Bikes Data

This repository contains the SQL script used for importing, compiling, and analyzing the Cyclistic Case Study as part of the Google Data Analytics Professional Certificate Capstone Project.

The visualisation of the analysis below is available on my profile in Tableau Public. Follow the link to check it out https://public.tableau.com/app/profile/camilo.toro/viz/cyclistic_capstone_project_17072760478730/Presentation

## Overview

- **Project Purpose**: Analyze Divvy Bikes data to understand usage patterns and user behavior.
- **Data Source**: [Divvy Bikes System Data](https://www.divvybikes.com/system-data)
- **Cloud Storage**: The data is stored in Amazon Web Services (AWS).

## Data Preparation Steps

### Step 1: Combine All Data into One Table

The goal of this step is to create a unified table containing ride records from all 12 months (January to December 2023).

```sql
CREATE TABLE `snappy-elf-359008.Cyclistic.12month_tripdata` AS
WITH combined_data AS (
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202301` -- January
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202302` -- February
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202303` -- March
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202304` -- April
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202305` -- May
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202306` -- June
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202307` -- July
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202308` -- August
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202309` -- September
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202310` -- October
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202311` -- November
  UNION ALL
  SELECT * FROM `snappy-elf-359008.Cyclistic.tripdata_202312` -- December
)
SELECT * FROM combined_data;
```

- Result: The combined table contains a total of **5,719,877 rows**.

### Step 2: Data Integrity Checks

To ensure data quality, we examined specific columns for empty (NULL) fields:

1. No missing `ride_id` values:
   ```sql
   SELECT COUNT(*) AS missing_ride_id
   FROM `snappy-elf-359008.Cyclistic.12month_tripdata`
   WHERE ride_id IS NULL;
   ```

2. No missing `rideable_type` values:
   ```sql
   SELECT COUNT(*) AS missing_rideable_type
   FROM `snappy-elf-359008.Cyclistic.12month_tripdata`
   WHERE rideable_type IS NULL;
   ```

3. No missing `started_at` values:
   ```sql
   SELECT COUNT(*) AS missing_started_at
   FROM `snappy-elf-359008.Cyclistic.12month_tripdata`
   WHERE started_at IS NULL;
   ```

4. No missing `ended_at` values:
   ```sql
   SELECT COUNT(*) AS missing_ended_at
   FROM `snappy-elf-359008.Cyclistic.12month_tripdata`
   WHERE ended_at IS NULL;
   ```

5. No missing `member_casual` values:
   ```sql
   SELECT COUNT(*) AS missing_member_casual
   FROM `snappy-elf-359008.Cyclistic.12month_tripdata`
   WHERE member_casual IS NULL;
   ```

### Step 3: Creating a New Table

- Renamed columns for better readability:
  - `rideable_type` → `bike_type`
  - `started_at` → `start_time`
  - `ended_at` → `end_time`
  - `member_casual` → `user_type`
- Extracted the day of the week from `start_time`.
- Calculated the duration of each trip in minutes (`minutes_per_trip`).
- Created a `month` column based on the month of each trip's start date.
- Filtered out rows where bikes were docked and `minutes_per_trip` was greater than 0.
- Ordered the data by `start_time`.


--This query creates a new table. In this new table, I created three new columns called 'day_of_week', 'month', and 'trip_duration_minutes. I also changed the name of the columns 'rideable_type', 'started_at', 'ended_at', and 'member_casual'. Finally, this query also deletes duplicate rows. 

```sql
CREATE TABLE `snappy-elf-359008.Cyclistic.trip_data_report` AS
WITH DeduplicatedData AS (
  SELECT
    ride_id,
    rideable_type AS bike_type,
    started_at AS start_time,
    ended_at AS end_time,
    FORMAT_TIMESTAMP('%a', started_at) AS day_of_week,
    FORMAT_TIMESTAMP('%b', started_at) AS month,
    TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS trip_duration_minutes,
    start_station_name,
    start_station_id,
    end_station_name,
    end_station_id,
    start_lat,
    start_lng,
    end_lat,
    end_lng,
    member_casual AS user_type,
    ROW_NUMBER() OVER (PARTITION BY ride_id ORDER BY started_at) AS row_num
  FROM
    `snappy-elf-359008.Cyclistic.12month_tripdata`
  WHERE
    rideable_type != 'docked_bike'
    AND TIMESTAMP_DIFF(ended_at, started_at, MINUTE) > 0
)
SELECT
  ride_id,
  bike_type,
  start_time,
  user_type,
  end_time,
  day_of_week,
  month,
  trip_duration_minutes,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
FROM
  DeduplicatedData
WHERE
  row_num = 1
ORDER BY
  start_time;
```

# Trends and Relationships in Bike Usage

In this section, we explore trends and relationships in bike usage data from Cyclistic, focusing on differences between annual members and casual riders. We'll analyze key metrics related to trip duration and bike types.

## Trip Duration Analysis

### Maximum Trip Duration

- **Casual Riders**:
  - The maximum trip duration for casual riders is **1,560 minutes**.
```sql
SELECT 
ROUND(MAX(trip_duration_minutes), 2) AS max_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
```

- **Member Riders**:
  - The maximum trip duration for member riders is also **1,560 minutes**.
```sql
SELECT 
ROUND(MAX(trip_duration_minutes), 2) AS max_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
```

### Minimum Trip Duration

- **Casual Riders**:
  - The minimum trip duration for casual riders is **1 minute**.
```sql
SELECT 
ROUND(MIN(trip_duration_minutes), 2) AS min_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
```

- **Member Riders**:
  - The minimum trip duration for member riders is also **1 minute**.
```sql
SELECT 
ROUND(MIN(trip_duration_minutes), 2) AS min_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
```

### Average Trip Duration

- **Casual Riders**:
  - The average trip duration for casual riders is **22.48 minutes**.
```sql
SELECT 
ROUND(AVG(trip_duration_minutes), 2) AS avg_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
```
- **Member Riders**:
  - The average trip duration for member riders is **12.73 minutes**.
```sql
SELECT 
ROUND(AVG(trip_duration_minutes), 2) AS avg_trip_duration_minutes
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
```
## Bike Type Analysis

### Overall ranking of number of trips 
  - Members were the most frequent users of the system, with classic bicycles being their preferred choice, followed by electric bikes. Among casual riders, electric bikes were more popular than classic bikes, making classic bike rides by casual users the least common.
```sql
WITH bike_stats AS (
  SELECT 
    bike_type,
    user_type,
    ROUND(AVG(trip_duration_minutes), 2) AS avg_trip_duration,
    COUNT ride_id AS number_of_trips
  FROM 
    `snappy-elf-359008.Cyclistic.trip_data_report`
  GROUP BY 
    user_type, bike_type
),
ranked_bikes AS (
  SELECT 
    bike_type,
    user_type,
    avg_trip_duration,
    number_of_trips,
    RANK() OVER (ORDER BY number_of_trips DESC) AS overall_rank
  FROM 
    bike_stats
)
SELECT 
  overall_rank,
  bike_type,
  user_type,
  avg_trip_duration,
  FORMAT("%'d", number_of_trips) AS formatted_number_of_trips
FROM 
  ranked_bikes
ORDER BY 
  overall_rank;
```

### Classic Bikes vs. Electric Bikes by user_type

- Members prefer **classic bicycles**, making **1,801,659**.
- Casual riders prefer **electric bicycles**, making **1,080,285 trips**.
```sql
SELECT 
  user_type,
  bike_type,
  COUNT ride_id AS number_of_trips
FROM 
  `snappy-elf-359008.Cyclistic.trip_data_report`
GROUP BY 
  user_type,
  bike_type
ORDER BY 
  user_type,
  number_of_trips DESC;
```

## Casual Riders vs. Members: Bike Usage Trends

### Top 20 Trip Durations by Bike Type (Casual Riders)

```sql
SELECT bike_type,
trip_duration_minutes,
COUNT ride_id as number_of_trips
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
GROUP BY trip_duration_minutes, bike_type
ORDER BY trip_duration_minutes DESC 
LIMIT 20
```
### Most Frequent Bike Type for Casual Riders

- **Electric Bikes**: Casual riders took **1,080,286 trips** on electric bikes.
- **Classic Bikes**: Casual riders took **870,140 trips** on classic bikes.
```sql
SELECT bike_type,
COUNT ride_id as number_of_trips
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
GROUP BY bike_type
ORDER BY number_of_trips DESC 
```

### Top 5 Trip Durations by Bike Type (Members)

- **Electric Bikes**:| Minutes per trip: **5** |  Number of trips: **148,714**
- **Electric Bikes**:| Minutes per trip: **4** | Number of trips: **146,475**
- **Classic Bikes**: | Minutes per trip: **5**  | Number of trips: **140,994**
- **Electric Bikes**:| Minutes per trip: **6** | Number of trips: **140,384**
- **Classic Bikes**: | Minutes per trip: **4**  | Number of trips: **137,156**
  
```sql
SELECT bike_type,
trip_duration_minutes,
COUNT ride_id as number_of_trips
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
GROUP BY trip_duration_minutes, bike_type
ORDER BY number_of_trips DESC 
LIMIT 5
```

### Most Popular Days for Riding

- **Casual Riders**:
  - Most popular day: **Saturday** (387,375 trips)
  - Least popular day: **Monday** (222,314 trips)
 
```sql
SELECT 
COUNT ride_id AS number_of_trips,
day_of_week
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
GROUP BY day_of_week
ORDER BY number_of_trips DESC
```

- **Members**:
  - Most popular day: **Thursday** (580,166 trips)
  - Least popular day: **Sunday** (402,425 trips)
```sql
SELECT 
COUNT ride_id AS number_of_trips,
day_of_week
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
GROUP BY day_of_week
ORDER BY number_of_trips DESC
```

### Most Popular Months for Riding
- **Casual Riders**:
  - Most popular month: **July**
  - Least popular month: **January**
```sql
SELECT 
COUNT ride_id AS number_of_trips,
month
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
GROUP BY month
ORDER BY number_of_trips DESC
```

- **Members**
  - Most popular month: **August**
  - Least popular month: **February**
```sql
SELECT 
COUNT ride_id AS number_of_trips,
month
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
GROUP BY month
ORDER BY number_of_trips DESC
```

## Popular Riding Times by Membership Status

### Casual Riders

- Most popular time of day: **5:00 PM** (peak hour) with **189,276 trips**.
- Least popular time of day: **4:00 AM** with **5,724 trips**.
```sql
SELECT 
user_type,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT ride_id AS number_of_rides
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'casual'
GROUP BY user_type, time_of_day
ORDER BY number_of_rides DESC
```
### Members

- Most popular time of day: **5:00 PM** (peak hour) with **381,846 trips**.
- Least popular time of day: **3:00 AM** with **7,824 trips**.
```sql
SELECT 
user_type,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT ride_id AS number_of_rides
FROM `snappy-elf-359008.Cyclistic.trip_data_report`
WHERE user_type = 'member'
GROUP BY user_type, time_of_day
ORDER BY number_of_rides DESC
```

## Findings

The results of the analysis show that:
- Most popular day with casual riders is Saturday, and least popular is Tuesday. The most popular day for members is  Wednesday, and least popular is Sunday.
- Most poplular month for casual riders is July, and least popular is December. Most popular month for member riders is August, and least is February.
- Most book rides for both groups occurs during hours 17, 18, and 16 in order (i.e. 5pm, 6pm, and 4pm).

## Recommendations

1. Launch a membership plan to target users of the system in Summer, as well as weekend membership packages. These measures might entice casual riders to pay for a membership.
2. Create a referral system that can benefit both members and the company. The former might benefit via discounts in their membership cost, whereas the latter might benefit from an increase in users from potential new users referred to the company by existing users. 



