# 📊 Segment E-commerce Customers via RFM Analysis Using Python 
## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  
### 🧩 Project Context

A **global e-commerce company** faced challenges in **targeting customer segments effectively**. The Marketing team wanted to reward loyal customers and identify those with high potential. However, customer segmentation was still done **manually in Excel**, which was **time-consuming** and **insufficient** for **large-scale strategy**.

To solve this, I developed a RFM (Recency – Frequency – Monetary) analysis using Python. The solution empowers the Marketing team to launch **data-driven, personalized campaigns, prioritize high-value users, and improve retention.**

### Objective:
### 📖 What is this project about? What Business Question will it solve?

This project uses Python to segment customers using the RFM (Recency – Frequency – Monetary) model and answers:

- 🔍 Who are the company’s most **valuable and loyal customers**?
- 🚨 Which customers are likely to **churn or need re-engagement**?
- 📣 How can we **optimize marketing campaigns** and **resource allocation** based on customer value?

🎯 The goal is to generate **actionable customer segments** that guide strategic **decisions in marketing, loyalty programs, and resource allocation**.

### 👤 Who is this project for?  

💼 **Marketing & CRM Teams:** improving campaign precision and retention strategies

📊 **Data Analysts:** exploring behavior-driven segmentation

🧭 **Business Decision Makers:** planning resource allocation based on customer value

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: UK-based online retail dataset (publicly available)
- Time Period: 01/12/2010 – 09/12/2011
- Size: ~541,909 rows × 8 columns
- Format: .csv


### 📊 Data Structure & Relationships  

#### 1️⃣ Table Used:  
The dataset contains **1 transactional table** that includes both customer and product-level details.

#### 2️⃣ Table Schema  

Table : Transaction Table

| Column Name   | Data Type | Description                                          |
|---------------|-----------|------------------------------------------------------|
| `InvoiceNo`   | Nominal   | Unique invoice number (starts with 'C' for cancelled orders) |
| `StockCode`   | Nominal   | Unique identifier for each product                  |
| `Description` | Nominal   | Name/description of the product                     |
| `Quantity`    | Numeric   | Quantity of items per transaction                   |
| `InvoiceDate` | DateTime  | Date and time of the transaction                    |
| `UnitPrice`   | Numeric   | Price per item (in GBP)                             |
| `CustomerID`  | Nominal   | Unique customer identifier                          |
| `Country`     | Nominal   | Customer’s country of residence                     |


---

## ⚒️ Main Process

## 1️⃣ Exploratory Data Analysis (EDA)
### 🔷 Check Data Types
- Performed **initial data inspection**:
  +   Verified datatypes and structure of dataset
  +   Identified **invalid entries and anomalies in data** (e.g., negative quantities, zero unit price)

```
# Check datatypes
print(df.info())
# Check numerical data
print(df.describe())
df.head()

#StockCode, Description, Country, InvoiceNo -- Datatype: String
df = df.astype({'StockCode':'string', 'Description':'string', 'Country':'string', 'InvoiceNo': 'string'})
#InvoiceDate -- datetime
df['InvoiceDate'] = df['InvoiceDate'].astype('datetime64[ns]')
#UnitPrice -- float
df['UnitPrice'] = df['UnitPrice'].str.replace(',','.').astype('float')
#CustomerID -- int64
df['CustomerID'] = df['CustomerID'].astype('Int64')

```
![Check Datatypes Screenshot](https://drive.google.com/uc?id=1gE8RJjvgSughUgDIg1Sl88QxTXGS6nJI)
    
### 🔷 Handle Missing & Duplicate Data

## 2️⃣ RFM Analysis Preprocess
### 🔷 RFM Calculation
### 🔷 RFM Score

## 3️⃣ Customer Segmentation & Visualization
### 🔷 Segment Customers
### 🔷 Visual Insights


- First, explain codes' purpose - what they do

- Then how your query/ code & Insert screenshots of your result

- Finally, explain your observations/ findings from the results  ts findings
  
 _Describe trends, key metrics, and patterns._  

---

## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📌 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3
