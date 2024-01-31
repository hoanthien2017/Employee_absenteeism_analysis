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
total_absences desc
;
```
<details>
  <summary>Results</summary>

|month_of_absence|	total_absences|	avg_absence_hours|
|----------------|	--------------|	----------------|
|March|	78	|4.42|
|February|	64	|4.22|
|October|	62	|4.24|
|November|	56	|4.50|
|May|	55	|4.18|
|July|	53	|4.23|
|September|	49	|4.73|
|August|	46	|4.46|
|January|	46	|3.93|
|June|	42	|4.14|
|December|	42	|3.93|
|April|	40	|3.58|
</details>

2 --Top 10 common Reasons for absences 
```sql
select
		r.reason
	,	count(a.id) as number_of_absence_cases
 	,	round((count(a.id) * 100.0) / sum(count(a.id)) over ()) as percentage_of_total_absences
from
	reasons r
		 join absence_infor a
			on r.number = a.reason_num
group by
		r.reason
order by
		number_of_absence_cases desc
limit 10
;
```
<details>
  <summary>Results</summary>

|reason| number_of_absence_cases| percentage_of_total_absences|
|------|------------------------|-----------------------------|
|medical consultation|	147|	23|
|dental consultation|	110|	17|
|physiotherapy|	68|	11|
|Diseases of the musculoskeletal system and connective tissue|	38|	6|
|patient follow-up|	37|	6|
|unjustified absence|	32|	5|
|laboratory examination|	31|	5|
|Injury, poisoning and certain other consequences of external causes|	26|	4|
|Diseases of the digestive system|	22|	3|
|Diseases of the respiratory system|	21|	3|
</details>

3-- Absence time by age group 
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
|20-30|	3.74|
|41-50|	4.38|
|31-40|	4.41|
|over 50|	5.00|

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
	absence_time_by_distance desc
;
```
**Results:**

|dis_fr_res_group|	absence_time_by_distance|
|----------------|	------------------------|
|near|	4.45|
|far|	4.43|
|Moderate|	3.85|

```sql
select
		trans_exp_group
	,	cast(avg(a.absence_time_hours) as decimal(10,2)) as absence_time_by_trans_exp
from
	(select
		com_exp_id	
	,	trans_exp
	,	case when trans_exp < 200 then 'low'
			 when trans_exp between 200 and 300 then 'medium'	
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
|high|	6.77|
|medium|	4.43|
|low|	3.57|

5-- Absence time by BMI 
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
	Weight_Status
order by 
	avg_absence_hours
;
```
**Results:**
|weight_status| avg_absence_hours|
|-------------| -----------------|
|thin|	2.85|
|fat|	4.25|
|normal|	4.43|

6 --Absence time by kid caregiving factor
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
|no children|	3.76|
|have 1-2 children|	4.44|
|have 3-4 children|	5.59|

7 --Absence time by social smoke and drink
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
|no drink and no smore|	3.80|
|drink or smoke|	4.55|

8 -- Explore the relationship between service time, workload, target achievement, and absence time
```sql
with professional_scores as (
	select
		d.dym_id
	
	--the number of years working at the company
	,	case
		when service_time between 1 and 5 then 1 
		when service_time between 5 and 15 then 2
		else 3
		end as service_time_score 
	
	--average daily workload
	,	case
		when workload_avg_day < 250 then 1
		when workload_avg_day between 250 and 350 then 2
		else 3
		end as workload_score
	
	--the percentage of the goal achieved,
	,	case
		when hit_target = 100  then 3
		when hit_target between 95 and 99 then 2
		else 1
		end as hit_target_score
	from
	dynamics d
	join absence_infor a 
		on d.dym_id = a.dym_id
where 
absence_time_hours <> 10
)

select
		(p.service_time_score + p.workload_score + p.hit_target_score) as professional_score
	,	cast(avg(a.absence_time_hours) as int) as avg_absence_hours
from
	absence_infor a
		join professional_scores p 
		on a.dym_id = p.dym_id
where 
absence_time_hours <> 10
group by
		professional_score
order by 
 avg_absence_hours desc
;
```
**Results:**

|professional_score|	avg_absence_hours|
|------------------|	-----------------|
|3	|6|
|4	|5|
|6	|4|
|7	|4|
|5	|4|
|8	|2|

9--Compensation Influence on Absence
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
		on c.comp_id = a.comp_id
group by
	comp_group
;
```
**Results:**

|comp_group|	avg_absence_hours|
|----------|	-----------------|
|under 30|	4.06|
|30-40|	4.15|
|40-55|	4.39|
