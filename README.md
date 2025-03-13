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

- ¿Cuál es el total de casos frente al total de muertes en Perú?

Descripción del conjunto de datos: La tabla de muertes por COVID-19 "CovidDeaths" tiene 26 columnas y 81 060 filas.

La tabla de vacunación contra la COVID-19 'CovidVaccinations' tiene 37 columnas y 85.171 filas
## Analisis con SQL Server
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
## Panel de análisis del impacto global de la COVID-19 en Tableau

![image](https://github.com/user-attachments/assets/80b525ae-bdf1-4a86-8e6c-8db3d5bb6537)


## Insights
### Casos Totales
- Se han registrado 151 millones de casos a nivel global.
La tasa de mortalidad es del 2.11%, lo que indica que aproximadamente 2 de cada 100 personas infectadas fallecen.

### Países con Mayor Tasa de Infección
* Andorra, Montenegro y Chequia lideran la lista con las tasas de infección más altas.
Esto sugiere que países con poblaciones pequeñas pueden tener tasas elevadas debido al menor denominador en la población total.

### Fallecimientos por Continente
- Europa y América del Norte presentan las mayores cifras de fallecidos (~0.16M y 0.15M millones respectivamente).
Oceanía casi no registra muertes, lo que podría deberse a restricciones estrictas y menor densidad poblacional.
### Total de Fallecidos por País

- EE.UU. (576K), Brasil (404K) y México (217K) son los países con más muertes totales.
Esto puede relacionarse con la alta densidad poblacional, sistemas de salud saturados o respuesta tardía a la pandemia.

### Distribución de Infectados
- El mapa de infectados muestra una concentración elevada en Asia, Europa y América del Norte, con África menos afectada en términos absolutos.
África muestra puntos más dispersos, lo que podría indicar un subregistro de casos debido a menor acceso a pruebas diagnósticas.

### Tendencia Temporal de Muertes
- Un pico significativo de fallecimientos se observa a finales de 2020 y principios de 2021, coincidiendo con nuevas variantes y olas de contagios.
La curva sugiere una posible estabilización en la fase final del período analizado (abril 2021), posiblemente debido a vacunaciones.

## Recomendaciones

### Acelerar la distribución de vacunas en países en desarrollo:
- Los datos revelan que los países con tasas de vacunación más altas experimentaron una desaceleración en las tasas de infección y un menor número de muertes. Es imperativo que los gobiernos, en particular en países de ingresos bajos y medios como Nigeria, prioricen la obtención y distribución de vacunas a una mayor proporción de la población. La colaboración internacional y el apoyo de organizaciones como la OMS serán fundamentales para garantizar un acceso equitativo a las vacunas.
### Mejorar la infraestructura de pruebas y notificación:
- Las tasas de infección en países como Nigeria sugieren una posible subnotificación debido a la limitada capacidad de prueba. Las inversiones en infraestructura sanitaria, especialmente en sistemas de pruebas y rastreo de contactos, ayudarán a detectar futuras pandemias de forma temprana y a controlar su propagación.
### Fortalecer la educación y la comunicación en materia de salud pública:
- Muchos países con mejores resultados en cuanto a tasas de mortalidad por COVID-19 contaban con una comunicación y directrices de salud pública claras. Los gobiernos deberían centrarse en educar a la población sobre la importancia de la vacunación, el distanciamiento social, el uso de mascarillas y otras medidas preventivas, especialmente en regiones donde prevalece la reticencia a vacunarse. 
### Mejorar los planes de preparación ante pandemias:
- Los países que experimentaron tasas de mortalidad abrumadoras (por ejemplo, EE. UU. y Brasil) tuvieron dificultades para prepararse. Todas las naciones deben desarrollar planes integrales de respuesta ante pandemias, que incluyan el aprovisionamiento de suministros médicos, respiradores, EPI y la ampliación de la capacidad de las unidades de cuidados intensivos (UCI). Los gobiernos también deben realizar simulacros de pandemia con regularidad para mantenerse preparados ante futuros brotes.
### Fortalecer la colaboración sanitaria mundial:
- Los hallazgos enfatizan la necesidad de cooperación mundial en la gestión de pandemias. Los países deben colaborar para compartir datos, mejores prácticas y recursos durante las crisis sanitarias mundiales. Organismos internacionales como la OMS deben estar capacitados para facilitar esta colaboración, garantizando intervenciones oportunas a través de las fronteras.
### Poblaciones vulnerables objetivo:
- Los grupos de alto riesgo, como las personas mayores, las personas inmunodeprimidas y las personas con afecciones preexistentes, deben tener prioridad para la vacunación y la atención médica durante las pandemias. Se deben desarrollar programas especiales para garantizar que estos grupos tengan fácil acceso a la atención médica, las vacunas y los servicios sociales. 
### Invertir en la resiliencia sanitaria a largo plazo:
- Los países deben aumentar la inversión en sus sistemas de salud para fortalecer su resiliencia ante futuras pandemias. Esto incluye mejorar la capacitación del personal sanitario, aumentar la financiación de la atención médica y crear soluciones escalables para afrontar brotes a gran escala.

Al implementar estas recomendaciones, los países estarán mejor posicionados para afrontar futuras pandemias, reducir las muertes y minimizar las interrupciones en los sistemas de salud mundiales. La pandemia de COVID-19 ha puesto de relieve la importancia de dar respuestas proactivas, unificadas y bien preparadas ante las amenazas sanitarias mundiales.




