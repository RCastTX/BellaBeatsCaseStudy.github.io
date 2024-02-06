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
