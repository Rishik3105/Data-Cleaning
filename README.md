# 🌍✨ Data Cleaning 🚀🌟

This script demonstrates a systematic approach to remove duplicate rows from a table named `country`. The process involves creating staging tables, identifying duplicates, and retaining only unique rows. Below is a detailed explanation of the workflow with some 💡 **best practices** 💡.

---

## 🏗️ **Creating a Staging Table** 🛠️✨
- A new table `country_staging` is created with the exact structure of the `country` table. 🎯  
  ```sql
  CREATE TABLE country_staging 
  LIKE country;
  ```

## 📥 **Copying Data into Staging** 📋🔄
- All rows from the `country` table are copied into the `country_staging` table. 📝  
  ```sql
  INSERT country_staging
  SELECT * 
  FROM country;
  ```

---

## 🔍 **Identifying Duplicates** 🕵️‍♂️✨
### 💡 Using `ROW_NUMBER()`:
- **`ROW_NUMBER()`** is applied with `PARTITION BY` to assign a unique number to each row based on duplicate fields. 🧠  
- If `row_num > 1`, it indicates a duplicate record. ⚠️
  ```sql
  SELECT *,
  ROW_NUMBER() OVER (
    PARTITION BY Code, Name, Continent, Region, SurfaceArea, IndepYear, Population, 
                 LifeExpectancy, GNP, GNPOld, LocalName, GovernmentForm, 
                 HeadOfState, Capital, Code2
  ) AS row_num
  FROM country_staging;
  ```

### ✅ **Best Practice**: Use CTEs for clarity and maintainability. 🔗✨
  ```sql
  WITH duplicate_cte AS (
    SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY Code, Name, Continent, Region, SurfaceArea, IndepYear, Population, 
                   LifeExpectancy, GNP, GNPOld, LocalName, GovernmentForm, 
                   HeadOfState, Capital, Code2
    ) AS row_num
    FROM country_staging
  )
  SELECT *
  FROM duplicate_cte
  WHERE row_num > 1;
  ```

---

## 🛠️ **Creating a New Table to Handle Duplicates** 🗂️✨
- A new table, `country_staging2`, is created to include an additional `row_num` column. 🌟  
  ```sql
  CREATE TABLE `country_staging2` (
    `Code` char(3) NOT NULL DEFAULT '',
    `Name` char(52) NOT NULL DEFAULT '',
    `Continent` enum('Asia','Europe','North America','Africa','Oceania','Antarctica','South America') NOT NULL DEFAULT 'Asia',
    `Region` char(26) NOT NULL DEFAULT '',
    `SurfaceArea` decimal(10,2) NOT NULL DEFAULT '0.00',
    `IndepYear` smallint DEFAULT NULL,
    `Population` int NOT NULL DEFAULT '0',
    `LifeExpectancy` decimal(3,1) DEFAULT NULL,
    `GNP` decimal(10,2) DEFAULT NULL,
    `GNPOld` decimal(10,2) DEFAULT NULL,
    `LocalName` char(45) NOT NULL DEFAULT '',
    `GovernmentForm` char(45) NOT NULL DEFAULT '',
    `HeadOfState` char(60) DEFAULT NULL,
    `Capital` int DEFAULT NULL,
    `Code2` char(2) NOT NULL DEFAULT '',
    PRIMARY KEY (`Code`),
    row_num INT
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
  ```

## 🚀 **Populating `country_staging2` with Data** 📊🔄
- Data is inserted into `country_staging2`, along with the `row_num` for identifying duplicates. ✨  
  ```sql
  INSERT INTO country_staging2
  SELECT *,
  ROW_NUMBER() OVER (
    PARTITION BY Code, Name, Continent, Region, SurfaceArea, IndepYear, Population, 
                 LifeExpectancy, GNP, GNPOld, LocalName, GovernmentForm, 
                 HeadOfState, Capital, Code2
  ) AS row_num 
  FROM country_staging;
  ```

---

## 🗑️ **Deleting Duplicate Rows** ❌🚫
- Disable safe update mode to allow deletion without a primary key constraint: ⚙️
  ```sql
  SET SQL_SAFE_UPDATES = 0;
  ```
- Remove rows where `row_num > 1`, keeping only unique records: ✂️  
  ```sql
  DELETE
  FROM country_staging2
  WHERE row_num > 1;
  ```

- Verify there are no duplicates: 🔎
  ```sql
  SELECT *
  FROM country_staging2
  WHERE row_num > 1;
  ```

---

## 🎯 **Key Features** ✨💼
- 🚀 **Efficient duplicate removal**: Uses `ROW_NUMBER()` for precise detection. 🌈
- 🔄 **Scalable process**: Easy to adapt for other tables or datasets. 🌍
- 🛠️ **Best practices**: Incorporates CTEs and staging tables for clarity and maintainability. 🌟

---

## 📝 **Note** 📌💡
- Ensure `SQL_SAFE_UPDATES` is disabled during the deletion process to avoid errors. 🚫
- Customize `PARTITION BY` columns based on the unique constraints of your dataset. 🛠️

---

✨ Enjoy a clean and duplicate-free dataset! ✨ 🚀🎉🌟
