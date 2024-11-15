# Data Cleaning in MySQL

## Project Overview
This project involves cleaning a dataset related to employee layoffs using MySQL. The aim is to ensure data quality and consistency which enables more accurate and insightful analysis. This documentation outlines the steps taken, the challenges encountered, and the solutions implemented during the data cleaning process.

## Objectives
1) remove duplicates
2) Standardize the data
3) Remove or populate NULL values/ BLANK values
4) Remove columns which are not required.

## Dataset Description
The dataset "employee layoffs" have the following fields:
- company
- location
- industry
- total_laid_off
- percentage_laid_off
- date
- stage
- country
- funds_raised_millions

## Process
### Importing Data
- Imported the dataset [layoffs](https://github.com/kevinmartis/MySQL-Data-Cleaning-Project/blob/26b38fb9b4c364486a0aeb26b16566cbac7f28c9/layoffs.csv) into MySQL database.
- Using the Table-Data import wizard, the data was loaded into the database.

### Data Exploration
- Conducted an initial exploration of the dataset using `SELECT` queries.
- Identified key data issues such as missing values, duplicates, and inconsistent formats.

### Creating a Staging Database
- Created a duplicate of the raw dataset to work on the data and to make required adjustments.
- Creating a staging database ensures that, in the event of faults or mistakes, the original raw data remains safeguarded.
```sql
create table layoffs_staging   
like layoffs;

INSERT layoffs_staging
select *
From layoffs;
```
### Removing duplicates
- Add a row column to filter out the duplicates.
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
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO layoffs_staging2
Select *,
row_number() over(
partition by company, location,industry,total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
From layoffs_staging;
```

- Deleting duplicates from the database
```sql
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

### Standardizing data
- To remove unwanted spaces from the `company` column.
```sql
SELECT company, (TRIM(company))
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);
```
- To identify and standardize similar names within `industry` column.
```sql
SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```  

- Trimming unwanted spaces from `country` column
```sql
SELECT country, (TRIM(TRAILING '.' FROM country))
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET country = (TRIM(TRAILING '.' FROM country))
WHERE country LIKE 'United States%';
```

- Changing the `date` format from text to date
```sql
SELECT `date`,
str_to_date(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = str_to_date(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

### To remove or populate NULL values/ BLANK values
- The empty cells are changed to NULL values to reduce complexity
```sql
UPDATE layoffs_staging2
SET industry = NULL 
WHERE industry = '';
```

- Using JOINS the NULL values are populated in `industry` column.
```sql
SELECT * 
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
     ON t1.company = t2.company
WHERE t1.industry IS NULL 
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
     ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

- Deleting NULL values from `total_laid_off` and `percentage_laid_off` column
```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

### Remove columns which are not required
```sql
SELECT *
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

### Cleaned Dataset
- [clean dataset](https://github.com/kevinmartis/MySQL-Data-Cleaning-Project/blob/26b38fb9b4c364486a0aeb26b16566cbac7f28c9/layoffs%20(cleaned).csv)
