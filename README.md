# sql-analysis-in-rdbms

![image](https://github.com/user-attachments/assets/eb690a7b-2d1e-4cbb-b582-f9b1fd712a39)

## 1. Total Sales by Category
This query calculates the total sales revenue generated by each product category. It helps identify the most profitable categories and understand the sales # distribution across different product types.

      SELECT c.category_name, 
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      JOIN categories c ON p.category_id = c.category_id
      GROUP BY c.category_name
      ORDER BY total_sales DESC;


## 2. Top Selling Products
This query identifies the top 5 best-selling products based on the total quantity sold. It helps understand customer preferences and product popularity.

      SELECT p.product_id, p.product_name, SUM(oi.quantity) AS total_quantity_sold
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      GROUP BY p.product_id, p.product_name
      ORDER BY total_quantity_sold DESC
      LIMIT 5;
 

## 3. Sales Trend Over Time (Monthly)
This query analyzes the sales trend over time, aggregating total sales by month. It helps visualize sales patterns, seasonality, and overall business growth.

      SELECT 
          strftime('%Y-%m', o.order_date) AS month_year,
          SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM orders o
      JOIN order_items oi ON o.order_id = oi.order_id
      GROUP BY month_year
      ORDER BY month_year;


## 4. Customer Segmentation by Purchase Frequency
This query segments customers into 'High', 'Medium', and 'Low' frequency groups based on the number of orders they have placed. It helps understand customer behavior and tailor marketing strategies.

      SELECT 
          customer_id, 
          COUNT(*) AS order_count,
          CASE 
              WHEN COUNT(*) > (SELECT AVG(order_count) + STDDEV(order_count) FROM (SELECT customer_id, COUNT(*) AS order_count FROM orders GROUP BY customer_id)) THEN 'High'
              WHEN COUNT(*) < (SELECT AVG(order_count) - STDDEV(order_count) FROM (SELECT customer_id, COUNT(*) AS order_count FROM orders GROUP BY customer_id)) THEN 'Low'
              ELSE 'Medium'
          END AS purchase_frequency_segment
      FROM orders
      GROUP BY customer_id;
 

## 5. Average Order Value
This query calculates the average order value, providing insights into the typical spending per transaction.

      SELECT AVG(total_order_value) AS average_order_value
      FROM (
          SELECT o.order_id, 
                 SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_order_value
          FROM orders o
          JOIN order_items oi ON o.order_id = oi.order_id
          GROUP BY o.order_id
      ) AS order_values;
 

## 6. Sales Performance by Store
This query compares the sales performance of different stores, helping identify top-performing locations and potential areas for improvement.

      SELECT 
          s.store_id,
          s.store_name,
          SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM orders o
      JOIN order_items oi ON o.order_id = oi.order_id
      JOIN stores s ON o.store_id = s.store_id
      GROUP BY s.store_id, s.store_name
      ORDER BY total_sales DESC;


## 7. Sales Performance by Staff
This query evaluates the sales performance of individual staff members, aiding in performance reviews and identifying top salespersons.

      SELECT 
          st.staff_id,
          st.first_name,
          st.last_name,
          SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM orders o
      JOIN order_items oi ON o.order_id = oi.order_id
      JOIN staffs st ON o.staff_id = st.staff_id
      GROUP BY st.staff_id, st.first_name, st.last_name
      ORDER BY total_sales DESC;


## 8. Product Sales by Brand
This query breaks down sales by brand, helping understand brand popularity and make informed decisions about inventory and marketing.

      SELECT 
          b.brand_name,
          SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      JOIN brands b ON p.brand_id = b.brand_id
      GROUP BY b.brand_name
      ORDER BY total_sales DESC;
 

## 9. Inventory Analysis
This query identifies products with low stock levels (less than 10) and the top 5 products by quantity sold, aiding in inventory management and demand forecasting

      SELECT 
          p.product_id,
          p.product_name,
          s.quantity AS stock_quantity,
          COALESCE(SUM(oi.quantity), 0) AS total_quantity_sold,
          s.quantity - COALESCE(SUM(oi.quantity), 0) AS available_stock
      FROM products p
      LEFT JOIN stocks s ON p.product_id = s.product_id
      LEFT JOIN order_items oi ON p.product_id = oi.product_id
      GROUP BY p.product_id, p.product_name, s.quantity
      HAVING available_stock < 10 
      UNION ALL
      SELECT 
          p.product_id,
          p.product_name,
          s.quantity AS stock_quantity,
          COALESCE(SUM(oi.quantity), 0) AS total_quantity_sold,
          s.quantity - COALESCE(SUM(oi.quantity), 0) AS available_stock
      FROM products p
      LEFT JOIN stocks s ON p.product_id = s.product_id
      LEFT JOIN order_items oi ON p.product_id = oi.product_id
      GROUP BY p.product_id, p.product_name, s.quantity
      ORDER BY total_quantity_sold DESC
      LIMIT 5;


## 10. Order Status Analysis
This query provides a breakdown of the number of orders in each status category, helping track order fulfillment and identify potential bottlenecks.

      SELECT order_status, COUNT(*) AS order_count
      FROM orders
      GROUP BY order_status; 
