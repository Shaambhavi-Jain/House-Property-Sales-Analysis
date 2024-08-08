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
