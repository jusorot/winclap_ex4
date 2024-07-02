
# Exercise 4

This document provides an overview of a potential solution for designing a data pipeline to create a week-over-week report using Google Analytics data. The report aims to offer insights into website traffic trends and key performance indicators for each week. The following sections outline the data extraction, modeling, and automation steps involved in the process.


## Data Pipeline Design:
To effectively create a week-over-week report using Google Analytics 4 (GA4) data, we'll leverage the direct integration with BigQuery through the BigQuery Data Transfer Service. This approach ensures seamless data extraction and loading for timely reporting on website traffic trends and key performance indicators.

## Data Extraction:
To extract data from GA4 and store it in BigQuery, the following approach can be used:

1. **Integration Setup**: Utilize GA4's built-in integration with BigQuery. This is configured within GA4's settings under "Admin" > "BigQuery Linking". This eliminates the need for manual API management.
2. **Automated Data Transfer**: Configure the BigQuery Data Transfer Service to automate data extraction from GA4 to BigQuery. This includes scheduling regular updates (e.g., daily) to maintain current data availability.
3. **Data Schema Management**: Google manages schema definition and table creation in BigQuery based on GA4's data structure. This simplifies setup and ensures data consistency without requiring manual schema definition.

Documentation: [BigQuery Export](https://developers.google.com/analytics/bigquery/overview)

Assuming we already have a table automatically updated in BigQuery called `events` with the following columns:

- Date: Date when the interaction occurred.
- Sessions: Number of user sessions initiated.
- Users: Number of unique users who visited the site.
- Pageviews: Total number of pages viewed.
- Bounce Rate: Percentage of sessions in which users exited the site after viewing only one page.
- Conversion Rate: Percentage of sessions that resulted in a conversion (e.g., purchase, registration, download).
- Revenue: Income generated from transactions on the site.

## Data Modeling using dbt:
We need to design a table to store the week-over-week report data.

## Generating the Week-over-Week Report:

We can follow this steps:
1. **Configure dbt:**:
    Create a dbt project and configure it for BigQuery. (Using service account authentication key into profile.yml file)

2. **Create a dbt model**:
    We can define the model `report_weekly.sql` to calculate the week-over-week metrics. The following query can be used:

```sql
-- report_weekly.sql
with weekly_metrics as (
    select
        date_trunc(Date, week) as week_start_date,
        sum(Sessions) as total_sessions,
        sum(Users) as total_users,
        sum(Pageviews) as total_pageviews,
        avg(Bounce_Rate) as avg_bounce_rate,
        avg(Conversion_Rate) as avg_conversion_rate,
        sum(Revenue) as total_revenue
    from
        {{ ref('events') }} -- Reference to the events table in BigQuery
    where
        Date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)  -- Filtering last 7 days (events should be partitioned by Date)
    group by
        week_start_date
),

previous_week_metrics as (
    select
        week_start_date,
        lag(total_sessions) over (order by week_start_date) as prev_total_sessions,
        lag(total_users) over (order by week_start_date) as prev_total_users,
        lag(total_pageviews) over (order by week_start_date) as prev_total_pageviews,
        lag(avg_bounce_rate) over (order by week_start_date) as prev_avg_bounce_rate,
        lag(avg_conversion_rate) over (order by week_start_date) as prev_avg_conversion_rate,
        lag(total_revenue) over (order by week_start_date) as prev_total_revenue
    from
        weekly_metrics
)

insert into {{ ref('report_weekly') }} -- Name of the weekly report table in BigQuery
select
    wm.week_start_date,
    wm.total_sessions,
    wm.total_users,
    wm.total_pageviews,
    wm.avg_bounce_rate,
    wm.avg_conversion_rate,
    wm.total_revenue,
    case when pw.prev_total_sessions > 0 then (wm.total_sessions - pw.prev_total_sessions) / pw.prev_total_sessions else null end as sessions_change_rate,
    case when pw.prev_total_users > 0 then (wm.total_users - pw.prev_total_users) / pw.prev_total_users else null end as users_change_rate,
    case when pw.prev_total_pageviews > 0 then (wm.total_pageviews - pw.prev_total_pageviews) / pw.prev_total_pageviews else null end as pageviews_change_rate,
    case when pw.prev_avg_bounce_rate <> 0 then (wm.avg_bounce_rate - pw.prev_avg_bounce_rate) / pw.prev_avg_bounce_rate else null end as bounce_rate_change_rate,
    case when pw.prev_avg_conversion_rate <> 0 then (wm.avg_conversion_rate - pw.prev_avg_conversion_rate) / pw.prev_avg_conversion_rate else null end as conversion_rate_change_rate,
    case when pw.prev_total_revenue <> 0 then (wm.total_revenue - pw.prev_total_revenue) / pw.prev_total_revenue else null end as revenue_change_rate
from
    weekly_metrics wm
left join
    previous_week_metrics pw on wm.week_start_date = pw.week_start_date;
```

3. **Create a dbt test**:
`schema.yml`
Defining tests for the report_weekly model:

```yaml
version: 2

models:
  - name: report_weekly
    columns:
      - name: week_start_date
        tests:
          - unique
          - not_null
      
      - name: sessions
        tests:
          - not_null
          - non_negative

      - name: pageviews
        tests:
          - not_null
          - non_negative

      - name: users
        tests:
          - not_null
          - non_negative

      - name: bounce_rate
        tests:
          - non_negative
          - range: { min: 0, max: 100 }

      - name: conversion_rate
        tests:
          - non_negative
          - range: { min: 0, max: 100 }

      - name: revenue
        tests:
          - not_null
          - non_negative
```

4. **Scheduling the Model**:

Configure dbt to run periodically and update the models. This job would run the model together with the tests once per week. This can be automated using dbt Cloud or Airflow.
