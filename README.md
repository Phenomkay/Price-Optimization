# Dynamic Pricing Optimization using Price Elasticity of Demand

## Project Overview

This project implements a data-driven **Dynamic Pricing Optimization** model for retail, leveraging **Price Elasticity of Demand** to maximize revenue and quantity sold. In competitive retail environments, static pricing often leads to missed opportunities. This initiative addresses the challenge of optimizing pricing strategies by systematically analyzing historical sales data, understanding customer responsiveness to price changes, and simulating the financial impact of dynamic adjustments.

## Problem Statement

The retail business operates in a competitive market, and its current static pricing strategy may not be fully optimizing revenue. The goal is to develop a dynamic pricing model that optimizes the prices of items to maximize revenue while remaining competitive in the market. This necessitates:

* Analyzing the current pricing strategy and its impact on sales and revenue.
* Comparing our pricing strategy with that of the competition to identify gaps and opportunities.

## Project Objective

The core objective of this project is to develop and validate a dynamic pricing framework that can strategically adjust item prices to significantly improve overall revenue and sales quantity compared to existing pricing strategies. This involves:

* Developing a dynamic pricing model that adjusts prices based on factors such as competitor pricing, demand elasticity, and market trends.
* Implementing and simulating the dynamic pricing model to compare its performance against the existing pricing strategy.


## Data Acquisition

The dataset utilized in this project was sourced from **Aman Kharwal**, a distinguished Data Scientist, Machine Learning, and AI Engineer. Aman is widely recognized for his extensive contributions to the data community through one of the largest data networks, where he consistently shares invaluable knowledge and insights on Data Analytics, Data Science, Machine Learning, and Artificial Intelligence.

The dataset is publicly available and can be accessed directly from [Statso.io: Price Optimization Case Study](https://statso.io/price-optimization-case-study/). His LinkedIn profile can be found here: [Aman Kharwal's LinkedIn](https://www.linkedin.com/in/aman-kharwal/).


## Data Loading

The initial step involved loading the raw `Competition_Data.csv` dataset into a Pandas DataFrame, typically named `df`. This foundational step prepares the data for all subsequent cleaning, transformation, and analysis processes. Key libraries such as `pandas` for efficient data manipulation, `numpy` for numerical operations, and `matplotlib.pyplot` and `seaborn` for comprehensive data visualization were utilized.

Upon loading, an initial inspection of the DataFrame was performed to understand its structure and content

## Column Dictionary

The dataset includes the following columns, providing a comprehensive view of pricing strategies and sales performance:

* **`Fiscal_Week_ID`**: The fiscal week identifier, indicating the specific week of the transaction.
* **`Store_ID`**: The unique identifier for the retail store where the transaction occurred.
* **`Item_ID`**: The unique identifier for the item sold.
* **`Price`**: The price of the item at our store during that fiscal week.
* **`Item_Quantity`**: The quantity of the item sold at our store during that fiscal week.
* **`Sales_Amount_No_Discount`**: The total sales amount for the item before any discounts were applied.
* **`Sales_Amount`**: The final sales amount for the item after discounts were applied.
* **`Competition_Price`**: The price of the same item at a competing store during that fiscal week.


## Initial Data Inspection

After loading the dataset, a preliminary inspection was conducted to understand its structure, data types, and basic statistical properties. This step helps identify potential data quality issues, missing values, or unexpected distributions early in the process.

### DataFrame Information (`df.info()`)

The `df.info()` output provides a summary of the DataFrame, including the number of entries, column names, the count of non-null values for each column, and their respective data types.

**Key Observations from `df.info()`:**
* The dataset contains **100,000 entries** with **9 columns**.
* Crucially, there are **no missing (Non-Null) values** in any of the columns, indicating a clean initial dataset in terms of completeness.
* Data types appear appropriate for most columns: numerical columns (`Price`, `Item_Quantity`, `Sales_Amount_No_Discount`, `Sales_Amount`, `Competition_Price`) are correctly identified as `float64` or `int64`.
* Categorical/identifier columns (`Fiscal_Week_ID`, `Store_ID`, `Item_ID`) are of `object` type, which is expected before further type conversions (e.g., `Fiscal_Week_ID` to datetime).

### DataFrame Description (`df.describe()`)

The `df.describe()` method provides descriptive statistics for numerical columns, offering insights into central tendency, dispersion, and shape of the distributions.

**Key Observations from `df.describe()`:**
* **Price and Competition_Price:** The mean `Price` for our store is ~167.02, while `Competition_Price` is ~174.28. This indicates that, on average, our store's prices are slightly lower than the competition's. Both have similar standard deviations, suggesting a comparable spread in pricing.
* **Item_Quantity:** The average `Item_Quantity` sold is ~399.61. Quantities range from 285 to 522.
* **Sales_Amount_No_Discount vs. Sales_Amount:** The mean `Sales_Amount_No_Discount` (~4771.15) is lower than `Sales_Amount` (~11396.87). This is an interesting observation, as typically discounts would *reduce* sales amount. This suggests that `Sales_Amount` might represent total revenue (Price * Quantity) and `Sales_Amount_No_Discount` might be an additional, unclarified metric, or that `Sales_Amount` includes other factors not explicitly stated. This will require further investigation or clarification during our optimization.


## Data Cleaning & Feature Engineering: Addressing Revenue Anomalies

During the initial data inspection, an inconsistency was observed between the `Sales_Amount_No_Discount` and `Sales_Amount` columns, where `Sales_Amount` was unexpectedly higher than `Sales_Amount_No_Discount` in most cases. Additionally, direct calculation of `Price * Item_Quantity` did not consistently match the `Sales_Amount_No_Discount` column. This anomaly required investigation to establish a reliable base revenue metric for the dynamic pricing model.

### Anomaly Investigation:

To understand the discrepancy, a temporary column `calculated_sales_no_discount` was created by multiplying `Price` and `Item_Quantity`. This was then compared against the original `Sales_Amount_No_Discount` to verify consistency. Furthermore, the relationship between `Sales_Amount` and `Sales_Amount_No_Discount` was explicitly checked.

**Key Findings:**

* **Inconsistency with `Sales_Amount_No_Discount`:** There was a significant difference between `Price * Item_Quantity` and the provided `Sales_Amount_No_Discount` for all records.

* **`Sales_Amount` Higher than `Sales_Amount_No_Discount`:** For almost all records, the `Sales_Amount` (after discounts) was unexpectedly higher than `Sales_Amount_No_Discount` (before discounts).


### Resolution: Defining the Primary Revenue Metric

Given these inconsistencies and to ensure the most accurate base for pricing optimization (which directly links `Price` and `Item_Quantity`), a definitive `Revenue` column was created. This new `Revenue` column is calculated as `Price * Item_Quantity`. This approach provides a clear and consistent metric for the *base revenue* generated by each item at its listed price, serving as the foundation for all subsequent elasticity and dynamic pricing calculations. The temporary `calculated_sales_no_discount` column was subsequently dropped.


## Competitive Landscape Analysis: Price Distribution

Understanding our pricing strategy in relation to competitors is vital. This analysis visualizes the distribution of prices for items in our own store compared to the prices of the same items at competing stores. This helps to identify general pricing trends and positioning in the market.

**Key Observation:**

The histograms below illustrate that while both our store and competitors operate across a similar price range, there are distinct differences in their price distributions. Our store's prices tend to be more concentrated in the lower-to-mid price segments, with prominent peaks around the 100-150 range. In contrast, competitor prices show a stronger presence and higher frequency in the mid-to-higher price brackets (e.g., 150-200 and extending further), indicating a generally higher pricing strategy from the competition. This observation aligns with the initial statistical analysis where our average price was slightly lower than the competition's.

![comparison of price distribution](https://github.com/Phenomkay/Price-Optimization/blob/370006c5ba396769c423c3d43f40fcdc95545f66/comparison%20of%20price%20distribution.png)


## Exploratory Data Analysis: Price-Revenue Relationships

To gain a deeper understanding of how price influences revenue, two key scatter plots were generated. These visualizations help to observe the direct relationship between our own store's pricing and the revenue generated, as well as the less direct influence of competitor pricing on our sales performance.

### Own Store Price vs. Revenue

This plot examines the correlation between the `Price` of an item at our store and the calculated `Revenue` (Price \* Quantity) for that item.

**Key Observation:**
The plot on the left shows a general positive correlation: as `Own Store Price` increases, `Own Store Revenue` also tends to increase. However, the spread of points at higher prices suggests that this relationship is not perfectly linear, and other factors, particularly the quantity sold, play a significant role. This broad distribution hints at varying demand responses for different items.

### Competition Price vs. Own Store Revenue

This second plot investigates whether there's a discernible pattern between the `Competition_Price` and our `Own Store Revenue`.

**Key Observation:**
The plot on the right displays a wide scatter of points, indicating **no strong direct linear correlation between `Competition Price` and `Own Store Revenue`**. Our revenue can be high or low regardless of the competitor's price point. This observation is crucial because it suggests that simply matching or reacting to competitor prices without understanding our own demand characteristics may not be an optimal strategy for maximizing our revenue. This lack of a clear direct relationship reinforces the necessity of an elasticity-based model to truly understand and optimize our pricing.

![price vs revenue](https://github.com/Phenomkay/Price-Optimization/blob/799af3ba1cf64956930c94f192db8c636c3f87b8/price%20vs%20revenue.png)


## Exploratory Data Analysis: Price Trends Over Time

To understand the historical context of pricing and identify any overarching trends, the average prices for both our store and competing stores were plotted over various fiscal weeks. This time-series analysis helps visualize the evolution of pricing strategies.

**Key Observation:**

The plot below reveals a consistent trend: **our store's average prices have historically been lower than those of the competition**. While both price lines show some fluctuations over time, the competition generally maintains a higher average price point. Our store's average price shows a noticeable dip around early April 2019, suggesting a potential pricing adjustment or promotional period during that time. This consistent pricing difference highlights the potential for strategic adjustments through dynamic pricing, either to maintain competitiveness in lower price ranges or to cautiously increase prices where market conditions allow.

![price change over time](https://github.com/Phenomkay/Price-Optimization/blob/799af3ba1cf64956930c94f192db8c636c3f87b8/price%20change%20over%20time.png)


## Key Metrics: Baseline Revenue and Quantity Comparison

Before implementing the dynamic pricing model, it was essential to establish a baseline by calculating the total revenue and quantity sold for our store, and to estimate the equivalent for the competition. This allows for a direct comparison of current market standing.

**Methodology:**
* **Own Store Revenue:** Calculated by summing the `Revenue` column, which was previously defined as `Price * Item_Quantity`.
* **Competition Revenue:** Estimated as a proxy by multiplying `Competition_Price` by our `Item_Quantity` and summing the result. This assumes that if competition had our quantities, this would be their revenue.
* **Quantity Sold:** For comparison purposes, the total `Item_Quantity` sold by our store is used as a consistent base for both own store and hypothetical competition quantity.

**Summary of Baseline Metrics:**

The table below summarizes the total revenue and quantity sold for both our store and the estimated competition:

| Metric            | Own Store     | Competition   |
| :---------------- | :------------ | :------------ |
| Total Revenue     | 6,601,342,000 | 6,965,710,000 |
| Total Quantity Sold | 39,961,130    | 39,961,130    |

**Key Observation:**

This comparison clearly shows that, based on current pricing and quantities, the **estimated total revenue for the competition is higher than our own store's total revenue** (approximately 6.97 billion vs. 6.60 billion). This highlights a significant revenue gap and underscores the opportunity for optimization through dynamic pricing strategies.
