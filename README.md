# Case Study: Bellabeat Fitness Data Analysis

**Author**: Caner Uçal

**Date**: 07.08.2023-Monday

<img src="https://play-lh.googleusercontent.com/1DEgw7f-f8Dtp7r0lZ3qn7FfsNb_zYGWVkrAdf5ht8eDFEnRi1HX5Qk-NRTJ9cwbzUg" width="200" alt="bella-logo">

This case study includes the six steps of data analysis process which suggested by Google:

### [Scenario](#scenario-1)

### [1-Ask](#1-ask-1)

### [2-Prepare](#2-prepare-1)

### [3-Process](#3-process-1)

### [4-Analyze](#4-analyze-1)

### [5-Share](#5-share-1)

### [6-Act](#6-act-1)

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