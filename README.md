# Employee_absenteeism_analysis
# Analysis of the Effectiveness of Marketing Campaigns
## Introduction
The purpose of the project is to analyze employee absence of a company.
### Objective
The project only analyzes data on employees who have ever taken a leave of absence and gives suggestions for human management strategy.  The key focus areas include filtering deep relevant data, determining absence trends, identifying main factors on both professional and personal factors.
## Project Analysis
### Data preparation
**Data cleaning**
-  Action: Adjust data format, standardize data format, use ifs function, use choose function to convert ordinal data to nominal data, create primary keys and foreign keys.
-  Outcome: Date format is changed from dd/mm/yyyy to yyyy-mm-dd. The dataset is divided into 6 tables including: absence information, dynamics, compensation, demographics, commuting table, reasons with primary keys and foreign keys. 

**Outlier Detection**
-  Action: Use tableau to measure central tendency, distribution shape to highlight the key features of the data
-  Outcome: Data stretches from left to right, with most of the data on the left, Mean > median, => right-skewed distribution, outliers is on the right side

**Applying extreme value analysis method to filter and remove outliers**
-  Action: Use sql to calculate average & standard deviation, upper/lower whiskers.
-  Outcome : Filter and remove 14 outliers

**Filtering and removing data that is recorded with zero absence hours for preventing skewing overall statistics**
-  Action: Use sql to isolate data that had zero absence hours 
-  Outcome: Filter and remove 44 data that is recorded with zero absence hours

### Data exploratory 

#### 1. Temporal Patterns and Seasonality in Absences
Assess employee’s absence trend during month and season. Figure out the month and season that has the most absence hours. Employ SQL to group month, season, and absence hours.

#### 2. Top 10 reasons for employee absences 
Investigate the most common absence reasons . HR can develop policies and interventions to address specific absence reasons and promote a healthier work environment.

#### 3. Absence time by age groups 
Explore the change of absence time trend over 4 age groups. HR can offer health and wellness programs specifically targeted at this age group.

#### 4. Assessing to commuting impact
Determine whether the distance from residence and transportation expenses are significant factors in employee absences. It is helpful in making policies that may include remote work options, flexible scheduling, or transportation subsidies. Use Case function on SQL to group commuting expenses and distance from home to work, access disparity in absence hour based on commuting factors.

####  5. Absence time by body mass index
Examine differences in absence rates based on body mass index. HR can design programs to address health concerns related to BMI, potentially reducing health-related absences. Use SQL to convey index score into 3 levels, and extract findings of absence hours based on these 3 levels.

#### 6. Absence time based on son and pet
Analyst how employee absence time changes between having kids and pets and not having kids and pets. Recognizing the impact of family and pet responsibilities on absence can help improve employee engagement. 

#### 7. Absence time by social smoke and drink
Investigate the impact of unhealthy habits on absence rate. HR can create a workplace culture that promotes healthier lifestyles, potentially reducing the health-related absences of employees.
 
#### 8. Explore the relationship between service time, workload, target achievement, and absence time
Examine employee absence hours based on working factors. HR can adjust workloads, set realistic targets, and implement strategies to eliminate employee absence. Combine service time, and workload per day, hit target to create professional scores from 1 to 10,  evaluate absence time based on the professional scores.

#### 9. Compensation Influence on Absence
Analyze whether compensation level affects absence rate. It helps in designing competitive compensation packages that motivate employees to maintain better attendance. Use SQL to divide compensation into 3 levels to access differences in employee absence by each level

### Dashboards
#### Dashboard 0: Outlier Detection
View dynamic [dashboard](https://public.tableau.com/app/profile/thien.nguyen1439/viz/pj2distributiontesting/Dashboard1)

![image](https://github.com/hoanthien2017/Employee_absenteeism_analysis/blob/a7baf9cd788b53b7eec3e7619ef68417a649babe/Dashboard%201%20(2).png)

#### dashboard 1: Overview Absence Analysis 
view dynamic [dashboard](https://public.tableau.com/app/profile/thien.nguyen1439/viz/db1pj2/Dashboard1)

![image](https://github.com/hoanthien2017/Employee_absenteeism_analysis/blob/837ca595c11ffb95121e592f0eed61e41eeecbc4/Dashboard%201%20(1).png)

#### dashboard 2: Detailed Absence Analysis 
view dynamic [dashboard](https://public.tableau.com/app/profile/thien.nguyen1439/viz/db2pj2/Dashboard1)

![image](https://github.com/hoanthien2017/Employee_absenteeism_analysis/blob/f0dfd0234d24f7409b35e10ab56bcd161300609d/Dashboard%201.png)


