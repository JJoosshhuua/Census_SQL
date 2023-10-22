
-- Used JOINS, SUBQUERIES, CTEs, AGGREGATE functions, etc to analyze the Census data

select * from Census_Data1;
select * from Census_Data2;

-- No of rows in Data 1 

select count(*) from Census_Data1;

select * from Census_Data2

-- No of rows in Data 2

select count(*) from Census_Data2;

-- Average Literacy rate

select avg(literacy) from Census_Data1;

select "state", literacy from Census_Data1
where "state" = 'Kerala';

select "State", avg(literacy) as Kerala_Lit from Census_Data1
group by "State"
having "State" = 'Kerala';

-- Top States with good literacy Rate

select Top 3 "State", avg(Literacy) as Literacy_Rate from Census_Data1
group by "State"
order by Literacy_Rate desc;

select "State", avg(Literacy) as Literacy_Rate, Rank () over (order by avg(Literacy) desc) as Rank_Literacy from Census_Data1
group by "State";

-- Bottom States on Literacy Rate

select Top 3 "State", avg(Literacy) as Literacy_Rate from Census_Data1
group by "State"
order by Literacy_Rate asc;

-- Top and bottom literacy rate

select * from (select Top 3 "State", round(avg(Literacy),0) as Literacy_Rate from Census_Data1 group by "State" order by Literacy_Rate desc) as a
union 
select * from (select Top 3 "State", round(avg(Literacy), 0) as Literacy_Rate from Census_Data1 group by "State" order by Literacy_Rate asc) as b;

-- States starting with letter "U"
select distinct State from Census_Data1 where State like 'U%';
-- States ending with letter "h"
select distinct State from Census_Data1 where lower(State) like '%h';
-- States ending with letter "m" or starting with "B"
select distinct State from Census_Data1 where lower(State) like '%h' or lower(State) like 'B%';
-- States starting with letter "M" and ending with "m"
select distinct State from Census_Data2 where lower(State) like 'M%' and lower(State) like '%m';

-- Top 10 states with good Population Growth rate

select Top 10 "State", avg(Growth)*100 as Growth_Rate from Census_Data1
group by "State"
order by Growth_Rate desc;

select Top 10 "State", avg(Growth)*100 as Growth_Rate, rank () over (Order by avg(Growth)*100 desc) as Full_Growth_Rank from Census_Data1
group by "State";

-- Top 10 states that performed well in reducing the population rate

select Top 10 "State", avg(Growth)*100 as Growth_Rate, rank () over (Order by avg(Growth)*100) as Growth_Reduction_Rank from Census_Data1
group by "State";

-- Top 10 states that have less Sex Ratio

select Top 10 "State", avg(Sex_Ratio) as Sex_Ratio_Rate, rank () over (Order by avg(Sex_Ratio)) as Sex_Ratio_Rank from Census_Data1
group by "State";

-- Top 10 states that have More Sex Ratio

select Top 10 "State", avg(Sex_Ratio) as Sex_Ratio_Rate, rank () over (Order by avg(Sex_Ratio) desc) as Sex_Ratio_Rank from Census_Data1
group by "State";

-- Total Population

select sum(Population) from Census_Data2;

Select "State", sum(population) from Census_Data2
group by "State";

select * from Census_Data1;

select * from Census_Data2;

select state, floor(sum("Population"/"Area_km2")) as Pop_by_area from Census_Data2 group by state order by Pop_by_area desc;

select state, floor(sum("Population")/sum("Area_km2")) as Pop_by_area from Census_Data2 group by state order by Pop_by_area desc;

select district, state, floor("Population"/"Area_km2") as Pop_by_area from Census_Data2 order by Pop_by_area desc;

With Big_Table as (
select a.district, a.State, a.Growth, a.Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District)
select * from Big_Table;

-- Male and Female Population

select district, state, round(population/(sex_ratio + 1), 0) as male, round((population*sex_ratio)/(sex_ratio + 1), 0) as female from
(select a.district, a.State, a.Growth, a.Sex_Ratio/1000 as Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c;

select d.state, sum(d.male) as male, sum(d.female) as female from
(select district, state, round(population/(sex_ratio + 1), 0) as male, round((population*sex_ratio)/(sex_ratio + 1), 0) as female from
(select a.district, a.State, a.Growth, a.Sex_Ratio/1000 as Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c) as d group by state order by female desc;

-- Literate People

select State, round(sum(Literacy1*population),0)  as Literate_people from (
select state, literacy/100 as Literacy1, Population from (select a.district, a.State, a.Growth, a.Sex_Ratio/1000 as Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c) as d group by State;

select state, round(sum(literacy/100),0) as Literacy, round(sum(Population),0) as Population from (select a.district, a.State, a.Growth, a.Sex_Ratio/1000 as Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c group by State order by State

-- Male and Female Literates from each state

Select State, round(avg(Literacy/100)*sum(male), 0) as Male_Literates, round(avg(Literacy/100)*sum(female), 0) as Female_Literates, sum(population) as Pop from (
select district, state, Literacy, round(population/(sex_ratio + 1), 0) as male, round((population*sex_ratio)/(sex_ratio + 1), 0) as female, population from
(select a.district, a.State, a.Growth, a.Sex_Ratio/1000 as Sex_Ratio, a.Literacy, b.Area_Km2, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c) as d group by State

-- Previous Census vs Current Census

select '1' as keyy,n. * from(
select state, round(sum(population/(1+growth)),0) as Prev_Pop, sum(Population) as Curr_Pop from(
select a.district, a.State, a.Growth, b.Population from Census_Data1 as a
join Census_Data2 as b
on a.District=b.District) as c group by State) as d
