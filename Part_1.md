1. a. Which prescriber had the highest total number of claims (totaled over all drugs)? Report the npi and the total number of claims.
  b. Repeat the above, but this time report the nppes_provider_first_name, nppes_provider_last_org_name,  specialty_description, and the total number of claims.

SELECT p1.npi, 
	p2.nppes_provider_first_name || 'BUTT', 
	p2.nppes_provider_last_org_name,  
	p2.specialty_description, 
	SUM(p1.total_claim_count) AS total_claims
FROM prescription AS p1
JOIN prescriber AS p2
ON p1.npi = p2.npi
GROUP BY 1,2,3,4
ORDER BY 5 DESC

2. a. Which specialty had the most total number of claims (totaled over all drugs)?

SELECT   
	p2.specialty_description, 
	SUM(p1.total_claim_count) AS total_claims
FROM prescription AS p1
LEFT JOIN prescriber AS p2
ON p1.npi = p2.npi
GROUP BY 1
ORDER BY 2 DESC

    b. Which specialty had the most total number of claims for opioids?
	
SELECT 
	p2.specialty_description, 
	TO_CHAR(SUM(p1.total_claim_count),'fm999G999') AS total_claims
FROM prescription AS p1
LEFT JOIN prescriber AS p2
ON p1.npi = p2.npi
LEFT JOIN drug AS d
ON p1.drug_name = d.drug_name
WHERE d.opioid_drug_flag = 'Y'
GROUP BY 1
ORDER BY 2 DESC


    c. **Challenge Question:** Are there any specialties that appear in the prescriber table that have no associated prescriptions in the prescription table?
	
SELECT specialty_description, COUNT(drug_name)
FROM prescriber
LEFT JOIN prescription 
USING(npi)
GROUP BY 1
HAVING COUNT(drug_name) = 0


    d. **Difficult Bonus:** *Do not attempt until you have solved all other problems!* For each specialty, report the percentage of total claims by that specialty which are for opioids. Which specialties have a high percentage of opioids?

3. a. Which drug (generic_name) had the highest total drug cost?
SELECT d.generic_name, 
	   SUM(p.total_drug_cost)::MONEY
FROM drug AS d
LEFT JOIN prescription AS p
ON d.drug_name = p.drug_name
WHERE p.total_drug_cost IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC


    b. Which drug (generic_name) has the hightest total cost per day? **Bonus: Round your cost per day column to 2 decimal places. Google ROUND to see how this works.
SELECT generic_name, 
	ROUND(SUM(total_drug_cost)/SUM(total_day_supply), 2)
FROM prescription
INNER JOIN drug
USING (drug_name)
GROUP BY generic_name
ORDER BY 2 DESC;


4. a. For each drug in the drug table, return the drug name and then a column named 'drug_type' which says 'opioid' for drugs which have opioid_drug_flag = 'Y', says 'antibiotic' for those drugs which have antibiotic_drug_flag = 'Y', and says 'neither' for all other drugs.
SELECT drug_name, 
	CASE WHEN opioid_drug_flag = 'Y' THEN 'opioid'
	WHEN antibiotic_drug_flag = 'Y' THEN 'antibiotic'
	ELSE 'neither' END
	AS drug_type
FROM drug

    b. Building off of the query you wrote for part a, determine whether more was spent (total_drug_cost) on opioids or on antibiotics. Hint: Format the total costs as MONEY for easier comparision.
SELECT SUM(prescription.total_drug_cost)::MONEY,
	CASE WHEN opioid_drug_flag = 'Y' THEN 'opioid'
	WHEN antibiotic_drug_flag = 'Y' THEN 'antibiotic'
	ELSE 'neither' END
	AS drug_type
FROM drug
LEFT JOIN prescription
ON prescription.drug_name = drug.drug_name
GROUP BY drug_type

5. a. How many CBSAs are in Tennessee? **Warning:** The cbsa table contains information for all states, not just Tennessee.

SELECT DISTINCT cbsa, cbsaname
FROM cbsa
WHERE cbsaname LIKE '%TN%';

SELECT
	COUNT(DISTINCT(c.cbsa))
FROM fips_county AS f
JOIN cbsa c ON f.fipscounty = c.fipscounty
WHERE f.state = 'TN';



    b. Which cbsa has the largest combined population? Which has the smallest? Report the CBSA name and total population.
SELECT DISTINCT cbsa, cbsaname, SUM(population)
FROM cbsa
INNER JOIN population
USING (fipscounty)
WHERE cbsaname LIKE '%TN%'
GROUP BY 1,2
ORDER BY 3

    c. What is the largest (in terms of population) county which is not included in a CBSA? Report the county name and population.
SELECT fips_county.county, population
FROM population
LEFT JOIN fips_county
ON fips_county.fipscounty = population.fipscounty
	WHERE population.fipscounty NOT IN
	(SELECT fipscounty
	FROM cbsa)
ORDER BY population DESC;

6. 
    a. Find all rows in the prescription table where total_claims is at least 3000. Report the drug_name and the total_claim_count.
SELECT drug_name, total_claim_count
FROM prescription
WHERE total_claim_count >= 3000
    b. For each instance that you found in part a, add a column that indicates whether the drug is an opioid.
SELECT prescription.drug_name, prescription.total_claim_count, drug.opioid_drug_flag
FROM prescription
	LEFT JOIN drug
	ON prescription.drug_name = drug.drug_name
WHERE prescription.total_claim_count >= 3000;

    c. Add another column to you answer from the previous part which gives the prescriber first and last name associated with each row.
SELECT prescription.drug_name, prescription.total_claim_count, drug.opioid_drug_flag, nppes_provider_last_org_name, nppes_provider_first_name
FROM prescription
	LEFT JOIN drug
	ON prescription.drug_name = drug.drug_name
	LEFT JOIN prescriber
	ON prescription.npi = prescriber.npi
WHERE prescription.total_claim_count >= 3000;



7. The goal of this exercise is to generate a full list of all pain management specialists in Nashville and the number of claims they had for each opioid.
    a. First, create a list of all npi/drug_name combinations for pain management specialists (specialty_description = 'Pain Managment') in the city of Nashville (nppes_provider_city = 'NASHVILLE'), where the drug is an opioid (opiod_drug_flag = 'Y'). **Warning:** Double-check your query before running it. You will likely only need to use the prescriber and drug tables.
SELECT npi, drug_name
FROM prescriber
CROSS JOIN drug
WHERE nppes_provider_city = 'NASHVILLE'
	AND specialty_description = 'Pain Management'
	AND opioid_drug_flag = 'Y'

    b. Next, report the number of claims per drug per prescriber. Be sure to include all combinations, whether or not the prescriber had any claims. You should report the npi, the drug name, and the number of claims (total_claim_count).
SELECT npi, drug_name, COALESCE(total_claim_count, 0)
FROM(SELECT npi, drug_name
	FROM prescriber
	CROSS JOIN drug
	WHERE nppes_provider_city = 'NASHVILLE'
	AND specialty_description = 'Pain Management'
	AND opioid_drug_flag = 'Y') AS a
LEFT JOIN prescription
USING(npi, drug_name)
ORDER BY npi;


    c. Finally, if you have not done so already, fill in any missing values for total_claim_count with 0. Hint - Google the COALESCE function.
		SELECT npi, drug_name, COALESCE(total_claim_count, 0)
	FROM
	(
		SELECT npi, drug_name
		FROM prescriber
		CROSS JOIN drug
		WHERE nppes_provider_city = 'NASHVILLE'
			AND specialty_description = 'Pain Management'
			AND opioid_drug_flag = 'Y'
	) AS a
		LEFT JOIN prescription
			USING(npi, drug_name)
	ORDER BY npi;


PART 2
	
1. How many npi numbers appear in the prescriber table but not in the prescription table?
	
SELECT COUNT(npi)
FROM prescriber
WHERE NOT EXISTS (
    SELECT npi
    FROM prescription
    WHERE prescriber.npi = prescription.npi);

Ross
SELECT COUNT(*)
FROM
(
	SELECT pr.npi AS provider_NPI,
		pn.npi AS claim_npi
	FROM prescriber pr
		LEFT JOIN prescription pn
			ON pr.npi = pn.npi
	WHERE pn.npi IS NULL
) npi_both;

Habeeb
SELECT (SELECT COUNT(DISTINCT npi) AS npi_count
		FROM prescriber) - 
		(SELECT COUNT(DISTINCT npi) AS npi_count
		FROM prescription) AS npi_difference


2.
    a. Find the top five drugs (generic_name) prescribed by prescribers with the specialty of Family Practice.
SELECT COUNT(p1.npi), specialty_description, generic_name, p2.drug_name
FROM prescriber AS p1
LEFT JOIN prescription as p2
ON p1.npi = p2.npi
LEFT JOIN drug as d
on p2.drug_name = d.drug_name
WHERE specialty_description='Family Practice'
GROUP BY 2,3,4
ORDER BY COUNT DESC
LIMIT 5;


SELECT drug_name, COUNT(drug_name)
FROM prescription AS p
WHERE npi IN
	(SELECT npi
	 FROM prescriber
	 WHERE specialty_description='Family Practice')
GROUP BY drug_name
ORDER BY COUNT DESC
LIMIT 5;
	

    b. Find the top five drugs (generic_name) prescribed by prescribers with the specialty of Cardiology.
SELECT COUNT(p1.npi), specialty_description, generic_name, p2.drug_name
FROM prescriber AS p1
LEFT JOIN prescription as p2
ON p1.npi = p2.npi
LEFT JOIN drug as d
on p2.drug_name = d.drug_name
WHERE specialty_description='Cardiology'
GROUP BY 2,3,4
ORDER BY COUNT DESC
LIMIT 5;

SELECT drug_name, COUNT(drug_name)
FROM prescription AS p
WHERE npi IN
	(SELECT npi
	 FROM prescriber
	 WHERE specialty_description='Cardiology')
GROUP BY drug_name
ORDER BY COUNT DESC
LIMIT 5;

    c. Which drugs appear in the top five prescribed for both Family Practice prescribers and Cardiologists? Combine what you did for parts a and b into a single query to answer this question.
SELECT COUNT(p1.npi), generic_name, p2.drug_name
FROM prescriber AS p1
LEFT JOIN prescription as p2
ON p1.npi = p2.npi
LEFT JOIN drug as d
on p2.drug_name = d.drug_name
WHERE specialty_description='Family Practice'
GROUP BY 2,3


INTERSECT

SELECT COUNT(p1.npi),generic_name, p2.drug_name
FROM prescriber AS p1
LEFT JOIN prescription as p2
ON p1.npi = p2.npi
LEFT JOIN drug as d
on p2.drug_name = d.drug_name
WHERE specialty_description='Cardiology'
GROUP BY 2,3
ORDER BY COUNT DESC
LIMIT 5;





3. Your goal in this question is to generate a list of the top prescribers in each of the major metropolitan areas of Tennessee.
    a. First, write a query that finds the top 5 prescribers in Nashville in terms of the total number of claims (total_claim_count) across all drugs. Report the npi, the total number of claims, and include a column showing the city.
SELECT npi, nppes_provider_city, nppes_provider_last_org_name, nppes_provider_first_name AS city, COALESCE (SUM(total_claim_count), 0) AS total_claims
FROM prescriber AS p1
LEFT JOIN prescription AS p2
USING(npi)
WHERE nppes_provider_city = 'NASHVILLE'
GROUP BY 1,2, 3, 4
ORDER BY total_claims DESC
LIMIT 5;

    b. Now, report the same for Memphis.
SELECT npi, nppes_provider_city AS city,nppes_provider_last_org_name, nppes_provider_first_name, COALESCE (SUM(total_claim_count), 0) AS total_claims
FROM prescriber AS p1
LEFT JOIN prescription AS p2
USING(npi)
WHERE nppes_provider_city = 'MEMPHIS'
GROUP BY 1,2,3,4
ORDER BY total_claims DESC
LIMIT 5;	

    c. Combine your results from a and b, along with the results for Knoxville and Chattanooga.
	
SELECT npi, nppes_provider_city, nppes_provider_last_org_name, nppes_provider_first_name AS city, COALESCE (SUM(total_claim_count), 0) AS total_claims
FROM prescriber AS p1
LEFT JOIN prescription AS p2
USING(npi)
WHERE nppes_provider_city = 'NASHVILLE'
GROUP BY 1,2, 3, 4
LIMIT 5

UNION 

SELECT npi, nppes_provider_city AS city,nppes_provider_last_org_name, nppes_provider_first_name, COALESCE (SUM(total_claim_count), 0) AS total_claims
FROM prescriber AS p1
LEFT JOIN prescription AS p2
USING(npi)
WHERE nppes_provider_city = 'MEMPHIS'
GROUP BY 1,2,3,4
ORDER BY total_claims DESC
	

 

4. Find all counties which had an above-average (for the state) number of overdose deaths in 2017. Report the county name and number of overdose deaths.

SELECT overdose_deaths, fipscounty, county, year
FROM overdose_deaths
LEFT JOIN fips_county
USING(fipscounty)
WHERE year=2017 AND overdose_deaths > 
  (SELECT AVG(overdose_deaths)
   FROM overdose_deaths
   WHERE year=2017)
GROUP BY 1,2,3, 4
ORDER BY overdose_deaths DESC


5.
    a. Write a query that finds the total population of Tennessee.
SELECT SUM(population)
FROM population

SELECT county, population, ROUND(population * 100.0/SUM(population) OVER(),2) AS percentage
FROM fips_county
JOIN population
USING(fipscounty)
GROUP BY 1,2
ORDER BY percentage DESC


    b. Build off of the query that you wrote in part a to write a query that returns for each county that county's name, its population, and the percentage of the total population of Tennessee that is contained in that county.



