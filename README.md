# BellaBeatsCaseStudy
## Case Study Scenario
Bellabeat, a high-manufacturer of health-focused products for women, has asked for an analysis of how consumers are using non-Bellabeat smart devices. They are looking for how trends in the data can help the marketing team improve one of their smart devices. 

## Question to consider
* What are some trends in smart device usage?
* How could these trends apply to Bellabeat customers?
* How could these trends help influence Bellabeat marketing strategy?

## Business Task
How can Bellabeat improve its smart devices by identifying consumer trends and implementing solid marketing strategies for Bellabeat Ivy Health Tracker?

# BigQuery Import Process
Had a problem importing certain .csv files into BigQuery since the format for the datetime in the files was incorrect. I wrote a Python code to correct all datetime columns to the right format. Having to convert 14 .csv files to the proper format is why in the code you will see *'column_name'* which I would replace with the correct column name for that table. Also didn't want to show the exact file path on my computer, so I put *'/path/to/table/being/formatted.csv'*
