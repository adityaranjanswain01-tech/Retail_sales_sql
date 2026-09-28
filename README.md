# 🛍️ Retail Sales Analysis — SQL Project (PostgreSQL)

An end-to-end SQL project on a retail sales dataset: cleaning the data, exploring it, and answering **20+ business questions** using aggregation, window functions, CTEs and date functions.

## 📑 Table of Contents
- [Project Overview](#-project-overview)
- [Dataset](#-dataset)
- [Database Setup](#-database-setup)
- [Data Cleaning](#-data-cleaning)
- [Data Exploration](#-data-exploration)
- [Business Questions & Solutions](#-business-questions--solutions)
- [Key Findings](#-key-findings)
- [SQL Concepts Used](#-sql-concepts-used)
- [Repository Structure](#-repository-structure)
- [How to Run](#-how-to-run)
- [Author](#-author)

---

## 📌 Project Overview

This project analyzes a retail sales dataset to uncover insights about customer behavior, product category performance, and sales trends over time. It was built to practice and demonstrate real-world SQL skills — from basic filtering to advanced window functions — on a business-style dataset.

**Objectives:**
1. Clean the raw dataset (find and remove null records).
2. Explore the data (customer count, category count, total sales).
3. Answer 20+ business questions covering customers, categories, time trends and revenue.

---

## 🗂 Dataset

- **Table name:** `retail_sales`
- **Source:** *(add your dataset link here, e.g. Kaggle)*
- **Records:** *(add row count here)*

| Column | Type | Description |
|---|---|---|
| `transactions_id` | INT | Unique ID for each transaction |
| `sale_date` | DATE | Date of the sale |
| `sale_time` | TIME | Time of the sale |
| `customer_id` | INT | Unique ID for each customer |
| `gender` | VARCHAR(15) | Customer gender |
| `age` | INT | Customer age |
| `category` | VARCHAR(20) | Product category |
| `quantiy` | INT | Quantity of items sold |
| `price_per_unit` | FLOAT | Price per unit |
| `cogs` | FLOAT | Cost of goods sold |
| `total_sale` | FLOAT | Total value of the sale |

---

## 🏗 Database Setup

```sql
DROP TABLE IF EXISTS retail_sales;

CREATE TABLE retail_sales (
    transactions_id  INT PRIMARY KEY,
    sale_date        DATE,
    sale_time        TIME,
    customer_id      INT,
    gender           VARCHAR(15),
    age              INT,
    category         VARCHAR(20),
    quantiy          INT,
    price_per_unit   FLOAT,
    cogs             FLOAT,
    total_sale       FLOAT
);
```

---

## 🧹 Data Cleaning

**Detect null values across all columns**
```sql
SELECT * FROM retail_sales
WHERE
    transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
    customer_id     IS NULL OR gender     IS NULL OR age       IS NULL OR
    category        IS NULL OR quantiy    IS NULL OR price_per_unit IS NULL OR
    cogs            IS NULL OR total_sale IS NULL;
```

**Remove rows with any null value**
```sql
DELETE FROM retail_sales
WHERE
    transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
    customer_id     IS NULL OR gender     IS NULL OR age       IS NULL OR
    category        IS NULL OR quantiy    IS NULL OR price_per_unit IS NULL OR
    cogs            IS NULL OR total_sale IS NULL;
```

---

## 🔎 Data Exploration

**Total number of sales**
```sql
SELECT COUNT(*) AS total_sales FROM retail_sales;
```

**Total number of unique customers**
```sql
SELECT COUNT(DISTINCT customer_id) AS total_customers FROM retail_sales;
```

**All product categories**
```sql
SELECT DISTINCT category FROM retail_sales;
```

---

## 💼 Business Questions & Solutions

### Q1. Retrieve all sales made on `2022-11-05`
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

### Q2. Transactions where category is 'Clothing' and quantity sold is more than 4, in November 2022
```sql
SELECT *
FROM retail_sales
WHERE
    category = 'Clothing'
    AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND quantiy > 4;
```

### Q3. Total sales and order count for each category
```sql
SELECT
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```

### Q4. Average age of customers who purchased from the 'Beauty' category
```sql
SELECT
    category,
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty'
GROUP BY category;
```

### Q5. All transactions where total sale is greater than 1000
```sql
SELECT *
FROM retail_sales
WHERE total_sale > 1000;
```

### Q6. Total number of transactions made by each gender in each category
```sql
SELECT
    gender,
    category,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY gender, category
ORDER BY category, total_transactions DESC;
```

### Q7. Average sale for each month, and the best-selling month of each year
```sql
SELECT year, month, avg_sale
FROM (
    SELECT
        EXTRACT(YEAR FROM sale_date)  AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER (
            PARTITION BY EXTRACT(YEAR FROM sale_date)
            ORDER BY AVG(total_sale) DESC
        ) AS rnk
    FROM retail_sales
    GROUP BY 1, 2
) AS t1
WHERE rnk = 1;
```

### Q8. Top 5 customers based on the highest total sales
```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_sale
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sale DESC
LIMIT 5;
```

### Q9. Number of unique customers who purchased from each category
```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category;
```

### Q10. Orders by shift — Morning (before 12), Afternoon (12–17), Evening (after 17)
```sql
WITH hourly_sale AS (
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT
    shift,
    COUNT(*) AS total_orders
FROM hourly_sale
GROUP BY shift;
```

### Q11. Total revenue generated by each gender
```sql
SELECT
    gender,
    SUM(total_sale) AS revenue
FROM retail_sales
GROUP BY gender;
```

### Q12. Total quantity sold for each category
```sql
SELECT
    category,
    SUM(quantiy) AS total_quantity
FROM retail_sales
GROUP BY category;
```

### Q13. Category that generated the highest total sales
```sql
SELECT
    category,
    SUM(total_sale) AS total_sale
FROM retail_sales
GROUP BY category
ORDER BY total_sale DESC
LIMIT 1;
```

### Q14. Average `total_sale` for each gender
```sql
SELECT
    gender,
    ROUND(AVG(total_sale)::numeric, 2) AS avg_sale
FROM retail_sales
GROUP BY gender;
```

### Q15. Number of transactions for each month of 2022
```sql
SELECT
    EXTRACT(YEAR FROM sale_date)  AS year,
    EXTRACT(MONTH FROM sale_date) AS month,
    COUNT(transactions_id) AS num_of_transactions
FROM retail_sales
WHERE TO_CHAR(sale_date, 'YYYY') = '2022'
GROUP BY 1, 2
ORDER BY 1, 2;
```

### Q16. Total sales for each month of 2022, sorted highest to lowest
```sql
SELECT
    EXTRACT(YEAR FROM sale_date)  AS year,
    EXTRACT(MONTH FROM sale_date) AS month,
    SUM(total_sale) AS total_sale
FROM retail_sales
WHERE TO_CHAR(sale_date, 'YYYY') = '2022'
GROUP BY 1, 2
ORDER BY total_sale DESC;
```

### Q17. Top 5 transactions with the highest quantity sold
```sql
SELECT *
FROM retail_sales
ORDER BY quantiy DESC, total_sale DESC
LIMIT 5;
```

### Q18. Customer who has spent the most money overall
```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_spent
FROM retail_sales
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 1;
```

### Q19. Average age of customers for each gender
```sql
SELECT
    gender,
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
GROUP BY gender;
```

### Q20. Customers who made more than 5 transactions
```sql
SELECT
    customer_id,
    COUNT(transactions_id) AS total_transactions
FROM retail_sales
GROUP BY customer_id
HAVING COUNT(transactions_id) > 5
ORDER BY total_transactions DESC;
```

### Q21. Categories where total sales are greater than 200,000
```sql
SELECT
    category,
    SUM(total_sale) AS total_sale
FROM retail_sales
GROUP BY category
HAVING SUM(total_sale) > 200000;
```

### Q22. Highest-selling transaction for each category
```sql
SELECT DISTINCT ON (category)
    category,
    transactions_id,
    total_sale
FROM retail_sales
ORDER BY category, total_sale DESC;
```

### Q23. Top 3 customers by total spending within each category
```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        category,
        ROUND(SUM(total_sale)::numeric, 2) AS total_spent,
        RANK() OVER (
            PARTITION BY category
            ORDER BY SUM(total_sale) DESC
        ) AS spend_rank
    FROM retail_sales
    GROUP BY customer_id, category
)
SELECT category, customer_id, total_spent, spend_rank
FROM customer_spending
WHERE spend_rank <= 3
ORDER BY category, spend_rank;
```

### Q24. Percentage contribution of each category to overall sales
```sql
SELECT
    category,
    ROUND(SUM(total_sale)::numeric, 2) AS category_sales,
    ROUND(
        (100.0 * SUM(total_sale) / SUM(SUM(total_sale)) OVER ())::numeric, 2
    ) AS pct_of_total
FROM retail_sales
GROUP BY category
ORDER BY pct_of_total DESC;
```

### Q25. Monthly sales trend and the difference from the previous month
```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', sale_date)::date AS month,
        SUM(total_sale) AS monthly_sales
    FROM retail_sales
    GROUP BY 1
)
SELECT
    month,
    ROUND(monthly_sales::numeric, 2) AS monthly_sales,
    ROUND(
        (monthly_sales - LAG(monthly_sales) OVER (ORDER BY month))::numeric, 2
    ) AS diff_from_prev_month,
    ROUND(
        (100.0 * (monthly_sales - LAG(monthly_sales) OVER (ORDER BY month))
         / LAG(monthly_sales) OVER (ORDER BY month))::numeric, 2
    ) AS pct_change
FROM monthly_sales
ORDER BY month;
```

---

## 📊 Key Findings

> *(Fill this in with your actual results after running the queries — this is the section recruiters read first.)*

- **Customers:** the dataset contains **`N`** unique customers across **`N`** transactions.
- **Top spender:** customer **`ID`** spent the most overall, at **`amount`**.
- **Best category:** **`Category`** generated the highest revenue (**`xx%`** of total sales).
- **Gender split:** average sale per transaction was **`X`** for female and **`Y`** for male customers.
- **Peak period:** the highest average sales occurred in **`month/year`**.
- **Shopping habits:** most orders were placed during the **`shift`** shift.
- **Loyalty:** **`N`** customers made more than 5 transactions.

---

## 🧠 SQL Concepts Used

- Data cleaning: `DELETE`, `IS NULL`, multi-column `WHERE`
- Aggregate functions: `SUM`, `AVG`, `COUNT`, `MAX`
- Filtering: `WHERE` vs `HAVING`
- Grouping: `GROUP BY`, multi-column grouping
- Window functions: `RANK() OVER()`, `LAG() OVER()`, `SUM() OVER()`, `DISTINCT ON`
- Common Table Expressions (CTEs) with `WITH`
- Date/time functions: `EXTRACT`, `TO_CHAR`, `DATE_TRUNC`
- Conditional logic: `CASE WHEN`
- Type casting (`::numeric`) and `ROUND`
- Sorting and limiting: `ORDER BY`, `LIMIT`

---

## 📁 Repository Structure
```
retail-sales-sql-analysis/
├── README.md
├── data/
│   └── retail_sales.csv
└── sql_query_p1.sql
```

## ▶️ How to Run

1. Install PostgreSQL and a client (pgAdmin, DBeaver, or `psql`).
2. Run the table creation script from `sql_query_p1.sql`.
3. Import `retail_sales.csv` into the table (via `COPY` or your client's import tool).
4. Run the cleaning, exploration and analysis queries in order.

```sql
COPY retail_sales FROM 'data/retail_sales.csv' DELIMITER ',' CSV HEADER;
```

---

## 👤 Author

**Jyotiraditya Ranjan Swain**
- GitHub:https://github.com/adityaranjanswain01-tech
- LinkedIn: https://www.linkedin.com/in/jyotiraditya-ranjan-swain

⭐ If you found this project useful, consider giving it a star!
