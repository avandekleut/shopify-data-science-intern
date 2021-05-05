```python
%config Completer.use_jedi = False
```

**Question 1: Given some sample data, write a program to answer the following: click here to access the required data set. On Shopify, we have exactly 100 sneaker shops, and each of these shops sells only one model of shoe. We want to do some analysis of the average order value (AOV). When we look at orders data over a 30 day window, we naively calculate an AOV of \$3145.13. Given that we know these shops are selling sneakers, a relatively affordable item, something seems wrong with our analysis.**


```python
import pandas as pd
```


```python
data = pd.read_csv('data.csv')
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>shop_id</th>
      <th>user_id</th>
      <th>order_amount</th>
      <th>total_items</th>
      <th>payment_method</th>
      <th>created_at</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>53</td>
      <td>746</td>
      <td>224</td>
      <td>2</td>
      <td>cash</td>
      <td>2017-03-13 12:36:56</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>92</td>
      <td>925</td>
      <td>90</td>
      <td>1</td>
      <td>cash</td>
      <td>2017-03-03 17:38:52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>44</td>
      <td>861</td>
      <td>144</td>
      <td>1</td>
      <td>cash</td>
      <td>2017-03-14 4:23:56</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>18</td>
      <td>935</td>
      <td>156</td>
      <td>1</td>
      <td>credit_card</td>
      <td>2017-03-26 12:43:37</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>18</td>
      <td>883</td>
      <td>156</td>
      <td>1</td>
      <td>credit_card</td>
      <td>2017-03-01 4:35:11</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4995</th>
      <td>4996</td>
      <td>73</td>
      <td>993</td>
      <td>330</td>
      <td>2</td>
      <td>debit</td>
      <td>2017-03-30 13:47:17</td>
    </tr>
    <tr>
      <th>4996</th>
      <td>4997</td>
      <td>48</td>
      <td>789</td>
      <td>234</td>
      <td>2</td>
      <td>cash</td>
      <td>2017-03-16 20:36:16</td>
    </tr>
    <tr>
      <th>4997</th>
      <td>4998</td>
      <td>56</td>
      <td>867</td>
      <td>351</td>
      <td>3</td>
      <td>cash</td>
      <td>2017-03-19 5:42:42</td>
    </tr>
    <tr>
      <th>4998</th>
      <td>4999</td>
      <td>60</td>
      <td>825</td>
      <td>354</td>
      <td>2</td>
      <td>credit_card</td>
      <td>2017-03-16 14:51:18</td>
    </tr>
    <tr>
      <th>4999</th>
      <td>5000</td>
      <td>44</td>
      <td>734</td>
      <td>288</td>
      <td>2</td>
      <td>debit</td>
      <td>2017-03-18 15:48:18</td>
    </tr>
  </tbody>
</table>
<p>5000 rows × 7 columns</p>
</div>



**a) Think about what could be going wrong with our calculation. Think about a better way to evaluate this data.**

First let's understand how the AOV was calculated.


```python
data['order_amount'].mean()
```




    3145.128



The average order amount was naively calculated as the average over orders. However, this number is much higher than we would expect based on the preview of the table, which should probably be somewhere between 200 and 300.


```python
import seaborn as sns
```


```python
sns.displot(data['order_amount'], log_scale=True)
```




    <seaborn.axisgrid.FacetGrid at 0x7f697b781cd0>




![png](output_9_1.png)


There are purchases with order amounts of almost one million dollars. The data is skewed far to the right, causing the mean to far exceed the median.

**b) What metric would you report for this dataset?**

This depends on what we are interested in knowing about. If we want to summarize the central tendency of the amount of money spent per transaction, a more realistic measure would be to report the median order value.

The median is computed based only on the *order* of the data, and is not impacted by the actual *value* of the data. This lets us avoid the problem of outliers with high values.

**c) What is its value?**


```python
data['order_amount'].median()
```




    284.0



**Question 2: For this question you’ll need to use SQL. Follow this link to access the data set required for the challenge. Please use queries to answer the following questions. Paste your queries along with your final numerical answers below.**

**a) How many orders were shipped by Speedy Express in total?**

Join Orders and Shippers to get Shipper name. Select rows with name matching 'Speedy Express' and use COUNT to count rows.

```sql
SELECT COUNT(*) FROM 
Orders
INNER JOIN Shippers
ON Orders.ShipperID = Shippers.ShipperID
WHERE Shippers.ShipperName = 'Speedy Express';
```

```
54 
```

**b) What is the last name of the employee with the most orders?**

First count orders by Employee ID and rename as NumOrders. Next join Employee ID on employee table to get last names. Finally, order by NumOrders in descending order, and grab the LastName from the top 1 row.

```sql
SELECT TOP 1, Employees.LastName
FROM 
    (SELECT EmployeeID, COUNT(*) as NumOrders
    From Orders
    GROUP BY EmployeeID) AS EmpOrders
INNER JOIN Employees 
ON EmpOrders.EmployeeID = Employees.EmployeeID
ORDER BY NumOrders DESC
```

**c) What product was ordered the most by customers in Germany?**


Product Name and Country need to be linked. I need to link the Product Table to the Customer Table through the Orders Table.

This query just involves linking the keys from Products -> OrderDetails -> Orders -> Customers. We do this using a series of Inner Joins. Next select only orders made by customers in Germany. Count by ProductName and sort in descending order. Finally, grab the ProductName from the top row.

```sql
SELECT TOP 1, ProductName
FROM
    (SELECT Products.ProductName, COUNT(*) AS Num
    FROM 
        (((Products
        INNER JOIN OrderDetails
        ON Products.ProductID = OrderDetails.ProductID)
        INNER JOIN Orders
        ON OrderDetails.OrderID = Orders.OrderID)
        INNER JOIN Customers
        ON Orders.CustomerID = Customers.CustomerID)
        WHERE Customers.Country = 'Germany'
    GROUP BY Products.ProductName)
ORDER BY Num DESC
  
```

```
Gorgonzola Telino 
```
