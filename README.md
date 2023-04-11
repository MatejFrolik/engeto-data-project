# Projekt ENGETO ACADEMY - datová analýzy  


  
Odkaz na můj **Linkedl** [ZDE](https://www.linkedin.com/in/matěj-frol%C3%ADk-183812230/).


----

## Úvod do projektu/zadání projektu


Na analytickém oddělení nezávislé společnosti, která se zabývá životní úrovní občanů, jsme se dohodli, že se pokusíme odpovědět na pár definovaných výzkumných otázek, které adresují dostupnost základních potravin široké veřejnosti. Kolegové již vydefinovali základní otázky, na které se pokusí odpovědět a poskytnout tuto informaci tiskovému oddělení. Toto oddělení bude výsledky prezentovat na následující konferenci zaměřené na tuto oblast.

Úkolem je připravit robustní datové podklady, ve kterých bude možné vidět porovnání dostupnosti potravin na základě průměrných příjmů za určité časové období.

K projektu jsou použity datové sady z Portálu otevřených dat ČR.

---
## Výzkumné otázky

1. Rostou v průběhu let mzdy ve všech odvětvích, nebo v některých klesají?  

2. Kolik je možné si koupit litrů mléka a kilogramů chleba za první a poslední srovnatelné období v dostupných datech cen a mezd?  

3. Která kategorie potravin zdražuje nejpomaleji (je u ní nejnižší percentuální meziroční nárůst)?  

4. Existuje rok, ve kterém byl meziroční nárůst cen potravin výrazně vyšší než růst mezd (větší než 10 %)?  

5. Má výška HDP vliv na změny ve mzdách a cenách potravin? Neboli, pokud HDP vzroste výrazněji v jednom roce, projeví se to na cenách potravin či mzdách ve stejném nebo násdujícím roce výraznějším růstem?  


---

## Průběh projektu
1. V první fázi projektu je třeba si vytvořit dvě tabulky a to první tabulku s názvem t_Matej_Frolik_project_SQL_primary_final, kde získáme potřebné sloupce pro zodpovězení výzkumných otázek a druhou tabulku s názvem t_Matej_Frolik_project_SQL_secondary_final, která bude obsahovat data pro ostatní státy. K vytvoření první tabulky jsou vytvořeny tři pomocné tabulky (czechia_price_assist, czechia_payroll_assist a czechia_gdp_assist) ze kterých pak výsledná tabulky vychází. 

```
CREATE OR REPLACE TABLE czechia_price_assist AS (
	SELECT cpc.name AS foodstuff_name , year(cp.date_from) AS price_year, 
        round(avg(cp.value),1) cost, cpc.price_value, cpc.price_unit 
	FROM czechia_price cp 
	LEFT JOIN czechia_price_category cpc ON cpc.code = cp.category_code 
	WHERE year(cp.date_from) BETWEEN 2006 AND 2018
	GROUP BY year(cp.date_from), cpc.name
);

CREATE OR REPLACE TABLE czechia_payroll_assist AS (
	SELECT cpib.name AS branch_name , cp.payroll_year , round(avg(cp.value),0) AS salary
	FROM czechia_payroll cp 
	LEFT JOIN czechia_payroll_industry_branch cpib ON cpib.code = cp.industry_branch_code 
	WHERE cp.value_type_code = 5958 AND cp.payroll_year BETWEEN 2006 AND 2018
	GROUP BY cp.payroll_year, cpib.name 
);

CREATE OR REPLACE TABLE czechia_gdp_assist AS (
	SELECT c.country, e.gdp , e.`year`  AS gdp_year
	FROM economies e
	LEFT JOIN countries c ON e.country = c.country 
	WHERE c.country LIKE 'Czech Republic' AND e.`year` BETWEEN 2006 AND 2018

);

CREATE OR REPLACE TABLE t_Matej_Frolik_project_SQL_primary_final AS (
	SELECT *
	FROM assist2 AS  a2
	JOIN assist1 AS a1 ON a1.price_year = a2.payroll_year
	JOIN assist3 AS a3 ON a3.gdp_year = a1.price_year 
);
SELECT * FROM t_Matej_Frolik_project_SQL_primary_final ;


CREATE OR REPLACE TABLE t_Matej_Frolik_project_SQL_secondary_final AS (
SELECT c.*, e.country AS eco_country, e.`year` , e.GDP, e.population eco_population, e.gini 
FROM countries c 
LEFT JOIN economies e ON c.country = e.country
```

2. U zodpovězení první otázky (Rostou v průběhu let mzdy ve všech odvětvích, nebo v některých klesají) si nejprve necháme načíst procentuální růst nebo pokles mezd pro jednotlivé odvětví a roky a druhým dotazem vybere jen ty odvětví, ve kterých nám mzda alespoň jednou klesala, čímž, zjistíme, že v 15 odvětví nám mzda ale jednou za dané období klesala. 

```
SELECT t.branch_name, t.payroll_year, t2.payroll_year AS year_prew, 
	   round( (t.salary - t2.salary) / t2.salary * 100, 1) AS salary_growth_percent
FROM t_Matej_Frolik_project_SQL_primary_final t
JOIN t_Matej_Frolik_project_SQL_primary_final t2 
	ON t.branch_name = t2.branch_name
	AND t.payroll_year = t2.payroll_year + 1
GROUP BY t.payroll_year, t2.payroll_year, t.branch_name
ORDER BY t.branch_name, t.payroll_year;

WITH salary_growth AS(
SELECT t.branch_name, t.payroll_year, t2.payroll_year AS year_prew, 
	   round( (t.salary - t2.salary) / t2.salary * 100, 1) AS salary_growth_percent
FROM t_Matej_Frolik_project_SQL_primary_final t
JOIN t_Matej_Frolik_project_SQL_primary_final t2 
	ON t.branch_name = t2.branch_name
	AND t.payroll_year = t2.payroll_year + 1
GROUP BY t.payroll_year, t2.payroll_year, t.branch_name
HAVING salary_growth_percent < 0
ORDER BY t.branch_name, t.payroll_year)
SELECT DISTINCT (branch_name)
FROM salary_growth;
```
3.  
