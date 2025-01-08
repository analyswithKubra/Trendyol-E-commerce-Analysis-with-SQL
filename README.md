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


### 4.Top Selling Products 

What are the top-selling products by total revenue, and how do these high-revenue products contribute to the company’s overall strategy? 

So far, we have looked at general month-by-month sales and analyzed sales by category. Now, I want to dig a bit further and take a closer look at the data to identify the best-selling products. By identifying which products generate the most revenue, we can evaluate the success of the company’s product offerings, understand consumer demand, and uncover trends that can inform strategic decisions.

```sql
SELECT  
    p.product_name AS product_name,  
    COUNT(o.order_id) AS quantity_sold,  
    SUM(oi.total_sales) AS total_sale  
FROM  
    order_items oi  
JOIN  
    orders o ON o.order_id = oi.order_id  
JOIN  
    products p ON p.product_id = oi.product_id  
GROUP BY  
    p.product_name  
ORDER BY  
    total_sale DESC  
LIMIT 10;
```
<img width="578" alt="Screenshot 2025-01-07 at 7 54 35 PM" src="https://github.com/user-attachments/assets/8bd69ba0-4390-42ee-bf0b-e21640be2b69" />

The 4k monitor, microphones, and standing desks are clearly hitting the mark with customers, showing strong demand for tech and productivity products. Household items like kitchen blenders and air purifiers are also steady performers, proving their ongoing appeal. The water bottle’s spot in the top 10 suggests there’s potential in high-volume or premium-priced items.

These insights give us a clearer picture of what’s working, helping us focus on the right products for inventory and promotions. Moving forward, it’s important to keep a balance across different product categories to ensure steady growth and minimize over-reliance on just a few top sellers. This will allow us to make smarter decisions that benefit the business in the long run.



### 5. Top 5 Suppliers and Their Contribution

Who are the top 10 suppliers based on total sales value, and what percentage of the company’s total revenue do they contribute?

We analyzed which products generate the most revenue to make informed decisions on pricing, stocking, and other product-related strategies. However, it's also crucial to consider our suppliers. As an e-commerce business, we source products from various suppliers to deliver to our customers.
In this analysis, I identified the top 10 suppliers based on total sales and calculated their contribution to overall revenue. Understanding supplier contributions helps the business prioritize relationships with key suppliers driving the most sales.

The query uses a Common Table Expression (CTE) to aggregate total sales per supplier and the company’s total sales, then calculates each supplier's contribution percentage using a CROSS JOIN.

```sql
WITH supplier_sales AS (
    SELECT
        s.supplier_id AS supplier_id,
        s.supplier_name AS supplier_name,
        SUM(oi.total_sales) AS total_sale
    FROM products p
    JOIN order_items oi ON oi.product_id = p.product_id
    JOIN suppliers s ON s.supplier_id = p.supplier_id
    GROUP BY s.supplier_id, s.supplier_name
),
total_sales AS (
    SELECT SUM(total_sale) AS company_total_sales
    FROM supplier_sales
)
SELECT
    ss.supplier_id,
    ss.supplier_name,
    ss.total_sale,
    (ss.total_sale * 100.0) / ts.company_total_sales AS contribution_percentage
FROM supplier_sales ss
CROSS JOIN total_sales ts
ORDER BY ss.total_sale DESC
LIMIT 10;

```
<img width="608" alt="Screenshot 2025-01-07 at 8 30 59 PM" src="https://github.com/user-attachments/assets/ccef2eef-4412-4eb5-957a-55bbc2e99ff5" />

The results highlight the significant contribution of the top 10 suppliers to the company's revenue. Next Level Systems leads with a contribution of 1.3%, followed closely by Precision Suppliers LLC. Together, these suppliers contribute nearly 3% of the company’s total revenue, showcasing their importance to the supply chain.
By understanding these contributions, the company can ensure strong partnerships with these key suppliers while exploring ways to optimize performance with others.



### 6. Suppliers Below Average Sales in the Last 6 Months
Which suppliers have made sales in the last 6 months but have total sales below the average of all suppliers during this period? How much are their sales below the average?

We analyzed the top 10 suppliers by total sales to strengthen relationships with key suppliers. However, this analysis focuses on identifying suppliers who made sales in the last six months but fell below the average total sales during this period.

By highlighting underperforming suppliers, the business can explore reasons for the decline, work with them to improve performance, or consider alternative options. The query calculates total sales over six months, compares them to the average, and identifies the shortfall for each supplier.

```sql
WITH sales_data AS (
    SELECT 
        s.supplier_id AS supplier_id,
        s.supplier_name AS supplier_name,
        MAX(o.order_date) AS last_sale_date,
        SUM(oi.quantity * oi.price_at_purchase) AS total_sales
    FROM suppliers s
    LEFT JOIN products p ON s.supplier_id = p.supplier_id
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH) -- Filter for last 6 months
    GROUP BY s.supplier_id, s.supplier_name
),
avg_sales AS (
    SELECT AVG(total_sales) AS avg_total_sale 
    FROM sales_data
)
SELECT 
    sd.supplier_id,
    sd.supplier_name,
    sd.last_sale_date,
    FORMAT(avg.avg_total_sale - sd.total_sales, 2) AS sales_difference
FROM sales_data sd
CROSS JOIN avg_sales avg 
WHERE sd.total_sales < avg.avg_total_sale
ORDER BY (avg.avg_total_sale - sd.total_sales) DESC
LIMIT 10;    
```
<img width="594" alt="Screenshot 2025-01-07 at 8 38 58 PM" src="https://github.com/user-attachments/assets/bec7518b-0aa3-4629-ab6e-aee02ddb4f31" />

As you can see, Tech Supplies Inc. shows the largest gap below the average. Unified Trading Co. and Global Goods Ltd. also have significant gaps, indicating opportunities for improvement.
By analyzing these gaps, the business can assess whether these suppliers face challenges like low demand, operational issues, or competitive pressure. Addressing these factors could improve supplier performance and strengthen their contributions to overall revenue.












