# Introduction
This project explores the top-paying jobs, in-demand skills, and where demand meets high salary.

# Background
Data from: https://lukebarousse.com/sql
## The questions I wanted to answer through my SQL queries were

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the optimal skills to learn?

# Tools I Used
- **SQL**: The backbone of my analysis, allowing me to query the databse and to generate insights.
- **PostgreSQL**: The chosen database management system.
- **Visual Studio Code**: Data management and executing SQL queries.
- **GitHub**: Used to publish the project.

# The Analysis

### 1. Top Paying Data Analyst Jobs
To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the highest-paying opportunities in the field.
```sql
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY 
    salary_year_avg DESC
```

Here is the breakdown of the top Data Analyst jobs in 2023:
- **Wide Salary Range**: Data analyst jobs ranged from $184,000 - $650,000, indicating significant salary potential in this field.
- **Job Title Variety**: There is a high variety of job titles, from Data Analyst to Director of Data Analyst, indicating various roles and specialisations within Data Analytics.
![Screenshot 2025-02-28 at 1 55 37 AM](https://github.com/user-attachments/assets/9605a17c-8066-45db-8f3a-3a1638695112)
*Bar graph was generated from CHatGPT, from my SQL query results*

### 2. Top Paying Job Skills
To identify the top-paying specific job skills for Data Analysts. I sorted the salary_year_avg in descending order and limited it to the top 10, to find the most important skills for these roles. Linking each job title with: company name, salary and list of required skills helped me to understand which high-paying roles demanded specific skill sets.
```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM   
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
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
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```

![Screenshot 2025-02-28 at 2 40 18 AM](https://github.com/user-attachments/assets/1faee439-be52-4afd-b9e0-a5a314aca78d)

From the table,
- **SQL, Python, and Tableau** are the most frequently required skills across top-paying roles.
- Cloud computing & big data (**AWS, Azure, Snowflake, Databricks**) are crucial for senior roles.
- BI & Data Visualization (**Power BI, Tableau, Excel**) are highly valued in decision-making positions.
- Collaboration & DevOps Tools (**GitLab, Bitbucket, Jira, Confluence**) appear in technical-heavy roles.

### 3. Top Demanded Skills
This Query helped identify the skills most frequently demanded in job postings, directing focus to areas with high demand.
```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5
```
![Screenshot 2025-02-28 at 2 45 12 AM](https://github.com/user-attachments/assets/92272bc6-853d-4cd9-b527-91ffe9184acb)
- **SQl** and **Excel** remained fundamental, indicating a strong need for foundational skills.
- **Programming** and **Visualisation Tools** like **Python** and **Power BI** are essential, showing the importance of technical skills in data visualisation and decision support.

##4. Top Paying Skills
This Query aimed to identify the highest-paying skills in data-related roles based on their average salary
```sql
SELECT 
    skills,
    ROUND (AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25
```

![Screenshot 2025-02-28 at 2 52 30 AM](https://github.com/user-attachments/assets/6d39e7fc-7354-4597-87cc-0797a02c5ac5)

From the results, we can conclude that higher-paying jobs require the individual to know more niche software and skills, which may show why they are for higher-paying jobs in more senior roles, which may not be optimal for a fresh grad to learn in breaking into the Data Analysis industry.

##5. Optimal Skills
From point 4. we have seen the skills required for the highest-paying jobs in the fields, however, this next Query aims to find the most optimal skill for an individual to learn which is most applicable to most roles, that still pay well. The Query considers both demand (number of job postings requiring the skill) and average salary.

```sql
WITH skills_demand AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
    ), average_salary AS (
    SELECT 
        skills_job_dim.skill_id,
        ROUND (AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
    GROUP BY
        skills_job_dim.skill_id
    )

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE
    demand_count>10
ORDER BY
    avg_salary DESC,
    demand_count DESC
    LIMIT 25
```
![Screenshot 2025-02-28 at 3 00 52 AM](https://github.com/user-attachments/assets/8b9f7bd2-2d16-4114-b2e8-42c5f9ec32e5)

High-Demand Programming Languages: Python and R stand out for their high demand, with demand counts of 236 and 148 respectively. Despite their high demand, their average salaries are around $101,397 for Python and $100,499 for R, indicating that proficiency in these languages is highly valued but also widely available.

Cloud Tools and Technologies: Skills in specialized technologies such as Snowflake, Azure, AWS, and BigQuery show significant demand with relatively high average salaries, pointing towards the growing importance of cloud platforms and big data technologies in data analysis.

Business Intelligence and Visualization Tools: Tableau and Looker, with demand counts of 230 and 49 respectively, and average salaries around $99,288 and $103,795, highlight the critical role of data visualization and business intelligence in deriving actionable insights from data.

Database Technologies: The demand for skills in traditional and NoSQL databases (Oracle, SQL Server, NoSQL) with average salaries ranging from $97,786 to $104,534, reflects the enduring need for data storage, retrieval, and management expertise.

# What I Learned
- **Query Crafting**: Through this project, I have learned the basics of query crafting, to generate insights.
- **Generating insights from Queries**: This project has allowed me to turn questions into actionable insights for my data analysis journey.
