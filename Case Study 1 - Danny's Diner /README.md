# Case Study 1 - Danny's Diner
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width = "600">

## Introduction

Danny decides to open a Japanese restaurant that sells his 3 favourite foods: sushi, curry and ramen in the beginning of 2021. However, the restaurant has captured some very basic data from the initial months of operation but have no idea to use their data to help improve the business.

## Problem Statement

Danny wants to use the data to answer some questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- Sales
- Menu
- Members

## Entity Relationship Diagram
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/2382c74c-f323-49e1-b4f1-9af827f9dc87)

## Case Study Questions & Solutions
**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT dannys_diner.sales.customer_id,
      SUM(dannys_diner.menu.price) AS Total_Amount
FROM dannys_diner.menu
RIGHT JOIN dannys_diner.sales
ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
GROUP BY customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/ebf5d985-b226-46e1-bba3-3392425ca3dd)

- FROM clause: From the question, I mainly focus on sales table, then I join it with the menu table to retrieve price data based on product_id as it serves as a common identifier between them.<br>
- GROUP BY clause: Then I group data based on customer_id to calculate the total amount.<br>
- SELECT statement: I use SUM function to calculate total amount spent by customers and include customer_id in the final result

**2. How many days has each customer visited the restaurant?**
```sql
SELECT customer_id,
      COUNT( DISTINCT order_date) AS Time_Visited
FROM dannys_diner.sales
GROUP BY customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/fc5ae8d0-5539-4e43-a038-432eec08557c)

- FROM clause: Retrieve data from the sales table.<br>
- GROUP BY clause: Group data based on customer_id to calculate the time visited .<br>
- SELECT statement: The COUNT function and DISTINCT were used to calculate the number of times a customer visits a restaurant based on order_date

**3. What was the first item from the menu purchased by each customer?**
```sql
--- Create a temporary table to track which products each customer ordered
WITH menu_sales AS (
SELECT customer_id,
            order_date,
            sales.product_id,
            product_name, 
            DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
FROM dannys_diner.menu
RIGHT JOIN dannys_diner.sales
ON menu.product_id = sales.product_id
)

SELECT customer_id,  product_name
FROM menu_sales
WHERE rank = 1;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/7ceebcd6-406f-4a59-8598-c4271654b9a8)

- WITH clause: This part creates a temporary table called menu_sales by combining two tables sales and menu. The DENSE_RANK() assigns a rank to each order for each customer based on order_date.  The earliest purchase from each customer gets a rank 1. If there is more than 1 purchase per date per customer, all of the orders from that customer get the same number.<br>
- SELECT statement: Specifies the columns that will be presented in the final result. It includes customer_id and product_name.<br>
- WHERE clause: Filters the result to include only rows with rank equals to 1. This ensures that only the first item from the menu on the first order date will be shown.

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT product_name, 
      COUNT(*) AS Purchased_Time
FROM dannys_diner.menu
RIGHT JOIN dannys_diner.sales
ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY Purchased_Time DESC
OFFSET 0 ROWS FETCH FIRST 1 ROWS ONLY;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/aa324c05-99dc-4213-8e5e-57f68ba90881)

- FROM Clause: This part combines two tables menu and product based on the common identifier product_id.<br>
- GROUP BY Clause: To calculate the number of purchasing time, the data was grouped by product_name.<br>
- SELECT statement: The COUNT function was used to calculate the number of times each product was purchased and product_name was also selected from the table.<br>
- ORDER BY Clause: This orders the result set by Purchased_Time in descending order.<br>
- OFFSET-FETCH Clause: This part limits the number of rows returned by the query, ensuring that only the the top product was retrieved

**5. Which item was the most popular for each customer?**
```sql
-- Create a table including customer, product and popular product rank
WITH popular_product_within_customer AS
  (SELECT customer_id, 
				  product_name, 
				  COUNT (*) AS Nbr_Order,
  DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT (*) DESC) AS popular_rank
  FROM dannys_diner.menu
  RIGHT JOIN dannys_diner.sales
  ON menu.product_id = sales.product_id
  GROUP BY customer_id, product_name)

-- Select the first popular product rank from the table created above
SELECT customer_id, product_name
FROM popular_product_within_customer
WHERE popular_rank = 1;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/4370f133-0b91-4b51-a32a-2d3afd9c3ad9)

**6. Which item was purchased first by the customer after they became a member?**
```sql
-- Create a table containing data after each customer became member (including joining date)
WITH sales_members AS
  (SELECT sales.customer_id, 
				  join_date, 
				  order_date, 
				  product_name,
  DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date) AS order_rank
  FROM dannys_diner.sales
  RIGHT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  WHERE join_date <= order_date)
  
SELECT customer_id, join_date, order_date, product_name
FROM sales_members
WHERE order_rank = 1;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/f9c8e822-dd4b-46ae-bfc0-0ddb8354a8bb)

**7. Which item was purchased just before the customer became a member?**
```sql
-- Create a table containing data before each customer became member 
WITH sales_members AS
  (SELECT sales.customer_id,
            join_date,
            order_date,
            product_name,
            DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date DESC) AS order_rank
  FROM dannys_diner.sales
  RIGHT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  WHERE join_date > order_date)
  
SELECT customer_id,
      join_date,
      order_date,
      product_name
FROM sales_members
WHERE order_rank = 1;
```

![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/def65be8-94a1-425c-af1f-cb0069435da4)

**8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT sales.customer_id,
      COUNT(*) AS Total_Items,
      SUM(price) AS Total_Amount
FROM dannys_diner.sales
RIGHT JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
WHERE join_date > order_date
GROUP BY sales.customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/9f10dcaf-ce43-4ad0-8d42-0ed99517c5de)

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
SELECT sales.customer_id,
  SUM(
  CASE 
	  WHEN product_name = 'sushi' THEN price * 10 * 2
	  ELSE price * 10
  END) AS Total_Points
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/68fe9b0b-006b-4793-bf87-4c3b81e7abbd)

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
SELECT sales.customer_id,
  SUM(
  CASE 
      WHEN order_date < DATEADD(week, 1, join_date) THEN price * 10 * 2
      ELSE price * 10
  END) AS Member_Points
FROM dannys_diner.sales
RIGHT JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
WHERE join_date <= order_date
AND order_date <= '2021-01-31'
GROUP BY sales.customer_id,
            join_date;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/539701a5-a993-4654-925c-bbad27f0e369)

**Bonus questions:**
**Recreate the table**
```sql
SELECT sales.customer_id,
  order_date,
  product_name,
  price,
  --- Create member status
  CASE
      WHEN join_date <= order_date THEN 'Y'
      ELSE 'N'
  END AS member
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
ON sales.customer_id = members.customer_id;
```
![image](https://github.com/han-nguyen97/8weeksqlchallenge/assets/83593831/89c86028-1eb6-4bf5-908b-ae76cd8f1528)

**Rank All The Things**

```sql
WITH sales_data AS
  (SELECT sales.customer_id,
            order_date,
            product_name,
            price,
--- Create a member status if the customer become member or not
      CASE
            WHEN join_date <= order_date THEN 'Y'
            ELSE 'N'
      END AS member,
--- Create a new member_id for customer who become meber
      CASE
            WHEN join_date <= order_date THEN sales.customer_id
            ELSE null
      END AS member_id
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id)

SELECT customer_id,
      order_date,
      product_name,
      price,
      member,
      CASE
      WHEN member = 'Y' THEN DENSE_RANK() OVER (PARTITION BY member_id ORDER BY order_date) 
      ELSE null
      END AS ranking
FROM sales_data
ORDER BY customer_id,
      order_date;
```

