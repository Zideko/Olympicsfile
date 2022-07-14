   ## SQL Queries on Olympics data
# _First create a database called Olympics on Postgresql and create two tables for the data given. First table is_:
```
drop table if exists Olympics_history; create table if not exists Olympics_history (id int,
name varchar,
sex varchar,
age varchar,
height varchar,
weight varchar,
team varchar,
noc varchar,
games varchar,
year int,
season varchar,
city varchar,
sport varchar,
event varchar,
medal varchar);
```
# _Second table_:
```
drop table if exists Noc_regions;
create table if not exists Noc_regions (
Noc varchar,
region varchar,
notes varchar,);
```
 Proceed to import the data and solve the following queries:
### 1. How many olympics games have been held?
```
select count (distinct games) as total_games from olympics_history;
```

### 2. List down all Olympics games held so far.
```
select distinct year, season, city from olympics_history order by year;
```
### 3. Mention the total no of nations who participated in each olympics game?
```
select distinct games, count(distinct noc) from olympics_history group by distinct games;
```
### 4. Which year saw the highest and lowest no of countries participating in olympics?
```
select distinct games,count(distinct noc) as position from 
olympics_history group by distinct games order by position limit 1;
```
### 5. Which nation has participated in all of the olympic games?
```
select team, noc, count(distinct games) as position from olympics_history group by team, noc order by position desc;
```
### 6. Identify the sport which was played in all summer olympics.
```
with t1 as
        (select count (distinct games) from olympics_history where season = 'Summer'),
    t2 as
        (select sport, count(distinct games) as total_games from 
olympics_history where season = 'Summer' group by sport order by total_games desc)
        select sport, total_games from t2;
```
### 7. Which Sports were just played only once in the olympics?
```
select sport, count(distinct games) as position
from olympics_history group by sport order by position;
```
### 8. Fetch the total no of sports played in each olympic games.
```
select games, count(distinct sport) as number_games from olympics_history group by games order by number_games;
```
### 9. Fetch details of the oldest athletes to win a gold medal.
```
select * from olympics_history where medal = 'Gold' order by age desc;
```
### 10. Find the Ratio of male and female athletes participated in all olympic games.
```
with t1 as 
     (select sex, count(1) as cnt from olympics_history group by sex),
     t2 as 
     (select *, row_number() over (order by cnt) as rn from t1),
      min_cnt as
        	(select cnt from t2	where rn = 1),
        max_cnt as
        	(select cnt from t2	where rn = 2)
    select concat('1 : ', round(max_cnt.cnt::decimal/min_cnt.cnt, 2)) as ratio
    from min_cnt, max_cnt;
```
### 11. Fetch the top 5 athletes who have won the most gold medals.
```
with t1 as
         (select name, team, count(medal) as total_medals from olympics_history
         where medal='Gold' 
         group by name, team
         order by total_medals desc),
      t2 as
         (select *, dense_rank() over(order by total_medals desc) as rnk from t1)
         
select name,team, total_medals, rnk from t2 where rnk <=5
```
### 12.Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).
```
with t1 as
          (select name, team, count(medal) as total_medals from olympics_history
           where medal in ('Gold','Bronze', 'Silver')
          group by name, team
          order by total_medals desc),
      t2 as
          (select *, dense_rank() over(order by total_medals desc) as rnk from t1)
  select name, total_medals, rnk from t2 where rnk <=5;

```
### 13. Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.
```
with t1 as
       (select noc_regions.region, count(medal) as total_medals from olympics_history 
        join noc_regions on olympics_history.noc = noc_regions.noc
       where medal in ('Gold','Silver','Bronze')
       group by noc_regions.region
       order by total_medals desc),
     t2 as
         (select *, dense_rank() over(order by total_medals desc) as rnk from t1)
         
 select * from t2 where rnk <=5;
```
### 14. List down total gold, silver and broze medals won by each country.
```
CREATE EXTENSION TABLEFUNC;

SELECT country
    	, coalesce(gold, 0) as gold
    	, coalesce(silver, 0) as silver
    	, coalesce(bronze, 0) as bronze
    FROM CROSSTAB('SELECT noc_regions.region as country
    			, medal
    			, count(medal) as total_medals
    			FROM olympics_history
    			JOIN noc_regions  ON noc_regions.noc = olympics_history.noc
    			where medal <> ''NA''
    			GROUP BY country ,medal
    			order BY country,medal',
            'values (''Bronze''), (''Gold''), (''Silver'')')
    AS FINAL_RESULT(country varchar, bronze bigint, gold bigint, silver bigint)
    order by gold desc, silver desc, bronze desc;

```
