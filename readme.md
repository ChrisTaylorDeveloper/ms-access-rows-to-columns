# MS Access rows to columns

Convert rows to columns when values are unknown beforehand.  

Run query `customer_orders_as_cols` inside the MS Access 2000 database file.

## Main query

``` sql
SELECT
    Customers.name AS customer_name,
    cust_comp.company_name AS supplier,
    cust_comp.ceo AS supplier_ceo,
    [1st_orders].item AS item_1,
    [1st_orders].quantity AS qty_1,
    [2nd_orders].item AS item_2,
    [2nd_orders].quantity AS qty_2,
    [3rd_orders].item AS item_3,
    [3rd_orders].quantity AS qty_3
FROM
    (
        (
            (SELECT * FROM Customers, Company) AS cust_comp
            LEFT JOIN 1st_orders ON cust_comp.id = [1st_orders].customer_id
        )
        LEFT JOIN 2nd_orders ON cust_comp.id = [2nd_orders].customer_id
    )
    LEFT JOIN 3rd_orders ON cust_comp.id = [3rd_orders].customer_id;
```

## Sub queries

| name ||
|---|---|
| 1st_orders | The oldest order for each customer. |
| 2nd_orders | The oldest order for each customer after their 1st orders have been are removed. |
| 3rd_orders | The oldest order for each customer after their 1st and 2nd orders have been removed. |

### 1st_orders
``` sql
SELECT Orders.customer_id, order_id, item, quantity
FROM Orders
INNER JOIN (
    SELECT customer_id, min(id) AS order_id
    FROM Orders
    GROUP BY customer_id
) AS 1st_orders ON Orders.id = [1st_orders].order_id;
```

### 2nd_orders
``` sql
SELECT Orders.customer_id, order_id, item, quantity
FROM Orders
INNER JOIN (
    SELECT [not_1st_orders].customer_id, Min([not_1st_orders].id) AS order_id
    FROM (
        SELECT Orders.customer_id, Orders.id
        FROM Orders
        LEFT JOIN 1st_orders ON Orders.id = [1st_orders].order_id
        WHERE [1st_orders].order_id IS NULL
    )  AS not_1st_orders
    GROUP BY [not_1st_orders].customer_id
) AS 2nd_orders ON Orders.id = [2nd_orders].order_id;
```

### 3rd_orders
``` sql
SELECT Orders.customer_id, order_id, item, quantity
FROM Orders
INNER JOIN (
    SELECT [not_1st_2nd_orders].customer_id, Min([not_1st_2nd_orders].id) AS order_id
    FROM (
        SELECT Orders.customer_id, Orders.id
        FROM Orders
        LEFT JOIN (
            SELECT * FROM 1st_orders UNION SELECT * FROM 2nd_orders
        ) AS 1st_2nd_orders ON Orders.id = [1st_2nd_orders].order_id
        WHERE [1st_2nd_orders].order_id IS NULL
    )  AS not_1st_2nd_orders
    GROUP BY [not_1st_2nd_orders].customer_id
) AS 3rd_orders ON Orders.id = [3rd_orders].order_id;
```
