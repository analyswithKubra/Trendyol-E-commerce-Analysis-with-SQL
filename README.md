# Trendyol E-Commerce Analysis: Insights for Growth

### Project Overview 
In the fast-paced world of e-commerce, staying ahead means making decisions backed by data. This project dives into a simulated dataset inspired by real-world operations to uncover what drives growth and efficiency. By examining product trends, customer behaviors, and operational performance, the goal is to transform raw data into strategies that matter. As well as providing value to the business and my fellows, this project demonstrates my understanding of both technical and business needs, connecting the dots between raw data and strategic decisions.

### Data Gathering 
I sourced the dataset for this project from Kaggle, where it is available in a CSV format. The dataset is simulated and inspired by real-world e-commerce operations. It contains 8 interconnected tables:
- Orders
- Order_items
- Products
- Customers
- Shippings
- Reviews
- Suppliers
- Payments

These tables allowed me to explore various aspects of an e-commerce business, from customer interactions to product details, providing a comprehensive foundation for analysis.

### Data Modeling 

Data modeling was a key step in this project. To ensure the analysis aligned with the dataset's structure and the company's strategic needs, I carefully designed the data model using Lucidchart. The relationships were based on a combination of one-to-many and one-to-one structures, selected to maintain data integrity, optimize query performance, and support accurate joins. This structured approach provided a reliable foundation for all subsequent analyses and ensured the data was well-organized for business-critical insights.

![Copy of Database ER diagram (crow's foot)](https://github.com/user-attachments/assets/52c63d1a-baf0-426b-8152-ec8a5bf112cf)


### Data Creation 
To prepare the dataset for analysis, I created the necessary tables using SQL. Below is an example query for creating the `order_items` table, which includes details about the items in each order and their relationship to the `orders` and `products` tables.

```sql
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT, -- Foreign key referencing orders
    product_id INT, -- Foreign key referencing products
    quantity INT,
    price_at_purchase DECIMAL(10,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
Similar queries were used to create the other tables, such as `orders`, `products`, `customers`, `shippings`, `reviews`, `suppliers`, and `payments`. Each table was designed with appropriate data types and constraints, including primary and foreign keys, to ensure data integrity and establish meaningful relationships.
This process provided a robust foundation for the database, ensuring accurate joins, optimized query performance, and reliable insights throughout the analysis.


### Data Cleaning Process 

To ensure the dataset was accurate, consistent, and analysis-ready, I performed the following key steps:

**Standardizing Text Data:** Used SQL functions like TRIM() and LOWER() to ensure uniform formatting in columns such as customer names and product categories.

**Handling Missing and Duplicate Data:** Addressed missing values through imputation or removal, and eliminated duplicates using ROW_NUMBER() or DISTINCT to maintain data integrity.

**Validating Referential Integrity:** Verified primary and foreign key constraints to ensure accurate relationships between tables and reliable joins.

**Data Type Verification:** Corrected column data types to match expected formats (e.g., dates, numbers, strings), optimizing SQL operations.

**Outlier Detection:** Identified and resolved outliers using statistical measures and SQL functions such as AVG() and STDDEV() to ensure reliable analysis.


### Exploratory Data Analysis (EDA)
 To become more familiar with the dataset and ensure its validity, I conducted exploratory data analysis. This involved:
- Reviewing distributions and summary statistics for key variables.
- Identifying potential outliers and inconsistencies.
- Validating data relationships to confirm the data aligns with business logic.


### Data Analysis 
In this step, I focused on answering key business questions based on industry challenges and my experience. These questions, queries, and analyses aim to help the business grow, address problem areas, identify top and bottom-performing product categories and suppliers, and understand customer behavior. This information enables informed decisions on investments and other actions. Here are the questions, queries, and analyses:

### 1.Month Over Month Sales Trend 
 To have a general idea about monthly revenue, I wanted to analyze month-over-month sales trends for the last 12 months. I calculated the total sales for each month and used the LAG window function to retrieve sales data from the previous month. This approach highlights fluctuations in sales performance over time, helping to identify growth or decline patterns.!

```sql

WITH monthly_sales AS (
    SELECT 
        YEAR(o.order_date) AS year,
        MONTH(o.order_date) AS month_number,
        MONTHNAME(o.order_date) AS month,
        SUM(oi.total_sales) AS total_sale
    FROM 
        orders o
    JOIN 
        order_items oi ON oi.order_id = o.order_id
    GROUP BY 
        year, month_number, month
    ORDER BY 
        year ASC, month_number ASC
)

SELECT 
    year,
    month_number,
    month,
    LAG(total_sale) OVER (ORDER BY year ASC, month_number ASC) AS last_month_sales
FROM 
    monthly_sales
ORDER BY 
    year ASC, month_number ASC;


```
<img width="647" alt="Screenshot 2025-01-07 at 4 34 24 PM" src="https://github.com/user-attachments/assets/498c9768-897c-476b-aabe-1dfb694c43de" />


The data shows a sharp increase in December 2023, likely due to holiday shopping. January and February 2024 maintained consistent sales, while March saw a slight dip, possibly due to seasonality. Sales from April to September remained steady, averaging between $3.5M and $3.7M. November 2024 showed a noticeable uptick, likely driven by pre-holiday demand. These trends highlight key periods of growth and slight declines, providing clarity for future planning.



### 2.Revenue By Category 

How does each product category contribute to the company's total revenue?

In the previous analysis, we broke down the products and highlighted the best-selling items. To provide deeper insight and understand the connection between product sales and their categories, I analyzed how each category contributes to the company’s overall revenue.
Using this query, I calculated the total revenue for each category and its percentage contribution to the total. This breakdown is essential for making informed decisions about resource allocation, marketing, and inventory management.

```sql
SELECT 
    p.category AS category,
    SUM(oi.total_sales) AS total_sale,
    (SUM(oi.total_sales) * 100.0) / SUM(SUM(oi.total_sales)) OVER () AS percentage_contribution
FROM 
    order_items oi
LEFT JOIN 
    products p ON p.product_id = oi.product_id
GROUP BY 
    p.category;
```

<img width="525" alt="Screenshot 2025-01-07 at 4 41 06 PM" src="https://github.com/user-attachments/assets/9845348d-ea5e-460f-9a3a-93903b47fb3c" />

As we can see, **Electronics** is the top-performing category, contributing 35.88% to the company’s total revenue. **Home & Kitchen** follows with 26.31%, reflecting strong demand for household essentials.
**Accessories,** contributing 23.14%, shows consistent performance, likely from smaller, high-volume items. Compared to other categories, **Furniture** has the smallest share at 14.66%, though it still contributes to diversifying revenue streams.


### 3.Best-Selling Categories by State

Which state generates the highest revenue for each product category, and how much is the total sales within that state?

We initially took a general look at revenue by category and observed that electronics had the highest sales.To explore this further, I wanted to break the data down by state and see which categories were selling the most in each one. I was curious to see if electronics would still be the top seller, or if the trends would be different in some states. Breaking the data down by state helps businesses understand local sales patterns and plan strategies or adjust operations based on what’s happening in each region.

I calculated total sales for each category by state and used the RANK() function to identify the top-selling state for each category. This analysis highlights regional trends and the best-selling categories in different areas.

```sql
WITH sales_data AS (
    SELECT 
        SUBSTRING_INDEX(c.address, ",", -1) AS states,
        p.category AS category,
        SUM(oi.total_sales) AS total_sale
    FROM orders o
    JOIN customers c ON c.customer_id = o.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN products p ON p.product_id = oi.product_id
    GROUP BY p.category, states
),
ranked_data AS (
    SELECT 
        states,
        category,
        total_sale,
        RANK() OVER (PARTITION BY states ORDER BY total_sale DESC) AS ranking
    FROM sales_data
)
SELECT 
    states,
    category,
    total_sale
FROM ranked_data
WHERE ranking = 1;

```

<img width="630" alt="Screenshot 2025-01-07 at 5 16 23 PM" src="https://github.com/user-attachments/assets/6246f219-6e7e-44a9-bdea-3e03835c8e58" />


I wanted to see if electronics is the best-selling category in each state. As we can see, electronics is the top seller in most states, but there are some states, like Tennessee (TN), Wisconsin (WI), and West Virginia (WV), where Home & Kitchen is the best seller. This shows that while we are doing really well with electronics, there is a significant gap between it and other categories. If we don’t want the company to be overly reliant on electronics sales in the future, we may need to focus on increasing sales in other categories. In our business, the goal shouldn’t just be success in one specific category, but to achieve similar revenue across all categories.
