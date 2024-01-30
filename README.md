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
-  Outcome: Correct errors and inconsistencies in data The dataset is divided into 6 tables including: absence information, dynamics, compensation, demographics, commuting table, reasons with primary keys and foreign keys. 

**Outlier Detection**
-  Action: Use tableau to measure central tendency, distribution shape to highlight the key features of the data
-  Outcome: Data stretches from left to right, with most of the data on the left, Mean > median, => right-skewed distribution, outliers is on the right side

**Apply extreme value analysis method to filter and remove outliers**
-  Action: Use sql to calculate average & standard deviation, upper/lower whiskers.
-  Outcome : Filter and remove 14 outliers

**Filter and remove data that is recorded with zero absence hours for preventing skewing overall statistics**
-  Action: Use sql to isolate data that had zero absence hours 
-  Outcome: Filter and remove 44 data that is recorded with zero absence hours

### Data exploratory 

#### 1. Temporal Patterns and Seasonality in Absences
Assess employee’s absence trend during months. Figure out the month and season that has the most absence hours. Employ SQL to group month and absence hours.

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

### Dashboard
#### Dashboard 0: Outlier Detection
View dynamic [dashboard](https://public.tableau.com/app/profile/thien.nguyen1439/viz/pj2distributiontesting/Dashboard1)

![image](https://github.com/hoanthien2017/Employee_absenteeism_analysis/blob/a7baf9cd788b53b7eec3e7619ef68417a649babe/Dashboard%201%20(2).png)

## Executive Summary
### Conclusion
May, June, and July are months with the highest average number of absence hours (6-6,5 hours)
- These times are characterized by warmer weather and a common preference for vacationing. 
- This period would coincide with summer school vacations, which may require parents to take leave to care for children and engage in family activities.
- May features Memorial Day and July includes Independence Day in the U.S., both significant holidays that may lead to employee absence.
 
38 percent of employee absences are due to medical and dental consultations.
- People are more likely to regular check-ups and early treatment to prevent more serious health issues.
- Many health insurance plans can cover regular medical and dental check-ups without significant out-of-pocket costs.
 
Employees aged 31-50 have absence hours ranging from 16 percent to 33.3 percent higher than those under 30 or over 50.
- Middle age is the peak period for family caregiving activities like child-rearing and caring for aging parents. On the other hand, they also deal with some onset of chronic health conditions.
- Young age has less health problems as well as life responsibilities.
- Over 50 people have less family duties.
 
Employees with high transportation expenses have an average absence time that is 1.59 time higher than for those with low transportation expenses and 1.39 time higher than for those with medium transportation expenses.
- Employees with high transportation expenses may have longer commute times that lead to stress, fatigue and exposure to potential accidents or delays.
- Higher expenses can cause transportation challenges like unreliable public transport, the need for multiple transport modes, or travel during peak hours.

Employees with children have a higher average absenteeism rate, ranging from 23 percent to 42 percent more compared to employees without children. 
- People with children would require more day off to attend to child-related needs, such as illnesses, school events, for childcare arrangements.
- Parents, especially to those with several kids may lead to mental problems like stress and fatigue, that lead to need for mental health breaks.

Employees who drink or smoke have an average absence time that is approximately a 50.57 percent greater than those who do neither.
- Employees who have habits of smoking or drinking would deal with range of health issues that lead to time break for medical appointments or illnesses.
- lifestyle with smoke and drink habit may involve behaviors like staying up late or socializing in ways that don't align with a regular work schedule.
 
People with the highest professional score have an average absence time that is 4.5 times lower than those with the lowest professional score. 
- Employees with high professional scores tend to perform higher job satisfaction, responsibility, and commitment.
- High performance employees are more likely to receive more tangible rewards, which can their motivation to maintain attendance.
- Employees with high professional scores would have goals for better professional development and career advancement, that powerful incentive to remain consistent work schedule and avoid taking leaves.

### Suggestions for Further Improvement
1. Flexible Leave Policies:
- Offer more flexible leave options during May, June, and July so employees can arrange their work and minimize   unexpected, unexplained absences.

2. Workplace Health Services:
- Provide periodic workplace health check-ups to eliminate time away to visit external healthcare facilities. Besides that, this is easier and more convenient for organizations to make schedules and avoid disruption.

3. Family Support Policies:
- Offer programs such as on-site childcare or financial support for childcare to support employees with young children. Adjusting flexible policies for employees with family caregiving responsibilities, like remote work, hybrid shift or shift work options, to accommodate caregiving needs.

4. Travel Assistance:
- Partner with transportation services to minimize travel costs and commuting time for employees. Consider providing travel allowances or remote work options to reduce the daily commute requirement.

5. Mental Health Support:
- launch mental health support programs and creating a supportive work environment to help employees scope with stress and fatigue.

6. Substance Abuse Programs:
- implement substance abuse programs and support for employees with habits of drinking and smoking to improve health and lessen sick leave.

7. Career Development and Performance Reviews:
- Create career development and provide regular feedback and a reward system based on professional scores on job performance to enhance commitment and maintain a consistent work presence.


