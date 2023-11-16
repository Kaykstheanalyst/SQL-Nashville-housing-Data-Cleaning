# SQL-Nashville-housing-Data-Cleaning
## Overview
This Project entails the cleaning of Nashville housing dataset. modifying the dataset to reveal insightful data that can be used for further analysis.

## Data Source
The datasets used for the analysis can be downloaded here [Nashville Housing Data for Data Cleaning.xlsx](https://github.com/Kaykstheanalyst/SQL-Nashville-housing-Data-Cleaning/files/13380022/Nashville.Housing.Data.for.Data.Cleaning.xlsx)

## Tools
-	SSMS
- Microsoft Excel

## Skills demonstrated
- Data Cleaning
- Data Transformation
- CTE's

```SQL
 /*

Cleaning Data in SQL Queries

*/

SELECT *
FROM Nashvillehousing


--------------------------------------------------------------------------------------------------------------------------

-- Standardize Date Format

SELECT SaleDateconverted, CONVERT(Date, saleDate)
FROM Nashvillehousing






-- If it doesn't Update properly


ALTER TABLE Nashvillehousing
ADD SaleDateconverted Date;

UPDATE Nashvillehousing
SET SaleDateconverted = CONVERT(Date, saleDate)




 --------------------------------------------------------------------------------------------------------------------------

-- Populate Property Address data

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashvillehousing a
JOIN Nashvillehousing b
     on a.ParcelID = b.ParcelID
	 AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL


UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Nashvillehousing a
JOIN Nashvillehousing b
     on a.ParcelID = b.ParcelID
	 AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL

SELECT *
FROM Nashvillehousing


--------------------------------------------------------------------------------------------------------------------------

-- Breaking out Address into Individual Columns (Address, City, State)


SELECT PropertyAddress
FROM Nashvillehousing
--Where PropertyAddress is null
--order by ParcelID

ALTER TABLE Nashvillehousing
ADD PropertysplitAddress NVARCHAR (255);
GO
UPDATE Nashvillehousing
SET PropertysplitAddress = SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

ALTER TABLE Nashvillehousing
ADD PropertysplitCity NVARCHAR (255);
GO
UPDATE Nashvillehousing
SET PropertysplitCity = SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress))

SELECT
SUBSTRING (PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) AS  Address,
SUBSTRING (PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) AS  Address
FROM Nashvillehousing


SELECT *
FROM Nashvillehousing

select
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM Nashvillehousing

ALTER TABLE Nashvillehousing
ADD OwnerSplitAddress NVARCHAR(255);
GO
UPDATE Nashvillehousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)
GO
ALTER TABLE Nashvillehousing
ADD OwnerSplitCity NVARCHAR(255);
GO
UPDATE Nashvillehousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)
GO
ALTER TABLE Nashvillehousing
ADD OwnerSplitState NVARCHAR(255);
GO
UPDATE Nashvillehousing
SET OwnerSplitState= PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)


--------------------------------------------------------------------------------------------------------------------------


-- Change Y and N to Yes and No in "Sold as Vacant" field

SELECT DISTINCT(SoldAsVacant)
FROM Nashvillehousing

SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM Nashvillehousing
GROUP BY SoldAsVacant
ORDER BY 2

SELECT SoldAsVacant,
CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
WHEN SoldAsVacant = 'N' THEN 'No'
ELSE SoldAsVacant
END
FROM Nashvillehousing

UPDATE Nashvillehousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
WHEN SoldAsVacant = 'N' THEN 'No'
ELSE SoldAsVacant
END



-----------------------------------------------------------------------------------------------------------------------------------------------------------

-- Remove Duplicates

WITH RowNumCTE AS (
SELECT *,
ROW_NUMBER() OVER (
PARTITION BY ParcelID,
             PropertyAddress,
			 SalePrice,
			 SaleDateconverted,
			 LegalReference
			 ORDER BY UniqueID 
			 ) row_num
FROM Nashvillehousing
)
 SELECT *
 FROM RowNumCTE
 WHERE row_num > 1
 ORDER BY PropertyAddress





---------------------------------------------------------------------------------------------------------

-- Delete Unused Columns


ALTER TABLE Nashvillehousing
DROP COLUMN PropertyAddress, OwnerAddress, TaxDistrict



SELECT *
FROM Nashvillehousing


-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------

--- Importing Data using OPENROWSET and BULK INSERT	

--  More advanced and looks cooler, but have to configure server appropriately to do correctly
--  Wanted to provide this in case you wanted to try it


--sp_configure 'show advanced options', 1;
--RECONFIGURE;
--GO
--sp_configure 'Ad Hoc Distributed Queries', 1;
--RECONFIGURE;
--GO


--USE PortfolioProject 

--GO 

--EXEC master.dbo.sp_MSset_oledb_prop N'Microsoft.ACE.OLEDB.12.0', N'AllowInProcess', 1 

--GO 

--EXEC master.dbo.sp_MSset_oledb_prop N'Microsoft.ACE.OLEDB.12.0', N'DynamicParameters', 1 

--GO 


---- Using BULK INSERT

--USE PortfolioProject;
--GO
--BULK INSERT nashvilleHousing FROM 'C:\Temp\SQL Server Management Studio\Nashville Housing Data for Data Cleaning Project.csv'
--   WITH (
--      FIELDTERMINATOR = ',',
--      ROWTERMINATOR = '\n'
--);
--GO


---- Using OPENROWSET
--USE PortfolioProject;
--GO
--SELECT * INTO nashvilleHousing
--FROM OPENROWSET('Microsoft.ACE.OLEDB.12.0',
--    'Excel 12.0; Database=C:\Users\alexf\OneDrive\Documents\SQL Server Management Studio\Nashville Housing Data for Data Cleaning Project.csv', [Sheet1$]);
--GO
```

















