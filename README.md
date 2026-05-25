# Baseball-SQL-Analytics-Project
Maven Analytics Final Project using MySQL

A comprehensive SQL analytics project exploring historical baseball player, school, and salary data using advanced SQL techniques including CTEs, window functions, ranking, cumulative analysis, and conditional aggregation.

#### Project Overview

This project analyzes baseball datasets to answer business and sports analytics questions across four major areas:

1. School Analysis
2. Salary Analysis
3. Player Career Analysis
4. Player Comparison Analysis

The project demonstrates proficiency in:

- Complex SQL querying
- Window functions
- Common Table Expressions (CTEs)
- Aggregations
- Ranking functions
- Conditional logic
- Data modeling and joins

#### Database Schema

The project uses the following tables:

| Table            | Description                               |
| ---------------- | ----------------------------------------- |
| `players`        | Player demographic and career information |
| `salaries`       | Annual salary records by player/team      |
| `schools`        | Player school attendance records          |
| `school_details` | Metadata about schools                    |

####  Entity Relationship Overview
##### Relationships

- players.playerID ↔ salaries.playerID
- players.playerID ↔ schools.playerID
- schools.schoolID ↔ school_details.schoolID

####  SQL Concepts Used
##### Advanced SQL Features

- Common Table Expressions (CTEs)
- Window Functions
- ROW_NUMBER()
- LAG()
- NTILE()
- Aggregate Functions
- Conditional Aggregation
- Ranking Logic
- Cumulative Calculations
- Date Functions
- Multi-table Joins

### Project Sections

#### PART I — School Analysis
Questions Answered
- Q1. How many schools produced players in each decade?
```sql
SELECT	FLOOR(yearID/10) * 10 AS decade, COUNT(DISTINCT schoolID) AS num_schools
FROM	schools
GROUP BY decade
ORDER BY decade;
```

- Q2. Which schools produced the most MLB players?
```sql
SELECT	sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
FROM	schools s
LEFT JOIN school_details sd
		ON s.schoolID = sd.schoolID
GROUP BY s.schoolID
ORDER BY num_players DESC
LIMIT 5;
```

- Q3. What were the top schools per decade?
```sql
WITH ds AS (SELECT	FLOOR(s.yearID/10) * 10 AS decade, sd.name_full, COUNT(DISTINCT s.playerID) AS num_players
			FROM	schools s
			LEFT JOIN school_details sd
					ON s.schoolID = sd.schoolID
			GROUP BY decade, s.schoolID),
	rn AS (SELECT	decade, name_full, num_players,
					ROW_NUMBER() OVER (PARTITION BY decade ORDER BY num_players DESC) AS row_num
		   FROM 	ds)
SELECT	decade, name_full, num_players
FROM	rn
WHERE 	row_num <= 3
ORDER BY decade DESC, row_num;
```

#### PART II — Salary Analysis
Questions Answered
- Q1. Which teams are in the top 20% for spending?
```sql
WITH ts AS (SELECT	teamID, yearID, SUM(salary) AS total_spend
			FROM	salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID),
     sp AS (SELECT	teamID, AVG(total_spend) AS avg_spend,
					NTILE(5) OVER (ORDER BY AVG(total_spend) DESC) AS spend_pct
			FROM	ts
			GROUP BY teamID)
SELECT	teamID, ROUND(avg_spend / 1000000, 1) AS avg_spend_millions
FROM 	sp
WHERE	spend_pct = 1;
```

- Q2. How has team spending accumulated over time?
```sql
WITH ts AS (SELECT	teamID, yearID, SUM(salary) AS total_spend
			FROM	salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID)
SELECT teamID, yearID,
	   ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID)/1000000, 1) AS cumulative_sum_millions
FROM ts;
```

- Q3. When did teams surpass $1 billion in cumulative payroll?
```sql
WITH ts AS (SELECT	teamID, yearID, SUM(salary) AS total_spend
			FROM	salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID),
	 cs AS (SELECT teamID, yearID,
				   SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID) AS cumulative_sum
			FROM ts),
	rn AS (SELECT	teamID, yearID, cumulative_sum,
					ROW_NUMBER() OVER (PARTITION BY teamID ORDER BY cumulative_sum) AS rn
		   FROM	cs
		   WHERE	cumulative_sum > 1000000000)
SELECT	teamID, yearID, ROUND(cumulative_sum/1000000000, 2) AS cumulative_sum_billions
FROM	rn
WHERE	rn = 1;
```

#### PART III — Player Career Analysis
Questions Answered
- Q1. What were players’ starting & ending ages and their career length (all in years)?
```sql
SELECT	nameGiven, 
		TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMOnth, '-', birthDay) AS DATE), debut) AS starting_age,
        TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMOnth, '-', birthDay) AS DATE), finalGame) AS ending_age,        
        TIMESTAMPDIFF(YEAR, debut, finalGame) AS career_length
FROM	players
ORDER BY career_length DESC;
```

- Q2. Which teams did players start and end with?
```sql
SELECT	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team, 
        e.yearID AS ending_year, e.teamID AS ending_team
FROM	players p
JOIN 	salaries s
		ON p.playerID = s.playerID
        AND YEAR(p.debut) = s.yearID
JOIN	salaries e
		ON p.playerID = e.playerID
        AND YEAR(finalGame) = e.yearID;
```

- Q3. Which players stayed with the same franchise for over a decade?
```sql
SELECT	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team, 
        e.yearID AS ending_year, e.teamID AS ending_team
FROM	players p
JOIN 	salaries s
		ON p.playerID = s.playerID
        AND YEAR(p.debut) = s.yearID
JOIN	salaries e
		ON p.playerID = e.playerID
        AND YEAR(finalGame) = e.yearID
WHERE 	s.teamID = e.teamID
		AND e.yearID - s.yearID > 10;
```

#### PART IV — Player Comparison Analysis
Questions Answered
- Q1. Which players share birthdays?
```sql
WITH bn AS (SELECT	CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthdate,
					nameGiven
			FROM 	players)
SELECT	birthdate, 
		GROUP_CONCAT(nameGiven SEPARATOR ', ') AS players
FROM	bn
WHERE 	YEAR(birthdate) BETWEEN 1980 AND 1990
GROUP BY birthdate
ORDER BY birthdate;
```

- Q2. What percentage of players bat right, left, or both for each team?
```sql
SELECT	s.teamID,
		ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END)/COUNT(s.playerID) * 100, 1) AS bats_right,
        ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END)/COUNT(s.playerID) * 100, 1) AS bats_left,
        ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END)/COUNT(s.playerID) * 100, 1) AS bats_both
FROM	players p
JOIN	salaries s
		ON p.playerID = s.playerID
GROUP BY s.teamID
ORDER BY s.teamID;
```

- Q3. How have player height and weight changed over decades?
```sql
WITH hw AS (SELECT	FLOOR(YEAR(debut)/10) * 10 AS decade, 
					AVG(height) AS avg_height, AVG(weight) AS avg_weight
			FROM	players
			GROUP BY decade)
SELECT	decade, 
		avg_height - LAG(avg_height) OVER (ORDER BY decade) AS height_diff,
        avg_weight - LAG(avg_weight) OVER (ORDER BY decade) AS weight_diff
FROM	hw
WHERE	decade IS NOT NULL;
```

#### Key Insights

From the above analysis, we can conclude that
- Certain schools consistently produce MLB talent across decades
- Team payrolls have dramatically increased over time
- A small number of franchises dominate cumulative salary spending
- Average player size has increased steadily across generations
- Long-tenured franchise players are relatively rare

#### Acknowledgements

Baseball historical datasets inspired by the Lahman Baseball Database.
