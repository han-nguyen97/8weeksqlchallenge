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


