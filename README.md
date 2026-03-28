# Hospital-Patient-Flow-Operational-Analysis
# Business Context
 ### Hospital leadership is concerned that patients are experiencing delays, emergency departments feel overcrowded at certain times, and some departments seem to struggle more than others with patient throughput. As a healthcare data analyst is to investigate the hospital's patient admissions and discharge data and provide those answers:
 * Where pressure is entering the hospital; which routes, departments or patient types?
 * How long patients are staying; and whether that varies in ways that signal inefficiency.
 * Where patients are getting “stuck”; bottlenecks in the flow from admission to discharge.
 * When congestion is most likely to occur; by hour, by day, any visible/hidden patterns.
   #### As a Healthcare Data Analyst who has just been handed access to three operational tables from the hospital's database. I  did not design these tables. I query it intelligently, extract meaningful patterns, and explain what those patterns mean to someone who will never look at a SQL query.
### Dataset:
* patients table:
* admissions table:
* diagnosis table:
  ## Project Objective
### Produce an Operational Patient Flow Analysis that helps hospital leadership understand. How efficiently patients move through the hospital, and where improvements may be needed:

## Business Questions to Answer
1. How many patients are admitted to the hospital each day?
```sql
   SELECT 
		    Date(admission_datetime) AS admission_date,
		    COUNT(*) AS patients_count
FROM admissions
GROUP BY admission_date
ORDER BY patients_count DESC;
```
2. What is the overall average length of stay (LOS) for admitted patients?
   ```sql
   SELECT 
	     ROUND(AVG(EXTRACT( EPOCH FROM(discharge_datetime - admission_datetime))/86400),2) AS avg_LOS
   FROM admissions;
   ```
3. How does average length of stay vary by department?
   ```sql
   SELECT 
	        department,
	        ROUND(AVG(EXTRACT( EPOCH FROM(discharge_datetime - admission_datetime))/86400),2) AS avg_LOS
   FROM admissions
   GROUP BY department
   ORDER BY avg_LOS DESC;
   ```
4. How are admissions distributed by admission type?
   ```sql
   SELECT 
	      admission_type,
	      COUNT(*) AS admission_count
   FROM admissions
   GROUP BY admission_type
   ORDER BY admission_count DESC;
   ```
5. Which admission type is associated with the longest average length of stay?
   ```sql
   SELECT 
      	admission_type,
      	ROUND(AVG(EXTRACT( EPOCH FROM(discharge_datetime - admission_datetime))/86400),2) AS avg_LOS
   FROM admissions
   GROUP BY admission_type
   ORDER BY avg_LOS DESC;
   ```
6. How do discharge dispositions break down across all admissions?
   ```sql
   SELECT 
      	discharge_disposition,
      	COUNT(*) AS total_count,
      	ROUND((count(*)*100) / (SELECT count(*) FROM admissions),2) AS disposition_pct
   FROM admissions
   GROUP BY discharge_disposition
   ORDER BY disposition_pct DESC;
   ```
7. Do certain discharge dispositions correspond to longer hospital stays?
   ```sql
   SELECT 
      	discharge_disposition,
      	COUNT(*) AS total_count,
      	ROUND((COUNT(*) * 100.00) / (SELECT COUNT(*) FROM admissions),2) AS disposition_pct,
      	ROUND(AVG(EXTRACT( EPOCH FROM (discharge_datetime - admission_datetime))/ 86400),2) AS avg_LOS
   FROM admissions
   GROUP BY discharge_disposition
   ORDER BY  avg_LOS DESC;
   ```
8. At what hours of the day do most admissions occur?
   ```sql
   SELECT 
        EXTRACT(HOUR FROM admission_datetime) AS hour_24,
        TO_CHAR(admission_datetime, 'HH12 AM') AS hour_12,
        COUNT(*) AS admission_count
   FROM admissions
   GROUP BY hour_24, hour_12
   ORDER BY hour_24; 
   ```
9. Are there specific days of the week with consistently higher admissions?
    ```sql
    SELECT
      	EXTRACT(DOW FROM admission_datetime) AS day_num,
      	TO_CHAR(admission_datetime, 'Day') AS day_name,
      	COUNT(*) AS patients_count
    FROM admissions
    GROUP BY day_num, day_name
    ORDER BY patients_count DESC;
    ```
10. Which departments experience the highest admission volume?
    ```sql
      SELECT 
      	Department,
      	COUNT(*) AS patients_count
     FROM admissions
    GROUP BY department
    ORDER BY patients_count DESC
    LIMIT 5;
    ```
11. How does patient age relate to length of stay?
    ```sql
    SELECT 
      	CASE
      		WHEN p.age  BETWEEN 0 AND  18 THEN 'Pediatric'
      		WHEN p.age BETWEEN 19 AND  35 THEN 'Young Adult'
      		WHEN p.age BETWEEN 36 AND 50 THEN 'Adult'
      		WHEN p.age BETWEEN 51 AND 65 THEN 'Middle Aged'
      		ELSE 'Ederly'
      	END AS age_group,
      	ROUND(AVG(EXTRACT( EPOCH FROM(a.discharge_datetime - a.admission_datetime))/86400),2) AS avg_LOS,
      	COUNT(*) AS patients_count
    FROM admissions a
    LEFT JOIN patients p
    USING(patient_id)
    WHERE a.discharge_datetime >= a.admission_datetime
    GROUP BY age_group
    ORDER BY age_group;
    ```
12. Are older patients more likely to be discharged to rehab or skilled nursing?
    ```sql
    SELECT 
    		 a.discharge_disposition,
        CASE 
            WHEN p.age BETWEEN 0 AND 18 THEN 'Pediatric'
            WHEN p.age BETWEEN 19 AND 35 THEN 'Young Adult'
            WHEN p.age BETWEEN 36 AND 50 THEN 'Adult'
            WHEN p.age BETWEEN 51 AND 65 THEN 'Middle Aged'
            ELSE 'Elderly'
        END AS age_group,
        COUNT(p.patient_id) AS patient_count
    FROM admissions a
    JOIN patients p
    USING(patient_id)
    WHERE discharge_disposition IN ('Skilled Nursing', 'Rehab') AND p.age > 65
    GROUP BY age_group, a.discharge_disposition
    ORDER BY age_group, patient_count DESC;
    ```
13. Which diagnoses are most commonly associated with hospital admissions?
    ```sql
    SELECT 
         	d.icd10_code, 
         	COUNT(*) AS patients_cnt
    FROM admissions a
    LEFT JOIN diagnosis d
    USING(admission_id)
    GROUP BY d.icd10_code
    ORDER BY patients_cnt DESC;
    ```
14. Do certain diagnoses result in longer average hospital stays?
    ```sql
    SELECT 
         	d.icd10_code, 
         	COUNT(*) AS patients_cnt,
          ROUND(AVG(EXTRACT( EPOCH FROM(a.discharge_datetime - a.admission_datetime))/86400),2) AS avg_LOS
    FROM admissions a
    LEFT JOIN diagnosis d
    USING(admission_id)
    WHERE a.discharge_datetime IS NOT NULL
    GROUP BY d.icd10_code
    ORDER BY avg_LOS DESC;
    ```
