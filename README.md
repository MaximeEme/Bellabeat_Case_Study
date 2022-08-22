# Bellabeat_Case_Study

# INTRODUCTION TO THE CAPSTONE PROJECT - BELLABEAT

This project is the capstone project of the Google Data Analytics Professional Certificate. It will follow the 6 steps of Data analysis learned during the online classes.

## Step 1 : Ask phase

1.  **Bellabeat**

Bellabeat is a high-tech company that manufactures health-focused smart product since 2013 and has known a rapdid growth and quickly positioned itself as a tech-driven wellness company for women.

2. **Business task**

Analyse smart data usage from fitbit users to discover trends and apply them to the Bellabeat marketing team.

3. **Key stakeholders**

**Urška Sršen**: Bellabeat's cofounder and Chief Creative Officier.
**Sando Mur**: Mathematician and Bellabeat's cofounder; key member of the Bellabeat executive team.
**Bellabeat marketing analytics team**

## Step 2 : Prepare

1.  **Information on Data**

The data has been shared publicly on Kaggle : Fitbit fitness tracker data and stored in 18 csv files. The files were generated by respondents through a survey distributed by Amazon between 12 March 2016 to 12 May 2016. The data includes physical activity recorded in minutes, heart rate, sleep monitoring, daily activity and steps.

2.  **Limitations of Data**

The data collected is from 2016, hence it may not be relevant anymore to draw satisfactory conclusions on health, calories consumption, sleeping habits or activity. The sample size is only 30 female and is not representative of the female population. The data has been collected through a survey , hence I won't be able to determine the integrity or accuracy of the data.

3.  **Is the data ROCC ?**

A good data source is ROCCC which stands for Reliable, Original, Comprehensive, Current, and Cited.

Reliable - LOW - Only 30 respondents. Original - LOW - It is third-party data. Comprehensive - MED - It matches most of Bellabeat product's parameters. Current - LOW - The data is from 2016. Cited - LOW - It is third-party data.

## Step 3 : Process

I used R to conduct my analysis due to the accessibility of the program, amount of data and being able to create the visualisations. 

1.  **Installing packages and loading libraries**

```{r}
install.packages("tidyverse")
install.packages("janitor")
install.packages("ggpubr")
install.packages("here")
install.packages("skimr")
install.packages("ggrepel")
```

```{r}
library(tidyverse)
library(janitor)
library(ggpubr)
library(here)
library(skimr)
library(ggrepel)
library(lubridate)
library(dplyr)
```

I choose these packages to help me with my analysis. 

2. **Importing datasets**

For my analysis, I decided to work on three datasets and imported them into R to answer my business task. 

> 
- daily_activity_merged
- hourlySteps_merged
- sleepday_merged 

```{r}
dailyActivity_merged <- read_csv("Fitbit data/dailyActivity_merged.csv")
hourlySteps_merged <- read_csv("Fitbit data/hourlySteps_merged.csv")
sleepDay_merged <- read_csv("Fitbit data/sleepDay_merged.csv")
```

3. **Preview the datasets**

I previewed the datasets and checked the summary of every columns. 

```{r}
head(dailyActivity_merged)
str(dailyActivity_merged)

head(sleepDay_merged)
str(sleepDay_merged)

head(hourlySteps_merged)
str(hourlySteps_merged)
```

4. **Data cleaning and formatting**

I proceeded to look for any errors or inconsistencies. 

- Numbers of users 
```{r}
n_unique(dailyActivity_merged$Id)
n_unique(sleepDay_merged$Id)
n_unique(hourlySteps_merged$Id)
```
>
- 33
- 24
- 33

- Duplicates
```{r}
sum(duplicated(dailyActivity_merged))
sum(duplicated(hourlySteps_merged))
sum(duplicated(sleepDay_merged))
```
> 
- 0
- 0
- 3

5. **Removing duplicates and N/A**

```{r}
sleepDay_merged <- sleepDay_merged %>% distinct() %>% drop_na()
```

I verified that duplicates had been removed . 
```{r}
sum(duplicated(sleepDay_merged))
```
> 
- 0

6. **Cleaning names and renaming columns**

I wanted to make sure that names were in the right syntax
and format in lower case. 

```{r}
clean_names(dailyActivity_merged)
dailyActivity_merged <- rename_with(dailyActivity_merged, tolower)
clean_names(hourlySteps_merged)
hourlySteps_merged <- rename_with(hourlySteps_merged, tolower)
clean_names(sleepDay_merged)
sleepDay_merged <- rename_with(sleepDay_merged, tolower)
```

7. **Date and time columns consistency**

```{r}
dailyActivity_merged <- dailyActivity_merged %>% rename(date = activitydate) %>% mutate(date = as_date(date, format = "%m/%d/%y"))

sleepDay_merged <- sleepDay_merged %>% rename(date = sleepday) %>% mutate(date = as_date(date, format = "%m/%d/%Y %H:%M:%S"))
```

I checked the cleaned datasets. 

```{r}
head(dailyActivity_merged)
head(sleepDay_merged)
```

```{r}
hourlySteps_merged <- hourlySteps_merged %>% rename(date_time = activityhour)%>% mutate(date_time = mdy_hms(date_time))
```

I checked that the date time columns was in the correct format. 

```{r}
head(hourlySteps_merged)
```

8. **Merging datasets**

I merged dailyActivity_merged and sleepDay_merged. 

```{r}
dailyActivity_sleep <- merge(dailyActivity_merged, sleepDay_merged, by=c("id", "date"))
```

## Step 4 : Analyse 

I analyzed trends of the Fitbit users and solve my business task. 

1. **Determine the type of users per activity level**

I didn't have demographic variables from the sample and I wanted to determine the type of users from the dataset by their activity level. 

I followed the classification of this site [link](https://observatoireprevention.org/2018/09/17/marcher-10-000-pas-par-jour/). I categorized the users as follow : 

- Sedentary : < 5000 steps a day 
- Lightly active : 5000 to 7499 steps a day 
- Fairly active :7500 to 9999 steps a day
- Very active : >10000 

2. **Daily steps average per users**

```{r}
daily_average <- dailyActivity_sleep %>% group_by(id)%>% summarise(mean_daily_steps = mean(totalsteps),mean_daily_calories = mean(calories),mean_daily_sleep = mean(totalminutesasleep))
```

I previewed the new dataframe.

```{r}
head(daily_average)
```

3. **Classification of users by daily average steps**

```{r}
user_type <- daily_average %>% mutate(user_type=case_when(mean_daily_steps < 5000 ~ "sedentary", mean_daily_steps >= 5000 & mean_daily_steps < 7499 ~ "lightly active", mean_daily_steps >= 7500 & mean_daily_steps < 9999 ~ "fairly active", mean_daily_steps >= 10000 ~ "very active" ))
```

I previewed the new dataframe.
```{r}
head(user_type)
```

4. **Calculation of steps and minutes asleep through the week**

I wanted to discover which day of the week, the users slept  more  and were the most active. I also verified that the users walked the recommended amount of steps and sleep. 

```{r}
weekday_steps_sleep <- dailyActivity_sleep %>% mutate(weekday = weekdays(date))

weekday_steps_sleep$weekday <- ordered(weekday_steps_sleep$weekday, levels=c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

weekday_steps_sleep <- weekday_steps_sleep %>% group_by(weekday) %>% summarize(daily_steps = mean(totalsteps), daily_sleep = mean(totalminutesasleep))

head(weekday_steps_sleep)
```

I plotted two graphs 'Daily steps through weekday' and 'Minutes asleep per weekday' with ggplot2. 

See graph 'Daily steps through weekday' and 'Minutes asleep per weekday'. 

As per the plots, I can confirm that the users walk a minimum of 7500 steps a day except on Sunday and they don't sleep the recommended 8 hours of sleep per day. 

5. **Hourly steps per 24h**

I wanted to know at what time, the users were the most active during the day. 

I used the hourlysteps dataframe and separated the date_time column. 

```{r}
hourly_steps <- hourlySteps_merged %>% separate(date_time, into = c("date", "time"), sep = " ") %>% mutate(date = ymd(date))

head(hourly_steps)
```

Then, I plotted the results. 

See graph [Steps per 24h](https://github.com/MaximeEme/Bellabeat_Case_Study/blob/main/B5C7887B-98CA-4D3F-9E9D-40B6B8A0BEC9.png)

I can confirm that the users were the most active during the period from 8am to 7pm. Lunch time having the most steps walked with evening. 

6. **Correlation**

I wanted to find out if there were some correlations between the dataset. 

See graph [Daily steps vs calories burned](https://github.com/MaximeEme/Bellabeat_Case_Study/blob/main/17270CC6-0A55-449E-A15C-B363BE37CAE6.png)

There is a positive correlation between the amount of steps walked and the calories burned. 

7. **Usage of smart device**

I wanted to understand the usage of the smart device by the users. 

I classified the users per the number of days they used their smart device on a 31 days interval. 

high use - users who use their device between 21 and 31 days.
moderate use - users who use their device between 10 and 20 days.
low use - users who use their device between 1 and 10 days.

```{r}
daily_use <- dailyActivity_sleep %>% group_by(id) %>% summarise(days_used=sum(n())) %>% mutate(usage = case_when(days_used >= 1 & days_used <= 10 ~ "low use", days_used >= 11 & days_used <= 20 ~ "moderate use", days_used >= 21 & days_used <= 31 ~ "high use",))

head(daily_use)
```

I created a percentage dataframe of the usage of the users. 

```{r}
daily_use_percent <- daily_use %>% group_by(usage) %>% summarise(total = n()) %>% mutate(totals = sum(total)) %>% group_by(usage) %>% summarise(total_percent = total/ totals) %>% mutate(labels = scales::percent(total_percent))

head(daily_use_percent)
```

I plotted the results.

See graph [Usage of device per day](https://github.com/MaximeEme/Bellabeat_Case_Study/blob/main/AA2BFA47-0212-4E50-A278-C0F8E05E1029.png)

From the results, I can say that 50% of the sample use frequently their device. 12% use their device from 12 to 21 days and 38% rarely use their device. 

8. **Usage of smart device in minutes**

I wanted to be more precise and see how many minutes the users used their smart device. I merged two dataframes (daily use and daily activity). 

```{r}
daily_use_merged <- merge(daily_use, dailyActivity_merged, by = c("id"))
head(daily_use_merged)
```

I created a new dataframe called "minutes worn" for the total amount of minutes the users worn their smart device and created three new categories : 
- All day 
- More than half a day 
- Less than half a day 

```{r}
minutes_worn <- daily_use_merged %>% mutate(total_minutes_worn = veryactiveminutes+fairlyactiveminutes+lightlyactiveminutes+sedentaryminutes) %>% mutate(percent_minutes_worn = (total_minutes_worn/1440)*100) %>% mutate(worn = case_when( percent_minutes_worn == 100 ~ "All day" , percent_minutes_worn < 100 & percent_minutes_worn >= 50 ~ "More than half a day", percent_minutes_worn < 50 & percent_minutes_worn > 0 ~ "Less than half a day"))
head(minutes_worn)
```

Then, I created three new dataframes based on the usage of the smart device plus one that summed it up
: 
- High use, users who use their device between 21 and 31 days.
- Moderate use, users who use their device between 10 and 20 days.
- Low use, users who use their device between 1 and 10 days.

```{r}
minutes_worn_percent <-  minutes_worn %>% group_by(worn) %>% summarise(total = n()) %>% mutate(totals = sum(total)) %>% group_by(worn) %>% summarise(total_percent = total/ totals) %>% mutate(labels = scales::percent(total_percent))
```
```{r}
minutes_worn_highuse <-  minutes_worn %>% filter(usage == "high use") %>% group_by(worn) %>% summarise(total = n()) %>% mutate(totals = sum(total)) %>% group_by(worn) %>% summarise(total_percent = total/ totals) %>% mutate(labels = scales::percent(total_percent))

minutes_worn_moderateuse <-  minutes_worn %>% filter(usage == "moderate use") %>% group_by(worn) %>% summarise(total = n()) %>% mutate(totals = sum(total)) %>% group_by(worn) %>% summarise(total_percent = total/ totals) %>% mutate(labels = scales::percent(total_percent))


minutes_worn_lowuse <-  minutes_worn %>% filter(usage == "low use") %>% group_by(worn) %>% summarise(total = n()) %>% mutate(totals = sum(total)) %>% group_by(worn) %>% summarise(total_percent = total/ totals) %>% mutate(labels = scales::percent(total_percent))
```

Once, I had created the dataframes according to my new categories, I decided to present them in a treemap plot. 

See graphs "Time worn per day", [High use - Users](https://github.com/MaximeEme/Bellabeat_Case_Study/blob/main/B7C7F0F4-9989-48B3-92D2-4AB953271FC2.png) "Low use - Users" and "Moderate use - Users". 

According to the plots, I can verify that 36% of the users in total wear the device all day, 60% more than half a day and 4% less than a day. 

If we want to be even more precise, I can say that in the High use category, 89% of the users, wear their device more than half a day and only just 6%, all day. The low use category are the users that wear the device, all day. 
## Step 5 : Share phase 

See the graphs throughout this presentation 

## Step 6 : Act phase 

I would encourage Bellabeat to continue their trend searching and to extend their datasets to be able to find more noticeable trends and have better results. 

1/ After I classified the users in four categories (sedentary, lightly active, fairly active and very active), I noticed than the average user was very active (i.e walk more that 7500 steps). We can encourage that behavior by sending notifications and alarm if the user hasn't reached its goal.

2/ The results I got on sleep from the dataset could be improve by setting alarms on desired amount of sleep and maybe notification on the health benefits of a good night of sleep. 

3/ Setting a reward system on reaching daily, monthly and quarterly goals. The rewards could go from a new background on the app to free-month subscription. 
