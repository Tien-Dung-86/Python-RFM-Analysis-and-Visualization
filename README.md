<h2>[Python] RFM Analysis and Visualization</h2> <br/><br/>

<h3>I. Introduction</h3>

In this project, I conducted an **RFM** (Recency, Frequency, Monetary) analysis for a **global retail company** - SuperStore, utilizing Python to segment customers and deliver actionable insights **for the Marketing and Sales teams**. Through exploratory **data analysis**, **segmentation modeling**, and visualizations, I supported the teams in optimizing customer engagement and enhancing strategic decision-making by identifying key customer groups for targeted campaigns.
<br/>

**Dataset** <br/><br/>
This is a transnational data set which contains all the transactions occurring between 01/12/2010 and 09/12/2011 for a UK-based and registered non-store online retail. The company mainly sells unique all-occasion gifts. Many customers of the company are wholesalers.
<br/>

| Column | Explaintion |
| :--: | :-- |
| InvoiceNo | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'C', it indicates a cancellation. |
| StockCode | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product. |
| Description | Product (item) name. Nominal. |
| Quantity | The quantities of each product (item) per transaction. Numeric. |
| InvoiceDate | Invoice Date and time. Numeric, the day and time when each transaction was generated. |
| UnitPrice | Unit price. Numeric, Product price per unit in sterling. |
| CustomerID | Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer. |
| Country | Country name. Nominal, the name of the country where each customer resides. |
<br/>

<h3>II. Data Preparation</h3>

**Convert type** <br/><br/>
'UnitPrice' and 'CustomerID' are current type is 'float64' -> convert type to 'int64' 
```
# Check column type
df.dtypes

# Convert type float64 to int64
df['UnitPrice'] = df['UnitPrice'].astype(int)
df['CustomerID'] = df['CustomerID'].astype(int)
```
<br/>

**Cleaning Data**<br/><br/>
This part involved checking for missing values, duplicates, and incorrect data types. Appropriate actions were taken, such as imputing missing values, removing duplicates, and correcting data types to ensure data integrity. Additionally, any incorrect or outlier values were identified and handled based on the dataset's context to maintain accuracy and consistency.

```
# Check for missing data in each column
df.isna().sum()

# Check duplicated
print(df.duplicated().sum(), " duplicates")

# Check incorrect value
df.describe()

```

**Actions**
```
# Remove record had null CustomerID
df = df.dropna(subset=['CustomerID'])

# Remove duplicated
df = df.drop_duplicates()

# Filter data where column 'Quantity' and 'UnitPrice' > 0, remove transaction with InvoiceNo start with 'C' character (Cancelled transaction)
df = df[
    (df['Quantity'] > 0) &
    (df['UnitPrice'] > 0) &
    (~df['InvoiceNo'].astype(str).str.startswith('C'))
]
```
<br/>

**RFM Calculation and Assign scores** <br/><br/>
After cleaning data, the Recency, Frequency, and Monetary values were calculated for each customer. Recency was determined based on the number of days since the last purchase, Frequency measured the total number of transactions, and Monetary value represented the total spending of each customer.
```
last_purchase_date = df.groupby('CustomerID')['InvoiceDate'].max().reset_index(name='LastPurchaseDate')

# Create 'total' column:
df['Total'] = df['Quantity'] * df['UnitPrice']

# Specify the reference date for recency calculations
reference_date = df['InvoiceDate'].max()

# Calculate rfm
rfm_df = df.groupby('CustomerID').agg(
    # calculate recency
    Recency=('InvoiceDate', lambda x: (reference_date - x.max()).days),
    # calculate frequency
    Frequency=('InvoiceNo', 'nunique'),
    # calculate monetary
    Monetary=('Total', 'sum')
).reset_index()
rfm_df = last_purchase_date.merge(rfm_df, on='CustomerID', how='left')
```

Assign scores to eac RFM component, categorizing customers into segments based on their relative RFM values. These scores served as the foundation for customer segmentation and further analysis. <br/> <br/>
```
# RFM Scoring
rfm_df['R_Score'] = pd.qcut(rfm_df['Recency'], q=5, labels=[5, 4, 3, 2, 1])
rfm_df['F_Score'] = pd.qcut(rfm_df['Frequency'].rank(method='first'), q=5, labels=[1, 2, 3, 4, 5])
rfm_df['M_Score'] = pd.qcut(rfm_df['Monetary'], q=5, labels=[1, 2, 3, 4, 5])

# Combine RFM scores into a single RFM score column
rfm_df['RFM_Score'] = rfm_df['R_Score'].astype(str) + rfm_df['F_Score'].astype(str) + rfm_df['M_Score'].astype(str)
```

**Segmentation**
```
segmentation = {
    'Champions': ['555', '554', '544', '545', '454', '455', '445'],
    'Loyal Customers': ['543', '444', '435', '355', '354', '345', '344', '335'],
    'Potential Loyalist': ['553', '551', '552', '541', '542', '533', '532', '531', '452', '451', '442', '441', '431', '453', '433', '432', '423', '353', '352', '351', '342', '341', '333', '323'],
    'New Customers': ['512', '511', '422', '421', '412', '411', '311'],
    'Promising': ['525', '524', '523', '522', '521', '515', '514', '513', '425', '424', '413', '414', '415', '315', '314', '313'],
    'Need Attention': ['535', '534', '443', '434', '343', '334', '325', '324'],
    'About To Sleep': ['331', '321', '312', '221', '213', '231', '241', '251'],
    'At Risk': ['255', '254', '245', '244', '253', '252', '243', '242', '235', '234', '225', '224', '153', '152', '145', '143', '142', '135', '134', '133', '125', '124'],
    'Cannot Lose Them': ['155', '154', '144', '214', '215', '115', '114', '113'],
    'Hibernating customers': ['332', '322', '233', '232', '223', '222', '132', '123', '122', '212', '211'],
    'Lost customers': ['111', '112', '121', '131', '141', '151']
}

# Map RFM scores to customer segments
def assign_segment(rfm_score):
    for segment, scores in segmentation.items():
        if rfm_score in scores:
            return segment
    return 'Other'

rfm_df['Segment'] = rfm_df['RFM_Score'].apply(assign_segment)
```
<br/>

<h3>III. Data Visualization and Insight</h3>

**RFM Distribution Analysis**
```
#Distribution of Recency
fig, ax = plt.subplots(figsize=(12, 3))
sns.histplot(data=rfm_df, x='Recency', bins =10, ax=ax)
ax.set_title('Distribution of Recency')
ax.set_xlim(left=0)
ax.yaxis.set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```
<br/>
**Recency Distribution:** The graph shows the distribution of customers based on how recently they made a purchase.

![alt text](https://github.com/Tien-Dung-86/Python-RFM-Analysis-and-Visualization/blob/master/Visualization/1_Recency.png)

**Observations:** The two highest bars are in range 0 - 100 (over 2700 customers). indicating that the company has a strong base of recent customers, which is positive.

```
#Distribution of Frequency
binsF = [0, 2, 5, 20, np.inf]
labelsF = ['1-2', '2-5', '5-20', '20+']
rfm_df['FrequencyGroup'] = pd.cut(rfm_df['Frequency'], bins=binsF, labels=labelsF)
fig, ax = plt.subplots(figsize=(8, 3))
sns.countplot(x='FrequencyGroup', data=rfm_df, ax=ax)
ax.set_title('Distribution of Frequency')
ax.yaxis.set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```

<br/>
**Frequency Distribution:** This graph shows how often customers make purchases.

![alt text](https://github.com/Tien-Dung-86/Python-RFM-Analysis-and-Visualization/blob/master/Visualization/2_Frequency.png)

**Observations:**
<ul>
    <li>The majority of customers in the 1-2 frequency group (2130). Hence, there's potential to convert one-time buyers into repeat customers.</li>
    <li>There are up to 92 customers in group 20+</li>
</ul>

```
<br/>

#Distribution of Monetary
binsM = [0, 100, 1000, 10000, np.inf]
labelsM = ['0-100', '100-1k', '1k-10k', '10k+']
rfm_df['MonetaryGroup'] = pd.cut(rfm_df['Monetary'], bins=binsM, labels=labelsM)
fig, ax = plt.subplots(figsize=(8, 3))
sns.countplot(x='MonetaryGroup', data=rfm_df, ax=ax)
ax.set_title('Distribution of Monetary')
ax.yaxis.set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```
<br/>

**Monetary Distribution:** This graph shows the distribution of customer spending.
![alt text](https://github.com/Tien-Dung-86/Python-RFM-Analysis-and-Visualization/blob/master/Visualization/3_Monetary.png)

**Observations:** The company has a lot of mid-range spenders with 2524 in range 100-1k and 1536 in the 1k-10k.
<br/><br/>


**Customer'number by RFM segments**
```
segment_colors = {
    'Champions': '#FF0000',
    'Loyal': '#00FFFF',
    'Potential Loyalist': '#00FF00',
    'At Risk': '#FFFF00',
    'Hibernating customers': '#800080',
    'Lost customers': '#FFA500',
    'Need Attention': '#A52A2A',
    'About To Sleep': '#808000',
    'New Customers': '#FFC0CB',
    'Promising': '#FF00FF',
    'Cannot Lose Them': '#736F6E'
}

# Calculate segment counts and percentage shares
segment_counts = rfm_df['Segment'].value_counts()
segment_shares = (segment_counts / segment_counts.sum()) * 100

# Create a bar chart
fig, ax = plt.subplots(figsize=(14, 8))

# Bar plot
bars = ax.bar(segment_counts.index,
              segment_counts.values,
              color=[segment_colors.get(segment, '#808080') for segment in segment_counts.index],
              edgecolor="black")

# Adding labels on top of bars
for bar, count, share in zip(bars, segment_counts.values, segment_shares.values):
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
            f"{count:,}\n{share:.1f}%",  # Format the percentage with 1 decimal
            ha='center', va='bottom', fontsize=12)

# Title and labels
plt.title('Number of Customers by RFM Segment', fontsize=16)
plt.ylabel('Number of Customers', fontsize=14)
plt.xticks(rotation=45, ha='right')  # Rotate x-axis labels for better readability

# Display the plot
plt.show()
```

![alt text](https://github.com/Tien-Dung-86/Python-RFM-Analysis-and-Visualization/blob/master/Visualization/4_RFM-segments.png)
<br/>

```
# % and Monetary values by Segment
segment_monetary = rfm_df.groupby('Segment')['Monetary'].sum().reset_index()
total_monetary = segment_monetary['Monetary'].sum()
segment_monetary['Percentage'] = segment_monetary['Monetary'] / total_monetary * 100

segment_monetary = segment_monetary.sort_values('Monetary', ascending=False)

# Create the treemap

fig, ax = plt.subplots(1, figsize=(20,8))
squarify.plot(sizes=segment_monetary['Monetary'],
              label=[f"{s}\n${int(m):,}\n{int(p)}%"
                     for s, m, p in zip(segment_monetary['Segment'],
                                        segment_monetary['Monetary'],
                                        segment_monetary['Percentage'])],
              color=[segment_colors.get(segment, '#808080') for segment in segment_counts.index],
              alpha=0.8,
              bar_kwargs=dict(linewidth=1.5, edgecolor="white"))

plt.title('Total Monetary Value by RFM Segment', fontsize=16)
plt.axis('off')

plt.show()
```

![alt text](https://github.com/Tien-Dung-86/Python-RFM-Analysis-and-Visualization/blob/master/Visualization/5_Heatmap-segments.png)

**Observations:**
<br/>
<br/>
<ul>
    <li>SuperStore Company's customer base primarily consists of "Champions" (19.8%), "Hibernating customers" (16.4%), "Lost customers" (11.2%), "Loyal Customers" and "Potential Loyalist" (9.7% for each segment) segments.</li>
    <li>"Champion" accounts for 66% of the company's total revenue, followed by "Loyal Customers" with 10%.</li>
    <li>The small "New Customers" (0%) and "promising" (1%) segments indicate a need for customer acquisition and growth strategies.</li>
</ul>
Conclude: SuperStore Company has a mix of highly loyal, potentially loyal, and at-risk customers, presenting opportunities for targeted retention, acquisition, and cultivation efforts to optimize the value of its customer base.
<br/>
<br/>

<h3>IV. Segment Characteristics and Recommendations</h3>

| Segment | Characteristics | Recommendations |
| :-- | :-- | :-- |
| Champions (19.8% of customers, 66% of value) | <ul><li>Highest monetary value, frequent recent purchases</li><li>Likely long-term customers with strong brand loyalty</li><li>High average order value</li></ul> | <ul><li>Send personalized "Year in Review" thank you cards highlighting their top purchases</li><li>Offer exclusive early access to holiday sales with additional discount</li><li>Provide complimentary gift wrapping and priority shipping</li><li>Invite to a virtual VIP holiday event with special product previews</li></ul> |
| Loyal (9.7% of customers, 10% of value) | <ul><li>High value, consistent engagement</li><li>Slightly lower recency or frequency than Champions</li></ul> | <ul><li>Send personalized holiday greeting with a thank you gift (e.g., branded calendar)/li><li>Offer a loyalty point multiplier for holiday purchases</li><li>Provide a surprise upgrade or add-on with their next purchase</li></ul> |
| At Risk (9.4% of customers, 7% of value) | <ul><li>Decreasing engagement</li><li>Previously valuable customers</li><li>High urgency for re-engagement</li></ul> | <ul><li>Provide a dedicated customer service line for any issues or questions</li><li>Send a survey with a gift card reward to understand their needs and preferences</li></ul> |
| Need Attention (6% of customers, 3% of value) | <ul><li>Moderate value</li><li>Declining engagement</li><li>May be price-sensitive or have changing needs</li></ul> | <ul><li>Create a personalized product recommendation list for holiday shopping</li><li>Offer incentives for feedback</li></ul> |
| Hibernating customers (15% of customers, 3% of value) | <ul><li>No recent activity</li><li>Previously engaged customers</li></ul> | <ul><li>Send a year-end catalog featuring best-sellers and new products</li><li>Consider retargeting ads</li></ul> |
| Potential Loyalists (9.7% of customers, 2% of value) | <ul><li>Recent purchases, moderate frequency</li><li>Lower monetary value than Loyal</li></ul> | <ul><li>Offer a special discount on a product category they haven't tried yet</li><li>Provide a free consultation or product demo as a holiday bonus</li></ul> |
| Cannot Lose Them (2.3% of customers, 2% of value) | <ul><li>High-value customers</li><li>Recent drop in engagement</li></ul> | <ul><li>Send a heartfelt "We Miss You" holiday message with a significant comeback offer</li><li>Offer an exclusive "loyal customer" discount on their favorite products</li></ul> |
| Promising (3.1% of customers, 1% of value) | <ul><li>Recent engagement</li><li>Moderate purchasing behavior</li></ul> | <ul><li>Nurture with targeted content and offers</li><li>Create a personalized "Holiday Must-Haves" list based on browsing history</li><li>Offer a progressive discount: save more on each subsequent holiday purchase</li></ul> |
| New Customers (6.2% of customers, 0% of value) | <ul><li>Recent first purchase</li><li>Unknown long-term value</li><li>Limited data on preferences and behavior</li></ul> | <ul><li>Offer a "Holiday Newcomer" discount on their second purchase</li><li>Provide a guided tour of product ranges and services via email series</li></ul> |
| About To Sleep (6.3% of customers, 0% of value) | <ul><li>Low recent activity</li><li>Previously active customers</li></ul> | <ul><li>Send a holiday-themed product update highlighting new features or improvements</li><li>Time-limited offers to encourage action</li></ul> |
| Lost customers (11.2% of customers, 0% of value) | <ul><li>Longest period of inactivity</li><li>Lowest engagement</li></ul> | <ul><li>Send a feedback request with a holiday gift incentive for responses</li><li>Analyze reasons for loss</li><li>Use insights to prevent future customer loss</li></ul> |
