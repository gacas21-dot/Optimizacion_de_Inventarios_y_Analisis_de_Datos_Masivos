# Instacart E-commerce Data Analysis - AI Agent Instructions

## Project Overview
This is a **Jupyter Notebook-based data analysis project** (Spanish-language) analyzing Instacart grocery delivery platform data. The project follows a structured three-phase workflow: **Data Description → Data Preprocessing → Exploratory Data Analysis (EDA)**.

**Key Focus:** Understanding customer purchase patterns, product popularity, reorder behavior, and cart composition in an e-commerce grocery delivery context.

---

## Dataset Architecture

### Five Core DataFrames (all use `;` semicolon separator)
1. **instacart_orders.csv** → `df_ins_order`
   - Master orders table: order metadata, customer IDs, timing (day/hour), days since prior order
   - Primary key: `order_id`

2. **order_products.csv** → `df_orders_pr`
   - Order line items: links orders to products, cart sequence, reorder flag
   - Composite key: `order_id` + `product_id`
   - Critical flag: `add_to_cart_order` (sequential position, 1-based)

3. **products.csv** → `df_products`
   - Product master data: names, aisle/department categorization
   - Primary key: `product_id`

4. **aisles.csv** → `df_aisles`
   - Aisle/category lookup (e.g., aisle_id=100 = missing data indicator)

5. **departments.csv** → `df_departments`
   - Department lookup for hierarchical product categorization

### Critical Data Quality Issues (Intentionally Introduced)
- **Missing values:** `product_name` (1,258 unknowns in aisle_id=100), `days_since_prior_order` (first orders only), `add_to_cart_order` (836 nulls filled with 999 as sentinel)
- **Duplicates:** 15 duplicate rows in orders; 1,361 duplicate product names (product_name.str.upper() check required)
- **CSV Format Issue:** All CSV files use `;` separator (non-standard UTF-8, not comma)

---

## Developer Workflows

### Loading Data (Required Pattern)
```python
import pandas as pd
import os
from matplotlib import pyplot as plt

# Use os.path.join() and always specify sep=';'
path_orders = os.path.join('Datasets', 'instacart_orders.csv')
df_ins_order = pd.read_csv(path_orders, sep=';')
```

### Data Validation Checklist
1. **Always call `.info()` and `.head()`** after loading each DataFrame
2. **Check for duplicates:** `df.duplicated().sum()` then `df.drop_duplicates()`
3. **Validate column-specific uniqueness:** `df['column_id'].duplicated().sum()`
4. **Fill missing values with context:**
   - `days_since_prior_order` → fill with `0` (indicates first order)
   - `product_name` → fill with `'Unknown'`
   - `add_to_cart_order` → fill with `999` then `.astype(int)` (sentinel for missing)
5. **Check numeric ranges:** `order_hour_of_day` [0-23], `order_dow` [0-6]

### EDA Pattern (Three-Phase Structure)
**Phase A (Easy):** Basic distributions, single-column aggregations, simple bar/line charts
**Phase B (Intermediate):** Multi-column comparisons, groupby operations, top-N product analysis  
**Phase C (Hard):** User-level calculations, proportions, behavioral segmentation

---

## Project-Specific Conventions

### Naming Conventions
- DataFrame variables use `df_` prefix with descriptive names (`df_ins_order`, `df_orders_pr`)
- Filtered DataFrames for analysis: `df_filtered`, `df_user_orders`, `df_merged`
- Result DataFrames: `resultado`, `resultado_final`, `resultado_c2`

### Aggregation Patterns
```python
# Standard groupby for counting
pedidos_por_dia = df_ins_order.groupby('order_dow')['order_id'].count()
usuarios_por_hora = df_ins_order.groupby('order_hour_of_day')['user_id'].nunique()

# Reorder analysis (filter where reordered == 1 first)
productos_reordenados = df_orders_pr[df_orders_pr['reordered'] == 1]
frecuencia_reorden = productos_reordenados['product_id'].value_counts()

# Multi-column groupby for user behavior
stats_by_user = df_merged.groupby('user_id')['reordered'].agg(['count', 'sum'])
```

### Visualization Standards
- Use `matplotlib.pyplot` (never plotly in this project)
- Include descriptive titles, labeled axes, and legend when comparing groups
- Add statistical annotations (mean/median lines) for distributions
- Always call `plt.show()` after plots
- Figure size: `figsize=(12, 6)` for wider plots

### Merging Dataframes
```python
# Pattern: prep series → DataFrame → merge with product/order tables
top_20_df = top_20_series.reset_index()
top_20_df.columns = ['product_id', 'count_column']
resultado = top_20_df.merge(df_products, on='product_id', how='left')
resultado[['product_id', 'product_name', 'metric']].head(20)
```

---

## Critical Integration Points

### Cross-DataFrame Analysis
- **Orders ↔ Products:** Merge via `order_id` → `df_orders_pr` → `product_id` → `df_products`
- **Products ↔ Categories:** Join via `aisle_id`/`department_id` to lookup category names
- **User Behavior:** Group orders by `user_id`, then analyze reorder rates within each user

### Reorder Metric (Frequently Analyzed)
- `reordered` column [0,1]: 0 = first time purchase, 1 = repeat purchase
- **Product-level:** Count `reordered==1` / total → product reorder rate
- **User-level:** Sum `reordered` / count of products → user repeat purchase proportion
- Interpretation: High reorder products = essentials/organics (bananas, milk); Low = experimental

---

## Project Insights (For Context)

### Key Findings Already Documented
- **Peak shopping hours:** 10 AM - 4 PM (especially 11 AM, 1 PM, 2 PM)
- **Peak days:** Sundays and Mondays > other days (Thursdays/Fridays lowest)
- **Top products:** Organic Banana (Banana ID: 24852), Organic Strawberries (high reorder rates)
- **Cart patterns:** Most customers purchase 8-10 items per order (median 8, mean 10.1)
- **Customer behavior:** ~55,000 customers make 1 order; ~36,000 make 2 orders (long tail of loyalty)
- **Organic preference:** 16/20 top reordered products are organic (market signal)

---

## Common Tasks for Agents

| Task | Pattern | Key Notes |
|------|---------|-----------|
| Find top-N products | `.value_counts().head(N)` → reset_index() → merge df_products | Always get product names for context |
| Compare day patterns | Filter by `order_dow`, create side-by-side bar plots | Use width offsets for readable overlays |
| Calculate user metrics | Groupby `user_id`, aggregate by `reordered` column | Divide sum/count for proportions |
| Identify missing data patterns | Filter by aisle_id/department_id → check `.isna().sum()` | Understand **why** data is missing (e.g., aisle_id=100 = unmapped products) |
| Validate range constraints | Check `.min()/.max()` for temporal columns, assert boolean columns are [0,1] | Prevents invalid downstream analysis |

---

## Don't Assume
- ❌ CSV files use comma separators (always `sep=';'`)
- ❌ Product names are unique (1,361 duplicates exist in uppercase comparison)
- ❌ All `add_to_cart_order` values present (836 nulls; understand they're sentinel 999)
- ❌ `days_since_prior_order` defines first orders (correctly NaN → fill with 0)
- ❌ Data is clean (intentional duplicates/missing values per project design)

---

## Language Note
Project is written in **Spanish**. Markdown headings, variable descriptions, and some comments use Spanish terminology (e.g., "Paso 1," "pedidos," "productos"). Maintain consistency when adding analysis or documentation.
