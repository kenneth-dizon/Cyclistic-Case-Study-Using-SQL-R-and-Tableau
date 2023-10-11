# Cyclistic Case Study Using SQL, R, and Tableau

<p align=”center”>
<img src="https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/8d76c80f-bf8f-4401-9a96-32e307f95d7c">
</p>
  
Welcome! In this case study, I used SQL and R to prepare and process and Tableau to visualize data for the fictional company Cyclistic. 

## Table of Contents
- [Scenario](#scenario)
- [Data Preparation and Processing, SQL](#data-preparation-and-processing-sql)
- [Data Preparation and Processing, R](#data-preparation-and-processing-r)
- [Data Visualization, Tableau](#data-visualization-tableau)
- [Recommendations](#Recommendations)

## Scenario
You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

About the company
“In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime.

Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One approach that helped make these things possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, Moreno believes that maximizing the number of annual members will be key to future growth. Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs.

Moreno has set a clear goal: Design marketing strategies aimed at converting casual riders into annual members. In order to do that, however, the marketing analyst team needs to better understand how annual members and casual riders differ, why casual riders would buy a membership, and how digital media could affect their marketing tactics. Moreno and her team are interested in analyzing the Cyclistic historical bike trip data to identify trends.”

Ask
Three questions will guide the future marketing program:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?


## Data Preparation and Processing, SQL

To access Cyclistic’s bike trip data, click [here](https://divvy-tripdata.s3.amazonaws.com/index.html). I downloaded all the zip files for the 2022 calendar year. Each file contained a CSV file that held the bike trip data for that month. For reference, below is the folder for January.

![Screenshot 2023-08-14 223745](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/03249e66-1228-4973-accc-296d08419707)

I then moved each CSV file to another folder in my computer titled “Bike Trip Data 2022” for easy access.

Next, I created a bucket titled bike_691 in Google Cloud Storage since most of the files are too big to upload to BigQuery directly.


![bucket](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/5246edb8-1612-43eb-bbd6-2d5dda3ade02)

Afterwards, I created a table for each file in Big Query titled by month. Once uploaded, I checked the schema of all the tables to ensure that the name and data type of each field were identical.

![Schema](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/fe18ebf5-a100-4783-8cfb-1136a9a1c7fc)

Because all tables have identical schemas, I used a UNION ALL clause to merge all tables into one titled 2022.

````sql
SELECT * 
FROM sodium-hue-391603.Bike_case_study.January
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.February
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.March
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.April
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.May
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.June
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.July
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.August
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.September
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.October
 UNION ALL

Select * 
FROM sodium-hue-391603.Bike_case_study.November
 UNION ALL
 
Select * 
FROM sodium-hue-391603.Bike_case_study.December
````

I then used the following query to create a new table titled 2022 that contained the day of the week, hour, month, and trip duration of each bike rental.

````sql
SELECT 
    *,
    CASE
    WHEN EXTRACT(DAYOFWEEK from started_at) = 1 THEN 'Sunday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 2 THEN 'Monday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 3 THEN 'Tuesday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 4 THEN 'Wednesday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 5 THEN 'Thursday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 6 THEN 'Friday'
    ELSE 'Sunday'
    END AS day_of_week,
    EXTRACT(HOUR from started_at) AS starting_hour,
    EXTRACT(MONTH from started_at) AS month,
    TIMESTAMP_DIFF(ended_at, started_at, second) AS trip_duration
    FROM `sodium-hue-391603.Bike_case_study.2022`
````

Considering that trips with a time duration of zero needed to be excluded from my analysis, I added a WHERE clause at the end of my query. Note that I nested the original query as a subquery for this query because SQL does not allow users to use column aliases (e.g. trip_duration) in the WHERE clause. Using a subquery allowed SQL to determine the trip duration first before determining whether the duration is greater than zero.

````sql
SELECT *
FROM 
  (SELECT 
    *,
    CASE
    WHEN EXTRACT(DAYOFWEEK from started_at) = 1 THEN 'Sunday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 2 THEN 'Monday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 3 THEN 'Tuesday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 4 THEN 'Wednesday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 5 THEN 'Thursday'
    WHEN EXTRACT(DAYOFWEEK from started_at) = 6 THEN 'Friday'
    ELSE 'Sunday'
    END AS day_of_week,
    EXTRACT(HOUR from started_at) AS starting_hour,
    EXTRACT(MONTH from started_at) AS month,
    TIMESTAMP_DIFF(ended_at, started_at, second) AS trip_duration
    FROM `sodium-hue-391603.Bike_case_study.2022`  
  )
  WHERE trip_duration > 0
````

With the data cleaned and properly formatted, I saved the query results to another table. I then clicked the Export button and set the location to the bucket, bike_391. Next, I titled the file, set Export Format as CSV, and set Compression as GZIP to compress the file since it contained over 55,000 rows. I downloaded the file to my computer and uploaded the file to Tableau.

Since I plan on visualizing the top 12 most popular bike stations amongst membership types, I wrote another query that extracted that information along with the latitude and longitude of each bike station. I wrote “casual” in the WHERE clause to extract information about casual riders and “member” for riders who are members.

````sql
SELECT start_station_name, COUNT(start_station_name) AS Count, AVG(start_lat) AS Avg_Start_Lat, AVG(start_lng) AS Avg_Start_Long
FROM `sodium-hue-391603.Bike_case_study.2022_v3` 
WHERE member_casual LIKE 'casual'
GROUP BY start_station_name
ORDER BY COUNT(start_station_name) desc
LIMIT 12
````

Here are the top twelve stations for casual riders…

![top 12 stations for casual rirders](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/639d59e6-a897-42bb-af62-fc3d71e97b88)

…and here for members.

![Top 12 stations for members](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/f569d588-ca8e-4bc7-9b8c-f9fdabf16799)

I saved the results of both queries to CSV files that I will use later for data visualization.

## Data Preparation and Processing, R
I repeated the same process in SQL in R to prepare and process data to create usable CSV files. Feel free to skip to the data visualization portion by clicking [here](#data-visualization-tableau). The purpose of this portion is to demonstrate my skills in R. 


To set up my workspace in RStudio, I installed the Tidyverse, Janitor, and Lubridate packages.

````r
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')
library(tidyverse)
library(janitor)
library(lubridate)
````

Then I uploaded all 12 CSV files to RStudio, one for each month. These are the same files I used in SQL. 

![uploading12csvfiles](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/37d6445e-7033-4ea0-a75e-c81fabc816df)

I then converted each file into a data frame using the read_csv() function.

````r
Jan <- read_csv("/cloud/project/files/202201-divvy-tripdata.csv")
````

I did this for each of the 12 files. I then used the str() function to read the schema of each data set to ensure that each column had the proper data type.

````r
> str(Jan)
'data.frame':	103770 obs. of  13 variables:
 $ ride_id           : chr  "C2F7DD78E82EC875" "A6CF8980A652D272" "BD0F91DFF741C66D" "CBB80ED419105406" ...
 $ rideable_type     : chr  "electric_bike" "electric_bike" "classic_bike" "classic_bike" ...
 $ started_at        : chr  "2022-01-13 11:59:47" "2022-01-10 08:41:56" "2022-01-25 04:53:40" "2022-01-04 00:18:04" ...
 $ ended_at          : chr  "2022-01-13 12:02:44" "2022-01-10 08:46:17" "2022-01-25 04:58:01" "2022-01-04 00:33:00" ...
 $ start_station_name: chr  "Glenwood Ave & Touhy Ave" "Glenwood Ave & Touhy Ave" "Sheffield Ave & Fullerton Ave" "Clark St & Bryn Mawr Ave" ...
 $ start_station_id  : chr  "525" "525" "TA1306000016" "KA1504000151" ...
 $ end_station_name  : chr  "Clark St & Touhy Ave" "Clark St & Touhy Ave" "Greenview Ave & Fullerton Ave" "Paulina St & Montrose Ave" ...
 $ end_station_id    : chr  "RP-007" "RP-007" "TA1307000001" "TA1309000021" ...
 $ start_lat         : num  42 42 41.9 42 41.9 ...
 $ start_lng         : num  -87.7 -87.7 -87.7 -87.7 -87.6 ...
 $ end_lat           : num  42 42 41.9 42 41.9 ...
 $ end_lng           : num  -87.7 -87.7 -87.7 -87.7 -87.6 ...
 $ member_casual     : chr  "casual" "casual" "member" "casual" ...
````

Here I saw that the “started_at” and “ended_at” columns, which indicated the start and end times of each rental, needed to be converted from character (chr) to timestamp (POSIXct).

I ran the code below to do so.

````r
Jan$started_at<-strptime(Jan$started_at, "%Y-%m-%d %H:%M:%S")
Jan$ended_at<-strptime(Jan$ended_at, "%Y-%m-%d %H:%M:%S")
````

When I ran the str() function again, both columns were reformated properly.

````r
$ started_at        :POSIXlt, format: "2022-01-13 11:59:47" "2022-01-10 08:41:56" ...
$ ended_at          :POSIXlt, format: "2022-01-13 12:02:44" "2022-01-10 08:46:17"
````

I checked all 12 CSV files for proper formatting and converted columns to the appropriate data type. Then, I merged all 12 CSV files to a data frame titled “year2022.”

````r
year2022 <- bind_rows(Jan, Feb, Mar, Apr, May, June, July, Aug, Sept, Oct, Nov, Dec)) 
````

I wanted to manipulate the data to observe the behaviors of casual riders and annual members. I first needed to build a table with new columns with relevant data.

To create a column named “day” that tells us the day of the week of each bike rental I used the following command:

````r
year2022$day <- wday(year2022$started_at, label = T, abbr =T)
````

To get the starting hour, I used:

````r
year2022$starting_hour <- format(as.POSIXct(year2022$started_at), '%H')
````

For month:

````r
year2022$month <- format(as.POSIXct(year2022$started_at), '%m')
````

For the duration of each ride:

````r
year2022$trip_duration <- difftime(year2022$ended_at, year2022$started_at, units = "secs")
````

To exclude trips with a duration of zero, I used the following command to create a new table:

````r
finalyear2022 <- year2022[!(year2022$trip_duration=0)]
````

I then saved the data frame to a CSV file that I uploaded later to Tableau.

````r
write_csv(finalyear2022, "finalyear2022.csv")
````

I also wanted to know the most popular stations of casual riders and annual members as well as the latitude and longitude of each station so I could visualize it in Tableau.

First, I needed to create two data frames that sorted the cleaned data by the two membership types — casual riders and annual members.

To create a data frame that shows only information about casual riders, I wrote the following command.

````r
casual_stations<-filter(finalyear2022, member_casual=='casual')
````

Then I created another data frame that listed the most popular stations amongst casual riders and the coordinates of those stations. Note that the tibble only shows the first 3 digits of the latitude and longitude. The actual data frame contains the full coordinates.

````r
casual_stations_with_coodinates <- casual_stations %>% 
  group_by(start_station_name) %>% 
  summarise(
    count = n(), 
    start_lat = mean(start_lat),
    start_long = mean(start_lng)) %>% 
    arrange(desc(count))

 A tibble: 772 × 4
   start_station_name                  count start_lat start_long
   <chr>                               <int>     <dbl>      <dbl>
 1  NA                                317678      41.9      -87.7
 2 "Streeter Dr & Grand Ave"           66353      41.9      -87.6
 3 "Millennium Park"                   33578      41.9      -87.6
 4 "Michigan Ave & Oak St"             29778      41.9      -87.6
 5 "Shedd Aquarium"                    23249      41.9      -87.6
 6 "Theater on the Lake"               21349      41.9      -87.6
 7 "Wells St & Concord Ln"             19888      41.9      -87.6
 8 "Lake Shore Dr & Monroe St"         19616      41.9      -87.6
 9 "Clark St & Lincoln Ave"            17033      41.9      -87.6
10 "Wells St & Elm St"                 16664      41.9      -87.6
````
I exported the data frame to a CSV file.

````r
write_csv(casual_stations_with_coodinates, "casual_stations_with_coodinates.csv")
````

I repeated the same process for annual members by changing the filter and naming conventions of the data frames and files.

With the CSV files created, I could now move to data visualization.

## Data Visualization, Tableau
I uploaded the CSV files in Tableau and made visualizations that allowed the data to sing!

According to the data, members outnumber casual riders by about half a million members.

![pie chart](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/32e97bd5-77e6-4d4e-9629-f9047b1650b5)

Members also consistently rent more bikes than casual riders throughout the year. During the summer months with more tourism in the city, ridership for members and casual riders is close and at the highest. However, ridership amongst casual riders plummets in autumn while ridership amongst members gradually decreases. Ridership increases again in the spring.

![Total Bike Rentals by Month  Bar](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/f5874c94-32f3-492c-8596-b8e785e891cf)

When examining consumer behavior, ridership amongst members is the highest on weekdays presumably from commuting. On the other hand, ridership amongst casual members is at the highest on weekends, even slightly outnumbering ridership amongst members.

![Total Bike Rentals by Day  Bar](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/2849dc77-bdc3-4e2a-ba67-3274d929779a)

It’s fair to assume that members are more likely to bike to commute while casual riders are more likely to bike for pleasure. This assumption is supported by looking at the average ride lengths of each member type by month and day.

![Average Ride Length of Casual and Member Bikers  Bar](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/d671c777-98a2-48d8-ac18-a161a1f21dc6)

![Average Ride Length of  Bike Rides by Day  Bar](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/45e02f05-6637-480c-b75c-628026571af1)


Casual riders bike twice as long as members. For 2022, the average trip for a casual rider is 1748 seconds (~29 minutes), while for members 763 seconds (~13 minutes).

When examining when members start their bike rides, it becomes clearer that they bike to commute. Rentals peak at 8:00-8:59 AM and 5:00-5:59 PM, which correlates to the typical 9 to 5 work schedule.

![Total Bike Rentals on Weekdays line](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/3ded9592-667b-481e-b0e6-bfd268a295b8)

On the weekends, the behavior of casual riders and members is similar. The most popular times for both groups to bike are between 12:00 PM and 5:00 PM, suggesting both groups bike on the weekends for fun. Casual riders slightly outnumber members during these peak times, strengthening the correlation between biking and pleasure for that group.

![Total Bike Rentals on Weekdays (2)](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/629457a6-a37e-459f-914b-d363dda22937)

By looking at the top 12 most popular stations by rider type, we find more evidence casual riders are more likely to ride for pleasure since all stations are located downtown or by the Lakefront Trail. Other stations popular amongst casual riders are located near popular tourist destinations such as Navy Pier, Millenium Park, Shedd Aquarium, and Dusable Harbor.

![Casual](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/954862f9-1b7d-49ae-966f-7b3fc56c33c3)

On the other hand, the most popular stations amongst members are scattered throughout the city and farther away from the Lakefront Trail, suggesting that members are more likely to use bikes for their commute. Popular stations are near the University of Illinois at Chicago (UIC)campus, the University of Chicago (UChicago) campus, and the north and northwest neighborhoods of downtown.

![Members](https://github.com/kenneth-dizon/Cyclistic-Data-Analytics/assets/141383645/79d76600-6b00-444e-9454-32ca4be8effc)


## Recommendations

Casual riders bike for fun while annual members bike to commute. If we want more casual riders to sign up for annual memberships, the marketing campaign should demonstrate to them that biking is not only fun but an effective way to commute. Below are my recommendations.

1. Casual riders are more likely to ride in the summer months and on weekends from 12:00 PM to 5:00 PM. I recommend that the digital marketing campaign launch ads at these times.
2.  Many casual riders use stations near tourist destinations and many annual members use stations near college campuses and neighborhoods with great bike infrastructure such as the ones north and northwest of downtown. We should launch a geo-targeted campaign at these sites.
3. We should also partner with institutions near stations where bike rentals are the highest. I recommend partnering with the University of Illinois at Chicago, the University of Chicago, and popular tourist destinations to give discounted rates to anyone who wants to sign up for annual memberships.
4. Launching monthly memberships or a summer pass would be great to increase ridership amongst casual riders. Since they’re more likely to ride only in the summer, they may be afraid to commit to an annual membership. Once the summer is over, we could target them with a special promotion that allows them to pay for an annual membership at a discounted rate.



















