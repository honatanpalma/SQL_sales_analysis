## Inventory and Sales Analysis

### Sales Data Exploration with SQL
This repository showcases fundamental SQL queries for analyzing and exploring sales data within a relational database. The dataset used is the sample database provided by SQL Server Tutorial (https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.zip), offering a practical context for demonstrating core SQL concepts.

### Analysis Overview
The provided SQL scripts cover a range of common sales analysis scenarios, providing a basic foundation for understanding key metrics and trends.

#### Sales Performance:

* **Total sales revenue by product category.**
  * Identification of top-selling products.
  * Monthly sales trends to visualize patterns and growth.
  * Sales performance comparison across stores and staff members.
  * Breakdown of product sales by brand.

* **Customer Behavior:**
  * Segmentation of customers based on their purchase frequency.
  * Calculation of average order value to understand spending habits.

* **Inventory Management:**
  * Analysis of inventory levels to identify low-stock and high-demand products.

* **Order Fulfillment:**
  * Overview of order statuses to track the sales process efficiency.
  
## Getting Started
* **1. Set up the database:**
  * Obtain the sample database from SQL Server Tutorial (https://www.sqlservertutorial.net/getting-started/sql-server-sample-database/).
  * Import the database into your preferred SQL environment.
* **2. Execute the SQL queries:**
* Run the provided scripts to explore and analyze the sales data.
* Feel free to modify and adapt the queries to answer your own business questions.
* Alternatively, you can establish a connection to this example database supabase client libraries:

    Project URL: https://rjyqwfjvlxrpibxyqscc.supabase.co
    
    API Key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJqeXF3Zmp2bHhycGlieHlxc2NjIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjUwNjQ0NzEsImV4cCI6MjA0MDY0MDQ3MX0.MDH3XU2E0UaIE-A0YyWR9vUsGCtCueIIA0gT24crI5U
  
  
  

![image](https://github.com/user-attachments/assets/eb690a7b-2d1e-4cbb-b582-f9b1fd712a39)

### SQL Scripts
The repository contains 10 SQL scripts, each designed to address a specific analysis question. These scripts demonstrate the use of joins, aggregations, and other SQL techniques to extract and transform data directly within the database.

### 1. Total Sales by Category
This query calculates the total sales revenue generated by each product category. It helps identify the most profitable categories and understand the sales # distribution across different product types.

      SELECT c.category_name, 
           SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      JOIN categories c ON p.category_id = c.category_id
      GROUP BY c.category_name
      ORDER BY total_sales DESC;


### 2. Top Selling Products
This query identifies the top 5 best-selling products based on the total quantity sold. It helps understand customer preferences and product popularity.

      SELECT p.product_id, p.product_name, SUM(oi.quantity) AS total_quantity_sold
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      GROUP BY p.product_id, p.product_name
      ORDER BY total_quantity_sold DESC
      LIMIT 5;
 

### 3. Sales Trend Over Time (Monthly)
This query analyzes the sales trend over time, aggregating total sales by month. It helps visualize sales patterns, seasonality, and overall business growth.

      SELECT
        TO_CHAR(o.order_date, 'YYYY-MM') AS month_year,
        SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM
        orders o
        JOIN order_items oi ON o.order_id = oi.order_id
      GROUP BY
        month_year
      ORDER BY
        month_year;


### 4. Customer Segmentation by Purchase Frequency
This query segments customers into 'High', 'Medium', and 'Low' frequency groups based on the number of orders they have placed. It helps understand customer behavior and tailor marketing strategies.

      SELECT
        customer_id,
        COUNT(*) AS order_count,
        CASE
          WHEN COUNT(*) > (
            SELECT
              AVG(order_count) + STDDEV(order_count)
            FROM
              (
                SELECT
                  customer_id,
                  COUNT(*) AS order_count
                FROM
                  orders
                GROUP BY
                  customer_id
              ) AS subquery1
          ) THEN 'High'
          WHEN COUNT(*) < (
            SELECT
              AVG(order_count) - STDDEV(order_count)
            FROM
              (
                SELECT
                  customer_id,
                  COUNT(*) AS order_count
                FROM
                  orders
                GROUP BY
                  customer_id
              ) AS subquery2
          ) THEN 'Low'
          ELSE 'Medium'
        END AS purchase_frequency_segment
      FROM
        orders
      GROUP BY
        customer_id;
 

### 5. Average Order Value
This query calculates the average order value, providing insights into the typical spending per transaction.

      SELECT AVG(total_order_value) AS average_order_value
      FROM (
          SELECT o.order_id, 
                 SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_order_value
          FROM orders o
          JOIN order_items oi ON o.order_id = oi.order_id
          GROUP BY o.order_id
      ) AS order_values;
 

### 6. Sales Performance by Store
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


### 7. Sales Performance by Staff
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


### 8. Product Sales by Brand
This query breaks down sales by brand, helping understand brand popularity and make informed decisions about inventory and marketing.

      SELECT 
          b.brand_name,
          SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_sales
      FROM order_items oi
      JOIN products p ON oi.product_id = p.product_id
      JOIN brands b ON p.brand_id = b.brand_id
      GROUP BY b.brand_name
      ORDER BY total_sales DESC;
 

### 9. Inventory Analysis
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


### 10. Order Status Analysis
This query provides a breakdown of the number of orders in each status category, helping track order fulfillment and identify potential bottlenecks.

      SELECT order_status, COUNT(*) AS order_count
      FROM orders
      GROUP BY order_status; 

**Note**
  * This scripts offers a basic introduction to SQL-based data exploration in a relational database context.
  * The sample dataset and queries provide a foundation for further analysis and can be extended to incorporate more advanced SQL techniques and visualizations.
