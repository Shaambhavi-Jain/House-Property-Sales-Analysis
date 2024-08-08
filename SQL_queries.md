## Table of Contents
- [Data Cleaning](#data-cleaning)
- [Exploratory Data Analysis](#exploratory-data-analysis-eda)
- [Statistical Analysis](#statistical-analysis)
- [Time Series Analysis](#time-series-analysis)
- [Data Visualisation](#data-visualisation)

### Data Cleaning
1. Check for inconsistencies in the data
```sql
```

2. Identify and handle missing values
```sql

```

3. Identify and address potential errors or anomalies in the data
```sql

```

### Exploratory Data Analysis (EDA)
1. Which date corresponds to the highest number of sales?
```sql
select date(datesold), count(*) from time_series.raw_sales
group by datesold
order by count(*) desc
limit 1 
```

2. Which postcode has the highest average price for total sales?
```sql
select postcode, avg(price) from time_series. raw_sales group by postcode
order by avg (price) desc
limit 1
```

3. Which year witnessed the lowest number of sales?
```sql
with cte as (select extract(year from datesold) as year, price from time_series.raw_sales)

select year, sum(price) from cte
group by year
order by sum (price)
limit 1
```

4. Which are the top 6 postcodes with highest revenue generated from sales for each year?
```sql
with cte as (select extract(year from datesold) as year, postcode, price from time_series.raw_sales)

select * from (select *,
row_number() over(partition by year order by price desc) as "r"
from cte) as rank_table
where r <= 6
```

5. What is the most common number of bedrooms?
```sql
select bedrooms, count(*) from time_series.raw_sales
group by bedrooms
order by count(*) desc
```

6. Find the number of properties sold in each quarter of 2017
```sql
select concat(extract(year from datesold), " Q", extract(quarter from datesold)) as "y_q", count(*)
from time_series.raw_sales
where extract(year from datesold) = 2017
group by concat(extract(year from datesold), " Q", extract(quarter from datesold))
```

### Statistical Analysis
1. What is the median property price?
```sql
with cte as (
select price,
row_number() over(order by price) as "row_num"
from time_series.raw_sales
),
median_numbers as (
select * from cte
where row_num in ((select count(*) from time_series.raw_sales)/2, (select count(*) from time_series.raw_sales)/2 + 1)
)

select round(avg(price)) as median from median_numbers
```

2. Determine the year with the highest median property price?
```sql
with cte as (
select extract(year from datesold) as y, price,
row_number() over(partition by extract(year from datesold) order by price) as "row_num"
from time_series.raw_sales
),
cnt_of_sale as (
select y, count(*) as cnt from cte
group by y
),
odd_median_numbers as (
select y, cnt,
(case
when cnt%2 = 1 then round ((cnt+1)/2)
end) as median_th_value
from cnt_of_sale
),
odd_years_median as (
select cte.y, cte.price as median from odd_median_numbers
join cte
on odd_median_numbers.y = cte.y and odd_median_numbers.median_th_value = cte.row_num
),
even_median_numbers as (
select y, cnt,
(case
when cnt%2 = 0 then round(cnt/2)
end) as median_th_value
from cnt_of_sale
),
even_years_median as (
select cte.y, avg(cte.price) as median from even_median_numbers
join cte
on even_median_numbers.y = cte.y
and (even_median_numbers.median_th_value = cte.row_num
or even_median_numbers.median_th_value + 1 = cte. row_num)
group by cte.y
),
medians as (
select * from odd_years_median
union all
select * from even_years_median
)

select * from medians
order by median desc
limit 1
```

3. Determine the outliers in the dataset using Interquartile range (IQR)
```sql

```

4. Determine the percentage of properties with prices above the average price
```sql
select 100 * (count(*)/(select count(*) from time_series.raw_sales)) from time_series.raw_sales
where price › (select round(AVG(price)) from time_series.raw_sales)
```

### Time Series Analysis
1. Calculate the total number of sales for each quarter?
```sql
with cte as (select *,
concat(extract (year from datesold), " Q", extract(quarter from datesold)) as year_quarter
from time_series. raw_sales)

select year_quarter, count(*) from cte
group by year_quarter
order by year_quarter
```

2. Calculate the total revenue from sales for each quarter?
```sql
with cte as (select *,
concat(extract (year from datesold), " Q", extract(quarter from datesold)) as year_quarter
from time_series. raw_sales)

select year_quarter, sum(price) from cte
group by year_quarter
order by year_quarter
```
3. What is the average price change over time for each postcode?
```sql
```

4. Identify the month with the highest total sales for each year (seasonal analysis).
```sql
with cte as (
select date_format(datesold, "%Y-%M") as "y_m", price from time_series.raw_sales
),
monthly_revenue as (
select left(y_m, 4) as year, y_m, sum(price) as revenue from cte
group by y_m
),
ranked_revenue as (
select *,
rank() over(partition by year order by revenue desc) as "r"
from monthly_revenue
)

select year, y_m, revenue from ranked_revenue
where r = 1
```

5. Calculate a 3-month moving average of property prices.
```sql
```

6. Calculate the year-over-year percentage change in property prices for each postcode.
```sql
with cte as (select postcode, extract(year from datesold) as y, sum(price) as revenue from time_series.raw_sales
group by postcode, y
order by postcode, y)

select c1.postcode, c2.y as current_year, round(100 * ((c2.revenue - c1.revenue) / c1.revenue), 2) as yoy_growth
from cte c1
join cte c2
on c1.postcode = c2.postcode and c1.y + 1 = c2.y
-- now any particular postcode can be selected from this query result to view it's YoY growth data
```

### Data Visualisation
