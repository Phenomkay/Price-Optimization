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
