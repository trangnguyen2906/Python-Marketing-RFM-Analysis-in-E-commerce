# 📊 Segment E-commerce Customers via RFM Analysis Using Python

<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1f7-rDGxryUE5_eM0DylfaMaF_hSRFE_S" />
</p>

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
3. [⚒️ Main Process](#%EF%B8%8F-main-process)
4. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  
### 🧩 Project Context

A **global e-commerce company** faced challenges in **targeting customer segments effectively**. The Marketing team wanted to reward loyal customers and identify those with high potential. However, customer segmentation was still done **manually in Excel**, which was **time-consuming** and **insufficient** for **large-scale strategy**.

### 📖 What is this project about? What Business Question will it solve?

This project uses Python to segment customers using the **RFM (Recency – Frequency – Monetary) model** , a proven marketing analysis technique that evaluates customers based on:

- **Recency:** How recently a customer made a purchase
- **Frequency** **:** How often they purchase
- **Monetary:** How much they spend
  
RFM analysis helps identify customer behavior patterns and enables businesses to **tailor strategies** accordingly. This project addresses the following **key business questions**:
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

<details>
<summary> <strong>Transaction Table</strong></summary>

| Column Name   | Data Type | Description                                                        |
|---------------|-----------|--------------------------------------------------------------------|
| `InvoiceNo`   | Nominal   | Unique invoice number (starts with ‘C’ for cancelled orders)       |
| `StockCode`   | Nominal   | Unique identifier for each product                                |
| `Description` | Nominal   | Name/description of the product                                   |
| `Quantity`    | Numeric   | Quantity of items per transaction                                 |
| `InvoiceDate` | DateTime  | Date and time of the transaction                                  |
| `UnitPrice`   | Numeric   | Price per item (in GBP)                                           |
| `CustomerID`  | Nominal   | Unique customer identifier                                        |
| `Country`     | Nominal   | Customer’s country of residence                                   |

</details>


## ⚒️ Main Process

## 1️⃣ Exploratory Data Analysis (EDA)
### 🔷 Check Data Types
- Performed **initial data inspection**:
  +   Verified datatypes and structure of dataset
  +   Identified **invalid entries and anomalies in data** (e.g., negative quantities, zero unit price)

![Your Image](https://drive.google.com/uc?export=view&id=1AYQHthnbtbJx0PL96k2K7RiZumgLaPz6)

### 🔷 Check Data Values
- The `Quantity` column contains **negative values**, which mostly correspond to **canceled orders** (identified by `InvoiceNo` starting with "C").
  
👉 These rows were removed to ensure accurate analysis.
- The `UnitPrice` column also has **negative values**, which are likely due to data entry errors or refund transactions.
  
👉 These entries were also **excluded** from further processing.


![Data values](https://drive.google.com/uc?export=view&id=1Hz6jg3dagY4QgPde-kzTH0vj0Z5TgytS)

### 🔷 Handle Missing & Duplicate Data
👉 **Detect missing data**

- Missing values were found **only** in the **`CustomerID` column.**
- These missing `CustomerIDs` are **not related to canceled transactions.**
- The majority of these null entries come from customers in the **United Kingdom.**

![Missing Values Screenshot](https://drive.google.com/uc?id=1nuU1tQSycOyaYMbv3agmEpGy3MgWBl7T)

👉 **Dealing with missing and duplicate data**
- **Missing values:** Only customerID is missing -> Doing RFM model need to group based on CustomerID --> **Drop** 25% missing value in CustomerID col
- **Duplicate data:** **drop** duplicated rows

## 2️⃣ RFM Analysis Preprocess
### 🔷 RFM Calculation
After cleaning and preparing the dataset, the RFM metrics were computed as follows:
- **Recency:**
  
  + Calculated as the number of days between the customer's most recent purchase and the last date in the dataset (`last_day`).
  + The smaller the number, the more recent the purchase, **more recent = better customer.**

- **Frequency:**
  
  + Measured by counting the number of **unique invoices** (`InvoiceNo`) per customer.
  + More orders indicate a **higher level of engagement and loyalty.**

- **Monetary:**
  
  + Total revenue generated by a customer, calculated as `Quantity * UnitPrice`.
  + Customers with higher spending are **more valuable to the business.**

- **Recency Reverse:**
  
  + To support scoring (where more recent = higher score), Recency values were reversed (turned negative).
  
  + This makes recent customers easier to rank higher using quantile scoring.

```python
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
- Each R, F, and M value was scored from **1 to 5 using quintile-based ranking**:
Quantile-based ranking divides a numerical feature into equal-sized groups, in this case, into 5 groups (quintiles). Each customer is assigned a score from **1 to 5** based on the quintile their value falls into:

- **Recency:**

  + Customers with **lower recency (more recent purchases)** are assigned a **higher score (5)**.
  
  + The most recent 20% of customers get score 5, the next 20% get 4, and so on down to 1.

- **Frequency & Monetary:**

  + Customers with **higher values** (more orders or more spending) get **higher scores.**
  
  + The top 20% (most frequent/spending customers) get 5, and the lowest 20% get 1.
- Scores were concatenated into a **3-digit RFM** score

```python
#Calculate RFM score
RFM_df['R_score'] = pd.qcut(RFM_df['Recency'], 5, labels=[5, 4, 3, 2, 1]).astype(str)
RFM_df['F_score'] = pd.qcut(RFM_df['Frequency'].rank(method='first'), 5, labels=[1, 2, 3, 4, 5]).astype(str)
RFM_df['M_score'] = pd.qcut(RFM_df['Monetary'], 5, labels=[1, 2, 3, 4, 5]).astype(str)
RFM_df['RFM_score'] = RFM_df['R_score'] + RFM_df['F_score'] + RFM_df['M_score']
RFM_df

```
![Quantile Explanation](https://drive.google.com/uc?export=view&id=1MGR3ZP6ShzpsTrd4RCWYcrPxepyClluE)


## 3️⃣ Customer Segmentation & Visualization
### 🔷 Segment Customers
- Customers were divided into **11 RFM-based segments** (e.g., 🏆 Champions, 🌱 Potential Loyalists, ⚠️ At Risk) to guide **personalized marketing and loyalty-focused strategies.**

<details>
<summary>🧩 <strong>RFM Score to Segment Mapping</strong></summary>

| Segment |  | Example RFM Scores | Customer Behavior |
|--------|------|---------------------|-------------------|
| **Champions** | 🏆 | `555`, `554`, `544`, `545` | Recently active, purchase frequently, and spend the most. |
| **Loyal Customers** | 💎 | `543`, `444`, `435`, `355` | Purchase often and spend consistently over time. |
| **Potential Loyalists** | 🌱 | `553`, `552`, `551`, `541` | Recently acquired with frequent or high spending patterns. |
| **New Customers** | 🆕 | `512`, `511`, `422`, `421` | Recently made first purchases with low frequency or spend. |
| **Promising** | ✨ | `525`, `524`, `523`, `522` | Moderately recent activity with medium purchase volume or value. |
| **Need Attention** | 👀 | `535`, `534`, `443`, `434` | Previously active but now purchasing less often or spending less. |
| **About to Sleep** | 😴 | `331`, `321`, `312`, `221` | Low purchase frequency and decreasing recency of transactions. |
| **At Risk** | ⚠️ | `255`, `254`, `245`, `244` | Historically high frequency or value, but no recent activity. |
| **Cannot Lose Them** | 🚨 | `155`, `154`, `144`, `214` | Used to be highly active and valuable, now inactive. |
| **Hibernating** | 🛌 | `332`, `322`, `233`, `232` | Infrequent purchases, low spend, and inactive for a while. |
| **Lost Customers** | ❌ | `111`, `112`, `121`, `131` | No recent activity, low purchase frequency, and minimal value. |

</details>

```python
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

### 🔹 1. Examine Distribution of RFM Metrics
> 🎯 **Goal:** Understand general behavior across Recency, Frequency, and Monetary metrics to identify broad customer patterns.

```python
fig, ax = plt.subplots(figsize=(10, 6))
color=sns.color_palette('Blues')[3]
sns.histplot(RFM_df_merge['Recency'], ax=ax, kde=True, stat='density', color = color , edgecolor='black')
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
sns.histplot(RFM_df_merge['Monetary'], ax=ax, kde=True, stat='density',color = color , edgecolor='black', binwidth = 500)
ax.set(xlabel='Monetary', ylabel='Distribution', xlim = (-1000,15000), title='Distribution of Monetary')
plt.show()
```

<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1ifkdRC4fwjxbZevRMHQJ9SnNHB7Wy2im" width="500"/>
</p>

![RFM Segment Distribution](https://drive.google.com/uc?export=view&id=1AXHH1bvS9OIy7k1Qzrn4lh3qKGQeaC3r)

**🔍 Key Findings:**
- 🔁 **Frequency:** Over **70%** of customers made **only one - two purchase**, revealing a significant retention issue and low repeat-buy behavior. **(KEY WEAKNESSES)**
- ⏱️ **Recency:** Most customers made purchases within the last **50 days**, showing strong short-term engagement. However, there's a long tail of users who **haven't returned for 6+ months**.
- 💰 **Monetary:** Spending is highly skewed. **Most customers** **spend little**, while a small number of high-value outliers generate the majority of revenue.
  
### 🔹 2. Segment Distribution by Count and Revenue
> 🎯 **Goal:** Compare customer segments based on their **population size** and **revenue contribution** to identify which groups are most valuable and which ones require re-evaluation.

```python
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

<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1Ia3SZA9J6Y0AoaWTLXZUbLAU26sXvneY" />
</p>

<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1v5XzgdM2nDMf8RFI8XzuaaBrRRC4f5mX" width="600" />
</p>

**🔍 Key Findings:**
- 👥 **Customer Count:** The majority of customers fall into **mid-tier segments** like **Need Attention, Potential Loyalists, and At Risk**, suggesting potential for growth or re-engagement.
- **💸 Revenue Contribution:** **Over 60%** of total revenue comes from **Champions alone**, followed by **Loyal and Need Attention** groups. Many segments (like Lost or New Customers) contribute little revenue despite notable size.
- ⚠️ Strategic Gap: Some segments with high population (Hibernating, At Risk) offer low return, indicating **poor engagement or lack of effective retention strategies.**

### 🔹 3. Group Segments for Marketing Campaign Strategy

🎯 **Goal**: Reorganize the 11 original RFM segments into **3 broader customer groups** to simplify targeting and align marketing efforts with customer value and potential.

**1.** **Loyal & VIP Customers**:  Champions, Loyal (R & F scores = 4/5; **contribute approximately 75% of total revenue**)

**2.** **Potential Customers**:  Potential Loyalist, Promising, Need Attention, New Customers (Moderate to high R & F scores: 2, 3, 4, 5 — lower share of revenue but strong potential)

**3.** **At Risk & Lost Customers**: At Risk, Cannot Lose Them, Hibernating Customers, Lost Customers, About To Sleep (Low recency score — haven’t purchased for a long time)

```python
def regroup_segment(segment):
    if segment in ['Champions', 'Loyal']:
        return 'Loyal & VIP Customers'
    elif segment in ['Potential Loyalist', 'New Customers', 'Promising', 'Need Attention']:
        return 'Potential Customers'
    else:
        return 'At Risk & Lost Customers'

RFM_df_merge['Customer_Group'] = RFM_df_merge['Segment'].apply(regroup_segment)
RFM_df_merge

plt.figure(figsize=(8, 5))
sns.countplot(data=RFM_df_merge, x='Customer_Group', palette='Set2')
plt.title('Customer Distribution by New Groups')
plt.xticks(rotation=15)
plt.show()
```
![Customer Distribution by New Groups](https://drive.google.com/uc?id=1HoAHVU734nGkNi4qCpvOgchsOlbnT9f0)

🔍 **Key Findings:**

- 🏆 **Group 1 – Loyal & VIP Customers**: Champions and Loyal customers with the **highest RFM scores (4–5)**. Small in number but **contribute ~75% of revenue**. Best targets for gratitude and loyalty-focused campaigns.

- 🌱 **Group 2 – Potential Customers**: Includes Potential Loyalists, Promising, Need Attention, and New Customers. **Moderate RFM scores** with growth potential. Ideal for nurturing via personalized offers or engagement tactics.

- ⚠️ **Group 3 – At Risk & Lost Customers**: At Risk, Cannot Lose Them, Hibernating, Lost, and About to Sleep segments. Low engagement or long inactivity. Costly to re-engage with uncertain returns.

### 🔹 4. Correlation Between RFM Metrics in Each Group
> 🎯 Goal: Analyze how Recency, Frequency, and Monetary values relate to each other within different customer groups to uncover consistent behavioral patterns or weak links.

![Recency Frequency Monetary AOV by Segment](https://drive.google.com/uc?id=1dl3XB0mKmQQtvl2KaEZyM_M7Ma8unNEo)


🔍 **Key Findings:**
Loyal & VIP Customers show a quite strong positive Frequency–Monetary **correlation (crl = 0.55)**, while Potential Customers show no clear RFM relationships, indicating inconsistent and less predictable behavior.

### 🔹 5. Convert Potential Customers → Loyal Customers
> 🎯 Goal: Identify key behavioral gaps between mid-tier segments and top-performing customers to guide conversion strategies for turning potential customers into loyal ones.

🧠 To do this, I compare mean and median values of Recency, Frequency, and Monetary, along with AOV (Average Order Value) to gain reliable insights into typical behavior (median) while still capturing value outliers (mean). This helps prioritize which segments have untapped value and how to upgrade them effectively.

```python
#mean, median of segments except at risk and lost customers
comparison_df = RFM_df_merge[RFM_df_merge['Customer_Group'] != 'At Risk & Lost Customers']
comparison_summary = comparison_df.groupby('Segment')[['Recency', 'Frequency', 'Monetary']].agg(['mean', 'median']).reset_index()
comparison_summary.columns = ['_'.join(col).strip('_') for col in comparison_summary.columns.values]
comparison_summary['AOV_mean'] = comparison_summary['Monetary_mean'] / comparison_summary['Frequency_mean']
comparison_summary['AOV_median'] = comparison_summary['Monetary_median'] / comparison_summary['Frequency_median']
comparison_summary

fig, axes = plt.subplots(2, 2, figsize=(16, 10))
sns.barplot(x='Recency_median', y='Segment', data=comparison_summary, ax=axes[0, 0], color='navy').set(title='Recency (Median)')
sns.barplot(x='Frequency_median', y='Segment', data=comparison_summary, ax=axes[0, 1], color='green').set(title='Frequency (Median)')
sns.barplot(x='Monetary_median', y='Segment', data=comparison_summary, ax=axes[1, 0], color='orange').set(title='Monetary (Median)')
sns.barplot(x='AOV_median', y='Segment', data=comparison_summary, ax=axes[1, 1], color='darkred').set(title='AOV (Median)')
plt.tight_layout()
plt.show()
```

<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1fj1IMV-yaQvx63Z3I8SzZPhnGElFg5Rz" width="700" />
</p>


<p align="center">
  <img src="https://drive.google.com/uc?export=view&id=1vRvKXjU1t3zZ0jSPzo811jJws_e3fKuO" width="900" />
</p>

🔍 **Key Findings:**
- 📅 **Recency:** Potential Loyalists and New Customers made relatively recent purchases, similar to Loyal customers, indicating recent engagement and reactivation potential.
- 💸 **Monetary & AOV:** Despite lower frequency, AOV for Potential Loyalists is fairly strong. This suggests they’re willing to spend but need encouragement to buy more often.
  
---

## 📌 Final Conclusion & Recommendations

👉🏻 **Based on the RFM analysis and behavioral insights**, I recommend the following targeted strategies to the **Marketing Team** and the **CRM Team**:

### 🌸 **Recommendations for the Marketing Team**  
To run an effective gratitude campaign, the focus should be on **high-impact segments** and **tailored campaign methods**.


### ✅ **Campaign Target Priorities**:  
Focus loyalty and gratitude campaigns on **Champions** and **Loyal customers (Group 1)**

- **Why**: Small in size but **contribute ~75% of total revenue**
- **Strategy**: Send **VIP appreciation gifts**, **early product access**, **birthday discounts**, or **tiered loyalty perks**
- 🎯 **Goal**: **Reinforce trust**, **maximize retention**, and encourage **repeat purchases and advocacy**


### 💜 Run Gratitude Campaigns for Group 2 – **Potential Customers**

- **Why**: **Strong Average Order Value (AOV)** but **low purchase frequency**, indicating **untapped potential**
- **Tactics**:  
  - **Email remarketing**  
  - **Personalized follow-up** after first or second purchase (💌 send thank-you messages)  
  - **Limited-time discounts** or **“Buy again and save”** offers  
- 🎯 **Goal**: Encourage **repeat behavior**, nudge toward **loyalty**, and acknowledge **early engagement**


### ❌ **Do NOT Run Gratitude Campaigns for Group 3 – At Risk & Lost Customers**

- **Why**: These users are **inactive**, show **long gaps in activity**, and **contribute little revenue**


### 🌟 **Recommendations for CRM Team**  
Focus on turning **Potential Customers** into **Loyal Customers** by improving **Frequency** (key metric).

- ⚙️ **Automate Post-Purchase Journeys**:  
  Trigger email/SMS after first or second purchase:  
  _“Thanks for buying! Get 10% off your next order”_ → Drives **2nd/3rd purchases** and helps **build habit**

- 🛍️ **Personalized product suggestions**

- 📈 **Show Progress Toward Rewards**:  
  E.g., _“1 order away from VIP”_ → Use **psychological momentum** to **increase purchase frequency**

  
