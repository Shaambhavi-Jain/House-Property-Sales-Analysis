#### Exploratory Data Analysis (EDA)
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
