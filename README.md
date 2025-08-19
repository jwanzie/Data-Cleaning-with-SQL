### Data Cleaning in SQL

**Dataset**: *https://www.kaggle.com/datasets/swaptr/layoffs-2022*

Data Cleaning is the process of transforming raw data into a consistent, reliable and usable format to be able to perform Exploratory Data Analysis and create visualisations.
This project demonstrates a structured SQL workflow to clean the layoffs dataset.

### Steps performed;

#### Step 1; Create Database and Import Data

Create a new schema within MySQL and name it `world_layoffs`.
```sql
create database world_layoffs;
```
Import tables `layoffs` into schema and verify imported data using `select` statement.

#### Step 2; Remove Duplicates

Create a staging table `layoffs_staging` and insert everything from `layoffs` table to protect the raw data.
```sql
create table layoffs_staging
like layoffs
;

select *
from layoffs_staging
;

insert layoffs_staging
select *
from layoffs
;
```

Without a unique row id within the data set, use `row_number()` as `row_num` and match it against the columns using `partition by` to identify duplicates.
```sql
select * ,
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, 'date',
stage, country, funds_raised_millions)
as row_num
from layoffs_staging
;
```

Filter against the row number using a cte `duplicate_cte` with a `row_num > 1` indicating a duplicate.
```sql
with duplicate_cte as (
select * ,
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, 'date',
stage, country, funds_raised_millions)
as row_num
from layoffs_staging
)
select *
from duplicate_cte
where row_num > 1
;
```

Create a new staging table `layoffs_staging2` name columns similar to `layoffs_staging` including the new `row_num` column and assign datatype and insert everything from `layoffs_staging` plus the row number.
```sql
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

insert layoffs_staging2
select * ,
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, 'date',
stage, country, funds_raised_millions)
as row_num
from layoffs_staging
;
```

Within `layoffs_staging2` filter data and delete duplicates.
```sql
delete 
from layoffs_staging2
where row_num > 1 
;
```

#### Step 3; Standardise Data

Standardising data is all about finding issues of inconsistency within the data and fixing it.
Working from `layoffs_staging2`, use `distinct` statement to check text data for any inconsistencies.
```sql
select distinct industry
from layoffs_staging2
order by 1
;
```

Use `trim()` statement and `%` to standardise text data within columns and update.
```sql
select company, trim(company)
from layoffs_staging2
;

update layoffs_staging2
set company = trim(company)
;
```

For date column, convert data to date format using `str_to_date()` and `%m/%d/%Y`. Update and alter table to convert column to date.
```sql
select `date`,
str_to_date(`date`, '%m/%d/%Y')
from  layoffs_staging2
;

update layoffs_staging2
set date = str_to_date(`date`, '%m/%d/%Y')
;

alter table layoffs_staging2
modify column `date` date
;
```


#### Step 4; Null/Blank Values

Use `is null` to select rows that contain null values within different columns and populate based on previous columns with similar values.
```sql
select *
from layoffs_staging2
where industry is null
or industry = ''
;
```

Set blank values to null and repeat the previous step.
```sql
update layoffs_staging2
set industry = null
where industry = ''
;

update layoffs_staging2 t1
join layoffs_staging2 t2
	on t1.company = t2.company
set t1.industry = t2.industry
where t1.industry is null 
and t2.industry is not null
;
```

#### Step 5; Unnecessary Columns

For columns not needed like the previously created `row_num` remove it using `drop column` statement.
```sql
alter table layoffs_staging2
drop column row_num
;
```
