
![image](https://github.com/jlgarciatucci/SQL-Calculating-Churn-Rates/assets/98712473/8b87143a-e9b7-46b4-add6-385d429989fc)



# SQL Calculating Churn-Rates

Data Analysis with SQL

## 1.- Context

Four months into launching Codeflix, management asks you to look into
subscription churn rates. It’s early on in the business and people are excited to
know how the company is doing.

The marketing department is particularly interested in how the churn compares
between two segments of users. They provide you with a dataset containing
subscription data for users who were acquired through two distinct channels.

## 2.- Codeflix Subscriber Data Sample and Queries

The dataset provided contains one SQL table, subscriptions. Within the table,
there are 4 columns:

* id - the subscription id

* subscription_start - the start date of the subscription

* subscription_end - the end date of the subscription

* segment - this identifies which segment the subscription owner belongs to

Codeflix requires a minimum subscription length of 31 days, so a user can never
start and end their subscription in the same month.

There are two segments, 87 and 30, which identify users acquired through two
separate marketing channels.

Data in the table goes from 2016-12-01 up to 2017-03-30.

Let's query the first 15 lines from the subscription table:

```sql
-- SQL Query
SELECT *
FROM subscriptions
LIMIT 15;
```

**Query Resutls:**

| id | subscription_start | subscription_end | segment |
|----|--------------------|------------------|---------|
| 1  | 2016-12-01         | 2017-02-01       | 87      |
| 2  | 2016-12-01         | 2017-01-24       | 87      |
| 3  | 2016-12-01         | 2017-03-07       | 87      |
| 4  | 2016-12-01         | 2017-02-12       | 87      |
| 5  | 2016-12-01         | 2017-03-09       | 87      |
| 6  | 2016-12-01         | 2017-01-19       | 87      |
| 7  | 2016-12-01         | 2017-02-03       | 87      |
| 8  | 2016-12-01         | 2017-03-02       | 87      |
| 9  | 2016-12-01         | 2017-02-17       | 87      |
| 10 | 2016-12-01         | 2017-01-01       | 87      |
| 11 | 2016-12-01         | 2017-01-17       | 87      |
| 12 | 2016-12-01         | 2017-02-07       | 87      |
| 13 | 2016-12-01         |                  | 30      |
| 14 | 2016-12-01         | 2017-03-07       | 30      |
| 15 | 2016-12-01         | 2017-02-22       | 30      |

**Database Schema: Subscription**

| name                | type    |
|---------------------|---------|
| id                  | INTEGER |
| subscription_start  | TEXT    |
| subscription_end    | TEXT    |
| segment             | INTEGER |

Rows: 2000

## 3.- Calculate Churn Rates for each segment

For calculating the churn rates let's create some temporary tables:

**1.- months temporary table**

```sql
-- SQL Query
WITH months AS (
  SELECT
    '2017-01-01' AS first_day,
    '2017-01-31' AS last_day
  UNION
  SELECT
    '2017-02-01' AS first_day,
    '2017-02-28' AS last_day
  UNION
  SELECT
    '2017-03-01' AS first_day,
    '2017-03-31' AS last_day
),
```

**2.- cross_join temporary table**

```sql
-- SQL Query
cross_join AS (
  SELECT *
  FROM subscriptions
  CROSS JOIN months
),
```

**3.- status temporary table, showing active status by segment**

```sql
-- SQL Query
status AS (
  SELECT
    id,
    first_day AS month,
    CASE
      WHEN segment = 87
        AND (subscription_start < first_day)
        AND (
          subscription_end > first_day
          OR subscription_end IS NULL
        ) THEN 1
      ELSE 0
    END AS is_active_87,
    CASE
      WHEN segment = 30
        AND (subscription_start < first_day)
        AND (
          subscription_end > first_day
          OR subscription_end IS NULL
        ) THEN 1
      ELSE 0
    END AS is_active_30,
```

**4.- Adding canceled status for both segments to the status table**

```sql
-- SQL Query
status AS (
  SELECT
    id,
    first_day AS month,
    CASE
      WHEN segment = 87
        AND (subscription_start < first_day)
        AND (
          subscription_end > first_day
          OR subscription_end IS NULL
        ) THEN 1
      ELSE 0
    END AS is_active_87,
    CASE
      WHEN segment = 30
        AND (subscription_start < first_day)
        AND (
          subscription_end > first_day
          OR subscription_end IS NULL
        ) THEN 1
      ELSE 0
    END AS is_active_30,
    CASE
      WHEN segment = 87
        AND subscription_end BETWEEN first_day AND last_day THEN 1
      ELSE 0
    END AS is_canceled_87,
    CASE
      WHEN segment = 30
        AND subscription_end BETWEEN first_day AND last_day THEN 1
    ELSE 0
    END AS is_canceled_30
FROM cross_join
```

**5.- Aggregating the sum of active and cancelled subscriptions by month, and calculation of the churn rates in percentage multiplying by 100**

```sql
-- SQL Query
status_aggregate AS (
  SELECT
    month,
    SUM(is_active_87) AS sum_active_87,
    SUM(is_active_30) AS sum_active_30,
    SUM(is_canceled_87) AS sum_canceled_87,
    SUM(is_canceled_30) AS sum_canceled_30
  FROM status
  GROUP BY month
)
SELECT
  month,
  100.0 * sum_canceled_30 / sum_active_30 AS churn_rate_30,
  100.0 * sum_canceled_87 / sum_active_87 AS churn_rate_87
FROM status_aggregate;
```










