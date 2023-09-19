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


