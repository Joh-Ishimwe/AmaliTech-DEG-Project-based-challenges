# Veridi Logistics — Delivery Performance Audit

> **AmaliTech Data Engineering Graduate Programme · DEG Challenge**
> Candidate: Josiane Ishimwe

---

## A. Executive Summary

An audit of 99,441 orders from the Olist Brazilian E-Commerce Dataset reveals that while **91.9% of deliveries arrive on time**, the 8.1% that are late create outsized reputational damage — super-late orders (5+ days past the promised date) score just **1.78 / 5** on average, a 58% collapse compared to on-time orders (4.29 / 5). The problem is not nationwide: it is heavily concentrated in **Northeast Brazil**, where states like Alagoas (AL, 23.9%), Maranhão (MA, 22.1%), and Piauí (PI, 20.4%) record late rates more than 8 percentage points above the national average of 8.1%, consistent with their distance from São Paulo's main distribution hub. At the product level, categories such as **audio** and **fashion underwear** combine high late rates with near-1-star reviews, making them the highest-priority targets for operational intervention. The CEO's gut feeling is confirmed by data: Veridi is over-promising and under-delivering, and the fix is geographic and category-specific, not a blanket nationwide programme.

---

## B. Project Links

| Deliverable | Link |
|---|---|
| Notebook (Google Colab) | *https://drive.google.com/file/d/117iLH9OEBEc2l7FfIun__Erpbhqvdi2i/view?usp=sharing* |
| Dashboard | *https://public.tableau.com/app/profile/josiane.ishimwe/viz/Veridi_Logistics_Delivery_Audit_Josiane/Sheet2#2* |
| Presentation (PDF/PPT) | *https://docs.google.com/presentation/d/19vidzTTFZYkByt1gQAmSruEPnyMOoljUS8Nphew9rk4/edit?usp=sharing* |


---

## C. Technical Explanation

### Data Cleaning

The raw Olist dataset is a relational dump across six CSV files. The cleaning pipeline addressed three main issues:

**1. Fan-out prevention (items table)**
The `olist_order_items_dataset.csv` has one row per product per order. An order with three items has three rows — joining this directly to orders would triple the row count. Fix: `drop_duplicates(subset="order_id", keep="first")` before merging, reducing 112,650 item rows to one per order.

**2. Duplicate reviews**
A small number of orders had two review entries (customers who edited their score). Fix: sort descending on `review_answer_timestamp` and `drop_duplicates(subset="order_id", keep="first")`, keeping only the most recent review. This removed 610 duplicate entries.

**3. Datetime conversion and status filtering**
All five date columns were stored as plain text. They were converted via `pd.to_datetime(..., errors="coerce")`, which safely coerces any malformed values to `NaT` instead of raising an error. Only orders with `order_status == "delivered"` were kept for delay analysis — cancelled, unavailable, and in-transit orders have no actual delivery date and were excluded rather than imputed.

**4. Delay classification**
`days_difference = order_estimated_delivery_date − order_delivered_customer_date`

| Value | Status |
|---|---|
| ≥ 0 | On Time (arrived on or before the promised date) |
| −5 to −1 | Late (1–5 days past the promise) |
| < −5 | Super Late (more than 5 days past the promise) |

**5. Join integrity assertion**
After chaining five left-joins, the final master DataFrame is asserted to have the same row count as the source `orders` table. This assertion acts as an automated data quality gate — if any join accidentally fans out, the notebook fails loudly rather than silently producing inflated counts.

---

### Candidate's Choice — Product Category Risk Matrix

**Feature:** A dual-encoded bar chart showing the top 15 product categories by late delivery rate, where bar *height* encodes the late rate (%) and bar *colour* encodes the average review score (red < 2.5 · amber 2.5–3.5 · green > 3.5).

**Business justification:** Not all lateness hurts equally. A category with a 15% late rate where customers still leave 4-star reviews (perhaps because the product is a non-urgent gift) is very different from a category with the same late rate but 2-star reviews. The dual-encoding surfaces this nuance in a single view — a red + tall bar means high lateness *and* low satisfaction, making it the unambiguous priority for operational action.

The analysis found that **audio** and **fashion underwear/beach** sit in the "worst quadrant" (high late rate + lowest review scores), while categories like **fashion bags** maintain moderate scores despite similar late rates, suggesting customers are more forgiving for fashion accessories. This nuance means a one-size-fits-all SLA policy would misallocate resources — the operations team should target red bars first.

**Translation note (Bonus Story):** All product category names were translated from Portuguese to English using the `product_category_name_translation.csv` file included in the dataset, joined on `product_category_name`. Any unmatched categories were labelled `"Unknown"` and excluded from the category chart to keep it clean.

---

*Dataset: [Olist Brazilian E-Commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) · License: CC BY-NC-SA 4.0*
