# Analyse E commerce en SQL
Un projet d'analyse de données e-commerce construit de A à Z : génération des données, modélisation, requêtes analytiques et recommendations.


## Pourquoi ce projet ?

Je voulais travailler sur un jeu de données e-commerce réaliste sans dépendre d'un dataset existant. J'ai donc choisi de générer mes propres données en SQL — ce qui m'a forcé à réfléchir à la modélisation avant même d'écrire la moindre requête d'analyse. Les objectifs concrets du projet sont les suivants: 


**• mesurer la performance commerciale**

**• identifier les pays et clients à forte valeur**

**• détecter les produits clés et les points faibles**

**•	aider la prise de décision marketing**

## Le schéma

Quatre tables, des relations claires et un un modèle en étoile

```
customers ──► orders ◄── order_details
                │
                ▼
            products
```

| Table | Contenu |
|-------|---------|
| `customers` | 1 000 clients, 5 pays, signup entre 2022 et 2025 |
| `products` | 40 produits, 5 catégories (Electroniques, Maison, Sport, Beauté et Fashion) , prix réalistes |
| `orders` | ~13. 000 lignes — 1 ligne par produit par commande |
| `order_details` | 6 combinaisons canal (web/mobile) × paiement |

**Choix de modélisation :** `channel` et `payment_method` sont isolés dans `order_details` pour éviter la redondance dans `orders` et faciliter les analyses par canal.

---

## Génération des données

Avant d'analyser, il faut des données cohérentes. Quelques contraintes que je me suis imposées :

- `order_date` toujours ≥ `signup_date` du client
- Toutes les dates bornées entre 2022 et 2025-12-31
- Distribution des clients irrégulière sur les 4 années (pas de quota fixe par année)
- Nombre de commandes variable par client : tous passent au moins 1 commande, certains en passent jusqu'à 5
- 1 à 5 produits par commande
- Prix fixés par produit (réalistes, pas aléatoires)

```sql

-- TABLE CUSTOMERS

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


-- TABLE PRODUCTS — 40 produits uniques (8 par catégorie)

CREATE TABLE products (
    product_id   INTEGER PRIMARY KEY AUTOINCREMENT,
    product_name TEXT,
    category     TEXT,
    unit_price   REAL
);

INSERT INTO products (product_name, category, unit_price) VALUES
-- Electronics : 15€ à 299€
('Wireless Headphones',  'Electronics', ROUND(49  + abs(random() % 150), 2)),
('Bluetooth Speaker',    'Electronics', ROUND(29  + abs(random() % 100), 2)),
('USB-C Hub',            'Electronics', ROUND(15  + abs(random() % 45),  2)),
('Mechanical Keyboard',  'Electronics', ROUND(59  + abs(random() % 120), 2)),
('Webcam HD',            'Electronics', ROUND(39  + abs(random() % 80),  2)),
('Smart Watch',          'Electronics', ROUND(99  + abs(random() % 200), 2)),
('Portable Charger',     'Electronics', ROUND(19  + abs(random() % 50),  2)),
('LED Desk Lamp',        'Electronics', ROUND(25  + abs(random() % 60),  2)),
-- Fashion : 12€ à 250€
('Slim Fit Jeans',       'Fashion',     ROUND(29  + abs(random() % 70),  2)),
('Wool Sweater',         'Fashion',     ROUND(39  + abs(random() % 80),  2)),
('Leather Jacket',       'Fashion',     ROUND(89  + abs(random() % 160), 2)),
('Running Sneakers',     'Fashion',     ROUND(49  + abs(random() % 120), 2)),
('Canvas Tote Bag',      'Fashion',     ROUND(12  + abs(random() % 40),  2)),
('Silk Scarf',           'Fashion',     ROUND(25  + abs(random() % 75),  2)),
('Chino Pants',          'Fashion',     ROUND(29  + abs(random() % 60),  2)),
('Polo Shirt',           'Fashion',     ROUND(19  + abs(random() % 50),  2)),
-- Home : 8€ à 200€
('Bamboo Cutting Board', 'Home',        ROUND(12  + abs(random() % 30),  2)),
('Coffee Maker',         'Home',        ROUND(39  + abs(random() % 120), 2)),
('Throw Pillow',         'Home',        ROUND(12  + abs(random() % 35),  2)),
('Scented Candle',       'Home',        ROUND(8   + abs(random() % 30),  2)),
('Air Purifier',         'Home',        ROUND(59  + abs(random() % 140), 2)),
('Storage Basket',       'Home',        ROUND(10  + abs(random() % 30),  2)),
('Non-stick Pan',        'Home',        ROUND(25  + abs(random() % 70),  2)),
('Wall Clock',           'Home',        ROUND(15  + abs(random() % 50),  2)),
-- Beauty : 5€ à 120€
('Vitamin C Serum',      'Beauty',      ROUND(19  + abs(random() % 60),  2)),
('Moisturizing Cream',   'Beauty',      ROUND(12  + abs(random() % 45),  2)),
('Hair Mask',            'Beauty',      ROUND(8   + abs(random() % 30),  2)),
('Lip Balm Set',         'Beauty',      ROUND(5   + abs(random() % 20),  2)),
('Face Wash Gel',        'Beauty',      ROUND(8   + abs(random() % 25),  2)),
('Eye Contour Cream',    'Beauty',      ROUND(15  + abs(random() % 50),  2)),
('Perfume 50ml',         'Beauty',      ROUND(35  + abs(random() % 85),  2)),
('Nail Polish Kit',      'Beauty',      ROUND(10  + abs(random() % 30),  2)),
-- Sports : 5€ à 100€
('Yoga Mat',             'Sports',      ROUND(19  + abs(random() % 50),  2)),
('Resistance Bands',     'Sports',      ROUND(8   + abs(random() % 25),  2)),
('Water Bottle 750ml',   'Sports',      ROUND(10  + abs(random() % 25),  2)),
('Foam Roller',          'Sports',      ROUND(15  + abs(random() % 35),  2)),
('Jump Rope',            'Sports',      ROUND(5   + abs(random() % 20),  2)),
('Gym Gloves',           'Sports',      ROUND(10  + abs(random() % 25),  2)),
('Protein Shaker',       'Sports',      ROUND(8   + abs(random() % 20),  2)),
('Sports Towel',         'Sports',      ROUND(5   + abs(random() % 20),  2));


-- TABLE ORDER_DETAILS

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


-- TABLE ORDERS

CREATE TABLE orders (
    order_id        INTEGER,
    customer_id     INTEGER,
    order_detail_id INTEGER,
    order_date      TEXT,
    product_id      INTEGER,
    quantity        INTEGER,
    unit_price      REAL,
    FOREIGN KEY (customer_id)    REFERENCES customers(customer_id),
    FOREIGN KEY (order_detail_id) REFERENCES order_details(order_detail_id),
    FOREIGN KEY (product_id)     REFERENCES products(product_id)
);

-- Étape 1 : table temporaire des commandes (1 ligne = 1 commande unique)
CREATE TEMPORARY TABLE temp_orders AS
SELECT
    ROW_NUMBER() OVER () AS order_id,
    c.customer_id,
    1 + abs(random() % 6) AS order_detail_id,
    date(c.signup_date, '+' || (abs(random() % MAX(1, CAST(julianday('2025-12-31') - julianday(c.signup_date) AS INTEGER)))) || ' days') AS order_date
FROM customers c,
     (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3) extra  -- base ~3000 commandes
-- Blocs supplémentaires pour distribution variable
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

-- Étape 2 : éclater chaque commande en 1 à 5 lignes produits
-- nb_produits est tiré aléatoirement entre 1 et 5 pour chaque commande
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
-- Générer entre 1 et 5 lignes par commande via jointure sur une série filtrée
JOIN (SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3
      UNION ALL SELECT 4 UNION ALL SELECT 5) serie
  ON serie.n <= 1 + abs(random() % 5)
-- Tirer un produit aléatoire parmi les 40
JOIN (
    SELECT product_id, unit_price,
           ROW_NUMBER() OVER () AS rn
    FROM products
) p ON p.rn = 1 + abs(random() % 40);

-- Nettoyage
DROP TABLE IF EXISTS temp_orders;
```

Après exécution de la requête ci-dessus, j'ai le modèle suivant: 

![Modèle data](./images/Modèle_data_ecommerce.png)



## Analyses

### 1. Performance temporelle

Je commence  par une vue temporelle qui me permettra d'identifier rapidement les tendances avant de chercher à les expliquer.

**Question :** Comment évolue le Chiffre d'affaires d'année en année? 

```sql
SELECT strftime('%Y',o.order_date) AS Year,
       ROUND(SUM(o.quantity * o.unit_price), 2) AS CA
FROM orders o
GROUP BY Year
ORDER BY Year;
```

**Résultat :** 

|Year|CA|
|----|--|
|2022|65115.0|
|2023|242985.0|
|2024|570158.0|
|2025|1257904.0|

On observe une forte et constante croissance du CA depuis 2022. Le CA atteint son pic en 2025 avec 1,257,904, de CA, ce qui représente une croissance de 220% par rapport à 2024. 

**Question :** Quel est le nombre de nouveaux clients chaque année ?
```sql
SELECT strftime('%Y',signup_date) as Annee, COUNT(customer_id) as Nouveaux_clients
FROM customers 
GROUP BY Annee; 

```
**Résultat :** 

|Annee|Nouveaux_clients|
|-----|----------------|
|2022|247|
|2023|250|
|2024|267|
|2025|236|

L'entreprise gagne plus de 230 clients chaque année, mais le chiffre en 2025 est en baisse par rapport à 2024 et c'est le plus bas toutes années confondues. 

**Question :** Comment le nombre de commandes a-t-il évolué? 

```sql
SELECT
     strftime('%Y',order_date)    AS annee,
    COUNT(DISTINCT order_id)       AS nombre_de_commandes
FROM orders
GROUP BY annee
ORDER BY annee;
```
**Résultat :** 

|annee|nombre_de_commandes|
|-----|-------------------|
|2022|133|
|2023|505|
|2024|1110|
|2025|2502|

Le nombre de commandes a constamment et fortement évolué depuis 2022 et a atteint un pic de 2502 commandes en 2025. 

**Question :** Comment le panier moyen par commande a-t-il évolué?
```sql
SELECT
    strftime('%Y',o.order_date)                                    AS year,
    ROUND(SUM(o.quantity * o.unit_price) / COUNT(DISTINCT o.order_id), 2) AS panier_moyen
FROM orders o
GROUP BY year
ORDER BY year;
```
**Résultat :** 
|Annee|panier_moyen|
|----|------------|
|2022|489.59|
|2023|481.16|
|2024|513.66|
|2025|502.76|

Il n'y a pas réellement de tendances mais le panier moyen est passé au dessus des 500 euros à partir de 2024, atteignant son pic la même année (environ 514 euros).  

### 2. Saisonnalité

**Question :** Certains mois génèrent-ils systématiquement plus de CA ? (Toutes années confondues)

Poue chaque mois, je calcule le CA moyen et l'écart en pourcentage par rapport à la moyenne globale , ce qui permet de voir les pics réels indépendamment de la croissance année sur année.

```sql
SELECT
    mois,
    ROUND(ca_moyen, 2)                          AS ca_moyen_du_mois,
    ROUND(moyenne_globale, 2)                   AS moyenne_globale,
    ROUND((ca_moyen - moyenne_globale) * 100.0
          / moyenne_globale, 1)                 AS ecart_pct
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
                ord.order_date                  AS order_date,
                SUM(ord.unit_price * ord.quantity) AS ca_mensuel
            FROM orders ord
            GROUP BY strftime('%Y', ord.order_date), strftime('%m', ord.order_date)
        ) ca
        GROUP BY strftime('%m', ca.order_date)
    )
)
ORDER BY ecart_pct DESC;
```
**Résultat :** 

|mois|ca_moyen_du_mois|moyenne_globale|ecart_pct|
|----|----------------|---------------|---------|
|12|77647.0|46036.0|68.7|
|10|64217.75|46036.0|39.5|
|11|59246.75|46036.0|28.7|
|08|46837.0|46036.0|1.7|
|09|44794.0|46036.0|-2.7|
|05|42553.25|46036.0|-7.6|
|07|40155.0|46036.0|-12.8|
|02|37635.67|46036.0|-18.2|
|04|36151.5|46036.0|-21.5|
|01|35930.33|46036.0|-22.0|
|06|34659.5|46036.0|-24.7|
|03|32604.25|46036.0|-29.2|

Les mois **d'octobre, novembre et particulièrement décembre** générent plus de chiffre d'affaires que la moyenne globale, ce sont les mois où il y a des pics de ventes. Les mois **de janvier, juin et mars** sont les mois très faibles en termes de ventes. 


### 3. Produits et Catégories 

**Question :** Quels produits génèrent le plus de CA ? Les plus vendus sont-ils les plus rentables ?

J'analyse les deux dimensions séparément car elles peuvent raconter des histoires opposées : un produit à 9.99€ vendu 500 fois génère moins de CA qu'un produit à 199.99€ vendu 80 fois.

```sql
SELECT p.product_name, p.category,
       SUM(o.quantity)                          AS quantity_sold,
       ROUND(SUM(o.quantity * o.unit_price), 2) AS CA,
       ROUND(SUM(o.quantity * o.unit_price)
             / SUM(o.quantity), 2)              AS prix_moyen
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id
ORDER BY CA DESC;
```
**Résultat :** 

|product_name|category|quantity_sold|CA|prix_moyen|
|------------|--------|-------------|--|----------|
|Leather Jacket|Fashion|937|176156.0|188.0|
|Wireless Headphones|Electronics|1027|171509.0|167.0|
|Smart Watch|Electronics|897|170430.0|190.0|
|Running Sneakers|Fashion|1042|142754.0|137.0|
|Mechanical Keyboard|Electronics|1044|105444.0|101.0|
|Perfume 50ml|Beauty|1033|96069.0|93.0|
|Webcam HD|Electronics|997|91724.0|92.0|
|Coffee Maker|Home|944|90624.0|96.0|
|Wool Sweater|Fashion|1002|87174.0|87.0|
|Air Purifier|Home|954|78228.0|82.0|
|Chino Pants|Fashion|964|74228.0|77.0|
|Slim Fit Jeans|Fashion|834|72558.0|87.0|
|Portable Charger|Electronics|1084|71544.0|66.0|
|Eye Contour Cream|Beauty|1026|64638.0|63.0|
|Vitamin C Serum|Beauty|912|62016.0|68.0|
|Wall Clock|Home|846|46530.0|55.0|
|Bluetooth Speaker|Electronics|1038|43596.0|42.0|
|Silk Scarf|Fashion|934|41096.0|44.0|
|Foam Roller|Sports|948|38868.0|41.0|
|LED Desk Lamp|Electronics|1033|35122.0|34.0|
|Scented Candle|Home|925|33300.0|36.0|
|Storage Basket|Home|934|31756.0|34.0|
|Yoga Mat|Sports|863|30205.0|35.0|
|Non-stick Pan|Home|971|27188.0|28.0|
|Bamboo Cutting Board|Home|991|23784.0|24.0|
|Throw Pillow|Home|951|22824.0|24.0|
|Nail Polish Kit|Beauty|930|21390.0|23.0|
|Polo Shirt|Fashion|912|19152.0|21.0|
|Canvas Tote Bag|Fashion|937|18740.0|20.0|
|Gym Gloves|Sports|930|17670.0|19.0|
|Moisturizing Cream|Beauty|954|17172.0|18.0|
|Jump Rope|Sports|994|16898.0|17.0|
|Water Bottle 750ml|Sports|1002|15030.0|15.0|
|Lip Balm Set|Beauty|843|14331.0|17.0|
|Hair Mask|Beauty|1009|14126.0|14.0|
|USB-C Hub|Electronics|868|13020.0|15.0|
|Face Wash Gel|Beauty|932|12116.0|13.0|
|Protein Shaker|Sports|984|11808.0|12.0|
|Resistance Bands|Sports|1020|8160.0|8.0|
|Sports Towel|Sports|898|7184.0|8.0|

**La jacket en cuivre, les écouteurs sans fil et la montre connectée** sont les 3 produits qui ont généré le plus de CA (+17 000 euros). C'est principalement dû à leur prix qui est plus élevé que les autres même si beaucoup d'écouteurs ont été vendus. **Le chargeur portable, la crème pour les yeux (lignes 13 et 14)** sont des exemples de produits qui génèrent du CA grâce à la quantité vendue. 

**Question :** Quels sont les 10 produits les plus vendus? 

```sql
SELECT p.product_id, p.product_name, p.category,
       SUM(o.quantity) AS quantity_sold
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY quantity_sold DESC
LIMIT 10;
```

**Résultat :** 

|product_id|product_name|category|quantity_sold|
|----------|------------|--------|-------------|
|7|Portable Charger|Electronics|1084|
|4|Mechanical Keyboard|Electronics|1044|
|12|Running Sneakers|Fashion|1042|
|2|Bluetooth Speaker|Electronics|1038|
|8|LED Desk Lamp|Electronics|1033|
|31|Perfume 50ml|Beauty|1033|
|1|Wireless Headphones|Electronics|1027|
|30|Eye Contour Cream|Beauty|1026|
|34|Resistance Bands|Sports|1020|
|27|Hair Mask|Beauty|1009|

5 des top 10 produits les plus vendus sont dans la catégorie **électroniques** et le **chargeur portable** est le produit qui a été le plus vendu au cours des 4 années. 

**Question :** Quels sont les 10 produits les moins vendus? 

```sql
SELECT p.product_id, p.product_name, p.category,
       SUM(o.quantity) AS quantity_sold
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY quantity_sold DESC
LIMIT 10;
```

**Résultat :** 

|product_id|product_name|category|quantity_sold|
|----------|------------|--------|-------------|
|9|Slim Fit Jeans|Fashion|834|
|28|Lip Balm Set|Beauty|843|
|24|Wall Clock|Home|846|
|33|Yoga Mat|Sports|863|
|3|USB-C Hub|Electronics|868|
|6|Smart Watch|Electronics|897|
|40|Sports Towel|Sports|898|
|16|Polo Shirt|Fashion|912|
|25|Vitamin C Serum|Beauty|912|
|20|Scented Candle|Home|925|

**Question :** Quel est le CA et la quantité vendue par catégorie? 

```sql
SELECT p.category, SUM(o.quantity) as Quantity_sold, SUM(o.quantity * o.unit_price) as CA 
FROM products p
INNER JOIN orders o ON p.product_id = o.product_id
GROUP BY p.category
ORDER BY Quantity_sold DESC;    
```

|category|Quantity_sold|CA|
|--------|-------------|--|
|Electronics|7988|702389.0|
|Sports|7639|145823.0|
|Beauty|7639|301858.0|
|Fashion|7562|631858.0|
|Home|7516|354234.0|

La catégories **Electroniques** vend le plus et génèrent de CA mais a des prix supérieurs aux produits des autres vatégories. Les produits **Maison** se vendent le moins mais génèrent plus de CA que les catégories **Beauté et Sports**. 

### 4. Pays

**Question :** Quel est le CA par pays ? Quel est celui qui rapporte le plus? 

```sql
SELECT
  c.country,
  SUM(o.quantity *o.unit_price) AS ca_par_pays
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.country
ORDER BY ca_par_pays DESC;
```
|country|ca_par_pays|
|-------|-----------|
|France|483866.0|
|Spain|452321.0|
|Italy|421936.0|
|Belgium|393177.0|
|Germany|384862.0|

La **France** représente le plus gros marché en termes de CA avec 483,866 euros, suivie de l'Espagne et l'Italie qui ont rapportés plus de 420,000 de CA. 


**Question :** Quel est le nombre de clients par pays?

|country|Nb_clients|
|-------|----------|
|France|216|
|Spain|211|
|Italy|199|
|Germany|188|
|Belgium|186|

Le plus gros marché en termes de clients est en **France (216)**, mais les écarts ne sont pas très significatifs avec des pays commme l'Espagne (211) et l'Italie(199). L'entreprise a moins de clients en Allemagne (188) et en Belgique (186). 

**Question :** Quels sont les comportements d'achats par pays?

Pour répondre à la question, j'analyse la moyenne des commandes par client, le panier moyen par commande et le panier moyen par client: 

```sql
SELECT
    c.country,
    COUNT(DISTINCT o.order_id)                              AS nb_commandes,
    COUNT(DISTINCT c.customer_id)                           AS nb_clients,
    ROUND(COUNT(DISTINCT o.order_id) * 1.0
          / COUNT(DISTINCT c.customer_id), 1)               AS commandes_par_client,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT o.order_id), 2)                  AS panier_moyen_commande,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT c.customer_id), 2)               AS panier_moyen_client
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.country
ORDER BY panier_moyen_client DESC;
```

|country|nb_commandes|nb_clients|commandes_par_client|panier_moyen_commande|panier_moyen_client|
|-------|------------|----------|--------------------|---------------------|-------------------|
|France|926|216|4.3|522.53|2240.12|
|Spain|899|211|4.3|503.14|2143.7|
|Italy|853|199|4.3|494.65|2120.28|
|Belgium|786|186|4.2|500.23|2113.85|
|Germany|786|188|4.2|489.65|2047.14|

Les commandes par client sont similaires entre pays (4,2 ou 4,3), ce qui montre que la comportement de réachat est homogène entre les pays et n'expliquent pas les écarts de CA entre pays. Mais en observant le **panier moyen par commande et le panier moyen par client**, on observent que les clients en France dépensent plus que les clients basés dans d'autres pays. De plus, les marchés en Belgique et en Allemagne sont similaires en termes de commandes et clients, mais les clients en Belgique dépensent un peu plus. 

En général, on observe que la fréquence de réachat est faible mais que les clients dépensent de grosses sommes lorsqu'ils achètent. 

### 5. Canaux & Paiement

**Question :** Les comportements d'achat diffèrent-ils entre web et mobile ?

Je regarde le panier moyen par canal — pas juste le volume — pour voir si les clients mobiles achètent différemment des clients web.

```sql
SELECT
    od.channel,
    COUNT(DISTINCT o.order_id)                     AS nb_commandes,
    ROUND(SUM(o.quantity * o.unit_price), 2)       AS CA,
    ROUND(SUM(o.quantity * o.unit_price)
          / COUNT(DISTINCT o.order_id), 2)         AS panier_moyen
FROM orders o
INNER JOIN order_details od ON o.order_detail_id = od.order_detail_id
GROUP BY od.channel;
```
|channel|nb_commandes|CA|panier_moyen|
|-------|------------|--|------------|
|mobile|2158|1091064.0|505.59|
|web|2092|1045098.0|499.57|

Sur la période 2022-2025, il y a eu plus de commandes mobile (2158) que de commandes web (2092) et les commandes mobile ont rapporté plus de CA. De plus, le panier moyen de commandes est légèrement plus élevé pour les commande mobile que web (+6 euros). 


### 6. Segmentation RFM

**Question :** Comment classer les clients selon leur comportement d'achat pour cibler les bonnes actions marketing ?

Le RFM est l'aboutissement logique du projet : après avoir compris les tendances globales, je descends au niveau client. Chaque client reçoit un score (allant de 1 à 5) sur 3 dimensions (Récence (R), Fréquence (F) , Montant (M), puis est assigné à un segment.

| Segment | Condition | Action |
|---------|-----------|--------|
| Champion | R=5, F=5, M=5 | Programme VIP |
| Client fidèle | F=5 | Récompenser |
| Nouveau client | R=5, F=1 | Encourager 2e achat |
| Client à risque | R≤2, F≥4 | Relancer rapidement |
| Client prometteur | R≥3, F≥3 | Nurturing |
| Client inactif | R≤2, F≤3 | Réactivation |
| Client perdu | R=1, F=1, M=1 | Win-back ou abandon |

```sql
WITH rfm_base AS (
    SELECT
        c.customer_id,
        c.country,
        c.gender,
        c.age,
        CAST(julianday('2025-12-31') - julianday(MAX(o.order_date)) AS INTEGER) AS recence,
        COUNT(DISTINCT o.order_id)                                               AS frequence,
        ROUND(SUM(o.unit_price * o.quantity), 2)                                 AS montant
    FROM customers c
    INNER JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id
),

rfm_scores AS (
    SELECT
        customer_id, country, gender, age,
        recence, frequence, montant,
        6 - NTILE(5) OVER (ORDER BY recence ASC)  AS score_r,
        NTILE(5) OVER (ORDER BY frequence ASC)    AS score_f,
        NTILE(5) OVER (ORDER BY montant ASC)      AS score_m
    FROM rfm_base
),

rfm_final AS (
    SELECT
        customer_id, country, gender, age,
        recence, frequence, montant,
        score_r, score_f, score_m,
        (score_r + score_f + score_m) AS score_rfm,
        CASE
            WHEN score_r = 5 AND score_f = 5 AND score_m = 5 THEN 'Champions'
            WHEN score_f = 5                                  THEN 'Clients fidèles'
            WHEN score_r = 5 AND score_f = 1                  THEN 'Nouveaux clients'
            WHEN score_r <= 2 AND score_f >= 4                THEN 'Clients à risque'
            WHEN score_r = 1 AND score_f = 1 AND score_m = 1  THEN 'Clients perdus'
            WHEN score_r >= 3 AND score_f >= 3                THEN 'Clients prometteurs'
            WHEN score_r <= 2 AND score_f <= 3                THEN 'Clients inactifs'
            ELSE 'Autres'
        END AS segment
    FROM rfm_scores
)

-- Répartition par segment
WITH rfm_base AS (
    SELECT
        c.customer_id,
        CAST(julianday('2025-12-31') - julianday(MAX(o.order_date)) AS INTEGER) AS recence,
        COUNT(DISTINCT o.order_id)                                               AS frequence,
        ROUND(SUM(o.unit_price * o.quantity), 2)                                 AS montant
    FROM customers c
    INNER JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id
),
rfm_scores AS (
    SELECT
        customer_id,
        6 - NTILE(5) OVER (ORDER BY recence ASC)  AS score_r,
        NTILE(5) OVER (ORDER BY frequence ASC)    AS score_f,
        NTILE(5) OVER (ORDER BY montant ASC)      AS score_m
    FROM rfm_base
),
rfm_final AS (
    SELECT
        CASE
            WHEN score_r = 5 AND score_f = 5 AND score_m = 5 THEN 'Champion'
            WHEN score_f = 5                                  THEN 'Clients fidèles'
            WHEN score_r = 5 AND score_f = 1                  THEN 'Nouveaux clients'
            WHEN score_r <= 2 AND score_f >= 4                THEN 'Clients à risque'
            WHEN score_r = 1 AND score_f = 1 AND score_m = 1  THEN 'Clients perdus'
            WHEN score_r >= 3 AND score_f >= 3                THEN 'Clients prometteurs'
            WHEN score_r <= 2 AND score_f <= 3                THEN 'Clients inactifs'
            WHEN score_r >= 4 AND score_f= 2                  THEN 'En cours d engagement'
            ELSE 'Clients lambdas'
        END AS segment
    FROM rfm_scores
)
SELECT segment,
       COUNT(*)                                            AS nb_clients,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct
FROM rfm_final
GROUP BY segment
ORDER BY nb_clients DESC;
```


|segment|nb_clients|pct|
|-------|----------|---|
|Clients prometteurs|249|24.9|
|Clients inactifs|217|21.7|
|Clients fidèles|163|16.3|
|Clients lambdas|124|12.4|
|Clients à risque|73|7.3|
|En cours d engagement|70|7.0|
|Clients perdus|40|4.0|
|Champion|37|3.7|
|Nouveaux clients|27|2.7|

Les **clients fidèles et Champions***   représentent environ 20% de la base, soit le cœur fidèle de l'entreprise — peu nombreux mais à fort potentiel de valeur. À cela s'ajoutent les **clients prometteurs et en cours d'engagement (31.9%)**, qui constituent le vivier de croissance le plus immédiat : récents (score R entre 3 et 5) et avec une fréquence d'achat déjà correcte (score F entre 2 et 5), ils sont les mieux placés pour rejoindre le segment fidèle avec les bonnes actions marketing.

Du côté des signaux d'alerte, les **clients perdus** (40 clients, 4%) et les **nouveaux clients** (27 clients, 2.7%) restent des segments minoritaires — les premiers nécessitent une campagne de réactivation, les seconds un onboarding soigné pour éviter qu'ils ne restent des acheteurs ponctuels.

Les clients **lambdas** (12.4%) avec un profil intermédiaire ambigu — ni assez fidèles, ni assez récents, ni clairement à risque.



## Stack

![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=for-the-badge&logo=dbeaver&logoColor=white)

---

*[Robin B.Rubangura]()*
