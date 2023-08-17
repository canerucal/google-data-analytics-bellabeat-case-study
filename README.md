# Case Study: Bellabeat Fitness Data Analysis

**Author**: Caner Uçal

**Date**: 07.08.2023


*This case study includes the six steps of data analysis process which suggested by Google:*

### [Scenario](#scenario-1)

### [1-Ask](#1-ask-1)

### [2-Prepare](#2-prepare-1)

### [3-Process](#3-process-1)

### [4-Analyze and Share](#4-analyze-and-share-1)

### [5-Act](#5-act-1)

# Scenario

You are a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused products for women. 

Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. 

Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. 

You have been asked to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. 

The insights you discover will then help guide marketing strategy for the company. 

You will present your analysis to the Bellabeat executive team along with your high-level recommendations for Bellabeat’s marketing strategy.

# 1-Ask
Sršen asks you to analyze smart device usage data in order to gain insight into how consumers use non-Bellabeat smart devices. 

She then wants you to select one Bellabeat product to apply these insights to in your presentation. These questions will guide your analysis:
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

#### The main stakeholders here are:

- Urška Sršen, Bellabeat’s co-founder and Chief Creative Officer
- Sando Mur, Mathematician and Bellabeat’s cofounder 
- Bellabeat marketing analytics team

# 2-Prepare
[FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) is used from Kaggle and it's open source. Data is collected from real personal data without name.

Data has 18 CSV files. These are:
- dailyActivity_merged
- dailyCalories_merged
- dailyIntensities_merged
- dailySteps_merged
- heartrate_seconds_merged
- hourlyCalories_merged
- hourlyIntensities_merged
- hourlySteps_merged
- minuteCaloriesNarrow_merged
- minuteCaloriesWide_merged
- minuteIntensitiesNarrow_merged
- minuteIntensitiesWide_merged
- minuteMETsNarrow_merged
- minuteSleep_merged
- minuteStepsNarrow_merged
- minuteStepsWide_merged
- sleepDay_merged
- weightLogInfo_merged

Unfortunately, we have 30 users data and only for 2 months long range. We may have sampling bias because of data size.

# 3-Process
In this data analysis project, R programming language is used, because of its accessibility, data manipulation tools/libraries and data visualization capabilities.

```r
library(ggpubr)
library(tidyverse)
library(here)
library(skimr)
library(janitor)
library(lubridate)
library(ggrepel)
```

## 3.1.Importing Datasets
Knowing the datasets we have, we will upload the datasets that will help us answer our business task. On our analysis we will focus on the following datasets:

- Daily_activity
- Daily_sleep
- Hourly_steps

Due to the the small sample we won't consider for this analysis Weight (8 Users) and heart rate (7 users)

```r
daily_activity <- read_csv(file= "../input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")

daily_sleep <- read_csv(file= "../input/fitbit/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")

hourly_steps <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/hourlySteps_merged.csv")
```

## 3.2.Preview dataset
Let's check the selected dataframes.

```
head(daily_activity)
str(daily_activity)

head(daily_sleep)
str(daily_sleep)

head(hourly_steps)
str(hourly_steps)
```

## 3.3.Data cleaning & formatting
We will know more about data types, and check whether if there is any error or not.

### 3.3.1.Checking user numbers
```
n_unique(daily_activity$Id)
n_unique(daily_sleep$Id)
n_unique(hourly_steps$Id)
```
33

24

33

### 3.3.2.Checking duplicates
```
sum(duplicated(daily_activity))
sum(duplicated(daily_sleep))
sum(duplicated(hourly_steps))
```
0

3

0

### 3.3.3.Remove N/A and duplicates
Removing the duplicate data and N/A data.
```
daily_activity <- daily_activity %>%
  distinct() %>%
  drop_na()

daily_sleep <- daily_sleep %>%
  distinct() %>%
  drop_na()

hourly_steps <- hourly_steps %>%
  distinct() %>%
  drop_na()
```

Let's verify that there are no duplicate data.

```
sum(duplicated(daily_sleep))
```

### 3.3.4.Rename and clean columns
We want to ensure that column names are using right syntax and same format in all datasets since we will merge them later on. We are changing the format of all columns to lower case.

```
clean_names(daily_activity)
daily_activity<- rename_with(daily_activity, tolower)
clean_names(daily_sleep)
daily_sleep <- rename_with(daily_sleep, tolower)
clean_names(hourly_steps)
hourly_steps <- rename_with(hourly_steps, tolower)
```

### 3.3.5.Consistency for date and time
Now that we have verified our column names and change them to lower case, we will focus on cleaning date-time format for daily_activity and daily_sleep since we will merge both data frames. Since we can disregard the time on daily_sleep data frame we are using as_date instead as as_datetime.

```
daily_activity <- daily_activity %>%
  rename(date = activitydate) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y"))

daily_sleep <- daily_sleep %>%
  rename(date = sleepday) %>%
  mutate(date = as_date(date,format ="%m/%d/%Y %I:%M:%S %p" , tz=Sys.timezone()))
```

Let's check cleaned dataset.

```
head(daily_activity)
head(daily_sleep)
```

For our hourly_steps dataset we will convert date string to date-time.

```
hourly_steps<- hourly_steps %>% 
  rename(date_time = activityhour) %>% 
  mutate(date_time = as.POSIXct(date_time,format ="%m/%d/%Y %I:%M:%S %p" , tz=Sys.timezone()))

head(hourly_steps)
```

### 3.4.Merging Datasets
We will merge daily_activity and daily_sleep to see any correlation between variables by using id and date as their primary keys.

```
daily_activity_sleep <- merge(daily_activity, daily_sleep, by=c ("id", "date"))
glimpse(daily_activity_sleep)
```

# 4-Analyze and Share
## 4.1.Type of users
Since we don't have any demographic variables from our sample we want to determine the type of users with the data we have. We can classify the users by activity considering the daily amount of steps. We can categorize users as follows:

    Sedentary - Less than 5000 steps a day.
    Lightly active - Between 5000 and 7499 steps a day.
    Fairly active - Between 7500 and 9999 steps a day.
    Very active - More than 10000 steps a day.

Classification has been made per the following article https://www.10000steps.org.au/articles/counting-steps/

First we will calculate the daily steps average by user.

```
daily_average <- daily_activity_sleep %>%
  group_by(id) %>%
  summarise (mean_daily_steps = mean(totalsteps), mean_daily_calories = mean(calories), mean_daily_sleep = mean(totalminutesasleep))

head(daily_average)
```

```
user_type <- daily_average %>%
  mutate(user_type = case_when(
    mean_daily_steps < 5000 ~ "sedentary",
    mean_daily_steps >= 5000 & mean_daily_steps < 7499 ~ "lightly active", 
    mean_daily_steps >= 7500 & mean_daily_steps < 9999 ~ "fairly active", 
    mean_daily_steps >= 10000 ~ "very active"
  ))

head(user_type)
```

```
user_type_percent <- user_type %>%
  group_by(user_type) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(user_type) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))

user_type_percent$user_type <- factor(user_type_percent$user_type , levels = c("very active", "fairly active", "lightly active", "sedentary"))


head(user_type_percent)

```

```
user_type_percent %>%
  ggplot(aes(x="",y=total_percent, fill=user_type)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold")) +
  scale_fill_manual(values = c("#85e085","#e6e600", "#ffd480", "#ff8080")) +
  geom_text(aes(label = labels),
            position = position_stack(vjust = 0.5))+
  labs(title="User type distribution")
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___31_0.png" width="400" alt="image">


## 4.2.Steps and minutes asleep per weekday

We want to know now what days of the week are the users more active and also what days of the week users sleep more. We will also verify if the users walk the recommended amount of steps and have the recommended amount of sleep.

Below we are calculating the weekdays based on our column date. We are also calculating the average steps walked and minutes sleeped by weekday.

```
weekday_steps_sleep <- daily_activity_sleep %>%
  mutate(weekday = weekdays(date))

weekday_steps_sleep$weekday <-ordered(weekday_steps_sleep$weekday, levels=c("Monday", "Tuesday", "Wednesday", "Thursday",
"Friday", "Saturday", "Sunday"))

 weekday_steps_sleep <-weekday_steps_sleep%>%
  group_by(weekday) %>%
  summarize (daily_steps = mean(totalsteps), daily_sleep = mean(totalminutesasleep))

head(weekday_steps_sleep)
```

```
ggarrange(
    ggplot(weekday_steps_sleep) +
      geom_col(aes(weekday, daily_steps), fill = "#006699") +
      geom_hline(yintercept = 7500) +
      labs(title = "Daily steps per weekday", x= "", y = "") +
      theme(axis.text.x = element_text(angle = 45,vjust = 0.5, hjust = 1)),
    ggplot(weekday_steps_sleep, aes(weekday, daily_sleep)) +
      geom_col(fill = "#85e0e0") +
      geom_hline(yintercept = 480) +
      labs(title = "Minutes asleep per weekday", x= "", y = "") +
      theme(axis.text.x = element_text(angle = 45,vjust = 0.5, hjust = 1))
  )
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___34_0.png" width="400" alt="image">



In the graphs above we can determine the following:

- Users walk daily the recommended amount of steps of 7500 besides Sunday's.

- Users don't sleep the recommended amount of minutes/ hours - 8 hours.

## 4.3.Type of users
```
hourly_steps <- hourly_steps %>%
  separate(date_time, into = c("date", "time"), sep= " ") %>%
  mutate(date = ymd(date)) 
  
head(hourly_steps)
```

```
hourly_steps %>%
  group_by(time) %>%
  summarize(average_steps = mean(steptotal)) %>%
  ggplot() +
  geom_col(mapping = aes(x=time, y = average_steps, fill = average_steps)) + 
  labs(title = "Hourly steps throughout the day", x="", y="") + 
  scale_fill_gradient(low = "green", high = "red")+
  theme(axis.text.x = element_text(angle = 90))
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___38_0.png" width="400" alt="image">

## 4.4.Correlations
We will now determine if there is any correlation between different variables:

- Daily steps and daily sleep
- Daily steps and calories

```
ggarrange(
ggplot(daily_activity_sleep, aes(x=totalsteps, y=totalminutesasleep))+
  geom_jitter() +
  geom_smooth(color = "red") + 
  labs(title = "Daily steps vs Minutes asleep", x = "Daily steps", y= "Minutes asleep") +
   theme(panel.background = element_blank(),
        plot.title = element_text( size=14)), 
ggplot(daily_activity_sleep, aes(x=totalsteps, y=calories))+
  geom_jitter() +
  geom_smooth(color = "red") + 
  labs(title = "Daily steps vs Calories", x = "Daily steps", y= "Calories") +
   theme(panel.background = element_blank(),
        plot.title = element_text( size=14))
)
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___40_1.png" width="400" alt="image">



Per our plots:

- There is no correlation between daily activity level based on steps and the amount of minutes users sleep a day.

- Otherwise we can see a positive correlation between steps and calories burned. As assumed the more steps walked the more calories may be burned.

## 4.5.Use of smart device
### 4.5.1.Days used smart device
Having examined certain patterns in activity, sleep, and calories expended, we aim to assess the frequency of device usage among the participants in our dataset. This will enable us to devise our marketing strategy and identify which functionalities could enhance the utility of smart devices.

We will compute the count of users who engage with their smart device on a daily basis, segmenting our dataset into three tiers within a 31-day time span:

- High usage: Users employing their device for 21 to 31 days.
- Moderate usage: Users utilizing their device for 10 to 20 days.
- Low usage: Users engaging with their device for 1 to 10 days.

Initially, we will generate a fresh data frame by grouping data by user Id, determining the number of days of device usage, and appending a new column that reflects the aforementioned categorization.

```
daily_use <- daily_activity_sleep %>%
  group_by(id) %>%
  summarize(days_used=sum(n())) %>%
  mutate(usage = case_when(
    days_used >= 1 & days_used <= 10 ~ "low use",
    days_used >= 11 & days_used <= 20 ~ "moderate use", 
    days_used >= 21 & days_used <= 31 ~ "high use", 
  ))
  
head(daily_use)
```

```
daily_use_percent <- daily_use %>%
  group_by(usage) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(usage) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))

daily_use_percent$usage <- factor(daily_use_percent$usage, levels = c("high use", "moderate use", "low use"))

head(daily_use_percent)
```

```
daily_use_percent %>%
  ggplot(aes(x="",y=total_percent, fill=usage)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold")) +
  geom_text(aes(label = labels),
            position = position_stack(vjust = 0.5))+
  scale_fill_manual(values = c("#006633","#00e673","#80ffbf"),
                    labels = c("High use - 21 to 31 days",
                                 "Moderate use - 11 to 20 days",
                                 "Low use - 1 to 10 days"))+
  labs(title="Daily use of smart device")
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___46_0.png" width="400" alt="image">

Half of the users in our study use their device often, ranging from 21 to 31 days.
Around 12% of users use the device for 11 to 20 days.
Notably, 38% of participants show low interaction with their device.


### 4.5.2.Time used smart device
To provide greater precision, our aim is to determine the daily duration of device usage in minutes by users. To achieve this, we will combine the generated daily_use data frame with daily_activity, enabling us to filter outcomes based on daily device usage.

```
daily_use_merged <- merge(daily_activity, daily_use, by=c ("id"))
head(daily_use_merged)
```

We should generate a fresh data frame that computes the total number of minutes users had the device on each day. This will involve establishing three distinct categories:

- All day - denoting when the device was worn throughout the entire day.
- More than half day - indicating instances where the device was worn for over half of the day.
- Less than half day - signifying occasions when the device was worn for less than half of the day.

```
minutes_worn <- daily_use_merged %>% 
  mutate(total_minutes_worn = veryactiveminutes+fairlyactiveminutes+lightlyactiveminutes+sedentaryminutes)%>%
  mutate (percent_minutes_worn = (total_minutes_worn/1440)*100) %>%
  mutate (worn = case_when(
    percent_minutes_worn == 100 ~ "All day",
    percent_minutes_worn < 100 & percent_minutes_worn >= 50~ "More than half day", 
    percent_minutes_worn < 50 & percent_minutes_worn > 0 ~ "Less than half day"
  ))

head(minutes_worn)
```

Just as we've done previously, for enhanced result visualization, we will construct new data frames. In this scenario, we will establish four distinct data frames, which will later be organized into a single visualization.

- The initial data frame will present the total user count and compute the percentage of device usage time, factoring in the three designated categories.

- The remaining three data frames will be sorted based on the daily user categories, thus allowing us to observe disparities in both daily and time-based device usage.'

```
minutes_worn_percent<- minutes_worn%>%
  group_by(worn) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(worn) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))


minutes_worn_highuse <- minutes_worn%>%
  filter (usage == "high use")%>%
  group_by(worn) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(worn) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))

minutes_worn_moduse <- minutes_worn%>%
  filter(usage == "moderate use") %>%
  group_by(worn) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(worn) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))

minutes_worn_lowuse <- minutes_worn%>%
  filter (usage == "low use") %>%
  group_by(worn) %>%
  summarise(total = n()) %>%
  mutate(totals = sum(total)) %>%
  group_by(worn) %>%
  summarise(total_percent = total / totals) %>%
  mutate(labels = scales::percent(total_percent))

minutes_worn_highuse$worn <- factor(minutes_worn_highuse$worn, levels = c("All day", "More than half day", "Less than half day"))
minutes_worn_percent$worn <- factor(minutes_worn_percent$worn, levels = c("All day", "More than half day", "Less than half day"))
minutes_worn_moduse$worn <- factor(minutes_worn_moduse$worn, levels = c("All day", "More than half day", "Less than half day"))
minutes_worn_lowuse$worn <- factor(minutes_worn_lowuse$worn, levels = c("All day", "More than half day", "Less than half day"))

head(minutes_worn_percent)
head(minutes_worn_highuse)
head(minutes_worn_moduse)
head(minutes_worn_lowuse)
```

```
ggarrange(
  ggplot(minutes_worn_percent, aes(x="",y=total_percent, fill=worn)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold"),
        plot.subtitle = element_text(hjust = 0.5)) +
    scale_fill_manual(values = c("#004d99", "#3399ff", "#cce6ff"))+
  geom_text(aes(label = labels),
            position = position_stack(vjust = 0.5), size = 3.5)+
  labs(title="Time worn per day", subtitle = "Total Users"),
  ggarrange(
  ggplot(minutes_worn_highuse, aes(x="",y=total_percent, fill=worn)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold"),
        plot.subtitle = element_text(hjust = 0.5), 
        legend.position = "none")+
    scale_fill_manual(values = c("#004d99", "#3399ff", "#cce6ff"))+
  geom_text_repel(aes(label = labels),
            position = position_stack(vjust = 0.5), size = 3)+
  labs(title="", subtitle = "High use - Users"), 
  ggplot(minutes_worn_moduse, aes(x="",y=total_percent, fill=worn)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold"), 
        plot.subtitle = element_text(hjust = 0.5),
        legend.position = "none") +
    scale_fill_manual(values = c("#004d99", "#3399ff", "#cce6ff"))+
  geom_text(aes(label = labels),
            position = position_stack(vjust = 0.5), size = 3)+
  labs(title="", subtitle = "Moderate use - Users"), 
  ggplot(minutes_worn_lowuse, aes(x="",y=total_percent, fill=worn)) +
  geom_bar(stat = "identity", width = 1)+
  coord_polar("y", start=0)+
  theme_minimal()+
  theme(axis.title.x= element_blank(),
        axis.title.y = element_blank(),
        panel.border = element_blank(), 
        panel.grid = element_blank(), 
        axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        plot.title = element_text(hjust = 0.5, size=14, face = "bold"), 
        plot.subtitle = element_text(hjust = 0.5),
        legend.position = "none") +
    scale_fill_manual(values = c("#004d99", "#3399ff", "#cce6ff"))+
  geom_text(aes(label = labels),
            position = position_stack(vjust = 0.5), size = 3)+
  labs(title="", subtitle = "Low use - Users"), 
  ncol = 3), 
  nrow = 2)
```

<img src="https://www.kaggleusercontent.com/kf/75989631/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..s4k9VwKPAZlqkxEB_nEDWA.5G615DRlx73jFXH42x87bZWAU0oiumHSQIxzXYbkPpfHCHvnp3HG0suADUdOXW_X1KMTQWyd86huo9qb4rfkllLza7O63cGWNEX3Ew_oibikNwEG9CHIt2c1dsUPa4EYnV4tttl5Cls6NbINguZl3hAI0Fx32PwcjzFgQfOOcma_H7Jlzp_eD92nrycyL8yhd9T6bHVsVxjcdVXzloPJvcwygpgzEqQG6use5nGA_D8aQ7iw_iCTO3M4XhwvpUIK9eFmhvr6zsu0dPe90jpyheqKu9dWtY5Vjm5DhFNU8g7A9N5uXUSB6Z7uSMAxa7Vovx5oHVGvplU3g-dQeE7JAYd8s54V_KiA5hqEy_0WgWPVTYOc6R9Y_cp_us5bupqbtO1Xfy9uFSfzWkryegpF8VoATgYfcbLuXQjbC4IM-YQCnRj3T_2GKG4uljOw6ok_R9wCCvaKOqQubMy5KLfGRr-EIkkpuE0jCE0UvPjsVv4rDSsYw0OdB2iObwVeiS1rv7ThlUANzwG7sgVwscLY5KP52GNsCG6kAxvzEZ7tr5YCZCg-Xn4PmQEBYJ65Xxq7WjzkYYGm0KXNbQkEYk5wQH0plFKSwGsM4kd9AH_x-f0otXgB9CreUOQY7QJR6yN9aikHPABQcxcRbE7td1Pn3SpVh7fKmvs-vD-Gxmz_DgQZUKAHs_El4P_9OGVcRNVN.edzKSpjtomukSs2zGXQdnA/__results___files/__results___54_0.png" width="400" alt="image">

Based on our visual representations, it becomes evident that 36% of the total users wear the device throughout the entire day, while 60% wear it for more than half a day, leaving only 4% who wear it for less than half a day.

When we narrow down our analysis to the total users based on their usage patterns and daily duration of device wear, the outcomes are as follows:

A quick recap:
- High use: Users who employ their device for 21 to 31 days.
- Moderate use: Users who utilize their device for 10 to 20 days.
- Low use: Users who engage with their device for 1 to 10 days.

Among high users, merely 6.8% of those who use the device for 21 to 31 days have it on all day. A significant 88.9% wear the device for more than half a day but not the entire day.

Moderate users, on a daily basis, tend to wear the device for a shorter duration.

Interestingly, low users tend to wear their device for a longer period on the days they choose to use it.

# 5-Act
Bellabeat's core objective revolves around empowering women through the provision of insightful data for self-discovery.

To effectively address our business objective and align with Bellabeat's mission, based on our findings, I would recommend leveraging proprietary tracking data for further analysis. The datasets utilized in our study possess a limited sample size and potential bias, attributed to the absence of user demographic details. Given that our primary target audience encompasses young and adult women, I strongly advocate for the continued exploration of trends to facilitate the creation of a marketing strategy tailored to this demographic.

With that in mind, post our comprehensive analysis, we have identified distinct trends that hold the potential to significantly enhance our online campaign and refine the Bellabeat app:

- Daily notification
- Sleep techniques
- Rewarding

Our analysis extended beyond the exploration of daily user habits; we also uncovered significant insights. Notably, only half of the users employ their device on a daily basis, and merely 36% of users wear the device throughout the entirety of their active day. This underscores the opportunity to further highlight the distinctive features of Bellabeat's products:

- Water Resistance: Emphasize the products' water-resistant nature, allowing users to wear them confidently even in wet conditions.
- Long-Lasting Batteries: Showcase the extended battery life, providing users with peace of mind and uninterrupted usage throughout the day.
- Fashionable and Elegant Design: Highlight the aesthetic appeal of the products, making them suitable for any occasion, whether casual or formal. Users can wear these products seamlessly without concerns about battery depletion.

These features collectively ensure that users can integrate Bellabeat's products seamlessly into their daily lives, enhancing both functionality and style.

Lastly:

In conclusion, our comprehensive analysis of user behaviors and device usage patterns has illuminated valuable insights for Bellabeat's strategic direction. By categorizing users based on their engagement levels and daily habits, we have gained a deeper understanding of their interactions with the devices.

We observed that a substantial portion of users, nearly 50%, engage with the device on a daily basis. This highlights a significant opportunity to further engage the remaining user base and promote consistent device usage.

Furthermore, our investigation into daily wear duration revealed that only 36% of users wear the device throughout the entirety of their active day. This underscores the importance of emphasizing key product features that contribute to seamless integration into users' routines. Water resistance, extended battery life, and fashionable design were identified as particularly compelling attributes that resonate with users seeking versatile and reliable wearable technology.

To address these findings, we recommend a multi-faceted approach:

- Daily Engagement Encouragement: Implementing a daily step notification system and in-app posts can incentivize users to meet the recommended step goals. This initiative can also capitalize on the positive correlation between step count and calorie expenditure.

- Enhancing Sleep Habits: Introducing sleep-related notifications and techniques can empower users to improve their sleep patterns, offering valuable resources to aid relaxation and restful sleep.

- Gamified Rewards: The introduction of a gamified reward system offers an innovative way to engage users and motivate them to maintain consistent activity levels. This approach complements traditional notifications and appeals to a wider range of users.

By aligning our marketing and product strategies with these recommendations, Bellabeat can strengthen its mission of empowering women through data-driven insights. By fostering deeper engagement, promoting key product features, and offering compelling incentives, Bellabeat can continue to make a positive impact on the lives of its users and contribute to their well-being and self-discovery journey.