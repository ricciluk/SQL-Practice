### 

#### 1285. Find the Start and End Number of Continuous Ranges

##### Window Function

```mysql
SELECT
    min(log_id) as start_id,
    max(log_id) as end_id
FROM
    (SELECT
        log_id, 
        log_id - row_number() OVER(ORDER BY log_id) as a
    FROM Logs) b
GROUP BY a
```

using row_number to generate the row index, then group by the difference



##### Subquery

```mysql
SELECT start_id, min(end_id) as end_id
from 
(select log_id start_id from logs where log_id-1 not in (select * from logs))a,
(select log_id end_id from logs where log_id+1 not in (select * from logs))b
where start_id<=end_id
group by start_id
```



#### 534. Game Play Analysis II

##### Window Function

```mysql
select player_id, event_date, sum(games_played) 
over(partition by player_id order by event_date rows between unbounded preceding and current row) as games_played_so_far 
from activity
```



####  550. Game Play Analysis IV

##### Subquery

```mysql
select round((select count(*)
from activity
where (player_id,event_date) in (select player_id,date(min(event_date)+1) from activity group by player_id))/(select count(distinct player_id) from activity),2) as fraction  
```

