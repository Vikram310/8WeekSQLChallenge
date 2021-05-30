#week-3#

**Query #1** How many customers has Foodie-Fi ever had?

    select count(distinct customer_id) as total_customers from subscriptions;

| total_customers |
| --------------- |
| 1000            |

---


**Query #2** What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

    SELECT distinct MONTH(start_date) as value1,
    count(plans.plan_id) as ans FROM subscriptions 
    join plans on plans.plan_id = subscriptions.plan_id
    where plan_name = 'trial'
    group by value1
    order by value1;

| value1 | ans |
| ------ | --- |
| 1      | 88  |
| 2      | 68  |
| 3      | 94  |
| 4      | 81  |
| 5      | 88  |
| 6      | 79  |
| 7      | 89  |
| 8      | 88  |
| 9      | 87  |
| 10     | 79  |
| 11     | 75  |
| 12     | 84  |

---


**Query #3**What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

    select plan_name,count(start_date) as answer from plans
    join subscriptions on plans.plan_id = subscriptions.plan_id
    where YEAR(subscriptions.start_date) = 2021
    group by plan_name;

| plan_name     | answer |
| ------------- | ------ |
| basic monthly | 8      |
| churn         | 71     |
| pro annual    | 63     |
| pro monthly   | 60     |

---


**Query #4** What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

    select churn_customers,
    	ROUND(churn_customers/total_customers * 100,1) as pct_of_total
    from ((
    select
    		count(customer_id) as churn_customers
    from subscriptions s
    join plans p on p.plan_id=s.plan_id 
    where p.plan_name ='churn') as A
      
    join
    
    (select 
     		count(customer_id) as total_customers 
     from subscriptions) as B);

| churn_customers | pct_of_total |
| --------------- | ------------ |
| 307             | 11.6         |

---


**Query #5** How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

    select
    	direct_churn_customers,
    	ROUND(direct_churn_customers/total_customers * 100,0) as pct_of_total
    from ((
    select
    		count(customer_id) as direct_churn_customers
    from subscriptions s
    join plans p on p.plan_id=s.plan_id 
    where p.plan_name NOT IN ('basic monthly','pro monthly','pro annual') ) as A
      
    join
    
    (select 
     		count(customer_id) as total_customers 
     from subscriptions) as B);

| direct_churn_customers | pct_of_total |
| ---------------------- | ------------ |
| 1307                   | 49           |

---


**Query #6**What is the number and percentage of customer plans after their initial free trial

    select
        	plan_name,
            Customers,
        	ROUND(Customers/total_customers * 100,1) as pct_of_total
        from ((
        select
        		plan_name,count(customer_id) as Customers
        from subscriptions s
        join plans p on p.plan_id=s.plan_id 
        group by plan_name
        having plan_name != 'trial') as A
          
        join
        
        (select 
         		count(customer_id) as total_customers 
         from subscriptions) as B);

| plan_name     | Customers | pct_of_total |
| ------------- | --------- | ------------ |
| basic monthly | 546       | 20.6         |
| churn         | 307       | 11.6         |
| pro annual    | 258       | 9.7          |
| pro monthly   | 539       | 20.3         |

---


**Query #7**What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

    select
        	plan_name,
            Customers_as_on_31Dec,
        	ROUND(Customers_as_on_31Dec/total_customers * 100,1) as pct_of_total
        from ((
        select
        		plan_name,count(customer_id) as Customers_as_on_31Dec
        from subscriptions s
        join plans p on p.plan_id=s.plan_id 
        where YEAR(s.start_date) = 2020
        group by plan_name )as A
          
        join
        
        (select 
         		count(customer_id) as total_customers 
         from subscriptions) as B);

| plan_name     | Customers_as_on_31Dec | pct_of_total |
| ------------- | --------------------- | ------------ |
| basic monthly | 538                   | 20.3         |
| churn         | 236                   | 8.9          |
| pro annual    | 195                   | 7.4          |
| pro monthly   | 479                   | 18.1         |
| trial         | 1000                  | 37.7         |

---


**Query #8**How many customers have upgraded to an annual plan in 2020?

    select 
    	count(customer_id) as annual_customers
    from 
    	 subscriptions s
    join plans p 
    on 	s.plan_id = p.plan_id
    where plan_name = 'pro annual' and YEAR(start_date) = 2020;

| annual_customers |
| ---------------- |
| 195              |

---
