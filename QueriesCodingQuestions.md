# QUESTION 1
## Workers With The Highest Salaries
You have been asked to find the job titles of the highest-paid employees. Your output should include the highest-paid title or multiple titles with the same salary.

Tables: *worker, title*
## PostgreSQL Solution
```sql
-- Create a temporary table that is the inner join of worker and title tables
WITH workerTitle AS (
    SELECT salary, worker_title
    FROM worker
    INNER JOIN title ON worker.worker_id = title.worker_ref_id
    )

-- Output the workerTitle table rows where the salary maximum    
SELECT worker_title
FROM workerTitle

-- To use the maximum salary, the MAX function has to be on a subquery
WHERE salary = (SELECT MAX(salary) FROM workerTitle)
```
## Python Solution
```python
# Import needed modules
import pandas as pd

# Inner join of worker and title tables, sort it by salary descending
workerTitle = worker[['worker_id','salary']].merge(title[['worker_ref_id','worker_title']], how='inner', left_on='worker_id', right_on='worker_ref_id').sort_values(by='salary', ascending=False)

# Output the workerTitle table rows where the salary maximum
workerTitle[workerTitle['salary'] == workerTitle['salary'].max()][['worker_title']]
```

<br/><br/>

# QUESTION 2
## Salaries Differences
Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments. Output just the absolute difference in salaries.
## PostgreSQL Solution
```sql
-- Create a temporary table that is the inner join of db_employee and title db_dept
WITH empDep AS (
    SELECT *
    FROM db_employee AS e
    INNER JOIN db_dept AS d ON e.department_id = d.id)

-- Output the absolute difference between the highest engineering and marketing salaries   
SELECT
    ABS(MAX(CASE WHEN department = 'engineering' THEN salary END) - MAX(CASE WHEN department = 'marketing' THEN salary END))
FROM empDep
```
## Python Solution
```python
# Import needed modules
import pandas as pd

# Inner join of db_employee and title db_dept
empANDdep = db_employee.merge(db_dept, left_on='department_id', right_on='id')

# Create dataframe with only engineering worker
eng = empANDdep.loc[empANDdep['department'] == 'engineering']

# Create dataframe with only marketing workers
mar = empANDdep.loc[empANDdep['department'] == 'marketing']

# Computing the absolute difference between the highest engineering and marketing salaries
answer = abs(eng['salary'].max() - mar['salary'].max())
answer
```
<br/><br/>

# QUESTION 3
## Unique Users Per Client Per Month
Write a query that returns the number of unique users per client per month.

Table: *fact_events*
## PostgreSQL Solution
```sql
-- Extract the month from the datetime field and count each unique user_id
SELECT client_id, EXTRACT(MONTH FROM time_id) AS month, COUNT(DISTINCT user_id) AS users
FROM fact_events

-- Group and order first by cliente_id, then by month
GROUP BY client_id, month
ORDER BY client_id, month
```
## Python Solution
```python
# Import needed modules
import pandas as pd

# Extract the month from the datetime field
fact_events['month'] = pd.DatetimeIndex(fact_events['time_id']).month

# Group and order first by cliente_id, then by month, use SQL grouping and count the unique values
fact_events[['client_id','month','user_id']].groupby(['client_id','month'], as_index=False).nunique()
```
<br/><br/>

# QUESTION 4
## Share of Active Users
Output share of US users that are active. Active users are the ones with an "open" status in the table.

Table: *fb_active_users*
## PostgreSQL Solution
```sql
SELECT
    -- Because the division (/) in PostgreSQL doesn't output a decimal, I use CAST for the conversion
    -- Conditional statement using CASE to select USA active users
    CAST(COUNT(CASE WHEN country = 'USA' AND status = 'open' THEN country END) AS DECIMAL(7,2)) 
    
    -- Then divide them the the total USA user to obtain share of USA-active/USA users
    / COUNT(CASE WHEN country = 'USA' THEN country END)
FROM fb_active_users
```
## Python Solution
```python
# Import needed modules
import pandas as pd

# Definition of USA users (us) and extracting their quantity
us = fb_active_users[fb_active_users['country'] == 'USA'].shape[0]

# Definition of USA active users (us_act) and extracting their quantity
us_act = fb_active_users[(fb_active_users['status'] == 'open') & (fb_active_users['country'] == 'USA')].shape[0]

# Then dividing them the the total USA user to obtain share of USA-active/USA users
us_act / us
```