# Retail Analytics: Predicting Customer Purchase Behavior
 
A multi-tool capstone project analyzing customer and marketing campaign data for a retail company, with the goal of identifying which customers are most likely to respond to a marketing campaign — and turning that into an actionable, profit-driven recommendation.
 
**Author:** Marina Ferreira
 
## Business Question
 
Companies send marketing campaigns to their entire customer base, but only a small fraction respond. This project asks: **can we predict, in advance, which customers are likely to respond — and use that to make campaigns more effective and profitable?**
 
## Dataset
 
- **Source:** `marketing_campaign.csv` — 2,240 customer records, 27 variables
- **Variables:** demographics (education, marital status, income, household), spending by product category (wine, meat, fish, sweets, gold, fruit), purchase channels (web, store, catalog), and responses to 6 past marketing campaigns
## Tools & Tech Stack
 
| Stage | Tool |
|---|---|
| Data cleaning & initial exploration | Excel (PivotTables, Data Analysis ToolPak, charts) |
| Database & preprocessing | MySQL (schema design, data loading, aggregate queries) |
| Exploratory data analysis & statistical testing | Python (pandas, matplotlib, seaborn, scipy) |
| Data visualization & dashboarding | Tableau, Python (matplotlib, seaborn) |
| Presentation | PowerPoint |
 
## Repository Structure
 
```
├── marketing-campaign-response-prediction-excel.xlsx         # Data cleaning, PivotTables, charts
├── marketing-campaign-response-prediction-sql.sql             # Schema, data load, preprocessing queries
├── marketing-campaign-response-prediction-python.ipynb        # EDA, univariate/bivariate analysis, Chi-Square test
├── marketing-campaign-response-prediction-tableau              # Required dashboard (customer behavior overview)
├── marketing-campaign-response-prediction-tableau-extra         # Extra dashboard built for the class presentation
├── marketing-campaign-response-prediction.pptx                   # Final slide deck
└── marketing_campaign.csv                                      # Raw dataset
```
 
## Methodology
 
### 1. Data Cleaning (Excel)
 
Identified and handled mixed date formats, missing income values (imputed with the median), and biologically implausible ages (3 records excluded). Built descriptive statistics, a cross-tabulation, and exploratory charts.
 
### 2. Database & Preprocessing (SQL)
 
Designed and loaded a relational table in MySQL, then wrote aggregate queries to unpivot campaign columns, bucket customers into age groups, and compute response distributions and per-group averages.
 
```sql
CREATE SCHEMA retail_data;
USE retail_data;
CREATE TABLE marketing_campaign (
  ID INT PRIMARY KEY,
  Year_Birth INT,
  Education VARCHAR(50),
  Marital_Status VARCHAR(50),
  Income DECIMAL(10,2),
  Kidhome INT,
  Teenhome INT,
  Dt_Customer DATE,
  Recency INT,
  MntWines INT,
  MntFruits INT,
  MntMeatProducts INT,
  MntFishProducts INT,
  MntSweetProducts INT,
  MntGoldProds INT,
  NumDealsPurchases INT,
  NumWebPurchases INT,
  NumCatalogPurchases INT,
  NumStorePurchases INT,
  NumWebVisitsMonth INT,
  AcceptedCmp1 INT,
  AcceptedCmp2 INT,
  AcceptedCmp3 INT,
  AcceptedCmp4 INT,
  AcceptedCmp5 INT,
  Complain INT,
  Response INT
);
```
 
```sql
-- Identify the top purchased products, ranked
SELECT 'Wines' AS Product, SUM(MntWines) AS Total FROM marketing_campaign
UNION ALL
SELECT 'Fruits', SUM(MntFruits) FROM marketing_campaign
UNION ALL
SELECT 'Meat', SUM(MntMeatProducts) FROM marketing_campaign
UNION ALL
SELECT 'Fish', SUM(MntFishProducts) FROM marketing_campaign
UNION ALL
SELECT 'Sweet', SUM(MntSweetProducts) FROM marketing_campaign
UNION ALL
SELECT 'Gold', SUM(MntGoldProds) FROM marketing_campaign
ORDER BY Total DESC;
```
 
```sql
-- Create an Age_group bucket and find average web visits per group
SELECT
  CASE
    WHEN (YEAR(CURDATE()) - Year_Birth) BETWEEN 18 AND 25 THEN '18-25'
    WHEN (YEAR(CURDATE()) - Year_Birth) BETWEEN 26 AND 35 THEN '26-35'
    WHEN (YEAR(CURDATE()) - Year_Birth) BETWEEN 36 AND 45 THEN '36-45'
    WHEN (YEAR(CURDATE()) - Year_Birth) BETWEEN 46 AND 55 THEN '46-55'
    ELSE '56+'
  END AS Age_group,
  AVG(NumWebVisitsMonth) AS avg_visits
FROM marketing_campaign
GROUP BY Age_group
ORDER BY Age_group;
```
 
*Full script with all preprocessing queries: [`marketing-campaign-response-prediction-sql.sql`](./marketing-campaign-response-prediction-sql.sql)*
 
### 3. Exploratory Data Analysis & Statistical Testing (Python)
 
Ran descriptive statistics, univariate and bivariate analysis, a correlation matrix against `Response`, and a **Chi-Square test** on categorical variables like Education.
 
```python
# Data quality checks
df.dtypes
df.isnull().sum()
df.describe()
```
 
```python
# Univariate analysis
df.hist(figsize=(15, 10))
 
counts = df['Education'].value_counts()
plt.bar(counts.index, counts.values)
plt.title("Distribution of Education Level")
```
 
```python
# Bivariate analysis: correlation with Response
corr_matrix = df.corr(numeric_only=True)
print(corr_matrix)
 
sns.boxplot(x='Response', y='MntWines', data=df)
plt.title("Wine Spending by Response")
```
 
```python
# Pareto chart: total spending by product category
mnt_cols = ['MntWines', 'MntFruits', 'MntMeatProducts', 'MntFishProducts', 'MntSweetProducts', 'MntGoldProds']
totals = df[mnt_cols].sum().sort_values(ascending=False)
cum_pct = totals.cumsum() / totals.sum() * 100
 
fig, ax1 = plt.subplots(figsize=(10, 6))
ax1.bar(totals.index, totals.values, color='steelblue')
ax2 = ax1.twinx()
ax2.plot(totals.index, cum_pct.values, color='red', marker='o')
ax2.axhline(80, color='gray', linestyle='--')
plt.title('Pareto Chart: Total Spending by Product Category')
```
 
```python
# Chi-Square test: Education vs Response — catches associations correlation misses
from scipy.stats import chi2_contingency
 
contingency_table = pd.crosstab(df['Education'], df['Response'])
chi2, p_value, dof, expected = chi2_contingency(contingency_table)
 
print(f"Chi-Square Statistic: {chi2:.2f}")
print(f"P-value: {p_value:.4f}")
 
if p_value < 0.05:
    print("There is a statistically significant association between Education and Response.")
```
 
```python
# Response rate by education level
response_rate_by_edu = df.groupby('Education')['Response'].mean().sort_values(ascending=False) * 100
print(response_rate_by_edu)
```
 
*Full notebook: [`marketing-campaign-response-prediction-python.ipynb`](./marketing-campaign-response-prediction-python.ipynb)*
 
### 4. Visualization (Tableau)
 
Built distribution, comparison, and relationship charts (boxplots, packed bubbles, scatter, grouped bars) into two dashboards — one covering general customer behavior, one focused specifically on what predicts campaign response.
 
## Presentation
 
The full findings are summarized in `marketing-campaign-response-prediction.pptx`, an 11-slide deck covering the business question, methodology, key findings, and final recommendation.
