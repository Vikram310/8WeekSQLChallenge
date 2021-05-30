**Schema of the database** 

CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
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
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');
      
**Query #1** What is the total amount each customer spent at the restaurant?

    SELECT 
    	s.customer_id, 
    	SUM(m.price) AS Total_spent 
    	FROM 
    sales AS s JOIN
    menu AS m ON
    s.product_id = m.product_id
    	GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---


**Query #2** How many days has each customer visited the restaurant?


    SELECT 
    	customer_id, 
    	COUNT(order_date) AS Days_Visited
    FROM
    	sales
    GROUP BY 
    	customer_id; 

| customer_id | days_visited |
| ----------- | ------------ |
| B           | 6            |
| C           | 3            |
| A           | 6            |

---

**Query #3**What was the first item from the menu purchased by each customer?

    WITH cte 
    AS 
    (
    	SELECT 
    		customer_id,
    		order_date,
    		product_id,
    		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_rank
    	FROM 
    		sales
    )
    
    SELECT 
    	c.customer_id,m.product_name
    FROM 
    	cte	AS	c 
    JOIN
    	menu AS	m 
    ON
    	c.product_id = m.product_id
    WHERE
    	order_rank = 1;

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---


**Query #4**What is the most purchased item on the menu and how many times was it purchased by all customers?

    with cte as
    (
    	select 
		menu.product_name,
		count(sales.product_id) as TotalPurchased
    	from 
		sales
    	join 
		menu on sales.product_id=menu.product_id
    	group by
		menu.product_name
    	
    )
    select *
    from cte
    order by TotalPurchased desc
    limit 1;

| product_name | totalpurchased |
| ------------ | -------------- |
| ramen        | 8              |

---


**Query #5**Which item was the most popular for each customer?

    with cte as
    (
    	select 
      		sales.customer_id, 
      		menu.product_name, 
      		count(sales.product_id) as TotalPurchased,
    		ROW_NUMBER() over(partition by sales.customer_id order by count(sales.product_id) desc) as RowNum
    	from 
      		sales
    	inner join 
		menu on sales.product_id=menu.product_id
    	group by
		sales.customer_id, menu.product_name
    )
    select
    	cte.customer_id,
        cte.product_name as PopularMenu
    from 
    	cte
    where 
	RowNum = 1;

| customer_id | popularmenu |
| ----------- | ----------- |
| A           | ramen       |
| B           | ramen       |
| C           | ramen       |

---


**Query #6** Which item was purchased first by the customer after they became a member?

    with cte as
    (
    	select 
      		sales.customer_id,
      		members.join_date, 
      		sales.order_date, 
      		menu.product_name,
    		ROW_NUMBER() over(partition by sales.customer_id order by sales.order_date desc) as RowNum
    	from 
      		sales
    	inner join 
      		menu on sales.product_id=menu.product_id
    	inner join
      		members on sales.customer_id=members.customer_id
    	where 
      		sales.order_date >= members.join_date
    )
    select 
    	cte.customer_id,
        cte.product_name as FirstPurchased
    from
    	cte
    where
    	RowNum = 1;

| customer_id | firstpurchased |
| ----------- | -------------- |
| A           | ramen          |
| B           | ramen          |

---



**Query #7** Which item was purchased just before the customer became a member?

    with cte as
    (
    	select 
      		sales.customer_id,
      		members.join_date, 
      		sales.order_date, 
      		menu.product_name,
    		  ROW_NUMBER() over(partition by sales.customer_id order by sales.order_date desc) as RowNum
    	from 
      		sales
    	inner join 
      		menu on sales.product_id=menu.product_id
    	inner join
      		members on sales.customer_id=members.customer_id
    	where 
      		sales.order_date < members.join_date
    )
    select 
    	cte.customer_id,
        cte.product_name as FirstPurchased
    from
    	cte
    where
    	RowNum = 1;

| customer_id | firstpurchased |
| ----------- | -------------- |
| A           | sushi          |
| B           | sushi          |

---


**Query #8** What is the total items and amount spent for each member before they became a member?

    select
    	sales.customer_id, 
        COUNT(sales.customer_id) as TotalItem, 
        SUM(menu.price) as TotalAmount
    from 
    	sales
    inner join 
    	menu on sales.product_id=menu.product_id
    inner join
    	members on sales.customer_id=members.customer_id
    where
    	sales.order_date < members.join_date
    group by
    	sales.customer_id;

| customer_id | totalitem | totalamount |
| ----------- | --------- | ----------- |
| B           | 3         | 40          |
| A           | 2         | 25          |

---

**Query #9** If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    SELECT 
    	s.customer_id,
    	SUM(CASE 
            	WHEN
    			m.product_name = 'sushi'
    		THEN
    			m.price * 20
    		ELSE
    			m.price * 10
    		END
    		) AS total_points	
    FROM 
    	sales AS s
    JOIN
    	menu AS m
    ON
    	s.product_id = m.product_id
    GROUP BY
    	s.customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| B           | 940          |
| C           | 360          |
| A           | 860          |

---


**Query #10** In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

    SELECT 
    	s.customer_id,
        SUM(CASE 
            	WHEN
    				product_name = 'sushi' 
    			THEN 
    				20 * price
                WHEN 
    				order_date BETWEEN join_date AND join_date+('7'* interval'1 day')
    			THEN 
    				20 * price
                ELSE 
    				10 * price
                END) AS total_points	
    FROM 
    	sales AS s
    JOIN
    	menu AS m
    ON
    	s.product_id = m.product_id
    JOIN
    	members AS mem
    ON
    	mem.customer_id = s.customer_id
    WHERE
    	s.order_date <= '2021-01-31'
    GROUP BY
    	s.customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 940          |

---



