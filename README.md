# Cyclistic-Bike-share-project

# ASK
You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share
company in Chicago. The director of marketing believes the company’s future success
depends on maximizing the number of annual memberships. Therefore, your team wants to
understand how casual riders and annual members use Cyclistic bikes differently. From these
insights, your team will design a new marketing strategy to convert casual riders into annual
members. But first, Cyclistic executives must approve your recommendations, so they must be
backed up with compelling data insights and professional data visualizations

## The business problem 
How do annual members and casual riders use Cyclistic bikes differently?


# PREPARE
Cyclistic’s historical trip data to analyze and identify trends. Download the previous 12
months of Cyclistic trip data here <https://divvy-tripdata.s3.amazonaws.com/index.html>. (Note: The datasets have a different name because Cyclistic is a fictional company. For the purposes of this case study, the datasets are appropriate and
will enable you to answer the business questions.

## Tools used for project 
R Studio and Tableau


# PROCESS
Remove columns we wont use
```{r}
tripdata_jan23_mar23 <- tripdata_jan23_mar23 %>%
  select(-c(start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng,))
```

Review new table
```{r}
colnames(tripdata_jan23_mar23)
nrow(tripdata_jan23_mar23)
head(tripdata_jan23_mar23)
str(tripdata_jan23_mar23)
```

New columns for date, month, day and year
```{r}
tripdata_jan23_mar23$date <- as.Date(tripdata_jan23_mar23$started_at) 
tripdata_jan23_mar23$month <- format(as.Date(tripdata_jan23_mar23$date), "%m")
tripdata_jan23_mar23$day <- format(as.Date(tripdata_jan23_mar23$date), "%d")
tripdata_jan23_mar23$year <- format(as.Date(tripdata_jan23_mar23$date), "%Y")
tripdata_jan23_mar23$day_of_week <- format(as.Date(tripdata_jan23_mar23$date), "%A")
```

Add ride_length
```{r}
tripdata_jan23_mar23$ride_length <- difftime(tripdata_jan23_mar23$ended_at,tripdata_jan23_mar23$started_at)
```

Convert ride_length data type to numeric
```{r}
is.factor(tripdata_jan23_mar23$ride_length)
tripdata_jan23_mar23$ride_length <- as.numeric(as.character(tripdata_jan23_mar23$ride_length))
is.numeric(tripdata_jan23_mar23$ride_length)
```

Arrange days of week
```{r}
tripdata_jan23_mar23$day_of_week <- ordered(tripdata_jan23_mar23$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```

# ANALYZE
Compare members and casual users
```{r}
aggregate(tripdata_jan23_mar23$ride_length ~ tripdata_jan23_mar23$member_casual, FUN = mean)
aggregate(tripdata_jan23_mar23$ride_length ~ tripdata_jan23_mar23$member_casual, FUN = median)
aggregate(tripdata_jan23_mar23$ride_length ~ tripdata_jan23_mar23$member_casual, FUN = max)
aggregate(tripdata_jan23_mar23$ride_length ~ tripdata_jan23_mar23$member_casual, FUN = min)
```

Average ride time by each day for members vs casual users
```{r}
aggregate(tripdata_jan23_mar23$ride_length ~ tripdata_jan23_mar23$member_casual + tripdata_jan23_mar23$day_of_week, FUN = mean)
```

Analyze ridership data by type and weekday
```{r}
tripdata_jan23_mar23 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>%  
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()							
            ,average_duration = mean(ride_length)) %>% 		
  arrange(member_casual, weekday)	
```

Visualize the number of rides by rider type
```{r}
tripdata_jan23_mar23 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge") + labs(title = "Number of rides during the week")
```
![Number of rides vs weekday](https://github.com/StephanieAfiaAduBoahen/Bellabeat-Projects/assets/158788793/3d339bdf-eff6-4822-9da4-746d55e56f73)

Visualization for average duration
```{r}
tripdata_jan23_mar23 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge") + labs(title = "Average duration")
```
![Average duration](https://github.com/StephanieAfiaAduBoahen/Bellabeat-Projects/assets/158788793/c0724d66-7545-42db-85e7-94923cbecc77)

Visualize Bikes rented
```{r}
tripdata_jan23_mar23 %>% 
  ggplot(aes(x = rideable_type, fill = member_casual)) + geom_bar(position = "dodge") +
  labs(x = "Type of bike", y = "Number of rentals", title = "Rented bikes", fill = "Type of membership")
ggsave("Rented bikes.png")
```
![Rented bikes](https://github.com/StephanieAfiaAduBoahen/Bellabeat-Projects/assets/158788793/d7150d54-0c2c-4dd9-9ec7-ddd85bd82941)


# SHARE
Bike share visualization with Tableau


# ACT
## Observation and conclusion:
1. Members total number of rides during the week is more than casual users. 
2. On weekends, both casual riders and members typically ride for longer periods of time.
3. Compared to casual riders, members typically use more types of bikes, including electric, docked, and classic bikes. Casual riders prefer electric bikes to classic bikes.

## Recommendations
1.	Since casual members average duration on bikes is higher, increase the rate for longer duration and introduce membership plans that makes riding longer duration appealing to casual riders.
2.	Create a new membership plan where electric bikes are more accessible and affordable to members.
3.	Advertise discounts for first 3 months of sign up. 
