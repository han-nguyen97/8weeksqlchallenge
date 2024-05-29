# Case Study 4 - Data Bank
<img src="https://8weeksqlchallenge.com/images/case-study-designs/4.png" width = "600">

## Introduction
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.
Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!\
Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team needs your help!\
This case study is all about calculating metrics, growth and helping the business analysing the data in a smart way to better forecast and plan for their future developments!

## Entity Relationship Diagram

![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/d7260901-3b8a-4c9f-bab2-ffa565f057da)

## Case Study Questions & Solutions
### A. Customer Nodes Exploration
**1. How many unique nodes are there on the Data Bank system?**
```sql
SELECT COUNT(DISTINCT node_id) AS nbr_unique_nodes
FROM customer_nodes;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/17cf2cfa-8ccb-4c8e-b286-506bd575a99a)

To calculate the total unique nodes, the query retrieves data from customer_nodes table. The COUNT function was used to calculate the number of nodes. The DISTINCT function is used to ensure there is no duplicate nodes.

**2. What is the number of nodes per region?**
```sql
SELECT region_name, 
				COUNT(node_id) AS nbr_nodes
FROM customer_nodes
INNER JOIN regions
ON customer_nodes.region_id = regions.region_id
GROUP BY region_name;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/ecf1fa6c-c7e8-4ccb-9481-41d1203eeb08)

First, the query joins two tables regions and customer_nodes to retrieve data relating to region and nodes. Then, the data is group by region_name to calculate the number of nodes per region. The COUNT function was used to aggregate the number of nodes.

**3. How many customers are allocated to each region?**
```sql
SELECT region_name, 
				COUNT(DISTINCT customer_id) AS nbr_customers
FROM customer_nodes
INNER JOIN regions
ON customer_nodes.region_id = regions.region_id
GROUP BY region_name;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/7eefaece-630b-4efd-a62e-69247c785ff8)

First, the query joins two tables regions and customer_nodes to retrieve data relating to region and nodes. Then, the data is grouped by region_name to calculate the number of customers per region. The COUNT and DISTINCT functions were used to aggregate the number of customers. In the SELECT statement, Region_name and nbr_customers were selected to show in the final result.

**4. How many days on average are customers reallocated to a different node?**
```sql
SELECT AVG(DATEDIFF(day, start_date, end_date))AS avg_date_diff
FROM customer_nodes
WHERE end_date <> '9999-12-31';
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/ec78d3ef-3c91-4d7a-b514-58bad960ffe7)

The query retrieves data from customer_nodes table, filtering out the end_date equals ‘9999-12-31’, then it calculates the number of days between start date and end date using DATEDIFF function. The average day in which the customers were reallocated to a new node is calculated using AVG function.

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**
```sql
SELECT DISTINCT region_name, 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DATEDIFF(day, start_date, end_date)) OVER (PARTITION BY region_name) AS median_diff_day,
  PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY DATEDIFF(day, start_date, end_date)) OVER (PARTITION BY region_name) AS percentile_80_diff_day,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY DATEDIFF(day, start_date, end_date)) OVER (PARTITION BY region_name) AS percentile_95_diff_day
FROM customer_nodes
INNER JOIN regions
ON customer_nodes.region_id = regions.region_id
WHERE end_date <> '9999-12-31';
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/b9a861a1-ad27-4b0f-be69-d6be1e31c840)

The query first retrieves al the data from customer_nodes and region table, then it filtering out the transactions which end_date equals ‘9999-12-31’, indicating ongoing allocations. Then it calculates mean, 80% percentile and 95% percentile for each region using PERCENTILE_CONT function.\
WITHIN GROUP (ORDER BY DATEDIFF(day, start_date, end_date)) orders the differences in days between `start_date` and `end_date` for the percentile calculation
`OVER (PARTITION BY region_name)`**: Partitions the data by`region_name` so that the median is calculated for each region independently.

### B. Customer Transactions
**1. What is the unique count and total amount for each transaction type?**
```sql
SELECT DISTINCT txn_type,
  COUNT(*) AS nbr_transaction,
  SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/95e65931-4f09-4782-aec9-fc00a13175b8)

The query retrieves data from customer_transactions table and groups by txn_type to calculate the number of transactions and the total amount for each type. In the SELECT statement, the COUNT and SUM functions were used to calculate the required fields.

**2. What is the average total historical deposit counts and amounts for all customers?**
```sql
SELECT COUNT(*) / COUNT(DISTINCT customer_id) AS avg_deposit_count,
  SUM(txn_amount) / COUNT(DISTINCT customer_id) AS avg_deposit_amount
FROM customer_transactions
WHERE txn_type = 'deposit';
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/3d6f0a6a-f3c3-4cb4-bf53-ca05cd72160c)

The data is retrieved from customer_transactions table, filtering rows where txn_type equals ‘deposit’.\
To calculate the average number of historical deposit, the query counts all the deposits using COUNT(*) and then divides it by total unique customers. The average deposit amount is calculated by dividing the total number of deposits by the total number of unique customers.

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
```sql
-- Create a table calculating the number of times each customer makes a deposit, purchase and withdrawal per month
WITH nbr_type_per_month AS
	(SELECT MONTH(txn_date) AS month,
				customer_id,
				SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS nbr_deposit,
				SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS nbr_withdrawal,
				SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS nbr_purchase
	FROM customer_transactions
	GROUP BY MONTH(txn_date), customer_id)

SELECT month,
			COUNT(DISTINCT customer_id) AS nbr_customer
FROM nbr_type_per_month
WHERE nbr_deposit > 1
AND( nbr_withdrawal >= 1 OR nbr_purchase >= 1)
GROUP BY month;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/fe2b6a83-4541-4252-ac47-4ee32ae7420a)

a. Temporary Table (nbr_type_per_month):\
The first query creates a temporary table to analyze transactions by month. The data is retrieved from customer_transactions table, then the data is grouped by month and customer_id. The function MONTH is used to extract month from the date.
It then calculates for each customer, in a specific month:
- The number of deposits (`nbr_deposit`)
- The number of withdrawals (`nbr_withdrawal`)
- The number of purchases (`nbr_purchase`)

b. Finding Active Customers (Final Query):\
We use the temporary table to find out, for each month:
- How many unique customers made more than one deposit (`nbr_deposit > 1`)
- AND also made at least one withdrawal (`nbr_withdrawal >= 1`) OR at least one purchase (`nbr_purchase >= 1`) in that same month.

**4. What is the closing balance for each customer at the end of the month?**
```sql
SELECT customer_id,
  SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) - 
  SUM(CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN txn_amount ELSE 0 END) AS closing_balance
FROM customer_transactions
GROUP BY customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/d3a84978-fd84-45fb-abff-e00988fdab86)

The query uses CASE statements to categorize transactions based on their type:
- If `txn_type` is 'deposit', it adds the `txn_amount` to the running total.
- If `txn_type` is 'withdrawal' or 'purchase', it subtracts the `txn_amount` from the total.
The `GROUP BY customer_id` clause ensures the calculations are done for each customer independently. By effectively separating deposits from withdrawals/purchases, the query essentially calculates the net change in each customer's balance throughout the entire period.
