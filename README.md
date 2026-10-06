# DataCo Supply Chain Analysis

**Question:** Where is this business losing money to operational failure, and are those failures concentrated or systemic?

**Answer:** They are systemic. Late delivery, discounting, and margins are effectively uniform across every market, department, and customer size. The one place the business does differentiate, its shipping tiers, is where it fails most: the faster tiers promise delivery windows the network does not meet.

**Tools:** SQL (DuckDB / JupySQL), pandas, Power BI
**Dataset:** DataCo Smart Supply Chain, 180,519 order line items (65,752 orders), 53 fields, January 2015 to January 2018

---

## Findings

### 1. The faster shipping tiers promise windows the network does not meet

| Shipping Mode | Promised days | Actual days | Late % | Share of all late orders |
|---|---|---|---|---|
| First Class | 1 | always 2 | 95.3% | 26.6% |
| Second Class | 2 | 2 to 6, avg 4.0 | 76.7% | 27.2% |
| Same Day | 0 | 0 or 1, avg 0.5 | 46.1% | 4.6% |
| Standard Class | 4 | 2 to 6, avg 4.0 | 38.1% | 41.6% |

Every order in a tier gets the same promise, regardless of distance, market, or product. An order is late exactly when its actual days exceed that promise.

First Class promises 1 day and always takes 2, so every First Class order that ships is late. Second Class and Standard Class have the same delivery distribution, spread evenly from 2 to 6 days. The only difference is the promise, so choosing Second Class buys no faster delivery.

**Implication:** Most lateness comes from the promise, not from slow fulfillment. Moving the First Class promise from 1 day to 2, which is what it already delivers, would remove about a quarter of all late orders with no operational change. Standard Class still produces the largest share of late orders, because 40% of its shipments take 5 or 6 days against a 4 day promise.

### 2. Late delivery is systemic, not regional or seasonal

54.8% of orders arrive late. By market the rate ranges from 54.1% (Africa) to 55.3% (Pacific Asia), a spread of about one percentage point. By department the four large departments all sit at 54.7% to 54.8%. By quarter it stays between 53.5% and 56.5% for the full three years.

**Implication:** Regional or product level intervention would not move the number. The cause sits upstream, in the scheduling policy described above.

### 3. Orders that never shipped forgo about 4% of profit

| Order Status | Orders | Foregone profit |
|---|---|---|
| Canceled | 1,367 | $75,346 |
| Suspected fraud | 1,488 | $85,137 |
| **Total** | **2,855** | **$160,482** |

Against $3,966,903 in total profit, that is about 4%. It is framed as foregone rather than lost profit, because these orders never generated revenue.

**Implication:** This is the only failure in the dataset with a direct dollar figure. Only the cancelled half is a clear recovery target. The suspected fraud half may be the fraud screen working as intended, so the realistic target is closer to $75K.

### 4. Discounting is undifferentiated

94.4% of line items carry a discount, and that share is 94% to 95% in every department. The average discount is about 10% of the pre-discount price. Individual customers vary, but not by how much they spend: every spend quintile averages about 10%, and the correlation between total spend and discount rate is essentially zero (-0.02).

**Implication:** Discounting works as a blanket price reduction, not a lever. There is no volume incentive, no loyalty tier, and no negotiated pricing.

### 5. Revenue concentrates in departments but not in customers

Four departments (Fan Shop, Apparel, Golf, Footwear) account for $30.3M of $33.1M in revenue, 91.6% of the total. The remaining seven are marginal, three of them under $100K.

Customers show the opposite pattern. Revenue is spread across 20,652 customers. The largest single customer is 0.03% of revenue, and the top 30 combined are under 1%.

**Implication:** There is no key account risk. Operational attention should follow the four large departments.

### 6. Margins are narrow and flat

Overall margin is 12.0%. By department it ranges from 7.8% to 13.0%, but the low end is the smallest departments (Book Shop sells $11K in total). The four large departments all sit between 11.4% and 12.3%. No department is structurally unprofitable and none is a standout.

---

## Method

Analysis was performed in SQL against the raw dataset using DuckDB inside a Jupyter notebook, with Power BI for visualization.

**Grain.** The dataset is one row per order line item, not per order. Delivery Status and Shipping Mode repeat on every line of an order, so order level rates are computed on `DISTINCT "Order Id"`. Dollar totals are summed at line level, where profit and revenue are recorded.

**Profit margin.** Computed as total profit divided by total revenue. Averaging the per line profit ratio was rejected because every line would count equally regardless of size, biasing the result toward small orders.

**Discount rate.** Total discount divided by gross sales (the `Sales` column, before discount).

**Cancelled orders.** Measured as foregone profit rather than foregone revenue. Revenue on a cancelled order was never earned, so treating it as a loss would overstate the impact.

**Late deliveries.** The data has no refund, penalty, or churn field, so lateness cannot be converted to dollars. It is expressed as a share of orders instead.

---

## Limitations

**The data is very likely simulated.** Actual shipping days are spread perfectly evenly from 2 to 6 days for Standard and Second Class, in every region. Real logistics data does not look like this. The findings describe the structure of the data, not a real company's operations.

**Product Status is 0 for all 180,519 rows.** No stockout or inventory analysis is possible.

**Maximum order quantity is 5.** There are no bulk orders, so bulk versus retail pricing comparisons are not meaningful.

**Department Id is a product division, not a physical location.** Findings by department describe product lines rather than warehouses or stores.

**Order Zipcode (155,679 nulls) and Product Description (180,519 nulls)** were unusable.

---

## Repository contents

| File | Contents |
|---|---|
| `DataCo_analysis.ipynb` | SQL analysis, DuckDB queries with inline reasoning |
| `DataCo-VIZZ.pbix` | Power BI dashboard, three pages |
| `DataCoSupplyChainDataset.csv` | Raw dataset |
| `README.md` | This document |

---

## Possible extensions

A classification model for late delivery would not add much here. Lateness is fully determined by shipping mode and actual days, and actual days look randomly assigned. A more useful extension would be running the same promise versus delivery check on a real operational dataset.
