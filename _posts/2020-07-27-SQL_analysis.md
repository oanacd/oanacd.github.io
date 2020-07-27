---
title: "SQL Data Analysis on Bike Stations Traffic in MTL"
date: 2020-07-27T12:34:30-04:00
categories:
  - blog
tags:
  - Jekyll
  - update
---
Analysis made in 2020 for the HEC cours: Logiciels statistiques pour la gestion.

## Create a new table with weather information

Code: 
```javascript
CREATE TABLE 
	dataset_meteo_Q_2018 AS
		SELECT 
			'Nom de la Station'n AS nom_station,
			'Date/Heure'n AS date,
			mois,
			'Temp moy.(°C)'n AS temp_moy_c,
			'Précip. tot. (mm)'n AS precip_tot_mm,
			'Neige au sol (cm)'n AS neige_sol_cm
		FROM 
			MTL_meteo_quotidienne_2018 ;
 ```
 
## Create a new table with bike stations
Code: 
```javascript
CREATE TABLE 
	dataset_velo_Q_2018 AS
		SELECT 	
			*,
			MONTH(date) as mois
		FROM 
			MTL_compteur_velo_q_2018
		WHERE 
			valeur IS NOT NULL;
```

## Count the number of days available per station
Code: 
```javascript
SELECT 
	compteur,
	COUNT(date) AS nbr_jours_dispo
FROM 
	dataset_velo_Q_2018
GROUP BY
	compteur ;
```

## Average monthly temperatures in 2018
Code: 
```javascript
SELECT  
	mois,
	AVG(temp_moy_c) AS avg_temp
FROM 
	dataset_meteo_Q_2018 
GROUP BY 
	mois
ORDER BY 
	avg_temp DESC;
```

## Total monthly rainfall and maximum level of snow
Code: 
```javascript
SELECT 
	mois,
	SUM(precip_tot_mm) AS total_precip,
	MAX(neige_sol_cm) AS max_neige
FROM 
	dataset_meteo_Q_2018
GROUP BY 
	mois;
```

## Highest and lowest average temperature in 2018
Code: 
```javascript
SELECT  
	nom_station,
	date,
	mois,
	temp_moy_c as max_min_temp,
	precip_tot_mm,
	neige_sol_cm	
FROM 
	dataset_meteo_Q_2018
GROUP BY 
	date
HAVING 
	max_min_temp = (SELECT MAX(temp_moy_c) AS max_min_temp
					FROM dataset_meteo_Q_2018
				)
	OR 
	max_min_temp = (SELECT MIN(temp_moy_c) AS max_min_temp 
					FROM dataset_meteo_Q_2018
				)
ORDER BY 
	max_min_temp DESC;
```

## The number of days without bike traffic
Code: 
```javascript
SELECT 
	compteur,
	COUNT(date) AS aucun_passage
FROM 
	dataset_velo_Q_2018
WHERE 
	valeur = 0
GROUP BY 
	compteur
ORDER BY 
	aucun_passage DESC;
```

## The number of days with at least one station without traffic
Code: 
```javascript

SELECT 
	COUNT (DISTINCT date) AS nbr_jours_zero_depl
FROM 
	dataset_velo_Q_2018 
WHERE 
	valeur = 0 ;
```

## The days having the most and the least bike traffic
Code: 
```javascript
CREATE TABLE deplacements AS
SELECT 
	date,
	SUM(valeur) as total_depl
FROM 
	dataset_velo_Q_2018
GROUP BY date;

SELECT 
	date, 
	total_depl
FROM 
	deplacements
HAVING 
	total_depl = MAX(total_depl)
	OR 
	total_depl = MIN(total_depl);
```
## Weather conditions when the stations registered 20 passages or less
Code: 
```javascript
SELECT 	
		compteur,
		SUM(valeur) as total_depl_max20parjour,
		MIN(temp_moy_c) AS min_temp,
		MAX(temp_moy_c) AS max_temp,
		AVG(temp_moy_c) AS avg_temp,
		MIN(precip_tot_mm) AS min_precip,
		MAX(precip_tot_mm) AS max_precip, 
		AVG(precip_tot_mm) AS avg_precip,
		MIN(neige_sol_cm) AS min_neige,
		MAX(neige_sol_cm) AS max_neige,
		AVG(neige_sol_cm) As avg_neige
FROM 
	dataset_meteo_Q_2018 m 
INNER JOIN
	dataset_velo_Q_2018 v
ON 
	m.date = v.date
WHERE 
	valeur <=20
GROUP BY 
	compteur;
```

## Monthly thermal amplitude
Code: 
```javascript
CREATE TABLE 
	dataset_meteo_M_2018 AS
SELECT 
	mois,
	AVG(temp_moy_c) AS avg_temp, 
	SUM(precip_tot_mm) AS total_precip,
	MAX(neige_sol_cm) AS max_neige,
	MAX(temp_moy_c) - MIN(temp_moy_c) AS amplitude
FROM 
	dataset_meteo_Q_2018 
GROUP BY 
	mois;
```

## Proportion of monthly traffic
Code: 
```javascript
CREATE TABLE 
	dataset_velo_M_2018 AS
SELECT 
	mois,
	SUM(valeur) AS total_depl,
	SUM(valeur)/(SELECT sum(valeur) FROM dataset_velo_Q_2018) AS prop_depl
FROM 
	dataset_velo_Q_2018 
GROUP BY 
	mois ;
```
## Total bike traffic and weather conditions
Code: 
```javascript
SELECT 
	m.mois,
	SUM(total_depl) AS total_depl_gr,
	AVG(avg_temp) AS avg_temp_gr,
	SUM(total_precip) AS total_precip_gr,
	MAX(max_neige) AS max_neige_gr
FROM 
	dataset_velo_M_2018 v
INNER JOIN
   dataset_meteo_M_2018 m
ON 
	m.mois = v.mois 
GROUP BY 
	m.mois;
```

## Grouping the bike traffic into two categories: bike season and off season.
Code: 
```javascript
SELECT 
	CASE 
		WHEN m.mois BETWEEN 5 AND 10
				THEN 'saison_velo'
		WHEN m.mois IN (1,2,3,4,11,12)
				THEN 'non_saison_velo'
		ELSE 'NA'
		END AS saison,
	SUM(total_depl) AS total_depl_gr,
	SUM(total_depl) / (SELECT SUM(total_depl) FROM dataset_velo_M_2018) AS prop_depl,
	AVG(avg_temp) AS avg_temp_gr,
	SUM(total_precip) AS total_precip_gr,
	MAX(max_neige) AS max_neige_gr
FROM
	dataset_velo_M_2018 v
INNER JOIN
   dataset_meteo_M_2018 m
ON 
	m.mois = v.mois
GROUP BY 
	saison ;
```
