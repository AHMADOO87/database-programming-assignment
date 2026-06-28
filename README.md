# database-programming-assignment 



```md
# Database Programming Assignment I: CTEs & SQL Window Functions

**Student Name:**Ahmed mubarak
**Student ID:** 27995/2024  
**Repository Name:** `database-programming-assignment`

---

## 1. Business Problem
This platform is a financial infrastructure system designed for installment-based purchasing. This architecture serves as a robust foundation for handling scalable payment timelines and managing financial risk—core requirements when building modern financial platforms like mysylla. The database tracks user acquisition (including a referral network), monitors purchase volumes, and analyzes installment payment behaviors to identify defaulted payments.

---

## 2. Database Schema & ER Diagram
The database consists of three related tables:
1. **Platform_Users:** Tracks customer details and referral hierarchies.
2. **Purchases:** Records the core transactions.
3. **Installment_Plans:** Tracks the individual payment schedules linked to each purchase.



*(Sample Data from Purchases Table)* <img width="418" height="111" alt="table 1" src="https://github.com/user-attachments/assets/17b56690-c6fa-41ff-a0cc-79e2aa241501" />

---

## 3. CTE Implementations

### Simple CTE
Isolates users who have defaulted on their payments.
```sql
WITH Defaulted_Payments AS (
    SELECT purchase_id, amount_due, due_date
    FROM Installment_Plans
    WHERE status = 'Defaulted'
)
SELECT * FROM Defaulted_Payments;

```
<img width="319" height="165" alt="cte 1" src="https://github.com/user-attachments/assets/7c18066b-50c2-43a4-9937-3dba2f5d31a1" />


### Multiple CTEs

Compares total revenue generated versus total revenue at risk.

```sql
WITH Total_Revenue AS (
    SELECT SUM(total_amount) as total_sales FROM Purchases
),
At_Risk_Revenue AS (
    SELECT SUM(amount_due) as defaulted_amount 
    FROM Installment_Plans WHERE status = 'Defaulted'
)
SELECT total_sales, defaulted_amount 
FROM Total_Revenue, At_Risk_Revenue;

```
<img width="395" height="191" alt="multiple cte" src="https://github.com/user-attachments/assets/78b8b2c5-4624-4c26-939a-dce40f925bb1" />


### Recursive CTE

Maps out the referral network to identify influential networkers.

```sql
WITH ReferralChain (user_id, full_name, referred_by_id, chain_level) AS (
    SELECT user_id, full_name, referred_by_id, 1
    FROM Platform_Users
    WHERE referred_by_id IS NULL
    UNION ALL
    SELECT p.user_id, p.full_name, p.referred_by_id, rc.chain_level + 1
    FROM Platform_Users p
    INNER JOIN ReferralChain rc ON p.referred_by_id = rc.user_id
)
SELECT * FROM ReferralChain ORDER BY chain_level;

```
<img width="490" height="438" alt="Recursive CTE" src="https://github.com/user-attachments/assets/0713ba57-4161-4e03-a2a9-84f8f61c77ac" />


### CTE with Aggregation

Summarizes total purchasing power per user.

```sql
WITH User_Spending AS (
    SELECT user_id, SUM(total_amount) as lifetime_spend, COUNT(purchase_id) as total_orders
    FROM Purchases
    GROUP BY user_id
)
SELECT * FROM User_Spending WHERE lifetime_spend > 1000;

```

<img width="546" height="195" alt="CTE with Aggregation" src="https://github.com/user-attachments/assets/7a35f0e9-3920-4a31-b233-daff0258af28" />


### CTE combined with JOIN

Connects user profiles directly to pending cash flow.

```sql
WITH Pending_Installments AS (
    SELECT purchase_id, amount_due, due_date
    FROM Installment_Plans
    WHERE status = 'Pending'
)
SELECT u.full_name, p.total_amount, pi.amount_due, pi.due_date
FROM Platform_Users u
JOIN Purchases p ON u.user_id = p.user_id
JOIN Pending_Installments pi ON p.purchase_id = pi.purchase_id;

```

<img width="558" height="441" alt="CTE combined with JOIN operations" src="https://github.com/user-attachments/assets/d0e325ea-3ac8-4a2a-9373-935182e06f59" />


---

## 4. Window Function Implementations

### Ranking Functions

```sql
SELECT 
    purchase_id, 
    total_amount,
    ROW_NUMBER() OVER(ORDER BY total_amount DESC) as row_num,
    RANK() OVER(ORDER BY total_amount DESC) as rank_val,
    DENSE_RANK() OVER(ORDER BY total_amount DESC) as dense_rank_val,
    PERCENT_RANK() OVER(ORDER BY total_amount DESC) as pct_rank
FROM Purchases;

```


### Aggregate Window Functions

```sql
SELECT 
    user_id, 
    purchase_date, 
    total_amount,
    SUM(total_amount) OVER(PARTITION BY user_id ORDER BY purchase_date) as running_total,
    AVG(total_amount) OVER(PARTITION BY user_id) as avg_user_spend,
    MAX(total_amount) OVER(PARTITION BY user_id) as max_user_spend,
    MIN(total_amount) OVER(PARTITION BY user_id) as min_user_spend
FROM Purchases;

```
<img width="540" height="417" alt="Aggregate Window Functions" src="https://github.com/user-attachments/assets/02b39db8-1be5-4305-97dc-6a2a66628a35" />



### Navigation Functions

```sql
SELECT 
    user_id, 
    purchase_date, 
    total_amount,
    LAG(total_amount) OVER(PARTITION BY user_id ORDER BY purchase_date) as prev_purchase,
    LEAD(total_amount) OVER(PARTITION BY user_id ORDER BY purchase_date) as next_purchase
FROM Purchases;

```

<img width="521" height="238" alt="Navigation Functions" src="https://github.com/user-attachments/assets/8f64cc3c-f560-4851-bb7d-8711fcba77a1" />


### Distribution Functions

```sql
SELECT 
    purchase_id, 
    total_amount,
    NTILE(4) OVER(ORDER BY total_amount DESC) as quartile,
    CUME_DIST() OVER(ORDER BY total_amount) as cum_dist
FROM Purchases;

```
<img width="378" height="193" alt="Distribution Functions" src="https://github.com/user-attachments/assets/58f3f042-3f77-413b-ab46-785f935f8095" />

---

## 5. Analysis and Findings

* **Descriptive Analysis (What happened?):** The platform processed varying purchase amounts ranging from 450.00 to 2000.00. While some users are reliably paying their installments, there is a recorded default on an 800.00 purchase. The referral network shows a clear hierarchy originating from early adopters.
* **Diagnostic Analysis (Why did it happen?):** By utilizing Window Functions and CTEs, we observe that defaults may correlate with users who make single, large purchases without a history of smaller, successful repayments. The referral tracking indicates that engaged users drive the most high-value acquisitions.
* **Prescriptive Analysis (What actions should be taken?):** The platform should implement a credit-limit system based on the Aggregate Window Function data; new users must successfully clear smaller installments before being approved for large 2000.00 purchases. Furthermore, the marketing team should automate rewards for users at Level 1 of the Recursive CTE referral chain to boost organic acquisition.

---

## 6. Statement

I certify that this repository represents my own original work, adhering to the academic integrity guidelines of UNILAK.

```

```
