
## Final Project Module 2: Statistical Analysis and Hypothesis Testing


* Student name: NICK OSELAND
* Student pace: FULL TIME
* Instructor name: RAFAEL CARRASCO


```python
jupyter nbconvert --to markdown student.ipynb
cp student.md README.md

```


      File "<ipython-input-50-1bdbe2c1b72d>", line 1
        jupyter nbconvert --to markdown student.ipynb
                        ^
    SyntaxError: invalid syntax



## Table of contents

* [Project Northwind Database](#Project)
* [Question #1](#Question#1)    
* [Question #2](#Question#2)  
* [Question #3](#Question#3)
* [Question #4](#Question#4)
* [Conclusion](#Conclusion)

![image.png](attachment:image.png)

Following prescribed methods, and the directions given, we will take this general approach for each question:
#### 1. EXPLORATORY RESEARCH
- Exploring the existing data
- Feature engineering

#### 2. Define the HYPOTHESIS
- Defining (clearly) the null and alternative hypothesis
- Explaining how to choose between using a one-tailed test (directional hypothesis) or a two-tailed test (non-directional hypothesis)
- Setting significance level

#### 3. STATISTICAL TESTING
- Choosing the appropriate statistical test
- Checking its assumptions
- Calculating test statistic and p-value
- Calculating effect size

#### 4. Drawing CONCLUSIONS 
- Evaluating and interpreting the results

## Project Northwind Database

The Northwind database is a sample database used by Microsoft to demonstrate the features of some of its products, including SQL Server and Microsoft Access. The database contains the sales data for a fictitious company called Northwind Traders, which imports and exports specialty foods from around the world.

![Northwind_ERD.png](attachment:Northwind_ERD.png)

# The Objective

The objective of this project is to query the database to get the data needed to perform a statistical analysis. In this statistical analysis, we'll need to perform a hypothesis test (or perhaps several) to answer the following questions:

- **Question 1**: Does  applying a discount have a statistically significant effect on the quantity of a product in an order? If so, at what level(s) of discount?
- **Question 2**: Do larger discount amounts affect the sales quantity more than lower discount amounts?
- **Question 3**: Is there a statistically significant difference in the output/performance of shipping companies?
- **Question 4**: Does the time in the year affect the quantity of product sold?

## Importing libraries


```python
import sqlite3 # for database
import pandas as pd # for dataframe
import matplotlib.pyplot as plt # plotting
import seaborn as sns # plotting
import numpy as np # analysis
from scipy import stats # significance levels, normality
import itertools # for combinations
import statsmodels.api as sm # anova
from statsmodels.formula.api import ols
import scipy.stats as scs

import warnings
warnings.filterwarnings('ignore') # hide matplotlib warnings
```

## Connecting to database


```python
# Connecting to database
conn = sqlite3.connect('Northwind_small.sqlite')
c = conn.cursor()
```


```python
# List of all tables
tables = c.execute("SELECT name FROM sqlite_master WHERE type='table';").fetchall()
tables = [i[0] for i in tables]
print(tables)
```

    ['Employee', 'Category', 'Customer', 'Shipper', 'Supplier', 'Order', 'Product', 'OrderDetail', 'CustomerCustomerDemo', 'CustomerDemographic', 'Region', 'Territory', 'EmployeeTerritory']


## Converting all tables into dataframes

I was doing this one table at a time, until I ran across this amazing bit of code.


```python
# Loop to put all tables into pandas dataframes
dfs = []
for i in tables:
    table = c.execute('select * from "'+i+'"').fetchall()
    columns = c.execute('PRAGMA table_info("'+i+'")').fetchall()
    df = pd.DataFrame(table, columns=[i[1] for i in columns])
    # function to make a string into variable name
    # great bit of code I found while researching
    conv = i+"_df"
    exec(conv + " = df") # => TableName_df
    # Keep all dataframe names in the list to remember what we have
    dfs.append(conv)
    print(conv)
```

    Employee_df
    Category_df
    Customer_df
    Shipper_df
    Supplier_df
    Order_df
    Product_df
    OrderDetail_df
    CustomerCustomerDemo_df
    CustomerDemographic_df
    Region_df
    Territory_df
    EmployeeTerritory_df


# Exploratory Data Analysis


```python
Order_df.head(10)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>EmployeeId</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>ShipVia</th>
      <th>Freight</th>
      <th>ShipName</th>
      <th>ShipAddress</th>
      <th>ShipCity</th>
      <th>ShipRegion</th>
      <th>ShipPostalCode</th>
      <th>ShipCountry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10248</td>
      <td>VINET</td>
      <td>5</td>
      <td>2012-07-04</td>
      <td>2012-08-01</td>
      <td>2012-07-16</td>
      <td>3</td>
      <td>32.38</td>
      <td>Vins et alcools Chevalier</td>
      <td>59 rue de l'Abbaye</td>
      <td>Reims</td>
      <td>Western Europe</td>
      <td>51100</td>
      <td>France</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10249</td>
      <td>TOMSP</td>
      <td>6</td>
      <td>2012-07-05</td>
      <td>2012-08-16</td>
      <td>2012-07-10</td>
      <td>1</td>
      <td>11.61</td>
      <td>Toms Spezialitäten</td>
      <td>Luisenstr. 48</td>
      <td>Münster</td>
      <td>Western Europe</td>
      <td>44087</td>
      <td>Germany</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10250</td>
      <td>HANAR</td>
      <td>4</td>
      <td>2012-07-08</td>
      <td>2012-08-05</td>
      <td>2012-07-12</td>
      <td>2</td>
      <td>65.83</td>
      <td>Hanari Carnes</td>
      <td>Rua do Paço, 67</td>
      <td>Rio de Janeiro</td>
      <td>South America</td>
      <td>05454-876</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10251</td>
      <td>VICTE</td>
      <td>3</td>
      <td>2012-07-08</td>
      <td>2012-08-05</td>
      <td>2012-07-15</td>
      <td>1</td>
      <td>41.34</td>
      <td>Victuailles en stock</td>
      <td>2, rue du Commerce</td>
      <td>Lyon</td>
      <td>Western Europe</td>
      <td>69004</td>
      <td>France</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10252</td>
      <td>SUPRD</td>
      <td>4</td>
      <td>2012-07-09</td>
      <td>2012-08-06</td>
      <td>2012-07-11</td>
      <td>2</td>
      <td>51.30</td>
      <td>Suprêmes délices</td>
      <td>Boulevard Tirou, 255</td>
      <td>Charleroi</td>
      <td>Western Europe</td>
      <td>B-6000</td>
      <td>Belgium</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10253</td>
      <td>HANAR</td>
      <td>3</td>
      <td>2012-07-10</td>
      <td>2012-07-24</td>
      <td>2012-07-16</td>
      <td>2</td>
      <td>58.17</td>
      <td>Hanari Carnes</td>
      <td>Rua do Paço, 67</td>
      <td>Rio de Janeiro</td>
      <td>South America</td>
      <td>05454-876</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10254</td>
      <td>CHOPS</td>
      <td>5</td>
      <td>2012-07-11</td>
      <td>2012-08-08</td>
      <td>2012-07-23</td>
      <td>2</td>
      <td>22.98</td>
      <td>Chop-suey Chinese</td>
      <td>Hauptstr. 31</td>
      <td>Bern</td>
      <td>Western Europe</td>
      <td>3012</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10255</td>
      <td>RICSU</td>
      <td>9</td>
      <td>2012-07-12</td>
      <td>2012-08-09</td>
      <td>2012-07-15</td>
      <td>3</td>
      <td>148.33</td>
      <td>Richter Supermarkt</td>
      <td>Starenweg 5</td>
      <td>Genève</td>
      <td>Western Europe</td>
      <td>1204</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <th>8</th>
      <td>10256</td>
      <td>WELLI</td>
      <td>3</td>
      <td>2012-07-15</td>
      <td>2012-08-12</td>
      <td>2012-07-17</td>
      <td>2</td>
      <td>13.97</td>
      <td>Wellington Importadora</td>
      <td>Rua do Mercado, 12</td>
      <td>Resende</td>
      <td>South America</td>
      <td>08737-363</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10257</td>
      <td>HILAA</td>
      <td>4</td>
      <td>2012-07-16</td>
      <td>2012-08-13</td>
      <td>2012-07-22</td>
      <td>3</td>
      <td>81.91</td>
      <td>HILARION-Abastos</td>
      <td>Carrera 22 con Ave. Carlos Soublette #8-35</td>
      <td>San Cristóbal</td>
      <td>South America</td>
      <td>5022</td>
      <td>Venezuela</td>
    </tr>
  </tbody>
</table>
</div>




```python
OrderDetail_df.head()
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
      <th>Id</th>
      <th>OrderId</th>
      <th>ProductId</th>
      <th>UnitPrice</th>
      <th>Quantity</th>
      <th>Discount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10248/11</td>
      <td>10248</td>
      <td>11</td>
      <td>14.0</td>
      <td>12</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10248/42</td>
      <td>10248</td>
      <td>42</td>
      <td>9.8</td>
      <td>10</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10248/72</td>
      <td>10248</td>
      <td>72</td>
      <td>34.8</td>
      <td>5</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10249/14</td>
      <td>10249</td>
      <td>14</td>
      <td>18.6</td>
      <td>9</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10249/51</td>
      <td>10249</td>
      <td>51</td>
      <td>42.4</td>
      <td>40</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
OrderDetail_df['Discount'].shape
```




    (2155,)



## Let's take a better look at what we are selling


```python
Product_df.head(30) # change to 77 to see all product descriptions
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
      <th>Id</th>
      <th>ProductName</th>
      <th>SupplierId</th>
      <th>CategoryId</th>
      <th>QuantityPerUnit</th>
      <th>UnitPrice</th>
      <th>UnitsInStock</th>
      <th>UnitsOnOrder</th>
      <th>ReorderLevel</th>
      <th>Discontinued</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Chai</td>
      <td>1</td>
      <td>1</td>
      <td>10 boxes x 20 bags</td>
      <td>18.00</td>
      <td>39</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Chang</td>
      <td>1</td>
      <td>1</td>
      <td>24 - 12 oz bottles</td>
      <td>19.00</td>
      <td>17</td>
      <td>40</td>
      <td>25</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Aniseed Syrup</td>
      <td>1</td>
      <td>2</td>
      <td>12 - 550 ml bottles</td>
      <td>10.00</td>
      <td>13</td>
      <td>70</td>
      <td>25</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Chef Anton's Cajun Seasoning</td>
      <td>2</td>
      <td>2</td>
      <td>48 - 6 oz jars</td>
      <td>22.00</td>
      <td>53</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Chef Anton's Gumbo Mix</td>
      <td>2</td>
      <td>2</td>
      <td>36 boxes</td>
      <td>21.35</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Grandma's Boysenberry Spread</td>
      <td>3</td>
      <td>2</td>
      <td>12 - 8 oz jars</td>
      <td>25.00</td>
      <td>120</td>
      <td>0</td>
      <td>25</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>3</td>
      <td>7</td>
      <td>12 - 1 lb pkgs.</td>
      <td>30.00</td>
      <td>15</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Northwoods Cranberry Sauce</td>
      <td>3</td>
      <td>2</td>
      <td>12 - 12 oz jars</td>
      <td>40.00</td>
      <td>6</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>Mishi Kobe Niku</td>
      <td>4</td>
      <td>6</td>
      <td>18 - 500 g pkgs.</td>
      <td>97.00</td>
      <td>29</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Ikura</td>
      <td>4</td>
      <td>8</td>
      <td>12 - 200 ml jars</td>
      <td>31.00</td>
      <td>31</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Queso Cabrales</td>
      <td>5</td>
      <td>4</td>
      <td>1 kg pkg.</td>
      <td>21.00</td>
      <td>22</td>
      <td>30</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Queso Manchego La Pastora</td>
      <td>5</td>
      <td>4</td>
      <td>10 - 500 g pkgs.</td>
      <td>38.00</td>
      <td>86</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>Konbu</td>
      <td>6</td>
      <td>8</td>
      <td>2 kg box</td>
      <td>6.00</td>
      <td>24</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>Tofu</td>
      <td>6</td>
      <td>7</td>
      <td>40 - 100 g pkgs.</td>
      <td>23.25</td>
      <td>35</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>Genen Shouyu</td>
      <td>6</td>
      <td>2</td>
      <td>24 - 250 ml bottles</td>
      <td>15.50</td>
      <td>39</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>Pavlova</td>
      <td>7</td>
      <td>3</td>
      <td>32 - 500 g boxes</td>
      <td>17.45</td>
      <td>29</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>Alice Mutton</td>
      <td>7</td>
      <td>6</td>
      <td>20 - 1 kg tins</td>
      <td>39.00</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>18</td>
      <td>Carnarvon Tigers</td>
      <td>7</td>
      <td>8</td>
      <td>16 kg pkg.</td>
      <td>62.50</td>
      <td>42</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>Teatime Chocolate Biscuits</td>
      <td>8</td>
      <td>3</td>
      <td>10 boxes x 12 pieces</td>
      <td>9.20</td>
      <td>25</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>Sir Rodney's Marmalade</td>
      <td>8</td>
      <td>3</td>
      <td>30 gift boxes</td>
      <td>81.00</td>
      <td>40</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>Sir Rodney's Scones</td>
      <td>8</td>
      <td>3</td>
      <td>24 pkgs. x 4 pieces</td>
      <td>10.00</td>
      <td>3</td>
      <td>40</td>
      <td>5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22</td>
      <td>Gustaf's Knäckebröd</td>
      <td>9</td>
      <td>5</td>
      <td>24 - 500 g pkgs.</td>
      <td>21.00</td>
      <td>104</td>
      <td>0</td>
      <td>25</td>
      <td>0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>23</td>
      <td>Tunnbröd</td>
      <td>9</td>
      <td>5</td>
      <td>12 - 250 g pkgs.</td>
      <td>9.00</td>
      <td>61</td>
      <td>0</td>
      <td>25</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>24</td>
      <td>Guaraná Fantástica</td>
      <td>10</td>
      <td>1</td>
      <td>12 - 355 ml cans</td>
      <td>4.50</td>
      <td>20</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>24</th>
      <td>25</td>
      <td>NuNuCa Nuß-Nougat-Creme</td>
      <td>11</td>
      <td>3</td>
      <td>20 - 450 g glasses</td>
      <td>14.00</td>
      <td>76</td>
      <td>0</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>26</td>
      <td>Gumbär Gummibärchen</td>
      <td>11</td>
      <td>3</td>
      <td>100 - 250 g bags</td>
      <td>31.23</td>
      <td>15</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>27</td>
      <td>Schoggi Schokolade</td>
      <td>11</td>
      <td>3</td>
      <td>100 - 100 g pieces</td>
      <td>43.90</td>
      <td>49</td>
      <td>0</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>28</td>
      <td>Rössle Sauerkraut</td>
      <td>12</td>
      <td>7</td>
      <td>25 - 825 g cans</td>
      <td>45.60</td>
      <td>26</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>28</th>
      <td>29</td>
      <td>Thüringer Rostbratwurst</td>
      <td>12</td>
      <td>6</td>
      <td>50 bags x 30 sausgs.</td>
      <td>123.79</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>29</th>
      <td>30</td>
      <td>Nord-Ost Matjeshering</td>
      <td>13</td>
      <td>8</td>
      <td>10 - 200 g glasses</td>
      <td>25.89</td>
      <td>10</td>
      <td>0</td>
      <td>15</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(Product_df)
```




    77




```python
# creating 2 populations to work with 
discount = OrderDetail_df[OrderDetail_df['Discount']!=0].groupby('ProductId')['Quantity'].mean()
no_discount = OrderDetail_df[OrderDetail_df['Discount']==0].groupby('ProductId')['Quantity'].mean()
```

##  Discounted and not discounted products by order quantity

Table `Product` has 77 entries, each entry is unique product

First we can check visually if discount really made a difference in order quantity


```python
def make_dist(arr, bins=40, color='b', alpha=.6):
    plt.figure(figsize=(8,6))
    plt.grid(zorder=0)
    # zorder pushes things in and out of the frame in front of grid
    plt.hist(arr, bins=bins,color=color, alpha=alpha, zorder=3)
    plt.show()
```


```python
# helps to see distributions
make_dist(discount)
make_dist(no_discount)
```


![png](student_files/student_29_0.png)



![png](student_files/student_29_1.png)



```python
print(discount.mean())
print(no_discount.mean())
```

    26.43253285866255
    21.81167852821319



```python
# ratio of standard deviation .8 - 1.2 is where you want to be, we are pretty close
np.std(no_discount)/np.std(discount)
```




    0.7851309352711258




```python
# shapiro test, tests for normality of populations
for sv in [no_discount, discount]:
    w, p = scs.shapiro(sv)
    print("p = {}".format(p))
```

    p = 0.0015117175644263625
    p = 0.7060348391532898


## Orders by discount level

let us see how many discount levels we have, and how many orders we average per level of discount.


```python
# Let's get all discount levels
discounts = OrderDetail_df['Discount'].unique()
discounts.sort()
print('Discount levels')
print(discounts)
```

    Discount levels
    [0.   0.01 0.02 0.03 0.04 0.05 0.06 0.1  0.15 0.2  0.25]


These are the different levels of discount that where recorded in the data, 0.04 is = to a 4% discount.


```python
# Group orders by discount amounts
# Each group is a DataFrame containing orders with certain discount level
groups = {}
for i in discounts:
    groups[i] = OrderDetail_df[OrderDetail_df['Discount']==i]
```


```python
# Create new DataFrame with Discounts and Order quantities
discounts_df = pd.DataFrame(columns=['Discount %','Orders','Avg. Order Quantity'])
for i in groups.keys():
    discounts_df = discounts_df.append({'Discount %':i*100,'Orders':len(groups[i]),'Avg. Order Quantity':groups[i]['Quantity'].mean()}, ignore_index=True)

discounts_df
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
      <th>Discount %</th>
      <th>Orders</th>
      <th>Avg. Order Quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>1317.0</td>
      <td>21.715262</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.0</td>
      <td>3.0</td>
      <td>1.666667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4.0</td>
      <td>1.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5.0</td>
      <td>185.0</td>
      <td>28.010811</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6.0</td>
      <td>1.0</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10.0</td>
      <td>173.0</td>
      <td>25.236994</td>
    </tr>
    <tr>
      <th>8</th>
      <td>15.0</td>
      <td>157.0</td>
      <td>28.382166</td>
    </tr>
    <tr>
      <th>9</th>
      <td>20.0</td>
      <td>161.0</td>
      <td>27.024845</td>
    </tr>
    <tr>
      <th>10</th>
      <td>25.0</td>
      <td>154.0</td>
      <td>28.240260</td>
    </tr>
  </tbody>
</table>
</div>



The table above shows us the average order quantity per level of discount. Two things stand out immediately. First, and most notable is the discount quantities of 1%, 2%, 3%, 4%, and 6% (respectively) all hardly have any sales, and as a result, probably wont be able to help provide much more information. it would probably be safe to drop those discount levels. The second thing I notice, though not as eye grabbing, is the relativley small amount of varience between the rest of the discount %'s(namely:5%,10%,15%,20%,25%), and their corresponding order quantity.


```python
plt.figure(figsize=(16,5))
plt.bar(discount.index, discount.values, alpha=.5, label='Discount', color='b')
plt.bar(no_discount.index, no_discount.values, alpha=0.5, label='No Discount', color='k')
plt.legend()
plt.title('Order quantities with/without discount')
plt.xlabel('Product ID')
plt.ylabel('Average quantities')
plt.show()

print('Conclusion')
print("On average {}% of discounted products were sold in larger quantities".format(round(sum(discount.values > no_discount.values)/len(discount.values)*100),2))
print("Average order quantity with Discount - {} items, without Discount - {} items".format(round(discount.values.mean(),2), round(no_discount.values.mean(),2)))
```


![png](student_files/student_39_0.png)


    Conclusion
    On average 70.0% of discounted products were sold in larger quantities
    Average order quantity with Discount - 26.43 items, without Discount - 21.81 items


At an initial glance, we can see it appears that "Discounted" product on average, has larger quantities of product sold. To prove this, let's run a quick experiment to be certain.

## Visualization


```python
def visualization(control, experimental):
    plt.figure(figsize=(12,8))
    sns.distplot(experimental, bins=80, color=None,  label='Experimental')
    sns.distplot(control, bins=80, color=None,  label='Control')

    plt.axvline(x=control.mean(), color='k', linestyle='--')
    plt.axvline(x=experimental.mean(), color='g', linestyle='--')

    plt.title('Control and Experimental Sampling Distributions', fontsize=12)
    plt.xlabel('Distributions')
    plt.ylabel('Frequency')
    plt.legend()
    plt.show()
```

## Cohen's d

Cohen's d is an effect size used to indicate the standardised difference between two means. It can be used, for example, to accompany reporting of t-test and ANOVA results. It is also widely used in meta-analysis. Cohen's d is an appropriate effect size for the comparison between two means.


```python
def Cohen_d(group1, group2):

    diff = group1.mean() - group2.mean()
    n1, n2 = len(group1), len(group2)
    var1 = group1.var()
    var2 = group2.var()
    # Calculate the pooled threshold as shown earlier
    pooled_var = (n1 * var1 + n2 * var2) / (n1 + n2)
    # Calculate Cohen's d statistic
    d = diff / np.sqrt(pooled_var)
    return abs(d)
```

## Bootstrap

Bootstrapping is a type of resampling where large numbers of smaller samples of the same size are repeatedly drawn, with replacement, from a single original sample.


```python
def bootstrap(sample, n):
    bootstrap_sampling_dist = []
    for i in range(n):
        bootstrap_sampling_dist.append(np.random.choice(sample, size=len(sample), replace=True).mean())
    return np.array(bootstrap_sampling_dist)
```

# Question#1

## Does  applying a discount have a statistically significant effect on the quantity of a product in an order? If so, at what level(s) of discount?

- $H_0$: there is no difference in order quantity due to discount
- $H_\alpha$: there is a difference in order quantity due to discount

Usually discount increases order quantity, so it would be reasonable to perform one-tailed test with $\alpha$ set to 0.025. If $p$ < $\alpha$, we reject null hypothesis.

## Welch's T-test

In statistics, Welch's t-test, or unequal variances t-test, is a two-sample location test which is used to test the hypothesis that two populations have equal means.

I created two distributions (control and experimental). Control distribution includes only order quantities without discounts, and the experimental distribution includes order quantities with discounts (at any level)

This experiment would answer a question if there is any difference in purchase quantity


```python
control = OrderDetail_df[OrderDetail_df['Discount']==0]['Quantity']
experimental = OrderDetail_df[OrderDetail_df['Discount']!=0]['Quantity']
```


```python
make_dist(control)
make_dist(experimental)
```


![png](student_files/student_52_0.png)



![png](student_files/student_52_1.png)


2 tailed test


```python
t_stat, p = stats.ttest_ind(control, experimental)
d = Cohen_d(experimental, control)

print('Reject Null Hypothesis') if p < 0.025 else print('Failed to reject Null Hypothesis')
print("Cohen's d:", d)
visualization(control, experimental)
```

    Reject Null Hypothesis
    Cohen's d: 0.2862724481729283



![png](student_files/student_54_1.png)


### Result

This shows us that there is in fact a _**statistically significant**_ difference in the amount of orders, therefore we can reject the null hypothesis with confidence. The question was posed in such a way that it asks if order quantity is different at different discount levels. The next step (part 2 of first question) is to find out specifically at what discount level does the change in order quantities become _**statisticaly significant**_. 
To show this, we will run the previous experiment again, but we will make distinctions by grouping discount amounts.


```python
discounts_significance_df = pd.DataFrame(columns=['Discount %','Null Hypothesis','Cohens d'], index=None)

discounts = [0.05, 0.1, 0.15, 0.2, 0.25]
control = OrderDetail_df[OrderDetail_df['Discount']==0]['Quantity']
for i in discounts:
    experimental = OrderDetail_df[OrderDetail_df['Discount']==i]['Quantity']
    st, p = stats.ttest_ind(control, experimental)
    d = Cohen_d(experimental, control)
    discounts_significance_df = discounts_significance_df.append( { 'Discount %' : str(i*100)+'%' , 'Null Hypothesis' : 'Reject' if p < 0.025 else 'Failed', 'Cohens d' : d } , ignore_index=True)    

discounts_significance_df
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
      <th>Discount %</th>
      <th>Null Hypothesis</th>
      <th>Cohens d</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.0%</td>
      <td>Reject</td>
      <td>0.346877</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10.0%</td>
      <td>Reject</td>
      <td>0.195942</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15.0%</td>
      <td>Reject</td>
      <td>0.372404</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20.0%</td>
      <td>Reject</td>
      <td>0.300712</td>
    </tr>
    <tr>
      <th>4</th>
      <td>25.0%</td>
      <td>Reject</td>
      <td>0.366593</td>
    </tr>
  </tbody>
</table>
</div>



These results show us that the remaining discount percentages (5%, 10%, 15%, 20%, 25%) less the ones we dropped earlier (1%,2%,3%,4%, and 6%), show a _**statistically significant**_ difference in order quantities then those having no discount.



Something is still bothering me though.... we now know that of the discount percentages we encountered, 5%, 10%, 15%, 20%, 25% all showed a _**statistically significant**_ difference in order quantities from 0 or no discounts (less the few "odd ball" discounts that didnt have enough order quantities to validate working with them). which is great, and we were able to validate with an experiment, even better. However, as I had mentioned earlier from a simple observation early on, there was not a lot of discernable difference (at a glance) in order quantities between discounts 5%, 10%, 15%, 20%, and 25%. That stands out as strange to me. Since we were able to conclude that applying a discount does in fact have a positive correlation to quantities sold, the findings lead me to the second question I want to ask.  

# Question#2

## Do larger discount amounts affect the sales quantity more than lower discount amounts?


- $H_0$: there is no difference in order quantity between discount rates.
- $H_\alpha$: there is a difference in order quantity between discount rates.

1 tailed test


```python
discounts = np.array([0.05, 0.1, 0.15, 0.2, 0.25])
# itertools.combinations works through every single combination without repeating a combination 
comb = itertools.combinations(discounts,2 )
discount_levels_df = pd.DataFrame(columns=['Discount %','Null Hypothesis','Cohens d'], index=None)

for i in comb:
    
    control =      OrderDetail_df[OrderDetail_df['Discount']==i[0]]['Quantity']
    experimental = OrderDetail_df[OrderDetail_df['Discount']==i[1]]['Quantity']
    
    st, p = stats.ttest_ind(experimental, control)
    d = Cohen_d(experimental, control)
    
    discount_levels_df = discount_levels_df.append( { 'Discount %' : str(i[0]*100)+'% - '+str(i[1]*100)+'%', 'Null Hypothesis' : 'Reject' if p < 0.05 else 'Failed', 'Cohens d' : d } , ignore_index=True)    

discount_levels_df.sort_values('Cohens d', ascending=False)
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
      <th>Discount %</th>
      <th>Null Hypothesis</th>
      <th>Cohens d</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>10.0% - 15.0%</td>
      <td>Failed</td>
      <td>0.149332</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10.0% - 25.0%</td>
      <td>Failed</td>
      <td>0.145146</td>
    </tr>
    <tr>
      <th>0</th>
      <td>5.0% - 10.0%</td>
      <td>Failed</td>
      <td>0.127769</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10.0% - 20.0%</td>
      <td>Failed</td>
      <td>0.089008</td>
    </tr>
    <tr>
      <th>7</th>
      <td>15.0% - 20.0%</td>
      <td>Failed</td>
      <td>0.068234</td>
    </tr>
    <tr>
      <th>9</th>
      <td>20.0% - 25.0%</td>
      <td>Failed</td>
      <td>0.062415</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.0% - 20.0%</td>
      <td>Failed</td>
      <td>0.047644</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5.0% - 15.0%</td>
      <td>Failed</td>
      <td>0.017179</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5.0% - 25.0%</td>
      <td>Failed</td>
      <td>0.010786</td>
    </tr>
    <tr>
      <th>8</th>
      <td>15.0% - 25.0%</td>
      <td>Failed</td>
      <td>0.006912</td>
    </tr>
  </tbody>
</table>
</div>



### Result

Result of the test shows that there is no _**statistically significant**_ difference in order quantity between discounts of 5%, 10%, 15%, 20% and 25%. This means we failed to reject the null hypothesis. This is not surprising based off our initial results, but I still find it very interesting information to know, and usefull. If you have stock that needs to be unloaded quickly, a 5% discount is likely to get the product out of the door just as fast as a 25% discount, from the data gathered thus far. The results could be used to further investigate and maybe test the effect of different discount levels as potential revenue boosters.

# Question#3
## Is there a statistically significant difference in the output of shipping companies?

- $H_0$: There is no difference in level of performance between categories
- $H_\alpha$: There is a difference in level of performance between categories


```python
Order_df.head(15)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>EmployeeId</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>ShipVia</th>
      <th>Freight</th>
      <th>ShipName</th>
      <th>ShipAddress</th>
      <th>ShipCity</th>
      <th>ShipRegion</th>
      <th>ShipPostalCode</th>
      <th>ShipCountry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10248</td>
      <td>VINET</td>
      <td>5</td>
      <td>2012-07-04</td>
      <td>2012-08-01</td>
      <td>2012-07-16</td>
      <td>3</td>
      <td>32.38</td>
      <td>Vins et alcools Chevalier</td>
      <td>59 rue de l'Abbaye</td>
      <td>Reims</td>
      <td>Western Europe</td>
      <td>51100</td>
      <td>France</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10249</td>
      <td>TOMSP</td>
      <td>6</td>
      <td>2012-07-05</td>
      <td>2012-08-16</td>
      <td>2012-07-10</td>
      <td>1</td>
      <td>11.61</td>
      <td>Toms Spezialitäten</td>
      <td>Luisenstr. 48</td>
      <td>Münster</td>
      <td>Western Europe</td>
      <td>44087</td>
      <td>Germany</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10250</td>
      <td>HANAR</td>
      <td>4</td>
      <td>2012-07-08</td>
      <td>2012-08-05</td>
      <td>2012-07-12</td>
      <td>2</td>
      <td>65.83</td>
      <td>Hanari Carnes</td>
      <td>Rua do Paço, 67</td>
      <td>Rio de Janeiro</td>
      <td>South America</td>
      <td>05454-876</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10251</td>
      <td>VICTE</td>
      <td>3</td>
      <td>2012-07-08</td>
      <td>2012-08-05</td>
      <td>2012-07-15</td>
      <td>1</td>
      <td>41.34</td>
      <td>Victuailles en stock</td>
      <td>2, rue du Commerce</td>
      <td>Lyon</td>
      <td>Western Europe</td>
      <td>69004</td>
      <td>France</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10252</td>
      <td>SUPRD</td>
      <td>4</td>
      <td>2012-07-09</td>
      <td>2012-08-06</td>
      <td>2012-07-11</td>
      <td>2</td>
      <td>51.30</td>
      <td>Suprêmes délices</td>
      <td>Boulevard Tirou, 255</td>
      <td>Charleroi</td>
      <td>Western Europe</td>
      <td>B-6000</td>
      <td>Belgium</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10253</td>
      <td>HANAR</td>
      <td>3</td>
      <td>2012-07-10</td>
      <td>2012-07-24</td>
      <td>2012-07-16</td>
      <td>2</td>
      <td>58.17</td>
      <td>Hanari Carnes</td>
      <td>Rua do Paço, 67</td>
      <td>Rio de Janeiro</td>
      <td>South America</td>
      <td>05454-876</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10254</td>
      <td>CHOPS</td>
      <td>5</td>
      <td>2012-07-11</td>
      <td>2012-08-08</td>
      <td>2012-07-23</td>
      <td>2</td>
      <td>22.98</td>
      <td>Chop-suey Chinese</td>
      <td>Hauptstr. 31</td>
      <td>Bern</td>
      <td>Western Europe</td>
      <td>3012</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10255</td>
      <td>RICSU</td>
      <td>9</td>
      <td>2012-07-12</td>
      <td>2012-08-09</td>
      <td>2012-07-15</td>
      <td>3</td>
      <td>148.33</td>
      <td>Richter Supermarkt</td>
      <td>Starenweg 5</td>
      <td>Genève</td>
      <td>Western Europe</td>
      <td>1204</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <th>8</th>
      <td>10256</td>
      <td>WELLI</td>
      <td>3</td>
      <td>2012-07-15</td>
      <td>2012-08-12</td>
      <td>2012-07-17</td>
      <td>2</td>
      <td>13.97</td>
      <td>Wellington Importadora</td>
      <td>Rua do Mercado, 12</td>
      <td>Resende</td>
      <td>South America</td>
      <td>08737-363</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10257</td>
      <td>HILAA</td>
      <td>4</td>
      <td>2012-07-16</td>
      <td>2012-08-13</td>
      <td>2012-07-22</td>
      <td>3</td>
      <td>81.91</td>
      <td>HILARION-Abastos</td>
      <td>Carrera 22 con Ave. Carlos Soublette #8-35</td>
      <td>San Cristóbal</td>
      <td>South America</td>
      <td>5022</td>
      <td>Venezuela</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10258</td>
      <td>ERNSH</td>
      <td>1</td>
      <td>2012-07-17</td>
      <td>2012-08-14</td>
      <td>2012-07-23</td>
      <td>1</td>
      <td>140.51</td>
      <td>Ernst Handel</td>
      <td>Kirchgasse 6</td>
      <td>Graz</td>
      <td>Western Europe</td>
      <td>8010</td>
      <td>Austria</td>
    </tr>
    <tr>
      <th>11</th>
      <td>10259</td>
      <td>CENTC</td>
      <td>4</td>
      <td>2012-07-18</td>
      <td>2012-08-15</td>
      <td>2012-07-25</td>
      <td>3</td>
      <td>3.25</td>
      <td>Centro comercial Moctezuma</td>
      <td>Sierras de Granada 9993</td>
      <td>México D.F.</td>
      <td>Central America</td>
      <td>05022</td>
      <td>Mexico</td>
    </tr>
    <tr>
      <th>12</th>
      <td>10260</td>
      <td>OTTIK</td>
      <td>4</td>
      <td>2012-07-19</td>
      <td>2012-08-16</td>
      <td>2012-07-29</td>
      <td>1</td>
      <td>55.09</td>
      <td>Ottilies Käseladen</td>
      <td>Mehrheimerstr. 369</td>
      <td>Köln</td>
      <td>Western Europe</td>
      <td>50739</td>
      <td>Germany</td>
    </tr>
    <tr>
      <th>13</th>
      <td>10261</td>
      <td>QUEDE</td>
      <td>4</td>
      <td>2012-07-19</td>
      <td>2012-08-16</td>
      <td>2012-07-30</td>
      <td>2</td>
      <td>3.05</td>
      <td>Que Delícia</td>
      <td>Rua da Panificadora, 12</td>
      <td>Rio de Janeiro</td>
      <td>South America</td>
      <td>02389-673</td>
      <td>Brazil</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10262</td>
      <td>RATTC</td>
      <td>8</td>
      <td>2012-07-22</td>
      <td>2012-08-19</td>
      <td>2012-07-25</td>
      <td>3</td>
      <td>48.29</td>
      <td>Rattlesnake Canyon Grocery</td>
      <td>2817 Milton Dr.</td>
      <td>Albuquerque</td>
      <td>North America</td>
      <td>87110</td>
      <td>USA</td>
    </tr>
  </tbody>
</table>
</div>




```python
Order_df['ShipRegion'].head()
```




    0    Western Europe
    1    Western Europe
    2     South America
    3    Western Europe
    4    Western Europe
    Name: ShipRegion, dtype: object




```python
Order_df['ShipRegion'].unique()
```




    array(['Western Europe', 'South America', 'Central America',
           'North America', 'Northern Europe', 'Scandinavia',
           'Southern Europe', 'British Isles', 'Eastern Europe'], dtype=object)




```python
Shipper_df
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
      <th>Id</th>
      <th>CompanyName</th>
      <th>Phone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Speedy Express</td>
      <td>(503) 555-9831</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>United Package</td>
      <td>(503) 555-3199</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Federal Shipping</td>
      <td>(503) 555-9931</td>
    </tr>
  </tbody>
</table>
</div>




```python
Order_df.OrderDate = pd.to_datetime(Order_df.OrderDate)
Order_df.ShippedDate = pd.to_datetime(Order_df.ShippedDate)
Order_df.RequiredDate = pd.to_datetime(Order_df.RequiredDate)

Order_df['ProcessingTime'] = Order_df.ShippedDate - Order_df.OrderDate
Order_df['ShippingTime'] = Order_df.RequiredDate - Order_df.ShippedDate

Order_df.ShippingTime = Order_df.ShippingTime.dt.days
Order_df.ProcessingTime = Order_df.ProcessingTime.dt.days
```


```python
make_dist(Order_df.ShippingTime)
```


![png](student_files/student_73_0.png)



```python
make_dist(Order_df.ProcessingTime)
```


![png](student_files/student_74_0.png)



```python
Order_df['ShippingTime'].head(15)
```




    0     16.0
    1     37.0
    2     24.0
    3     21.0
    4     26.0
    5      8.0
    6     16.0
    7     25.0
    8     26.0
    9     22.0
    10    22.0
    11    21.0
    12    18.0
    13    17.0
    14    25.0
    Name: ShippingTime, dtype: float64




```python
Order_df['ProcessingTime'].head(15)
```




    0     12.0
    1      5.0
    2      4.0
    3      7.0
    4      2.0
    5      6.0
    6     12.0
    7      3.0
    8      2.0
    9      6.0
    10     6.0
    11     7.0
    12    10.0
    13    11.0
    14     3.0
    Name: ProcessingTime, dtype: float64




```python
Order_df.ProcessingTime.head(15)
```




    0     12.0
    1      5.0
    2      4.0
    3      7.0
    4      2.0
    5      6.0
    6     12.0
    7      3.0
    8      2.0
    9      6.0
    10     6.0
    11     7.0
    12    10.0
    13    11.0
    14     3.0
    Name: ProcessingTime, dtype: float64




```python
Order_df.ProcessingTime.mean()
```




    8.491965389369591




```python
Order_df.groupby('ShipVia').mean()
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
      <th>Id</th>
      <th>EmployeeId</th>
      <th>Freight</th>
      <th>ProcessingTime</th>
      <th>ShippingTime</th>
    </tr>
    <tr>
      <th>ShipVia</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>10667.594378</td>
      <td>4.232932</td>
      <td>65.001325</td>
      <td>8.571429</td>
      <td>19.485714</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10674.963190</td>
      <td>4.536810</td>
      <td>86.640644</td>
      <td>9.234921</td>
      <td>18.765079</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10641.592157</td>
      <td>4.400000</td>
      <td>80.441216</td>
      <td>7.473896</td>
      <td>19.963855</td>
    </tr>
  </tbody>
</table>
</div>



### ANOVA Test

Analysis of variance (ANOVA) is a statistical technique that is used to check if the means of two or more groups are significantly different from each other. ... When we have only two samples, t-test and ANOVA give the same results. However, using a t-test would not be reliable in cases where there are more than 2 samples


```python
formula = 'ProcessingTime ~ C(ShipVia)'
lm = ols(formula, Order_df).fit()
table = sm.stats.anova_lm(lm, typ=2)
print(table)
```

                      sum_sq     df         F    PR(>F)
    C(ShipVia)    433.501581    2.0  4.676819  0.009563
    Residual    37354.696194  806.0       NaN       NaN



```python
lm.summary()
```




<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>     <td>ProcessingTime</td>  <th>  R-squared:         </th> <td>   0.011</td>
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>       <th>  Adj. R-squared:    </th> <td>   0.009</td>
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>  <th>  F-statistic:       </th> <td>   4.677</td>
</tr>
<tr>
  <th>Date:</th>             <td>Sun, 16 Jun 2019</td> <th>  Prob (F-statistic):</th>  <td>0.00956</td>
</tr>
<tr>
  <th>Time:</th>                 <td>10:52:43</td>     <th>  Log-Likelihood:    </th> <td> -2698.1</td>
</tr>
<tr>
  <th>No. Observations:</th>      <td>   809</td>      <th>  AIC:               </th> <td>   5402.</td>
</tr>
<tr>
  <th>Df Residuals:</th>          <td>   806</td>      <th>  BIC:               </th> <td>   5416.</td>
</tr>
<tr>
  <th>Df Model:</th>              <td>     2</td>      <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>      <td>nonrobust</td>    <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
         <td></td>            <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>Intercept</th>       <td>    8.5714</td> <td>    0.435</td> <td>   19.707</td> <td> 0.000</td> <td>    7.718</td> <td>    9.425</td>
</tr>
<tr>
  <th>C(ShipVia)[T.2]</th> <td>    0.6635</td> <td>    0.580</td> <td>    1.144</td> <td> 0.253</td> <td>   -0.475</td> <td>    1.802</td>
</tr>
<tr>
  <th>C(ShipVia)[T.3]</th> <td>   -1.0975</td> <td>    0.613</td> <td>   -1.792</td> <td> 0.074</td> <td>   -2.300</td> <td>    0.105</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>338.799</td> <th>  Durbin-Watson:     </th> <td>   1.897</td> 
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td>  <th>  Jarque-Bera (JB):  </th> <td>1191.616</td> 
</tr>
<tr>
  <th>Skew:</th>          <td> 2.055</td>  <th>  Prob(JB):          </th> <td>1.75e-259</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 7.296</td>  <th>  Cond. No.          </th> <td>    3.91</td> 
</tr>
</table><br/><br/>Warnings:<br/>[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.



### Result

Result of the test shows that there is a _**statistically significant**_ difference in performance of shipping companies, hence we reject null hypothesis

# Question#4
## Are there any times in the year where demand is higher than others, or lower?

- $H_0$: There is no difference in demand of produce each month
- $H_\alpha$: There is a difference in demand of produce each month

### Read Database


```python
produce = pd.read_sql_query('''

                                SELECT O.OrderDate, OD.Quantity, OD.Discount, CategoryId FROM [Order] AS O
                                JOIN OrderDetail AS OD
                                ON O.Id = OD.OrderId
                                JOIN Product
                                ON Product.Id = OD.ProductId
                                WHERE Product.CategoryId = 7

''',conn)   
```


```python
produce.head(15)
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
      <th>OrderDate</th>
      <th>Quantity</th>
      <th>Discount</th>
      <th>CategoryId</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2012-07-05</td>
      <td>9</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2012-07-05</td>
      <td>40</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2012-07-08</td>
      <td>35</td>
      <td>0.15</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2012-07-11</td>
      <td>21</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2012-07-22</td>
      <td>15</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2012-07-23</td>
      <td>36</td>
      <td>0.25</td>
      <td>7</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2012-08-09</td>
      <td>20</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012-08-27</td>
      <td>2</td>
      <td>0.10</td>
      <td>7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2012-09-10</td>
      <td>28</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-09-23</td>
      <td>4</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2012-10-02</td>
      <td>14</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2012-10-09</td>
      <td>9</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2012-10-18</td>
      <td>10</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2012-10-22</td>
      <td>48</td>
      <td>0.20</td>
      <td>7</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2012-11-22</td>
      <td>30</td>
      <td>0.00</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>



### Group by month


```python
produce.OrderDate = pd.to_datetime(produce.OrderDate)
produce['Month'] = produce.OrderDate.dt.month
```


```python
produce['Month'].head()
```




    0    7
    1    7
    2    7
    3    7
    4    7
    Name: Month, dtype: int64




```python
produce.groupby('Month').mean()
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
      <th>Quantity</th>
      <th>Discount</th>
      <th>CategoryId</th>
    </tr>
    <tr>
      <th>Month</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>16.545455</td>
      <td>0.050000</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15.555556</td>
      <td>0.011111</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21.500000</td>
      <td>0.004545</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29.105263</td>
      <td>0.028947</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>12.888889</td>
      <td>0.075556</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>21.285714</td>
      <td>0.085714</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>26.375000</td>
      <td>0.050000</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>15.666667</td>
      <td>0.038889</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>17.500000</td>
      <td>0.025000</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>33.250000</td>
      <td>0.037500</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>16.000000</td>
      <td>0.055556</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>26.842105</td>
      <td>0.100000</td>
      <td>7.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.barplot(produce['Month'],produce['Quantity'])
#plt.figure(figsize=(12,8))
plt.show()
```


![png](student_files/student_93_0.png)


### ANOVA Test


```python
formula = 'Quantity ~ C(Month)'
lm = ols(formula, produce).fit()
table = sm.stats.anova_lm(lm, typ=2)
print(table)
```

                    sum_sq     df         F    PR(>F)
    C(Month)   4834.012843   11.0  1.318794  0.221691
    Residual  41319.957745  124.0       NaN       NaN


### Result

There is a _**statistically significant**_ difference in order quantity between months, hence we reject the null hypothesis

# Conclusion

- There is a statistically significant difference in the order quantity of products that have had a discount applied. it becomes significant at the 5% discount and remains significant up to 25%, less the percentages dropped.
- There is no statistically significant difference in order quantity between discounts of 5%, 10%, 15%, 20% and 25%.
- There is a statistically significant difference in performance of shipping companies.
- Total order quantities vary a statistically significant amount from month to month. with April, July, October, and December being the highest volume months (October is the highest).

# recommendations

- We now have confirmed that using discounts is still a viable option when trying to move larger quantities of goods, or improve the item count in a single order, not earth shattering information, but still good to have confirmed. 
- I do however recommend that from this point forward, we experiment with teh level of discount and gather more data on sales quantities, as well as comparing per sale profit margins of the different discount levels.
- I recommend that we look further into our shipping contracts as well. While we were able to confirm that they do produce different levels of performance, without filling in more of the picture, I can't really make any concrete statements about what these figures mean, other then saying our 3 shipping companies do not perform equally across the board.
- In regards to sale fluctuations month to month, again, this doesn't do much by its self. However, I do feel that this is highlighting missed opportunties. Since we know what months we will see more traffic through our site, we need to make sure we are doing everything we can during these periods of increased customer traffic to promote ourselves and our products for the upcoming months/events....

### Future Steps

- Look further into shipping, namely regions covered, size of area covered, & potential ways to improve logistics.
- Look into tracking individual employee performance levels.
- try to better identify who our customers are, and what makes them buy from us.
- Look into discounts and sales figures from months with higher sales, see if they behave the same as months with lower sales.
- begin experimenting with offering different discount levels in order continue gathering data on what discount levels move the most amount of product out the door, as well as better determining at what level of discount will see diminished returns despite volume of product sold.
