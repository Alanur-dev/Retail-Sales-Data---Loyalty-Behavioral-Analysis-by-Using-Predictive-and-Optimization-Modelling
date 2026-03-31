# Retail-Sales-Data---Loyalty-Behavioral-Analysis-by-Using-Predictive-and-Optimization-Modelling
Advanced analytics project using event study, logistic regression, chi-square testing, and linear programming on electronic sales data.


## Project Overview
This project analyses customer behaviour in an electronic retail sales dataset to understand how loyalty membership changes are associated with customer purchasing patterns and order cancellations. The analysis combines exploratory data analysis, event-study methodology, predictive modelling, post-cancellation segmentation, and a simple optimisation approach to generate business-focused recommendations.

The main aim is to answer three questions:

1. How does customer behaviour change when a customer signs up for or cancels loyalty membership?
2. Can loyalty cancellations be predicted using pre-event customer behaviour?
3. Where do cancellations concentrate, and how can limited intervention budget be allocated more effectively?

The dataset contains **20,000 observations** and includes variables such as customer demographics, loyalty membership, product type, order status, payment method, total price, purchase date, shipping type, and add-on purchases. 

---

## Dataset
The project uses the file:

- `Individual Assignment Data File Electronic_sales.xlsx`

### Key variables
- `Customer ID`
- `Age`
- `Gender`
- `Loyalty Member`
- `Product Type`
- `SKU`
- `Rating`
- `Order Status`
- `Payment Method`
- `Total Price`
- `Unit Price`
- `Quantity`
- `Purchase Date`
- `Shipping Type`
- `Add-ons Purchased`
- `Add-on Total` 

---

## Project Workflow

### 1. Data Cleaning and Preparation
The dataset was first cleaned and standardised before moving into analysis.

Main preprocessing steps included:
- Filling missing `Gender` values with `"Unknown"`
- Filling missing `Add-ons Purchased` values with `"None"`
- Standardising categorical values such as payment methods, loyalty labels, shipping types, and add-on names
- Creating helper flags such as:
  - `is_cancelled`
  - `has_addon`
- Converting `Purchase Date` into datetime format
- Sorting transactions by `Customer ID` and `Purchase Date` to identify loyalty transitions over time. 

---

## Objective 1: Loyalty Transition Event Study

### Goal
Instead of looking only at whether a customer is a loyalty member, this project focuses on **changes in loyalty status over time**:
- `signup`: customer moves from `no` to `yes`
- `cancel`: customer moves from `yes` to `no` 

### Method
An **event-study approach** was applied using a **60-day window** before and after the transition date. For each customer, the following metrics were compared:

- transaction count
- average order value (AOV)
- add-on attachment rate
- order cancellation rate 

### Event counts
The notebook identified:
- **1,344 sign-up events**
- **1,377 cancellation events** 

### Summary of findings
After removing missing pre/post comparisons, the event-study summary showed:

#### Sign-up customers
- Customers analysed: **1,328**
- Median change in transactions: **+1**
- Mean change in transactions: **+0.67**
- Median change in AOV: **+224.18**
- Mean change in AOV: **+170.28**
- Mean change in add-on rate: **-0.0269**
- Mean change in cancellation rate: **-0.0114**

#### Cancellation customers
- Customers analysed: **1,353**
- Median change in transactions: **+1**
- Mean change in transactions: **+0.68**
- Median change in AOV: **+103.75**
- Mean change in AOV: **+106.15**
- Mean change in add-on rate: **+0.0511**
- Mean change in cancellation rate: **+0.0321** 

### Interpretation
The event-study results suggest that loyalty sign-up is associated with stronger post-event spending improvement than loyalty cancellation. Sign-up customers show a larger increase in average order value and a slight decline in cancellation rate, while cancellation customers show a smaller AOV increase and a rise in cancellation rate. 

---

## Objective 2: Predicting Loyalty Cancellation

### Goal
A logistic regression model was built to test whether loyalty cancellations could be predicted using customer behaviour in the **pre-event window**.

### Initial model
The first model used a **60-day pre-window** and generated features such as:
- `pre_tx`
- `pre_aov`
- `pre_addon_rate`
- `pre_cancel_rate`
- `Age`
- `Gender` 

This reduced the usable sample considerably:
- Cancel cohort: **598**
- Control cohort: **468** 

### Expanded model
To improve coverage, the window was extended to **120 days**, increasing the sample to:
- Cancel cohort: **971**
- Control cohort: **797** 

### Model performance
#### Base logistic regression
- ROC-AUC: **0.4766**
- Accuracy: **0.47** 

#### Expanded logistic regression with additional categorical features
- ROC-AUC: **0.4793**
- Accuracy: **0.47** 

### Interpretation
The cancellation prediction model performed poorly and stayed close to random classification. This suggests that the available behavioural variables are **not sufficient to explain loyalty cancellation** in a reliable way. The notebook explicitly concludes that additional variables would be needed to improve prediction quality. 

Possible future features:
- recency of purchase
- return behaviour
- delivery issues
- customer support interactions
- loyalty benefit usage
- campaign exposure

---

## Objective 3: Where Do Cancellations Concentrate?

This objective was divided into two parts.

### Part A: Post-cancellation segmentation
A **60-day post-cancellation window** was created for customers who cancelled loyalty membership. For each customer, the dominant post-event values were captured for:
- payment method
- shipping type
- product type 

After merging this with the change in cancellation rate and removing missing deltas:
- observations before drop: **1,377**
- observations after drop: **613** 

### Segmentation findings

#### By shipping type
Highest mean increase in cancellation rate:
- `expedited`: **0.1409**
- `same day`: **0.0456**

Lower or negative change:
- `standard`: **0.0062**
- `overnight`: **-0.0217**
- `express`: **-0.0233** 

#### By payment method
Highest mean increase in cancellation rate:
- `credit card`: **0.0708**
- `paypal`: **0.0608**

Lower or negative change:
- `bank transfer`: **0.0163**
- `debit card`: **-0.0093**
- `cash`: **-0.0786** 

#### By product type
Highest mean increase in cancellation rate:
- `headphones`: **0.1546**
- `tablet`: **0.0706**
- `laptop`: **0.0441`

Lower or negative change:
- `smartphone`: **-0.0131**
- `smartwatch`: **-0.0413** 

### Interpretation
The segmentation analysis suggests that cancellation behaviour is not evenly distributed across customer segments after loyalty cancellation. Faster shipping types and some product categories appear to be more exposed to increased cancellation rates. 

---

## Objective 4: Order Cancellation Modelling

A second logistic regression model was developed at the **order level** to explain general order cancellations using operational variables.

### Features considered
- shipping type
- payment method
- product type
- loyalty member
- has_addon
- unit price
- quantity

`Total Price` was removed because it is mechanically related to `Unit Price × Quantity`, which would create high correlation. 

### Model performance
- ROC-AUC: **0.5077**
- Accuracy: **0.49** 

### Interpretation
This model also performed very weakly. The notebook concludes that the dataset is likely synthetic and that relationships between variables and cancellations are weak, making it difficult to explain cancellation behaviour through traditional predictive modelling. 

---

## Objective 5: Linear Programming for Budget Allocation

Since predictive performance was weak, the project moved to an optimisation-based approach.

### Idea
A segment-level table was created by grouping orders by:
- `Shipping Type`
- `Payment Method`

For each segment, the project estimated:
- number of orders
- cancellation rate
- expected cancellations

Then assumed intervention **cost** and **effectiveness** by shipping type:
- expedited: cost = 5, effect = 0.06
- same day: cost = 4, effect = 0.05
- overnight: cost = 3, effect = 0.04
- express: cost = 2, effect = 0.03
- standard: cost = 1, effect = 0.02 

### Example high-volume segments
Top expected cancellations included:
- `standard + credit card`: **664**
- `standard + paypal`: **645**
- `same day + bank transfer`: **375**
- `standard + bank transfer`: **370** 

### Most efficient segments by preventable cancellations per unit cost
Top segments included:
- `standard + credit card`: **13.28**
- `standard + paypal`: **12.90**
- `standard + bank transfer`: **7.40**
- `standard + cash`: **5.04** 

### Interpretation
The optimisation step reframed the problem from “predict who will cancel” to “where can a limited budget reduce the most cancellations?” This is useful when predictive models are weak but operational prioritisation is still needed. 

---

## Tools and Libraries
This project was developed in Python using common analytics and machine learning libraries, including:

- pandas
- numpy
- matplotlib
- scikit-learn
- openpyxl
- scipy (for hypothesis testing where relevant) 

---

## Key Business Takeaways
- Loyalty transitions are more informative than a static loyalty label.
- Customers who sign up for loyalty show stronger post-event value uplift than customers who cancel.
- The available dataset does not support strong prediction of loyalty cancellation.
- Post-cancellation behaviour varies by shipping type, payment method, and product category.
- When prediction is weak, optimisation can still help prioritise interventions under budget constraints. 

---

## Limitations
- The dataset appears to behave like a synthetic dataset, which may weaken real-world relationships.
- Important behavioural and operational drivers of cancellation are missing.
- Cost and effect assumptions in the optimisation stage are illustrative rather than observed from real intervention data. 

---

## Future Improvements
- Add customer service, return, delivery, and complaint data
- Include loyalty benefit usage and campaign response data
- Test tree-based models such as XGBoost or Random Forest
- Perform time-based validation instead of only random train/test splitting
- Replace assumed intervention effects with real business estimates

---

## Repository Structure
```bash
├── README.md
├── Individual Assignment Data File Electronic_sales.xlsx
├── Electronic Sales EDA.ipynb
└── outputs/
    ├── charts/
    └── tables/
