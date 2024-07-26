# Insurance-Policy-Analysis
Insurance Policy Analysis Dashboard

# Project Overview:
In this project, I utilized MySQL for data processing and Power BI for visualization to analyze insurance policy data. The aim was to gain insights into insured values, yearly premiums, and the impact of natural disasters on different business types and construction methods.

# Tools and Technologies:
Database: MySQL
Visualization: Power BI

# Key Insights and Features:
- Sum of Insured Value by Business Type: This line chart shows the total insured value for various business types, helping identify sectors with the highest and lowest coverage. Insight: Sectors like Apartments and Offices have the highest insured values, while Retail has the lowest.
- Average Yearly Premium by Business Type: This bar chart illustrates the average yearly premium for each business type. Insight: Business types like Construction and Apartments have higher average premiums, indicating a higher risk or higher coverage amounts.
- Count of Policies by Risk Rating and Description: A pie chart displaying the distribution of policies based on risk ratings. Insight: Most policies fall under risk rating A0, suggesting that the majority are considered low-risk.
- Impact of Natural Disasters by Construction Type: A table summarizing the number of policies affected by floods and earthquakes for each construction type. Insight: Frame construction is the most affected by natural disasters, while Fire Resist construction is the least affected.
- Policy Count and Goal Tracking: A card visual showing the total count of policies and progress towards set goals. Insight: Helps track the performance and targets in policy issuance.
- Interactive Time Travel for Data Analysis: An interactive time slider to filter data based on specific date ranges, regions, and locations. Insight: Allows for dynamic data analysis and trend observation over time.
# Dashboard Images
https://app.powerbi.com/groups/me/reports/3a2e6ebe-c5e6-488c-bcc1-6ba878133ff0/ReportSection?experience=power-bi

[Dashboard 1](Dashboard1.png)
[Dashboard 2](Dashboard2.png)

## SQL Queries Used

### Create Policy_Detail Table and Insert Data

```sql
CREATE TABLE Policy_Detail(
    PolicyId INT,
    StartDate DATE,
    Location TEXT,
    State TEXT,
    Region TEXT,
    InsuredValue INT,
    Construction TEXT,
    BusinessType TEXT,
    Earthquake INT,
    Flood INT
);

LOAD DATA INFILE 'D:/one-click-test/one-click-test/sql_data/Policy_Detail.csv'
INTO TABLE Policy_Detail
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\\r\\n'
IGNORE 1 ROWS;
```

### Select Low Risk Policies in East Region

```sql
SELECT pd.* 
FROM Policy_Detail pd
LEFT JOIN Policy_RiskRating prr ON pd.PolicyID = prr.PolicyID
LEFT JOIN RiskRating rr ON prr.RiskRating = rr.RiskRating
WHERE rr.Description = 'Low' AND pd.Region = 'East';
```

### Month-on-Month Growth/Decline of Policies

```sql
WITH MonthlyPolicyCounts AS (
    SELECT
        BusinessType,
        DATE_FORMAT(StartDate, '%Y-%m') AS Month,
        COUNT(*) AS NumPolicies
    FROM
        Policy_Detail
    GROUP BY
        BusinessType, DATE_FORMAT(StartDate, '%Y-%m')
)
SELECT
    BusinessType,
    Month,
    NumPolicies,
    ROUND(((NumPolicies - LAG(NumPolicies) OVER (PARTITION BY BusinessType ORDER BY Month)) / NULLIF(LAG(NumPolicies) OVER (PARTITION BY BusinessType ORDER BY Month), 0)) * 100, 2) AS GrowthPercentage
FROM
    MonthlyPolicyCounts;
```

### Construction Types Most and Least Affected by Natural Disasters

```sql
SELECT
    Construction,
    SUM(Earthquake) AS EarthquakeAffected,
    SUM(Flood) AS FloodAffected,
    SUM(Earthquake + Flood) AS TotalAffected
FROM
    Policy_Detail
GROUP BY
    Construction
ORDER BY
    TotalAffected ASC, Construction;
```

### Region Least Affected by Natural Disasters

```sql
WITH RegionDisasterCounts AS (
    SELECT
        Region,
        SUM(Earthquake + Flood) AS TotalAffected,
        RANK() OVER (ORDER BY SUM(Earthquake + Flood)) AS RankByDisaster
    FROM
        Policy_Detail
    GROUP BY
        Region
)
SELECT
    Region
FROM
    RegionDisasterCounts
WHERE
    RankByDisaster = 2;
```

### Calculate Yearly Premium for Each Policy

```sql
ALTER TABLE Policy_Detail
ADD COLUMN YearlyPremium DECIMAL(10, 2);

UPDATE Policy_Detail pd
JOIN Policy_RiskRating prr ON pd.PolicyID = prr.PolicyID
JOIN RiskRating rr ON prr.RiskRating = rr.RiskRating
SET pd.YearlyPremium = (CAST(pd.InsuredValue AS DECIMAL(20, 2)) * CAST(rr.PremiumAdjustment AS DECIMAL(20, 2)))
WHERE pd.PolicyID IS NOT NULL;
```
```
