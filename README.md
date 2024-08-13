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


