# sherry zhang
case studies here

## [lilac](https://thelilac.app/)
lilac is an early-stage food delivery startup I have worked with on a couple projects to improve customer engagement. one analysis I have done is understanding order behavior to run push notifications at the right times. using Postgres SQL, here is how I have achieved it:

### data description
in our Supabase database, we have an `orders` table that includes

`id` - unique identifier for every entry\
`user_id` - id for every user\
`menu_item_id` - id for the item the customer ordered\
`created_at` - timestamp when the customer ordered\
`quantity` - how many of the menu item the customer ordered\
`discount_percentage` - percentage of discount applied to the order\
`first_name` - first name of the customer\
`last_name` - last name of the customer\
`payment_confirmed` - TRUE/FALSE of whether the payment went through\
`final_price` - price of the order\
`cancelled` - TRUE/FALSE of whether the customer cancelled the order\
`user_email` - email of the customer\
`menu_item_name` - name of the item the customer ordered\
`restaurant_name` - name of the restaurant customer ordered from\
`saving` - the amount the customer is saving on the order\
`restaurant_id` - id for the restaurant customer ordered from\
`order_transaction_id` - id for the payment transaction\
`order_number` - record of the order in text


### field selection
the first step is to comb through the data table for relevant fields in which there are only 4 (haha this table could use a normalization)

### the query
first extract the hour customers ordered at. because it is stored in UTC, I have to convert it to local EST time
```
SELECT 
    EXTRACT(HOUR FROM created_at AT TIME ZONE 'America/New_York') AS EST_hour,
    COUNT(*) AS order_counts
FROM 
    orders
WHERE payment_confirmed = TRUE
    AND cancelled = FALSE
```

### the analysis
<img width="545" height="395" alt="Screenshot 2025-10-23 at 11 36 26 AM" src="https://github.com/user-attachments/assets/a9f71c45-2a11-4024-b51b-e75ef9d9c725" />


### the enhancement
let's add time buckets

```
WITH EST_order_hours AS (
    SELECT 
        EXTRACT(HOUR FROM created_at AT TIME ZONE 'America/New_York') AS EST_hour
    FROM 
        orders
    WHERE payment_confirmed = TRUE
    AND cancelled = FALSE
)

SELECT
    CASE
        WHEN EST_hour BETWEEN 0 AND 5 THEN 'Late Night'
        WHEN EST_hour BETWEEN 6 AND 11 THEN 'Morning'
        WHEN EST_hour BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Night'
    END AS time_bucket,
    COUNT(*) AS order_counts
FROM EST_order_hours
GROUP BY time_bucket
```

the output is\
<img width="201" height="177" alt="Screenshot 2025-10-23 at 2 47 34 PM" src="https://github.com/user-attachments/assets/2d24d6e4-c9c8-4319-9e32-4df66bb3bfa8" />

### the recommendation

Users visit the site and order food delivery at night, specifically at 8 pm. It would be interesting to run push notifications at that time and see how it affects order count. 2pm against 8pm A/B testing. Although this may be too general.

### the deepdive

Per user, how often and consistent do they order to target notifications

```
WITH orders_with_hour AS (
    SELECT 
        user_id,
        first_name,
        last_name,
        EXTRACT(HOUR FROM created_at AT TIME ZONE 'America/New_York') AS order_hour,
        COUNT(*) AS order_count
    FROM 
        orders
    WHERE payment_confirmed = TRUE
    AND cancelled = FALSE
    GROUP BY 
        user_id, first_name, last_name, order_hour
),

user_totals AS (
    SELECT 
        user_id,
        SUM(order_count) AS total_orders
    FROM 
        orders_with_hour
    GROUP BY 
        user_id
)

SELECT 
    first_name,
    last_name,
    ROUND(AVG(order_hour), 0) AS mean_hour,
    ROUND(STDDEV(order_hour), 0) AS std_hour,
    total_orders
FROM 
    orders_with_hour oh
JOIN 
    user_totals ut ON oh.user_id = ut.user_id
GROUP BY first_name, last_name, total_orders
ORDER BY total_orders DESC
```

### the application

if the standard deviation is less than 3 hours, the user is consistent and it triggers a notification. else, 8pm.
an improvement to this is automation with AI

###

