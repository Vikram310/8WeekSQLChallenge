**Query #1**How many customers has Foodie-Fi ever had?

    select count(distinct customer_id) as total_customers from subscriptions;

| total_customers |
| --------------- |
| 1000            |

---


**Query #2**What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

    SELECT EXTRACT(MONTH FROM start_date) AS months, COUNT(*)
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    GROUP BY months
    ORDER BY months;

| months | count |
| ------ | ----- |
| 1      | 88    |
| 2      | 68    |
| 3      | 94    |
| 4      | 81    |
| 5      | 88    |
| 6      | 79    |
| 7      | 89    |
| 8      | 88    |
| 9      | 87    |
| 10     | 79    |
| 11     | 75    |
| 12     | 84    |

---


**Query #3**What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

    select 
    	plan_name,
        count(start_date) as Answer 
    from 
    	foodie_fi.plans join foodie_fi.subscriptions 
    on 
    	plans.plan_id = subscriptions.plan_id
    where
    	extract(year from subscriptions.start_date) = '2021'
    group by
    	plan_name
    order by 
    	Answer;

| plan_name     | answer |
| ------------- | ------ |
| basic monthly | 8      |
| pro monthly   | 60     |
| pro annual    | 63     |
| churn         | 71     |

---


**Query #4**What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

    SELECT 
    	COUNT(DISTINCT(s.customer_id)) as customer_count,
    	ROUND((COUNT(DISTINCT(s.customer_id))::numeric/1000::numeric)*100,1) as churn_customers_percentage
    FROM 
        foodie_fi.subscriptions s JOIN foodie_fi.plans p
    on
        s.plan_id = p.plan_id
    WHERE
        p.plan_name='churn';

| customer_count | churn_customers_percentage |
| -------------- | -------------------------- |
| 307            | 30.7                       |

---


**Query #5**How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

    DROP TABLE IF EXISTS total_count;
    CREATE TEMP TABLE total_count AS (
        SELECT COUNT(DISTINCT customer_id) AS num
        FROM foodie_fi.subscriptions
    );
    DROP TABLE IF EXISTS next_plan;
    CREATE TEMP TABLE next_plan AS(
        SELECT *, 
            LEAD(plan_id, 1) 
            OVER(PARTITION BY customer_id ORDER BY start_date) as next_plan
        FROM foodie_fi.subscriptions
    );

    WITH direct_churner_cte AS (
        SELECT COUNT(DISTINCT customer_id) AS direct_churner
        FROM next_plan
        WHERE plan_id = 0 AND next_plan = 4
    )
    
    SELECT direct_churner, direct_churner::FLOAT/num::FLOAT * 100 AS percent_churned
    FROM direct_churner_cte, total_count;

| direct_churner | percent_churned |
| -------------- | --------------- |
| 92             | 9.2             |

---


**Query #6**What is the number and percentage of customer plans after their initial free trial?

    DROP TABLE IF EXISTS total_count;
    CREATE TEMP TABLE total_count AS (
    SELECT COUNT(DISTINCT customer_id) AS num
    FROM foodie_fi.subscriptions
    );
    with cte2 as(
    	with cte1 as(
    		SELECT
          		p.plan_id,
    			s.customer_id,
    			s.start_date,
    			p.plan_name,
    			LEAD(p.plan_name,1) OVER(partition by s.customer_id order by s.start_date) as next_plan
    		FROM foodie_fi.subscriptions s
    		JOIN foodie_fi.plans p
    			ON s.plan_id = p.plan_id)
    	SELECT
      		next_plan,
    		COUNT(*) as total_customer_count,
    		COUNT(
    			CASE
    				WHEN plan_name='trial' THEN 1 
    			END) as customer_plans_after_free_trial
      	FROM cte1
      	where next_plan != 'null'
    	GROUP BY next_plan)
    SELECT
    	next_plan,
        customer_plans_after_free_trial,
    	ROUND(((customer_plans_after_free_trial::numeric/num::numeric)*100 ),0)as percentage
    FROM cte2,total_count;

| next_plan     | customer_plans_after_free_trial | percentage |
| ------------- | ------------------------------- | ---------- |
| pro annual    | 37                              | 4          |
| churn         | 92                              | 9          |
| pro monthly   | 325                             | 33         |
| basic monthly | 546                             | 55         |

---


**Query #7**What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

    DROP TABLE IF EXISTS total_count;
    CREATE TEMP TABLE total_count AS
    (
    	SELECT COUNT(DISTINCT customer_id) AS num
    	FROM foodie_fi.subscriptions
    );

    WITH next_date_cte AS (
        SELECT p.plan_id as plan_id,p.plan_name as plan_name,s.customer_id,s.start_date,
      			
                LEAD (start_date, 1) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
        FROM foodie_fi.subscriptions s join foodie_fi.plans p on s.plan_id = p.plan_id
    ),
    customers_on_date_cte AS (
        SELECT plan_id, plan_name,COUNT(DISTINCT customer_id) AS customers
        FROM next_date_cte
        WHERE (next_date IS NOT NULL AND ('2020-12-31'::DATE > start_date AND '2020-12-31'::DATE < next_date))
            OR (next_date IS NULL AND '2020-12-31'::DATE > start_date)
        GROUP BY plan_id,plan_name
    )
    
    SELECT plan_id, plan_name, customers, ROUND(CAST(customers::FLOAT / num::FLOAT * 100 AS NUMERIC), 2) AS percent
    FROM customers_on_date_cte, total_count;

| plan_id | plan_name     | customers | percent |
| ------- | ------------- | --------- | ------- |
| 0       | trial         | 19        | 1.90    |
| 1       | basic monthly | 224       | 22.40   |
| 2       | pro monthly   | 326       | 32.60   |
| 3       | pro annual    | 195       | 19.50   |
| 4       | churn         | 235       | 23.50   |

---


**Query #8**How many customers have upgraded to an annual plan in 2020?

    select 
        	count(customer_id) as annual_customers
        from 
        	 foodie_fi.subscriptions s
        join foodie_fi.plans p 
        on 	s.plan_id = p.plan_id
        where plan_name = 'pro annual' and date_part('year', start_date) = 2020;

| annual_customers |
| ---------------- |
| 195              |

---


**Query #9**How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

    WITH join_date AS (
        SELECT customer_id, start_date 
        FROM foodie_fi.subscriptions 
        WHERE plan_id = 0
    ),
    pro_date AS (
        SELECT customer_id, start_date AS upgrade_date 
        FROM foodie_fi.subscriptions 
        WHERE plan_id = 3
    )
    
    SELECT ROUND(AVG(upgrade_date - start_date), 0) AS avg_days_to_upgrade
    FROM join_date JOIN pro_date
        ON join_date.customer_id = pro_date.customer_id;

| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---


**Query #10** Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

    WITH join_date AS (
        SELECT customer_id, start_date 
        FROM foodie_fi.subscriptions 
        WHERE plan_id = 0
    ),
    pro_date AS (
        SELECT customer_id, start_date AS upgrade_date 
        FROM foodie_fi.subscriptions 
        WHERE plan_id = 3
    ),
    buckets AS (
        SELECT WIDTH_BUCKET(upgrade_date - start_date, 0, 360, 12) AS avg_days_to_upgrade
        FROM join_date JOIN pro_date
            ON join_date.customer_id = pro_date.customer_id
    )
    
    
    SELECT ((avg_days_to_upgrade - 1)*30 || '-' || (avg_days_to_upgrade)*30) AS "30-day-range", COUNT(*)
    FROM buckets
    GROUP BY avg_days_to_upgrade
    ORDER BY avg_days_to_upgrade;

| 30-day-range | count |
| ------------ | ----- |
| 0-30         | 48    |
| 30-60        | 25    |
| 60-90        | 33    |
| 90-120       | 35    |
| 120-150      | 43    |
| 150-180      | 35    |
| 180-210      | 27    |
| 210-240      | 4     |
| 240-270      | 5     |
| 270-300      | 1     |
| 300-330      | 1     |
| 330-360      | 1     |

---


**Query #11**How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

    DROP TABLE IF EXISTS next_plan_cte;
    CREATE TEMP TABLE next_plan_cte AS(
        SELECT *, 
            LEAD(plan_id, 1) 
            OVER(PARTITION BY customer_id ORDER BY start_date) as next_plan
        FROM foodie_fi.subscriptions
      	WHERE DATE_PART('year', start_date) = 2020
    );

    SELECT COUNT(*) AS customers_downgraded
    FROM next_plan_cte
    WHERE plan_id=2 AND next_plan=1;

| customers_downgraded |
| -------------------- |
| 0                    |

---
