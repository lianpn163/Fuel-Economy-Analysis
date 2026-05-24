# 1. Project Objective
The primary objective of this project is to analyze global automotive manufacturing trends between $1970$ and $1982$ to identify the core design and macroeconomic factors driving fleet fuel efficiency. By evaluating how consumer demands and international regulatory pressures shifted after the 1973 and 1979 energy crises, this analysis seeks to reverse-engineer the strategic product decisions that allowed foreign automotive manufacturers to successfully capture market share from domestic brands.

# 2. Dataset Discovery
The analysis is grounded in a historic fleet dataset consisting of 398 unique vehicle models. The data captures a multi-dimensional profile for each vehicle across three key areas

## Efficiency & Performance: 
Fuel economy measured in miles per gallon ($\text{MPG}$) and performance measured by acceleration times (0 to 60 mph in seconds).

## Engineering Specifications: 
Physical and mechanical attributes including vehicle weight, engine displacement, horsepower, and cylinder configurations ($4$, $6$, or $8$ cylinders).

## Temporal & Geographic Context: 
Production timeline spanning model years 1970-1982 and manufacturing regional origins segmented into North America (USA), Europe, and Asia (primarily Japan).

# 3.Methodology
To transform the raw vehicle metrics into meaningful market insights, the project followed a structured three-phase analysis

## Chronological Trend Analysis: 
*Tracked the baseline movement of fleet efficiency across the twelve-year timeline to map how manufacturers scaled down vehicle sizes in direct response to the global oil shocks.*

SELECT 
    model_year, 
    ROUND (AVG(mpg),1) AS avg_mpg, 
    COUNT(*) AS car_count
FROM automotive-fuel-economy.economy.auto_mpg
GROUP BY model_year
ORDER BY model_year;

## Statistical Core Relation Mapping: 
*Quantified the exact strength of the relationships between physical vehicle dimensions and engine performance against ultimate fuel output. This allowed the isolation of weight as the single most critical structural constraint on vehicle efficiency.*

SELECT 
'weight' as characteristics,
    (COUNT(*) * SUM(weight * mpg) - SUM(weight) * SUM(mpg)) / 
    (SQRT(COUNT(*) * SUM(weight * weight) - SUM(weight) * SUM(weight)) * SQRT(COUNT(*) * SUM(mpg * mpg) - SUM(mpg) * SUM(mpg))) AS correlation
FROM automotive-fuel-economy.economy.auto_mpg
WHERE weight IS NOT NULL

UNION ALL

SELECT 
'displacement' as characteristics,
    (COUNT(*) * SUM(displacement * mpg) - SUM(displacement) * SUM(mpg)) / 
    (SQRT(COUNT(*) * SUM(displacement*displacement) - SUM(displacement) * SUM(displacement)) * SQRT(COUNT(*) * SUM(mpg * mpg) - SUM(mpg) * SUM(mpg))) AS correlation
FROM automotive-fuel-economy.economy.auto_mpg
WHERE displacement IS NOT NULL

UNION ALL

SELECT 'horsepower' AS characteristic,
    (COUNT(SAFE_CAST(horsepower AS FLOAT64)) * SUM(SAFE_CAST(horsepower AS FLOAT64) * mpg) - SUM(SAFE_CAST(horsepower AS FLOAT64)) * SUM(mpg)) / 
    (SQRT(COUNT(SAFE_CAST(horsepower AS FLOAT64)) * SUM(SAFE_CAST(horsepower AS FLOAT64) * SAFE_CAST(horsepower AS FLOAT64)) - SUM(SAFE_CAST(horsepower AS FLOAT64)) * SUM(SAFE_CAST(horsepower AS FLOAT64))) * SQRT(COUNT(SAFE_CAST(horsepower AS FLOAT64)) * SUM(mpg * mpg) - SUM(mpg) * SUM(mpg))) AS correlation
FROM automotive-fuel-economy.economy.auto_mpg 
WHERE SAFE_CAST(horsepower AS FLOAT64) IS NOT NULL

UNION ALL

SELECT 
'cylinders' as characteristics,
    (COUNT(*) * SUM(cylinders * mpg) - SUM(cylinders) * SUM(mpg)) / 
    (SQRT(COUNT(*) * SUM(cylinders*cylinders) - SUM(cylinders) * SUM(cylinders)) * SQRT(COUNT(*) * SUM(mpg * mpg) - SUM(mpg) * SUM(mpg))) AS correlation
FROM automotive-fuel-economy.economy.auto_mpg
WHERE cylinders IS NOT NULL

UNION ALL

SELECT 
'acceleration' as characteristics,
    (COUNT(*) * SUM(acceleration * mpg) - SUM(acceleration) * SUM(mpg)) / 
    (SQRT(COUNT(*) * SUM(acceleration*acceleration) - SUM(acceleration) * SUM(acceleration)) * SQRT(COUNT(*) * SUM(mpg * mpg) - SUM(mpg) * SUM(mpg))) AS correlation
FROM automotive-fuel-economy.economy.auto_mpg
WHERE acceleration IS NOT NULL;


## Macro-Regulatory Segmentation: 
*Segmented the data by manufacturing region to uncover the engineering design paradigms of the USA, Europe, and Asia. These data gaps were then contextualized against real-world international trade frameworks, such as Japan's stringent $2,000\text{ cc}$ engine tax laws, regional urban infrastructure limits, and competitive market positioning.*

SELECT  
    CASE origin
        WHEN 1 THEN 'USA (North America)'
        WHEN 2 THEN 'Europe'
        WHEN 3 THEN 'Asia (primarily Japan)'
        ELSE 'Unknown'
    END AS car_origin,
   
    round (AVG(mpg) ,1) AS avg_mpg,
    round (AVG(cylinders) ,1) AS avg_cylinders,
    round (AVG(displacement) ,1) AS avg_displacement,
    round (AVG(SAFE_CAST(horsepower AS FLOAT64)) ,1) AS avg_horsepower,
    round (AVG(weight) ,1) AS avg_weight,
    round (AVG(acceleration) ,1) AS avg_acceleration,
    COUNT(*) AS total_cars
FROM automotive-fuel-economy.economy.auto_mpg
GROUP BY origin
ORDER BY origin;

