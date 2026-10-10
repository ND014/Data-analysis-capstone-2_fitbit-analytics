# 🌿 Bellabeat Smart Device Analytics: Consumer Health & Behavior Case Study
### Google Data Analytics Professional Certificate — Case Study 2

[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![pandas](https://img.shields.io/badge/pandas-2.0+-green.svg)](https://pandas.pydata.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)

---

## 📌 1. Executive Summary & Company Overview

**Bellabeat** is a high-tech manufacturer of beautifully designed health-focused smart products for women, founded in 2013 by **Urška Sršen** (Chief Creative Officer) and **Sando Mur** (mathematician). By collecting data on activity, sleep, stress, and reproductive health, Bellabeat empowers women with personalized wellness insights into their daily routines.

While Bellabeat has experienced rapid global growth since its inception, the executive team recognized that analyzing broader consumer fitness tracker data could unlock powerful new marketing opportunities and drive app subscription growth.

### Core Bellabeat Product Ecosystem
* **Bellabeat App**: The central wellness platform providing daily activity, sleep, hydration, stress, and menstrual cycle guidance.
* **Leaf**: Bellabeat’s signature wellness tracker worn as a bracelet, necklace, or clothing clip. Connects wirelessly to the Bellabeat app.
* **Time**: A hybrid analog-smart wellness watch tracking movement, rest, and stress resilience.
* **Spring**: A smart water bottle tracking daily hydration habits and reminding users to drink throughout the day.
* **Bellabeat Membership**: A subscription-based coaching program offering tailored fitness, nutrition, and mindfulness routines based on individual biometric data.

---

## 🎯 2. Business Task & Key Questions

> **Business Statement**: Analyze non-Bellabeat consumer smart device data to identify behavioral trends in activity, sedentary habits, sleep, and feature adoption. Translate these findings into actionable marketing strategies to fuel the growth of Bellabeat's product lines and premium membership.

### Guiding Business Questions:
1. **What are some trends in smart device usage?**
2. **How could these trends apply to Bellabeat customers?**
3. **How could these trends help influence Bellabeat marketing strategy?**

---

## 📁 3. Data Source & Preparation

| Detail | Specifications |
|:---|:---|
| **Dataset** | [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) |
| **Publisher** | Mobius (via Kaggle) |
| **License** | CC0: Public Domain |
| **Study Window** | April 12, 2016 – May 12, 2016 (31 days) |
| **Participants** | 33 eligible Fitbit users who consented to tracking submission |
| **Data Types** | Daily activity, heart rate, sleep monitoring, hourly movement, and weight logs |

### Data Triage & Selection (The Process Phase)
The raw dataset originally contained 18 separate CSV files representing different SQL aggregations and table shapes (e.g., Narrow vs. Wide minute-level exports). During initial data auditing:
* **13 redundant and granular files were triaged out**: 
  - `dailySteps`, `dailyCalories`, and `dailyIntensities` were already combined into `dailyActivity_merged.csv`.
  - Minute-level Narrow/Wide tables (>250 MB) provided identical constituent rows already aggregated into hourly and daily rollups.
  - Heart rate data (5-second intervals) suffered from severe missing user coverage (<50% of users).
* **Focused on 4 Core Worksheets**:
  1. `dailyActivity_merged.csv` (Overall daily movement, distances, and calories)
  2. `sleepDay_merged.csv` (Nocturnal sleep duration and time in bed)
  3. `hourlySteps_merged.csv` & `hourlyCalories_merged.csv` (24-hour diurnal patterns)
  4. `weightLogInfo_merged.csv` (Feature compliance and logging friction)

---

## 🧹 4. Data Cleaning & Transformation Pipeline

All transformations were implemented programmatically in Python within [`code.ipynb`](code.ipynb):

1. **Standardizing Formats & Column Casing**:
   - Converted inconsistent date formats (`ActivityDate`, `SleepDay`, `Date`) into a uniform `Date` column typed as `datetime64[ns]`.
   - Extracted calendar features: `DayOfWeek`, `DayNum`, and `IsWeekend`.
2. **Deduplication**:
   - Identified and permanently removed **3 duplicate records** in `sleepDay_merged.csv` (reducing rows from 413 to 410 on disk).
3. **Single Master Worksheet Creation (`combined_daily_fitbit_data.csv`)**:
   - Merged `dailyActivity` and `sleepDay` on `['Id', 'Date']` into a unified master health table (410 complete daily records across 24 unique users).
4. **Engineered Health Metrics**:
   - `TotalActiveMinutes` = $\text{VeryActiveMinutes} + \text{FairlyActiveMinutes} + \text{LightlyActiveMinutes}$
   - `TimeLyingInBedNotAsleep` = $\text{TotalTimeInBed} - \text{TotalMinutesAsleep}$ (Sleep latency / restless wake time)
   - `SleepEfficiency` = $\frac{\text{TotalMinutesAsleep}}{\text{TotalTimeInBed}} \times 100$
5. **Data Quality & Physical Validation Checks**:
   - **24-Hour Day Limit**: Verified that total tracked minutes ($\text{Active} + \text{Sedentary}$) never exceeded 1,440 minutes/day (Max: 1,398 mins).
   - **Sleep Consistency**: Verified that $\text{Minutes Asleep} \le \text{Time In Bed}$ across 100% of rows (0 violations).
   - **Zero-Value Audit**: Verified 0 null values across all 24 columns in the master table. Identified that `LoggedActivitiesDistance` is 95.4% zeroes because consumers avoid manual workout entry.

---

## 📊 5. Key Analytical Findings

### 1. The 10,000 Step Deficit (User Activity Tiers)
Using the clinical Tudor-Locke & Bassett (2004) activity classification:
* **78.8% of users fail** to reach the CDC-recommended 10,000 steps/day benchmark.
* **51.5% of users** fall into either the *Sedentary* (<5,000 steps) or *Low Active* (5,000–7,499 steps) categories.
* Average daily steps across the study: **7,638 steps** (Median: 7,406 steps).

### 2. Overwhelming Sedentary Lifestyle
* Users spend an average of **991 minutes (16.5 hours) sedentary per day**.
* **Over 81.3% of total tracked waking hours are spent sitting** or inactive.
* Users only spend ~35 combined minutes in moderate-to-vigorous physical activity daily.

### 3. The 39-Minute Sleep Latency Gap & "Sunday Night Insomnia"
* Users spend an average of **39.3 minutes awake in bed each night** tossing, turning, or lingering awake before falling asleep.
* **Sunday Night Spike**: Sleep latency jumps to **50.8 minutes on Sunday nights**—evidence of pre-workweek stress and irregular weekend sleep timing.
* **Sedentary Link ($r = -0.60$)**: Strong negative correlation showing that **higher daytime sedentary time directly associates with shorter, lower-quality sleep at night**.

### 4. Day-of-Week Behavioral Trends
| Day of Week | Avg Steps | Avg Calories | Avg Sleep (Hours) | Avg Awake in Bed (Mins) | Pattern |
|:---|:---:|:---:|:---:|:---:|:---|
| **Monday** | 7,781 | 2,324 | 7.0 hrs | 37.8 mins | Workweek start; moderate activity |
| **Tuesday** | 8,125 | 2,356 | 6.7 hrs | 38.8 mins | High midweek activity |
| **Wednesday** | 7,559 | 2,303 | 7.2 hrs | 35.3 mins | Midweek baseline |
| **Thursday** | 7,406 | 2,200 | 6.7 hrs | 33.6 mins | Lowest sleep night |
| **Friday** | 7,448 | 2,332 | 6.8 hrs | 39.6 mins | End-of-week wind-down |
| **Saturday** | **8,153** | **2,355** | 7.0 hrs | 40.8 mins | **Peak activity day** (outdoor leisure & errands) |
| **Sunday** | 6,933 | 2,263 | **7.5 hrs** | **50.8 mins** | **Rest & recharge day**; longest sleep latency |

### 5. 24-Hour Movement Windows
* **Midday Lunch Walk (12:00 PM – 2:00 PM)**: Steady activity bump averaging ~540 steps/hour.
* **Evening Peak (5:00 PM – 7:00 PM)**: Peak movement window of the day, peaking at **18:00 (599 steps/hour)** from gym sessions and commutes.
* **Afternoon Slump (2:30 PM – 3:30 PM)**: Deepest daytime sedentary lull.

### 6. User Retention vs. Manual Logging Friction
* **Hardware Wear Adherence**: **87.9% of users (29 of 33)** wore their devices consistently for 21 to 31 days, confirming strong wearable habit formation.
* **Manual Feature Rejection**: **75.8% of users** completely avoided logging weight. Of those who did, 61.2% were forced to enter numbers manually and stopped after 1–3 days. Users demand 100% automated, passive background tracking.

---

## 📈 6. Visualizations Summary

All visualizations are generated and saved in [`code.ipynb`](code.ipynb):

1. **Viz 1: Daily Step Distribution & CDC Benchmark**:
   Histogram + KDE showing that the bulk of user days cluster between 5,000 and 8,500 steps, highlighting the unmet 10,000-step target.
2. **Viz 2: Time Lying Awake in Bed (Sleep Latency Analysis)**:
   Distribution plot showing the 39.3-minute average awake gap alongside a bar chart illustrating the Sunday night spike (50.8 mins).
3. **Viz 3: User Activity Tiers & Daily Time Allocation**:
   Dual donut charts comparing user categorization against the 81.3% sedentary time allocation.
4. **Viz 4: 24-Hour Diurnal Movement Curve**:
   Hourly step bar chart highlighting the lunch (12–2 PM) and evening post-work (5–7 PM) spikes.
5. **Viz 5: Health & Lifestyle Correlations**:
   Dual regression scatter plots illustrating:
   - Daily Steps vs. Calories Burned ($r = +0.59$)
   - Daytime Sedentary Minutes vs. Sleep Duration ($r = -0.60$)

---

## 💡 7. Strategic Recommendations for Bellabeat (The Act Phase)

```
╔══════════════════════════════════════════════════════════════════════════════╗
║         TOP 4 STRATEGIC MARKETING & PRODUCT RECOMMENDATIONS FOR BELLABEAT    ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### Recommendation 1: Inactivity Alerts & Smart Hydration Pairing
* **The Insight**: Consumers spend 81.3% of their day sitting (16.5 hrs), with a pronounced activity lull at 2:30–3:30 PM.
* **Product Strategy**: Configure the *Bellabeat Leaf* clip and *Time* watch to emit gentle, customizable haptic vibrations after 60 continuous sedentary minutes. Pair in-app reminders with the *Bellabeat Spring* smart water bottle to prompt hydration breaks during the mid-afternoon slump.
* **Target Audience**: Desk-bound professionals and sedentary workers.
* **Marketing Message**: *"Time to bloom: take a stretch and sip fresh water to recharge."*

### Recommendation 2: Bedtime Wind-Down & Sleep Coaching
* **The Insight**: Users lose ~39 minutes every night lying awake in bed (peaking at 50.8 minutes on Sunday nights), and daytime inactivity directly shortens nighttime sleep ($r = -0.60$).
* **Product Strategy**: Introduce an automated "Evening Wind-Down" routine in the *Bellabeat App* triggering at 9:30 PM. Provide guided meditations, breathing exercises, and bedtime audio stories to reduce sleep latency. Deliver educational in-app prompts: *"Getting 20 minutes of brisk movement today will help you fall asleep 15 minutes faster tonight."*
* **Target Audience**: Working women dealing with stress, restlessness, and "Sunday Night Insomnia."
* **Marketing Message**: *"Struggling to drift off? Let Bellabeat guide your evening wind-down."*

### Recommendation 3: Progressive Step Milestones & Saturday Challenges
* **The Insight**: 78.8% of users fail the rigid 10,000-step target, leading to goal demotivation. However, Saturday is naturally the most active day of the week (8,153 steps).
* **Product Strategy**: Replace the intimidating all-or-nothing 10,000-step benchmark with progressive, personalized milestones (5,000, 7,500, 10,000 steps). Introduce gamified "Saturday Step Collectives" and social milestone badges within the *Bellabeat Membership* community.
* **Target Audience**: Low Active and Somewhat Active users looking to build sustainable wellness routines.
* **Marketing Message**: *"Level up your weekend: join the 8,000-step Saturday collective."*

### Recommendation 4: Position 100% Seamless Passive Tracking Over Manual Entry
* **The Insight**: 75.8% of consumers completely avoided manual weight tracking, and manual loggers quickly abandoned the feature due to high friction.
* **Product Strategy**: Center Bellabeat's brand positioning around **effortless, zero-input wellness**. Unlike traditional trackers that require tedious meal and workout logging, emphasize that the *Bellabeat Leaf*, *Time*, and *Spring* automatically monitor activity, stress, hydration, and sleep silently in the background.
* **Target Audience**: Busy women seeking holistic wellness clarity without the mental load of manual logging.
* **Marketing Message**: *"Effortless wellness tracking that fits your life — zero manual logging required."*

---

## 💻 8. Repository Structure

```text
├── README.md                      # Comprehensive capstone project report
├── code.ipynb                     # Executed Jupyter Notebook with complete analysis & charts
├── combined_daily_fitbit_data.csv # Master consolidated daily health tracking dataset (68 KB)
├── .gitignore                     # Git configuration ignoring large raw CSVs
```

---

## 👤 Author
* **Nithish Dhanasekar**
* **Repository**: [Data-analysis-capstone-2_fitbit-analytics](https://github.com/ND014/Data-analysis-capstone-2_fitbit-analytics)
* **Capstone 1**: [Data-analysis-capstone-1_cyclist-marketing-analytics](https://github.com/ND014/Data-analysis-capstone-1_cyclist-marketing-analytics)
