🇫🇷 [Lire en français](README.md)


# 🛒 E-commerce Data Analysis with SQL

An end-to-end e-commerce data analysis project: data generation, data modeling, analytical queries, and business recommendations.


## 🚀 Why this project?

I wanted to work with a realistic e-commerce dataset without relying on an existing one.
So I chose to generate my own data using SQL — which forced me to think about data modeling before writing any analytical queries.

The main objectives of this project are:

* Measure business performance
* Identify high-value customers and countries
* Detect key products and underperforming areas
* Support marketing decision-making

---

## 🧩 Data Model

The project is built on four tables with clear relationships using a **star schema**.

| Table           | Description                                                                  |
| --------------- | ---------------------------------------------------------------------------- |
| `customers`     | 1,000 customers across 5 countries (2022–2025)                               |
| `products`      | 40 products across 5 categories (Electronics, Home, Sports, Beauty, Fashion) |
| `orders`        | ~13,000 rows — one row per product per order                                 |
| `order_details` | Channel (web/mobile) and payment method                                      |

**Modeling choice:**
`channel` and `payment_method` are separated into `order_details` to reduce redundancy and enable easier channel-based analysis.

---

## ⚙️ Data Generation

Before analyzing data, I ensured strong data consistency with the following constraints:

* `order_date` is always ≥ customer `signup_date`
* Dates range from 2022 to 2025-12-31
* Uneven customer distribution across years
* Each customer places at least 1 order (up to 5)
* Each order contains 1 to 5 products
* Product prices are fixed and realistic

---

## 📊 Analysis

### 1. Time Performance

**Question:** How does revenue evolve over time?

**Insight:**
Revenue shows strong and continuous growth since 2022, peaking in 2025 at €1.25M, representing a +220% increase vs 2024.

---

**Question:** How many new customers are acquired each year?

**Insight:**
The company acquires over 230 customers annually, but growth slows in 2025, marking the lowest acquisition year.

---

**Question:** How does order volume evolve?

**Insight:**
Orders increased significantly, reaching a peak of 2,502 orders in 2025.

---

**Question:** How does the average basket evolve?

**Insight:**
No strong trend, but average order value exceeds €500 from 2024 onward, peaking at ~€514.

---

### 2. Seasonality

**Insight:**
October, November, and especially December consistently outperform the average revenue, while January, March, and June are the weakest months.

---

### 3. Products & Categories

**Key insight:**
Top revenue products include:

* Leather Jacket
* Wireless Headphones
* Smart Watch

These are driven by higher prices rather than volume.

In contrast:

* Portable Charger and Eye Cream generate revenue through volume.

Electronics dominate both in volume and revenue.

---

### 4. Countries

**Insight:**
France is the top market in terms of:

* Revenue (€483K)
* Number of customers
* Average basket size

Customer behavior is relatively consistent across countries, but higher spending explains France’s dominance.

---

### 5. Channels & Payment

**Insight:**
Mobile slightly outperforms web:

* More orders
* Higher revenue
* Slightly higher average basket (+€6)

However, behavior across channels remains very similar.

---

### 6. RFM Segmentation

Customers are segmented based on:

* Recency
* Frequency
* Monetary value

**Key insight:**

* 20% are loyal/high-value customers
* 31.9% are promising customers (strong growth potential)
* Majority are mid-tier customers with conversion potential

---

## 🧠 Key Takeaways

* Growth is driven by **volume, not basket size**
* Strong **seasonality in Q4**
* Revenue depends on a few **high-value products**
* Customer behavior is **consistent across countries**
* Mobile slightly dominates but without major differences
* Customer base is mostly **in development stage**

---

## 💡 Business Recommendations

1. **Leverage Q4 seasonality**
   Launch campaigns early and prioritize high-value products

2. **Reduce low-season impact**
   Run reactivation campaigns during weak months

3. **Convert promising customers**
   Implement loyalty programs and personalized offers

4. **Focus on France & analyze weaker markets**
   Strengthen top-performing markets and investigate low basket sizes

5. **Optimize low-value categories**
   Adjust pricing or introduce cross-selling strategies

6. **Win back lost customers**
   Launch targeted reactivation campaigns

---

## 👤 Author

* Robin Rubangura
* Data Analyst
