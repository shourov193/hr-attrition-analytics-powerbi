SELECT
    COUNT(*)                          AS total_employees,
    SUM(Attrition_Num)                AS employees_left,
    COUNT(*) - SUM(Attrition_Num)     AS active_employees,
    ROUND(AVG(Attrition_Num) * 100, 1) AS attrition_rate_pct,
    ROUND(AVG(MonthlyIncome), 0)       AS avg_monthly_income,
    ROUND(AVG(YearsAtCompany), 1)      AS avg_tenure_years
FROM hr_clean;

SELECT
    Department,
    COUNT(*)                           AS total,
    SUM(Attrition_Num)                 AS left_count,
    ROUND(AVG(Attrition_Num)*100, 1)   AS attrition_pct,
    ROUND(AVG(MonthlyIncome), 0)        AS avg_salary,
    ROUND(AVG(JobSatisfaction), 2)      AS avg_job_satisfaction
FROM hr_clean
GROUP BY Department
ORDER BY attrition_pct DESC;

-- THIS IS YOUR HEADLINE FINDING.
-- Overtime employees leave at approximately 3x the rate.
SELECT
    OverTime,
    COUNT(*)                           AS total,
    SUM(Attrition_Num)                 AS left_count,
    ROUND(AVG(Attrition_Num)*100, 1)   AS attrition_pct,
    ROUND(AVG(MonthlyIncome), 0)        AS avg_salary,
    ROUND(AVG(JobSatisfaction), 2)      AS avg_job_satisfaction,
    ROUND(AVG(WorkLifeBalance), 2)      AS avg_work_life_balance
FROM hr_clean
GROUP BY OverTime;
-- Expected result:
-- OverTime=Yes: attrition_pct ~30.5%
-- OverTime=No:  attrition_pct ~10.4%


SELECT
    Salary_Band,
    COUNT(*)                           AS total,
    ROUND(AVG(Attrition_Num)*100, 1)   AS attrition_pct
FROM hr_clean
GROUP BY Salary_Band
ORDER BY FIELD(Salary_Band,'Low (< 3K)','Mid (3K-6K)','High (6K-10K)','Very High (10K+)');


SELECT
    Tenure_Band,
    COUNT(*)                             AS total,
    ROUND(AVG(Attrition_Num)*100, 1)     AS attrition_pct,
    ROUND(AVG(YearsSinceLastPromotion),1) AS avg_yrs_no_promo
FROM hr_clean
GROUP BY Tenure_Band
ORDER BY FIELD(Tenure_Band,'0-2 Years (New)','3-5 Years','6-10 Years','10+ Years');


SELECT
    Department,
    JobRole,
    COUNT(*)                           AS total,
    SUM(Attrition_Num)                 AS left_count,
    ROUND(AVG(Attrition_Num)*100, 1)   AS attrition_pct,
    ROUND(AVG(MonthlyIncome), 0)        AS avg_salary,
    -- Rank each role within its department by attrition rate
    RANK() OVER (PARTITION BY Department ORDER BY AVG(Attrition_Num) DESC) AS risk_rank
FROM hr_clean
GROUP BY Department, JobRole
ORDER BY Department, risk_rank;


USE hr_clean;
 
CREATE OR REPLACE VIEW vw_dept_summary AS
SELECT Department,
    COUNT(*) AS total,
    SUM(Attrition_Num) AS left_count,
    ROUND(AVG(Attrition_Num)*100,1) AS attrition_pct,
    ROUND(AVG(MonthlyIncome),0) AS avg_salary,
    ROUND(AVG(JobSatisfaction),2) AS avg_job_sat
FROM hr_clean GROUP BY Department;
 
CREATE OR REPLACE VIEW vw_role_summary AS
SELECT Department, JobRole,
    COUNT(*) AS total,
    SUM(Attrition_Num) AS left_count,
    ROUND(AVG(Attrition_Num)*100,1) AS attrition_pct,
    ROUND(AVG(MonthlyIncome),0) AS avg_salary
FROM hr_clean GROUP BY Department, JobRole;
 
-- Main view — Power BI connects here
CREATE OR REPLACE VIEW vw_hr_main AS
SELECT * FROM hr_clean;
 
SELECT COUNT(*) FROM vw_hr_main;  -- Must be 1,470
