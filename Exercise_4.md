## Data Pipeline Design

Design a data pipeline to create a week-over-week report using Google Analytics data. The report should provide insights into website traffic trends and key performance indicators for each week.

### Data Extraction

Describe your approach to extract data from the Google Analytics API and store it in BigQuery. Explain the steps you would take to achieve this, including any necessary authentication, data retrieval, and data loading into BigQuery. You can use any method or tool of your choice, and there is no restriction on using open-source or proprietary solutions.

### Data Modeling

Design a table to store the week-over-week report data. The table should have columns for `week_start_date`, `sessions`, `pageviews`, `users`, `bounce_rate`, `conversion_rate`, etc. The `week_start_date` column will be used to represent each reporting week's starting date.

### Generating the Week-over-Week Report

Use a SQL query to aggregate the data from the extracted table to calculate the week-over-week metrics. For example, calculate the percentage change in sessions, pageviews, users, etc., from the previous week.

### Bonus: Using dbt for Data Modeling (Partial Implementation)

For bonus points, you can use dbt to automate the data modeling process. Create dbt models to define the data transformations required to calculate week-over-week metrics. Store the results in a separate table.

### Note

This is a simplified technical test, and you are not required to implement the entire solution. Focus on the high-level design and key steps of the data pipeline, data extraction from the Google Analytics API, and data modeling process. Feel free to use any method or tool of your choice for the data extraction part.
