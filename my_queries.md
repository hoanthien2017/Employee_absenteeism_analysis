# Data preparation
--- Filter and Remove outliers
```sql
with absence_per_id as (
	select
		id
	,	season
	,	sum(absence_time_hours) as total_absenteeism_time_in_hours
	from
		absence_infor
	group by
		id
	,	season
)
,	avg_std as (
	select
		season
	,	sum(total_absenteeism_time_in_hours) as sum_absence_time
	,	count(*) number_of_id
	,	avg(total_absenteeism_time_in_hours) as avg_absence_time
	,	stddev(total_absenteeism_time_in_hours) as std_absence_time
	from
		absence_per_id
	group by
		season
	)

,	upper_lower_whisker as (
	select
		season
	,	avg_absence_time
	,	std_absence_time
	,	number_of_id
	,	(avg_absence_time + std_absence_time*3) as upper_whisker
	,	case when (avg_absence_time - std_absence_time*3) < 0 then 0
		else (avg_absence_time - std_absence_time*3) end as lower_whisker
	from
		avg_std
)

,	find_outlier as (
	select
		api.id
	,	api.season
	,	api.total_absenteeism_time_in_hours
	,	ulw.number_of_id
	,	ulw.std_absence_time
	,	ulw.avg_absence_time
	,	ulw.upper_whisker
	,	ulw.lower_whisker
	,	case when api.total_absenteeism_time_in_hours > upper_whisker
		or api.total_absenteeism_time_in_hours < lower_whisker
		then 'outlier' else 'expected' end as outlier_status
	from
		absence_per_id api
		left join upper_lower_whisker ulw
		on api.season = ulw.season
)
, remove_outlier as (
select
		season
	,	number_of_id
	,	std_absence_time
	,	upper_whisker
	,	lower_whisker
	,	avg_absence_time
	,	avg(total_absenteeism_time_in_hours) as avg_absence_time_no_outlier
	,	count(*) as new_number_of_id
	,	count(*)/number_of_id*100 as remaining_data
from
	find_outlier
where 
	outlier_status = 'expected'
group by
		season
	,	number_of_id
	,	std_absence_time
	,	upper_whisker
	,	lower_whisker
	,	avg_absence_time
;
```
**Results:**
  
| season | number_of_id | std_absence_time | upper_whisker | lower_whisker | avg_absence_time | avg_absence_time_no_outlier | new_number_of_id |
| ------ | ------------ | ---------------- | ------------- | ------------- | ---------------- | --------------------------- | ---------------- |
|Autumn|195|13.76|48.63|0|7.34|5.43|190|
|Spring|192|11.53|40.91|0|6.32|5.33|190|
|Summer|183|15.00|52.46|0|7.47|5.67|179|
|Winter|170|12.89|45.21|0|6.55|4.99|167|

</details>

# Data exploratory
1 --Temporal Patterns and Seasonality in Absences--
```sql
select
		month_of_absence
	,	count(id) as total_absences
	,	cast(avg(absence_time_hours) as decimal (10,2)) as avg_absence_hours
from
	absence_infor 
group by
		month_of_absence
order by
avg_absence_time desc
;
```
<details>
  <summary>Results</summary>

|month_of_absence|	total_absences|	avg_absence_hours|
|----------------|	--------------|	----------------|
|May|	60|	6.50|
|June|	47|	6.43|
|July|	60|	6.00|
|October|	68|	5.87|
|April|	44|	5.80|
|March|	83|	5.70|
|November|	59|	5.63|
|August|	49|	5.49|
|September|	50|	5.44|
|January|	50|	5.38|
|February|	68|	5.26|
|December|	44|	4.84|
</details>

2 --Top 10 common Reasons for absences 
```sql
select
		r.reason
	,	count(a.id) as number_of_absence_cases
from
	reasons r
		 join absence_infor a
			on r.number = a.reason_num
group by
		r.reason
order by
	count(a.id) desc
limit 10
;
```
**Results:**
	
|reason| number_of_absence_cases|
|------|------------------------|
|medical consultation |	149|
|dental consultation |	112|
|physiotherapy |	68|
|Diseases of the musculoskeletal system and connective tissue|	52|
|patient follow-up |	38|
|Injury, poisoning and certain other consequences of external causes|	35|
|unjustified absence |	33|
|laboratory examination | 31|
|Diseases of the digestive system|	25|
|Diseases of the respiratory system|	25|

3-- absence time by age group 
```sql
select
	case 
	when d.age between 20 and 30 then '20-30'
	when d.age between 30 and 40 then '31-40'
	when d.age between 40 and 50 then '41-50'
	else 'over 50'
	end as age_group
	, cast(avg(a.absence_time_hours) as decimal(10,2)) as avg_absence_hours
from
	demographics d
	join absence_infor a	
		on d.demog_id = a.demog_id
group by
	age_group
order by 
	avg_absence_hours 
;
```
**Results:**

|age_group|	avg_absence_hours|
|---------|	-----------------|
|20-30|	4.62|
|over 50|	5.00|
|41-50|	5.80|
|31-40|	6.16|

4 --Check if longer commute distances or higher transportation expenses impact absence time
```sql
select
		dis_fr_res_group
	,	cast(avg(a.absence_time_hours) as decimal(10,2)) as absence_time_by_distance
from
	(	select
		com_exp_id	
	,	dis_fr_res
	,	case when dis_fr_res < 20 then 'near'
			when dis_fr_res between 20 and 30 then 'Moderate' 
			else 'far'
			end as dis_fr_res_group
	from
		com_and_exp
) as commuting_distance
	join absence_infor a
		on a.com_and_exp_id = commuting_distance.com_exp_id
group by
		dis_fr_res_group
order by  
	absence_time_by_dis desc
;
```
**Results:**

|dis_fr_res_group|	absence_time_by_distance|
|----------------|	------------------------|
|near|	6.14|
|far|	6.03|
|Moderate|	4.97|

```sql
select
		trans_exp_group
	,	cast(avg(a.absence_time_hours) as decimal(10,2)) as absence_time_by_trans_exp
from
	(select
		com_exp_id	
	,	trans_exp
	,	case when trans_exp < 200 then 'low'
			when trans_exp BETWEEN 200 AND 300 then 'medium'	
			else 'high'
			end as trans_exp_group 
	 	from
		com_and_exp 
	) as commuting_free
	join absence_infor a
		on a.com_and_exp_id = commuting_free.com_exp_id
group by
		trans_exp_group
order by  
	absence_time_by_trans_exp desc
;
```
**Results:**
|trans_exp_group|	absence_time_by_trans_exp|
|---------------|	-------------------------|
|high|	8.12|
|medium|	5.86|
|low|	5.10|

5 -- absence time by health factors
```sql
select
	case 
	when d.body_mass_index < 20 then 'thin'
	when d.body_mass_index between 20 and 25 then 'normal'
	else 'fat'
	end as Weight_Status
	, cast(avg(a.absence_time_hours) as decimal(10,2)) as avg_absence_hours
from
	demographics d
	join absence_infor a	
		on d.demog_id = a.demog_id
group by
	health
order by 
	avg_absence_hours
```
**Results:**
|Weight_Status|	avg_absence_hours|
|-------------|	-----------------|
|thin|	4.20|
|fat|	5.29|
|normal|	6.35|
	
6 --absence time by son and pet
```sql
select
	case 
	when d.son = 0 then 'no children'
	when d.son between 1 and 2 then 'have 1-2 children'
	else 'have 3-4 children'
	end as son_num
	, cast(avg(a.absence_time_hours) as decimal(10,2)) as avg_absence_hours
from
	demographics d
	join absence_infor a	
		on d.demog_id = a.demog_id
group by
		son_num
order by 
	avg_absence_hours 
;

```
**Results:**
|son_num|	avg_absence_hours|
|-------|	-----------------|
|no children|	4.97|
|have 1-2 children|	6.12|
|have 3-4 children|	7.06|

7 --absence time by social smoke and drink
```sql
select
	case 
	when d.soc_smk = 'yes'
	or d.soc_dnk = 'yes' then 'drink or smoke'
	else 'no drink and no smore'
	end as soc_dnk_smk
	, cast(avg(a.absence_time_hours) as decimal(10,2)) as avg_absence_hours
from
	demographics d
	join absence_infor a	
		on d.demog_id = a.demog_id
group by
	soc_dnk_smk
order by 
	avg_absence_hours 
;
```
**Results:**
|soc_dnk_smk|	avg_absence_hours|
|-----------|	-----------------|
|no drink and no smore|	4.37|
|drink or smoke|	6.58|

8 -- Explore the relationship between service time, workload, target achievement, and absence time
```sql
with professional_scores as (
	select
		dym_id
	,	case
		when service_time between 1 and 5 then 1
		when service_time between 5 and 15 then 2
		else 3
		end as service_time_score
	,	case
		when workload_avg_day < 3  then 1
		when workload_avg_day between 3 and 10 then 2
		else 3
		end as sworkload_avg_day 
	,	case
		when hit_target = 100  then 3
		when hit_target between 95 and 99 then 2
		else 1
		end as hit_target_score
	from
	dynamics
)

select
		(p.service_time_score + p.sworkload_avg_day + p.hit_target_score) as professional_score
	,	cast(avg(a.absence_time_hours) as int) as avg_absence_hours
from
	absence_infor a
		join professional_scores p 
		on a.dym_id = p.dym_id
group by
		professional_score
order by 
 avg_absence_hours desc
;
```
**Results:**

|professional_score|	avg_absence_hours|
|------------------|	-----------------|
|5	|9|
|6	|6|
|8	|6|
|7	|5|
|9	|2|

9--Compensation Influence on Absence:
```sql
select
		case 
		when c.comp_hr < 30 then 'under 30'
		when c.comp_hr between 30 and 40 then '30-40'
		else '40-55'
		end as comp_group
	,	cast(avg(absence_time_hours) as decimal(10,2)) as avg_absence_hours
	
from
	compensation c
	join absence_infor a
		on	c.comp_id = a.comp_id
group by
	comp_group
;
```
**Results:**

|comp_group|	avg_absence_hours|
|----------|	-----------------|
|under 30|	5.39|
|30-40|	5.42|
|40-55|	6.06|
