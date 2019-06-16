
## Final Project Module 2: Statistical Analysis and Hypothesis Testing


* Student name: NICK OSELAND
* Student pace: FULL TIME
* Instructor name: RAFAEL CARRASCO

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

## Connecting to database


print(tables)


    ['Employee', 'Category', 'Customer', 'Shipper', 'Supplier', 'Order', 'Product', 'OrderDetail', 'CustomerCustomerDemo', 'CustomerDemographic', 'Region', 'Territory', 'EmployeeTerritory']


## Converting all tables into dataframes

I was doing this one table at a time, until I ran across this amazing bit of code.


python
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

## Let's take a better look at what we are selling


python
Product_df.head(30) # change to 77 to see all product descriptions


##  Discounted and not discounted products by order quantity

Table `Product` has 77 entries, each entry is unique product

First we can check visually if discount really made a difference in order quantity

## Orders by discount level

let us see how many discount levels we have, and how many orders we average per level of discount.
python
# Let's get all discount levels
discounts = OrderDetail_df['Discount'].unique()
discounts.sort()
print('Discount levels')
print(discounts)


    Discount levels
    [0.   0.01 0.02 0.03 0.04 0.05 0.06 0.1  0.15 0.2  0.25]


These are the different levels of discount that where recorded in the data, 0.04 is = to a 4% discount.

## Visualization


## Cohen's d

Cohen's d is an effect size used to indicate the standardised difference between two means. It can be used, for example, to accompany reporting of t-test and ANOVA results. It is also widely used in meta-analysis. Cohen's d is an appropriate effect size for the comparison between two means.

python
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


## Bootstrap

Bootstrapping is a type of resampling where large numbers of smaller samples of the same size are repeatedly drawn, with replacement, from a single original sample.


python
def bootstrap(sample, n):
    bootstrap_sampling_dist = []
    for i in range(n):
        bootstrap_sampling_dist.append(np.random.choice(sample, size=len(sample), replace=True).mean())
    return np.array(bootstrap_sampling_dist)


# Question#1

## Does  applying a discount have a statistically significant effect on the quantity of a product in an order? If so, at what level(s) of discount?

- $H_0$: there is no difference in order quantity due to discount
- $H_\alpha$: there is a difference in order quantity due to discount

Usually discount increases order quantity, so it would be reasonable to perform one-tailed test with $\alpha$ set to 0.025. If $p$ < $\alpha$, we reject null hypothesis.

## Welch's T-test

In statistics, Welch's t-test, or unequal variances t-test, is a two-sample location test which is used to test the hypothesis that two populations have equal means.

I created two distributions (control and experimental). Control distribution includes only order quantities without discounts, and the experimental distribution includes order quantities with discounts (at any level)

This experiment would answer a question if there is any difference in purchase quantity


2 tailed test



### Result

This shows us that there is in fact a _**statistically significant**_ difference in the amount of orders, therefore we can reject the null hypothesis with confidence. The question was posed in such a way that it asks if order quantity is different at different discount levels. The next step (part 2 of first question) is to find out specifically at what discount level does the change in order quantities become _**statisticaly significant**_. 
To show this, we will run the previous experiment again, but we will make distinctions by grouping discount amounts.


These results show us that the remaining discount percentages (5%, 10%, 15%, 20%, 25%) less the ones we dropped earlier (1%,2%,3%,4%, and 6%), show a _**statistically significant**_ difference in order quantities then those having no discount.



Something is still bothering me though.... we now know that of the discount percentages we encountered, 5%, 10%, 15%, 20%, 25% all showed a _**statistically significant**_ difference in order quantities from 0 or no discounts (less the few "odd ball" discounts that didnt have enough order quantities to validate working with them). which is great, and we were able to validate with an experiment, even better. However, as I had mentioned earlier from a simple observation early on, there was not a lot of discernable difference (at a glance) in order quantities between discounts 5%, 10%, 15%, 20%, and 25%. That stands out as strange to me. Since we were able to conclude that applying a discount does in fact have a positive correlation to quantities sold, the findings lead me to the second question I want to ask.  

# Question#2

## Do larger discount amounts affect the sales quantity more than lower discount amounts?


- $H_0$: there is no difference in order quantity between discount rates.
- $H_\alpha$: there is a difference in order quantity between discount rates.

1 tailed test


python
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

### Result

Result of the test shows that there is no _**statistically significant**_ difference in order quantity between discounts of 5%, 10%, 15%, 20% and 25%. This means we failed to reject the null hypothesis. This is not surprising based off our initial results, but I still find it very interesting information to know, and usefull. If you have stock that needs to be unloaded quickly, a 5% discount is likely to get the product out of the door just as fast as a 25% discount, from the data gathered thus far. The results could be used to further investigate and maybe test the effect of different discount levels as potential revenue boosters.

# Question#3
## Is there a statistically significant difference in the output of shipping companies?

- $H_0$: There is no difference in level of performance between categories
- $H_\alpha$: There is a difference in level of performance between categories

python
Order_df['ShipRegion'].head()





    0    Western Europe
    1    Western Europe
    2     South America
    3    Western Europe
    4    Western Europe
    Name: ShipRegion, dtype: object

python
Order_df['ShipRegion'].unique()




    array(['Western Europe', 'South America', 'Central America',
           'North America', 'Northern Europe', 'Scandinavia',
           'Southern Europe', 'British Isles', 'Eastern Europe'], dtype=object)
python
Order_df['ShippingTime'].head(15)





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




python
Order_df['ProcessingTime'].head(15)





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

### ANOVA Test

Analysis of variance (ANOVA) is a statistical technique that is used to check if the means of two or more groups are significantly different from each other. ... When we have only two samples, t-test and ANOVA give the same results. However, using a t-test would not be reliable in cases where there are more than 2 samples


python
formula = 'ProcessingTime ~ C(ShipVia)'
lm = ols(formula, Order_df).fit()
table = sm.stats.anova_lm(lm, typ=2)
print(table)


                      sum_sq     df         F    PR(>F)
    C(ShipVia)    433.501581    2.0  4.676819  0.009563
    Residual    37354.696194  806.0       NaN       NaN



python
lm.summary()

### Result

Result of the test shows that there is a _**statistically significant**_ difference in performance of shipping companies, hence we reject null hypothesis

# Question#4
## Are there any times in the year where demand is higher than others, or lower?

- $H_0$: There is no difference in demand of produce each month
- $H_\alpha$: There is a difference in demand of produce each month

### Read Database


python
produce = pd.read_sql_query('''

                                SELECT O.OrderDate, OD.Quantity, OD.Discount, CategoryId FROM [Order] AS O
                                JOIN OrderDetail AS OD
                                ON O.Id = OD.OrderId
                                JOIN Product
                                ON Product.Id = OD.ProductId
                                WHERE Product.CategoryId = 7

''',conn)   

### Group by month


python
produce.OrderDate = pd.to_datetime(produce.OrderDate)
produce['Month'] = produce.OrderDate.dt.month

### ANOVA Test


python
formula = 'Quantity ~ C(Month)'
lm = ols(formula, produce).fit()
table = sm.stats.anova_lm(lm, typ=2)
print(table)


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
