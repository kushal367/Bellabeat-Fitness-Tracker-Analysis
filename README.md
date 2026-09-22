drop table if exists daily_activity;
create table daily_activity
(
	Customer_ID						bigint,
	Activity_Date					date,
	Day_of_Week						varchar(20),
	Total_Steps						int,
	Total_Distance					decimal,
	Tracker_Distance				decimal,
	Very_Active_Distance			decimal,
	Moderately_Active_Distance		decimal,
	Light_Active_Distance			decimal,
	Sedentary_Active_Distance		decimal,
	Very_Active_Minutes				int,
	Fairly_Active_Minutes			int,
	Lightly_Active_Minutes			int,
	Sedentary_Minutes				int,
	Calories						int
);

drop table if exists weight_log;
create table weight_log
(
	Customer_ID			bigint,
	Datetimes			timestamp,
	Day_of_Week			varchar(20),
	Dates 				date,	
	Times				time,			
	Weight_Kg			decimal,
	Weight_Pounds		decimal,
	Fat					int,
	BMI					decimal,
	Is_Manual_Report	boolean,
	Manual_Report		int,
	Log_Id				decimal
);

drop table if exists sleep_day;
create table sleep_day
(
	Customer_Id				bigint,
	Sleep_Day				date,
	Day_of_Week				varchar(20),
	Total_Sleep_Records		int,
	Total_Minutes_Asleep	int,
	Total_Time_In_Bed		int
);


select * from daily_activity
select * from weight_log
select * from sleep_day

1 Identify the day of the week when the customers are most active and least 
active. Active is determined based on the no of steps.

select distinct most_active , least_active from
(select day_of_week, sum(total_steps) as totalsteps,
first_value(day_of_week) over(order by sum(total_steps) desc) as most_active,
first_value(day_of_week) over(order by sum(total_steps)) as least_active
from daily_activity
group by day_of_week) as x

2 Identify the customer who has the most effective sleep. Effective sleep is 
determined based on is customer spent most of the time in bed sleeping.
select customer_id, sum(total_time_in_bed) ,
first_value(customer_id) over(order by sum(total_time_in_bed) desc) as effective_sleep
from sleep_day
group by customer_id
limit 1

3 Identify customers with no sleep record.
select  distinct customer_id
from daily_activity
where customer_id not in(select distinct customer_id 
from sleep_day)

4 Fetch all customers whose daily activity, sleep and weight logs are all present.

select distinct d.customer_id 
from daily_activity d
join weight_log w on d.customer_id = w.customer_id
join sleep_day s on d.customer_id = s.customer_id


5 For each customer, display the total hours they slept for each day of the week. 
Your output should contains 8 columns, first column is the customer id and the 
next 7 columns are the day of the week (like monday, tuesday etc)

select * from sleep_day

select distinct customer_id,
sum(
case when lower(day_of_week) like '%monday%'
then total_minutes_asleep/60.0
end
) as Monday,
sum(
case when lower(day_of_week) like '%tuesday%'
then total_minutes_asleep/60.0
end
) as Tuesday,
sum(
case when lower(day_of_week) like '%wednesday%'
then total_minutes_asleep/60.0
end
) as Wednesday,
sum(
case when lower(day_of_week) like '%thursday%'
then total_minutes_asleep/60.0
end
) as Thursday,
sum(
case when lower(day_of_week) like '%friday%'
then total_minutes_asleep/60.0
end
) as Friday,
sum(
case when lower(day_of_week) like '%saturday%'
then total_minutes_asleep/60.0
end
) as Saturday,
sum(
case when lower(day_of_week) like '%sunday%'
then total_minutes_asleep/60.0
end
) as Sunday

from sleep_day
group by customer_id


6 For each customer, display the following:
customer_id
date when they had the highest_weight(also mention weight in kg)
date when they had the lowest_weight(also mention weight in kg

select  * from weight_log
SELECT
    customer_id,

    FIRST_VALUE(weight_kg) OVER (
        PARTITION BY customer_id
        ORDER BY weight_kg DESC
    ) AS highest_weight,

    FIRST_VALUE(dates) OVER (
        PARTITION BY customer_id
        ORDER BY weight_kg DESC
    ) AS highest_weight_date

FROM weight_log;

7 Fetch the day when customers sleep the most.

SELECT
    day_of_week,
    SUM(total_minutes_asleep),
    FIRST_VALUE(day_of_week) OVER (
        ORDER BY SUM(total_minutes_asleep) DESC
    ) AS most_sleep
FROM sleep_day
GROUP BY day_of_week;

8 For each day of the week, determine the percentage of time customers spend 
lying on bed without sleeping.

SELECT
    customer_id,
    ROUND(
        SUM(total_time_in_bed - total_minutes_asleep) * 100.0
        / SUM(total_time_in_bed),
        2
    ) AS percentage_not_sleeping
FROM sleep_day
GROUP BY customer_id;

9 Identify the most repeated day of week. Repeated day of week is when a day 
has been mentioned the most in entire database
.
--(select * from daily_activity
--select * from weight_log
--select * from sleep_day)
SELECT
    day_of_week,
    COUNT(*) AS total_records
FROM sleep_day
GROUP BY day_of_week
ORDER BY total_records DESC
LIMIT 1;
10 Based on the given data, identify the average kms a customer walks based on 
6000 steps
SELECT
    AVG(tracker_distance) AS average_km
FROM daily_activity
WHERE total_steps >= 6000;
