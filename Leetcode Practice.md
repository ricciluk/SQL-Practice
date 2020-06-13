
#### 1285. Find the Start and End Number of Continuous Ranges

* Window Function

```mysql
SELECT
    MIN(log_id) as start_id,
    MAX(log_id) as end_id
FROM
    (SELECT
        log_id, 
        log_id - row_number() OVER(ORDER BY log_id) as a
    FROM Logs) b
GROUP BY a
```
rationale:  
using `row_number` to generate the row index, then group by the difference between value and row index



* Subquery

```mysql
SELECT start_id, MIN(end_id) as end_id
FROM 
(SELECT log_id start_id FROM logs WHERE log_id-1 NOT IN (select * from logs))a,
(SELECT log_id end_id FROM logs WHERE log_id+1 NOT IN (select * from logs))b
WHERE start_id<=end_id
GROUP BY start_id
```

***

#### 534. Game Play Analysis II

* Window Function

```mysql
SELECT player_id, event_date, SUM(games_played) 
OVER(PARTITION BY player_id ORDER BY event_date 
ROWS BETWEEN UNBOUNDED PRECEDING and CURRENT ROW) as games_played_so_far 
FROM activity
```

***


####  550. Game Play Analysis IV

* Subquery

```mysql
SELECT ROUND((select count(*)
FROM activity
WHERE (player_id,event_date) in (SELECT player_id,DATE(MIN(event_date)+1) 
FROM activity GROUP BY player_id))/(SELECT COUNT(DISTINCT player_id) FROM activity),2) as fraction  
```
***

####  571. Find Median Given Frequency of Numbers

* 1:1 Recursive

```mysql
SELECT AVG(number) as median
FROM (
SELECT n1.number
FROM numbers n1, numbers n2
WHERE n1.number>n2.number
GROUP BY n1.number
HAVING SUM(n2.frequency)>=(SELECT SUM(frequency) FROM number)/2
and SUM(n2.frequency)-AVG(n1.frequency)<=(SELECT SUM(frequency) FROM number)/2) s
```
***

#### 1212. Team Scores in Football Tournament
[1212](https://leetcode-cn.com/problems/team-scores-in-football-tournament/)

* UNION, CASE WHEN, LEFT JOIN, Subquery
```mysql
SELECT teams.team_id,team_name, IFNULL(SUM(num_points),0) AS num_points
FROM Teams left join (
SELECT host_team AS team_id, SUM(host_score) as num_points, 'host' as type    
FROM 
(SELECT host_team, CASE WHEN host_goals>guest_goals THEN 3 WHEN host_goals<guest_goals THEN 0 ELSE 1 END AS host_score
FROM Matches) a 
GROUP BY host_team
UNION
SELECT guest_team AS team_id, SUM(guest_score) as num_points, 'guest' as type    
FROM 
(SELECT guest_team, CASE WHEN host_goals>guest_goals THEN 0 WHEN host_goals<guest_goals THEN 3 ELSE 1 END AS guest_score
FROM Matches) b
GROUP BY guest_team) c
on teams.team_id=c.team_id
GROUP BY teams.team_id
ORDER BY num_points DESC, teams.team_id
```
