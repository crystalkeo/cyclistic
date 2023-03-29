# cyclistic
Cyclistic bike share case study: Exploring differences in ridership 
----

Introduction
Cyclistic is a bike-share company in Chicago that has a fleet of 5,824 bicycles and 692 stations across the city. The company offers a flexible pricing plan that includes single-ride passes, full-day passes, and annual memberships. Cyclistic's finance analysts have determined that annual members are more profitable than casual riders, and the director of marketing, Lily Moreno, believes that maximizing the number of annual memberships is key to the company's future growth.

Business Task: Analyze the historical bike trip data to understand how annual members and casual riders use Cyclistic bikes differently. 

Defining the problem
To achieve the goal of converting casual riders into annual members, Moreno's team needs to understand why casual riders would buy a membership and how digital media can influence their decision-making process. The team will use the insights gained from analyzing the historical bike trip data to identify trends and develop strategies that target the needs and preferences of casual riders.

The success of Cyclistic's future growth depends on maximizing the number of annual memberships. My role as a junior data analyst in the marketing analytics team is crucial in helping Cyclistic achieve this goal by providing data-driven insights and recommendations that will be used to design a new marketing strategy aimed at converting casual riders into annual members.

Bike data is provided by Motivate International Inc.

----
Preparing and Cleaning Data
Google Sheets
The next step in this process is to properly prepare and clean data in preparation for analysis. After downloading and unzipping the 12 files, store the .CSV files in appropriate folders. Import each file into Google Sheets to add two columns: ride_length and day_of_week. Following these steps:

Download data and store data appropriately on device
Split .CSV files if file size is greater than 100,000 KB. Google Sheets cannot store files that exceed a limit
Sort and filter data
Calculate length of ride time (ride_length)
=D2-C2
Format values into HH:MM:SS duration
Determine which day of week each rider departs from the start station (day_of_week)
=weekday(C2,1)
Remove values that are negative or 0 from ride_length
Store complete .CSV files in appropriate folders

----

SQL BigQuery
After preparing, cleaning, and processing data on Google Sheets, use SQL BigQuery to perform further processing, aggregation, and analysis. 

To continue working on the data, upload the large datasets to BigQuery’s Google Cloud bucket. Importing the files directly to BigQuery can cause errors due to file size exceeding the limit for upload. 

Create tables for each month and combine the tables into quarters for easier processing. Quarters can be divided into:
Q1 - January, February, March
Q2 - April, May, June
Q3 - July, August, September
Q4 - October, November, December

The data sources downloaded are from the previous 12 months and may contain incomplete quarters. Q1-2022 only contains data from March 2022 and Q1-2023 only contains data from January 2023 and February 2023. 

Querying rides per quarter
After combining the months into Quarters, use the count function to determine how many distinct rides there were per quarter. 

Casual vs. Annual members
Create a nested query to perform a COUNTIF function to identify members with the values of ‘casual’ or ‘member’ and create an alias as casual_members or annual_members. Perform another COUNT function to determine total trips that members have taken. 
Create an outer query to calculate the percentage of casual and annual members that make up the total trips per quarter. Use UNION ALL to combine results from different quarters together, creating a comprehensive statistics of riders and ORDER BY quarter.

Calculate average ride time of casual and annual membership riders
Create queries to determine the average ride time per quarter of casual, annual, and all members. Use the TIMESTAMP_DIFF function to calculate the difference of ended_at from started_at to obtain the ride length in seconds. Use the WHERE clause to further filter for members that are ‘casual’ or ‘members’.

With that as a nested query, calculate the AVG ride length from the nested query and divide by 60 to obtain ride length in minutes. Repeat for each quarter.

