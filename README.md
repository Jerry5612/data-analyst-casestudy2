# Data Analytics Capstone: Global Mart Case Study
## Introduction
In this case study, I will perform many real-world tasks of a junior data analyst at a fictional company, Global Mart. In order to answer the key business questions, I will follow the steps of the data analysis process: Ask, Prepare, Process, Analyse, Share, and Act.

Quick links:

Data Source: [kaggle_dataset](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci) [accessed on 20/11/25]

SQL Queries:
01. Data Exploration
02. Data Cleaning
03. Data Analysis

Data Visualisations: [Tableau](https://public.tableau.com/app/profile/jerry.soh/viz/ecommerce-customertype-casestudy/ConversionScatterPlotRFMTargeting)
## Background
### Global Mart 
A UK-based online non-store retailer specialising in unique, all-occasion gift-ware and household accessories. This specialisation allows the platform to maintain its uniqueness while generating revenue through product sales, premium subscriptions plans (like ‘Mart Priority’), and loyalty programmes. The platform continues to guarantee expansive product availability and reliable logistical services, ensuring its appeal and accessibility to a variety of consumers.

Until now, Global Mart’s marketing strategy relied on building general awareness and appealing to broad consumer segments. The vast majority of Global Mart’s customers are Regular Shoppers (making standard purchases), but a significant portion of the platform’s volume comes from a specific segment: Wholesalers and Small Businesses who make bulk purchases of products for resale or commercial use. The highly engaged segment comprising the Loyalty Tier Customers (who subscribe to 'Mart Priority’) is defined as a group of high-volume Wholesalers, combined with the most frequent General Retail Shoppers, who maintain the recurring subscription. These subscribers utilise exclusive services such as special discount rates, priority delivery, and product reservation.

Global Mart’s financial analysts have concluded that Loyalty Tier Customers contribute more to the platform’s profits than Regular Shoppers due to their higher lifetime value (LTV) and consistent spending habits, particularly the high transaction volume driven by the Wholesalers. Moreno, the marketing director, believes that maximising the number of Loyalty Tier Customers is the key to future profit growth and stability. Rather than creating a marketing campaign that appeals to a broad base of customers, Moreno believes that converting high-potential Regular Shoppers—specifically those Regular Shoppers who exhibit high purchase frequency or large average basket sizes but have not subscribed to Mart Priority and/or engaged in loyalty programmes —into Loyalty Tier Customers is the most desirable strategy, as they are already invested in the Global Mart ecosystem.

Moreno has set a clear goal: Design marketing strategies aimed at converting these high-potential Regular Shoppers into Loyalty Tier Customers. The marketing analyst team, therefore, needs to better understand how Loyalty Tier and Regular Shoppers differ based on transaction metrics, what signals would trigger the conversion, and how digital channels could affect their marketing tactics. Moreno and her team are interested in analysing Global Mart historical transactional data to identify these conversion trends.

### Scenario
I assume the role of Junior Data Analyst working in the marketing analyst team at Global Mart. The marketing director believes the company’s future success depends on maximising the number of Loyalty Tier Customers. Therefore, my team needs to understand how Regular Shoppers and Loyalty Tier Customers engage with Global Mart services respectively. These insights would enable my team to design a new strategy aimed at converting Regular Shoppers into Loyalty Tier Customers. However, GlobalMart executives must approve our recommendations, hence the strategy must be backed up with compelling data insights and professional data visualisations.

## Ask
### Business Task
Devise marketing and product strategies to convert Regular Shoppers into Loyalty Tier Customers to maximise the profits of Global Mart.
### Analysis Questions
Three questions will guide the future marketing programme:
1. How do Loyalty Tier Customers and Regular Shoppers differ in their transactional and behavioral patterns on the platform?
2. What transactional or website behavioral signals would motivate a Regular Shopper to convert into a Loyalty Tier Customer?
3. How can Global Mart use digital channels and in-app promotions to convince Regular Shoppers to become Loyalty Tier Customers?

## Prepare
### Data Source
I will use Global Mart’s historical transaction data to analyse and identify trends which can be downloaded from [kaggle_dataset](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci). The data has been made available by Creative Commons under this [license](https://creativecommons.org/publicdomain/zero/1.0/). This is public data that can be used to explore how different customer types are using Global Mart services. But note that data-privacy issues prohibit me from using customers’ personally identifiable information. This means that I will not be able to connect subscription purchases to credit card numbers to determine how many subscription plans have Regular Shoppers purchased over time. 
### Data Organization
There is 1 Comma-separated values (CSV) file with naming of online_retail_II.csv and it contains all transactions occurring for a UK-based and registered, non-store online retail for 2 years, between 01/12/2009 and 09/12/2011. The Attribute Information are Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID and Country.

### Data Dictionary Table 
#### Transactional and Product Data

The dataset includes the Invoice No., a nominal field formatted as a 6-digit integral with a 'C' prefix for cancellations. This serves as the unique transaction identifier, crucial for cleaning cancelled orders and analyzing overall transaction volume. Stock Code / Description provides both a 5-digit nominal integral code and the Item Name, used for product identification, inventory analysis, and sales volume tracking. The Quantity field is a numeric integer representing the volume of goods sold, which is utilized for calculating total sales and identifying top-selling items.

#### Temporal, Financial, and Customer Data

Invoice Date is recorded as a numeric timestamp, providing granular temporal data necessary for time-series analysis and identifying peak sales periods. The Price is a numeric value in Sterling (£), which is essential for accurate revenue calculation and margin analysis. Customer ID is a nominal 5-digit integral used as a unique customer identifier, vital for market segmentation and purchase frequency analysis (RFM modeling). Finally, Country is a nominal field providing the geographical dimension needed for market analysis and identifying high-value regions.

## Process
BigQuery is used to clean this dataset and perform analysis on its data.
Reason:
Due to its inability in managing large amounts of information, a Microsoft Excel worksheet can only handle up to 1,048,576 rows of data. Since the Global Mart historical transactional dataset has 1,067,371 rows of data, it is essential to use a platform like BigQuery that supports huge volumes of data.

### Data Exploration
SQL Query: Data Exploration

Before cleaning the data, I am familiarising myself with the data to find the inconsistencies.

1. The table below shows all the column names and their data types. The Invoice and StockCode columns are my primary keys.
 
![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image3.png)

Reason for 2 Primary Keys:
A Composite Primary Key was established to accurately define the granularity of the transactional data. The Invoice column alone is insufficient, as a single order often contains multiple line items, resulting in repeated Invoice IDs. By combining Invoice and StockCode (Product ID), a unique identifier is created for each row, thereby ensuring the integrity of product-specific metrics and enabling reliable query performance.

2. The following table shows the number of null values in each column.
 
![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image13.png) 

The point of this query is to identify Invalid Data (Missing & Non-Sales) for removal. This is because Transactional Data often contains missing customer IDs, Invoice Numbers, Descriptions or has non-sales rows (like cancelled orders or service charges leading to quantity values of goods sold being less than 1 or their corresponding unit price being less than £0).

### Data Cleaning
SQL Query: Data Cleaning

1. A base of valid sales transaction data will be created with rows that cannot be determined as successful purchases removed including:
*Rows missing a Customer_ID (cannot link to a customer).
*Rows missing a Description (cannot analyse product category/type).
*Rows with Quantity ≤0 or Price ≤0 (these are usually returns, cancellations, or errors, and should be excluded from profit/revenue modeling).

2. The following screenshot shows a table of valid sales transaction data with the Line_Total (Monetary Value/ Revenue generated per Invoice ID and StockCode) calculated as well as potential duplicate entries identified and removed.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image11.png)

A composite key is used (Invoice and StockCode) along with a window function (ROW_NUMBER()) to flag and keep only one instance of any duplicate row. As a result, the number of rows in the dataset has been reduced from 1 067 371 to 779 421. 

3. To enable robust Long-Term Value (LTV) analysis, the dataset was subjected to Feature Engineering, thereby creating crucial analytical dimensions. These newly constructed metrics were designed specifically to facilitate customer segmentation and time-based trend analysis, directly supporting the computation and prediction of LTV.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image7.png)

After querying BigQuery, a final table using cleaned table named customer_rfm_metrics is generated as shown above to calculate Recency* (days since last purchase), Frequency (total unique invoices), and Monetary Value (total expenditure) for every customer.

### Technical Definition of Reference Date/Recency
In a Recency, Frequency and Monetary Value (RFM) analysis using a historical dataset, the calculation of Recency (time elapsed since a customer's last purchase) requires a fixed point in time, known as the Reference Date or the analytical “Today".

Recency (Days) = Reference Date − Customer’s Last Purchase Date

To ensure the analysis is comprehensive and fair across all customers, the Reference Date is defined as the day immediately following the latest transaction found within the Global Mart dataset.

### Justification and Rationale
1. Consistency: Using the day after the last recorded transaction (MAX(Sale_Date)+1 day) ensures that every customer's Recency is measured against the same, fixed point in time. This allows for a fair and accurate comparison of purchasing behaviour across the entire customer base.
2. Completeness: By referencing the final date in the dataset, the model incorporates the entirety of the available transactional history, maximising the data used for the segmentation analysis.
3. Reproducibility: This explicit definition provides transparency for the methodology, ensuring that any future analyst can replicate the RFM segmentation exactly.

## Analyse
### Data Analysis
SQL Query: Data Analysis

The analysis stage begins with the RFM Analysis, as this is the industry-standard method for segmenting customers based on their transactional behavior. This step is critical as it transforms raw line-item data into meaningful, customer-level metrics that directly address Question 1 (How do Loyalty Tier Customers and Regular Shoppers differ in their transactional and behavioral patterns?) and lay the foundation for answering Question 2 (What transactional or website behavioral signals would motivate a Regular Shopper to transition into a Loyalty Tier Customer?).

The RFM calculation generates a single row of summarized purchase history per unique customer, utilizing three primary and two supplementary dimensions. Recency (R) is defined as the days since the customer's last purchase, and its primary purpose is to measure engagement and activity level. Frequency (F) represents the total number of unique orders or invoices, serving to measure customer loyalty and habit. The final primary metric, Monetary (M), is the total value of all purchases (Total Expenditure), which measures the customer's overall financial contribution and value to Global Mart.

Two supplementary metrics provide further depth: Avg. Basket Size calculates the average transaction value to help identify high-volume purchasing, particularly wholesaler behavior. Lastly, Total Items tracks the total count of all products purchased, measuring the total volume handled by the customer.

The key challenge is that the dataset does not have a simple "customer_type" column or equivalent. Therefore, I must define the Loyalty Tier segment based on the criteria in my project’s background:

"The highly engaged segment comprising the Loyalty Tier Customers (who subscribe to 'Mart Priority’) is defined as a group of high-volume Wholesalers, combined with the most frequent General Retail Shoppers, who maintain the recurring subscription. These subscribers utilise exclusive services such as special discount rates, priority delivery, and product reservation."

I will use the Frequency_Orders and Monetary_Value metrics to identify these high-value customers.

This query creates two new tables:
1. A temporary table to determine the high-value thresholds (such as the 90th percentile for both Frequency and Monetary Values) to provide the technical definition for "Loyalty Tier Customer."

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image14.png)

2. The final table applying those thresholds to compare the two segments by assigning a Customer_Tier flag (Loyalty Tier or Regular Shopper) to every customer based on those thresholds. This query uses the provided thresholds (F90​=13 and M90​=5467.82) to create the final segmentation table and calculate the aggregated differences between the two groups to present evidence and insights showing how the two groups differ (eg. average monetary value, average basket size, average items purchased), addressing Question 1.

 ![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image10.png)

The comparison between Regular Shoppers (RS) and Loyalty Tier Customers (LTC) reveals the latter to be a highly elite and valuable segment. The Total Customer count shows LTCs are a small segment, consisting of only 442 customers compared to 5,436 RS (a 12:1 ratio). Despite their size, LTCs are extremely consistent, with an Avg. Frequency Orders of 33.2 orders, which is 8.1 times higher than the 4.1 orders placed by RS, confirming they are highly engaged in Global Mart services.

Financially, the disparity is dramatic: the Avg. Monetary Value for LTCs is £21,414.63, an astounding 14.7 times higher than the RS value, demonstrating that LTCs contribute disproportionately high revenue. This high spend is supported by a 1.6 times higher Avg. Basket Size (£585.41 vs. £368.90), suggesting greater volume or wholesale purchasing behavior. Furthermore, LTCs exhibit far superior Avg. Recency, purchasing on average every 32.0 days, which is 6.7 times more recently than the Regular Shopper's 215.0 days.

### Key findings for Question 1
Loyalty Tier Customers (LTCs) are the most valuable asset, generating nearly 15 times the total revenue of a Regular Shopper (RS), primarily through extreme purchase frequency (8x higher) and significantly larger average order values. The conversion goal is highly justified, as moving an RS to the LTC tier results in an exponential increase in Global Mart’s profits.

## The next step is to answer Question 2: What transactional or website behavioral signals would motivate a Regular Shopper to transition into a Loyalty Tier Customer?

Identifying the High-Potential Regular Shoppers is crucial, as this segment represents future growth—individuals who possess high-value traits but do not meet the combined top-tier criteria (F ≥ 13 AND M ≥£5467.82). 

To accurately capture this potential, I defined a "High Basket Size Signal." This data-driven threshold is established by running a query to find the 75th percentile of Average Basket Size across all Regular Shoppers. The resulting threshold defines the High Basket Size Signal as any average basket size greater than £389.78.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image9.png)

Thereafter, I ran another query to generate the final list of High-Potential Regular Shoppers that Global Mart’s marketing campaign will target based on the two key signals: High Frequency (frequent shoppers) and High Average Basket Size (wholesalers/bulk buyers).

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image1.png)

### Interpretation of Conversion Signals
The output of the query gave me a list of customers to target. The two target conversion signals identified in the analysis are the critical triggers for Global Mart’s marketing strategy which correlate with distinct customer priorities:

1. High Basket Size Shoppers (Average Order Value (AOV)>£389.78): These customers exhibit Wholesaler-like behavior where the primary motivation is profit maximisation and cost reduction. Therefore, the campaign targeting this group will strategically highlight the Special Discount Rates benefit, as it directly addresses their monetary motivation.

2. High Frequency Shoppers (F≥7 Orders): These customers are already engaged and loyal. Their motivation is driven by service convenience, exclusivity, and reward. The campaign for this group will strategically highlight the Priority Delivery and Product Reservation benefits, positioning the subscription as an elevated, exclusive service tier for their loyalty.

### Change in marketing strategy for Mart Priority Subscription Service: Segmenting the Value Proposition
Following the interpretation of Conversion Signals, it is concluded that while the Mart Priority subscription service offers a comprehensive bundle of benefits—including special discount rates, priority delivery, and product reservation—the core marketing strategy requires segmenting the value proposition to effectively motivate the different high-potential segments. 

Next, I ran a query to generate a table to show the total count of customers for each Conversion_Signal group. This will tell the size of each target segment as seen in the picture below.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image8.png)

### Key findings for Question 2
The analysis confirms that High-Potential Regular Shoppers are almost equally divided into two distinct groups, requiring tailored marketing approaches:

1. The first segment, identified by the High Basket Size Signal, comprises 1,087 potential customers. Their primary behavior indicates they are Wholesalers or Bulk Buyers who prioritize volume and cost savings. The strategic goal is to convert this group to the Mart Priority Discount Rate, directly appealing to their motivation for monetary benefits.

2. The second segment, defined by the High Frequency Signal, consists of 1,079 customers. Their behavior points to them being highly engaged and loyal retail shoppers who place a high value on excellent service and convenience. The strategic goal for this cohort is to convert them to Mart Priority Exclusive Services, leveraging benefits centered on priority and service to align with their primary motivation.

## The next step is to answer Question 3: How can Global Mart use digital channels and in-app promotions to convince Regular Shoppers to become Loyalty Tier Customers?

The success of the marketing programme depends on its ability to convert the 2,166 identified High-Potential Regular Shoppers (HPRS) into Loyalty Tier Customers (LTCs). The strategy is a two-pronged digital campaign designed to target the unique motivations derived from the Conversion Signals identified in Question 2 (High Frequency vs. High Basket Size).

### Strategy 1: The Monetary Incentive (Targeting the Wholesaler/Bulk Buyer)
This strategy targets the 1,087 Regular Shoppers identified by the High Basket Size Signal (AOV>£389.78). This segment demonstrates behaviouur focused on volume and cost-efficiency.

1. The core Value Proposition Focus of this tactical plan centers on offering Special Discount Rates. The rationale is that these specific customers are profit-driven, and a guaranteed discount directly offsets their high volume costs, which either maximises their profit margins or reduces their overall expenditure. The promotion will be deployed through the Digital Channel, utilising a Checkout Page Promotion (In-App) and follow-up Email Retargeting, based on the understanding that the highest-impact moment is when the customer is actively making a large purchase.

2. A specific Promotion Example is a Checkout Page Pop-up that triggers when the customer's cart value exceeds a threshold (e.g., £300). The message is designed to offer immediate value: "Unlock a flat 10% discount on THIS order AND all future purchases by subscribing to Mart Priority. Maximise your savings now!" This tactic uses the customer's current large transaction as a direct incentive, highlighting immediate financial savings. The Conversion Trigger is met when the High Potential Retail Shopper (HPRS) crosses the Average Order Value (AOV) threshold of >£389.78 or when the current cart value is >£300, ensuring the cost-saving solution is presented precisely when the data signals bulk purchasing behaviour.

### Strategy 2: The Service & Priority Incentive (Targeting the Loyal/Frequent Shopper)
This strategy targets the 1,079 Regular Shoppers identified by the High Frequency Signal (F≥7 Orders). This segment is already highly engaged and sensitive to rewards, service quality, and exclusive access.

1. The tactical plan for the loyalty-driven segment is built around a Value Proposition Focus on Priority Delivery and Product Reservation. The rationale is that these customers value convenience and exclusivity as recognition for their continued loyalty. The campaign will be executed via the Digital Channel, utilising In-App Home Banners and Post-Purchase Confirmation Screens to target customers when they are actively browsing or have just reinforced their habitual buying behavior.

2. A specific Promotion Example is a Post-Purchase Banner displayed after the customer's 6th order. The message is designed to congratulate them: "Thank you for your loyalty! Upgrade to Mart Priority today for free priority delivery on all future purchases and early access to new gift-ware for your loved ones." This approach leverages their existing purchase count (the Frequency metric) to position the subscription as a natural "next-tier" reward. The Conversion Trigger for this promotion is met when the High Potential Retail Shopper (HPRS) places their 6th or 7th order (meeting the F ≥7 threshold), ensuring the data signal of habitual behaviour is immediately met with a message of recognition and a service upgrade.

## Share
### Data Visualisation
### Data Visualisations: [Tableau](https://public.tableau.com/app/profile/jerry.soh/viz/ecommerce-customertype-casestudy/ConversionScatterPlotRFMTargeting) 

### Visualisation 1: Loyalty Tier vs. Regular Shopper Comparative Metrics
The goal of this visualisation is to quantify the transactional disparity between the highly valuable Loyalty Tier Customers (LTCs) and Regular Shoppers (RSs). I used a Comparative Bar Chart to clearly illustrate the difference in Average Monetary Value and Average Frequency of Orders. 

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image2.png)

### A. Interpretation 

The analysis concludes that the Loyalty Tier Customer (LTC) segment exhibits transactional behavior that is exponentially more valuable than the Regular Shopper (RS) segment. This is quantified by the massive financial difference: the Average LTC Spend of £21,414.63 is 14.7 times higher than the Average RS Spend of £1,454.92. This substantial gap makes conversion mandatory for profit stability and fully justifies dedicated investment in the Mart Priority funnel.

### B. Key Insight 

The core Key Insight is that the value gap confirms the exponential increase in Lifetime Value (LTV) realized by upgrading an RS to the LTC tier. Behaviorally, the difference is driven by consistency, with the Average LTC Frequency of 33.2 Orders being 8.1 times higher than the RS frequency of 4.1 Orders. Based on this, the target KPI must focus on driving transactional behavior to approach the lower-bound LTC thresholds of Frequency (F≥13) and Monetary (M≥£5,467).

### C. Implications For Strategy

The Implications For Strategy necessitate a dual-focus approach: the LTC segment is driven by both high consistency and volume, meaning the strategy must reward both frequency and large average order values. Although LTCs are a small group, comprising only 7.5% of all customers, they contribute a disproportionately high amount to Global Mart’s revenue. Therefore, the strategic mandate is that all marketing resources must be dedicated to these targeted conversion campaigns.

### Visualisation 2: Analysis of Customer Recency Distribution (Engagement Level)
The goal of this visualisation is to compare the distribution of Days Since Last Purchase (Recency) for the Loyalty Tier Customer (LTC) and Regular Shopper (RS) segments. I used a Side-by-Side Box Plot (Two Segments) to visually demonstrate the vast difference in engagement: the LTC distribution is heavily compressed and centered near zero, proving their consistent, highly active purchasing habits, while the RS distribution is wide and dispersed, confirming their sporadic engagement.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image12.png)

### A. Interpretation 

The Recency Box Plot visualization clearly demonstrates that Loyalty Tier Customers (LTCs) are highly engaged and active, while Regular Shoppers (RSs) are largely dormant and inconsistent. The LTC distribution is tightly clustered near zero days, contrasting sharply with the widely dispersed RS distribution. This is quantified by the medians: the LTC Median is 11 Days, while the RS Median is 123 Days (an 11x difference). This Recency data confirms the necessity of a retention focus for LTCs (rewarding their consistent engagement) and a specific reactivation strategy for RSs.

### B. Key Insight and Consistency

A key insight derived from the Upper Quartile (Q3) proves the consistent purchasing habit of the LTCs: 75% of LTCs purchased within the last 29 days. In stark contrast, 75% of RSs have not purchased for 389 days (over a year). The consistently high Recency of the LTCs supports the strategic promotion of Priority Delivery and Product Reservation as highly valued perks for rewarding this consistent engagement (Strategy 2).

### C. Implications For Strategy

The Implications For Strategy highlight that the low Recency (high activity) in the LTC segment signifies they place a high value on speed and reliability. The LTC Interquartile Range (Q3−Q1) is a tight 26 days, while the RS IQR is a wide 358 days. This high consistency means that the High Frequency HPRS, who exhibit similar activity patterns, will be highly sensitive to the service and convenience benefits of Mart Priority, as these perks directly align with their perceived value of time and access.

### Visualisation 3: Customer Segment Size
The goal of this visualisation is to quickly visualise the sheer volume of customers in the Regular Shopper segment relative to the elite Loyalty Tier segment. I used a Treemap to confirm the high concentration of customers within the Regular Shopper segment, providing context for the high revenue generated by the tiny, high-value Loyalty Tier group.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image4.png)

### A. Interpretation and Customer Volume

The Treemap visualization clearly illustrates the customer base breakdown: the Regular Shopper (RS) segment occupies the vast majority of the customer volume (5,436 customers), while the Loyalty Tier Customer (LTC) segment is an elite, small fraction (442 customers) of the total base. This finding reinforces the primary business task: marketing efforts must focus on converting the small, high-value tail of the RS segment, not the low-value mass.

### B. Key Insight and Growth Focus

A key insight is that the 92.5% of customers classified as RS are where potential growth volume resides. However, this large segment contributes disproportionately low revenue, as noted in previous analyses. Conversely, LTCs represent approximately 7.5% of the customer base. Therefore, the focus must fundamentally shift from general customer acquisition metrics to highly targeted conversion metrics specifically within the RS segment.

### C. Implications For Strategy

The Implications For Strategy mandate a significant reallocation of resources. The marketing budget must be heavily allocated towards retention (for existing LTCs) and conversion (for High-Potential RSs), moving away from general, mass-market acquisition efforts. Furthermore, the selection of marketing channels must be refined to effectively reach quality customers over quantity customers.

### Visualisation 4: Analysis of Target Audience Composition (Conversion Signals)
The goal of this visualisation is to confirm the size and dual nature of the High-Potential Regular Shopper (HPRS) target segment, validating the need for a targeted conversion strategy. I used a Bar Chart (Two Segments) to clearly illustrate that the target audience is almost equally split between customers exhibiting the High Frequency Signal and those showing the High Basket Size Signal.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image6.png)

#### *Data Discrepancy
A conflict arose in defining the size of the High Frequency segment due to tool logic differences. While the final strategic count was 1,079, the initial raw count was 1,521, indicating a significant overlap.

Source of Conflict: The difference stemmed from 291 overlapping customers who met both the High Frequency and the High Basket Size thresholds.
Tool Logic:
1. SQL (Hierarchical Logic): The segmentation query was designed to prioritise and assign overlapping customers to the High Frequency group first, yielding the clean count of 1,079.
2. Tableau (Set-Based Logic): This logic double-counted every customer meeting either rule, resulting in the misleading 1,521 number.

Resolution: A separate calculation confirmed that 291 Regular Shoppers qualified for both high-value signals.

#### Strategic Prioritisation and Final Assignment
To ensure a clean, non-overlapping budget split and prevent conflicting marketing messages, a strategic assignment decision was made for the 291 overlapping customers.
1. Prioritisation Rationale: All 291 overlapping customers were strategically assigned to the High Frequency Segment. This choice was based on the premise that high frequency is the strongest indicator of customer loyalty and greater readiness to adopt the Mart Priority service perks.
2. Final Distinct Segments: This methodology resulted in two distinct, non-overlapping target groups, perfectly balancing the budget:

The high-potential audience has been finalized into two distinct and nearly equal target segments for dedicated campaigns. The High Frequency segment totals 1,079 customers. 
The criteria for this group includes the original high-frequency segment plus the 291 customers who were identified as exhibiting overlapping high frequency and high basket size behaviors. 
This segment will receive a message focused on Loyalty and Service and will be funded by Budget A. The High Basket Size segment totals 1,087 customers and consists solely of customers who met the Basket Size rule only. This group will receive a message focused on Monetary benefits and Acquisition funded by Budget B. 

### A. Interpretation 

The analysis confirms that the High-Potential Regular Shopper (HPRS) segment is split almost perfectly into two groups based on their primary transactional signal: 1,087 customers in the High Basket Size segment and 1,079 customers in the High Frequency segment. This finding confirms the audience is not monolithic and requires tailored messaging. Consequently, a single, unified marketing campaign will fail to resonate with half of the target audience, necessitating a dedicated, segmented, two-pronged strategy.

### B. Key Insight 

The balanced split of the segment provides the key insight that justifies a 50/50 budget allocation between the two distinct promotion streams. This optimal distribution ensures resources are maximized across the largest growth opportunity, totaling 2,166 HPRS (approximately 40% of all Regular Shoppers). Ultimately, this conversion campaign is scalable and targets the specific customers most likely to successfully transition into the high-value Loyalty Tier.

### C. Implications For Strategy

The Implications For Strategy demand that the Mart Priority benefits must directly align with the customer's proven transactional behavior: providing cost savings for the high basket group and focusing on service and convenience for the high frequency group. The success of the entire effort is dependent upon the accurate delivery of this tailored message, reinforcing the highly segmented approach.

### Visualisation 5: Analysis of Conversion Scatter Plot (Targeting View)
The goal of this visualisation is to use the RFM map to visually confirm the proximity of the High-Potential Regular Shoppers (HPRS) to the highly profitable Loyalty Tier Customer (LTC) cluster. I used a Scatter Plot (RFM Map) with a Logarithmic Scale to clearly delineate the LTCs (top-right) and visually circle the two adjacent HPRS groups.

![image](https://github.com/Jerry5612/data-analyst-casestudy2/blob/Jerry5612-patch-2/image5.png)

### A. Interpretation 

The Scatter Plot provides crucial visual confirmation that the High-Potential Regular Shoppers (HPRS) are clustered directly at the threshold of conversion, immediately surrounding the highly profitable Loyalty Tier Customer (LTC) cluster. This visual adjacency, locating the HPRS dots next to the LTC cluster in the top-right quadrant, validates the HPRS as the immediate, highest-leverage target. The marketing campaign should, therefore, be focused on applying the necessary push to move these customers just across the conversion threshold into the highest-value segment.

### B. Key Insight

The placement of the two target groups confirms the precise action required for each: the High Frequency Targets are clustered to the left (high F, lower M), meaning the campaign needs to push them UPWARD along the Monetary axis (for more spend). Conversely, the High Basket Targets are clustered below (high M, lower F), meaning they need to be pushed RIGHTWARD along the Frequency axis (for more habit). This visualization provides the final rationale for the two-pronged strategy: push for more spend (Frequency group) and more habit (Basket group).

### C. Implication for Strategy

This chart is the most powerful tool for convincing stakeholders and securing budget approval from executives, as it visually isolates the Return on Investment (ROI) target. By providing undeniable visual evidence that the targeted segmentation and messaging are based on transactional maturity, it minimizes marketing waste and offers clear justification for the entire strategy.

## Summary of Findings: RS vs. LTC

The Loyalty Tier Customer (LTC) segment exhibits significantly superior metrics across all features compared to the Regular Shopper (RS) segment. In terms of Total Value (Monetary), the LTC segment is Extremely High, spending 14.7 times more than the RS group. This financial contribution is sustained by Purchase Frequency, as LTCs are Highly Consistent, buying 8 times more often, while RSs are sporadic. Regarding Engagement (Recency), the LTC segment is Always Active, with 50% purchasing within 11 days, whereas RSs are Largely Dormant, with 50% not having purchased for 123 days.

This behavior suggests RSs are primarily motivated by Price/Cost Savings (confirmed by high AOV in the target group), while LTCs are driven by Rewards and Exclusive Service (confirmed by their low Recency). Consequently, the Strategic Focus for the RS segment is Conversion, specifically targeting the 2,166 High-Potential Regular Shoppers (HPRS) to drive exponential growth. Conversely, the focus for the LTC segment is simply Retention to maintain this crucial profit engine.

## Act
### I. Targeting High Basket Size Shoppers

The first strategy focuses on the Monetary Conversion Stream, targeting the 1,087 High Basket Size HPRS (those above the AOV >£389.78 threshold). The action is to immediately target them using checkout pop-ups and email retargeting, with messaging focusing entirely on the financial savings and Special Discount Rates of Mart Priority. The estimated impact is the conversion of high-volume/wholesaler customers by leveraging the cognitive bias of cost-aversion, ensuring they lock in their highest-value transactions through subscription.

### II. Targeting High Frequency Shoppers

The second strategy targets the Loyalty Incentive and Service Conversion Stream, focusing on the High Frequency HPRS (those at or above the F ≥7 Orders threshold). The action is to programme a priority reward trigger to deploy an in-app notification immediately after the 6th order, offering Priority Delivery and exclusive access to reserved top-selling items. The estimated impact is the conversion of loyal/habitual retail customers by appealing to their value for convenience and service, ultimately pushing them to increase their monetary value (LTV).

### III. Strategy Monitoring (Success Metrics)

The final step is Strategy Monitoring for all 2,166 HPRS post-campaign. Success is defined using two primary, cross-functional KPIs: 1. an increase in Average Basket Size among the High Frequency group and 2. an increase in Frequency of Orders among the High Basket group. This monitoring ensures the budget is generating the correct behavioral shift among shoppers and provides data-backed evidence for continued investment in the two distinct conversion streams.

## Conclusion 

This capstone project successfully utilised RFM segmentation to quantify the 14.7x value disparity between Loyalty Tier and Regular Shoppers, confirming the business task. Through targeted analysis, 2,166 High-Potential Regular Shoppers (HPRS) were identified using two key conversion signals (F≥7 and AOV>£389.78). This resulted in a two-pronged digital marketing strategy—segregated messaging to focus on Monetary Incentives for High Basket Shoppers and Service/Priority Perks for High Frequency Shoppers—providing a highly targeted and efficient path to convert this group.

The next steps for this project should include A/B testing the two distinct marketing streams against a control group to validate the assumed motivations and measure the actual conversion uplift. Future analysis should also involve determining the optimal price point for the Mart Priority subscription among the HPRS segment to maximise subscription rates.

## Executive Summary: Mart Priority Conversion Strategy
### Business Objective
The primary goal of this analysis is to devise a targeted marketing strategy to convert Regular Shoppers (RS) into high-value Loyalty Tier Customers (LTCs) by incentivising them to subscribe to Mart Priority, thereby driving exponential growth in Global Mart’s profits.

### Key Findings & Problem Statement
Analysis of the customer base confirms that profitability is highly concentrated:
1. The Value Gap: Loyalty Tier Customers (LTCs) are the engine of profit, generating 14.7 times the revenue and purchasing 8.1 times more frequently than the average Regular Shopper. Maximising this segment is the core profit strategy.

2. The Opportunity: While the majority of customers are RSs, a small group of 2,166 High-Potential Regular Shoppers (HPRS) exhibit transactional behavior (such as high shopping frequency or high average expenditure) placing them immediately next to the LTC threshold. This HPRS group represents the most efficient and scalable growth opportunity.

### Strategic Recommendation: Two-Pronged Digital Campaign
To effectively convert the 2,166 HPRS, the strategy avoids a single mass-market message and instead employs two distinct, targeted digital streams based on the customer's primary conversion signal.
The high-potential target audience is precisely split into two segments, each with a tailored strategy. The first segment, the High Frequency HPRS, comprises 1,079 customers who meet the threshold of placing 7 or more orders (F≥7). Their primary motivation is Convenience and Loyalty Reward, so the Mart Priority benefit focus for this group is Priority Delivery and Product Reservation, emphasizing service. The second segment, the High Basket Size HPRS, comprises 1,087 customers who meet the threshold of an Average Order Value greater than £389.78 (AOV>£389.78). Their primary motivation is Cost Reduction and Value, and thus the Mart Priority benefit focuses on Special Discount Rates, emphasizing monetary savings.

### Implementation & Action
Service Stream Action: Deploy in-app notifications after the 6th order, positioning the subscription as an exclusive loyalty upgrade offering benefits like priority access to top sellers (eg. World War 2 Gliders).
Monetary Stream Action: Deploy checkout pop-ups and email retargeting when the customer exceeds the £389.78 AOV threshold, demonstrating immediate savings and turning high volume into guaranteed profit.

### Summary Conclusion & Projected Impact
By strictly adhering to this targeted, signal-based approach, the marketing budget will be split evenly and efficiently across the two conversion streams. This methodology guarantees that every dollar spent is directed toward a unique, high-potential customer with a clear, defined motivation, resulting in a quantifiable and exponential increase in customer LTV as HPRS transitions into the highly profitable Loyalty Tier segment.
