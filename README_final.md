# DataCo Supply Chain Analysis

**Question:** Where is this business losing money to operational failure, and are those failures concentrated or systemic?

**Answer:** They are systemic. Delivery performance, discounting, and customer mix are effectively uniform across every market, department, and customer tier. The one place the business does differentiate, its expedited shipping tiers, is the one place it fails, promising delivery windows the network never meets.

**Tools:** SQL (DuckDB / JupySQL), pandas, Power BI
**Dataset:** DataCo Smart Supply Chain, 180,519 order line items, 53 fields

---

## Findings

### 1. Shipping tiers are a pricing fiction. The network ships everything in about 4 days.

| Shipping Mode | Scheduled days | Actual days | Gap |
|---|---|---|---|
| Second Class | 2.0 | 4.0 | +2.0 |
| First Class | 1.0 | 2.0 | +1.0 |
| Same Day | 0.0 | 0.5 | +0.5 |
| Standard Class | 4.0 | 4.0 | -0.004 |

Scheduled days are fixed integers. Every order within a tier receives an identical promise regardless of distance, market, or product. Actual transit time converges near 4 days for both Second Class and Standard Class.

Standard Class is the only mode whose promise matches operational reality, and it is effectively exact. Every tier sold as faster than Standard misses its window, and Second Class misses by the widest margin at 2 full days, despite being priced below First Class.

**Implication:** The tiers do not reflect differentiated fulfillment capability. They reflect differentiated promises against a network that ships at one speed. Correcting the scheduled windows on Second Class and First Class would resolve most late deliveries without any operational change, though it would also expose that expedited shipping delivers little real speed advantage. That is a commercial question rather than a logistics one.

### 2. Late delivery is systemic, not regional

Roughly 55% of orders arrive late, and that rate is nearly identical across all markets. LATAM has the lowest late rate on the highest order volume. Africa and USCA have the highest rates on much smaller volumes, but the spread between best and worst market is narrow.

**Implication:** This is not a regional logistics problem, and regional intervention would not move the number. The cause sits upstream of geography, in the scheduling policy described above.

### 3. Cancellations forgo about 4% of potential profit

| Metric | Value |
|---|---|
| Foregone profit from cancelled orders | $160,482 |
| Total profit | $3,966,903 |
| Foregone share | 4.05% |

This is framed as foregone rather than lost profit. Cancelled orders never generated revenue, so the figure represents profit that did not materialize rather than a write-off.

**Implication:** This is the only failure mode in the dataset with a concrete dollar figure attached, which makes it the most defensible target for intervention.

### 4. Discounting is undifferentiated

Discount rates sit near 10.2% for essentially every customer, and roughly 94% of orders across all departments carry a discount. Plotting total discount received against total spend produces a flat line. The largest buyers receive no better terms than the smallest.

**Implication:** Discounting functions as a blanket price reduction rather than a lever. There is no volume incentive, no loyalty tier, and no negotiated pricing.

### 5. Revenue concentrates in departments but not in customers

Four departments (7, 4, 5, and 3) account for roughly $30M of about $33M in revenue, close to 92% of the total. The remaining seven departments are marginal, three of them under $100K.

Customers show the opposite pattern. The top 30 customers each contribute around 2% of revenue, with no single account materially larger than another.

**Implication:** There is no key account risk on the customer side. Departmental concentration means operational attention should follow revenue rather than department count.

### 6. Margins are narrow and flat

Profit margin by department ranges from about 8% to 13%. The low end is occupied by departments with negligible revenue, one of which turns only $11K, so the volume weighted picture is tighter than the raw range suggests. No department is structurally unprofitable and none is a standout.

---

## Method

Analysis was performed in SQL against the raw dataset using DuckDB inside a Jupyter notebook, with Power BI for visualization.

**Grain.** The dataset is one row per order line item, not per order. Order level counts therefore require `DISTINCT Order Id`, since row counts overstate order volume. This distinction governs every aggregation in the notebook.

**Profit ratio.** Aggregate margin is computed as total profit divided by total revenue. Averaging the per order profit ratio was rejected because that average is biased toward small orders, since every order contributes equally regardless of size.

**Cancelled orders.** Measured as foregone profit rather than foregone revenue. Revenue on a cancelled order was never earned, so treating it as a loss would overstate the impact.

**Late deliveries.** No dollar loss is quantifiable here because payment has already been made. Exposure is instead expressed as the share of orders and profit flowing through late deliveries.

---

## Limitations

**Product Status is 0 for all 180,519 rows.** No stockout or inventory availability analysis is possible with this data. The question was asked and the data could not answer it.

**Maximum order quantity is 5.** There are no bulk orders, so bulk versus retail pricing comparisons are not meaningful.

**Department Id is a product division, not a physical location.** Findings by department describe product lines rather than warehouses or stores.

**Order Zipcode (155,679 nulls) and Product Description (180,519 nulls)** were unusable.

**No time series analysis.** Seasonality and trend in late delivery were not examined.

---

## Repository contents

| File | Contents |
|---|---|
| `DataCo_analysis.ipynb` | SQL analysis, DuckDB queries with inline reasoning |
| `DataCo-VIZZ.pbix` | Power BI dashboard, three pages |
| `README.md` | This document |

---

## Possible extensions

Modeling late delivery as a classification problem would test whether lateness is predictable at order time or effectively random given the fixed scheduling policy. Time series decomposition would establish whether the 55% late rate is stable or trending. Both would build on the scheduling finding rather than restate it.
