
````sql
CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);
````
````sql
INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 ````
````sql
CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);
````
````sql
INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  ````
````sql
CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);
````
````sql
INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
````
````sql
  SELECT *
  FROM Sales
````
````sql
  SELECT *
  FROM Menu
````
  ````sql
  SELECT *
  FROM Members
  ````
  --1.	What is the total amount each customer spent at the restaurant?
  ````sql
  Select S.customer_id, 
  SUM (price) as TotalAmount
  From menu As M 
  JOIN sales 
  AS S On M.product_id=S.product_id
  Group by S.customer_id;
````
![image](https://github.com/user-attachments/assets/e111f017-3117-411d-892a-82592619f0d1)

````sql
  --2. How many days has each customer visited the restaurant?
  Select
  Distinct
  customer_id,
  COUNT ( Order_date) 
  As Totalday 
  From sales
  Group by 
  customer_id;
````
![image](https://github.com/user-attachments/assets/8905711b-b153-4328-81b3-886ffc60d9c5)

  --3.What was the first item from the menu purchased by each customer?
````sql
 SELECT 
 DISTINCT
 customer_id,
 product_name
 FROM
 ( SELECT 
s.customer_id,
 s.product_id, 
s.order_date, 
m.product_name,
 DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) "dense_rnk"
 FROM [dbo].[sales] s
 JOIN [dbo].[menu] m
 ON s.product_id = m.product_id
 ) a
 WHERE dense_rnk = 1
````
![image](https://github.com/user-attachments/assets/ab7e7f05-8456-491b-a4e3-bc2aba1a0a9b)

--4.	What is the most purchased item on the menu and how many times was it purchased by all customers?
 ````sql
Select 
Distinct
Customer_id, 
Product_name
 FROM 
(Select S.customer_id,
 M.Product_name, 
M.product_id,
 DENSE_RANK() Over(PARTITION BY S.customer_id 
Order by M.product_id DESC) Densrank From [dbo].[menu] M 
Join [dbo].[sales] S ON S.product_id = M.product_id) Densrank
 Where Densrank= 1;
````
![image](https://github.com/user-attachments/assets/0dc727f1-e3cf-4b70-8a50-2ffb71923013)

 --5.Which item was the most popular for each customer?
 ````sql
 Select
	Distinct
	Customer_id,
	Product_name
FROM
(Select 
	S.customer_id,
	M.Product_name,
	M.product_id,
DENSE_RANK() Over(PARTITION BY S.customer_id Order by M.product_id DESC) Densrank
From
	[dbo].[menu] M
	Join [dbo].[sales] S
ON
	S.product_id = M.product_id) Densrank
Where
	Densrank= 1;
````
![image](https://github.com/user-attachments/assets/7c2b0790-07e1-4014-b361-126bbaa90400)

````sql
 --6.	Which item was purchased first by the customer after they became a member?
 select 
	Top 1
	S.customer_id,
	product_name
From
(Select 
	S.customer_id,
	order_date 
From
	[dbo].[sales] S
Join[dbo].[members] MB
ON
	S.customer_id = MB.customer_id
Where order_date > join_date) FT
Join[dbo].[sales] S
ON
FT.customer_id= S.customer_id AND FT.order_date = S.order_date
JOIN [dbo].[menu] MU
ON
S.product_id = mu.product_id
Order by FT.customer_id;
````
![image](https://github.com/user-attachments/assets/80eb4e40-d90f-422e-97bb-d47d339b7fc6)

  --7.Which item was purchased just before the customer became a member?
````sql
  Select 
	Top 1
	S.customer_id,
	product_name
From
(Select 
	S.customer_id,
	order_date 
From
	[dbo].[sales] S
Join[dbo].[members] MB
ON
	S.customer_id = MB.customer_id
Where order_date < join_date) FT
Join[dbo].[sales] S
ON
FT.customer_id= S.customer_id AND FT.order_date = S.order_date
JOIN [dbo].[menu] MU
ON
S.product_id = mu.product_id
Order by FT.customer_id;
````
![image](https://github.com/user-attachments/assets/f0b61e37-ac3d-4a24-9ea0-c3a1e01285dd)

---8.What is the total items and amount spent for each member before they became a member?
````sql
 SELECT
 S.customer_id,
 COUNT(S.product_id) AS totalitems,
 SUM(M.price) AS totalprice
 FROM  Sales S
 JOIN Menu M
 ON M.product_id = S.product_id
 JOIN Members MB
 ON S.customer_id = MB.customer_id
 WHERE S.order_date < MB.join_date
 GROUP BY 
	S.customer_id;
````
![image](https://github.com/user-attachments/assets/57280d46-0ca8-4dc0-8e2a-83af5db0f5ff)

--9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
SELECT
S.customer_id,
	SUM(CASE 
	WHEN m.product_id=1 
	THEN price*20 
	ELSE price*10 
	END) AS points
FROM sales s
JOIN
	MENU M
ON s.product_id=m.product_id
GROUP BY
	S.customer_id;
````
![image](https://github.com/user-attachments/assets/d96b7b24-aac1-4021-b256-b60dbfb3b8cc)

--10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
SELECT
S.customer_id,
SUM(CASE
	WHEN m.product_id IN (1,2,3)
	THEN price*20
	END)
	AS points
FROM sales s
JOIN
	MENU M
ON s.product_id=m.product_id
JOIN Members MB
 ON S.customer_id = MB.customer_id
 WHERE S.order_date < MB.join_date
 GROUP BY 
 S.customer_id;
````
![image](https://github.com/user-attachments/assets/ce6bb9fb-e479-43f2-801a-7ebfdd799464)


 
