# Analyse E commerce en SQL
Un projet d'analyse de données e-commerce construit de A à Z : génération des données, modélisation, requêtes analytiques et recommendations.

---

## Pourquoi ce projet ?

Je voulais travailler sur un jeu de données e-commerce réaliste sans dépendre d'un dataset existant. J'ai donc choisi de générer mes propres données en SQL — ce qui m'a forcé à réfléchir à la modélisation avant même d'écrire la moindre requête d'analyse.

L'idée était simple : **partir d'un schéma vide et arriver à une segmentation client complète**, en passant par toutes les étapes intermédiaires qu'on rencontre dans un vrai projet data.

---

## Le schéma

Quatre tables, des relations claires, un modèle en étoile compatible Power BI.

```
customers ──► orders ◄── order_details
                │
                ▼
            products
```

| Table | Contenu |
|-------|---------|
| `customers` | 1 000 clients, 5 pays, signup entre 2022 et 2025 |
| `products` | 40 produits, 5 catégories, prix réalistes |
| `orders` | ~15 000 lignes — 1 ligne par produit par commande |
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
-- Distribution naturelle des clients sur 4 ans
date('2022-01-01', '+' || abs(random() % 1461) || ' days')

-- Commande garantie pour chaque client (100%)
-- + commandes supplémentaires selon probabilité décroissante
-- Bloc 2 : 75% des clients
WHERE abs(random() % 4) < 3
-- Bloc 3 : 50% des clients
WHERE abs(random() % 2) = 0
```

📄 [`generate_data.sql`](./sql/generate_data.sql)

---

## Analyses

### 1. Performance temporelle

**Question :** Comment évolue le CA mois par mois ? Y a-t-il des tendances annuelles ?

Je commence toujours par une vue temporelle — c'est le socle de toute analyse e-commerce. Elle permet de détecter les tendances avant de chercher à les expliquer.

```sql
SELECT
    strftime('%Y', order_date) AS annee,
    strftime('%m', order_date) AS mois,
    ROUND(SUM(quantity * unit_price), 2) AS CA,
    COUNT(DISTINCT order_id)             AS nb_commandes
FROM orders
GROUP BY annee, mois
ORDER BY annee, mois;
```

📄 [`analysis_temporal.sql`](./sql/analysis_temporal.sql)

---

### 2. Saisonnalité

**Question :** Certains mois sont-ils structurellement plus forts, toutes années confondues ?

Plutôt que de regarder chaque mois individuellement, je calcule l'écart par rapport à la moyenne globale — ce qui permet de voir les pics réels indépendamment de la croissance année sur année.

```sql
-- Écart en % par rapport à la moyenne globale
ROUND((ca_moyen - moyenne_globale) * 100.0 / moyenne_globale, 1) AS ecart_pct
```

📄 [`analysis_seasonality.sql`](./sql/analysis_seasonality.sql)

---

### 3. Produits

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

📄 [`analysis_products.sql`](./sql/analysis_products.sql)

---

### 4. Canaux & Paiement

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

📄 [`analysis_channels.sql`](./sql/analysis_channels.sql)

---

### 5. Segmentation RFM

**Question :** Comment classer les clients selon leur comportement d'achat pour cibler les bonnes actions marketing ?

Le RFM est l'aboutissement logique du projet : après avoir compris les tendances globales, je descends au niveau client. Chaque client reçoit un score sur 3 dimensions (Récence, Fréquence, Montant) via `NTILE(5)`, puis est assigné à un segment.

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
-- Score Récence : 5 = le plus récent
6 - NTILE(5) OVER (ORDER BY recence ASC) AS score_r
-- Score Fréquence et Montant : 5 = le meilleur
NTILE(5) OVER (ORDER BY frequence ASC)  AS score_f
NTILE(5) OVER (ORDER BY montant ASC)    AS score_m
```

> **Pourquoi `6 - NTILE` pour la récence ?** `NTILE` attribue 1 aux plus petites valeurs. Or une petite récence (peu de jours depuis le dernier achat) signifie un bon client — il faut donc inverser.

📄 [`analysis_rfm.sql`](./sql/analysis_rfm.sql)

---

## Structure du projet

```
ecommerce-sql-analysis/
├── README.md
├── sql/
│   ├── generate_data.sql
│   ├── analysis_temporal.sql
│   ├── analysis_seasonality.sql
│   ├── analysis_products.sql
│   ├── analysis_channels.sql
│   └── analysis_rfm.sql
└── powerbi/
    └── dashboard.pbix
```

---

## Stack

![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=for-the-badge&logo=dbeaver&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)

---

*[Votre Nom](https://github.com/votre-profil)*
