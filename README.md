# NashvilleHousing-Data-Cleaning-Project-SQL
Project Title: Nashville Housing Data Cleaning Project (SQL)
Description:
The Nashville Housing Data Cleaning Project focuses on utilizing SQL queries to clean and prepare the Nashville housing dataset for further analysis. The primary goal of this project is to ensure the dataset is free from inconsistencies, errors, and missing data, making it suitable for accurate analysis. The data cleaning process involves transforming, standardizing, and restructuring various fields such as property addresses, sale dates, and owner information. This project highlights the step-by-step approach to data cleaning, including date formatting, filling in missing values, splitting address fields, handling duplicates, and transforming categorical data.

Key Objectives:

Standardizing Date Formats:
The dataset contains sale date information, which may be in inconsistent formats. The goal is to ensure all dates follow the YYYY-MM-DD format for consistency and ease of use in future analysis.

SQL CONVERT() function is used to standardize the date format.
A new column, SaleDateConverted, is created to store the converted date format.

Update NashvilleHousing
SET SaleDate = CONVERT(Date, SaleDate)

ALTER TABLE NashvilleHousing
ADD SaleDateConverted Date;

Update NashvilleHousing
SET SaleDateConverted = CONVERT(Date, SaleDate);

Populating Missing Property Addresses:
Some rows in the dataset lack property addresses. The objective is to populate the missing values by matching ParcelID and filling in the missing address data from related records with the same ParcelID. SQL joins and the ISNULL() function are used to achieve this.

Update a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
From NashvilleHousing a
JOIN NashvilleHousing b
   ON a.ParcelID = b.ParcelID
   AND a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is null;

Splitting Address Fields:
Property addresses are stored as a single field, making it difficult to analyze components like street, city, and state separately. The objective is to split the PropertyAddress into multiple columns for better granularity and organization. SQL string manipulation functions such as SUBSTRING() and CHARINDEX() are used to extract parts of the address into separate fields.

ALTER TABLE NashvilleHousing
ADD PropertySplitAddress nvarchar(225);

Update NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1);

ALTER TABLE NashvilleHousing
ADD PropertySplitCity nvarchar(225);

Update NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress));

Handling Owner Information:
Similar to property addresses, owner addresses are also split into individual columns (Address, City, State) using SQL string functions like PARSENAME() and REPLACE().
This step improves the structure of the data, making it easier to analyze owner-related information separately.

ALTER TABLE NashvilleHousing
ADD OwnerSplitAddress nvarchar(225);

Update NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.') ,3);

ALTER TABLE NashvilleHousing
ADD OwnerSplitCity nvarchar(225);

Update NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.') ,2);

ALTER TABLE NashvilleHousing
ADD OwnerSplitState nvarchar(225);

Update NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.') ,1);

Transforming Categorical Data:
The field SoldAsVacant contains binary values ('Y' for yes and 'N' for no). To make this field more readable, these values are transformed into 'YES' and 'NO' using a CASE statement.

Update NashvilleHousing
SET SoldAsVacant = CASE 
    When SoldAsVacant = 'Y' THEN 'YES'
    When SoldAsVacant = 'N' THEN 'NO'
    ELSE SoldAsVacant
END;

Removing Duplicate Records:
Duplicate records can distort analysis, so the goal is to identify and remove duplicates based on multiple fields such as ParcelID, PropertyAddress, SalePrice, SaleDate, and LegalReference. A ROW_NUMBER() function is used to assign a row number to each record, and any row with a number greater than 1 is considered a duplicate.

WITH RowNumCTE AS (
Select *,
    ROW_NUMBER() OVER (
    PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
    ORDER BY UniqueID
) row_num
From NashvilleHousing
)
Select *
From RowNumCTE
Where row_num > 1;

Dropping Unnecessary Columns:
After cleaning the data and performing transformations, certain columns such as OwnerAddress, TaxDistrict, and PropertyAddress become redundant. These columns are dropped to streamline the dataset.

ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress;

Conclusion:
The Nashville Housing Data Cleaning Project demonstrates the power of SQL in transforming a raw, unstructured dataset into a clean and well-organized dataset that is ready for analysis. Through handling missing data, standardizing formats, splitting fields, transforming categorical data, and removing duplicates, this project showcases how data cleaning lays a solid foundation for accurate and insightful data analysis. The final dataset is now structured, error-free, and optimized for further real estate trend analysis, price prediction, or market segmentation studies.
