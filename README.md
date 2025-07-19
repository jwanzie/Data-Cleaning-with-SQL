### Data Cleaning in SQL

**Dataset**: *https://www.kaggle.com/datasets/swaptr/layoffs-2022*

Data Cleaning is the process in which you manipulate raw data to get it in a more usable format to be able to perform Exploratory Data Analysis and create visualisations.

#### Steps;

1. Create a database and import the dataset into the database.
2. Remove duplicates within data.
3. Standardise the data and fix inconsistencies.
4. Deal with null or blank values within the data.
5. Remove unnecessary columns.

#### Step 1; Create Database

Create a new schema within MySQL and name it (**world_layoffs**).
Import tables(**layoffs** data) into the created schema and take a look at the imported data using the select statement.

#### Step 2; Remove Duplicates

Create a staging table(**layoffs_staging**) within your schema and insert everything from layoffs table to the staging table. This allows you to manipulate the data within the staging table without affecting the raw data.
Without a unique row id within the data set, use **row_number()** and name it as **row_num** and match it against the columns using **partition by** to be able to identify duplicates.
Filter against the row number using a cte(**duplicate_cte**) with a **row_num** greater than 1 indicating a duplicate.
Create a new staging table(**layoffs_staging2**) name columns similar to **layoffs_staging** including the new **row_num** column and assign datatype and insert everything from **layoffs_staging** plus the row number.
Within **layoffs_staging2** filter data where **row_num** is greater than 1 and delete it.

#### Step 3; Standardise Data

Standardising data is all about finding issues of inconsistency within the data and fixing it.
Working from **layoffs_staging2**, use **distinct** statement to check text data for any inconsistencies.
Use **trim()** statement and **%** to standardise text data within columns and update.
For date column, convert data to date format using **str_to_date()** using format **%m/%d/%Y**, update and alter table to convert column to date.

#### Step 4; Null/Blank Values

Use **is null** to select rows that contain null values within different columns and populate based on previous columns with similar values.
Set blank values to null and repeat the previous step.

#### Step 5; Unnecessary Columns

For columns not needed like the previously created **row_num** remove it using **drop column** statement.
