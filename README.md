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

### 📌 Task 1: Monthly Absenteeism Pattern
Monthly analysis is conducted to identify **seasonal trends in absenteeism** throughout the year. Understanding when absences tend to increase helps the company **anticipate operational risks and prepare backup staffing during peak periods**

<details>
<summary><b>View SQL Code</b></summary>
  
```sql
SELECT
  Month_of_absence_name AS Month,
  COUNT(*) AS Absence_cases,
  SUM(Absenteeism_time_in_hours) AS Absence_hours,
  RANK() OVER (
    ORDER BY SUM(Absenteeism_time_in_hours) DESC) AS Rank_month
FROM `Absence.Absence`
WHERE Month_of_absence_name IS NOT NULL
GROUP BY Month_of_absence, Month_of_absence_name
```
</details>

### Result

<img width="781" height="526" alt="image" src="https://github.com/user-attachments/assets/783ebf99-ccec-4050-9c9f-84e6d071f7b7" />

Based on monthly analysis, employees tend to take more leave during the transition period from **late Q1 to mid Quarter 2.** **March recorded the highest absenteeism** in the year, with 749 absence hours and 87 absence cases in a single month. The next highest months were July and April, both exceeding 400 absence hours and more than 50 absence cases

Workload distribution shows that March and April both **fall into workload level 3** (high workload out of the company’s four levels). Having high absenteeism during these months could **directly affect operational capacity and delivery performance during peak periods**

### Recommendation
Before identifying the exact causes, the company should consider the following preventive measures:
- Hire temporary staff about **two weeks before March, including training time.** Recruitment planning and budget approval should be **prepared at least one month in advance** according to company procedures
- Since **January** has the lowest absence hours and cases, **preparation for peak season** backup staff should ideally start during this period
- Require employees to **notify planned leave at least two weeks in advance,** except for emergency situations. **Leave schedules should be shared within the team** so members can coordinate rotational leave rather than taking leave simultaneously
- Set a guideline that **no more than 20% of a team can be absent on the same day.** For smaller teams, stricter limits may be needed
- In case temporary staff fail to show up, permanent team members may need to absorb the workload. Recruitment and training teams should work to **keep the backup no show rate below 10%** and **overtime (OT) costs** should be **included in the initial planning budget**

### 📌 Task 2: Absenteeism by Day of Week
Analyzing absenteeism by weekday helps determine whether absences are **clustered on specific days** which may reflect workload distribution, employee fatigue or behavioral patterns related to work schedules

<details>
<summary><b>View SQL Code</b></summary>
  
```sql
SELECT
  Day_of_week_name AS Day_of_week,
  COUNT(*) AS Absence_cases,
  SUM(Absenteeism_time_in_hours) AS Absence_hours,
  RANK() OVER (
    ORDER BY SUM(Absenteeism_time_in_hours) DESC) AS Rank_weekday
FROM `Absence.Absence`
WHERE Day_of_week_name IS NOT NULL
GROUP BY Day_of_the_week, Day_of_week_name
```
</details>

### Result

<img width="856" height="238" alt="image" src="https://github.com/user-attachments/assets/6eac2319-de00-48a6-98e3-df552de8875a" />

Analysis by weekday shows that absences tend to occur more frequently at the beginning of the week, both in total cases and hours. Since workload across weekdays varies by month, **no particular weekday consistently carries the highest workload.** Therefore, absenteeism during weekdays may be influenced by other factors such as **health issues, previously approved leave and burnout or fatigue after the weekend**

### Recommendation
Given the fast-paced nature of the courier industry where employees frequently travel and handle large volumes of deliveries, company should **consider mental health and well-being initiatives to support employees**

### 📌 Task 3: Health vs Non-Health Reasons for Absence
This analysis separates absences caused by **health related conditions and non health reasons.** The goal is to understand whether absenteeism is primarily driven by **medical issues or operational and personal factors** which require different management approaches

<details>
<summary><b>View SQL Code</b></summary>
  
```sql
SELECT
  Month_of_absence_name AS Month ,
  CASE WHEN Absence_reason_ID IN (1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21) THEN 'Reason for absence (ICD)'
    ELSE 'Reason for absence without (ICD)' END AS Reason_group,
  COUNT(*) AS Absence_cases,
  SUM(Absenteeism_time_in_hours) AS Absence_hours
FROM `Absence.Absence`
WHERE Month_of_absence_name IS NOT NULL
GROUP BY Month_of_absence, Month_of_absence_name, Reason_group
ORDER BY Month_of_absence,Month_of_absence_name, Absence_hours DESC
```
</details>

### Result

Absence reasons were grouped into two major categories - health related reasons and non-health reasons. Health reasons are classified based on the International Classification of Diseases (ICD) which includes 21 medical categories

Monthly analysis shows that **in 10 out of 12 months, health related reasons account for more absence hours than non-health reasons,** highlighting the importance of employee health monitoring.

**March recorded the highest health related absence** hours at 568 hours, which is **324% higher than the lowest month.** This aligns with March also being the month with the highest absence cases and hours overall

### Recommendation
Before identifying specific medical causes, the company should:
- Promote **health awareness** among employees
- Introduce employee **well-being programs**
- Consider conducting **health checkups twice per year** instead of once, which is common in some Asian countries
- Encourage **physical activities** or fitness programs

For example, the company could **partner with gyms or provide fitness programs based on employee preference surveys.** Coaches could be invited for training sessions or employees could attend gyms directly with monitored check ins. Due to the nature of courier work, employees spend long hours driving or traveling which limits opportunities for physical activity, especially during peak periods

### 📌 Task 4: Major Health Related Causes of Absenteeism
Examining the most common health-related causes helps identify **which medical conditions frequently lead to employee absences.** This allows the company to evaluate potential links between **job demands and employee health risks**. besides that, some absence reasons occur less frequently but result in **longer recovery periods.** Analyzing these cases helps the company understand which conditions may lead to **extended workforce shortages and operational disruptions**

Health related reasons were analyzed under two perspectives:
- Reasons causing the highest number of absence cases per month and 
- Reasons causing the longest absence duration per month
Since the analysis is conducted monthly, the leading reasons may differ between months

#### 4.1 Reasons causing the highest number of absence cases

<details>
<summary><b>View SQL Code</b></summary>

```sql
SELECT
Month_of_absence_name AS Month,
Absence_reason,
COUNT(*) AS Absence_cases
FROM `Absence.Absence`
WHERE Month_of_absence_name IS NOT NULL AND Absenteeism_time_in_hours > 0
GROUP BY Month_of_absence, Month_of_absence_name, Absence_reason
QUALIFY
ROW_NUMBER()
 OVER(PARTITION BY Month_of_absence_name
  ORDER BY COUNT(*) DESC) = 1
ORDER BY Month_of_absence  
```
</details>

### Result

<img width="886" height="520" alt="image" src="https://github.com/user-attachments/assets/c2c4d170-83a9-4d3e-8f0f-2ffaba7561e8" />

The most **frequent absence cases** are mainly related to **recovery or medical visits,** including **physiotherapy/ rehabilitation, dental consultation and general medical consultation**

These conditions typically **require multiple treatment sessions but do not involve long hospital stays,** meaning employees may take multiple short absences rather than long leaves. In many cases, employees may even return to work after half-day appointments

### Recommendation
Because these appointments are usually scheduled in advance, teams can prepare internally by arranging temporary coverage and compensating overtime if necessary

#### 4.2 Reasons causing the longest absence*

<details>
<summary><b>View SQL Code</b></summary>

```sql
SELECT
  Month_of_absence_name AS Month,
  Absence_reason,
  SUM(Absenteeism_time_in_hours) AS Absence_hours
FROM `Absence.Absence`
WHERE Month_of_absence_name IS NOT NULL AND Absenteeism_time_in_hours > 0
GROUP BY Month_of_absence, Month_of_absence_name, Absence_reason
QUALIFY
ROW_NUMBER()
 OVER(PARTITION BY Month_of_absence_name
  ORDER BY SUM(Absenteeism_time_in_hours) DESC) = 1
ORDER BY Month_of_absence
```
</details>

### Result

<img width="1281" height="522" alt="image" src="https://github.com/user-attachments/assets/9f7db28e-c116-4550-baae-64483908bc6c" />

Reasons for longer absences were grouped into three categories:
**- Illness:** skin diseases, respiratory diseases, nervous system diseases, musculoskeletal diseases
**- Injury / physical treatment:** injury or poisoning, physiotherapy
**- Medical visit or unspecified:** medical consultation, unspecified symptoms, other causes

Although the specific causes vary monthly, musculoskeletal diseases and injuries appear frequently. Notably, **March, April and June are among the top five months with the highest absence hours,** largely related to **musculoskeletal conditions or injuries.** This is understandable given the physical nature of courier work, which involves heavy lifting and extensive movement.

### Recommendation

**Ergonomic training**
- Proper lifting techniques
- Correct weight distribution when carrying packages
- Stretching before shifts
- Providing supportive equipment such as carts, back-support belts or anti slip gloves

**Workload management**
- Distribute delivery routes more evenly
- Prevent employees from handling excessive orders
- Monitor average workload by day to detect overly demanding shifts
- Allocate additional staff during high workload months

### 📌 Task 5: Distance from Home to Workplace
Commuting distance is analyzed to assess whether **long travel times influence absenteeism frequency or employee fatigue.** This helps determine whether commuting conditions may indirectly affect workforce availability

<details>
<summary><b>View SQL Code</b></summary>

```sql
-- Absence frequency per employee/ Tần suất nghỉ trung bình trên mỗi nhân viên
-- Formula/Công thức: Frequency(tổng số lần nghỉ)/ Employees(tổng số nhân viên)

SELECT
  CASE
    WHEN Distance_from_Residence_to_Work <= 10 THEN '≤ 10km'
    WHEN Distance_from_Residence_to_Work BETWEEN 11 AND 20 THEN '11-20km'
    WHEN Distance_from_Residence_to_Work BETWEEN 21 AND 30 THEN '21-30km'
    ELSE '> 30km' END AS Distance_group,
  COUNT(DISTINCT Case_ID) AS Total_employees,
  ROUND(AVG(Absenteeism_time_in_hours),2) AS Avg_absence_hours,
  ROUND(COUNT(*)/ COUNT(DISTINCT Case_ID),2) AS Absence_frequency_per_employee 
FROM `Absence.Absence`
WHERE Absenteeism_time_in_hours > 0
GROUP BY Distance_group
ORDER BY Absence_frequency_per_employee DESC
```
</details>

### Result

<img width="1065" height="205" alt="image" src="https://github.com/user-attachments/assets/144db6a0-04c3-4e15-831b-8e1e452d2a6d" />

Employees living closest to the workplace **(≤10 km)** show the highest absence frequency per person, but this group only includes two employees so the sample is too small to draw strong conclusions

Employees living **more than 30 km away** rank third in absence frequency at around 20 absences per person, while the group living at moderate distances shows the lowest absence frequency

### Recommendation
Since commuting distance cannot easily be changed, possible solutions include:
- Creating **multiple smaller pickup hubs** instead of relying on a single warehouse
- Providing **transportation allowances or shuttle services**
- Implementing **shift scheduling adjustments,** such as longer shifts with fewer working days or later start times to avoid peak traffic
- Considering **regional workforce planning,** such as recruiting employees closer to warehouses or opening hubs near employee residential areas

### 📌 Task 6: Workload Levels
Workload analysis evaluates whether employees with **different workload intensities experience different absenteeism patterns.** This helps identify potential operational pressure points and workforce dependency risks

<details>
<summary><b>View SQL Code</b></summary>

```sql
SELECT
  CASE
    WHEN Work_load_Average_by_day < 250000 THEN '200,000 - 249,999'
    WHEN Work_load_Average_by_day < 300000 THEN '250,000 - 299,999'
    WHEN Work_load_Average_by_day < 350000 THEN '300,000 - 349,999' ELSE '350,000 - 399,999' END AS Workload_group,

  CASE
    WHEN Work_load_Average_by_day < 250000 THEN 'Workload level 1'
    WHEN Work_load_Average_by_day < 300000 THEN 'Workload level 2'
    WHEN Work_load_Average_by_day < 350000 THEN 'Workload level 3' ELSE 'Workload level 4' END AS Workload_label,

  COUNT(DISTINCT Case_ID) AS Total_employees,
  ROUND(AVG(Absenteeism_time_in_hours),2) AS Avg_absence_hours,
  ROUND(COUNT(*)/ COUNT(DISTINCT Case_ID),2) AS Absence_frequency_per_employee
FROM `Backup_Workload_Sheet.Backup_workload`
WHERE Work_load_Average_by_day IS NOT NULL AND Absenteeism_time_in_hours > 0
GROUP BY Workload_group, Workload_label
ORDER BY  Workload_group
```
</details>

### Result

<img width="1252" height="199" alt="image" src="https://github.com/user-attachments/assets/0fe09b3c-7315-4fc0-a840-0a32fa92c4dd" />

Workload was divided into **four levels from lowest to highest.** Only **12 employees belong to workload level 4** and they have the **lowest absence frequency among all groups.** This is likely because replacements for these roles are difficult to find

However, their **average absence duration is the highest, 39.2% longer than the company’s average absence duration.** This suggests these employees handle critical tasks that are difficult to replace. They rarely take leave, but when they do, the absence tends to be longer

### Recommendation
To mitigate this risk, the company should:
- Develop a succession pipeline for these roles
- Rotate and train employees from lower levels with potential
- Prevent overburdening existing staff which could lead to higher turnover

If recruiting experienced employees is difficult, the company could hire junior staff to handle smaller tasks so senior employees can focus on specialized responsibilities

### 📌 Task 7: Overall Absence Reasons
Reviewing overall absence reasons provides a **general understanding of the main drivers of absenteeism across the organization.** This helps prioritize which factors require the most attention from management

<details>
<summary><b>View SQL Code</b></summary>

```sql
SELECT
Absence_reason,
  COUNT(DISTINCT Case_ID) AS Total_employees,
  ROUND(AVG(Absenteeism_time_in_hours),2) AS Avg_absence_hours,
  ROUND(COUNT(*)/ COUNT(DISTINCT Case_ID),2) AS Absence_frequency_per_employee 
FROM `Absence.Absence`
WHERE Absenteeism_time_in_hours > 0
GROUP BY Absence_reason
ORDER BY Absence_frequency_per_employee DESC
```
</details>

### Result

<img width="1486" height="637" alt="image" src="https://github.com/user-attachments/assets/2f657dc3-a23e-40ad-b7bc-53dcbb36da12" />

<img width="1507" height="640" alt="image" src="https://github.com/user-attachments/assets/8c4be16c-a26c-4800-ab41-2d180560bb4a" />

Across the company, **physiotherapy** remains the most frequent reason for absence, **although it is 17.8% lower than the average absence cases.** This aligns with the physically demanding nature of courier work

On the other hand, the conditions causing longer absence durations include:
- Nervous system diseases (e.g., migraines, neurological disorders, dizziness)
- Skin diseases
- Neoplasms (tumor-related conditions)
Each of these conditions results in **over 22 absence hours on average,** indicating longer recovery periods despite relatively few affected employees.

### Recommendation
These conditions are generally **not directly related to job characteristics,** so employees may need to manage them personally. However, the company can still promote health awareness and encourage employees to monitor potential health risks

### 📌 Task 8: BMI and Health Absence reason Analysis
BMI is analyzed to explore whether **employees’ physical health indicators are associated with specific medical conditions or absence durations** which may reveal potential health risk patterns within the workforce

<details>
<summary><b>View SQL Code</b></summary>

```sql
SELECT
  CASE 
    WHEN Weight / POWER(Height/100,2) < 20 THEN 'BMI <20'
    WHEN Weight / POWER(Height/100,2) < 25 THEN 'BMI 20-25'
    WHEN Weight / POWER(Height/100,2) < 30 THEN 'BMI 25-30' ELSE 'BMI 30+' END AS BMI_group,
  Absence_reason,
  ROUND(AVG(Absenteeism_time_in_hours),2) AS Avg_absence_hours,
  ROUND(COUNT(*) / COUNT(DISTINCT Case_ID),2) AS Absence_frequency_per_employee
FROM `Absence.Absence`
WHERE Absenteeism_time_in_hours > 0
GROUP BY BMI_group, Absence_reason
HAVING Avg_absence_hours >
  (SELECT Avg_absence_hours
  FROM `Absence.Avg_data`)
ORDER BY BMI_group, Absence_frequency_per_employee DESC
```
</details>

### Result

<img width="1549" height="687" alt="image" src="https://github.com/user-attachments/assets/8f684aef-edf4-48bb-ab89-406e23c78ca7" />

<img width="1561" height="693" alt="image" src="https://github.com/user-attachments/assets/d4e0116d-69ee-46ac-bf02-5a69c222a85e" />

<img width="1567" height="301" alt="image" src="https://github.com/user-attachments/assets/03cdbdad-7fb6-432f-8ead-bb9c4c0660ad" />

Further analysis examined whether **BMI could be a contributing factor** for above health conditions. Results show:
- **Nervous system diseases** occur in both **BMI 20–25 and BMI 30+**, but employees in the **20–25 group tend to have longer absence durations**
- **Skin diseases** affect primarily **BMI 20–25 group** with some cases also appearing in **BMI <20 group**
- **Neoplasms** appear mainly in the BMI **25–30 group**

### Recommendation

Although BMI does not show a strong single pattern across all absence cases, certain health conditions appear more frequently within specific BMI ranges. This suggests that **employee physical health may indirectly influence absenteeism duration and recovery time**

The company could consider implementing **preventive health and wellness initiatives** such as regular health screenings, nutrition awareness programs and physical activity support. Encouraging healthier lifestyle habits may help reduce potential health risks and improve overall employee well-being which could contribute to lower absenteeism over time

## 🔎 Final Conclusion and Recommendation

To reduce absenteeism risks and maintain operational efficiency, the company may consider the following actions:

**Workforce Planning**
- Prepare temporary staffing during historically high absence months
- Establish succession planning for critical workload roles

**Workplace Safety**
- Implement ergonomic training for lifting and handling packages
- Provide supportive equipment to reduce injury risks

**Employee Well-being**
- Introduce regular health screening and wellness programs
- Encourage physical activity and preventive healthcare

**Operational Management**
- Improve workload distribution and route planning
- Monitor workload intensity during peak periods

**Commuting Support**
- Consider transportation allowances, shuttle services or decentralized hubs to reduce commuting burden

These measures can help the company maintain operational stability, reduce absenteeism risks and support long term workforce sustainability
