# Customer Shopping Behavior Analysis

This project is an end-to-end **Customer Shopping Analysis** transforming raw retail data into **actionable business insights** through Python-based data cleaning, SQL analytics, and an interactive Power BI dashboard.

---


## 📊 Project Overview

The goal of this project is to simulate a **corporate-grade, end-to-end data analytics workflow**, demonstrating the ability to translate raw data into **strategic business intelligence** by:

✅ **Data Preparation, Modeling & Exploratory Data Analysis (Python):** Clean and transform the raw dataset for analysis.

✅ **Data Analysis (SQL):** Run analytical business queries to extract insights on customer segments, loyalty, and purchase drivers.

✅ **Visualization & Insights (Power BI):** Interactive dashboard highlighting key patterns and trends for data-driven decision-making.

✅ **Report & Presentation:** Communicate insights and recommendations in a business-focused manner.

<img width="4872" height="2656" alt="image" src="https://github.com/user-attachments/assets/ec9aa909-b6ba-42e2-8f86-af4bec7df255" />

---


### ✅ Run the Notebook — `Customer_Shopping_Behavior_Analysis.ipynb`

This notebook includes:

* 📥 Data Import
* 🔍 Data Exploration (EDA)
* 🧹 Data Cleaning & Feature Engineering
* 🔗 SQL Database Connection (PostgreSQL / MySQL / MS SQL Server)
* ⬆️ Loading Clean Data into SQL Database

### ✅ SQL Analysis

* Create database in SQL Server
* Run Python code (from notebook) to populate the database
* Execute queries in **customer_behavior_sql_queries.sql** to answer business questions

### ✅ Power BI Dashboard

* Connect Power BI to SQL Database
* Open: `customer_behavior_dashboard.pbix`
* Explore interactive visual insights

### ✅ Reporting & Presentation

* Prepare a **business report** summarizing findings
* Create a **presentation deck** using Power BI screenshots & business impact (optionally using **Gamma AI**)
---


## 📊 Dashboard Showcase

The final deliverable is an **interactive Power BI dashboard** that allows business teams to:

* Filter insights by **season**, **location**, and **customer type**
* Track revenue trends and shopping behavior
* Identify high-value segments for marketing

> Designed for **data-driven decisions** and business storytelling.

---

## 🧩 Tech Stack

| Stage         | Technology      | Purpose                           |
| ------------- | --------------- | --------------------------------- |
| Data Cleaning | Python (Pandas) | Transform & enrich raw data       |
| Database      | PostgreSQL      | Scalable data storage & querying  |
| Connector     | SQLAlchemy      | Seamless Python ➝ SQL integration |
| Visualization | Power BI        | Interactive dashboards & insights |

---

## 🔄 Project Workflow

This project follows a **3‑phase data pipeline**:

### ✅ 1️⃣ Data Cleaning & Preparation (Python)

Raw dataset: `customer_shopping_behavior.csv` (3,900 rows)

Key transformations performed:

* 🧹 **Missing Value Imputation**

  * 37 missing `review_rating` values filled using **median rating per product category**
* 🏷️ **Column Standardization**

  * Converted headers to `snake_case` for SQL compatibility
* 🧠 **Feature Engineering**

  * `age_group` via statistical quartiles → Young Adult / Adult / Middle‑aged / Senior
  * `purchase_frequency_days` converted into numeric values (e.g., Weekly → 7)
* 🚫 **Redundant Column Removal**

  * Dropped duplicate column `promo_code_used`

➡️ Final cleaned dataset loaded into PostgreSQL table: **`customer`**

---

### 📈 2️⃣ Data Analysis (SQL)

Below are the core SQL queries used to derive key business insights:

---

#### ✅ Q1 — Total Revenue by Gender

```sql
SELECT gender, SUM(purchase_amount) AS revenue
FROM customer
GROUP BY gender;
```
<img width="240" height="114" alt="Q1 (Revenue by Gender)" src="https://github.com/user-attachments/assets/d70cb58b-af01-4cd8-9085-db1c724ece2e" />

#### ✅ Q2 — Discount Users Spending Above Average

```sql
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes' AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);
```
<img width="424" height="393" alt="Q2 (High-Value Discount Users)" src="https://github.com/user-attachments/assets/c576a0ba-3514-4dd1-8bef-e67107e629c1" />


#### ✅ Q3 — Top 5 Products by Highest Avg Review Rating

```sql
SELECT item_purchased, ROUND(AVG(review_rating::numeric),2) AS "Average Product Rating"
FROM customer
GROUP BY item_purchased
ORDER BY AVG(review_rating) DESC
LIMIT 5;
```
<img width="405" height="204" alt="Q3 (Top Rated Products)" src="https://github.com/user-attachments/assets/7104c786-f4b0-4a03-8c2e-fd0cdaab225b" />


#### ✅ Q4 — Compare Avg Spend: Standard vs Express Shipping

```sql
SELECT shipping_type,
ROUND(AVG(purchase_amount),2)
FROM customer
WHERE shipping_type IN ('Standard','Express')
GROUP BY shipping_type;
```
<img width="288" height="123" alt="Q4 (Shipping Type vs  Spend)" src="https://github.com/user-attachments/assets/16336a97-a97f-4dd2-8d82-3f3245a6f53e" />


#### ✅ Q5 — Subscriber vs Non‑Subscriber Spending & Revenue

```sql
SELECT subscription_status,
       COUNT(customer_id) AS total_customers,
       ROUND(AVG(purchase_amount),2) AS avg_spend,
       ROUND(SUM(purchase_amount),2) AS total_revenue
FROM customer
GROUP BY subscription_status
ORDER BY total_revenue, avg_spend DESC;
```
<img width="622" height="111" alt="Q5 (Subscriber Spend)" src="https://github.com/user-attachments/assets/7fd22a02-2714-4e69-bbf6-90b602eb3e87" />


#### ✅ Q6 — Top 5 Products with Highest Discount Usage

```sql
SELECT item_purchased,
       ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*),2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```
<img width="340" height="207" alt="Q6 (Highest Discount Rate Products)" src="https://github.com/user-attachments/assets/3fb885d8-392c-44de-8564-4595f2e13445" />


#### ✅ Q7 — Customer Segmentation by Purchase History

```sql
WITH customer_type AS (
SELECT customer_id, previous_purchases,
CASE 
    WHEN previous_purchases = 1 THEN 'New'
    WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
    ELSE 'Loyal'
    END AS customer_segment
FROM customer
)

SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```
<img width="417" height="146" alt="Q7 (Customer Segments)" src="https://github.com/user-attachments/assets/e9a1eb9f-e465-416d-be3d-4a0559bb315d" />


#### ✅ Q8 — Top 3 Most Purchased Items per Category

```sql
WITH item_counts AS (
    SELECT category,
           item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```
<img width="539" height="420" alt="Q8 (Top 3 Items per Category)" src="https://github.com/user-attachments/assets/680cd4de-9863-4e51-874b-a3697d057894" />

#### ✅ Q9 — Subscription Likelihood Among Repeat Buyers

```sql
SELECT subscription_status,
       COUNT(customer_id) AS repeat_buyers
FROM customer
WHERE previous_purchases > 5
GROUP BY subscription_status;
```
<img width="368" height="112" alt="Q9 (Repeat Buyers   Subscription)" src="https://github.com/user-attachments/assets/7823d93e-0378-433a-b00b-e11447aa82a8" />


#### ✅ Q10 — Revenue Contribution by Age Group

```sql
SELECT age_group,
       SUM(purchase_amount) AS total_revenue
FROM customer
GROUP BY age_group
ORDER BY total_revenue DESC;
```
<img width="298" height="179" alt="Q10 (Revenue by Age Group)" src="https://github.com/user-attachments/assets/de096e64-2ffa-43f1-95b5-32bca8f6b441" />

---

### 📊 3️⃣ Data Visualization (Power BI)

Live connection to PostgreSQL database to ensure **real‑time, refreshable insights**.
<img width="1347" height="733" alt="Power BI Interactive Dashboard Demo" src="https://github.com/user-attachments/assets/c8293aa2-c49d-46e1-b658-ded466930da6" />


---

## 🔍 Key Insights & Business Questions Answered

| Question                     | Insight                          | Value Delivered                      |
| ---------------------------- | -------------------------------- | ------------------------------------ |
| Revenue by Gender            | Men spent far more               | Revenue focus on male audience       |
| High‑Spending Discount Users | 839 customers identified         | Target for profitable promo strategy |
| Top Rated Products           | Gloves, Sandals, Boots...        | Inventory optimization               |
| Shipping Type Impact         | Spend nearly identical           | Express shipping not revenue driver  |
| Subscriber Value             | Non‑subscribers = $170k+ revenue | Big conversion opportunity           |
| Discount‑Driven Products     | Hat & Sneakers most influenced   | Price sensitivity segmentation       |
| Customer Segmentation        | Loyal: 3,116                     | Retention program potential          |
| Top Products by Category     | "Hero" products identified       | Perfect for marketing campaigns      |
| Repeat Buyers & Subscription | 2,518 not subscribed             | Upsell campaign focus                |
| Revenue by Age Group         | Young Adult leads                | Demographic targeting                |

---

## 🎯 Business Recommendations

Strategic actions based on findings:

✅ Convert **2,518 loyal non‑subscribers** → membership program, special onboarding campaigns

✅ Implement **loyalty rewards** for 3,100+ regular buyers to boost retention

✅ Optimize discount strategy

* Promote price‑sensitive items (Hats, Sneakers)
* Protect margins on high‑demand products

✅ Marketing campaign focus

* "Young Adult" & "Middle-aged" segments → highest spending
* Feature **top performing products** in ads

---

## 📁 Project Structure

```
├── main.py                # Python data cleaning + PostgreSQL loader
├── analysis.sql           # SQL business insights
├── customer_behaviour.pbix # Power BI dashboard file
├── customer_shopping_behavior.csv # Raw dataset
└── README.md              # Project documentation
```

---

## 🏁 Final Outcome

✅ A complete retail analytics system
✅ SQL‑powered decision intelligence
✅ Dashboard‑based story for stakeholders

> This project showcases **data engineering + analytics + visualization** expertise for real‑world business value.

---
