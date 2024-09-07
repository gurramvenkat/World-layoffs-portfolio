**Data Cleaning in World Layoffs Dataset: SQL Process Overview**

This SQL data cleaning project, executed in MySQL, aims to improve the accuracy and usability of the "world_layoffs" dataset.

**Step 1: View the Original Table**
Before making any changes, it's essential to examine the full dataset to understand its structure. We can do this by running:

**SELECT * FROM layoffs;**

Since data cleaning often requires modifications such as removing, updating, and renaming values, it’s prudent to work on a duplicate copy of the original table. I created a new table named layoffs_staging as a working copy:

**CREATE TABLE layoffs_staging AS SELECT * FROM world_layoffs;**

After copying the data, I viewed the new table to ensure everything transferred correctly. Now, the cleaning process can begin.

**Step 2: Identify and Remove Duplicate Rows**
The first step in data cleaning is identifying duplicates. To accomplish this, I used the ROW_NUMBER() function. This assigns a unique row number to each record based on specific columns, helping to flag duplicate entries:

**WITH duplicate_cte AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY <columns_to_check> ORDER BY <columns_to_check>) AS row_num
    FROM layoffs_staging
)
SELECT * FROM duplicate_cte WHERE row_num > 1;**

However, attempting to delete duplicates directly within a CTE will trigger the error: "The target table duplicate_cte of the DELETE is not updatable." To bypass this, I created a new table to hold all data without duplicates, ensuring a clean slate for deletion:
**
DELETE FROM layoffs_staging
WHERE <condition_to_identify_duplicates>;**

**Step 3: Standardize Data**
Next, I standardized the dataset by performing the following transformations:
a) Remove Leading/Trailing Whitespace: I used the TRIM() function to eliminate unnecessary white spaces:

**UPDATE layoffs_staging SET column_name = TRIM(column_name);**

b) Unify Similar Categories: To ensure consistency across the dataset, I used the LIKE operator to match patterns and standardize similar values. For example, I merged the categories "Crypto" and "CryptoCurrency" under the unified term "Crypto":

**UPDATE layoffs_staging
SET category = 'Crypto'
WHERE category LIKE 'Crypto%';**

**Step 4: Convert Text to Date Format**
One of the columns, "date", was stored in a text format. To convert this into a proper date format, I first used the STR_TO_DATE() function, specifying the original format ('%m/%d/%Y'), and then applied the ALTER command to change the column's data type to DATE:

**UPDATE layoffs_staging
SET date = STR_TO_DATE(date, '%m/%d/%Y');
ALTER TABLE layoffs_staging
MODIFY COLUMN date DATE;**

**Step 5: Handle Null and Blank Values**
I checked for null values using the IS NULL constraint and handled blank entries by filtering with an empty string. For instance, to locate rows where the "industry" column was blank:

**SELECT * FROM layoffs_staging
WHERE industry = '';**

Once identified, I either updated these rows with appropriate values or removed them, depending on their relevance.

**Step 6: Remove Irrelevant Columns and Rows**

**DELETE FROM layoffs_staging WHERE <condition>;
ALTER TABLE layoffs_staging DROP COLUMN <column_name>;**

**Final Steps: Review and Continue**
With the data cleaned, the last step was to review the cleaned table to confirm all transformations were successful. From here, you can either proceed to Exploratory Data Analysis (EDA) or prepare the dataset for visualization and reporting.
In the final step, I deleted any columns or rows deemed unnecessary. To remove rows, I used the DELETE command, while the ALTER command allowed me to drop irrelevant columns:

**SELECT * FROM layoffs_staging;**

Additional Notes
Import Data: First, import the CSV file into the layoffs_raw table in your MySQL database.
Run the Script: Execute the SQL script to clean the data as outlined.
View Cleaned Data: Query the layoffs_staging2 table to review the cleaned dataset before further analysis.
Happy exploring!
