# Data Analyst Job Market SQL Analysis

## Introduction

This project explores the data analyst job market using SQL. The analysis focuses on remote Data Analyst roles, salary trends, in-demand skills, and the skills that offer the best balance between demand and pay.

SQL queries are available in the [project_sql folder](project_sql/).

## Background

The goal of this project is to better understand what employers are looking for in Data Analyst roles and which skills can improve job opportunities. The dataset includes job postings, companies, salaries, locations, and skills connected to each posting.

The analysis answers five main questions:

1. What are the top-paying remote Data Analyst jobs?
2. What skills are required for the top-paying Data Analyst jobs?
3. What skills are most in demand for Data Analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn based on demand and salary?

## Tools Used

- **SQL:** Used to query, join, aggregate, and analyze the job posting data.
- **PostgreSQL:** Database system used for storing and querying the dataset.
- **pgAdmin / Visual Studio Code:** Used to write and run SQL queries.
- **Git & GitHub:** Used for version control and project sharing.

## The Analysis

Each SQL file in `project_sql` answers one analysis question.

### 1. Top-Paying Data Analyst Jobs

File: [project_sql/1.sql](project_sql/1.sql)

This query identifies the top 10 highest-paying remote Data Analyst roles where salary data is available. It joins job postings with company data so the output includes the company name.

```sql
SELECT 
    job_id,
    job_title,
    job_location,
    salary_year_avg,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN 
    company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

This helps show which remote Data Analyst roles offer the highest salary potential and which companies are hiring for those roles.

### 2. Skills Required for Top-Paying Jobs

File: [project_sql/2.sql](project_sql/2.sql)

This query first finds the top-paying remote Data Analyst jobs, then joins those jobs to the skills tables to show which skills are required for those roles.

```sql
WITH top_paying_jobs AS
(   SELECT 
        job_id,
        job_title,
        job_location,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN 
        company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE 
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)
SELECT 
    top_paying_jobs.*,
    skills_dim.skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC;
```

This shows which technical skills appear in the highest-paying opportunities.

### 3. Most In-Demand Skills

File: [project_sql/3.sql](project_sql/3.sql)

This query counts how often each skill appears in Data Analyst job postings. The file includes both all Data Analyst postings and remote-only postings.

```sql
SELECT 
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND job_work_from_home = TRUE
GROUP BY skills
ORDER BY demand_count DESC
LIMIT 5;
```

This helps identify the skills that appear most often in Data Analyst job postings.

### 4. Skills Based on Salary

File: [project_sql/4.sql](project_sql/4.sql)

This query calculates the average salary for each skill in Data Analyst postings. It helps identify which skills are linked to higher-paying roles.

```sql
SELECT 
    skills_dim.skills,
    ROUND(AVG(salary_year_avg), 2) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY skills
ORDER BY avg_salary DESC
LIMIT 25;
```

This analysis is useful for comparing skills by salary impact, not just popularity.

### 5. Most Optimal Skills to Learn

File: [project_sql/5.sql](project_sql/5.sql)

This query combines demand and salary by finding skills that appear in more than 10 remote Data Analyst job postings and calculating their average salary.

```sql
SELECT 
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(salary_year_avg), 2) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY skills_dim.skill_id
HAVING COUNT(skills_job_dim.job_id) > 10
ORDER BY avg_salary DESC, demand_count DESC
LIMIT 25;
```

This query helps find skills that are both practical to learn and valuable in the job market.

## What I Learned

- How to use `JOIN` to combine job postings, company data, and skill data.
- How to use `GROUP BY`, `COUNT()`, and `AVG()` to summarize job market trends.
- How to use CTEs with `WITH` to break complex analysis into readable steps.
- How to filter results with `WHERE` and `HAVING` depending on whether the condition applies before or after aggregation.
- How to rank results using `ORDER BY` and `LIMIT`.

## Conclusions

This project shows how SQL can be used to answer practical job market questions. The analysis focuses on salary, demand, and skill requirements for Data Analyst roles.

Key takeaways:

1. Remote Data Analyst roles can have strong salary potential when salary data is available.
2. Top-paying roles often require technical skills that go beyond basic spreadsheet work.
3. In-demand skills are useful for employability, while salary-based analysis shows which skills may be more financially valuable.
4. The most optimal skills are those that combine high demand with strong average salaries.
5. SQL is central to this analysis and remains an important skill for data analyst work.
