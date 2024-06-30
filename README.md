

## SQL Server Tables and Data

```sql
-- Create Users Table
CREATE TABLE users (
    USER_ID INT PRIMARY KEY,
    USER_NAME VARCHAR(20) NOT NULL,
    USER_STATUS VARCHAR(20) NOT NULL
);

-- Create Logins Table
CREATE TABLE logins (
    USER_ID INT,
    LOGIN_TIMESTAMP DATETIME NOT NULL,
    SESSION_ID INT PRIMARY KEY,
    SESSION_SCORE INT,
    FOREIGN KEY (USER_ID) REFERENCES USERS(USER_ID)
);

-- Insert Data into Users Table
INSERT INTO USERS VALUES (1, 'Alice', 'Active');
INSERT INTO USERS VALUES (2, 'Bob', 'Inactive');
INSERT INTO USERS VALUES (3, 'Charlie', 'Active');
INSERT INTO USERS VALUES (4, 'David', 'Active');
INSERT INTO USERS VALUES (5, 'Eve', 'Inactive');
INSERT INTO USERS VALUES (6, 'Frank', 'Active');
INSERT INTO USERS VALUES (7, 'Grace', 'Inactive');
INSERT INTO USERS VALUES (8, 'Heidi', 'Active');
INSERT INTO USERS VALUES (9, 'Ivan', 'Inactive');
INSERT INTO USERS VALUES (10, 'Judy', 'Active');

-- Insert Data into Logins Table
INSERT INTO LOGINS VALUES (1, '2023-07-15 09:30:00', 1001, 85);
INSERT INTO LOGINS VALUES (2, '2023-07-22 10:00:00', 1002, 90);
INSERT INTO LOGINS VALUES (3, '2023-08-10 11:15:00', 1003, 75);
INSERT INTO LOGINS VALUES (4, '2023-08-20 14:00:00', 1004, 88);
INSERT INTO LOGINS VALUES (5, '2023-09-05 16:45:00', 1005, 82);
INSERT INTO LOGINS VALUES (6, '2023-10-12 08:30:00', 1006, 77);
INSERT INTO LOGINS VALUES (7, '2023-11-18 09:00:00', 1007, 81);
INSERT INTO LOGINS VALUES (8, '2023-12-01 10:30:00', 1008, 84);
INSERT INTO LOGINS VALUES (9, '2023-12-15 13:15:00', 1009, 79);
INSERT INTO LOGINS VALUES (1, '2024-01-10 07:45:00', 1011, 86);
INSERT INTO LOGINS VALUES (2, '2024-01-25 09:30:00', 1012, 89);
INSERT INTO LOGINS VALUES (3, '2024-02-05 11:00:00', 1013, 78);
INSERT INTO LOGINS VALUES (4, '2024-03-01 14:30:00', 1014, 91);
INSERT INTO LOGINS VALUES (5, '2024-03-15 16:00:00', 1015, 83);
INSERT INTO LOGINS VALUES (6, '2024-04-12 08:00:00', 1016, 80);
INSERT INTO LOGINS VALUES (7, '2024-05-18 09:15:00', 1017, 82);
INSERT INTO LOGINS VALUES (8, '2024-05-28 10:45:00', 1018, 87);
INSERT INTO LOGINS VALUES (9, '2024-06-15 13:30:00', 1019, 76);
INSERT INTO LOGINS VALUES (10, '2024-06-25 15:00:00', 1010, 92);
INSERT INTO LOGINS VALUES (10, '2024-06-26 15:45:00', 1020, 93);
INSERT INTO LOGINS VALUES (10, '2024-06-27 15:00:00', 1021, 92);
INSERT INTO LOGINS VALUES (10, '2024-06-28 15:45:00', 1022, 93);
INSERT INTO LOGINS VALUES (1, '2024-01-10 07:45:00', 1101, 86);
INSERT INTO LOGINS VALUES (3, '2024-01-25 09:30:00', 1102, 89);
INSERT INTO LOGINS VALUES (5, '2024-01-15 11:00:00', 1103, 78);
INSERT INTO LOGINS VALUES (2, '2023-11-10 07:45:00', 1201, 82);
INSERT INTO LOGINS VALUES (4, '2023-11-25 09:30:00', 1202, 84);
INSERT INTO LOGINS VALUES (6, '2023-11-15 11:00:00', 1203, 80);
```

## Detailed Explanation for Each Question

### 1. Users who did not log in the last 5 months

To identify users who did not log in within the last 5 months, we use a Common Table Expression (CTE) to find the latest login date for each user. We then filter out users whose last login was before 5 months ago.

```sql
WITH cte AS (
    SELECT USER_ID, MAX(LOGIN_TIMESTAMP) AS latest_login
    FROM logins
    GROUP BY USER_ID
    HAVING MAX(LOGIN_TIMESTAMP) <= DATEADD(MONTH, -5, GETDATE())
)
SELECT users.USER_NAME
FROM cte 
JOIN users ON cte.USER_ID = users.USER_ID
WHERE cte.latest_login <= DATEADD(MONTH, -5, GETDATE());
```

### 2. Quarterly Analysis: User and Session Counts

This query calculates the number of distinct users and sessions for each quarter. The `DATEPART` and `DATETRUNC` functions are used to extract and group data by quarter.

```sql
SELECT DATEPART(YEAR, LOGIN_TIMESTAMP) Year, 
       DATETRUNC(QUARTER, MIN(LOGIN_TIMESTAMP)) First_day_of_Quarter, 
       COUNT(DISTINCT USER_ID) users_count, 
       COUNT(DISTINCT SESSION_ID) session_count
FROM logins 
GROUP BY DATEPART(YEAR, LOGIN_TIMESTAMP), DATEPART(QUARTER, LOGIN_TIMESTAMP)
ORDER BY First_day_of_Quarter DESC;
```

### 3. Users who logged in January 2024 but not in November 2023

This query finds users who logged in January 2024 but did not log in November 2023. It uses `CAST` and `CONCAT` to format dates and compare them.

```sql
SELECT DISTINCT USER_ID
FROM logins
WHERE CAST(CONCAT(DATENAME(MONTH, LOGIN_TIMESTAMP), ' ', DATEPART(YEAR, LOGIN_TIMESTAMP)) AS VARCHAR(MAX)) = 'January 2024' 
  AND USER_ID NOT IN (
    SELECT DISTINCT USER_ID
    FROM logins
    WHERE CAST(CONCAT(DATENAME(MONTH, LOGIN_TIMESTAMP), ' ', DATEPART(YEAR, LOGIN_TIMESTAMP)) AS VARCHAR(MAX)) = 'November 2023'
  );
```

### 4. Quarterly Session Count and Percentage Change

This query calculates the session count for each quarter and the percentage change from the previous quarter using the `LAG` window function.

```sql
WITH cte AS (
    SELECT DATEPART(YEAR, LOGIN_TIMESTAMP) Year, 
           DATETRUNC(QUARTER, MIN(LOGIN_TIMESTAMP)) First_day_of_Quarter, 
           COUNT(DISTINCT SESSION_ID) session_count
    FROM logins 
    GROUP BY DATEPART(YEAR, LOGIN_TIMESTAMP), DATEPART(QUARTER, LOGIN_TIMESTAMP)
)
SELECT *, 
       LAG(session_count, 1) OVER (ORDER BY First_day_of_Quarter) prev_session_cnt,
       (session_count - LAG(session_count, 1) OVER (ORDER BY First_day_of_Quarter)) * 100.0 / session_count AS Session_percent_change
FROM cte
ORDER BY First_day_of_Quarter DESC;
```

### 5. Highest Session Score per Day

This query finds the user with the highest session score for each day by using `DATETRUNC` to group by day and `MAX` to find the highest score.

```sql
SELECT DATETRUNC(DAY, CAST(l.LOGIN_TIMESTAMP AS DATE)) Date,
       u.USER_NAME name,
       MAX(l.SESSION_SCORE) score
FROM logins AS l
JOIN users AS u ON l.USER_ID = u.USER_ID
GROUP BY DATETRUNC(DAY, CAST(l.LOGIN_TIMESTAMP AS DATE)), u.USER_NAME
ORDER BY Date, score DESC;
```

### 6. Users with Sessions on Every Day Since First Login

This query identifies users who logged in every single day since their first login by comparing the number of distinct login days to the total number of days since the first login.

```sql
WITH cte AS (
    SELECT USER_ID, 
           LOGIN_TIMESTAMP, 
           MIN(LOGIN_TIMESTAMP) OVER (PARTITION BY USER_ID) first_login, 
           COUNT(USER_ID) OVER (PARTITION BY USER_ID ORDER BY USER_ID) cnt,
           ABS(DATEDIFF(DAY, LOGIN_TIMESTAMP, MIN(LOGIN_TIMESTAMP) OVER (PARTITION BY USER_ID))) + 1 days
    FROM logins
)
SELECT USER_ID
FROM cte 
GROUP BY USER_ID, cnt
HAVING MAX(days) = cnt;
```

### 7. Dates with No Logins

This query finds dates with no

 logins by generating a series of dates and checking for gaps.

```sql
WITH DateList AS (
    SELECT MIN(LOGIN_TIMESTAMP) AS MIN_DATE, MAX(LOGIN_TIMESTAMP) AS MAX_DATE
    FROM logins
    UNION ALL
    SELECT MIN_DATE + INTERVAL 1 DAY AS MIN_DATE, MAX_DATE
    FROM DateList
    WHERE MIN_DATE < MAX_DATE
)
SELECT DateList.MIN_DATE
FROM DateList
LEFT JOIN logins ON DateList.MIN_DATE = CAST(logins.LOGIN_TIMESTAMP AS DATE)
WHERE logins.LOGIN_TIMESTAMP IS NULL;
```

### 8. Highest Session Score User Status Proportion

This query calculates the proportion of active and inactive users with the highest session score per day.

```sql
WITH cte AS (
    SELECT DATETRUNC(DAY, CAST(l.LOGIN_TIMESTAMP AS DATE)) Date,
           u.USER_NAME name,
           u.USER_STATUS,
           MAX(l.SESSION_SCORE) score
    FROM logins AS l
    JOIN users AS u ON l.USER_ID = u.USER_ID
    GROUP BY DATETRUNC(DAY, CAST(l.LOGIN_TIMESTAMP AS DATE)), u.USER_NAME, u.USER_STATUS
    ORDER BY Date, score DESC
)
SELECT USER_STATUS,
       COUNT(USER_NAME) as USER_COUNT
FROM cte 
GROUP BY USER_STATUS;
```

