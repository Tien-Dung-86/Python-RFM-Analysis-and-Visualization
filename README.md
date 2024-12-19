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
df['UnitPrice'] = df['UnitPrice'].astype(int)
df['CustomerID'] = df['CustomerID'].astype(int)
```
<br/><br/>

**Cleaning Data**<br/><br/>
This part involved checking for missing values, duplicates, and incorrect data types. Appropriate actions were taken, such as imputing missing values, removing duplicates, and correcting data types to ensure data integrity. Additionally, any incorrect or outlier values were identified and handled based on the dataset's context to maintain accuracy and consistency.
