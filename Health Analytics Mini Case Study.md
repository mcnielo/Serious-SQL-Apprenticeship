# ⛑️ Health Analytics Mini Case Study

## 📌 Solution

### 1. How many unique users exist in the logs dataset?

````sql
SELECT 
  COUNT(DISTINCT id) AS unique_users
FROM health.user_logs
````
### Answer:
| unique_users |
| -------------|
| 554          |

***

### For Q2 to Q8, we will create a temporary table.

````sql
DROP TABLE IF EXISTS user_measure_count;

CREATE TEMP TABLE user_measure_count AS(
SELECT
  id,
  COUNT(*) AS measure_count,
  COUNT(DISTINCT measure) AS unique_measure_count
FROM health.user_logs
GROUP BY id);
````
***

### 2. How many total measurements do we have per user on average?

````sql
SELECT 
  ROUND(AVG(measure_count),2) AS avg_measurement
FROM user_measure_count;
````

### Answer:

| avg_measurement |
| ----------------|
| 79.23           |

***

### 3. What about the median number of measurements per user?

````sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
````
### Answer:
| median_value |
| -------------|
| 2            |

***

### 4. How many users have 3 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3
````
### Answer:
| count |
| ------|
| 209   |

***

### 5. How many users have 1,000 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000
````

### Answer:
| count |
| ------|
| 5     |

***

### Looking at the logs data -
### 6. What is the number and percentage of the active user base who have logged blood glucose measurements?

````sql
SELECT
  measure,
  COUNT(DISTINCT id) AS unique_blood_glucose_user,
  ROUND(100 * COUNT(DISTINCT id)::NUMERIC / SUM(COUNT(DISTINCT id)) OVER (),2) AS blood_glucose_percentage
FROM health.user_logs
GROUP BY measure;
````

### Answer:
| measure       | unique_blood_glucose_user | blood_glucose_percentage |
| ------------- | ------------------------- | ------------------------ |
| blood_glucose | 325                       | 40.22                    |

***

### 7. What is the number and percentage of the active user base who have at least 2 types of measurements?

````sql
WITH measure_more_than_2 AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count >= 2)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(100 * COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN measure_more_than_2 AS m
  ON u.id = m.id;
````

### Answer:

| unique_user | unique_user_percentage | 
| ----------- | ---------------------- | 
| 204         | 36.82                  | 

***
### 8. What is the number and percentage of the active user base who have all 3 measures - blood glucose, weight and blood pressure?

````sql
WITH all_measures AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count = 3)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN all_measures AS m
  ON u.id = m.id;
````
### Answer:

| unique_user | unique_user_percentage | 
| ----------- | ---------------------- | 
| 50          | 9.03                   | 

***
### 9. For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?

````sql
SELECT
  'blood_pressure' AS measure_name,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS systolic_median,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS diastolic_median
FROM health.user_logs
WHERE measure = 'blood_pressure';
````
### Answer:

| measure_name   | systolic_median | diastolic_median |
| -------------- | --------------- | ---------------- |
| blood_pressure | 126             | 79               |

***

