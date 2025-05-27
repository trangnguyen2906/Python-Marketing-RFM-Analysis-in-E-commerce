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

### 🔷 Check Data Values
- The `Quantity` column contains **negative values**, which mostly correspond to **canceled orders** (identified by `InvoiceNo` starting with "C").
  
👉 These rows were removed to ensure accurate analysis.
- The `UnitPrice` column also has **negative values**, which are likely due to data entry errors or refund transactions.
  
👉 These entries were also **excluded** from further processing.

```
# Check negative values in Quantity column
df[df['Quantity'] < 0]
  ## Almost negative value in Quantity col has InvoiceNo start with 'C' -> Check
check_cancel = df['InvoiceNo'].str.startswith('C')
negative_quan_cancel = df[(df['Quantity'] < 0) & (check_cancel == True)]
  ### other negative quantity
negative_quan_nocancel = df[(df['Quantity'] < 0) & (check_cancel == False)]

#Check negative values in UnitPrice col
df[df['UnitPrice'] < 0]

#Drop rows that cancel and has negative values in Quantity column
df = df[~((df['Quantity'] < 0) & (check_cancel == True))]
#Drop negative values in UnitPrice col
df = df[df['UnitPrice'] > 0]

df.describe()

```

![Data Values Screenshot](https://drive.google.com/uc?id=1Mpr6TEZuKCyI02VNSF1YiYa-SiGKzDoi)

### 🔷 Handle Missing & Duplicate Data
👉 **Detect missing data**

- Missing values were found **only** in the **`CustomerID` column.**
- These missing `CustomerIDs` are **not related to canceled transactions.**
- The majority of these null entries come from customers in the **United Kingdom.**

```
check_null = pd.DataFrame(df.isnull().sum())
check_null['%missing'] = check_null[0] / len(df) * 100
check_null.columns = ['count', '%missing']
check_null # Missing many values in CustomerID column (~25%)

## Check if missing CustomerIDs are related to cancelled transactions
cancelled = df[df['CustomerID'].isnull() & df['InvoiceNo'].str.startswith('C')]
print(cancelled)

## Check if missing CustomerIDs are linked to specific countries
missing_countries = df[df['CustomerID'].isnull()]['Country'].value_counts()
print("\nCountries with missing CustomerID:\n", missing_countries)

check_missing['MonthYear'] = df['InvoiceDate'].dt.to_period('M')
# Count missing CustomerID per month
missing_by_month = check_missing[check_missing['CustomerID'].isnull()]['MonthYear'].value_counts().sort_index()
print(missing_by_month)

```
![Missing Values Screenshot](https://drive.google.com/uc?id=1nuU1tQSycOyaYMbv3agmEpGy3MgWBl7T)

👉 **Dealing with missing and duplicate data**
- **Missing values:** Only customerID is missing -> Doing RFM model need to group based on CustomerID --> **Drop** 25% missing value in CustomerID col
- **Duplicate data:** **drop** duplicated rows

```
# Missing values
df = df.dropna(subset=['CustomerID'])
# Duplicated rows
df_clean = df.drop_duplicates().copy()
df_clean.isnull().sum()
```

## 2️⃣ RFM Analysis Preprocess
### 🔷 RFM Calculation
- RFM stands for Recency, Frequency, and Monetary:
    + **Recency:** Number of days since the last purchase (calculated from the most recent invoice date)
    + **Frequency:** Total number of purchases by each customer
    + **Monetary:** Total revenue generated by each customer
- RFM values were computed using customer-level transaction aggregation.
- Recency was **reversed** to align with the scoring logic: **More recent = higher score.**

```
df_clean['Date'] = df_clean['InvoiceDate'].dt.normalize()
df_clean['MonthYear'] = df_clean['InvoiceDate'].dt.to_period('M')
df_clean['Cost'] = df_clean['Quantity']*df_clean['UnitPrice']
df_clean

# Define last day in dataset
last_day = df_clean['Date'].max()

# Create RFM dataframe
RFM_df = df_clean.groupby('CustomerID').agg(
    Recency=('Date', lambda x: last_day - x.max()),
    Frequency=('InvoiceNo', 'nunique'),
    Monetary=('Cost', 'sum'),
    Start_day=('Date', 'min')
).reset_index()

RFM_df['Recency'] = RFM_df['Recency'].dt.days.astype('int16')
RFM_df['Recency_reverse'] = -RFM_df['Recency']
RFM_df['Start_Month'] = RFM_df['Start_day'].apply(lambda x: x.replace(day=1))
RFM_df.head()
```
![RFM Calculation](https://drive.google.com/uc?id=14Wl6TJ5mSWpusS4Zqhilckt6lRZ-tFJK)

### 🔷 RFM Score
- Each R, F, and M value was scored from 1 to 5 using quintile-based ranking:
  +   Higher Recency → lower score (less recent)
  +   Higher Frequency & Monetary → higher scores
- Scores were concatenated into a 3-digit RFM score

```
#Calculate RFM score
RFM_df['R_score'] = pd.qcut(RFM_df['Recency'], 5, labels=[5, 4, 3, 2, 1]).astype(str)
RFM_df['F_score'] = pd.qcut(RFM_df['Frequency'].rank(method='first'), 5, labels=[1, 2, 3, 4, 5]).astype(str)
RFM_df['M_score'] = pd.qcut(RFM_df['Monetary'], 5, labels=[1, 2, 3, 4, 5]).astype(str)
RFM_df['RFM_score'] = RFM_df['R_score'] + RFM_df['F_score'] + RFM_df['M_score']
RFM_df

```
![RFM Score](https://drive.google.com/uc?id=1b4zG2HlHBdus8rf-uboPDpEUC7-5YLDV)


## 3️⃣ Customer Segmentation & Visualization
### 🔷 Segment Customers
- Customers were divided into **11 RFM-based segments** (e.g., 🏆 Champions, 🌱 Potential Loyalists, ⚠️ At Risk) to guide **personalized marketing and loyalty-focused strategies.**

```
#Segment Customers
segment = pd.read_csv('/content/drive/MyDrive/UNIGAP/Python/Final Project/segmentation.csv')
segment['RFM Score'] = segment['RFM Score'].str.split(',')
# Tách array của RFM score
segment = segment.explode('RFM Score').reset_index().drop(columns=['index'])
segment

#Mapping with RFM_df
RFM_df['RFM_score'] = RFM_df['RFM_score'].astype(str).str.strip()
segment['RFM Score'] = segment['RFM Score'].astype(str).str.strip()
RFM_df_merge = RFM_df.merge(segment, left_on='RFM_score', right_on='RFM Score', how='left')
RFM_df_merge

```

![Mapping](https://drive.google.com/uc?id=1OSQB1qa64aTvfOwr5sKSip2bzQKWqo5t)

  
### 🔷 Visual Insights

#### 🔹 1. Examine Distribution of RFM Metrics
> 🎯 **Goal:** Understand general behavior across Recency, Frequency, and Monetary metrics to identify broad customer patterns.

```
fig, ax = plt.subplots(figsize=(10, 6))
color=sns.color_palette('Blues')[3]
sns.histplot(RFM_df_merge['Recency'], ax=ax, kde=True, stat='density', color = color , edgecolor='black')  # add binwidth
ax.set(xlabel='Recency', ylabel='Distribution', title='Distribution of Recency')
plt.show()

fig, ax = plt.subplots(figsize=(10, 6))
color=sns.color_palette('Blues')[3]
sns.histplot(RFM_df_merge['Frequency'], ax=ax, kde=True, stat='density',
             binwidth=1, color = color , edgecolor='black')  # add binwidth
ax.set(xlabel='Frequency', ylabel='Distribution', xlim=(0, 20), title='Distribution of Frequency')
plt.show()

fig, ax = plt.subplots(figsize=(10, 6))
color=sns.color_palette('Blues')[3]
sns.histplot(RFM_df_merge['Monetary'], ax=ax, kde=True, stat='density',color = color , edgecolor='black', binwidth = 500)  # add binwidth
ax.set(xlabel='Monetary', ylabel='Distribution', xlim = (-1000,15000), title='Distribution of Monetary')
plt.show()
```

![AOV by Segment](https://drive.google.com/uc?id=1v-cwnyVyOjP-4WtEhvFOHAtKQiOGvaEa)

**🔍 Key Findings:**
- ⏱️ **Recency:** Most customers made purchases within the last **50 days**, showing strong short-term engagement. However, there's a long tail of users who **haven't returned for 6+ months**.
- 🔁 **Frequency:** Over **70%** of customers made **only one - two purchase**, revealing a significant retention issue and low repeat-buy behavior. **(KEY WEAKNESSES)**
- 💰 **Monetary:** Spending is highly skewed. **Most customers** **spend little**, while a small number of high-value outliers generate the majority of revenue.
  
#### 🔹 2. Segment Distribution by Count and Revenue

```
segment_counts = RFM_df_merge['Segment'].value_counts()
total = segment_counts.sum()
labels = [f"{seg}\n({count*100/total:.1f}%)"
          for seg, count in zip(segment_counts.index, segment_counts.values)]

# Create subplots
fig, axes = plt.subplots(1, 2, figsize=(20, 8))  # 1 row, 2 columns

# Left: Treemap
squarify.plot(sizes=segment_counts.values,
              label=labels,
              color=sns.color_palette('Paired'),
              pad=True,
              alpha=0.8,
              ax=axes[0])
axes[0].set_title('Treemap of Customer Segments')
axes[0].axis('off')

#Right: Bar Chart
bar = sns.countplot(data=RFM_df_merge,
                    x='Segment',
                    order=segment_counts.index,
                    palette='Paired',
                    ax=axes[1])
axes[1].set_title('Number of Customers in Each Segment')
axes[1].set_xlabel('Customer Segment')
axes[1].set_ylabel('Number of Customers')
axes[1].tick_params(axis='x', rotation=45)

# Add data labels
for p in bar.patches:
    height = p.get_height()
    axes[1].text(p.get_x() + p.get_width() / 2,
                 height + 10,
                 f'{height:.0f}',
                 ha='center',
                 fontsize=9)

plt.tight_layout()
plt.show()


Monetary_total = RFM_df_merge.groupby('Segment')['Monetary'].sum()

# Tính tổng Monetary toàn bộ
total = RFM_df_merge['Monetary'].sum()

# Tạo labels với % đóng góp
labels = [f"{seg}\n ({count*100/total:.1f}%)"
          for seg, count in zip(Monetary_total.index, Monetary_total.values)]

# Treemap
plt.figure(figsize=(14, 8))
squarify.plot(
    sizes=Monetary_total.values,
    label=labels,
    color=sns.color_palette('Paired'),
    pad=True,
    alpha=0.9
)
plt.title('Treemap of Customer Segments by Total Revenue', fontsize=14)
plt.axis('off')
plt.show()
```


---

## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📌 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3
