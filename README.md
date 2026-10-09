# Customer Segmentation-Driven Churn Prediction for a Retail Bank

A two-stage analytics project built in **IBM SPSS Modeler**. It first groups bank customers into natural segments (clustering), then predicts which customers are likely to leave (classification), and finally turns both results into a retention strategy for each group.

This was my final project for BBA Business Analytics.

---

## The problem

Keeping an existing customer costs far less than winning a new one, yet banks often notice churn only after the customer has already gone. The goal here is an early-warning view: predict who is likely to leave, and understand *why*, so retention effort can be targeted instead of generic.

## Objectives

1. Segment customers with unsupervised clustering (K-Means) to see the natural groups in the customer base.
2. Predict churn with two classification models (CHAID and Logistic Regression) and compare them.
3. Translate the findings into a specific action plan for each segment.

## Dataset

Bank Customer Churn dataset (publicly available on Kaggle): **10,000 customer records, 12 fields, no missing values** (confirmed with a Data Audit node). Overall churn rate is about 20.4%.

| Field | Role |
|---|---|
| `customer_id` | Identifier, excluded from modeling |
| `credit_score`, `age`, `tenure`, `balance`, `estimated_salary` | Continuous inputs |
| `country`, `gender`, `products_number` | Categorical inputs |
| `credit_card`, `active_member` | Flag inputs (0/1) |
| `churn` | **Target** (0 = stayed, 1 = churned) |

A note on zeros: a `0` in `balance`, `tenure`, `credit_card`, `active_member` or `churn` is a real value, not missing data. For example, many customers genuinely hold a zero balance.

## Methodology

The full stream is saved in this repo. The flow is:

1. **Import and audit**: Excel source node, then a Data Audit node to check data quality.
2. **Type**: set field roles, with `churn` as the target and `customer_id` as none.
3. **Derive**: create a `Balance_Salary_ratio` field (balance divided by salary), followed by a second Type node so the new field is read correctly.
4. **K-Means clustering**: segment customers on balance, salary, age, products and credit score. Churn is deliberately left out, since clustering is unsupervised.
5. **Partition**: 70% training (6,970 records) and 30% testing (3,030 records).
6. **Classification**: train CHAID (decision tree) and Logistic Regression on the same partition.
7. **Analysis**: compare accuracy, coincidence matrices and AUC/Gini for both models.

## Results

### Stage 1: Customer segments

| Segment | Share | Profile |
|---|---|---|
| **Single-Product High-Balance Customers** | 51.3% | High average balance (about 98,886), only one product |
| **Affluent Multi-Product Customers** | 24.5% | Highest salary (about 151,107), two products |
| **Budget-Conscious Multi-Product Customers** | 24.2% | Lower balance (about 49,425) and lower salary (about 50,918), two products |

The largest segment holds a lot of money but uses only one product, which makes it the biggest cross-sell opportunity in the data.

### Stage 2: What drives churn (CHAID tree)

The very first split in the tree is on `products_number`:

| Products held | Churn rate |
|---|---|
| 2 products | **7.77%** |
| 3 to 4 products | **85.77%** |

Customers with three or four products leave at roughly eleven times the rate of customers with two. A likely explanation is over-selling: customers pushed into products they did not need disengage and leave.

Within the lower-risk two-product group, age is the next driver, and churn rises steadily with it:

| Age band | Churn rate |
|---|---|
| 35 or under | 2.9% |
| 36 to 42 | 6.6% |
| 43 to 45 | 11.3% |
| Over 45 | 22.5% |

For customers over 45, `active_member` is the next split, with inactive members showing the highest churn. Balance and gender appear in deeper splits. All splits were statistically significant.

### Model comparison

| Model | Training accuracy | Testing accuracy |
|---|---|---|
| **CHAID** | **85.41%** | **85.87%** |
| Logistic Regression | 82.87% | 82.67% |

Logistic Regression reached a testing AUC of 0.815 (Gini 0.631). CHAID is the final model. It is about three points more accurate, and a decision tree is much easier to explain to a non-technical audience. For both models the testing accuracy is slightly higher than the training accuracy, so there is no sign of overfitting.

## Business recommendations

| Segment or group | Recommended action |
|---|---|
| Single-Product High-Balance (51.3%) | Offer one well-matched second product. Do not push three or four at once. |
| Affluent Multi-Product (24.5%) | Maintain the relationship, with a premium service tier. |
| Budget-Conscious Multi-Product (24.2%) | Focus on fee transparency and loyalty rewards rather than new products. |
| Customers over 45 who are inactive | Proactive re-engagement calls and reactivation of dormant accounts. |
| Customers holding 3 to 4 products | Review these accounts immediately and simplify the product mix. |

## Repository contents

```
.
├── README.md
├── data/
│   └── Bank_Customer_Churn.xlsx        # dataset used in the stream
├── stream/
│   └── churn_stream.str                # SPSS Modeler stream
├── presentation/
│   └── Bank_Churn_SPSS_Project.pptx    # final presentation (with speaker notes)
└── screenshots/                        # stream canvas, cluster output, CHAID tree, model evaluation
```

## How to run it

1. Open **IBM SPSS Modeler** and load `stream/churn_stream.str` (File, then Open Stream).
2. Double-click the Excel source node and point it to `data/Bank_Customer_Churn.xlsx` on your machine.
3. Run the stream. The K-Means branch builds the segments, and the Partition branch builds and evaluates the CHAID and Logistic Regression models.

## Limitations and next steps

- The dataset has no transaction-level or complaint data, which would likely strengthen the predictions.
- The over-selling explanation is a hypothesis drawn from the pattern in the data. It would need to be tested, for example by checking whether a one-product-at-a-time cross-sell lowers churn for the largest segment over the following quarter.
- The clusters overlap to some degree (the silhouette measure was in the lower range), which is normal for customer behaviour data but worth keeping in mind when reading the segments.

## Tools

IBM SPSS Modeler (Excel source, Data Audit, Type, Derive, K-Means, Partition, CHAID, Logistic Regression and Analysis nodes), Microsoft Excel.
