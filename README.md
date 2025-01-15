# Trendyol E-Commerce Analysis: Insights for Growth

Note: This overview provides detailed insights and recommendations. If you wish to view only the SQL queries used in the analysis, please click.sql queries.
https://github.com/analyswithKubra/Trendyol-E-commerce-Analysis-with-SQL/blob/348d87738ca17ee150b4a8335511e74e5650b549/sql%20queries.




### Project Overview 
In the fast-paced world of e-commerce, staying ahead means making decisions backed by data. This project dives into a simulated dataset inspired by real-world operations to uncover what drives growth and efficiency. By examining product trends, customer behaviors, and operational performance, the goal is to transform raw data into strategies that matter. As well as providing value to the business and my fellows, this project demonstrates my understanding of both technical and business needs, connecting the dots between raw data and strategic decisions.
Note:This overview cover detailed insights and rfecommendations.If you want to see only queries please click SQL Queries

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

### 1. Month Over Month Sales Trend 
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



### 2. Revenue By Category 

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


### 3. Best-Selling Categories by State

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


### 4. Top Selling Products 

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


### 7. Customer Lifetime Value 

Which customers have the highest lifetime value, and how can we rank them based on their total contributions to the company?
 
So far, we have focused on total numbers by month, category, and product perspective, and we’ve analyzed the best and worst suppliers. Moving forward, we will delve deeper into customer behavior to provide actionable insights.

To begin, we analyzed customer behavior by calculating the average order value. Additionally, I calculated the total value of orders placed by each customer over their lifetime, a metric known as Customer Lifetime Value (CLTV). This analysis highlights the customers who contribute the most to overall revenue. By using the DENSE_RANK() window function, I ranked customers based on their CLTV, enabling the company to identify high-value customers and prioritize them for loyalty programs, personalized marketing, and exclusive promotions.

```sql
SELECT 
    o.customer_id AS customer_id,
    CONCAT(c.first_name, " ", c.last_name) AS customer_name,
    FORMAT(SUM(oi.total_sales), 0) AS customer_lifetime_value,
    DENSE_RANK() OVER (ORDER BY SUM(oi.total_sales) DESC) AS customer_rank
FROM 
    orders o
JOIN 
    order_items oi ON oi.order_id = o.order_id
JOIN 
    customers c ON c.customer_id = o.customer_id
GROUP BY 
    o.customer_id, c.first_name, c.last_name
ORDER BY 
    customer_rank ASC
LIMIT 20;
```

<img width="666" alt="Screenshot 2025-01-07 at 8 57 09 PM" src="https://github.com/user-attachments/assets/a550ccce-544b-4b9d-b70d-5ccaefb67c5e" />

**Brandy Beck** ranks as the customer with the highest lifetime value, contributing **$19,651** in total sales. This ranking provides insights into customer contributions, enabling the company to identify its most loyal and profitable customers. Understanding these high-value customers allows the business to craft targeted retention strategies, fostering long-term relationships that drive future growth.



### 8. Customers with No Purchases
Who are the customers that registered but did not make any purchases?
 
To identify customers who registered but did not place any orders, I used a combination of a LEFT JOIN and a NOT IN clause. The LEFT JOIN ensures that all customers are included in the result set, regardless of whether they made a purchase or not. The NOT IN condition filters out customers who appear in the orders table, leaving only those who have registered but have not made any transactions. This helps pinpoint inactive customers, enabling the company to strategize ways to re-engage them.
 
```sql
SELECT 
    c.customer_id AS customer_id,
    CONCAT(c.first_name, " ", c.last_name) AS customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id NOT IN (
    SELECT customer_id 
    FROM orders
)
GROUP BY customer_id, customer_name

UNION 

SELECT 
    'N/A' AS customer_id,
    'N/A' AS customer_name
WHERE NOT EXISTS (
    SELECT 1
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    WHERE c.customer_id NOT IN (
        SELECT customer_id 
        FROM orders
    )
);
```

<img width="330" alt="Screenshot 2025-01-07 at 9 01 33 PM" src="https://github.com/user-attachments/assets/41836ad8-ca8e-439a-b4d2-53ad82b16803" />

The query results indicate no inactive customers in the dataset. This means every registered customer has completed at least one purchase. While this is a positive finding, it’s important to continue monitoring customer engagement levels to ensure all users remain active and satisfied.

### 9.Shipment Delays Analysis 

What is the average time taken (in days) for shipments to be processed after an order is placed, and how many shipments took 3 or more days to ship?

We have analyzed customer behaviors so far. Now, I want to analyze something related to shipping performance. I believe shipping is one of the most important factors affecting customer behavior, especially when analyzing e-commerce businesses. According to our business, if the shipping takes more than 3 days after the order is completed, it is considered a shipping delay. To analyze this, I used DATEDIFF() to calculate the days between the order_date and shipment_date. Additionally, I included shipment companies to gain a general understanding of their performance as well.

```sql

SELECT 
  s.shipment_id AS shipment_id,
  CONCAT(c.first_name, " ", c.last_name) AS customer,
  o.order_date AS order_date,
  s.shipment_date AS shipment_date,
  DATEDIFF(s.shipment_date, o.order_date) AS shipment_days,
  s.carrier
FROM shipment s
JOIN orders o ON o.order_id = s.order_id
JOIN customers c ON c.customer_id = o.customer_id
GROUP BY s.shipment_id, c.first_name, c.last_name, o.order_date, s.shipment_date, s.carrier
HAVING shipment_days >= 3;

```
<img width="868" alt="Screenshot 2025-01-07 at 9 18 31 PM" src="https://github.com/user-attachments/assets/1c697ac1-f0b2-4752-9a0e-48de58565d99" />

By analyzing shipping delays, we can assess our shipping performance. As mentioned, shipping performance is a significant factor that affects customers and can even be a major reason for customer churn or retention. We can further analyze the delayed shipping percentage relative to total shipments, or we can examine delays by state to identify if certain states experience more delays than others. Additionally, we can analyze shipping performance by shipping company to pinpoint the main reasons for delays. These analyses provide a general perspective, but we definitely need to explore further and look at the issue from different angles













### 10. Payment Success Rate
What is the percentage of successful payments across all orders, and how do payment success rates vary by payment method and transaction month?
 
So far, we have analyzed various aspects such as categories, products, customer behavior, and shipping delays. To further assess our payment performance, I focused on transaction data to calculate the percentage of successful payments across all orders. This analysis includes a breakdown by payment status, method, and transaction month. Using the WITH clause and advanced window functions like SUM() OVER, I was able to compute both overall status percentages and monthly trends, offering deeper insights. This analysis helps the company monitor payment reliability, detect evolving trends, and assess the performance of different payment methods.
 
```sql
WITH payment_summary AS (
    SELECT 
        p.transaction_status,
        p.payment_method,
        DATE_FORMAT(o.order_date, '%Y-%m') AS transaction_month,
        COUNT(p.payment_id) AS payment_count,
        SUM(p.amount) AS total_amount
    FROM payment p
    JOIN orders o ON p.order_id = o.order_id
    GROUP BY p.transaction_status, p.payment_method, transaction_month
),
payment_percentage AS (
    SELECT 
        transaction_status,
        payment_method,
        transaction_month,
        payment_count,
        total_amount,
        (payment_count * 100.0) / SUM(payment_count) OVER () AS overall_status_percentage,
        (payment_count * 100.0) / SUM(payment_count) OVER (PARTITION BY transaction_month) AS monthly_status_percentage
    FROM payment_summary
)
SELECT 
    transaction_status,
    payment_method,
    transaction_month,
    payment_count,
    total_amount,
    overall_status_percentage,
    monthly_status_percentage
FROM payment_percentage
ORDER BY transaction_month ASC, overall_status_percentage DESC;
  
```
<img width="993" alt="Screenshot 2025-01-07 at 9 05 51 PM" src="https://github.com/user-attachments/assets/b19377f2-e4e7-468b-a247-fd79c2239192" />


The percentage of successful payments provides insight into how reliable the company’s payment systems are. A high success rate suggests operational efficiency, while failed transactions might highlight issues like payment gateway errors or declined cards. By analyzing monthly success rates, the company can identify seasonal patterns or technical issues specific to certain months, such as increased failures during peak periods like holidays. Evaluating payment methods like credit cards, debit cards, and PayPal helps pinpoint methods with higher failure rates. This allows the company to negotiate with providers or improve customer instructions for specific methods.



#### Conclusion 
This project has provided a comprehensive analysis of Trendyol's simulated e-commerce operations, leveraging key data points to drive strategic decision-making. From sales trends and customer behavior to supplier performance and shipping efficiency, the findings have highlighted numerous opportunities for growth and optimization.

While the analysis has given valuable insights, there is potential to further delve into each of the key areas for even more specific decisions. By examining deeper facets such as customer segments, regional preferences, and product-specific trends, Trendyol can make more targeted improvements. Continuous monitoring and fine-tuning of these areas will help in staying competitive and driving profitability.

In conclusion, this data-driven approach connects raw information to strategic business decisions, making it a crucial part of the company’s growth strategy. By implementing the insights from this project, Trendyol can not only address current challenges but also take proactive steps toward sustainable growth.



**Author - Kubra DIZLEK**

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

Stay Updated and Join the Community

To stay updated on SQL, data analysis, and related insights, follow me on social media:

**LinkedIn:** Connect with me professionally.

**ORCID:** Connect for a unique research identifier.

I appreciate your support and am excited to connect with you!
