# **SQL Queries for the Analytics challenge**


## **Data Setup**

### **Creating the Marketing Data Table**
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

--Import data from CSV file

### **Creating the Marketing Data Table**
```sql
CREATE TABLE marketing_data2
AS (SELECT * FROM marketing_data);


