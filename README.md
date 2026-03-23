# 📊 HR Analytics: Employee Absenteeism Analysis in Logistics Operations (SQL)

### Author: Vo Tran Mai Khanh
### Date: 02/2026
### Tools Used: SQL

## 📌 Background & Overview

### 📖 Objective - This project uses SQL to analyze absenteeism pattern from courier company to 
- Ensure operational continuity and workforce efficiency by anticipating periods of high absenteeism
- Provide a data-driven basis for backup staffing and workforce planning
- Support succession training and knowledge transfer to reduce dependency on critical roles
- Identify risk factors and patterns of absenteeism affecting operations
- Recommend employee well-being initiatives to reduce health-related absences
- Improve cost control, particularly related to overtime (OT) and temporary staffing

### 👤 Who is this project for?  
- Operations Management Team – to ensure operational continuity and workforce availability
- HR Department – to monitor absenteeism patterns and design employee well-being initiatives
- Workforce Planning Team – to support backup staffing and succession training strategies
- Business Management / Leadership – to support data-driven decisions on overtime costs and operational efficiency

### 📊 Data Source
- Source: courier company  
- Size: 740 records across in one table
- Format: .xls

### 📊 Data Structure & Relationships 

#### 1️⃣ Tables Used: 
Absenteeism_at_work – employee absenteeism dataset including absence timing, ICD-based medical reasons, employee demographics, commuting distance, workload indicators and performance related attributes

#### 2️⃣ Table Schema & Data Snapshot

<img width="1061" height="61" alt="image" src="https://github.com/user-attachments/assets/96e15e6e-44c9-45ae-8539-3e9c4dbc1d94" />

<img width="1011" height="61" alt="image" src="https://github.com/user-attachments/assets/d90febf8-1650-49ca-90ec-a8367a2ff09d" />

## ⚒️ Main Process

1️⃣ Data Cleaning & Preprocessing  
2️⃣ Exploratory Data Analysis (EDA)  
3️⃣ SQL Analysis 

### 📌 Task 1: Monthly Workforce Workload & Absence Trends

#### Query

```sql
SELECT
  Month_of_absence,
  Monthly_Avg_Workload,
CASE 
  WHEN Monthly_Avg_Workload < 250000 THEN 'Level 1 (Low)'
  WHEN Monthly_Avg_Workload > 250000 AND Monthly_Avg_Workload <= 275000 THEN 'Level 2 - Medium'
  WHEN Monthly_Avg_Workload > 275000 AND Monthly_Avg_Workload <= 300000 THEN 'Level 3 - Medium'
  WHEN Monthly_Avg_Workload > 300000 THEN 'Level 4 - High'
ELSE 'Unknow' END AS Monthly_workload_label,
  Total_absence_hours,
  Absence_case
FROM
  (SELECT  
    Month_of_absence,
    CAST(ROUND(AVG(Avg_daily_workload)) AS INT64) AS Monthly_Avg_Workload,
    SUM(Absent_hours) AS Total_absence_hours,
    COUNT(*) AS Absence_case
  FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
  WHERE Month_of_absence > 0 AND Month_of_absence IS NOT NULL
  GROUP BY 1)
ORDER BY 1
```
#### Result

<img width="1131" height="523" alt="image" src="https://github.com/user-attachments/assets/bd594163-b065-40e9-9b65-9c94a589a55f" />

Analysis of the monthly workload data identifies two critical risk peaks in **March (749 hours) and July (724 hours).** While the number of absence cases remains relatively stable throughout the year, the total absence hours during these periods **skyrocket to 2 to 3 times the annual average.** This discrepancy suggests that employees are facing more severe health issues or significant operational disruptions requiring extended recovery time rather than routine, short-term check-ups. Furthermore, the system triggers a Level 4 - High workload label in January despite lower absence hours, serving as an early indicator of operational pressure before more severe crises manifest later in the year. Ultimately, these trends mark a clear transition from short-term leave to long-term clinical treatment, signaling a widespread decline in workforce health during seasonal transitions

### 📌 Task 2: Absence Breakdown by Category

#### Query
  
```sql
SELECT * FROM (
  SELECT 
    Month_of_absence, 
    Absence_type
  FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
  WHERE Month_of_absence > 0)
PIVOT(
  COUNT(*)
  FOR Absence_type IN (
    'Disease' AS Disease, 
    'Medical Consultation' AS Medical_Consultation, 
    'Non-Medical/Disciplinary' AS Non_Medical_or_Disciplinary))
ORDER BY 1
```

#### Result

<img width="885" height="517" alt="image" src="https://github.com/user-attachments/assets/27bc0257-89e8-4d9d-a751-890eaa92df29" />

Analysis of absence types reveals that Disease and Medical Consultation are the overwhelming drivers of workforce unavailability throughout the year. Medical Consultations **peak particularly in February** with 51 cases, while Diseases reach their **highest frequency in March** with 36 cases. In contrast, Non-Medical or Disciplinary absences remain negligible, rarely exceeding a few cases per month with a minor exception in July. This data confirms that the productivity loss is not due to behavioral or disciplinary issues but is a direct consequence of employee health challenges. The simultaneous high volume of both medical consultations and active disease cases suggests a reactive environment where health issues are addressed only once they become severe rather than through proactive wellness management

### 📌 Task 3: Productivity Gap and Extra Workload Assessment (FTE)

#### Query
  
```sql
SELECT
  Month_of_absence,
  Disease_Hours,
  ROUND(Disease_Hours/176,2) AS Extra_Workload_Disease,
  Medical_Hours,
  ROUND(Medical_Hours/176,2) AS Extra_Workload_Medical
FROM
  (SELECT 
    Month_of_absence,
    SUM(CASE WHEN Absence_type = 'Disease' THEN Absent_hours ELSE 0 END) AS Disease_Hours,
    SUM(CASE WHEN Absence_type = 'Medical Consultation' THEN Absent_hours ELSE 0 END) AS Medical_Hours,
    SUM(Absent_hours) AS Total_Hours
  FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
  WHERE Month_of_absence > 0
  GROUP BY 1)
ORDER BY 1
```
#### Result

<img width="1192" height="526" alt="image" src="https://github.com/user-attachments/assets/208f9e8a-c063-4f1e-936c-2db2ad2a8aa5" />

The conversion of total absence hours into Full-Time Equivalent (FTE) units identifies record productivity **gaps in March (3.23 FTE) and July (3.06 FTE).** This deficit far exceeds the tolerance threshold of a standard team, losing the equivalent of three full-time employees simultaneously creates a critical operational vacuum. Because this missing capacity must be absorbed by the remaining staff, it creates a cycle of excessive overtime and mounting pressure, directly correlating with high rates of burnout reported during these periods. This data demonstrates that the absenteeism recorded is not merely a series of individual incidents but a collective operational crisis that leaves the workforce consistently under-resourced during seasonal peaks

### 📌 Task 4: Deep-dive into Occupational Health and Operational Risk

#### Query

```sql
SELECT
  Month_of_absence,
  Reason_detail,
  'Highest absence cases' AS Insight_Type,
  COUNT(*) AS Metric_value,
  SUM(Absent_hours) AS Total_hours,
  ROUND((SUM(Absent_hours)/176),2) AS Extra_workload_FTE,
  CASE 
		WHEN ROUND((SUM(Absent_hours)/176),2) > 1.0 THEN 'High Risk: Hiring/Backup Needed'
    WHEN ROUND((SUM(Absent_hours)/176),2) BETWEEN 0.5 AND 1.0 THEN 'Medium Risk: OT Expected'
    ELSE 'Low Risk: Team Cover' END AS Operational_Advice
FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
WHERE Absence_type = 'Disease' AND Month_of_absence IN(3,4,5,7)
GROUP BY 1,2,3
QUALIFY RANK() OVER(PARTITION BY Month_of_absence ORDER BY (Metric_value) DESC)=1

UNION ALL

SELECT
  Month_of_absence,
  Reason_detail,
  'Longest absence hours' AS Insight_Type,
  CAST(ROUND(AVG(Absent_hours),2) AS INT64) AS Metric_value,
  SUM(Absent_hours) AS Total_hours,
  ROUND(SUM(Absent_hours)/176,2) AS Extra_workload_FTE,
  CASE 
		WHEN (SUM(Absent_hours)/176) > 1.0 THEN 'High Risk: Hiring/Backup Needed'
    WHEN (SUM(Absent_hours)/176) BETWEEN 0.5 AND 1.0 THEN 'Medium Risk: OT Expected'
    ELSE 'Low Risk: Team Cover' END AS Operational_Advice
FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
WHERE Absence_type = 'Disease' AND Month_of_absence IN(3,4,5,7)
GROUP BY 1,2,3
QUALIFY RANK() OVER(PARTITION BY Month_of_absence ORDER BY (Metric_value) DESC)=1
ORDER BY 1
```
#### Result

<img width="2004" height="444" alt="image" src="https://github.com/user-attachments/assets/314fdef8-d465-470b-a5ef-0738b6e1cb1e" />

Detailed diagnostic data identifies **musculoskeletal diseases and physical injuries** as the **primary drivers of long-term workforce unavailability.** In logistics and delivery sector, these conditions are frequently linked to high intensity manual handling and repetitive physical tasks. The data highlights a critical peak in **April** where musculoskeletal issues alone **accounted for 200 absence hours,** creating a staggering **1.14 FTE gap.** This specific health risk necessitates a high risk operational classification, as a deficit of this magnitude cannot be absorbed by existing team members without specialized backup or additional hiring. Unlike seasonal illnesses, these physical conditions result in prolonged recovery periods, making them the most significant threat to sustained operational stability and indicating an urgent need for targeted ergonomic interventions

### 📌 Task 5: Impact of Commuting Barriers on Recovery Time

#### Query

```sql
WITH Categorized_Data AS (
    SELECT 
        Month_of_absence,
        Absence_type,
        Absent_hours,
        CASE 
            WHEN Transport_group = '> 250' THEN 'High Transport Cost' 
            WHEN Transport_group = '< 150' THEN 'Low Transport Cost'
            WHEN Transport_group = '150 - 250' THEN 'Normal Transport Cost'
            ELSE 'Unknown' 
        END AS Transport_Category
    FROM `hr-operations-analysis.Workforce.Absenteeism_Clean`
    WHERE Absence_type = 'Disease' 
      AND Month_of_absence IN (3, 4, 5, 7)
),
Aggregated_Metrics AS (
SELECT 
	Month_of_absence,
	Transport_Category,
	Absence_type,
  COUNT(*) AS Total_Cases,
  SUM(Absent_hours) AS Total_Disease_Hours,
  ROUND(AVG(Absent_hours), 2) AS Avg_Hours_Per_Case,
  ROUND(SUM(Absent_hours) / 176, 2) AS Extra_Workload
    FROM Categorized_Data
    GROUP BY 1, 2, 3
)
SELECT *,
	CASE 
	  WHEN Extra_Workload > 1.0 THEN 'High Risk: Hiring/Backup Needed'
    WHEN Extra_Workload BETWEEN 0.5 AND 1.0 THEN 'Medium Risk: OT Expected'
    ELSE 'Low Risk: Team Cover' END AS Operational_Advice
FROM Aggregated_Metrics
ORDER BY Month_of_absence ASC, Extra_Workload DESC
```
#### Result

<img width="1849" height="529" alt="image" src="https://github.com/user-attachments/assets/43fd2421-315b-46ed-98bb-da9bca0f55cd" />

There is a significant inverse correlation between transport costs and the speed of an employee's return to work. Analysis of the transport categories reveals that High transport cost group - those living furthest from facility - recorded an average recovery time of **23.11 hours per case in March,** which is **60% higher than Normal group's 14.21 hours.** For staff recovering from labor intensive injuries common in logistics, distance acts as a substantial psychological and physical barrier, these employees are far more likely to extend their leave to avoid a long, painful, or costly commute. Consequently, transport cost becomes a critical catalyst variable that inflates the total FTE gap, indicating that recovery is often hindered as much by commuting difficulty as by the medical condition itself. This data justifies the need for targeted travel support or flexible return to-work policies for employees in high distance categories

## 🔎 Final Conclusion and Recommendation

### Summary
The analysis reveals a critical operational vulnerability where seasonal peaks in March (3.23 FTE loss) and July (3.06 FTE loss) are driven by severe health issues rather than routine absences. The intersection of musculoskeletal injuries (1.14 FTE gap) and high transport costs (60% longer recovery time) creates a double barrier to productivity. To maintain operational stability, the company must shift from reactive leave management to proactive risk mitigation

### Recommendation

**- Operational Ergonomics:** invest in mechanical lifting aids, trolleys, automated conveyors to address the root cause of musculoskeletal diseases, specifically targeting 200-hour deficit recorded in April

**- Safe Manual Handling Training:** implement mandatory workshops on proper lifting techniques for delivery and warehouse staff to reduce the frequency of spinal injuries and back pain

**- Contingent Resource Planning:** establish a flexible talent pool of part-time or gig workers to bridge the identified 3.0+ FTE gaps in March and July, preventing burnout and excessive overtime for the permanent core team

**- Targeted Commuting Support:** provide travel subsidies (public transportation vouchers) for High transport cost group during their first 3 days back from sick leave. This aims to reduce their 23.11-hour recovery average down to the 14.21-hour baseline

**Job Rotation Policy:** allow employees returning from physical injuries to temporarily transition to lighter duties (e.g., inventory tagging or dispatch coordination) for the first week, ensuring they return to work sooner without risking re-injury
