
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

`````
-- SQL Query
SELECT *
FROM subscriptions
LIMIT 15;
`````

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

`````
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
`````







