# Cyclistic
Cyclistic bike share case study: Exploring differences in ridership 
----

### Introduction
Cyclistic is a bike-share company in Chicago that has a fleet of 5,824 bicycles and 692 stations across the city. The company offers a flexible pricing plan that includes single-ride passes, full-day passes, and annual memberships. Cyclistic's finance analysts have determined that annual members are more profitable than casual riders, and the director of marketing, Lily Moreno, believes that maximizing the number of annual memberships is key to the company's future growth.

**Business Task: Analyze the historical bike trip data to understand how annual members and casual riders use Cyclistic bikes differently**

### Defining the problem

To achieve the goal of converting casual riders into annual members, Moreno's team needs to understand why casual riders would buy a membership and how digital media can influence their decision-making process. The team will use the insights gained from analyzing the historical bike trip data to identify trends and develop strategies that target the needs and preferences of casual riders.

The success of Cyclistic's future growth depends on maximizing the number of annual memberships. My role as a junior data analyst in the marketing analytics team is crucial in helping Cyclistic achieve this goal by providing data-driven insights and recommendations that will be used to design a new marketing strategy aimed at converting casual riders into annual members.

Data Source: Bike data is provided by [Motivate International Inc.](https://divvy-tripdata.s3.amazonaws.com/index.html)

----
### Preparing and Cleaning Data

The next step in this process is to properly prepare and clean data in preparation for analysis. After downloading and unzipping the 12 files, store the .CSV files in appropriate folders. Import each file into Google Sheets to add two columns: ride_length and day_of_week. Following these steps:
- Download data and store data appropriately on device
- Split .CSV files if file size is greater than 100,000 KB. Google Sheets cannot store files that exceed a limit
- Sort and filter data
  - Calculate length of ride time (ride_length)
    - =D2-C2
    - Format values into HH:MM:SS duration
  - Determine which day of week each rider departs from the start station (day_of_week)
    - =weekday(C2,1)
- Remove values that are negative or 0 from ride_length
- Store complete .CSV files in appropriate folders

----

### Data processing with SQL BigQuery

After preparing, cleaning, and processing data on Google Sheets, use SQL BigQuery to perform further processing, aggregation, and analysis. 

To continue working on the data, upload the large datasets to BigQuery’s Google Cloud bucket. Importing the files directly to BigQuery can cause errors due to file size exceeding the limit for upload. 

Create tables for each month and combine the tables into quarters for easier processing. Quarters can be divided into:
- Q1 - January, February, March
- Q2 - April, May, June
- Q3 - July, August, September
- Q4 - October, November, December

> Note: The data sources downloaded are from the previous 12 months and may contain incomplete quarters. Q1-2022 only contains data from March 2022 and Q1-2023 only contains data from January 2023 and February 2023. 

After combining the months into Quarters, use the count function to determine how many distinct rides there were per quarter. 

### Explore differences of casual vs. annual members

**Percentage of casual riders vs. annual members**

1. Create a nested query to perform a **COUNTIF** function to identify members with the values of ‘casual’ or ‘member’ and create an alias as casual_members or annual_members. Perform another **COUNT** function to determine total trips that members have taken. 
2. Create an outer query to calculate the percentage of casual and annual members that make up the total trips per quarter. Use UNION ALL to combine results from different quarters together, creating a comprehensive statistics of riders and ORDER BY quarter.

```
(SELECT
  'Q2-2022' AS quarter,
  total_trips,
  casual_members,
  annual_members,
  ROUND((casual_members/total_trips)*100) AS casual_member_percentage,
  ROUND((annual_members/total_trips)*100) AS annual_member_percentage
FROM
(SELECT
  COUNTIF(member_casual = 'casual') AS casual_members,
  COUNTIF(member_casual = 'member') AS annual_members,
  COUNT(ride_id) AS total_trips
FROM
  `projects-381620.tripdata.quarter_2`
)
)
UNION ALL
(SELECT
  'Q3-2022' AS quarter,
  total_trips,
  casual_members,
  annual_members,
  ROUND((casual_members/total_trips)*100) AS casual_member_percentage,
  ROUND((annual_members/total_trips)*100) AS annual_member_percentage
FROM
(SELECT
  COUNTIF(member_casual = 'casual') AS casual_members,
  COUNTIF(member_casual = 'member') AS annual_members,
  COUNT(ride_id) AS total_trips
FROM
  `projects-381620.tripdata.quarter_3`
)
)
UNION ALL
(SELECT
  'Q4-2022' AS quarter,
  total_trips,
  casual_members,
  annual_members,
  ROUND((casual_members/total_trips)*100) AS casual_member_percentage,
  ROUND((annual_members/total_trips)*100) AS annual_member_percentage
FROM
(SELECT
  COUNTIF(member_casual = 'casual') AS casual_members,
  COUNTIF(member_casual = 'member') AS annual_members,
  COUNT(ride_id) AS total_trips
FROM
  `projects-381620.tripdata.quarter_4`
)
)
UNION ALL
(SELECT
  'Q1-2023' AS quarter,
  total_trips,
  casual_members,
  annual_members,
  ROUND((casual_members/total_trips)*100) AS casual_member_percentage,
  ROUND((annual_members/total_trips)*100) AS annual_member_percentage
FROM
(SELECT
  COUNTIF(member_casual = 'casual') AS casual_members,
  COUNTIF(member_casual = 'member') AS annual_members,
  COUNT(ride_id) AS total_trips
FROM
  `projects-381620.tripdata.quarter_1_2023`
)
)
ORDER BY
  quarter ASC

```

**Calculate average ride time of casual and annual membership riders**

Create queries to determine the average ride time per quarter of casual, annual, and all members. 
1. Use the **TIMESTAMP_DIFF** function to calculate the difference of ended_at from started_at to obtain the ride length in seconds. Use the WHERE clause to further filter for members that are ‘casual’ or ‘members’.
2. With that as a nested query, calculate the AVG ride length from the nested query and divide by 60 to obtain ride length in minutes. Repeat for each quarter.
3. Use **JOIN** to join tables together for a comprehensive average table for all four quarters

```
SELECT
  Q2.membership_type,
  Q2.average_ride_minutes AS Q2_22_avg,
  Q3.average_ride_minutes AS Q3_22_avg,
  Q4.average_ride_minutes AS Q4_22_avg,
  Q1_23.average_ride_minutes AS Q1_23_avg
FROM
  `projects-381620.tripdata.average_rideQ2_22` AS Q2
JOIN 
  `projects-381620.tripdata.average_rideQ3_22` as Q3 ON
Q2.membership_type = Q3.membership_type
JOIN
  `projects-381620.tripdata.average_rideQ4_22` AS Q4 ON
Q2.membership_type = Q4.membership_type
JOIN
  `projects-381620.tripdata.average_rideQ1_23` AS Q1_23 ON
Q2.membership_type = Q1_23.membership_type
```

![Sheet 3 (1)](https://user-images.githubusercontent.com/128414862/228644266-84fc9dd4-0f7f-4a5d-b562-a24878ec6790.png)



**Peak days**

Now that we know average ride time, which days do riders prefer the most? 

```
SELECT
  day_of_week,
  count(CASE WHEN member_casual = 'casual' THEN 1 ELSE null END) AS casual_members,
  count(CASE WHEN member_casual = 'member' THEN 1 ELSE null END) AS annual_members
FROM
  (SELECT
    member_casual,
    CASE 
    WHEN day_of_week = 1 THEN 'Sunday'
    WHEN day_of_week = 2 THEN 'Monday'
    WHEN day_of_week = 3 THEN 'Tuesday'
    WHEN day_of_week = 4 THEN 'Wednesday'
    WHEN day_of_week = 5 THEN 'Thursday'
    WHEN day_of_week = 6 THEN 'Friday'
    WHEN day_of_week = 7 THEN 'Saturday'
    END AS day_of_week
  FROM
    `projects-381620.tripdata.quarter_2`
  )
GROUP BY
  day_of_week 
ORDER BY
CASE
    WHEN day_of_week = 'Sunday' THEN 1
    WHEN day_of_week = 'Monday' THEN 2
    WHEN day_of_week = 'Tuesday' THEN 3
    WHEN day_of_week = 'Wednesday' THEN 4
    WHEN day_of_week = 'Thursday' THEN 5
    WHEN day_of_week = 'Friday' THEN 6
    WHEN day_of_week = 'Saturday' THEN 7
    END ASC
   ```
   
 > Note: Repeat for each quarter
 
 ![Dashboard 1](https://user-images.githubusercontent.com/128414862/228640151-f7f3a922-3188-43d2-875b-d14d2b7d68b9.png)


### Insights 
Revisiting the business task: How do annual members and casual riders use Cyclistic bikes differently?


- There is a peak in casual riders who use Cyclistic bikeshare during Spring and Summer months (Q2-Q3)
- Casual riders increased 20% during these Spring and Summer months
- Annual members peaked during Fall and Winter months (Q4-Q1)
- Casual riders typically had a ride length greater than annual members
- Casual riders peak during Thursday-Sunday
- Annual riders peak during Monday-Thursday

### Recommendations

1. Spring/Summer promotion

Cyclistic has an opportunity to attract casual riders by leveraging its campaigns and promotions during the Spring and Summer months. By crafting compelling social media content, they can capture the attention of their audience and drive engagement. Social media content can include:

- Photos and videos showcasing scenic bike routes or beautiful destinations that can be reached by bike
- Tips and tricks on how to choose the right bike for casual rides or how to maintain your bike
- Inspiring stories of people who have taken up cycling as a hobby and how it has positively impacted their lives
- Interactive challenges or contests that encourage people to share their biking experiences or participate in group rides.

2. Weekend packages

To cater to the needs of casual riders who enjoy leisurely bike rides on weekends, it would be beneficial to promote packages that offer partnerships with local businesses to explore the stunning city of Chicago. These packages could be tailored to the preferences of riders who typically ride for extended periods. These packages could include:

- Coffee shop tours
- Musuem visits
- Guided tours
- Food tour

To optimize the company's growth and draw in more annual members, it's imperative to tap into the potential of tourism and bike riding. This exclusive and delightful experience can attract casual riders looking for something unique to try. The marketing team should focus on leveraging social media engagement and collaboration with local businesses



