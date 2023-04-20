# MS Access rows to columns

Convert rows to columns when values are unknown beforehand.

## Main query

``` sql
SELECT
    Customers.name,
    [1st_order].item AS item_1,
    [1st_order].quantity AS qty_1,
    [2nd_order].item AS item_2,
    [2nd_order].quantity AS qty_2,
    [3rd_order].item AS item_3,
    [3rd_order].quantity AS qty_3
FROM
    ((Customers LEFT JOIN (SELECT * FROM Orders WHERE id IN (SELECT order_id FROM 1st_orders))  AS 1st_order ON Customers.id=[1st_order].customer_id)
    LEFT JOIN (SELECT * FROM Orders WHERE id IN (SELECT order_id FROM 2nd_orders)) AS 2nd_order ON Customers.id=[2nd_order].customer_id)
    LEFT JOIN (SELECT * FROM Orders WHERE id IN (SELECT order_id FROM 3rd_orders)) AS 3rd_order ON Customers.id=[3rd_order].customer_id;
```

## Sub queries

| name ||
|---|---|
| 1st_orders | The oldest order for each customer. |
| 2nd_orders | The oldest order for each customer after their 1st orders have been are removed. |
| 3rd_orders | The oldest order for each customer after their 1st and 2nd orders have been removed. |
