# 🛒 Zepto E-Commerce Inventory & Pricing SQL Analysis

An end-to-end quick-commerce inventory and pricing data analysis project using **PostgreSQL, SQL, Python, and Excel** to evaluate stock availability, discount strategies, revenue potential, and product catalog performance.

---

## 📌 Project Overview

This project analyzes real-world e-commerce product catalog data scraped from **Zepto**, one of India's leading quick-commerce platforms. Quick-commerce relies on hyper-local delivery within 10–15 minutes, making optimal inventory stocking, pricing, and stockout prevention critical to business profitability and customer retention.

The objective of this analysis is to evaluate catalog health by examining **3,731 SKUs** across **14 product categories**. Through PostgreSQL queries and analytical modeling, I explored inventory valuation, catalog discount distribution, out-of-stock bottlenecks, and product weight classifications.

---

## 🎯 Business Problem

Quick-commerce platforms operate on thin margins and fast-moving inventory. Unoptimized pricing or inventory gaps directly lead to lost revenue and customer churn. 

This project aims to answer key operational and commercial questions:

- **Revenue Allocation:** Which product categories hold the highest estimated inventory value?
- **Stockout Impact:** How many high-value (MRP > ₹300) products are out of stock, causing revenue loss?
- **Pricing Strategy:** Which categories offer the highest average discounts, and are high-MRP items being discounted appropriately?
- **Packaging & Weight Distribution:** How are products distributed by weight, and how does that fit quick-commerce delivery constraints?
- **Catalog Cleaning:** How can raw catalog data (with currency in paise and invalid zero-MRP rows) be sanitized for business reporting?

---

## 📊 Dataset Overview

The dataset contains scraped catalog details from Zepto's live product inventory. Each record represents an individual **SKU (Stock Keeping Unit)** with attributes detailing pricing, discounts, stock quantities, and package weights.

| Attribute | Description | Data Type | Sample / Unit |
| :--- | :--- | :--- | :--- |
| **`sku_id`** | Synthetic Primary Key | `SERIAL PRIMARY KEY` | `1, 2, 3...` |
| **`category`** | Product Category | `VARCHAR(120)` | `Cooking Essentials, Snacks, etc.` |
| **`name`** | Product Display Name | `VARCHAR(150)` | `Onion, Patanjali Ghee` |
| **`mrp`** | Maximum Retail Price | `NUMERIC(8,2)` | Converted to Rupees (₹) |
| **`discountPercent`** | Applied Discount Percentage | `NUMERIC(5,2)` | `16.00%` |
| **`discountedSellingPrice`** | Final Customer Price | `NUMERIC(8,2)` | Converted to Rupees (₹) |
| **`availableQuantity`** | Inventory Stock Count | `INTEGER` | `3, 10, 50` units |
| **`weightInGms`** | Product Package Weight | `INTEGER` | `1000` grams |
| **`outOfStock`** | Availability Flag | `BOOLEAN` | `TRUE / FALSE` |
| **`quantity`** | Package Unit Count | `INTEGER` | `1` |

### Dataset Summary Statistics
- **Raw Records:** 3,732 rows
- **Cleaned Active SKUs:** 3,731 rows (1 invalid `mrp = 0` record dropped)
- **Product Categories:** 14 unique categories
- **Total Stock Units:** 14,959 units available

---

## 🛠️ Tools & Technologies

| Category | Tool / Technology | Purpose / Application |
| :--- | :--- | :--- |
| **Database & Querying** | PostgreSQL | Relational database storage, schema definition, DDL |
| **Database Management** | pgAdmin 4 | Database creation, data import, CSV mapping |
| **Query Language** | SQL | Data exploration, data cleaning, aggregation, conditional logic |
| **Data Scripting** | Python | Metric verification, raw CSV auditing, data validation |
| **Spreadsheet Analysis** | Microsoft Excel | Workbook validation, pivot tables, quick formula checks |
| **Version Control** | Git / GitHub | Code hosting, documentation, portfolio sharing |

---

## ⚙️ Project Workflow

```
Raw Scraped CSV Data (Paise Values)
            │
            ▼
Data Cleaning & Standardizing (Remove MRP=0, Convert Paise to ₹)
            │
            ▼
PostgreSQL Table Creation & Data Ingestion (pgAdmin 4)
            │
            ▼
Exploratory Data Analysis (EDA - Null checks, SKU counts, Stock breakdown)
            │
            ▼
Business SQL Queries (Aggregations, CASE Statements, Ranking, Grouping)
            │
            ▼
Key Insights Extraction (Revenue drivers, Stockout loss, Discounting)
            │
            ▼
Actionable Business Recommendations
```

---

## 🧹 Data Cleaning & Preparation

Raw inventory catalog datasets contain real-world irregularities. Before running analytical queries, I performed the following data sanitization steps in SQL:

1. **Filtering Zero-Price Records:** Identified and removed 1 record where `mrp = 0` to prevent skewing discount and price averages.
   ```sql
   DELETE FROM zepto WHERE mrp = 0;
   ```
2. **Currency Unit Conversion (Paise to Rupees):** Raw MRP and discounted selling prices were stored in paise (e.g., 2500 paise = ₹25.00). Converted values to Indian Rupees for clear business reporting:
   ```sql
   UPDATE zepto
   SET mrp = mrp / 100.0,
       discountedSellingPrice = discountedSellingPrice / 100.0;
   ```
3. **Data Type Standardization:** Enforced `NUMERIC(8,2)` for financial fields, `BOOLEAN` for stock flags, and `INTEGER` for stock counts.
4. **Encoding Resolution:** Resolved UTF-8 import errors in pgAdmin by converting raw CSV file encoding to `CSV UTF-8`.

---

## 📈 Data Analysis & SQL Techniques

The SQL analysis (`Zepto_SQL_data_analysis.sql`) covers data exploration, inventory valuation, and pricing strategy audit.

### 1. Revenue Estimation per Category
Calculated total potential inventory value by multiplying available quantity with discounted selling price:
```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue DESC;
```
*Why:* Helps supply chain managers understand where inventory capital is tied up.

### 2. High-MRP Products Out of Stock Audit
Identified high-value items (MRP > ₹300) currently out of stock:
```sql
SELECT DISTINCT name, category, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```
*Why:* Stockouts on premium items cause immediate revenue leakage and customer disappointment.

### 3. Top Discounted Categories
Ranked categories by average discount percentage using aggregate and rounding functions:
```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```
*Why:* Evaluates whether margin dilution is concentrated in specific categories.

### 4. Weight Categorization using Conditional Logic
Segmented product SKUs into Low, Medium, and Bulk weight buckets:
```sql
SELECT DISTINCT name, weightInGms,
       CASE WHEN weightInGms < 1000 THEN 'Low (<1kg)'
            WHEN weightInGms < 5000 THEN 'Medium (1-5kg)'
            ELSE 'Bulk (>=5kg)'
       END AS weight_category
FROM zepto;
```
*Why:* Assists logistics teams in planning delivery bag payload limits for 10-minute bike deliveries.

---

## 📌 Key KPIs

Below is the executive KPI summary derived from the sanitized dataset:

| Key Performance Indicator | Value |
| :--- | :--- |
| **Total Active SKUs** | **3,731** |
| **Total Product Categories** | **14** |
| **Total Inventory Stock Units** | **14,959 units** |
| **Total Estimated Inventory Revenue** | **₹22,43,080.60** |
| **Average Product MRP** | **₹156.84** |
| **Average Discounted Selling Price** | **₹141.97** |
| **Overall Catalog Avg Discount** | **7.62%** |
| **In-Stock SKUs** | **3,278 (87.86%)** |
| **Out-of-Stock SKUs** | **453 (12.14%)** |

---

## 💡 Key Insights

### Insight 1 — Revenue Capital Concentrated in Cooking Essentials & Munchies
- **Finding:** Top inventory value is concentrated in daily household staples and impulse snack categories.
- **Evidence:** *Cooking Essentials* (₹3,37,369.00 across 514 SKUs) and *Munchies* (₹3,37,369.00 across 514 SKUs) represent the highest revenue inventory, followed by *Personal Care* (₹2,70,849.00).
- **Business Meaning:** High inventory capital allocation in fast-moving consumer goods (FMCG) aligns with high order frequency, requiring tight reorder cycles.

### Insight 2 — Stockout Leakage on High-MRP Premium Items
- **Finding:** 8 premium products with MRP > ₹300 are currently marked as Out of Stock.
- **Evidence:** High-value items unavailable include **Patanjali Cow's Ghee (MRP ₹565.00)**, **MamyPoko Pants Diapers XL (MRP ₹399.00)**, and **Aashirvaad Multigrain Atta (MRP ₹315.00)**.
- **Business Meaning:** Stockouts on high-ticket daily essentials lead customers to switch to competing quick-commerce apps (e.g., Blinkit, Instamart).

### Insight 3 — Aggressive Discounting in Perishables & Impulsive Snacks
- **Finding:** Fresh produce and perishable categories carry significantly higher discount rates than packaged goods.
- **Evidence:** **Fruits & Vegetables** lead with an average discount of **15.46%**, followed by **Meats, Fish & Eggs (11.03%)**. Individual snack SKUs (e.g., Dukes Waffy Wafers) see discounts up to **51%**.
- **Business Meaning:** Higher discounts in fresh produce are necessary to clear perishable stock before expiration, minimizing wastage.

### Insight 4 — Under-Discounted High-Ticket Staples
- **Finding:** 82 premium products with MRP > ₹500 offer minimal customer discounts (< 10%).
- **Evidence:** Essential cooking oils like **Saffola Gold 5L Jar (MRP ₹1,240, 0% discount)** and **Dhara Mustard Oil Jar (MRP ₹1,250, 8% discount)** have low discount margins.
- **Business Meaning:** High-ticket staples maintain stable demand without heavy discounting, preserving gross margins for the platform.

### Insight 5 — Catalog Tailored for Quick-Commerce Delivery Payload
- **Finding:** Over 90% of inventory items fall into the low-weight package category.
- **Evidence:** **3,392 SKUs (90.9%)** weigh less than 1,000g, while only **46 SKUs (1.2%)** are bulk items (≥ 5,000g).
- **Business Meaning:** Fits the operational delivery model where dark-store riders transport orders on two-wheelers.

---

## 🎯 Business Recommendations

### Recommendation 1: Prioritize Stock Replenishment for High-MRP Staples
- **Action:** Implement automated reorder alerts for SKUs with MRP > ₹300 when inventory drops below 5 units.
- **Reason:** Prevent stockouts on high-ticket items like Ghee, Diapers, and Atta (Insight 2).
- **Expected Impact:** Could capture up to 5–8% in lost top-line revenue and improve retention.

### Recommendation 2: Dynamic Markdown Strategy for Perishables
- **Action:** Implement automated time-decay pricing for *Fruits & Vegetables* to incrementally increase discounts as stock nears shelf-life limit.
- **Reason:** Currently *Fruits & Vegetables* average 15.46% discount overall (Insight 3). Targeted markdowns reduce blanket discounting.
- **Expected Impact:** May reduce shrinkage/spoilage costs while preserving margins on fresh inventory.

### Recommendation 3: Bundle Premium Low-Discount Oils with High-Margin Snacks
- **Action:** Create combo bundles (e.g., Premium Cooking Oil + High-Margin Spices/Snacks).
- **Reason:** High-MRP cooking oils have low discounts (0–8%) but high cart value (Insight 4).
- **Expected Impact:** Could boost Average Order Value (AOV) without discounting core oil products.

---

## 🖼️ Dashboard / Visualization Overview

*(If integrating Power BI or Tableau visual dashboards, place previews here)*

```
+-----------------------------------------------------------------------+
|                       ZEPTO INVENTORY DASHBOARD                       |
+--------------------------+--------------------+-----------------------+
| Total Revenue: ₹22.43L   | Total SKUs: 3,731  | Stockout Rate: 12.14% |
+--------------------------+--------------------+-----------------------+
|  Revenue by Category     |  Stock Status      |  Discount Breakdown   |
|  [ Bar Chart ]           |  [ Donut Chart ]   |  [ Heatmap ]          |
+--------------------------+--------------------+-----------------------+
```

![Dashboard Preview](images/dashboard.png)

---

## 📂 Project Structure

```
Zepto Inventory and Pricing Analysis/
│
├── zepto-SQL-data-analysis-project-main/
│   ├── zepto_v2.csv                   # Raw scraped dataset (3,732 rows)
│   ├── Zepto_SQL_data_analysis.sql    # Full SQL script
│   └── README.md                      # Project documentation
│
├── Zepto_SQL_data_analysis.sql        # Root SQL script (Schema DDL, EDA, Cleaning, Queries)
├── zepto_v2.csv                       # Root CSV dataset
├── zepto_v1.xlsx                      # Excel workbook with calculations
├── promptforreadme.txt                # Prompt instructions file
└── README.md                          # Main project documentation
```

---

## 🚀 How to Run / Reproduce the Project

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/amlanmohanty/zepto-SQL-data-analysis-project.git
   cd zepto-SQL-data-analysis-project
   ```

2. **Database Setup (PostgreSQL):**
   - Open pgAdmin 4 or VS Code SQL extension.
   - Create a database named `zepto_db`.

3. **Execute SQL Script:**
   - Open and execute `Zepto_SQL_data_analysis.sql` to create the table structure.

4. **Import Dataset:**
   - Import `zepto_v2.csv` into table `zepto` (ensure file encoding is `UTF-8`).
   - If using `psql` command line:
     ```sql
     \copy zepto(category, name, mrp, discountPercent, availableQuantity, discountedSellingPrice, weightInGms, outOfStock, quantity)
     FROM 'zepto_v2.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', QUOTE '"', ENCODING 'UTF8');
     ```

5. **Run Data Cleaning & Business Queries:**
   - Execute the cleaning section (zero MRP removal & paise to rupee update).
   - Run business analytical queries Q1 through Q8.

---

## 🛠️ Skills Demonstrated

### Technical Skills
- **SQL & PostgreSQL:** DDL, DML, `CASE WHEN` conditional logic, aggregations (`SUM`, `AVG`, `ROUND`), `GROUP BY`, `HAVING`, sorting, filtering.
- **Data Cleaning:** Handling invalid values, data conversion (paise to rupees), schema definition.
- **Python:** Data validation, script-based verification of metrics.
- **Excel:** Pivot table validation and data inspection.

### Analytical & Business Skills
- **Inventory Analytics:** Revenue valuation, weight bucket analysis, stockout auditing.
- **Pricing Strategy:** Discount distribution analysis, high-MRP margin preservation.
- **Problem Solving:** Converting raw scraped e-commerce data into business recommendations.

---

## 💡 Key Learnings

- **Handling Raw Currency Formats:** Discovered raw e-commerce data often stores monetary metrics in smaller denominations (paise) to prevent floating-point rounding errors during online transactions.
- **Quick-Commerce Operational Realities:** Learned how product package weights directly impact hyper-local delivery feasibility.
- **Stockout Value Risk:** Recognized that stockouts in quick commerce carry higher customer attrition risk compared to traditional e-commerce.

---

## 📝 Conclusion

This project demonstrates practical data analyst competencies in **PostgreSQL data cleaning, inventory analysis, and pricing evaluation**. By analyzing 3,731 SKUs from Zepto, I translated raw catalog attributes into actionable insights regarding revenue concentration, high-value stockout risks, and weight-based logistics planning.


