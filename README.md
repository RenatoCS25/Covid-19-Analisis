# Pandemia de COVID-19: Un análisis de datos completo con SQL y Power BI

## Breve introducción
La pandemia de COVID-19 fue única y se cobró muchas vidas. Me embarqué en un viaje para comprender mejor el impacto de la COVID-19 en el mundo.

## Conjunto de datos:
El análisis realizado sobre la pandemia de COVID-19 utilizando el conjunto de datos de la Organización Mundial de la Salud (OMS) se centra en métricas críticas como el total de casos, el número de muertes, las tasas de infección, la vacunación y el impacto del virus en los diferentes continentes.

Este análisis busca responder las siguientes preguntas:

- ¿Cuál es el porcentaje de la población vacunada?

- ¿Cuál es el número de muertes por continente?

- ¿Cuáles son los continentes con el mayor número de muertes por población?

- ¿Cuáles son los países con la mayor tasa de infección en comparación con la población?

- ¿Cuál es el total de casos y el porcentaje de población infectada?

- ¿Cuál es el total de casos frente al total de muertes en Nigeria?

Descripción del conjunto de datos: La tabla de muertes por COVID-19 "CovidDeaths" tiene 26 columnas y 81 060 filas.

Covid-19 Vaccinations table 'CovidVaccinations' has 37 columns and 85,171 rows
## Analisis 
Seleccionemos los datos que vamos a utilizar:

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM CovidDeaths
WHERE continent is not null and  total_deaths is not null
ORDER BY 1,2

### Total de casos vs. total de muertes en Perú
```sql
-- Muestra la probabilidad de morir si contraes COVID-19 en Perú

SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathsPercentage
FROM CovidDeaths
WHERE location LIKE '%Peru' 
AND continent IS NOT NULL
ORDER BY 1,2,
```

### Total de casos vs. Total de muertes en Perú
```sql
-- Muestra qué porcentaje de la población está infectada con Covid

SELECT location, date, population, total_cases, 
	  (total_cases/population)*100 AS PercentPopulationInfected 
FROM CovidDeaths
ORDER BY 1,2
```

### Países con mayor tasa de infección en comparación con la población

```sql
-- Países con mayor tasa de infección en comparación con la población

SELECT location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM CovidDeaths
WHERE continent is not null 
GROUP BY location, population
ORDER BY PercentPopulationInfected DESC
```
### Países con mayor número de muertes por población

```sql
-- Países con mayor número de muertes por población

SELECT location, MAX(cast(total_deaths as int)) AS TotalDeathsCount
FROM CovidDeaths
WHERE continent is not null 
GROUP BY location
ORDER BY TotalDeathsCount DESC
```
### Países con mayor número de muertes por población

```sql
-- Países con mayor número de muertes por población

SELECT location, MAX(cast(total_deaths as int)) AS TotalDeathsCount
FROM CovidDeaths
WHERE continent is not null 
GROUP BY location
ORDER BY TotalDeathsCount DESC
```
### Cifras globales de COVID-19: porcentaje de muertes

```sql
-- Porcentaje de muertes por total de casos

SELECT SUM(CAST(new_cases AS INT)) AS Total_cases, 
SUM(CAST(new_deaths AS INT)) AS Total_Deaths, 
SUM(CAST(new_deaths AS INT))/ SUM(new_cases )*100 AS DeathsPercentage
FROM CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1,2
```

### Poblacion Total vs Vacunaciones

```sql
-- Muestra el porcentaje de la población que ha recibido al menos una vacuna contra la COVID-19.

WITH popvsvac (Continent, Location, Date, Population, New_vaccinations, RollingPeopleVaccinated) AS  
(
SELECT dea.continent, dea.location, dea.date,dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (partition BY dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated
FROM CovidDeaths dea
JOIN CovidVaccinations vac
ON dea.location = vac.location
and dea.date = vac.date
WHERE dea.continent IS NOT NULL
)
SELECT*, (RollingPeopleVaccinated/Population)*100 AS PercentRollingPeopleVaccinated
FROM popvsvac
```



