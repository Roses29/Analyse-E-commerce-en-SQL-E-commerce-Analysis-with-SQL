🇫🇷 [Lire en Français](README.md)

# E-Commerce Analysis in SQL
A data analysis project built from scratch: data generation, modelling, analytical queries and recommendations.

## Why this project?

I wanted to work on a realistic e-commerce dataset without relying on an existing one. I chose to generate my own data in SQL — which forced me to think about the data model before writing a single analytical query. The concrete objectives of the project are:

**• measure business performance**

**• identify high-value countries and customers**

**• detect key products and weak spots**

**• support marketing decision-making**

## Schema

Four tables, clear relationships and a star model.

```
customers ──► orders ◄── order_details
                │
                ▼
            products
```

| Table | Content |
|-------|---------|
| `customers` | 1,000 customers, 5 countries, signups between 2022 and 2025 |
| `products` | 40 products, 5 categories (Electronics, Home, Sports, Beauty and Fashion), realistic prices |
| `orders` | ~13,000 rows — 1 row per product per order |
| `order_details` | 6 combinations of channel (web/mobile) × payment method |

**Modelling decision:** `channel` and `payment_method` are isolated in `order_details` to avoid redundancy in `orders` and to facilitate channel-based analysis.

---

## Data Generation

Before analysing, coherent data is needed. A few constraints I set for myself:

- `order_date` always ≥ customer's `signup_date`
- All dates bounded between 2022 and 2025-12-31
- Irregular customer distribution across the 4 years (no fixed quota per year)
- Variable number of orders per customer: all place at least 1 order, some place up to 5
- 1 to 5 products per order
- Prices fixed per product (realistic, not random)

```sql
-- CUSTOMERS TABLE

CREATE TABLE customers (
    customer_id  INTEGER PRIMARY KEY AUTOINCREMENT,
    signup_date  TEXT,
    country      TEXT,
    gender       TEXT,
    age          INTEGER
);

INSERT INTO customers (signup_date, country, gender, age)
SELECT
    date('2022-01-01', '+' || abs(random() % 1461) || ' days'),
    CASE abs(random() % 5)
        WHEN 0 THEN 'France'
        WHEN 1 THEN 'Germany'
        WHEN 2 THEN 'Spain'
        WHEN 3 THEN 'Italy'
        ELSE        'Belgium'
    END,
    CASE abs(random() % 2)
        WHEN 0 THEN 'M'
        ELSE        'F'
    END,
    18 + abs(random() % 50)
FROM (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
      UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10) a,
     (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
      UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10) b,
     (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
      UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10) c;


-- PRODUCTS TABLE — 40 unique products (8 per category)

CREATE TABLE products (
    product_id   INTEGER PRIMARY KEY AUTOINCREMENT,
    product_name TEXT,
    category     TEXT,
    unit_price   REAL
);

INSERT INTO products (product_name, category, unit_price) VALUES
-- Electronics: €15 to €299
('Wireless Headphones',  'Electronics', ROUND(49  + abs(random() % 150), 2)),
('Bluetooth Speaker',    'Electronics', ROUND(29  + abs(random() % 100), 2)),
('USB-C Hub',            'Electronics', ROUND(15  + abs(random() % 45),  2)),
('Mechanical Keyboard',  'Electronics', ROUND(59  + abs(random() % 120), 2)),
('Webcam HD',            'Electronics', ROUND(39  + abs(random() % 80),  2)),
('Smart Watch',          'Electronics', ROUND(99  + abs(random() % 200), 2)),
('Portable Charger',     'Electronics', ROUND(19  + abs(random() % 50),  2)),
('LED Desk Lamp',        'Electronics', ROUND(25  + abs(random() % 60),  2)),
-- Fashion: €12 to €250
('Slim Fit Jeans',       'Fashion',     ROUND(29  + abs(random() % 70),  2)),
('Wool Sweater',         'Fashion',     ROUND(39  + abs(random() % 80),  2)),
('Leather Jacket',       'Fashion',     ROUND(89  + abs(random() % 160), 2)),
('Running Sneakers',     'Fashion',     ROUND(49  + abs(random() % 120), 2)),
('Canvas Tote Bag',      'Fashion',     ROUND(12  + abs(random() % 40),  2)),
('Silk Scarf',           'Fashion',     ROUND(25  + abs(random() % 75),  2)),
('Chino Pants',          'Fashion',     ROUND(29  + abs(random() % 60),  2)),
('Polo Shirt',           'Fashion',     ROUND(19  + abs(random() % 50),  2)),
-- Home: €8 to €200
('Bamboo Cutting Board', 'Home',        ROUND(12  + abs(random() % 30),  2)),
('Coffee Maker',         'Home',        ROUND(39  + abs(random() % 120), 2)),
('Throw Pillow',         'Home',        ROUND(12  + abs(random() % 35),  2)),
('Scented Candle',       'Home',        ROUND(8   + abs(random() % 30),  2)),
('Air Purifier',         'Home',        ROUND(59  + abs(random() % 140), 2)),
('Storage Basket',       'Home',        ROUND(10  + abs(random() % 30),  2)),
('Non-stick Pan',        'Home',        ROUND(25  + abs(random() % 70),  2)),
('Wall Clock',           'Home',        ROUND(15  + abs(random() % 50),  2)),
-- Beauty: €5 to €120
('Vitamin C Serum',      'Beauty',      ROUND(19  + abs(random() % 60),  2)),
('Moisturizing Cream',   'Beauty',      ROUND(12  + abs(random() % 45),  2)),
('Hair Mask',            'Beauty',      ROUND(8   + abs(random() % 30),  2)),
('Lip Balm Set',         'Beauty',      ROUND(5   + abs(random() % 20),  2)),
('Face Wash Gel',        'Beauty',      ROUND(8   + abs(random() % 25),  2)),
('Eye Contour Cream',    'Beauty',      ROUND(15  + abs(random() % 50),  2)),
('Perfume 50ml',         'Beauty',      ROUND(35  + abs(random() % 85),  2)),
('Nail Polish Kit',      'Beauty',      ROUND(10  + abs(random() % 30),  2)),
-- Sports: €5 to €100
('Yoga Mat',             'Sports',      ROUND(19  + abs(random() % 50),  2)),
('Resistance Bands',     'Sports',      ROUND(8   + abs(random() % 25),  2)),
('Water Bottle 750ml',   'Sports',      ROUND(10  + abs(random() % 25),  2)),
('Foam Roller',          'Sports',      ROUND(15  + abs(random() % 35),  2)),
('Jump Rope',            'Sports',      ROUND(5   + abs(random() % 20),  2)),
('Gym Gloves',           'Sports',      ROUND(10  + abs(random() % 25),  2)),
('Protein Shaker',       'Sports',      ROUND(8   + abs(random() % 20),  2)),
('Sports Towel',         'Sports',      ROUND(5   + abs(random() % 20),  2));


-- ORDER_DETAILS TABLE

CREATE TABLE order_details (
    order_detail_id  INTEGER PRIMARY KEY AUTOINCREMENT,
    channel          TEXT,
    payment_method   TEXT
);

INSERT INTO order_details (channel, payment_method) VALUES
('web',    'card'),
('web',    'E-wallet'),
('web',    'Klarna'),
('mobile', 'card'),
('mobile', 'E-wallet'),
('mobile', 'Klarna');


-- ORDERS TABLE

CREATE TABLE orders (
    order_id        INTEGER,
    customer_id     INTEGER,
    order_detail_id INTEGER,
    order_date      TEXT,
    product_id      INTEGER,
    quantity        INTEGER,
    unit_price      REAL,
    FOREIGN KEY (customer_id)     REFERENCES customers(customer_id),
    FOREIGN KEY (order_detail_id) REFERENCES order_details(order_detail_id),
    FOREIGN KEY (product_id)      REFERENCES products(product_id)
);

-- Step 1: temporary orders table (1 row = 1 unique order)
CREATE TEMPORARY TABLE temp_orders AS
SELECT
    ROW_NUMBER() OVER () AS order_id,
    c.customer_id,
    1 + abs(random() % 6) AS order_detail_id,
    date(c.signup_date, '+' || (abs(random() % MAX(1, CAST(julianday('2025-12-31') - julianday(c.signup_date) AS INTEGER)))) || ' days') AS order_date
FROM customers c,
     (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3) extra  -- base ~3000 orders
UNION ALL
SELECT
    ROW_NUMBER() OVER () + 3000,
    c.customer_id,
    1 + abs(random() % 6),
    date(c.signup_date, '+' || (abs(random() % MAX(1, CAST(julianday('2025-12-31') - julianday(c.signup_date) AS INTEGER)))) || ' days')
FROM customers c WHERE abs(random() % 4) < 3   -- 75%
UNION ALL
SELECT
    ROW_NUMBER() OVER () + 4000,
    c.customer_id,
    1 + abs(random() % 6),
    date(c.signup_date, '+' || (abs(random() % MAX(1, CAST(julianday('2025-12-31') - julianday(c.signup_date) AS INTEGER)))) || ' days')
FROM customers c WHERE abs(random() % 2) = 0;  -- 50%

-- Step 2: split each order into 1 to 5 product lines
INSERT INTO orders (order_id, customer_id, order_detail_id, order_date, product_id, quantity, unit_price)
SELECT
    t.order_id,
    t.customer_id,
    t.order_detail_id,
    t.order_date,
    p.product_id,
    1 + abs(random() % 5)  AS quantity,
    p.unit_price
FROM temp_orders t
JOIN (SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3
      UNION ALL SELECT 4 UNION ALL SELECT 5) serie
  ON serie.n <= 1 + abs(random() % 5)
JOIN (
    SELECT product_id, unit_price,
           ROW_NUMBER() OVER () AS rn
    FROM products
) p ON p.rn = 1 + abs(random() % 40);

-- Cleanup
DROP TABLE IF EXISTS temp_orders;
```

After running the script above, I get the following data model:

![Data model](./images/Modèle_data_ecommerce.png)

---

## Analyses

### 1. Revenue Over Time

I start with a time-based view to quickly identify trends before trying to explain them.

**Question:** How has revenue evolved year over year?

```sql
SELECT strftime('%Y',o.order_date) AS Year,
       ROUND(SUM(o.quantity * o.unit_price), 2) AS Revenue
FROM orders o
GROUP BY Year
ORDER BY Year;
```

**Result:**

| Year | Revenue |
|------|---------|
| 2022 | 65,115 |
| 2023 | 242,985 |
| 2024 | 570,158 |
| 2025 | 1,257,904 |

There is a strong and consistent revenue growth since 2022. Revenue peaked in 2025 at €1,257,904, representing a 220% increase compared to 2024.

**Question:** How many new customers were acquired each year?

```sql
SELECT strftime('%Y', signup_date) AS year, COUNT(customer_id) AS new_customers
FROM customers
GROUP BY year;
```

**Result:**

| Year | New customers |
|------|--------------|
| 2022 | 247 |
| 2023 | 250 |
| 2024 | 267 |
| 2025 | 236 |

The business acquires over 230 new customers each year, but the 2025 figure is the lowest across all years and lower than 2024.

**Question:** How has the number of orders evolved?

```sql
SELECT
    strftime('%Y', order_date)  AS year,
    COUNT(DISTINCT order_id)    AS number_of_orders
FROM orders
GROUP BY year
ORDER BY year;
```

**Result:**

| Year | Number of orders |
|------|-----------------|
| 2022 | 133 |
| 2023 | 505 |
| 2024 | 1,110 |
| 2025 | 2,502 |

The number of orders has grown strongly and consistently since 2022, peaking at 2,502 orders in 2025.

**Question:** How has the average order value evolved?

```sql
SELECT
    strftime('%Y', o.order_date)                                          AS year,
    ROUND(SUM(o.quantity * o.unit_price) / COUNT(DISTINCT o.order_id), 2) AS avg_basket
FROM orders o
GROUP BY year
ORDER BY year;
```

**Result:**

| Year | avg_basket |
|------|-----------|
| 2022 | 489.59 |
| 2023 | 481.16 |
| 2024 | 513.66 |
| 2025 | 502.76 |

There is no clear trend, but the average basket exceeded €500 from 2024 onwards, peaking that same year at approximately €514.

---

### 2. Seasonality

**Question:** Do certain months consistently generate more revenue? (All years combined)

For each month, I calculate the average revenue and the percentage deviation from the global average — this reveals genuine seasonal peaks independently of year-on-year growth.

```sql
SELECT
    mois,
    ROUND(ca_moyen, 2)                          AS avg_monthly_revenue,
    ROUND(moyenne_globale, 2)                   AS global_average,
    ROUND((ca_moyen - moyenne_globale) * 100.0
          / moyenne_globale, 1)                 AS deviation_pct
FROM (
    SELECT
        mois,
        ca_moyen,
        AVG(ca_moyen) OVER ()                   AS moyenne_globale
    FROM (
        SELECT
            strftime('%m', ca.order_date)       AS mois,
            AVG(ca.ca_mensuel)                  AS ca_moyen
        FROM (
            SELECT
                ord.order_date                     AS order_date,
                SUM(ord.unit_price * ord.quantity) AS ca_mensuel
            FROM orders ord
            GROUP BY strftime('%Y', ord.order_date), strftime('%m', ord.order_date)
        ) ca
        GROUP BY strftime('%m', ca.order_date)
    )
)
ORDER BY deviation_pct DESC;
```

**Result:**

| Month | avg_monthly_revenue | global_average | deviation_pct |
|-------|--------------------|--------------------|---------------|
| 12 | 77,647 | 46,036 | +68.7% |
| 10 | 64,218 | 46,036 | +39.5% |
| 11 | 59,247 | 46,036 | +28.7% |
| 08 | 46,837 | 46,036 | +1.7% |
| 09 | 44,794 | 46,036 | -2.7% |
| 05 | 42,553 | 46,036 | -7.6% |
| 07 | 40,155 | 46,036 | -12.8% |
| 02 | 37,636 | 46,036 | -18.2% |
| 04 | 36,152 | 46,036 | -21.5% |
| 01 | 35,930 | 46,036 | -22.0% |
| 06 | 34,660 | 46,036 | -24.7% |
| 03 | 32,604 | 46,036 | -29.2% |

**October, November and especially December** generate above-average revenue — these are the peak sales months. **January, June and March** are the weakest months in terms of sales.

---

### 3. Products & Categories

**Question:** Which products generate the most revenue? Are the best-sellers the most profitable?

I analyse both dimensions separately as they can tell opposite stories: a €9.99 product sold 500 times generates less revenue than a €199.99 product sold 80 times.

```sql
SELECT p.product_name, p.category,
       SUM(o.quantity)                          AS quantity_sold,
       ROUND(SUM(o.quantity * o.unit_price), 2) AS revenue,
       ROUND(SUM(o.quantity * o.unit_price)
             / SUM(o.quantity), 2)              AS avg_price
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id
ORDER BY revenue DESC;
```

**Result:**

| product_name | category | quantity_sold | revenue | avg_price |
|--------------|----------|--------------|---------|-----------|
| Leather Jacket | Fashion | 937 | 176,156 | 188 |
| Wireless Headphones | Electronics | 1,027 | 171,509 | 167 |
| Smart Watch | Electronics | 897 | 170,430 | 190 |
| Running Sneakers | Fashion | 1,042 | 142,754 | 137 |
| Mechanical Keyboard | Electronics | 1,044 | 105,444 | 101 |
| Perfume 50ml | Beauty | 1,033 | 96,069 | 93 |
| Webcam HD | Electronics | 997 | 91,724 | 92 |
| Coffee Maker | Home | 944 | 90,624 | 96 |
| Wool Sweater | Fashion | 1,002 | 87,174 | 87 |
| Air Purifier | Home | 954 | 78,228 | 82 |

The **Leather Jacket, Wireless Headphones and Smart Watch** are the top 3 revenue-generating products (€170,000+), primarily due to their higher price point. The **Portable Charger and Eye Contour Cream** are examples of products that generate revenue through volume rather than unit price.

**Question:** What are the 10 best-selling products?

```sql
SELECT p.product_id, p.product_name, p.category,
       SUM(o.quantity) AS quantity_sold
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY quantity_sold DESC
LIMIT 10;
```

**Result:**

| product_id | product_name | category | quantity_sold |
|------------|--------------|----------|--------------|
| 7 | Portable Charger | Electronics | 1,084 |
| 4 | Mechanical Keyboard | Electronics | 1,044 |
| 12 | Running Sneakers | Fashion | 1,042 |
| 2 | Bluetooth Speaker | Electronics | 1,038 |
| 8 | LED Desk Lamp | Electronics | 1,033 |
| 31 | Perfume 50ml | Beauty | 1,033 |
| 1 | Wireless Headphones | Electronics | 1,027 |
| 30 | Eye Contour Cream | Beauty | 1,026 |
| 34 | Resistance Bands | Sports | 1,020 |
| 27 | Hair Mask | Beauty | 1,009 |

5 of the top 10 best-selling products are in the **Electronics** category, and the **Portable Charger** is the single most sold product over the 4-year period.

**Question:** What are the 10 least-sold products?

```sql
SELECT p.product_id, p.product_name, p.category,
       SUM(o.quantity) AS quantity_sold
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY quantity_sold ASC
LIMIT 10;
```

**Result:**

| product_id | product_name | category | quantity_sold |
|------------|--------------|----------|--------------|
| 9 | Slim Fit Jeans | Fashion | 834 |
| 28 | Lip Balm Set | Beauty | 843 |
| 24 | Wall Clock | Home | 846 |
| 33 | Yoga Mat | Sports | 863 |
| 3 | USB-C Hub | Electronics | 868 |
| 6 | Smart Watch | Electronics | 897 |
| 40 | Sports Towel | Sports | 898 |
| 16 | Polo Shirt | Fashion | 912 |
| 25 | Vitamin C Serum | Beauty | 912 |
| 20 | Scented Candle | Home | 925 |

**Question:** What is the revenue and quantity sold by category?

```sql
SELECT p.category,
       SUM(o.quantity)                          AS quantity_sold,
       SUM(o.quantity * o.unit_price)           AS revenue
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.category
ORDER BY quantity_sold DESC;
```

**Result:**

| category | quantity_sold | revenue |
|----------|--------------|---------|
| Electronics | 7,988 | 702,389 |
| Sports | 7,639 | 145,823 |
| Beauty | 7,639 | 301,858 |
| Fashion | 7,562 | 631,858 |
| Home | 7,516 | 354,234 |

The **Electronics** category sells the most and generates the highest revenue, driven by higher unit prices. **Home** products sell the least but generate more revenue than **Beauty and Sports**.

---

### 4. Countries

**Question:** What is the revenue by country? Which market performs best?

```sql
SELECT
    c.country,
    SUM(o.quantity * o.unit_price) AS revenue_by_country
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.country
ORDER BY revenue_by_country DESC;
```

**Result:**

| country | revenue_by_country |
|---------|-------------------|
| France | 483,866 |
| Spain | 452,321 |
| Italy | 421,936 |
| Belgium | 393,177 |
| Germany | 384,862 |

**France** is the largest market in terms of revenue (€483,866), followed by Spain and Italy both exceeding €420,000.

**Question:** How many customers per country?

| country | nb_clients |
|---------|-----------|
| France | 216 |
| Spain | 211 |
| Italy | 199 |
| Germany | 188 |
| Belgium | 186 |

The largest market in terms of customers is **France (216)**, but the gap is not significant compared to **Spain (211)** and **Italy (199)**. The business has fewer customers in Germany (188) and Belgium (186).

**Question:** What are the purchasing behaviours by country?

To answer this, I analyse the average number of orders per customer, the average order basket and the average basket per customer:

```sql
SELECT
    c.country,
    COUNT(DISTINCT o.order_id)                              AS nb_orders,
    COUNT(DISTINCT c.customer_id)                           AS nb_customers,
    ROUND(COUNT(DISTINCT o.order_id) * 1.0
          / COUNT(DISTINCT c.customer_id), 1)               AS orders_per_customer,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT o.order_id), 2)                  AS avg_order_basket,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT c.customer_id), 2)               AS avg_customer_basket
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.country
ORDER BY avg_customer_basket DESC;
```

**Result:**

| country | nb_orders | nb_customers | orders_per_customer | avg_order_basket | avg_customer_basket |
|---------|-----------|-------------|--------------------|-----------------|--------------------|
| France | 926 | 216 | 4.3 | 522.53 | 2,240.12 |
| Spain | 899 | 211 | 4.3 | 503.14 | 2,143.70 |
| Italy | 853 | 199 | 4.3 | 494.65 | 2,120.28 |
| Belgium | 786 | 186 | 4.2 | 500.23 | 2,113.85 |
| Germany | 786 | 188 | 4.2 | 489.65 | 2,047.14 |

Orders per customer are similar across countries (4.2 or 4.3), showing that repurchase behaviour is homogeneous and does not explain the revenue gaps between markets. However, looking at the **average order basket and average customer basket**, customers in France spend more than those in other countries. Belgium and Germany show similar order and customer volumes, but Belgian customers spend slightly more.

Overall, repurchase frequency is low, but customers tend to spend significant amounts when they do purchase.

---

### 5. Channels & Payment

**Question:** Do purchasing behaviours differ between web and mobile?

I look at the average basket per channel — not just volume — to see whether mobile customers buy differently from web customers.

```sql
SELECT
    od.channel,
    COUNT(DISTINCT o.order_id)                      AS nb_orders,
    ROUND(SUM(o.quantity * o.unit_price), 2)        AS revenue,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT o.order_id), 2)          AS avg_basket
FROM orders o
INNER JOIN order_details od ON o.order_detail_id = od.order_detail_id
GROUP BY od.channel;
```

**Result:**

| channel | nb_orders | revenue | avg_basket |
|---------|-----------|---------|-----------|
| mobile | 2,158 | 1,091,064 | 505.59 |
| web | 2,092 | 1,045,098 | 499.57 |

Over the 2022–2025 period, there were more mobile orders (2,158) than web orders (2,092), and mobile generated more revenue. The average basket is slightly higher for mobile than web (+€6).

---

### 6. RFM Segmentation

**Question:** How can customers be classified by purchasing behaviour to target the right marketing actions?

RFM is the logical conclusion of the project: after understanding global trends, I drill down to the customer level. Each customer receives a score (from 1 to 5) on 3 dimensions — Recency (R), Frequency (F) and Monetary value (M) — and is then assigned to a segment.

| Segment | Condition | Action |
|---------|-----------|--------|
| Champion | R=5, F=5, M=5 | VIP programme |
| Loyal customer | F=5 | Reward |
| New customer | R=5, F=1 | Encourage 2nd purchase |
| At-risk customer | R≤2, F≥4 | Re-engage quickly |
| Promising customer | R≥3, F≥3 | Nurturing |
| Inactive customer | R≤2, F≤3 | Reactivation |
| Lost customer | R=1, F=1, M=1 | Win-back or abandon |

```sql
WITH rfm_base AS (
    SELECT
        c.customer_id,
        c.country,
        c.gender,
        c.age,
        CAST(julianday('2025-12-31') - julianday(MAX(o.order_date)) AS INTEGER) AS recency,
        COUNT(DISTINCT o.order_id)                                               AS frequency,
        ROUND(SUM(o.unit_price * o.quantity), 2)                                 AS monetary
    FROM customers c
    INNER JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id
),
rfm_scores AS (
    SELECT
        customer_id, country, gender, age,
        recency, frequency, monetary,
        6 - NTILE(5) OVER (ORDER BY recency ASC)   AS score_r,
        NTILE(5) OVER (ORDER BY frequency ASC)     AS score_f,
        NTILE(5) OVER (ORDER BY monetary ASC)      AS score_m
    FROM rfm_base
),
rfm_final AS (
    SELECT
        customer_id, country, gender, age,
        recency, frequency, monetary,
        score_r, score_f, score_m,
        (score_r + score_f + score_m) AS rfm_score,
        CASE
            WHEN score_r = 5 AND score_f = 5 AND score_m = 5 THEN 'Champion'
            WHEN score_f = 5                                  THEN 'Loyal customer'
            WHEN score_r = 5 AND score_f = 1                  THEN 'New customer'
            WHEN score_r <= 2 AND score_f >= 4                THEN 'At-risk customer'
            WHEN score_r = 1 AND score_f = 1 AND score_m = 1  THEN 'Lost customer'
            WHEN score_r >= 3 AND score_f >= 3                THEN 'Promising customer'
            WHEN score_r <= 2 AND score_f <= 3                THEN 'Inactive customer'
            WHEN score_r >= 4 AND score_f = 2                 THEN 'Engaging customer'
            ELSE 'Average customer'
        END AS segment
    FROM rfm_scores
)
SELECT segment,
       COUNT(*)                                            AS nb_customers,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct
FROM rfm_final
GROUP BY segment
ORDER BY nb_customers DESC;
```

**Result:**

| segment | nb_customers | pct |
|---------|-------------|-----|
| Promising customer | 249 | 24.9% |
| Inactive customer | 217 | 21.7% |
| Loyal customer | 163 | 16.3% |
| Average customer | 124 | 12.4% |
| At-risk customer | 73 | 7.3% |
| Engaging customer | 70 | 7.0% |
| Lost customer | 40 | 4.0% |
| Champion | 37 | 3.7% |
| New customer | 27 | 2.7% |

**Loyal customers and Champions** make up approximately 20% of the base — the loyal core of the business, few in number but with high value potential. **Promising and engaging customers (31.9%)** represent the most immediate growth opportunity: recently active (R score between 3 and 5) with a decent purchase frequency (F score between 2 and 5), they are best positioned to move into the loyal segment with the right marketing actions.

**Average customers** (12.4%) show an ambiguous intermediate profile — not loyal enough, not recent enough, and not clearly at risk. They can also be targeted for loyalty initiatives.

On the warning side, **lost customers** (40 customers, 4%) and **new customers** (27 customers, 2.7%) remain minority segments — the former need a reactivation campaign, the latter need careful onboarding to avoid remaining one-time buyers.

---

## Conclusions & Recommendations

### Key Findings

**Strong growth driven by volume, not by basket size**

Revenue increased 19x between 2022 and 2025, driven primarily by a surge in order volume (+1,780%) rather than a higher average basket, which remained stable at €490–515. This indicates that growth is built on customer acquisition and order frequency rather than on upselling or premiumisation.

**A marked seasonality in Q4**

October, November and December concentrate the sales peaks, with December reaching +68.7% above the monthly average. Conversely, January, March and June are the weakest months (-22% to -29%). This pattern is typical in e-commerce and signals a strong dependency on the holiday season.

**A catalogue dominated by a few high-value products**

The top 3 products by revenue (Leather Jacket, Wireless Headphones, Smart Watch) account for a significant share of total revenue, driven by high unit prices. Sports products and small Beauty accessories generate little revenue despite comparable sales volumes. Electronics is the strongest category on both dimensions — volume and value.

**Homogeneous purchasing behaviour across countries**

Repurchase frequency is similar across all countries (4.2 to 4.3 orders per customer). Revenue gaps are mainly explained by the number of customers and the average order basket — France leads on both metrics. Germany, despite having a similar customer count to Belgium, shows the lowest average basket.

**Mobile slightly dominant but not meaningfully different**

The mobile channel generates more orders and revenue than web, but the basket difference is marginal (+€6). Both channels behave very similarly — there is no radically different purchasing profile between the two.

**A customer base mostly in a development phase**

Only 20% of customers are clearly loyal (Champions + Loyal customers). The majority (46.6%) consists of promising, engaging or average customers — intermediate profiles with conversion potential. Warning signals remain limited: 7.3% at-risk customers and 4% lost customers.

---

### Recommendations

**1. Maximise Q4 performance**

The October–December sales peaks are an opportunity to capitalise on. Launch targeted promotional campaigns from September, anticipate stock on high-value products (Leather Jacket, Smart Watch, Wireless Headphones) and offer Electronics + accessories bundles to increase the average basket.

**2. Reduce dependency on weak months**

January, March and June show structural revenue troughs. Targeted reactivation campaigns aimed at inactive and at-risk customers during these slow periods would help smooth annual revenue and make underperforming months more profitable.

**3. Convert promising and engaging customers**

With 31.9% of the base in these two segments, this is the most immediate growth lever. A loyalty programme with personalised offers based on purchase history and a points system would accelerate their progression towards the loyal segment.

**4. Strengthen the French market and conduct deeper analysis of other markets**

France leads on all indicators (revenue, customer count, average basket). Investing more in acquisition and retention in this market would offer the best return. Conversely, Germany — which shows the lowest average basket — deserves a deeper investigation to understand whether the issue lies in the product mix or in purchasing behaviour.

**5. Increase the value of Sports and small Beauty accessories**

These categories perform well in volume but generate little revenue due to low price points. Two levers are available: revise pricing upward on selected products, or develop cross-selling strategies with higher-value categories (e.g. Smart Watch purchase → recommend Gym Gloves or Protein Shaker).

**6. Launch a win-back programme for lost customers**

The 40 lost customers (4%) represent a non-negligible revenue potential. A reactivation campaign with an exclusive, time-limited offer could reconvert a portion of them at a lower cost than acquiring new customers.

---

## Stack

![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=for-the-badge&logo=dbeaver&logoColor=white)

---

*[Robin B.Rubangura](https://roses29.github.io/My-data-portfolio/)*
