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


## Exploratory Data Analysis: Revenue Distribution by Price Bracket

To gain granular insights into pricing effectiveness, the total revenue for both our store and the estimated competition was analyzed across predefined price brackets. This allows for a segmented view of market performance and helps identify which price points are most lucrative and where significant revenue disparities exist.

### Methodology:

1.  **Define Price Brackets:** Standardized price bins and labels were created to categorize both `Price` (Own Store) and `Competition_Price` into consistent ranges.
2.  **Calculate Revenue per Bracket:** For our store, `Revenue` (Price \* Quantity) was summed per `price_bracket`. For the competition, a proxy revenue (`Competition_Price * Item_Quantity`) was used and summed per `competition_price_bracket`.
3.  **Merge and Visualize:** The results were merged to facilitate a direct comparison and visualized using a clustered bar chart.

### Revenue Breakdown:

The table below shows the calculated total revenue for our store and the competition within each defined price bracket:

| Price Bracket | Own Store Revenue | Competition Revenue |
| :------------ | :---------------- | :------------------ |
| 0-50          | 4,244,868         | 8,105,461           |
| 51-100        | 696,846,300       | 525,276,700         |
| 101-150       | 1,265,876,000     | 1,308,102,000       |
| 151-200       | 1,291,495,000     | 1,138,420,000       |
| 201-250       | 1,748,658,000     | 1,732,585,000       |
| 251-300       | 1,470,184,000     | 2,074,060,000       |
| 301-350       | 124,037,200       | 179,162,100         |
| 351-400       | 0                 | 0                   |
| 401-450       | 0                 | 0                   |
| 451-500       | 0                 | 0                   |

**Key Observations from Visual Analysis:**

The clustered bar chart provides a clear visual comparison of revenue performance across price segments:

  * **Dominant Brackets:** Both our store and the competition generate the bulk of their revenue in the mid-to-higher price brackets, particularly from `101-150` up to `251-300`.
  * **Competitive Strengths:** Our store shows strong revenue performance in the `151-200` and `201-250` brackets, where we match or slightly exceed competition.
  * **Competitor Dominance:** The competition significantly outperforms our store in the `251-300` price bracket, which is their highest revenue segment. They also have higher revenue in the `0-50` and `301-350` brackets.
  * **Strategic Opportunity:** The substantial revenue generated by competitors in the `251-300` bracket represents a key opportunity for our dynamic pricing strategy to make items more competitive and capture a larger share of this high-value segment.

![Total revenue by price bracket.png](https://github.com/Phenomkay/Price-Optimization/blob/5ad9423df183a22209ff3fc0b082a0f72aec5ec7/Total%20revenue%20by%20price%20bracket.png)


## Price Elasticity of Demand Calculation & Analysis

Price Elasticity of Demand (PED) is a fundamental concept in this project, measuring the responsiveness of the quantity demanded to a change in an item's price. A negative elasticity indicates that as price increases, demand decreases, and vice versa. Understanding this metric at an item level is crucial for making informed dynamic pricing decisions to maximize revenue.

### Methodology:

The price elasticity for each item was calculated using the following formula:

$\\text{Elasticity} = \\frac{%, \\text{Change in Quantity Demanded}}{%, \\text{Change in Price}}$

The steps involved in the calculation were:

1.  **Data Sorting:** The DataFrame was sorted by `Item_ID`, `Store_ID`, and `Fiscal_Week_ID` to ensure correct sequential calculation of percentage changes.
2.  **Percentage Change Calculation:** The percentage change in `Price` and `Item_Quantity` was computed for each item within each store across consecutive fiscal weeks using `pct_change()`.
3.  **Elasticity Calculation:** The `price_elasticity_item` was then derived by dividing the `quantity_change_item` by the `price_change_item`.
4.  **Handling Edge Cases:** Infinite values (resulting from division by zero, i.e., no price change) and `NaN` values (from initial entries or no quantity/price change) were handled by replacing them with `NaN` and then dropping rows where elasticity could not be computed.

### Distribution of Calculated Price Elasticity:

The distribution of the calculated price elasticity values provides a comprehensive view of how sensitive different items are to price changes.

**Descriptive Statistics of Price Elasticity:**

```
count      8234.000000
mean         -0.461013
std          35.573810
min       -1455.268409
25%          -0.796818
50%           0.000000
75%           0.765515
max        1127.613636

```

  * **Valid Values:** A total of 8,234 valid elasticity values were calculated after handling missing data.
  * **Mean Elasticity:** The average elasticity is approximately -0.46. While this number is slightly skewed by outliers, the negative sign generally indicates that as prices increase, quantity demanded tends to decrease across the dataset.
  * **Median Elasticity:** The median elasticity is 0.00, suggesting that for half of the observed price changes, there was no corresponding quantity change, or that elasticity values are heavily concentrated around zero due to many instances of stable price/quantity.
  * **Wide Range & Variability:** The standard deviation is very high (35.57), and the range from min (-1455) to max (1127) is extremely broad. This indicates a wide spectrum of demand responsiveness, with some items being highly elastic (very sensitive to price changes) and others very inelastic (less sensitive). The presence of positive elasticities suggests either anomalous data, Giffen goods (rare), or luxury Veblen goods, or more commonly, uncaptured variables influencing demand.

The histogram below visually represents this distribution, showing a concentration around zero and negative values, with long tails representing highly elastic or inelastic behaviors. The red dashed line at 0 indicates unitary elasticity.

![distribution of calculated price elasticity of demand.png](https://github.com/Phenomkay/Price-Optimization/blob/5ad9423df183a22209ff3fc0b082a0f72aec5ec7/distribution%20of%20calculated%20price%20elasticity%20of%20demand.png)


## Dynamic Pricing Model Implementation & Simulation

With the price elasticity of demand calculated, the next critical step was to implement a dynamic pricing strategy and simulate its potential impact on revenue and quantity sold. This phase translates the insights from elasticity into actionable pricing adjustments.

### Methodology:

1.  **Aggregating Elasticity:**
    * To apply elasticity effectively, the average price elasticity (`avg_item_elasticity`) was calculated for each unique `Item_ID`. This aggregation provides a stable measure of an item's overall demand responsiveness across different stores and weeks.
    * For items where elasticity could not be computed (due to insufficient price/quantity changes), the overall average elasticity from the dataset was used to impute missing values, ensuring all items could be included in the simulation.

2.  **Defining Dynamic Pricing Rules:**
    * A strategic approach was developed to adjust prices based on an item's calculated average elasticity. The core principle is to increase prices for items with inelastic demand (where consumers are less sensitive to price changes) and decrease prices for items with elastic demand (where consumers are highly sensitive).
    * The following simplified rules were applied to determine the `dynamic_price`:
        * If `avg_item_elasticity` is between **-1 (inelastic) and 0**: Price was increased by **5%**. (Expected to increase revenue)
        * If `avg_item_elasticity` is **less than -1 (elastic)**: Price was decreased by **5%**. (Expected to increase quantity significantly, potentially boosting revenue)
        * If `avg_item_elasticity` is **greater than or equal to 0 (positive elasticity)**: Price was increased by **2%**. (Acknowledging complex demand patterns or luxury goods, a conservative increase was applied).

3.  **Simulating New Quantity:**
    * The projected `simulated_item_quantity` was calculated using the fundamental elasticity formula:
        $\text{New Quantity} = \text{Old Quantity} \times (1 + \text{Elasticity} \times \% \text{Change in Price})$
      
    * The `% Change in Price` was derived from the difference between the `dynamic_price` and the `Price` (original price).
      
    * Quantities were capped at a minimum of zero to ensure realistic outcomes.

4.  **Calculating Dynamic Revenue:**
    * The final `dynamic_revenue` for each record was then computed by multiplying the `dynamic_price` by the `simulated_item_quantity`.

### Simulated Impact Comparison:

The simulation's primary goal was to compare the total revenue and quantity sold under the new dynamic pricing strategy against the existing pricing.

| Metric              | Existing Pricing | Dynamic Pricing (Simulated) |
| :------------------ | :--------------- | :-------------------------- |
| Total Revenue       | 6,601,342,000    | 7,266,980,000               |
| Total Quantity Sold | 39,961,130       | 43,720,290                  |

**Key Simulation Results:**

The simulation projects a significant positive impact from the dynamic pricing model:
* **Total Revenue Increase:** A projected increase in total revenue from **6.60 billion to 7.27 billion**. This represents a substantial financial gain.
* **Total Quantity Sold Increase:** A projected increase in total quantity sold from **39.96 million to 43.72 million units**. This indicates that the strategy is expected to boost sales volume alongside revenue.

These results validate the potential of the dynamic pricing model to achieve the project's objectives of maximizing both revenue and sales quantity.


## Data Integration and Final Refinement

After simulating the dynamic pricing impact, the calculated `dynamic_price`, `simulated_item_quantity`, and `dynamic_revenue` needed to be seamlessly integrated back into the main dataset. This phase also involved rigorous data cleaning and deduplication to ensure the final DataFrame was robust, accurate, and ready for comprehensive analysis and dashboarding.

### Methodology:

1.  **Date Format Standardization:** `Fiscal_Week_ID` columns in both the original and simulated DataFrames were explicitly converted to datetime format, a prerequisite for accurate merging.
2.  **Preparing Simulated Results:** The newly calculated dynamic results (`dynamic_price`, `simulated_item_quantity`, `dynamic_revenue`) were extracted into a separate, temporary DataFrame. Importantly, duplicates based on the primary keys (`Fiscal_Week_ID`, `Store_ID`, `Item_ID`) were removed from this temporary DataFrame to prevent unintended data duplication during the merge.
3.  **Merging Datasets:** A `left merge` was performed to combine the original DataFrame with the calculated dynamic pricing results, ensuring that all original records were preserved and augmented with their simulated counterparts.
4.  **Addressing Data Duplication:** Exact duplicate rows that might have been introduced during previous processing or were inherently present in the source were identified and removed using `drop_duplicates()`, ensuring each unique transaction record was represented only once.
5.  **Resolving Column Naming Conflicts:** A common challenge in Pandas merges is the automatic addition of suffixes (e.g., `_x`, `_y`, `_original`, `_calculated`) when columns with the same name exist in both DataFrames. Extensive logic was implemented to systematically:
    * Prioritize and select the correct `dynamic_price`, `simulated_item_quantity`, and `dynamic_revenue` columns from the newly calculated results.
    * Rename columns with `_calculated` or `_original` suffixes to their clean base names.
    * Remove any redundant or temporary columns (e.g., `_x`, `_y`, or unneeded `_original`/`_calculated` versions), resulting in a single, well-named column for each metric.
6.  **Final Verification and Export:** The final DataFrame's shape and column structure were verified to confirm the successful integration and cleaning. The consolidated and refined dataset was then saved to a new CSV file (`Competition_Data_Dynamic_Pricing_Updated.csv`) for future use and reproducibility.

This meticulous data integration and refinement process ensures the integrity and usability of the final dataset, which serves as the foundation for the project's analytical conclusions and the interactive Power BI dashboard.

![Price Optimization_page-0001](https://github.com/Phenomkay/Price-Optimization/blob/7fbcd4f9e80025e3602db8c1ee5855a9219a1d89/Price%20Optimization_page-0001.jpg)


## Integrating Item-Level Elasticity

To ensure that every transaction record in the primary dataset (`df`) is enriched with its corresponding average item-level price elasticity, a final merge operation was performed. This integration is vital for subsequent detailed analysis, performance evaluation, and the creation of the interactive dashboard, as it directly links the simulated outcomes with their underlying elasticity drivers.

### Methodology:

1.  **Preparation of Elasticity Data:** A dedicated DataFrame containing unique `Item_ID` and their respective `avg_item_elasticity` values (calculated in the previous "Dynamic Pricing Model Implementation & Simulation" step) was prepared. This ensures a one-to-one mapping for merging.
2.  **Merging:** A `left merge` was executed, using `Item_ID` as the common key, to bring the `avg_item_elasticity` column into the main `df`. This preserves all original transaction records while appending the relevant elasticity information.

### Verification:

The merge operation successfully added the `avg_item_elasticity` column to the main DataFrame. A subsequent check confirmed that there were **no missing values** in the `avg_item_elasticity` column, indicating that all items in the dataset successfully received their calculated elasticity value.


## Granular Price Scenario Simulation with Elasticity

To gain a deeper, item-level understanding of how revenue and quantity might respond to various price adjustments, a granular simulation was conducted. This step leverages the calculated average price elasticity for each item to project outcomes under different hypothetical price changes.

### Methodology:

A dedicated function, `simulate_price_scenarios_with_elasticity`, was developed to perform this simulation for each record in the dataset. For every individual item transaction, the model explored a range of relative price adjustments (e.g., -20%, -10%, 0%, +10%, +20%).

For each `percentage_change` scenario:
1.  **New Price Calculation:** A `new_price` was determined by applying the percentage change to the `Original_Price`.
2.  **Simulated Quantity:** The `simulated_quantity` was then calculated using the core price elasticity formula:
    $\text{Simulated Quantity} = \text{Original Quantity} \times (1 + \text{Average Item Elasticity} \times \% \text{Change in Price})$
    A crucial safeguard `max(0, ...)` was applied to ensure that the simulated quantity does not drop below zero, maintaining practical realism.
3.  **Simulated Revenue:** Finally, the `simulated_revenue` for each scenario was computed by multiplying the `New_Price_Simulated` by the `Simulated_Quantity`.

These simulated results, including the `Original_Price`, `Original_Quantity`, `Original_Revenue`, `Avg_Item_Elasticity`, `Pct_Change_Applied`, `New_Price_Simulated`, `Simulated_Quantity`, and `Simulated_Revenue`, were collected into a new DataFrame, `sim_df_elasticity`.

### Simulation Results Overview:

The `sim_df_elasticity` DataFrame contains a comprehensive set of projected outcomes for each item under various pricing scenarios. This granular data allows for detailed analysis of revenue and quantity responses across the entire product catalog.

The `sim_df_elasticity` DataFrame, with its 500,000 entries (100,000 original rows * 5 scenarios), forms the backbone for identifying optimal pricing strategies and demonstrating the potential revenue lift.


## Model Evaluation: Item-Level Price-Revenue Simulation

To concretely demonstrate the impact of the dynamic pricing model, a detailed simulation of potential revenue changes was performed for a sample item across various price adjustment scenarios. This item-level analysis highlights how the calculated price elasticity guides optimal pricing decisions for individual products.

### Methodology:

A specific item (identified by its original DataFrame `Index`) was selected for this granular analysis. The `Simulated_Revenue` for this sample item was plotted against the `Applied_Price_Change (%)` (ranging from -20% to +20%). The `Original Revenue` at a 0% price change was explicitly marked to provide a baseline for comparison.

### Key Observation:

The plot below illustrates the expected revenue response to different price changes for a particular item, considering its specific price elasticity. For the displayed sample item:

  * The curve shows that revenue initially increases with a price decrease (e.g., at -20% and -10% changes) from the original price point. This indicates that the item's demand is likely elastic in this range, meaning a price reduction leads to a proportionally larger increase in quantity, thus boosting total revenue.
  * The `Original Revenue @ 0% Change` point is marked, serving as the benchmark.
  * As the price increases (e.g., at +10% and +20% changes), the `Simulated Revenue` begins to significantly decline. This confirms that for this item, pushing the price too high leads to a substantial drop in demand, resulting in lower overall revenue.

This individual item simulation provides a tangible example of how understanding price elasticity can guide decisions to either increase or decrease prices to maximize revenue for specific products within the portfolio.

![simulated revenue vs price change](https://github.com/Phenomkay/Price-Optimization/blob/7fbcd4f9e80025e3602db8c1ee5855a9219a1d89/simulated%20revenue%20vs%20price%20change.png)


## Model Performance Metrics: Revenue Lift

To quantify the direct financial benefit of implementing the dynamic pricing model, two key performance metrics were calculated: `dynamic_lift` and `lift_percent`. These metrics measure the absolute and percentage increase in revenue, respectively, attributable to the proposed dynamic pricing strategy compared to existing pricing.

### Calculation of Lift:

* **`dynamic_lift`**: Represents the absolute increase in revenue and is calculated as the difference between the `dynamic_revenue` (simulated revenue under dynamic pricing) and the `Revenue` (original revenue from existing pricing).
    $\text{Dynamic Lift} = \text{Dynamic Revenue} - \text{Original Revenue}$
* **`lift_percent`**: Represents the percentage increase in revenue and is calculated by dividing the `dynamic_lift` by the `Revenue` (original revenue) and multiplying by 100.
    $\text{Lift Percent} = \frac{\text{Dynamic Lift}}{\text{Original Revenue}} \times 100$

### Descriptive Statistics of `lift_percent`:

The `lift_percent` column provides a granular view of revenue improvement across all individual records. Its descriptive statistics offer insights into the overall performance and variability of the lift achieved.

```
count    100000.000000
mean         13.830897
std          39.650970
min         -68.868322
25%           2.371116
50%           3.839419
75%           7.718971
max         323.438186
Name: lift_percent, dtype: float64
```

**Key Observations:**

* **Overall Positive Lift:** The **mean `lift_percent` is approximately 13.83%**, indicating that, on average, the dynamic pricing strategy is projected to increase revenue by nearly 14%. This aligns with the overall simulated revenue increase observed earlier (from 6.60 billion to 7.27 billion), which represents an approximate 10.88% increase.
* **Median Lift:** The median lift is 3.84%, suggesting that at least half of the transactions are expected to see a revenue increase of nearly 4% or more.
* **Variability:** The standard deviation of 39.65 and the wide range from a minimum of -68.87% to a maximum of 323.44% highlight significant variability in the revenue lift across different items and transactions. While the strategy generally yields positive results, some items may experience a decrease in revenue, while others see very substantial gains. This variability underscores the importance of the item-level elasticity calculation.

These metrics provide a quantitative measure of the success and potential impact of the dynamic pricing strategy.


## Model Performance Visualization: Distribution of Revenue Lift

To provide a comprehensive view of the dynamic pricing model's impact, the distribution of the `lift_percent` (percentage revenue improvement) across all transactions was visualized. This histogram clearly illustrates the spread and frequency of different levels of revenue increase or decrease achieved by the simulated strategy.

### Visualization:

A histogram with a Kernel Density Estimate (KDE) was generated for the `lift_percent` column.

### Key Observations:

  * **Positive Skew and Overall Gain:** The distribution is heavily skewed to the right, with a large concentration of data points showing a positive revenue lift. This visually reinforces the mean `lift_percent` of approximately 13.83% calculated earlier, indicating that the dynamic pricing strategy generally leads to increased revenue.
  * **Concentration Around Small Positive Lift:** A significant portion of transactions experience a modest but positive revenue improvement, clustering between 0% and 10-20% lift.
  * **Negative Lift Instances:** While the overall trend is positive, there are instances where the `lift_percent` is negative, indicating a decrease in revenue for some transactions. These are typically items where the pricing strategy, based on their elasticity, may have led to a less optimal outcome, or where the elasticity calculation itself was influenced by sparse data.
  * **High-Impact Outliers:** The long tail extending to the right shows a smaller number of transactions that experienced exceptionally high revenue improvements (e.g., above 100% or even 300%). These represent significant opportunities identified by the model.

This visualization provides clear evidence of the model's overall effectiveness in driving revenue growth, while also acknowledging the variability in outcomes across different items.

![Revenue improvement from dynamic pricing](https://github.com/Phenomkay/Price-Optimization/blob/7fbcd4f9e80025e3602db8c1ee5855a9219a1d89/Revenue%20improvement%20from%20dynamic%20pricing.png)


## Challenges and Solutions

Throughout the development of the dynamic pricing model, several challenges were encountered, primarily related to data quality, consistency, and the inherent complexities of real-world retail data. Addressing these challenges was crucial for building a robust and reliable model.

### 1. Data Anomalies and Inconsistencies in Sales Metrics
* **Challenge:** Initial data inspection revealed inconsistencies where `Sales_Amount` (after discounts) was unexpectedly greater than `Sales_Amount_No_Discount` (before discounts). Furthermore, the `Sales_Amount_No_Discount` column did not consistently align with the direct calculation of `Price * Item_Quantity`, indicating potential data capture or definition issues.
* **Solution:** To establish a reliable base revenue metric, a new `Revenue` column was explicitly defined as `Price * Item_Quantity`. This ensured a consistent and logical foundation for all subsequent analyses and the dynamic pricing model.

### 2. Missing Elasticity Values and Extreme Outliers
* **Challenge:** Calculating price elasticity involves percentage changes, which can result in `NaN` (Not a Number) values when there's no price change or division by zero, and `infinite` values in cases of very small price changes leading to large quantity shifts. Additionally, the overall elasticity distribution showed a very wide range and high standard deviation.
* **Solution:**
    * `inf` and `-inf` values were explicitly replaced with `NaN`, and then rows with `NaN` elasticity were dropped for the calculation phase, ensuring only valid numerical elasticities were used.
    * For items that still lacked a calculated elasticity (e.g., due to insufficient historical price changes), the overall average elasticity of the dataset was imputed to allow all items to be included in the dynamic pricing simulation.
    * During quantity simulation, `simulated_item_quantity` was capped at `max(0, ...)` to prevent unrealistic negative quantities that could arise from extreme elasticity values.

### 3. Data Integration and Column Management after Merges
* **Challenge:** Merging calculated dynamic pricing results and elasticity values back into the main DataFrame often leads to duplicate rows or ambiguous column names (e.g., `_x`, `_y`, `_original`, `_calculated` suffixes added by Pandas). This complicates subsequent analysis and dashboard creation.
* **Solution:** A rigorous data integration process was implemented:
    * `Fiscal_Week_ID` was standardized to datetime format for accurate merging.
    * Explicit `drop_duplicates()` calls were used to remove any exact duplicate rows.
    * Comprehensive logic was applied to systematically identify, select, and rename the correct columns (e.g., `dynamic_price`, `simulated_item_quantity`, `dynamic_revenue`) while dropping redundant or temporary columns generated during the merge process, ensuring a clean and usable final dataset.

### 4. Simulating Realistic Price-Quantity Responses
* **Challenge:** Applying a simple elasticity formula can sometimes lead to unrealistic simulated quantities, especially with volatile or extreme elasticity values.
* **Solution:** Beyond capping simulated quantities at zero, the dynamic pricing rules were designed to be reasonable (e.g., +/- 5% price changes). The use of *average item elasticity* helped to smooth out individual week-to-week fluctuations, providing a more stable and actionable elasticity measure for broader pricing decisions.


## Actionable Insights and Recommendations

The comprehensive analysis and simulation of an elasticity-driven dynamic pricing model provide several key actionable insights and recommendations for optimizing sales and revenue:

* **Significant Revenue Optimization Opportunity:** The simulation projects a substantial overall increase in total revenue by approximately **10.88% (from 6.60 billion to 7.27 billion)** and a **9.79% increase in total quantity sold**. This clearly indicates the high potential of implementing dynamic pricing based on item-level elasticity.
* **Strategic Price Adjustments based on Elasticity:**
    * For items exhibiting **inelastic demand** (average elasticity between -1 and 0, with an overall mean elasticity of -0.46), the model recommends **increasing prices (e.g., by 5%)**. Consumers for these items are less sensitive to price changes, allowing for revenue maximization without significant loss in quantity.
    * For items with **elastic demand** (average elasticity less than -1), the model suggests **decreasing prices (e.g., by 5%)**. This strategy is designed to stimulate a proportionally larger increase in quantity demanded, ultimately boosting overall revenue for these sensitive products.
* **Targeted Competitive Strategy:**
    * Our store's prices are generally lower than those of the competition. While this maintains competitiveness, the analysis of revenue by price bracket reveals that competitors significantly outperform our store in the **251-300 price bracket**.
    * **Recommendation:** Focus dynamic pricing efforts on items within the 251-300 range that exhibit favorable elasticity. Strategic price adjustments in this segment could capture a larger share of the competitor's high-value revenue, which is their highest revenue segment.
* **Continuous Monitoring and Iteration:** Dynamic pricing is not a static solution. It requires continuous monitoring of actual sales performance against simulated projections.
    * **Recommendation:** Regularly re-calculate price elasticity as new sales data becomes available to adapt to changing market conditions and consumer behavior. Refine pricing rules and thresholds based on real-world outcomes.
* **A/B Testing for Validation:**
    * **Recommendation:** Implement controlled A/B tests in select stores or for specific product categories to validate the simulated price changes and their impact before a full-scale rollout. This minimizes risk and provides empirical evidence of the model's effectiveness.
* **Consideration of External Factors:** While the current model focuses on price and quantity, future enhancements could incorporate external factors such as promotional activities, competitor specific sales events, seasonality beyond fiscal week, and product lifecycle stages to further refine pricing decisions.

## Conclusion

This project successfully developed and simulated an **elasticity-driven dynamic pricing model** designed to optimize revenue and quantity sold for the retail store. By meticulously cleaning and analyzing historical sales data, calculating item-level price elasticity of demand, and simulating the impact of elasticity-informed price adjustments, the project has demonstrated a clear path to significant financial improvement.

The simulation results are compelling, projecting an **overall revenue increase of 10.88% and a quantity increase of 9.79%**. These findings underscore the immense value of leveraging data analytics and economic principles to make precise and impactful pricing decisions.

The comprehensive Power BI dashboard serves as the final, actionable output, providing stakeholders with an interactive tool to monitor key performance indicators, visualize price and revenue trends, compare against competition, and track the projected lift from the dynamic pricing strategy. This project lays a strong foundation for implementing a data-driven pricing strategy that can adapt to market dynamics and continuously maximize business performance.

Thank You

