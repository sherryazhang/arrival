# nov 2025

## [lilac](https://thelilac.app/)
lilac is an early-stage food delivery startup I have worked with on a couple projects to improve user engagement. One analysis I have done is understanding order behavior to run push notifications at the right times. Using Postgres SQL, here is how I have achieved it:

### data description
In Supabase database, there is an `orders` table that includes

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
The first step is to comb through the data table for relevant fields. In this case, they are `created_at`, `payment_confirmed`, and `cancelled`. `created_at` helps me understand what hours users usually order at during a 24 hour period. `payment_confirmed` and `cancelled` log the status of the order, ensuring it is confirmed and not cancelled.

[side note - this table could use a normalization in the future for better performance and user privacy]

### the query
The next step is to count the number of orders per hour. Because `created_at` is a timestamp stored in UTC, I convert it to local EST time then extract the hour. And to filter for confirmed and not cancelled orders, I set `payment_confirmed = TRUE` and `cancelled = FALSE`.
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
The output of the query is number of orders for each hour from 0 to 23. With it, I create a bar chart to visualize the order distribution:\
\
<img width="545" height="395" alt="Screenshot 2025-10-23 at 11 36 26 AM" src="https://github.com/user-attachments/assets/a9f71c45-2a11-4024-b51b-e75ef9d9c725" />

It looks like, in the past couple months, users visited the [lilac](https://thelilac.app/) platform and ordered food delivery the most around 8pm.

### the recommendation
The push notification is currently set to run at 2pm. The bar chart above shows signs of users not acting on it. I would recommend to run push notifications at 8pm instead and compare the percent difference in order count after a month or so.


### the deepdive
My approach so far looks at overall behavior across all users. But order behavior is subjective. Here, I propose an interesting question to answer - how often and consistent do each user order? The insights would be useful for user segmentation and to deliver personalized experiences.

The method I use is statistics, looking at the mean and standard deviation of order time along with total number of orders per user. A low standard deviation and high order count point to consistent behavior.

In Postgres SQL, I first make two CTEs to get the total number of orders per user:

```
WITH EST_order_hours AS (
    SELECT 
        user_id,
        EXTRACT(HOUR FROM created_at AT TIME ZONE 'America/New_York') AS EST_hour,
        COUNT(*) AS order_count
    FROM 
        orders
    WHERE payment_confirmed = TRUE
        AND cancelled = FALSE
    GROUP BY 
        user_id, EST_hour
),

user_totals AS (
    SELECT 
        user_id,
        SUM(order_count) AS total_orders
    FROM 
        EST_order_hours
    GROUP BY 
        user_id
)
```

Then calculate the mean and standard deviation rounding to the nearest integer:
```
SELECT 
    user_id,
    ROUND(AVG(order_hour)) AS mean_hour,
    ROUND(STDDEV(order_hour)) AS std_hour,
    total_orders
FROM 
    EST_order_hours oh JOIN user_totals ut ON oh.user_id = ut.user_id
GROUP BY ut.user_id, total_orders
ORDER BY total_orders DESC
```

### the output
boxplot


### the deeper dive
For users who are inconsistent, what I can do is dive deeper into their hourly behavior. The following SQL script calculates the median




### the future
Analyses like the above are useful in a workflow that feeds into push notification deployment. However, my current process is manual and repetitive. It is ideal to have an autonomous system in place. As the [lilac](https://thelilac.app/) matures and a lot more data become available, I would implement a model that learns user behavior and decides when the best time is to send push notifications to each user.
