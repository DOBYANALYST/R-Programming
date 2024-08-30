# R-Programming
Coursera Case Study on Divvy_Trips

install.packages("tidyverse")
install.packages("lubridate")
install.packages("ggplot2")

library(tidyverse)
library(lubridate)
library(ggplot2)
library(readr)

library(readxl)
Divvy_Trips_2019_Q1 <- read_excel("Divvy_Trips_2019_Q1.xlsx")
Divvy_Trips_2020_Q1 <- read_excel("Divvy_Trips_2020_Q1.xlsx")

colnames(Divvy_Trips_2019_Q1)
colnames(Divvy_Trips_2020_Q1)

(Divvy_Trips_2019_Q1 <-rename(Divvy_Trips_2019_Q1
                              ,ride_id=...1
                              ,rideable_type = bikeid
                              ,started_at = start_time
                              ,ended_at = end_time
                              ,start_station_name = from_station_name
                              ,start_station_id = from_station_id
                              ,end_station_name = to_station_name
                              ,end_station_id = to_station_id
                              ,member_casual = usertype
                               ))
colnames(Divvy_Trips_2019_Q1)
colnames(Divvy_Trips_2020_Q1)

str(Divvy_Trips_2019_Q1)
str(Divvy_Trips_2020_Q1)

Divvy_Trips_2019_Q1<- mutate(Divvy_Trips_2019_Q1,ride_id=as.character(ride_id)
                            ,rideable_type=as.character(rideable_type))

all_trips <- bind_rows(Divvy_Trips_2019_Q1,Divvy_Trips_2020_Q1)

view(all_trips)

all_trips<-all_trips%>%
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender, tripduration))

view(all_trips)

all_trips<-all_trips %>%
  mutate(member_casual=recode(member_casual
                              ,"Subscriber"="member"
                              ,"Customer"="casual"))
summary(all_trips)

str(all_trips)

head(all_trips)

table(all_trips$member_casual)


all_trips$date <- as.Date(all_trips$started_at) 
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")

view(all_trips)

all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)

str(all_trips)

is.factor(all_trips$ride_length)

all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))

is.numeric(all_trips$ride_length)

str(all_trips)

all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR"| all_trips$ride_length < 0), ]

view(all_trips_v2)

summary(all_trips_v2$ride_length)

mean_ride_length <- aggregate(ride_length ~ member_casual, data = all_trips_v2, FUN = mean)
median_ride_length <- aggregate(ride_length ~ member_casual, data = all_trips_v2, FUN = median)
max_ride_length <- aggregate(ride_length ~ member_casual, data = all_trips_v2, FUN = max)
min_ride_length <- aggregate(ride_length ~ member_casual, data = all_trips_v2, FUN = min)

mean_ride_length
median_ride_length
max_ride_length
min_ride_length

avg_ride_time <- aggregate(ride_length ~ member_casual + day_of_week, data = all_trips_v2, FUN = mean)

avg_ride_time

all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))

#rerun the avg_ride_time

all_trips_v2 <- mutate(all_trips_v2, weekday = wday(started_at, label = TRUE))
summary_stats <- all_trips_v2 %>%
  group_by(member_casual, weekday) %>%
summarise(number_of_rides = n()
,average_duration = mean(ride_length)) %>% 
arrange(member_casual, weekday)  
head(all_trips_v2)
view(all_trips_v2)

ggplot(data=summary_stats, aes(x = weekday, y = number_of_rides, fill = member_casual))+
  geom_col(position = "dodge")


