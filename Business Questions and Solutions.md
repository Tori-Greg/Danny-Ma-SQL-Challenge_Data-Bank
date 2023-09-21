# SQL Challenge Week 4 _ Data Bank

# Table of Contents:
- [A. Customer Nodes Exploration](#A.-Customer-Nodes-Exploration)
- [B. Customer Transactions](#B.-Customer-Transactions)

# A. Customer Nodes Exploration 
#### 1. How many unique nodes are there on the Data Bank system?

**Query:**

    Select 
    	Count(distinct Node_id) as Unique_nodes 
    from data_bank.Customer_Nodes;

**Output:**

| unique_nodes |
| ------------ |
| 5            |

---

#### 2. What is the number of nodes per region?

**Query:**

    SELECT COUNT(Node_id) AS No_of_nodes, Region_name
    FROM data_bank.Customer_Nodes
    INNER JOIN data_bank.Regions
    ON Customer_Nodes.Region_id = Regions.Region_id
    GROUP BY Region_name
    ORDER BY 1 DESC;

**Output:**

| no_of_nodes | region_name |
| ----------- | ----------- |
| 770         | Australia   |
| 735         | America     |
| 714         | Africa      |
| 665         | Asia        |
| 616         | Europe      |

---

#### 3. How many customers are allocated to each region?

**Query:**

    SELECT COUNT(DISTINCT Customer_id) AS No_of_Customers, Region_name
    FROM data_bank.Customer_Nodes
    INNER JOIN data_bank.Regions
    ON Customer_Nodes.Region_id = Regions.Region_id
    GROUP BY Region_name
    ORDER BY 1 DESC;

**Output:**

| no_of_customers | region_name |
| --------------- | ----------- |
| 110             | Australia   |
| 105             | America     |
| 102             | Africa      |
| 95              | Asia        |
| 88              | Europe      |

---

#### 4. How many days on average are customers reallocated to a different node?

**Query:**

    SELECT AVG(DATEDIFF(DAY, Start_date, End_date)) AS Avgdays
    FROM DataBank..Customer_Nodes
    WHERE End_date != '9999-12-31';

**Output:**
| Avgdays |
| --------------- |
| 14      |

#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

**Query:**

    WITH CTE AS (
        SELECT CN.Region_id, Region_name, (DATEDIFF(Day, End_date, Start_date)) AS days_reallocation
        FROM data_bank.Customer_Nodes AS CN
        JOIN data_bank.Regions AS R ON CN.Region_id = R.Region_id
        WHERE End_date != '9999-12-31'
    )

    SELECT
        DISTINCT Region_id,
        Region_name,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY days_reallocation) OVER (PARTITION BY Region_name) AS median_days,
        PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY days_reallocation) OVER (PARTITION BY Region_name) AS percentile_80,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY days_reallocation) OVER (PARTITION BY Region_name) AS percentile_95
    FROM CTE
    ORDER BY Region_id;

**Output:**

| Region_id | region_name | median_days | percentile_80 | percentile_95 |
| --------- | ----------- | ----------- | ------------- | ------------- |
| 1         | Australia   |  -15        |  -6           |  -1           
| 2         | America     |  -15        |  -6           |  -1
| 3         | Africa      |  -15        |  -6           |  -1.75
| 4         | Asia        |  -15        |  -6           |  -1
| 5         | Europe      |  -15        |  -6           |  -1


# B. Customer Transactions

#### 1. What is the unique count and total amount for each transaction type?

**Query:**

    SELECT
        DISTINCT txn_type,
        COUNT(txn_type) OVER (PARTITION BY txn_type ORDER BY txn_type) AS unique_count,
        SUM(txn_amount) OVER (PARTITION BY txn_type ORDER BY txn_type) AS total_amount
    FROM
        data_bank.Customer_Transactions;

**Output:**

| txn_type   | unique_count | total_amount |
| ---------- | ------------ | ------------ |
| purchase   | 1617         | 806537       |
| withdrawal | 1580         | 793003       |
| deposit    | 2671         | 1359168      |

---

#### 2. What is the average total historical deposit counts and amounts for all customers?

**Query:**

    WITH CTE AS (
        SELECT
            txn_type,
            txn_amount,
            Customer_id,
            COUNT(*) OVER (PARTITION BY Customer_id) AS item_count
        FROM
            data_bank.Customer_Transactions
        WHERE
            txn_type = 'deposit'
        GROUP BY
            txn_type, txn_amount, customer_id
    )
    
    SELECT
        txn_type,
        CAST(AVG(item_count) AS INTEGER) AS avg_count,
        CAST(AVG(txn_amount) AS INTEGER) AS avg_amount
    FROM
        CTE
    GROUP BY
        txn_type;

**Output:**

| txn_type | avg_count | avg_amount |
| -------- | --------- | ---------- |
| deposit  | 7         | 509        |

---

#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

**Query:**

    WITH Monthly_transactions_count AS (
        SELECT 
            customer_id,
            EXTRACT(MONTH FROM txn_date) AS Month_no,
            TO_CHAR(txn_date, 'Month') AS Month_name,
            SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS Count_of_deposit,
            SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS Count_of_purchase,
            SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS Count_of_withdrawal
        FROM data_bank.Customer_Transactions
        GROUP BY customer_id, EXTRACT(MONTH FROM txn_date), TO_CHAR(txn_date, 'Month')
    )
    
    SELECT
        Month_no,
        Month_name,
        COUNT(DISTINCT customer_id) AS customer_count
    FROM Monthly_transactions_count
    WHERE Count_of_deposit > 1 
      AND (Count_of_purchase = 1 OR Count_of_withdrawal = 1)
    GROUP BY Month_no, Month_name
    ORDER BY Month_no;

**Output:**

| month_no | month_name | customer_count |
| -------- | ---------- | -------------- |
| 1        | January    | 115            |
| 2        | February   | 108            |
| 3        | March      | 113            |
| 4        | April      | 50             |

---

#### 4. What is the closing balance for each customer at the end of the month?

**Query:**

    WITH Monthend_closingbalance AS (
        SELECT
            Customer_id,
            EXTRACT(MONTH FROM txn_date) AS Month_no,
            TO_CHAR(txn_date, 'Month') AS Month_name,
            SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS Total_deposit,
            SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS Total_purchases,
            SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS Total_withdrawal
        FROM data_bank.Customer_Transactions
        GROUP BY
            Customer_id,
            EXTRACT(MONTH FROM txn_date),
            TO_CHAR(txn_date, 'Month')
    )
    
    SELECT
        Customer_id,
        Month_no,
        Month_name,
        SUM(Total_deposit) - SUM(Total_purchases) - SUM(Total_withdrawal) +
            LAG(SUM(Total_deposit) - SUM(Total_purchases) - SUM(Total_withdrawal), 1, 0)
            OVER (PARTITION BY Customer_id ORDER BY Month_no) AS closing_balance
    FROM Monthend_closingbalance
    GROUP BY
        Customer_id,
        Month_no,
        Month_name
    ORDER BY Customer_id, Month_no;

**Output:**

| customer_id | month_no | month_name | closing_balance |
| ----------- | -------- | ---------- | --------------- |
| 1           | 1        | January    | 312             |
| 1           | 3        | March      | -640            |
| 2           | 1        | January    | 549             |
| 2           | 3        | March      | 610             |
| 3           | 1        | January    | 144             |
| 3           | 2        | February   | -821            |
| 3           | 3        | March      | -1366           |
| 3           | 4        | April      | 92              |
| 4           | 1        | January    | 848             |
| 4           | 3        | March      | 655             |
| 5           | 1        | January    | 954             |
| 5           | 3        | March      | -1923           |
| 5           | 4        | April      | -3367           |
| 6           | 1        | January    | 733             |
| 6           | 2        | February   | -52             |
| 6           | 3        | March      | -393            |
| 7           | 1        | January    | 964             |
| 7           | 2        | February   | 3173            |
| 7           | 3        | March      | 1569            |
| 7           | 4        | April      | -550            |


__Kindly be aware that the output is limited to the first 20 rows because it is extensive__

---

#### 5. What is the percentage of customers who increase their closing balance by more than 5%?

**Query:**

    WITH Monthend_breakdown AS (
        SELECT
            Customer_id,
            EXTRACT(MONTH FROM txn_date) AS Month_no,
            TO_CHAR(txn_date, 'Month') AS Month_name,
            SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS Total_deposit,
            SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS Total_purchases,
            SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS Total_withdrawal
        FROM data_bank.Customer_Transactions
        GROUP BY
            Customer_id, 
            EXTRACT(MONTH FROM txn_date),
            TO_CHAR(txn_date, 'Month')
    ),
    monthend_closingbalance AS (
        SELECT
            Customer_id,
            Month_no,
            Month_name,
            SUM(Total_deposit) - SUM(Total_purchases) - SUM(Total_withdrawal) +
                LAG(SUM(Total_deposit) - SUM(Total_purchases) - SUM(Total_withdrawal), 1, 0)
                OVER (PARTITION BY Customer_id ORDER BY Month_no) AS closing_balance,
            LAG(SUM(Total_deposit) - SUM(Total_purchases) - SUM(Total_withdrawal), 1, 0)
                OVER (PARTITION BY Customer_id ORDER BY Month_no) AS pm_balance
        FROM Monthend_breakdown
        GROUP BY
            Customer_id,
            Month_no,
            Month_name
    ),
    Perc_increase AS (
        SELECT
            customer_id,
            Month_name,
            closing_balance,
            100 * (closing_balance - pm_balance) / NULLIF(pm_balance, 0) AS percentage_increase
        FROM monthend_closingbalance
    )
    SELECT
        100 * COUNT(DISTINCT customer_id) / CAST((SELECT COUNT(DISTINCT customer_id) FROM data_bank.Customer_Transactions) AS FLOAT) AS Customers_percentage
    FROM Perc_increase
    WHERE percentage_increase > 5;

**Output:**

| customers_percentage |
| -------------------- |
| 75.6                 |

---


