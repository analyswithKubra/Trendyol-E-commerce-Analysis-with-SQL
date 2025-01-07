# Trendyol E-Commerce Analysis: Insights for Growth

In the fast-paced world of e-commerce, staying ahead means making decisions backed by data. This project dives into a simulated dataset inspired by real-world operations to uncover what drives growth and efficiency. By examining product trends, customer behaviors, and operational performance, the goal is to transform raw data into strategies that matter. Additionally, this project demonstrates my understanding of both technical and business needs, connecting the dots between raw data and strategic decisions.

---

## Project Steps

### 1. Data Gathering

The dataset for this project was sourced from Kaggle, available in CSV format. It is simulated and inspired by real-world e-commerce operations. The dataset contains 8 interconnected tables:

- **Orders**  
- **Order_items**  
- **Products**  
- **Customers**  
- **Shippings**  
- **Reviews**  
- **Suppliers**  
- **Payments**  

These tables allowed me to explore various aspects of an e-commerce business, from customer interactions to product details, providing a comprehensive foundation for analysis.

---

### 2. Data Modeling

Data modeling was a key step in this project. To ensure the analysis aligned with the dataset's structure and the company's strategic needs, I carefully designed the data model using Lucidchart.

The relationships were based on a combination of one-to-many and one-to-one structures, selected to maintain data integrity, optimize query performance, and support accurate joins. This structured approach provided a reliable foundation for all subsequent analyses and ensured the data was well-organized for business-critical insights.

---

### 3. Data Creation

To prepare the dataset for analysis, I created the necessary tables using SQL. Below is an example query for creating the `order_items` table, which includes details about the items in each order and their relationship to the `orders` and `products` tables:

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

---

---

**4.Data Cleaning Process**

Before diving into analysis, I ensured the data was clean, consistent, and ready for querying using SQL. Key steps included:

Standardizing Text Data: Ensured uniformity in columns like customer names and product categories using SQL functions such as TRIM() and LOWER().
Removing Duplicates and Nulls: Used SQL queries to identify and eliminate duplicate rows and handle missing values, maintaining data integrity.
Validating Relationships: Verified primary and foreign key relationships across tables using SQL to ensure accurate joins and referential integrity.
Using SQL for data cleaning allowed for precision and efficiency, ensuring the dataset was ready for detailed analysis and strategic decision-making.

---

**5.Exploratory Data Analysis (EDA)**
To become more familiar with the dataset and ensure its validity, I conducted exploratory data analysis (EDA). This involved:

Reviewing distributions and summary statistics for key variables.
Identifying potential outliers and inconsistencies.
Validating data relationships to confirm the data aligns with business logic

---


