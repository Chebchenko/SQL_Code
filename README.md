# **SQL challenge**

## **Question 1**
Write a query that would provide the earnings per session (earnings/sessions) metric for every day of the week to calculate the best performing day of the week in terms of this metric. Round the metric to 2.

**Answer**
```sql
WITH sessions_by_day AS (
    SELECT
        EXTRACT(DOW FROM visit_date) AS dow,  
        COUNT(DISTINCT session_id) AS sessions  
    FROM 
        user_visits
    GROUP BY 
        EXTRACT(DOW FROM visit_date) 
),

earnings_by_day AS (
    SELECT
        EXTRACT(DOW FROM lo.visit_date) AS dow,  
        SUM(lo.earnings) AS earnings 
    FROM 
        leadouts lo
    GROUP BY 
        EXTRACT(DOW FROM lo.visit_date)  
)
SELECT
    sbd.dow,  
    sbd.sessions,  
    ebd.earnings,  
    ROUND(ebd.earnings / sbd.sessions, 2) AS eps  
FROM
    sessions_by_day sbd
LEFT JOIN
    earnings_by_day ebd ON sbd.dow = ebd.dow  
ORDER BY
    sbd.dow;  
```





# **SQL Queries for the Analytics challenge**


## **Data Setup**

### **Create the Marketing Data Table**
```sql
CREATE TABLE marketing_data (
    partition_date DATE,
    site TEXT,
    traffic_source TEXT,
    mkt_channel TEXT,
    impressions INTEGER,
    ad_clicks INTEGER,
    costs NUMERIC(10, 2),
    sessions INTEGER,
    conversions INTEGER,
    earnings NUMERIC(10, 2)
);
```

Import data from CSV file

### **Create Backup Table**
```sql
CREATE TABLE marketing_data2
AS (SELECT * FROM marketing_data);
```

## **Data Cleaning**

### **Removing Duplicates**
```sql
WITH duplicate_cte AS (
    SELECT *,
    ROW_NUMBER() OVER (
        PARTITION BY partition_date, site, traffic_source, mkt_channel, impressions, ad_clicks, costs, sessions, conversions, earnings
    ) AS row_num
    FROM marketing_data2
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```

### **Standardizing Data**
```sql
UPDATE marketing_data2
SET traffic_source = 'SEO'
WHERE traffic_source LIKE 'seo';

UPDATE marketing_data2
SET traffic_source = 'Bing'
WHERE traffic_source LIKE 'bing';

UPDATE marketing_data2
SET traffic_source = 'Google'
WHERE traffic_source LIKE 'google';

UPDATE marketing_data2
SET traffic_source = 'Facebook'
WHERE traffic_source LIKE 'facebook';

UPDATE marketing_data2
SET traffic_source = 'TikTok'
WHERE traffic_source LIKE 'tiktok';

UPDATE marketing_data2
SET traffic_source = 'YouTube'
WHERE traffic_source LIKE 'youtube';

UPDATE marketing_data2
SET traffic_source = 'Direct'
WHERE traffic_source LIKE 'direct';

UPDATE marketing_data2
SET traffic_source = 'Affiliates'
WHERE traffic_source LIKE 'affiliates';

UPDATE marketing_data2
SET traffic_source = 'Pinterest'
WHERE traffic_source LIKE 'pinterest';

UPDATE marketing_data2
SET traffic_source = 'Remarketing 2'
WHERE traffic_source LIKE 'remarketing 2';

UPDATE marketing_data2
SET traffic_source = 'Remarketing 1'
WHERE traffic_source LIKE 'remarketing 1';

UPDATE marketing_data2
SET traffic_source = 'Hisharethat'
WHERE traffic_source LIKE 'hisharethat';
```

### **Changing Data Types**
```sql
ALTER TABLE marketing_data2
ALTER COLUMN sessions TYPE INT;

ALTER TABLE marketing_data2
ALTER COLUMN conversions TYPE INT;
```

### **Handling Zeros and NULLs**
**Identifying Zeros**
```sql
SELECT COUNT(earnings)
FROM marketing_data2
WHERE earnings = 0;

SELECT COUNT(costs)
FROM marketing_data2
WHERE costs = 0;

SELECT COUNT(*)
FROM marketing_data2
WHERE earnings = 0 AND costs = 0;
```
**Identifying NULLs**
```sql
SELECT COUNT(*)
FROM marketing_data2
WHERE earnings IS NULL;

SELECT COUNT(*)
FROM marketing_data2
WHERE costs IS NULL;

SELECT COUNT(*)
FROM marketing_data2
WHERE earnings IS NULL AND costs IS NULL;
```
**Updating NULLs to Zeros**
```sql
UPDATE marketing_data2
SET impressions = 0
WHERE impressions IS NULL;

UPDATE marketing_data2
SET ad_clicks = 0
WHERE ad_clicks IS NULL;

UPDATE marketing_data2
SET costs = 0
WHERE costs IS NULL;

UPDATE marketing_data2
SET sessions = 0
WHERE sessions IS NULL;

UPDATE marketing_data2
SET conversions = 0
WHERE conversions IS NULL;

UPDATE marketing_data2
SET earnings = 0
WHERE earnings IS NULL;
```

**Deleting Rows Where Both Costs and Earnings Are Zero**
```sql
DELETE FROM marketing_data2
WHERE costs = 0 AND earnings = 0;
```

## **Exploratory Data Analysis (EDA)**
### **Total Costs, Earnings, and ROAS**
```sql
SELECT 
    ROUND(SUM(costs), 2) AS total_cost
FROM marketing_data2;

SELECT 
    ROUND(SUM(earnings), 2) AS total_earnings
FROM marketing_data2;

SELECT 
    ROUND(
        CASE 
            WHEN SUM(costs) = 0 THEN 0
            ELSE (SUM(earnings) / SUM(costs)) * 100
        END, 2
    ) AS total_ROAS_percentage
FROM marketing_data2;
```

### **Total Costs and Earnings Grouped by Month**
```sql
SELECT 
    DATE_TRUNC('month', partition_date) AS month,
    ROUND(SUM(costs), 2) AS total_costs,
    ROUND(SUM(earnings), 2) AS total_earnings
FROM marketing_data2
WHERE partition_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY DATE_TRUNC('month', partition_date)
ORDER BY month;
```

### **Identify & Delete data from 2024**
```sql
SELECT *
FROM marketing_data2
WHERE partition_date = '2024-01-01';

DELETE FROM marketing_data2
WHERE partition_date = '2024-01-01';
```

### **ROAS for Each Marketing Channel**
```sql
SELECT 
    mkt_channel,
    SUM(earnings) AS total_earnings,
    SUM(costs) AS total_costs,
    ROUND(
        CASE 
            WHEN SUM(costs) = 0 THEN 0
            ELSE (SUM(earnings) / SUM(costs)) * 100
        END, 2
    ) AS ROAS_percentage
FROM marketing_data2
GROUP BY mkt_channel
ORDER BY ROAS_percentage DESC;
```

### **ROAS for Each Traffic Source**
```sql
SELECT 
    traffic_source,
    SUM(earnings) AS total_earnings,
    SUM(costs) AS total_costs,
    ROUND(
        CASE 
            WHEN SUM(costs) = 0 THEN 0
            ELSE (SUM(earnings) / SUM(costs)) * 100
        END, 2
    ) AS ROAS_percentage
FROM marketing_data2
GROUP BY traffic_source
ORDER BY ROAS_percentage DESC;
```

### **ROAS for Combinations of Marketing Channel and Traffic Source**
```sql
SELECT 
    mkt_channel,
    traffic_source,
    SUM(earnings) AS total_earnings,
    SUM(costs) AS total_costs,
    ROUND(
        CASE 
            WHEN SUM(costs) = 0 THEN 0
            ELSE (SUM(earnings) / SUM(costs)) * 100
        END, 2
    ) AS ROAS_percentage
FROM marketing_data2
GROUP BY mkt_channel, traffic_source
ORDER BY ROAS_percentage DESC;
```

### **Monthly ROAS Variation by Marketing Channel**
```sql
SELECT 
    DATE_TRUNC('month', partition_date) AS month,
    mkt_channel,
    SUM(earnings) AS total_earnings,
    SUM(costs) AS total_costs,
    (SUM(earnings) / NULLIF(SUM(costs), 0)) AS roas
FROM marketing_data2
GROUP BY month, mkt_channel
ORDER BY month, mkt_channel;
```

### **Monthly ROAS Variation by Traffic Source**
```sql
SELECT 
    DATE_TRUNC('month', partition_date) AS month,
    traffic_source,
    SUM(earnings) AS total_earnings,
    SUM(costs) AS total_costs,
    (SUM(earnings) / NULLIF(SUM(costs), 0)) AS roas
FROM marketing_data2
GROUP BY month, traffic_source
ORDER BY month, traffic_source;
```





