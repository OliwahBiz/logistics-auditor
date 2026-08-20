# The "Last Mile" Logistics Auditor
**Client:** Veridi Logistics (Global E-Commerce Aggregator)
**Deliverable:** Public Dashboard, Python Notebook & Insight Presentation
**Author:** Olivier BIZIMUNGU
**Dataset:** Olist Brazilian E-Commerce Dataset (Kaggle)

---

## A. The Executive Summary

Our delivery performance audit of 95,824 verified orders reveals that 92.0% of
deliveries arrived on or before the estimated date, while 8.0% (7,661 orders)
experienced fulfillment delays. Delivery punctuality is directly and severely
correlated with customer satisfaction: on-time deliveries maintain an average
review score of 4.29 out of 5.0 stars, whereas late deliveries drop to 3.46
stars, and Super Late orders (more than 5 days overdue) collapse to just 1.78
stars — a 59% satisfaction loss. Regional analysis uncovers critical geographic
disparities, with remote Northeastern states such as Alagoas (23.35% late) and
Maranhão (19.24% late) experiencing failure rates nearly four times higher than
central distribution hubs like São Paulo (5.70% late) and Minas Gerais (5.40%
late). To prevent continued negative review spikes, Veridi Logistics must
dynamically adjust estimated delivery windows for remote routes, clear Monday
and Friday warehouse backlogs, and introduce dedicated freight carriers for
high-delay product categories such as electronics and office furniture.

---

## B. Project Links

- **Google Colab Notebook:** [View Notebook](https://colab.research.google.com/drive/1nxwwA1pIyJU6piq1hKnz-41tP45xFlit?usp=sharing)
- **Public Dashboard:** [View Looker Studio Dashboard](REPLACE_WITH_YOUR_LOOKER_STUDIO_LINK)
- **Insight Presentation:** [View Google Slides Presentation](REPLACE_WITH_YOUR_GOOGLE_SLIDES_LINK)
- **Video Walkthrough (Optional):** Not submitted

---

## C. Technical Explanation

### 1. Data Cleaning & Schema Construction

**Ingestion:** Three CSV files were loaded using strictly relative paths:
`olist_orders_dataset.csv`, `olist_order_reviews_dataset.csv`, and
`olist_customers_dataset.csv`. No absolute or machine-specific file paths were
used at any point in the notebook.

**Filtering Non-Delivered Orders:** Orders with `order_status` other than
`delivered` (such as `canceled` or `unavailable`) were excluded. Any remaining
delivered orders with a missing `order_delivered_customer_date` were removed
using `.dropna(subset=['order_delivered_customer_date'])`. This left a clean
base of delivered orders with valid timestamps.

**Date Parsing:** All three date columns (`order_purchase_timestamp`,
`order_delivered_customer_date`, and `order_estimated_delivery_date`) were
converted from raw string format to Python `datetime64` objects using
`pd.to_datetime()`.

**Delay Calculation:** A new column `delay_days` was computed as: 
delay_days = (order_estimated_delivery_date - order_delivered_customer_date)
converted to total seconds / 86400



A positive value means the package arrived before the promised date (on time).
A negative value means it arrived after the promised date (late).

**Delivery Classification:**
- `On Time`: delay_days >= 0
- `Late`: delay_days between 0 and -5
- `Super Late`: delay_days < -5 (more than 5 days overdue)

**Deduplication Before Joining:** The reviews dataset contained multiple review
entries per order in some cases. To prevent 1-to-many row multiplication during
the merge, reviews were sorted by `review_creation_date` and deduplicated on
`order_id`, keeping only the most recent review per order.

**Master Dataset Construction:** The cleaned delivered orders were inner-joined
with the deduplicated reviews on `order_id`, then inner-joined with the
customers table on `customer_id`. The resulting master dataset contained
exactly 95,824 rows across 13 columns with zero row duplication.

---

### 2. Candidate's Choice Addition: Purchase Day Bottleneck Audit

**Feature Added:** A new analysis was added to audit the percentage of late
deliveries and average customer review scores broken down by the day of the
week on which each order was placed. The `purchase_day` column was extracted
from `order_purchase_timestamp` using `.dt.day_name()`.

**Results:**

| Day of Week | % Late Deliveries | Avg Review Score |
|-------------|-------------------|------------------|
| Monday      | 8.86%             | 4.17             |
| Tuesday     | 8.40%             | 4.17             |
| Wednesday   | 7.68%             | 4.18             |
| Thursday    | 7.45%             | 4.15             |
| Friday      | 8.33%             | 4.11             |
| Saturday    | 7.45%             | 4.14             |
| Sunday      | 7.46%             | 4.16             |

**Business Justification:** This analysis pinpoints specific operational
dispatch bottlenecks that are not visible in the standard delivery audit.
Orders placed on Mondays experience the highest delay rate (8.86%) because
weekend order accumulation floods warehouse queues on Monday morning, creating
processing backlogs. Orders placed on Fridays also show elevated delays (8.33%)
due to end-of-week carrier handoff cutoffs. Midweek orders (Thursday and
Wednesday) consistently achieve the lowest late rates. This finding enables
Veridi Logistics to implement targeted staffing adjustments on Mondays and
Fridays to directly reduce fulfillment failures on the highest-risk days.

---

### 3. Bonus Story: Product Category English Translation

**Implementation:** Three additional files were loaded and merged:
`olist_order_items_dataset.csv`, `olist_products_dataset.csv`, and
`product_category_name_translation.csv`. Portuguese category names (e.g.,
`cama_mesa_banho`) were mapped to English (e.g., `bed_bath_table`) via a left
merge on `product_category_name`. Unmapped categories were filled with `other`.
The items table was deduplicated on `order_id` before merging with the master
dataset to prevent row inflation from multi-item orders.

**Findings (Categories with 500+ orders):**

| Product Category               | Total Orders | % Late | Avg Score |
|--------------------------------|--------------|--------|-----------|
| electronics                    | 2,488        | 9.85%  | 4.13      |
| baby                           | 2,741        | 9.16%  | 4.12      |
| office_furniture               | 1,236        | 9.06%  | 3.65      |
| construction_tools_construction| 723          | 8.99%  | 4.13      |
| musical_instruments            | 601          | 8.99%  | 4.25      |
| health_beauty                  | 8,562        | 8.84%  | 4.24      |
| bed_bath_table                 | 9,072        | 8.69%  | 4.01      |
| auto                           | 3,774        | 8.56%  | 4.15      |
| telephony                      | 4,052        | 8.49%  | 4.06      |

**Business Insight:** Bulky and fragile product categories such as
`office_furniture` and `electronics` suffer the highest delay rates and lowest
customer satisfaction. `office_furniture` recorded the lowest category review
score of 3.65 stars — indicating that heavy freight logistics corridors require
dedicated carrier contracts to reduce fulfillment failures.

---

## D. Full Results Summary

| Metric                          | Value                    | Business Impact                            |
|---------------------------------|--------------------------|--------------------------------------------|
| Total Valid Delivered Orders    | 95,824                   | Clean master dataset, zero row duplication |
| On-Time Deliveries              | 88,163 (92.0%)           | Baseline satisfaction: 4.29                |
| Late Deliveries (1-5 Days)      | 3,568 (3.7%)             | Satisfaction drops to 3.46  (-19%)         |
| Super Late Deliveries (>5 Days) | 4,093 (4.3%)             | Satisfaction collapses to 1.78 (-59%)      |
| Worst State: Alagoas (AL)       | 23.35% late              | Remote corridor — 4x higher than hubs      |
| Worst State: Maranhão (MA)      | 19.24% late              | Long-haul Northeastern transit failure     |
| Best State: Rondônia (RO)       | 3.50% late               | Strong regional hub performance            |
| Best Hub: São Paulo (SP)        | 5.70% late               | Central distribution advantage             |
| Peak Delay Day                  | Monday (8.86% late)      | Weekend warehouse backlog                  |
| Worst Product Category          | Electronics (9.85% late) | Heavy/fragile freight friction             |
| Lowest Satisfaction Category    | Office Furniture (3.65)  | Bulky item delivery strain                 |

---

## E. Strategic Recommendations

1. **Dynamic Estimated Delivery Dates:** Add a 3 to 5 day buffer for all
   orders shipping to Alagoas, Maranhão, Piauí, Ceará, and Sergipe to prevent
   over-promising and eliminate the review score collapse caused by Super Late
   deliveries.

2. **Monday and Friday Dispatch Staffing:** Increase warehouse staffing on
   Mondays and Fridays to clear backlogs. A 1.4 percentage point reduction in
   Monday delay rates alone would affect thousands of orders annually.

3. **Dedicated Freight Carriers for Bulky Categories:** Contract specialized
   heavy-freight carriers for electronics, office furniture, and baby products
   to reduce the 9%+ late rates in these high-value, high-risk segments.

---

## F. Repository Contents

| File                      | Description                                       |
|---------------------------|---------------------------------------------------|
| `logistics_auditor.ipynb` | Complete Python notebook with all code and charts |
| `logistics_auditor.pdf`   | PDF export of the notebook with all visual outputs|
| `README.md`               | This documentation file                           |

**Note:** Raw dataset CSV files are NOT included in this repository.
The Olist dataset is publicly available at:
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
