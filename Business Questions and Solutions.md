# SQL Challenge Week 4 _ Data Bank

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

