
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
[CLICK HERE FOR QUESTION](https://leetcode-cn.com/problems/team-scores-in-football-tournament/)

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

***

#### 569. Median Employee Salary
[CLICK HERE FOR QUESTION](https://leetcode-cn.com/problems/median-employee-salary/)

* 1:1 Recursive
```mysql
SELECT id, company, salary
FROM employee
WHERE id IN (
    SELECT e1.id
    FROM employee e1, employee e2
    WHERE e1.company=e2.company
    GROUP BY e1.id
    HAVING SUM(CASE WHEN e1.salary>=e2.salary THEN 1 ELSE 0 END) >= COUNT(*)/2 
    AND SUM(CASE WHEN e1.salary<=e2.salary THEN 1 ELSE 0 END) >= COUNT(*)/2
)
GROUP BY company,salary
```
***
#### 1454. Active Users
[CLICK HERE FOR QUESTION](https://leetcode-cn.com/problems/active-users/)

* WINDOW Function
* LAG Function(return values from a previous row in the table), LEAD Function(return values from a next row in the table)
```mysql
SELECT DISTINCT b.id, name 
FROM 
(SELECT id, DATEDIFF(login_date, lag(login_date, 4) OVER(PARTITION BY id ORDER BY id, login_date)) AS diff 
FROM (SELECT DISTINCT * FROM Logins) a) b, accounts
WHERE b.id=accounts.id AND diff=4
ORDER BY b.id
```
```mysql
SELECT DISTINCT b.id, name 
FROM 
(SELECT id, DATEDIFF(lead(login_date, 4) OVER(PARTITION BY id ORDER BY id, login_date),login_date) AS diff 
FROM (SELECT DISTINCT * FROM Logins) a) b, accounts
WHERE b.id=accounts.id AND diff=4
ORDER BY b.id
```

***

#### 1280. Students and Examinations
[CLICK HERE FOR QUESTION](https://leetcode-cn.com/problems/students-and-examinations/)

* CROSS JOIN(Cartesian Product, the number of rows in the first table multiplied by the number of rows in the second table)
```mysql
SELECT a.student_id, a.student_name, a.subject_name, IFNULL(attended_exams,0) AS attended_exams
FROM (
SELECT *
FROM students CROSS JOIN subjects) a LEFT JOIN (
SELECT *, COUNT(*) AS attended_exams 
FROM examinations
GROUP BY student_id,subject_name) b
ON a.student_id =b.student_id AND a.subject_name =b.subject_name 
ORDER BY a.student_id, a.subject_name
```

***

#### 1336. Number of Transactions per Visit
[CLICK HERE FOR QUESTION](https://leetcode-cn.com/problems/number-of-transactions-per-visit/)

* @i := @i + 1(generate sequential numbers)
* CAST Function(convert value into different types)
```
BINARY[(N)]
CHAR[(N)]
DATE
DATETIME
DECIMAL[(M[,D])]
SIGNED [INTEGER]
TIME
UNSIGNED [INTEGER]
```

```mysql
SELECT c.transactions_count, IFNULL(visits_count,0) AS visits_count
FROM
(SELECT transactions_count, count(*) AS visits_count 
FROM (
SELECT CASE WHEN transaction_date is null THEN 0 ELSE count(*) END AS transactions_count 
FROM (
SELECT v.user_id,v.visit_date,t.transaction_date,t.amount
FROM visits v
LEFT JOIN transactions t
ON v.user_id=t.user_id AND v.visit_date=t.transaction_date) a
GROUP BY a.user_id,visit_date) a
GROUP BY transactions_count) b
RIGHT JOIN
(SELECT CAST(@i := @i + 1 AS UNSIGNED) AS transactions_count
FROM transactions, (SELECT @i := -1) val
WHERE @i < (
    SELECT IFNULL(count(*),0) transactions_count
    FROM transactions 
    GROUP BY user_id, transaction_date
    ORDER BY transactions_count DESC
    LIMIT 1
) UNION SELECT 0) c
ON c.transactions_count=b.transactions_count
```
