# Sql_project
Sql_data_cleaning

-- CLEANING PROJECT

-- PREVIEW THE DATA TO SEE WHAT NEEDS TO BE CLEANED

SELECT	* FROM layoffs;

-- CREATING BACKUP DATABASE

CREATE TABLE layoffs_backup
LIKE layoffs;

INSERT layoffs_backup
SELECT * 
FROM layoffs;

SELECT * FROM layoffs_backup;

-- REMOVING DUPLICATES

SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_backup;

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_backup
)
SELECT *
FROM duplicate_cte
WHERE row_num >1;

-- CREATING ANOTHER BACKUP TABLE

CREATE TABLE `layoffs_backup3` (
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

SELECT *
FROM layoffs_backup3;

INSERT INTO layoffs_backup3
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_backup;

-- FILTER ROW NUM > 1 TO DELETE THEM

SELECT *
FROM layoffs_backup3
WHERE row_num > 1;

DELETE
FROM layoffs_backup3
WHERE row_num > 1;

SELECT *
FROM layoffs_backup3;

-- STANDARDIZING THE DATA

SELECT company, TRIM(company)
FROM layoffs_backup3;

UPDATE layoffs_backup3
SET company = TRIM(company);

SELECT DISTINCT industry
FROM layoffs_backup3
ORDER BY 1;


-- CLEANING CRYPTO TO BE THE SAME

SELECT *
FROM layoffs_backup3
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_backup3
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

-- CHECK IF WE CLEANED IT

SELECT DISTINCT industry
FROM layoffs_backup3
ORDER BY 1;

-- CHECKING THE COUNTRY COLUMN

SELECT DISTINCT country
FROM layoffs_backup3
ORDER BY 1;

-- CLEANING THE UNITED STATES FROM DOT

SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_backup3
ORDER BY 1;

UPDATE layoffs_backup3
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

-- CHANGING DATE COLUMN FROM TEXT TO DATE FORMAT

SELECT `date`,
str_to_date(`date`, '%m/%d/%Y')
FROM layoffs_backup3;

UPDATE layoffs_backup3
SET `date` = str_to_date(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_backup3
MODIFY COLUMN `date` DATE;

-- DELETING ROWS AND COLUMNS WITH NULL VALUES THAT I DONT NEED FOR EXPLORATORY DATA ANALYSIS LATER ON

SELECT *
FROM layoffs_backup3
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- I DONT NEED THIS COLUMNS WITH NULL VALUES FOR FUTHER ANALYSIS SO I WILL DELETE IT

DELETE
FROM layoffs_backup3
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- PREVIEW THE DATA AGAIN

SELECT *
FROM layoffs_backup3;

-- DELETING COLUMN row_num BECAUSE I DONT NEED IT

ALTER TABLE layoffs_backup3
DROP COLUMN row_num;

SELECT *
FROM layoffs_backup3;
