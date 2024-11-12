# Layoffs_dataSheet
<h1>Data Cleaning for Layoffs Dataset</h1>
<h5>This project demonstrates a systematic approach to cleaning and transforming a layoffs dataset. The dataset is used to track companies that have conducted layoffs, including information about the company, industry, location, number of layoffs, and other associated metrics. The code processes and cleans the raw data using SQL queries, ensuring consistency and removing duplicates for better analysis.</h5>
 <h2>Overview</h2>
    <p>This repository contains SQL queries for the following tasks:</p>
    <ul>
        <li><strong>Data Staging</strong>: Creating a temporary staging area for data transformations.</li>
        <li><strong>Duplicate Removal</strong>: Identifying and deleting duplicate entries.</li>
        <li><strong>Data Standardization</strong>: Standardizing company names, industry types, and date formats.</li>
        <li><strong>Exploratory Data Analysis</strong>: Analyzing the cleaned dataset to derive insights.</li>
    </ul>
    <h2>Steps Involved in the Data Cleaning Process</h2>

    <h3>1. Data Staging</h3>
    <p>The process starts by creating a staging table that mirrors the structure of the original dataset (<code>layoffs</code>) to facilitate data manipulation.</p>
    <pre><code>create table layoffs_staging like layoffs;</code></pre>
    <p>Data is then copied into the staging table:</p>
    <pre><code>insert into layoffs_staging
select * from layoffs;</code></pre>
<h3>2. Removing Duplicates</h3>
    <p>To remove duplicate records, a <code>ROW_NUMBER()</code> function is used. The query partitions the data by columns such as <code>company</code>, <code>location</code>, <code>industry</code>, <code>total_laid_off</code>, etc., and assigns a row number to each duplicate group. Only the first row is kept, and duplicates are deleted.</p>
    <h4>Identifying duplicates:</h4>
    <pre><code>WITH duplicate_cte AS (
  select *, ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off, `date`, stage, country, funds_raised_millions) as row_nums
  FROM layoffs_staging
)
SELECT * FROM duplicate_cte
WHERE row_nums > 1;</code></pre>
 <h4>Removing duplicates:</h4>
    <pre><code>DELETE from layoffs_staging2 
WHERE row_nums > 1;</code></pre>
 <h3>3. Data Standardization</h3>
    <p>This step involves cleaning and standardizing various columns in the dataset, including company names, industry types, country names, and dates.</p>

    <h4>Trim whitespace from company names:</h4>
    <pre><code>UPDATE layoffs_staging2
SET company = TRIM(company);</code></pre>
    <h4>Standardize industry names:</h4>
    <p>Some industry names (e.g., "Crypto") are standardized to ensure consistency.</p>
    <pre><code>UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';</code></pre>

    <h4>Standardize country names:</h4>
    <p>Trailing periods in country names are removed to ensure uniformity.</p>
    <pre><code>UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';</code></pre>

    <h4>Date format conversion:</h4>
    <p>The <code>date</code> column is transformed into a standardized date format.</p>
    <pre><code>UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;</code></pre>

    <h4>Handling missing or empty values in industry:</h4>
    <p>Empty or null industry fields are updated.</p>
    <pre><code>UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';</code></pre>

    <p>The missing industry values are filled by matching them with non-null values from the same company:</p>
    <pre><code>UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;</code></pre>

    <h4>Handling missing values for layoffs:</h4>
    <p>Records with missing values in <code>total_laid_off</code> and <code>percentage_laid_off</code> are deleted:</p>
    <pre><code>DELETE from layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;</code></pre>

    <h3>4. Data Analysis</h3>
    <p>Once the data is cleaned, exploratory data analysis (EDA) is conducted to derive insights.</p>

    <h4>Maximum layoffs and percentage laid off:</h4>
    <pre><code>SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;</code></pre>

    <h4>Companies with the highest total layoffs:</h4>
    <pre><code>SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;</code></pre>

    <h4>Layoffs by industry:</h4>
    <pre><code>SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;</code></pre>

    <h4>Layoffs by country:</h4>
    <pre><code>SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;</code></pre>

    <h4>Layoffs over time:</h4>
    <pre><code>SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;</code></pre>

    <h4>Layoffs by stage:</h4>
    <pre><code>SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 1 DESC;</code></pre>

    <h2>Conclusion</h2>
    <p>The steps outlined above help in transforming a raw layoffs dataset into a clean and structured version suitable for analysis. By removing duplicates, standardizing key fields, and handling missing data, the dataset becomes much more reliable and useful for generating insights.</p>
