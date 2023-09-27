# CovidPortfolio
Select *
From PortfolioProject..['Coviddeaths$']
where continent is not null
Order by 3,4

--Select *
--From PortfolioProject..Covidvaccinations$
--Order by 3,4

--Select location, date, total_cases, new_cases, total_deaths, population
--From PortfolioProject..['Coviddeaths$']
--order by 1,2

Select continent, location, date, total_cases, total_deaths, (CONVERT(float, total_deaths) / NULLIF(CONVERT(float, total_cases), 0))*100 as DeathPercentage
From PortfolioProject..['Coviddeaths$']
where location = 'United States'
order by 1,2

---

Select location, date, population, total_cases, (CONVERT(float, total_cases) / population ) *100 as PercentagePopulationAffected
From PortfolioProject..['Coviddeaths$']
where location = 'United States'
order by 1,2

-- Looking at countries with highest infection rate compared to population


Select location, population, max (total_cases) as HighestInfectionCount, Max (CONVERT(float, total_cases) / population ) *100 as PercentagePopulationAffected
From PortfolioProject..['Coviddeaths$']
Group by location, population
order by PercentagePopulationAffected desc

-- Countries with highest death count per population


Select continent, Max(cast (total_deaths as int)) as TotalDeathCount
From PortfolioProject..['Coviddeaths$']
where continent is not null
Group by continent
order by TotalDeathCount desc

--- GLOBAL NUMBERS

Select SUM(new_cases)as total_cases, SUM(Cast(new_deaths as int)) as total_deaths, SUM(Cast(new_deaths as int))/ SUM(new_cases) * 100 as DeathPercentage
From PortfolioProject..['Coviddeaths$']
--where location = 'United States'
where continent is not null
--Group by date
order by 1,2

--Looking at Total Population vs Vaccinations

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location, dea.Date) as RollingPeopleVaccinated
, (RollingPeopleVaccinated/population) * 100
From PortfolioProject..['Coviddeaths$'] dea
join PortfolioProject..Covidvaccinations$ vac
     On dea.location = vac.location
	 and dea.date = vac.date
where dea.continent is not null
order by 2,3


-- USE CTE

With PopvsVac (Continent, Location, Date, Population, New_vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population) * 100
From PortfolioProject..['Coviddeaths$'] dea
join PortfolioProject..Covidvaccinations$ vac
     On dea.location = vac.location
	 and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)* 100
From PopvsVac

-- TEMP TABLE

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar (255),
Location nvarchar (255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population) * 100
From PortfolioProject..['Coviddeaths$'] dea
join PortfolioProject..Covidvaccinations$ vac
     On dea.location = vac.location
	 and dea.date = vac.date
--where dea.continent is not null
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)* 100
From #PercentPopulationVaccinated

-----Creating view to store data for later visualization

Create View PercentPopulationVaccinated as 
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population) * 100
From PortfolioProject..['Coviddeaths$'] dea
join PortfolioProject..Covidvaccinations$ vac
     On dea.location = vac.location
	 and dea.date = vac.date
where dea.continent is not null
--order by 2,3

--Tableau Tables
1
Select SUM(new_cases)as total_cases, SUM(Cast(new_deaths as int)) as total_deaths, SUM(Cast(new_deaths as int))/ SUM(new_cases) * 100 as DeathPercentage
From PortfolioProject..['Coviddeaths$']
--where location = 'United States'
where continent is not null
--Group by date
order by 1,2

-- 2
Select location, SUM(Cast(new_deaths as int)) as TotalDeathCount
From PortfolioProject..['Coviddeaths$']
--where location like '%States%'
where continent is null
and location not in ('World', 'European Union', 'International')
Group by location
order by TotalDeathCount desc

--3
Select location, population, max (total_cases) as HighestInfectionCount, Max (CONVERT(float, total_cases) / population ) *100 as PercentagePopulationInfected
From PortfolioProject..['Coviddeaths$']
--where location like '%States%'
Group by location, population
order by PercentagePopulationInfected desc

--4
Select location, population, date, max (total_cases) as HighestInfectionCount, Max (CONVERT(float, total_cases) / population ) *100 as PercentagePopulationInfected
From PortfolioProject..['Coviddeaths$']
--where location like '%States%'
Group by location, population, date
order by PercentagePopulationInfected desc
