# Cyclistic Case Study: Data Analysis with Divvy Bikes Data

This repository contains the SQL script used for importing, compiling, and analyzing the Cyclistic Case Study as part of the Google Data Analytics Professional Certificate Capstone Project.

The visualisation of the analysis below is available on my profile in Tableau Public. Follow the link to check it out https://public.tableau.com/app/profile/camilo.toro/viz/cyclistic_capstone_project_17072760478730/Totalridesbyhour

## Overview

- **Project Purpose**: Analyze Divvy Bikes data to understand usage patterns and user behavior.
- **Data Source**: [Divvy Bikes System Data](https://www.divvybikes.com/system-data)
- **Cloud Storage**: The data is stored in Amazon Web Services (AWS).

## Data Preparation Steps

### Step 1: Combine All Data into One Table

The goal of this step is to create a unified table containing ride records from all 12 months (January to December 2023).

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

- Result: The combined table contains a total of **5,719,743 rows**.

### Step 2: Data Integrity Checks

To ensure data quality, we examined specific columns for empty (NULL) fields:

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

3. No missing `started_at` values:
   ```sql
   SELECT COUNT(*)
   FROM `model-framing-412705.cyclistic.12month_tripdata`
   WHERE started_at IS NULL;
   ```

4. No missing `ended_at` values:
   ```sql
   SELECT COUNT(*)
   FROM `model-framing-412705.cyclistic.12month_tripdata`
   WHERE ended_at IS NULL;
   ```

5. No missing `member_casual` values:
   ```sql
   SELECT COUNT(*)
   FROM `model-framing-412705.cyclistic.12month_tripdata`
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


--This query creates a new table. In this new table, I created three new columns called 'day_of_week' and 'minutes_per_trip' and 'month'. I also changed the name of the columns 'rideable_type', 'started_at', 'ended_at', and 'member_casual'

```sql
CREATE TABLE `model-framing-412705.cyclistic.trip_data_report` AS
SELECT
  ride_id,
  rideable_type AS type_of_bike,
  started_at AS start_time,
  ended_at AS finish_time,
  CASE EXTRACT(DAYOFWEEK FROM started_at)
    WHEN 1 THEN 'Sun'
    WHEN 2 THEN 'Mon'
    WHEN 3 THEN 'Tue'
    WHEN 4 THEN 'Wed'
    WHEN 5 THEN 'Thur'
    WHEN 6 THEN 'Frid'
    WHEN 7 THEN 'Sat'
  END AS day_of_week,
  CASE EXTRACT(MONTH FROM started_at) 
    WHEN 1 THEN 'Jan'
    WHEN 2 THEN 'Feb'
    WHEN 3 THEN 'Mar'
    WHEN 4 THEN 'Apr'
    WHEN 5 THEN 'May'
    WHEN 6 THEN 'Jun'
    WHEN 7 THEN 'July'
    WHEN 8 THEN 'Aug'
    WHEN 9 THEN 'Sep'
    WHEN 10 THEN 'Oct'
    WHEN 11 THEN 'Nov'
    WHEN 12 THEN 'Dec'
  END AS month,
  DATETIME_DIFF(ended_at, started_at, MINUTE) AS minutes_per_trip,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  member_casual AS membership_status 
FROM
  (
    SELECT *
    FROM `model-framing-412705.cyclistic.12month_tripdata`
    WHERE rideable_type != "docked_bike" 
      AND DATETIME_DIFF(ended_at, started_at, MINUTE) > 0
  ) AS filtered_data
ORDER BY
  start_time ASC;
```
# Trends and Relationships in Bike Usage

In this section, we explore trends and relationships in bike usage data from Cyclistic, focusing on differences between annual members and casual riders. We'll analyze key metrics related to trip duration and bike types.

## Trip Duration Analysis

### Maximum Trip Duration

- **Casual Riders**:
  - The maximum trip duration for casual riders is **1,560 minutes**.
```sql
SELECT 
ROUND(MAX(minutes_per_trip), 2) AS max_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
```

- **Member Riders**:
  - The maximum trip duration for member riders is also **1,560 minutes**.
```sql
SELECT 
ROUND(MAX(minutes_per_trip), 2) AS max_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
```

### Minimum Trip Duration

- **Casual Riders**:
  - The minimum trip duration for casual riders is **1 minute**.
```sql
SELECT 
ROUND(MIN(minutes_per_trip), 2) AS min_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
```

- **Member Riders**:
  - The minimum trip duration for member riders is also **1 minute**.
```sql
SELECT 
ROUND(MIN(minutes_per_trip), 2) AS min_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
```

### Average Trip Duration

- **Casual Riders**:
  - The average trip duration for casual riders is **22.48 minutes**.
```sql
SELECT 
ROUND(AVG(minutes_per_trip), 2) AS avg_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
```
- **Member Riders**:
  - The average trip duration for member riders is **12.73 minutes**.
```sql
SELECT 
ROUND(AVG(minutes_per_trip), 2) AS avg_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
```
## Bike Type Analysis

### Most Frequent Bike Type for Casual Riders

- **Bike Type**: Electric Bike
- **Average Trip Duration**: 6 minutes per trip
- **Number of Trips**: 74,409

### Classic Bikes vs. Electric Bikes by membership status

- Among casual riders, the most frequent bike type is **classic bicycles**, with **41,006 trips**.

## Casual Riders vs. Members: Bike Usage Trends

### Top 20 Trip Durations by Bike Type (Casual Riders)

```sql
SELECT 
DISTINCT type_of_bike,
minutes_per_trip,
COUNT(ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY minutes_per_trip, type_of_bike
ORDER BY number_of_trips DESC 
LIMIT 20
```
### Most Frequent Bike Type for Casual Riders

- **Electric Bikes**: Casual riders took **1,080,285 trips** on electric bikes.
- **Classic Bikes**: Casual riders took **870,139 trips** on classic bikes.
```sql
SELECT 
type_of_bike,
COUNT(DISTINCT ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY type_of_bike
ORDER BY number_of_trips DESC 
```

### Top 5 Trip Durations by Bike Type (Members)

- **Electric Bikes**:| Minutes per trip: **5** |  Number of trips: **148,714**
- **Electric Bikes**:| Minutes per trip: **4** | Number of trips: **146,475**
- **Classic Bikes**: | Minutes per trip: **5**  | Number of trips: **140,994**
- **Electric Bikes**:| Minutes per trip: **6** | Number of trips: **140,384**
- **Classic Bikes**: | Minutes per trip: **4**  | Number of trips: **137,156**
  
```sql
SELECT 
DISTINCT type_of_bike,
minutes_per_trip,
COUNT(ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY minutes_per_trip, type_of_bike
ORDER BY number_of_trips DESC 
LIMIT 5
```

### Most Popular Days for Riding

- **Casual Riders**:
  - Most popular day: **Saturday** (387,375 trips)
  - Least popular day: **Monday** (222,314 trips)
 
```sql
SELECT 
COUNT(membership_status) AS number_of_trips,
day_of_week
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY day_of_week
ORDER BY number_of_trips DESC
```

- **Members**:
  - Most popular day: **Thursday** (580,166 trips)
  - Least popular day: **Sunday** (402,425 trips)
```sql
SELECT 
COUNT(membership_status) AS number_of_trips,
day_of_week
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY day_of_week
ORDER BY number_of_trips DESC
```

### Most Popular Months for Riding
- **Casual Riders**:
  - Most popular month: **July**
  - Least popular month: **January**
``sql
SELECT 
COUNT(membership_status) AS number_of_trips,
month
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY month
ORDER BY number_of_trips DESC
``

- **Members**
  - Most popular month: **August**
  - Least popular month: **February**
```sql
SELECT 
COUNT(membership_status) AS number_of_trips,
month
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY month
ORDER BY number_of_trips DESC
```

## Popular Riding Times by Membership Status

### Casual Riders

- Most popular time of day: **5:00 PM** (peak hour) with **189,276 trips**.
- Least popular time of day: **4:00 AM** with **5,724 trips**.
```sql
SELECT 
membership_status,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT(*) as number_of_rides
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY membership_status, time_of_day
ORDER BY number_of_rides DESC
```
### Members

- Most popular time of day: **5:00 PM** (peak hour) with **381,846 trips**.
- Least popular time of day: **3:00 AM** with **7,824 trips**.
```sql
SELECT 
membership_status,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT(*) as number_of_rides
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY membership_status, time_of_day
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



