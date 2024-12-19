**[Python] RFM Analysis and Visualization** <br/><br/>
**I. Introduction** <br/><br/>
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

**II. Data Preparation** <br/><br/>
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

**III. Data Visualization and Insight** <br/><br/>
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
