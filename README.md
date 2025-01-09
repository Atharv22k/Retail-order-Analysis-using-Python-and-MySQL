# Retail Order Analysis using Python and MySQL

## Project Overview
<p><span style="font-size: 20px;">This project aims to analyze retail order data using a combination of Python and SQL. The goal is to extract, transform, and load data for analysis and generate insights.</span></p>

## Prerequisites
- <span style="font-size: 18px;">Python environment with necessary packages (e.g., pandas, sqlalchemy).</span>
- <span style="font-size: 18px;">MySQL installed and configured.</span>

## Steps to Run the Project

### Python Script
1. <span style="font-size: 16px;">**Import Libraries and Download Dataset**</span>
    ```python
    #import libraries
    #!pip install kaggle
    import kaggle

    !kaggle datasets download ankitbansal06/retail-orders -f orders.csv
    ```

2. <span style="font-size: 16px;">**Extract File from Zip Archive**</span>
    ```python
    #extract file from zip file
    import zipfile
    zip_ref = zipfile.ZipFile('orders.csv.zip') 
    zip_ref.extractall() # extract file to dir
    zip_ref.close() # close file
    ```

3. <span style="font-size: 16px;">**Read Data and Handle Null Values**</span>
    ```python
    #read data from the file and handle null values
    import pandas as pd
    df = pd.read_csv('orders.csv', na_values=['Not Available', 'unknown'])
    df['Ship Mode'].unique()
    ```

4. <span style="font-size: 16px;">**Rename Columns**</span>
    ```python
    #rename columns names ..make them lower case and replace space with underscore
    #df.rename(columns={'Order Id':'order_id', 'City':'city'})
    #df.columns = df.columns.str.lower()
    #df.columns = df.columns.str.replace(' ', '_')
    df.head(5)
    ```

5. <span style="font-size: 16px;">**Derive New Columns**</span>
    ```python
    #derive new columns discount, sale price, and profit
    #df['discount'] = df['list_price'] * df['discount_percent'] * .01
    #df['sale_price'] = df['list_price'] - df['discount']
    df['profit'] = df['sale_price'] - df['cost_price']
    df
    ```

6. <span style="font-size: 16px;">**Convert Order Date to Datetime**</span>
    ```python
    #convert order date from object data type to datetime
    df['order_date'] = pd.to_datetime(df['order_date'], format="%Y-%m-%d")
    ```

7. <span style="font-size: 16px;">**Drop Unnecessary Columns**</span>
    ```python
    #drop cost price, list price, and discount percent columns
    df.drop(columns=['list_price', 'cost_price', 'discount_percent'], inplace=True)
    ```

8. <span style="font-size: 16px;">**Load Data into SQL Server**</span>
    ```python
    #load the data into sql server using replace option
    import sqlalchemy as sal
    engine = sal.create_engine('mssql://ANKIT\\SQLEXPRESS/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
    conn = engine.connect()
    
    #load the data into sql server using append option
    df.to_sql('df_orders', con=conn, index=False, if_exists='append')
    ```

### SQL Queries
1. <span style="font-size: 16px;">**Top 10 Highest Revenue Generating Products**</span>
    ```sql
    --find top 10 highest revenue generating products 
    select top 10 product_id, sum(sale_price) as sales
    from df_orders
    group by product_id
    order by sales desc;
    ```

2. <span style="font-size: 16px;">**Top 5 Highest Selling Products in Each Region**</span>
    ```sql
    --find top 5 highest selling products in each region
    with cte as (
        select region, product_id, sum(sale_price) as sales
        from df_orders
        group by region, product_id)
    select * from (
        select *, row_number() over(partition by region order by sales desc) as rn
        from cte) A
    where rn <= 5;
    ```

3. <span style="font-size: 16px;">**Month over Month Growth Comparison for 2022 and 2023 Sales**</span>
    ```sql
    --find month over month growth comparison for 2022 and 2023 sales
    with cte as (
        select year(order_date) as order_year, month(order_date) as order_month, sum(sale_price) as sales
        from df_orders
        group by year(order_date), month(order_date))
    select order_month,
        sum(case when order_year=2022 then sales else 0 end) as sales_2022,
        sum(case when order_year=2023 then sales else 0 end) as sales_2023
    from cte
    group by order_month
    order by order_month;
    ```

4. <span style="font-size: 16px;">**Highest Sales Month for Each Category**</span>
    ```sql
    --for each category which month had highest sales
    with cte as (
        select category, format(order_date, 'yyyyMM') as order_year_month, sum(sale_price) as sales
        from df_orders
        group by category, format(order_date, 'yyyyMM'))
    select * from (
        select *, row_number() over(partition by category order by sales desc) as rn
        from cte) A
    where rn = 1;
    ```

5. <span style="font-size: 16px;">**Subcategory with Highest Growth by Profit in 2023 Compared to 2022**</span>
    ```sql
    --which subcategory had highest growth by profit in 2023 compared to 2022
    with cte as (
        select sub_category, year(order_date) as order_year, sum(sale_price) as sales
        from df_orders
        group by sub_category, year(order_date)),
    cte2 as (
        select sub_category,
            sum(case when order_year=2022 then sales else 0 end) as sales_2022,
            sum(case when order_year=2023 then sales else 0 end) as sales_2023
        from cte
        group by sub_category)
    select top 1 *, (sales_2023 - sales_2022) as sales_growth
    from cte2
    order by sales_growth desc;
    ```

## Conclusion
This README provides a detailed step-by-step guide to performing retail order analysis using Python and SQL. Follow the steps to replicate and analyze the retail data effectively.

---
