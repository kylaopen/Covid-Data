# Covid-Data
First Project 
SELECT location, total_cases, new_cases, total_deaths, population
FROM `Covid2019.deaths`
order by 1, 2;

-- looking at total cases vs total deaths
-- Shows the likihood of dying if you contract codi in your contry
SELECT location, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from `Covid2019.deaths`
where location= 'United States'
order by 1, 2;

--looking at total cases vs population
-- shows what percentage of populatuon

SELECT location, date, total_cases, population, total_cases, (total_cases/population)*100 as DeathPercentage
from `Covid2019.deaths`
where location= 'United States'
order by 1, 2;

--Looking at countrie with highest infection rate compared to populatuon
SELECT location, population, MAX(total_cases) as HighesInfectinCount, MAX((total_cases/population))*100 as PercentPopulationInfected
from `Covid2019.deaths`
group by location, population
order by PercentPopulationInfected desc;

--Showing Countries with Highest Death Count per Population
SELECT location,MAX (cast(total_deaths as int)) as totaldeathcount
from `Covid2019.deaths`
where continent is not null
group by location
order by totaldeathcount desc;

-- Break Down by Continent
-- showig contintents with the highes death count per population
SELECT continent,MAX (cast(total_deaths as int)) as totaldeathcount
from `Covid2019.deaths`
where continent is not null
group by continent
order by totaldeathcount desc;

-- Global Numbers
SELECT sum(new_cases), sum(cast(new_deaths as int)) as total_deaths, sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPercentage
from `Covid2019.deaths`
where continent is not null
--group by date
order by 1, 2;

-- Looking at toal populatuon vs vacciations

select dea.continent, dea.location, dea.population, vac.new_vaccinations, SUM (cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as RollingPeopleVaccinated
from `Covid2019.deaths` dea
join `Covid2019.vaccinations` vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
order by 2, 3;


-- USE CTE

WITH PopsvsVac
as 
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM (cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as RollingPeopleVaccinated
from `Covid2019.deaths` dea
join `Covid2019.vaccinations` vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
)
select *, (RollingPeopleVaccinated/population)*100
from PopsvsVac;


--temp table 


CREATe temp table PercentPopulationVaccinated
as 
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM (cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as RollingPeopleVaccinated
from `Covid2019.deaths` dea
join `Covid2019.vaccinations` vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null);

select *, (rollingpeoplevaccinated/population)*100
from PercentPopulationVaccinated;

--Creating View to store  data for later visualization
create view PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM (cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as RollingPeopleVaccinated
from `Covid2019.deaths` dea
join `Covid2019.vaccinations` vac
on dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null;

select* 
from percentpopulationvaccinated
