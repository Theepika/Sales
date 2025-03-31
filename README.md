# Sales


## Analyzing a sales data.

### When I have enrolled  in Google Data Analytics Certificate course , i came across many datasets for practice. One of the dataset was Sales data , where it has a Product table , an inventory table and sales data table.

### I thought of giving it a try for my SQL practice. 


## About the dataset

### I created a dataset name under my project and downloaded the tables that are required for my analzing under the dataset that i have created. I used three tables for my analyzing.
    
### Product table - It has the productid(unique identifier) , product name , supplier and the cost of the product

### Inventory table - Inventory details like product ud, store id its name and address and the quantity available

### Sales data table - salesid , productid, storeid, along with the unit price and the quantity sold with the date



## I used SQL bigquery for my cleaning and analyzing. Here are my analysis.


##  Cleaning

## *Finding the NULL values and duplicates*

       SELECT  *
        FROM `myfirstproject-445620.sales.products_data` 
        where ProductId is null  
        

 Found one null value in store name . Since the store name is missing , I have update the store name according to the storeid

    select * from `myfirstproject-445620.sales.inventory_data` 
    where StoreName is  null

    select * from `myfirstproject-445620.sales.inventory_data` 
    where Storeid = 21791

    UPDATE `myfirstproject-445620.sales.inventory_data` 
    set StoreName='Dollar Tree'
    where StoreId = 21791

 Checking for duplicate ID.

    SELECT  distinct ProductId
    FROM `myfirstproject-445620.sales.products_data` 


##  Analyzing    

## *Finding how many products does a store sell*

  I have Joined two different tables (products and inventory) to fetch the products and the name of the store that supplies those products. 
  Used an aggregate function (COUNT) to count the number of products a particular store is selling.

    select  StoreName,
    count(products.ProductId) as num_of_products,
    from `myfirstproject-445620.sales.inventory_data` as inventory
    join 
    `myfirstproject-445620.sales.products_data` as products
    on inventory.ProductId = products.ProductId
    group by StoreName


## *total number of sales per product id*

  Finding the total number of sales for a product . Used aggregate function to count the salesid, for the total sales
  and grouped it by productid.
    
    select ProductId,
    count(SalesId) as total_sales
    from `myfirstproject-445620.sales.sales_data`
    group by ProductId 
    order by ProductId


## *Finding the Total sales , Total sales for a product and Total sales for a product in a store*


  Used an aggregate function to count the total sales , 
  used window function to part it along with product id and store id to get the total sales for a product in a store.
  By doing this we can find the top and least selling product in a store .

    select StoreId,ProductId,
    count(SalesId) over() as total_sales,
    count(SalesId) over(partition by ProductId) as total_sales_products,
    count(SalesId) over(partition by ProductId , StoreId) as total_sales_products_store,
    from `myfirstproject-445620.sales.sales_data`
    order by count(SalesId) over(partition by ProductId , StoreId) desc
    

  *Find Top selling product in a store*
  

  Used the row_number() and window function to partition the data by store id and productid ,
  Used aggregate function to count the number of salesid for the store , for a product. ordered it in *descending* order .
  Atlast used the above statement in a subquery and used  the filter where rownumber=1 to fetch the number 1 product that is sold mostly in a store 
  

    select *
    from
    (
    select StoreId,ProductId,
    count(SalesId) over() as total_sales,
    count(SalesId) over(partition by ProductId) as total_sales_products,
    count(SalesId) over(partition by ProductId , StoreId) as total_sales_products_store,
    row_number() over(partition by StoreId,ProductId) as rownumber,
    from `myfirstproject-445620.sales.sales_data`
    order by count(SalesId) over(partition by ProductId , StoreId) desc )
    where rownumber=1
    

*Find Least selling product in a store*


  Used the row_number() and window function to partition the data by store id and productid ,
  Used aggregate function to count the number of salesid for the store , for a product. ordered it in *ASC* order .
  Atlast used the above statement in a subquery and used  the filter where rownumber=1 to fetch the number 1 product that is sold least in a store

  

    select *
    from
    (
    select StoreId,ProductId,
    count(SalesId) over() as total_sales,
    count(SalesId) over(partition by ProductId) as total_sales_products,
    count(SalesId) over(partition by ProductId , StoreId) as total_sales_products_store,
    row_number() over(partition by StoreId,ProductId) as rownumber,
    from `myfirstproject-445620.sales.sales_data`
    order by count(SalesId) over(partition by ProductId , StoreId) )
    where rownumber=1

##  *Finding the profit in a year for the store*



 Joined two tables products and sales , to get the unit price, product cost and quantity for calculating the profit.
 First -> Calculated the profit  (UnitPrice - ProductCost)*Quantity , Used the aggregate  & window function to count the salesid for a store
 second -> used the above statement in a subquery and found the SUM of profit , grouped by year and store id.
 This helps us finding the total profit for a store in a year.
 

 

    select
    round(sum(year_profit_per_store.profit),2) as profit_store , year_profit_per_store.year, year_profit_per_store.storeid,
    from
    (select round((sales.UnitPrice - products.ProductCost)* sales.Quantity , 2) as profit , 
    sales.date , count(sales.SalesId) over (partition by sales.StoreId) as sales_per_store, sales.StoreId,
    extract(year from date) as year,
    from `myfirstproject-445620.sales.products_data` as products
    join
    `myfirstproject-445620.sales.sales_data` as sales
    on 
    products.ProductId=sales.ProductId) as year_profit_per_store
    group by year_profit_per_store.year , year_profit_per_store.storeid
    order by year_profit_per_store.year , round(sum(year_profit_per_store.profit),2) desc


## *Which store has highest/least number of sales*


This helps to find the overall highest/least sales in a store.


   *Which store has highest number of sales*

   
    select sales.StoreId,inventory.StoreName,
    count(sales.SalesId)  as total_sales
    from `myfirstproject-445620.sales.sales_data` as sales
    join `myfirstproject-445620.sales.inventory_data` as inventory
    on sales.StoreId=inventory.StoreId
    group by sales.StoreId , inventory.StoreName
    order by total_sales desc

  *Which store has lowest number of sales*


    select sales.StoreId,inventory.StoreName,
    count(sales.SalesId)  as total_sales
    from `myfirstproject-445620.sales.sales_data` as sales
    join `myfirstproject-445620.sales.inventory_data` as inventory
    on sales.StoreId=inventory.StoreId
    group by sales.StoreId , inventory.StoreName
    order by total_sales asc


## *Which product has highest/least number of sales*

This helps to find the overall highest/least selling product.


*Which product has highest number of sales*


    select sales.ProductId,products.ProductName,
    count(sales.ProductId) as num_of_product_sold
    from `myfirstproject-445620.sales.sales_data` as sales
    join `myfirstproject-445620.sales.products_data` as products
    on sales.ProductId=products.ProductId
    group by sales.ProductId , products.ProductName
    order by num_of_product_sold desc


*which product has the least number of sales*

    select sales.ProductId,products.ProductName,
    count(sales.ProductId) as num_of_product_sold
    from `myfirstproject-445620.sales.sales_data` as sales
    join `myfirstproject-445620.sales.products_data` as products
    on sales.ProductId=products.ProductId
    group by sales.ProductId , products.ProductName
    order by num_of_product_sold asc



##  *Finding the profit , running total , rolling total , Total average and moving average*

This is done to find the trend os sales over time.
   First , Joined two tables to find the profit .
   second , used the profit as a subquery to find the running total , rolling total , Total average and moving average for a product according to the order date


    select calc.profit, calc.productid,calc.date,
    sum(calc.profit) over(partition by calc.productid) as profit_product,
    round(sum(calc.profit) over(partition by calc.productid order by calc.date),2) as running_total,
    round(sum(calc.profit) over(partition by calc.productid order by calc.date rows between 2 preceding and current row) ,2)as rolling_total,
    round(avg(calc.profit) over(partition by calc.productid order by calc.date),2) as moving_average,
    round(avg(calc.profit) over(partition by calc.productid),2) as average_by_product,
    from
    (
      select sales.UnitPrice,sales.Quantity,products.ProductCost , sales.Date , sales.ProductId, 
    round((sales.UnitPrice-products.ProductCost)*sales.Quantity ,2)as profit
    from
    `myfirstproject-445620.sales.sales_data` as sales
    join
    `myfirstproject-445620.sales.products_data` as products
    on sales.ProductId=products.ProductId ) as calc


##  *Finding Month over Month analysis for a product in a year*

   In a sales data , we need to check the month over month analysis to check the trend , for the products , which sells the high/low at during the months in a year .
   Doing this we can get to know at what period of time , which product sales goes the highest / lowest or we can stock up those products in as store . 
   we can also check whether the profit of the product is increased/decreased when compared to the previous months.


   Created a temporary table to sum up the profit of a product , in a month , in a year.
   Combined that query as a subquery , then used the lead function to compare the next month sales with the current month .
   In case to see the prevoius month sales when compared to the current month sales used the lag function.



    with monthlydata as(
    SELECT 
    productid,
    round(sum(profit) ,2)as total_profit,
    extract(month from date) as month_profit,
    extract(year from date) as year_profit
    FROM `myfirstproject-445620.sales.profit_overtime` 
    group by productid , extract(month from date) ,
    extract(year from date) 
    )
    
    select * ,
    lag(total_profit) over(partition by productid , year_profit order by month_profit) as previous_month_sales,
    lead(total_profit) over(partition by productid , year_profit order by month_profit) as next_month_sales,
    round(lead(total_profit) over(partition by productid , year_profit order by month_profit) - total_profit ,2) as MoMchange,
    round((lead(total_profit) over(partition by productid , year_profit order by month_profit) - total_profit) *100/total_profit ,2) as MoMpercentagechange,
    from 
    (
    SELECT 
    productid,
    round(sum(profit) ,2)as total_profit,
    extract(month from date) as month_profit,
    extract(year from date) as year_profit
    FROM `myfirstproject-445620.sales.profit_overtime` 
    group by productid , extract(month from date) ,
    extract(year from date) 
    )

## These are the analysis for the sales data .
## Thankyou for visiting.







    
