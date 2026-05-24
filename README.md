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
- How many schools produced players in each decade?
- Which schools produced the most MLB players?
- What were the top schools per decade?

##### Techniques Used
- COUNT(DISTINCT)
- Decade grouping
- Ranking with window functions

#### PART II — Salary Analysis
Questions Answered
- Which teams are in the top 20% for spending?
- How has team spending accumulated over time?
- When did teams surpass $1 billion in cumulative payroll?

###### Techniques Used
- Cumulative sums
- NTILE()
- Window functions
- Salary aggregation

#### PART III — Player Career Analysis
Questions Answered
- What were players’ starting and ending ages?
- How long were player careers?
- Which teams did players start and end with?
- Which players stayed with the same franchise for over a decade?

###### Techniques Used
- Date calculations
- Career duration analysis
- Multi-table joins
- Player tracking

#### PART IV — Player Comparison Analysis
Questions Answered
- Which players share birthdays?
- What percentage of players bat right, left, or switch for each team?
- How have player height and weight changed over decades?

##### Techniques Used
- Conditional aggregation
- GROUP_CONCAT()
- LAG()
- Trens analysis

#### Example Insights

Some insights generated from the analysis include:
- Certain schools consistently produce MLB talent across decades
- Team payrolls have dramatically increased over time
- A small number of franchises dominate cumulative salary spending
- Average player size has increased steadily across generations
- Long-tenured franchise players are relatively rare

#### Acknowledgements

Baseball historical datasets inspired by the Lahman Baseball Database.
