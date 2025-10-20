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
the first step is to comb through the data table for relevant fields in which there are only 4 - haha this table could use a normalization

### the query
then create a CTE to extract the hour customers ordered at. because it is stored in a timezone 4 hours ahead, I have to convert it to local time `- INTERVAL '4 hours'`. the other consideration to make is we are 
```
WITH order_hours AS (
    SELECT 
        user_id,
        first_name,
        last_name,
        EXTRACT(HOUR FROM created_at - INTERVAL '4 hours') AS order_hour
    FROM 
        orders
    WHERE payment_confirmed = TRUE
        AND cancelled = FALSE
)
```


```
order_counts AS (
    SELECT 
        user_id,
        first_name,
        last_name,
        order_hour,
        COUNT(*) AS orders_count
    FROM 
        orders_with_hour
    GROUP BY 
        user_id, first_name, last_name, order_hour
),
user_totals AS (
    SELECT 
        user_id,
        SUM(orders_count) AS total_orders
    FROM 
        order_counts
    GROUP BY 
        user_id
)
```

```
SELECT 
    oc.user_id,
    oc.first_name,
    oc.last_name,
    oc.order_hour,
    oc.orders_count,
    ut.total_orders
FROM 
    order_counts oc
JOIN 
    user_totals ut ON oc.user_id = ut.user_id
ORDER BY 
    ut.total_orders DESC,
    oc.user_id,
    oc.order_hour;
```
Weekly Active Users
