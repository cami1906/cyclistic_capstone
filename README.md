/*
This is the SQL script used to import, compile and analyse the Cyclistic Case 
Study for the Capstone Project of the Google Data Analytics Professional Certificate.

It uses the data from Divvy Bikes (https://www.divvybikes.com/system-data).

The data used in this case study is located in the cloud of Amazon Web Services

I followed the next steps:

1. Download the individual CSV documents
2. Import each into separate tables, regularising data types
3. Combine all the data into one table
4. Inspect data for anomalies
5. Identify and exclude data with anomalies
6. Create queries for data visualisations

--The data is contained in 12 ‘.csv.’ spreadsheets. Each one contains data from January through December 2023, and 13 columns with the following information for each table:
o	ride_id
o	rideable_type
o	started_at
o	ended_at
o	start_station_name
o	start_station_id
o	end_station_name
o	end_station_id
o	start_lat
o	start_lng
o	end_lat
o	end_lng
o	member_casual

--I downloaded the original files from https://divvy-tripdata.s3.amazonaws.com/index.html, saved them on my computer and changed their names. For example, the name of the spreadsheet with the data for January 2023 is called ‘202301_tripdata’. When I first opened each file, I found that the TIMESTAMP format in ‘started_at’ and ‘ended_at’ columns was DD/MM/YYYY h:mm, so I changed it into YYYY-DD-MM h:mm:ss. This allowed me to upload the files in BigQuery ensuring format compatibility. 
The data is reliable, original, comprehensive, current, and cited. The data has valuable information that can help me find answers to the questions I have been tasked with. However, I still have to sort and filter information, and use SQL to compute some calculations that can help me understand the patterns of usage of the different types of users of the bike-share system.

The code below starts at step 3

--Step 3: Combine all the data into one table

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
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202303_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202304_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202305_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202306_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202307_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202308_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202309_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202310_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202311_tripdata`
UNION ALL
SELECT
  *
FROM
  `model-framing-412705.cyclistic.202312_tripdata`
--The resulting table had 5,719,743 rows. 

--When I initially explored the datasets in the ‘.csv’ spreadsheets, I found many fields were empty. I consider ‘ride_id’, ‘rideable_type’, ‘started_at’, ‘ended_at’, and ‘member_casual’ to be the columns where there should not be empty (NULL) fields. So I ran a few queries on the newly created `model-framing-412705.cyclistic.12month_tripdata`  to ensure data integrity in this respect. 

--1.	This query resulted in zero fields. 
SELECT 
COUNT (*)
FROM `model-framing-412705.cyclistic.12month_tripdata`
WHERE ride_id IS NULL

--2.	This query resulted in zero fields. 
SELECT 
COUNT (*)
FROM `model-framing-412705.cyclistic.12month_tripdata`
WHERE rideable_type IS NULL

--3.	 This query resulted in zero fields. 
SELECT 
COUNT (*)
FROM `model-framing-412705.cyclistic.12month_tripdata`
WHERE started_at IS NULL

--4.	This query resulted in zero fields. 
SELECT 
COUNT (*)
FROM `model-framing-412705.cyclistic.12month_tripdata`
WHERE ended_at IS NULL

--5.	This query resulted in zero fields. 
SELECT 
COUNT (*)
FROM `model-framing-412705.cyclistic.12month_tripdata`
WHERE member_casual IS NULL

--Since I was tasked with finding usage patterns of Cyclistic users, created a new table where I changed the names of the columns ‘rideable_type’, ‘started_at’, ‘ended_at’, and ‘member_casual’, for names that are more easily understandable to a broad audience. I also extracted the day of the week from the ‘started_at’ column, and calculated the number of minutes per trip. ‘minutes_per_trip’ is the difference in minutes between ‘ended_at’ and ‘started_at’. Additionally, I created a column called ‘month’ that contains the month where each trip started, which was extracted from ‘started_at’.  I considered all the rows where the bikes where not docked and minutes per trip is greater than 0. The new table contains the data ordered by ‘start_time’.

--This query creates a new table. In this new table, I created three new columns called 'day_of_week' and 'minutes_per_trip' and 'month'. I also changed the name of the columns 'rideable_type', 'started_at', 'ended_at', and 'member_casual'

CREATE TABLE `model-framing-412705.cyclistic.trip_data_report`
AS
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
  DATETIME_DIFF(ended_at, started_at, MINUTE) as minutes_per_trip,
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
  ) as filtered_data
ORDER BY
  start_time ASC;

--Trends or relationships in the data

--Maximum minutes_per_trip by membership_status
---Casual riders= 1560
SELECT 
ROUND(MAX(minutes_per_trip), 2) AS max_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'

---Member riders= 1560
SELECT 
ROUND(MAX(minutes_per_trip), 2) AS max_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'

--Minimum minutes_per_trip by membership_status
---Casual riders= 1
SELECT 
ROUND(MIN(minutes_per_trip), 2) AS min_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'

---Member riders= 1
SELECT 
ROUND(MIN(minutes_per_trip), 2) AS min_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'

--Average minutes_per_trip by membership status
---Casual riders =  22.48
SELECT 
ROUND(AVG(minutes_per_trip), 2) AS avg_min_per_trip_casual
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'

---Member riders = 12.73
SELECT 
ROUND(AVG(minutes_per_trip), 2) AS avg_min_per_trip_member
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'

--Maximum number of trips by membership status and type of bike with the most frequent duration

---Casual rider
type_of_bike: electric_bike: 6 minutes per trip; number_of_trips= 74,409
Since I was very interested in finding out how frequently casual riders used classic bikes compared to their electric counterpart, I expanded the search up to the most frequent trip duration in minutes. I found that the most frequent there were 41,006 trips on classic bycicles done by casual riders.

SELECT 
DISTINCT type_of_bike,
minutes_per_trip,
COUNT(ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY minutes_per_trip, type_of_bike
ORDER BY number_of_trips DESC 
LIMIT 20

--Then, I decided to calculate the number of trips by type of bike done by casual members.

SELECT 
type_of_bike,
COUNT(DISTINCT ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY type_of_bike
ORDER BY number_of_trips DESC 

---electric_bike= 1,080,285 trips, 
---classic_bike= 870,139.


---Member rider
type_of_bike: electric_bike: 5 minutes per trip; number_of_trips= 148,714
The rest of the top 5 is comprised by trips on electric bikes with the exception of classic bikes, which ranked 3rd with the most number of trips

SELECT 
DISTINCT type_of_bike,
minutes_per_trip,
COUNT(ride_id) as number_of_trips
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY minutes_per_trip, type_of_bike
ORDER BY number_of_trips DESC 
LIMIT 5

--Next, let's find out day_of_week by membership status, to calculate most popular day where both types of users make trips.

---Casual rider: Most popular is Saturday= 387375. Least popular: Monday= 222,314
SELECT 
COUNT(membership_status) AS number_of_trips,
day_of_week
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY day_of_week
ORDER BY number_of_trips DESC


---Member rider: Most popular is Thursday= 580,166. Least popular: Sunday= 402,425
SELECT 
COUNT(membership_status) AS number_of_trips,
day_of_week
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY day_of_week
ORDER BY number_of_trips DESC

--Next, let's find out month by membership status, to calculate most popular month where both types of users make trips.
---Casual rider: Most popular month: July. Least popular month: January
SELECT 
COUNT(membership_status) AS number_of_trips,
month
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY month
ORDER BY number_of_trips DESC

---Member rider: Most popular month: August. Least popular month: February
SELECT 
COUNT(membership_status) AS number_of_trips,
month
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY month
ORDER BY number_of_trips DESC

--Now, let's calculate the times where the most number of trips are made by casual riders. 
---Casual riders
SELECT 
membership_status,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT(*) as number_of_rides
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'casual'
GROUP BY membership_status, time_of_day
ORDER BY number_of_rides DESC

---Member riders
SELECT 
membership_status,
EXTRACT(HOUR FROM start_time) AS time_of_day,
COUNT(*) as number_of_rides
FROM `model-framing-412705.cyclistic.trip_data_report`
WHERE membership_status = 'member'
GROUP BY membership_status, time_of_day
ORDER BY number_of_rides DESC

--Most booked rides for both groups occur between 16 and 18 hours (4pm and 6:pm respectively). The peak time is 5pm

