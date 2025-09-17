## Raw data file name : `layoffs.csv`
## Data cleaning file name : `data cleaning.sql`
## Data analysis file name : `cleaned_data_explore.sql`
##Cleaning Steps (what the script does)
### 1) Create Staging & Copy Raw
- CREATE TABLE layoffs_staging LIKE layoffs;
- Copy raw into staging:
  - INSERT layoffs_staging SELECT * FROM layoffs;

### 2) Identify & Remove Duplicates
- Use ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) to flag duplicates.
- Materialize into layoffs_staging3 with an extra row_num column.
- DELETE rows where row_num > 1.

### 3) Standardize Text & Categories
- *Company names:* TRIM(company)
- *Industry:* collapse variants like Crypto... → Crypto
- *Country:* remove trailing dot from United States. → United States

### 4) Fix Dates
- Convert text dates to real DATE:
  - UPDATE ... SET date = STR_TO_DATE(date, '%m/%d/%Y');
  - ALTER TABLE ... MODIFY COLUMN date DATE;

### 5) Handle Missing / Empty Values
- Turn empty strings to NULL in industry.
- *Propagate missing industry* by self-join on the same company (and location) when another row has it filled.

### 6) Drop Unusable Rows
- Remove records where *both* total_laid_off *and* percentage_laid_off are NULL.

### 7) Finalize
- Drop helper column: ALTER TABLE layoffs_staging3 DROP COLUMN row_num;  
- layoffs_staging3 is the *cleaned table*.
