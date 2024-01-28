
# Data preparation
## remove outliers
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

