# BellaBeatsCaseStudy
## Case Study Scenario
Bellabeat, a high-manufacturer of health-focused products for women, has asked to analyze how consumers use non-Bellabeat smart devices. They are looking for how trends in the data can help the marketing team improve one of their smart devices. 

## Question to consider
* What are some trends in smart device usage?
* How could these trends apply to Bellabeat customers?
* How could these trends help influence Bellabeat's marketing strategy?

## Business Task
How can Bellabeat improve its smart devices by identifying consumer trends and implementing solid marketing strategies for Bellabeat Ivy Health Tracker?

# BigQuery Import Process
Had a problem importing certain .csv files into BigQuery since the format for the datetime in the files was incorrect. I wrote a Python code to correct all datetime columns to the right format. Having to convert 14 .csv files to the proper format is why in the code you will see *'column_name'* which I would replace with the correct column name for that table. Also didn't want to show the exact file path on my computer, so I put *'/path/to/table/being/formatted.csv'*

# Python code for datetime conversion
```python
#converting "table_name" to proper datatime format
import pandas as pd

def convert_datetime(row):
    #Convert the date and time string to a pandas datetime object
    dt=pd.to_datetime(row['column_name'],format='%m/%d/%Y%I:%M:%S%p')
    return dt

#Input file path
input_file_path = '/path/to/table/being/formatted.csv'

#Load the CSV file
df=pd.read_csv(input_file_path)

#Apply the conversion function to the timestamp column
df['column_name']=df.apply(convert_datetime,axis=1)

#Save the modified DataFrame to a new CSV file
output_file_path='file/path/to/formatted/table.csv'
df.to_csv(output_file_path,index=False)
```
# Cleaning Process through BigQuery
**1 Step:**
Use the *Distinct* function to make sure there weren't any duplicates in any of my tables. I checked Id and datetime columns. Replaced "table_name" with table name checked. 

```sql
SELECT
    DISTINCT Id
FROM 'clear-style-381214.BellaBeats."table_name"'
ORDER BY
Id;
```
```sql
SELECT
    DISTINCT ActivityDate
FROM 'clear-style-381214.BellaBeats."table_name"'
ORDER BY
    ActivityDate
```
**2nd Step:**
Use the *ROUND* function to round all decimals to the second decimal place for cleaner tables. Only 3 tables needed to be cleaned up this way; dailyActiviy, minuteCaloriesNarrow, and weightLogInfo
```sql
#Query for dailyActivity table
SELECT
    Id,
    ActivityDate,
    TotalSteps,
    ROUND(TotalDistance,2) AS TotalDistance,
    ROUND(TrackerDistance,2) AS TrackerDistance,
    LoggedActivitiesDistance,
    ROUND(VeryActiveDistance,2) AS VeryActiveDistance,
    ROUND(ModeratelyActiveDistance,2) AS ModeratelyActiveDistance,
    ROUND(LightActiveDistance,2) AS LightActiveDistance,
    ROUND(SedentaryActiveDistance,2) AS SendentaryActiveDistance,
    VeryActiveMinutes,
    FairlyActiveMinutes,
    LightlyActiveMinutes,
    SedentaryMinutes,
    Calories
FROM 'clear-style-381214.BellaBeats.dailyActivity'
```
```sql
#Query for minuteCaloriesNarrow table
SELECT
    Id,
    ActivityMinute,
    ROUND(Calories,3) AS Calories
FROM 'clear-style-381214.BellaBeats.minuteCalories_Narrow'
ORDER BY
    Id
```
```sql
#Query for weightLogInfo table
SELECT
    Id,
    Date,
    ROUND(WeightKg,2) AS WeightKg,
    ROUND(WeightPounds,2) AS WeightPounds,
    COALESCE(Fat,0) AS Fat,
    ROUND(BMI,2) AS BMI,
    IsManualReport,
    LogId
FROM 'clear-style-381214.BellaBeats.weight_LogInfo'
ORDER BY
Id
```
**3rd Step:**
Make sure no tables have Null Values. Only 1 table had null values and it was the 
weightLogInfo table. Used the COALESCE function in the previous code to change them to zeros.
Went through every column name in the next query for every table.
```sql
SELECT
  Col1,
  Col2,
  Col3
FROM `clear-style-381214.BellaBeats.”table_name”`
WHERE
  “column_name” is NULL
ORDER BY
 “column_name”
```
# Analyze Dataset's
I started by making a comprehensive table of breaking down dow(day of the week) and time of day for physical activity based on the intensity level. Started by setting variables and then setting variables for the time of day/ dow analysis.

```sql
DECLARE
 MORNING_START INT64;
DECLARE
 MORNING_END INT64;
DECLARE
 AFTERNOON_END INT64;
DECLARE
 EVENING_END INT64;
SET
 MORNING_START = 6;
SET
 MORNING_END = 12;
SET
 AFTERNOON_END = 18;
SET
 EVENING_END = 21;
WITH
 user_dow_summary AS (
 SELECT
   Id,
   FORMAT_TIMESTAMP("%w", ActivityHour) AS dow_number,
   FORMAT_TIMESTAMP("%A", ActivityHour) AS day_of_week,
   CASE
     WHEN FORMAT_TIMESTAMP("%A", ActivityHour) IN ("Sunday", "Saturday") THEN "Weekend"
     WHEN FORMAT_TIMESTAMP("%A", ActivityHour) NOT IN ("Sunday",
     "Saturday") THEN "Weekday"
   ELSE
   "ERROR"
 END
   AS part_of_week,
   CASE
     WHEN TIME(ActivityHour) BETWEEN TIME(MORNING_START, 0, 0) AND TIME(MORNING_END, 0, 0) THEN "Morning"
     WHEN TIME(ActivityHour) BETWEEN TIME(MORNING_END,
     0,
     0)
   AND TIME(AFTERNOON_END,
     0,
     0) THEN "Afternoon"
     WHEN TIME(ActivityHour) BETWEEN TIME(AFTERNOON_END, 0, 0) AND TIME(EVENING_END, 0, 0) THEN "Evening"
     WHEN TIME(ActivityHour) >= TIME(EVENING_END,
     0,
     0)
   OR TIME(TIMESTAMP_TRUNC(ActivityHour, MINUTE)) <= TIME(MORNING_START,
     0,
     0) THEN "Night"
   ELSE
   "ERROR"
 END
   AS time_of_day,
   SUM(TotalIntensity) AS total_intensity,
   SUM(AverageIntensity) AS total_average_intensity,
   AVG(AverageIntensity) AS average_intensity,
   MAX(AverageIntensity) AS max_intensity,
   MIN(AverageIntensity) AS min_intensity
 FROM
   `clear-style-381214.BellaBeats.hourly_Intensities`
 GROUP BY
   1,
   2,
   3,
   4,
   5),
 intensity_deciles AS (
 SELECT
   DISTINCT dow_number,
   part_of_week,
   day_of_week,
   time_of_day,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.1) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_first_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.2) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_second_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.3) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_third_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.4) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_fourth_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.6) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_sixth_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.7) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_seventh_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.8) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_eigth_decile,
   ROUND(PERCENTILE_CONT(total_intensity,
       0.9) OVER (PARTITION BY dow_number, part_of_week, day_of_week, time_of_day),4) AS total_intensity_ninth_decile
 FROM
   user_dow_summary ),
 basic_summary AS (
 SELECT
   part_of_week,
   day_of_week,
   time_of_day,
   SUM(total_intensity) AS total_total_intensity,
   AVG(total_intensity) AS average_total_intensity,
   SUM(total_average_intensity) AS total_total_average_intensity,
   AVG(total_average_intensity) AS average_total_average_intensity,
   SUM(average_intensity) AS total_average_intensity,
   AVG(average_intensity) AS average_average_intensity,
   AVG(max_intensity) AS average_max_intensity,
   AVG(min_intensity) AS average_min_intensity
 FROM
   user_dow_summary
 GROUP BY
   1,
   dow_number,
   2,
   3)
SELECT
 *
FROM
 basic_summary
LEFT JOIN
 intensity_deciles
USING
 (part_of_week,
   day_of_week,
   time_of_day)
ORDER BY
 1,
 dow_number,
 2,
 CASE
   WHEN time_of_day = "Morning" THEN 0
   WHEN time_of_day = "Afternoon" THEN 1
   WHEN time_of_day = "Evening" THEN 2
   WHEN time_of_day = "Night" THEN 3
END
```
Next, I created a query to calculate the sleep patterns of the participants. With
seeing how much sleep they were getting per day and how many naps per day.
```sql
 SELECT
 Id,
 sleep_start AS sleep_date,
 COUNT(logId) AS number_naps,
 SUM(EXTRACT(HOUR
   FROM
     time_sleeping)) AS total_time_sleeping
FROM (
 SELECT
   Id,
   logId,
   MIN(DATE(date)) AS sleep_start,
   MAX(DATE(date)) AS sleep_end,
   TIME( TIMESTAMP_DIFF(MAX(date),MIN(date),HOUR),
     MOD(TIMESTAMP_DIFF(MAX(date),MIN(date),MINUTE),60),
     MOD(MOD(TIMESTAMP_DIFF(MAX(date),MIN(date),SECOND),3600),60) ) AS time_sleeping
 FROM
   `clear-style-381214.BellaBeats.minute_Sleep`
 WHERE
   value=1
 GROUP BY
   1,
   2)
WHERE
 sleep_start=sleep_end
GROUP BY
 1,
 2
ORDER BY
 1 ASC;
```
The last data set I used was a data set of fitness consumer survey data that asked 30 participants various questions on how they used a fitness wearable in their daily lives. Imported this into a Google sheet with the dow results table and sleep patterns summary results. 

With my results imported into multiple Google sheets, I created six pivot tables that were used to show my final analysis of the data. 
# List of pivot tables 
* dow_Intensities - dow_intensities
* Sleeping_Patterns - Sleep_Averages_and_Totals
* Fitness_Consumer_Survey_Data - Age_group_wearing_and_tracking_fitness
* Fitness_Consumer_Survey_Data - How_often_consumers_wear_fitness_wearable
* Fitness_Consumer_Survey_Data - Sleep_patterns_Improvement_Male_and_Female

# Data Visualization through Tableau
## Day of the Week summary
An analysis of the physical intensity of consumers based on the day of the week and time of day.
![DOW_Visual - Copy](https://github.com/RCastTX/BellaBeatsCaseStudy.github.io/assets/128720212/64e2ad3e-a9ba-481a-a844-9b3c0274aba6)

## Insights
*Afternoons are the most active time during the DOW for consumers.

*Tuesday, Wednesday, and Thursday are the top 3 most active days for the week.

*Afternoons are the most active, but mornings are a very close second. 

*Afternoons and mornings are the top times for physical activities with averages for both at least doubling or tripling intensities during the evening and night times.

## Age and Gender demographics
Analysis of gender groups and age groups that engage in fitness tracking. A data set of 30 interviewed consumers about tracking fitness through a fitness wearable.
![Age_and_Gender_Demographics_for_tracking_fitness - Copy](https://github.com/RCastTX/BellaBeatsCaseStudy.github.io/assets/128720212/de501d1c-94af-49ab-830f-b2bec0637dfc)
![Frequency_of_Tracking_Fitness - Copy](https://github.com/RCastTX/BellaBeatsCaseStudy.github.io/assets/128720212/f1137720-9c9d-474d-9571-05887ad412f7)

## Insights
*The 18-24 age group seems to be the most engaged with a fitness wearable.

*Nearly 70 percent of all 18-24-year-olds wear the fitness tracker every day or every other day.

*When it comes to gender, 30 percent of Male consumers wear the fitness tracker every day to every other day. While 23 percent of Females do.



