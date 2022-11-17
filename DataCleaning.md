# 🧹 Cleaning Data in SQL Queries 

💡 This work is focused on cleaning the data using SQL

💡 The dataset is available [here](https://www.kaggle.com/tmthyjames/nashville-housing-data).

## 📚 Table of Contents
- [Data Exploration](#-data-exploration)
- [Standardize Date Format](#-standardize-date-format)
- [Populate Property Address data](#-populate-property-address-data)
- [Breaking out Address into induvidual columns (Address, City)](#-breaking-out-address-into-induvidual-columns-address-city)
- [Breaking Owner Address into induvidual columns (Address, City and State)](#-breaking-owner-address-into-induvidual-columns-address-city-and-state)
- [Change Y and N to Yes and No in "Sold as Vacant" field](#-change-y-and-n-to-yes-and-no-in-sold-as-vacant-field)
- [Removing duplicates](#-removing-duplicates)
- [Delete Unused Columns](#-delete-unused-columns)


##  📌 Data Exploration

```sql
select count(*) as Qty_Lines from  dataCleaning.nashville_housing_data_for_data_cleaning 
```

|Qty_Lines|
|---------|
|56477|


```sql
describe  dataCleaning.nashville_housing_data_for_data_cleaning 
```

|COLUMN_NAME|COLUMN_TYPE|
|-----------|-----------|
|id|int(11)|
|ParcelID|varchar(50)|
|LandUse|varchar(50)|
|PropertyAddress|varchar(50)|
|SaleDate|date|
|SalePrice|int(11)|
|LegalReference|varchar(50)|
|SoldAsVacant|varchar(50)|
|OwnerName|varchar(100)|
|OwnerAddress|varchar(200)|
|Acreage|varchar(50)|
|TaxDistrict|varchar(50)|
|LandValue|int(11)|
|BuildingValue|int(11)|
|TotalValue|int(11)|
|YearBuilt|int(11)|
|Bedrooms|int(11)|
|FullBath|int(11)|
|HalfBath|int(11)|

```sql
select *
from dataCleaning.nashville_housing_data_for_data_cleaning 
limit 5
```

|id|ParcelID|LandUse|PropertyAddress|SaleDate|SalePrice|LegalReference|SoldAsVacant|OwnerName|OwnerAddress|Acreage|TaxDistrict|LandValue|BuildingValue|TotalValue|YearBuilt|Bedrooms|FullBath|HalfBath|
|--|--------|-------|---------------|--------|---------|--------------|------------|---------|------------|-------|-----------|---------|-------------|----------|---------|--------|--------|--------|
|2045|007 00 0 125.00|SINGLE FAMILY|1808  FOX CHASE DR, GOODLETTSVILLE|2013-04-09|240000|20130412-0036474|No|FRAZIER, CYRENTHA LYNETTE|1808  FOX CHASE DR, GOODLETTSVILLE, TN|2,3|GENERAL SERVICES DISTRICT|50000|168200|235700|1986|3|3|0|
|16918|007 00 0 130.00|SINGLE FAMILY|1832  FOX CHASE DR, GOODLETTSVILLE|2014-06-10|366000|20140619-0053768|No|BONER, CHARLES & LESLIE|1832  FOX CHASE DR, GOODLETTSVILLE, TN|3,5|GENERAL SERVICES DISTRICT|50000|264100|319000|1998|3|3|2|
|54582|007 00 0 138.00|SINGLE FAMILY|1864 FOX CHASE  DR, GOODLETTSVILLE|2016-09-26|435000|20160927-0101718|No|WILSON, JAMES E. & JOANNE|1864  FOX CHASE DR, GOODLETTSVILLE, TN|2,9|GENERAL SERVICES DISTRICT|50000|216200|298000|1987|4|3|0|
|43070|007 00 0 143.00|SINGLE FAMILY|1853  FOX CHASE DR, GOODLETTSVILLE|2016-01-29|255000|20160129-0008913|No|BAKER, JAY K. & SUSAN E.|1853  FOX CHASE DR, GOODLETTSVILLE, TN|2,6|GENERAL SERVICES DISTRICT|50000|147300|197300|1985|3|3|0|
|22714|007 00 0 149.00|SINGLE FAMILY|1829  FOX CHASE DR, GOODLETTSVILLE|2014-10-10|278000|20141015-0095255|No|POST, CHRISTOPHER M. & SAMANTHA C.|1829  FOX CHASE DR, GOODLETTSVILLE, TN|2|GENERAL SERVICES DISTRICT|50000|152300|202300|1984|4|3|0|

In the initial analysis we see that the table has null data in many columns, the most important being <code>PropertyAddress</code>, which we have to resolve.
But first, let's standardize the format of the <code>SalesDate</code>, also adding the time.



##  📌 Standardize Date Format

```sql
select SaleDate , convert (nh.SaleDate , datetime)
from dataCleaning.nashville_housing_data_for_data_cleaning nh
```

|SaleDate|convert (nh.SaleDate , datetime)|
|--------|--------------------------------|
|2013-04-09|2013-04-09 00:00:00.000|
|2014-06-10|2014-06-10 00:00:00.000|
|2016-09-26|2016-09-26 00:00:00.000|
|2016-01-29|2016-01-29 00:00:00.000|
|2014-10-10|2014-10-10 00:00:00.000|


Updating the format in the table:

```sql
update dataCleaning.nashville_housing_data_for_data_cleaning nh
set SaleDate = convert (nh.SaleDate , datetime)
```



##  📌 Populate Property Address data

There are null addresses and doing order by with the <code>parcelID</code> we see that we have repetitions of them and addresses

```sql
select * 
from dataCleaning.nashville_housing_data_for_data_cleaning nh
where PropertyAddress is null
order by ParcelID  desc
```

|id|ParcelID|LandUse|PropertyAddress|
|--|--------|-------|---------------|
|51703|114 15 0A 030.00|RESIDENTIAL CONDO||
|51930|113 14 0A 002.00|VACANT RESIDENTIAL LAND||
|24197|110 03 0A 061.00|SINGLE FAMILY||
|15886|109 04 0A 080.00|VACANT RES LAND||
|14753|108 07 0A 026.00|RESIDENTIAL CONDO||


Let's make the table have a JOIN with itself to see if the same <code>ParcelID</code> always have the same addresses with unique<code>ID</code>

```sql
Select a.id ,a.ParcelID, a.PropertyAddress, b.id , b.ParcelID, b.PropertyAddress #, ISNULL(a.PropertyAddress,b.PropertyAddress)
From dataCleaning.nashville_housing_data_for_data_cleaning a
JOIN dataCleaning.nashville_housing_data_for_data_cleaning b
	on a.ParcelID = b.ParcelID
	AND a.id != b.id 
Where a.PropertyAddress is null 
limit 10 
```

|id|ParcelID|PropertyAddress|id|ParcelID|PropertyAddress|
|--|--------|---------------|--|--------|---------------|
|43076|025 07 0 031.00||38077|025 07 0 031.00|410  ROSEHILL CT, GOODLETTSVILLE|
|39432|026 01 0 069.00||22721|026 01 0 069.00|141  TWO MILE PIKE, GOODLETTSVILLE|
|45290|026 05 0 017.00||4521|026 05 0 017.00|208  EAST AVE, GOODLETTSVILLE|
|53147|026 06 0A 038.00||19828|026 06 0A 038.00|109  CANTON CT, GOODLETTSVILLE|
|43080|033 06 0 041.00||7003|033 06 0 041.00|1129  CAMPBELL RD, GOODLETTSVILLE|
|45295|033 06 0A 002.00||12406|033 06 0A 002.00|1116  CAMPBELL RD, GOODLETTSVILLE|
|48731|033 15 0 123.00||39439|033 15 0 123.00|438  W CAMPBELL RD, GOODLETTSVILLE|
|36531|034 03 0 059.00||33057|034 03 0 059.00|2117  PAULA DR, MADISON|
|36531|034 03 0 059.00||36532|034 03 0 059.00|2117  PAULA DR, MADISON|
|46919|034 07 0B 015.00||45329|034 07 0B 015.00|2524  VAL MARIE DR, MADISON|

To solve this, we update our table using COALESCE to copy the addresses into the NULL spaces

```sql
update dataCleaning.nashville_housing_data_for_data_cleaning a
	JOIN dataCleaning.nashville_housing_data_for_data_cleaning b
		on a.ParcelID = b.ParcelID
		AND a.id != b.id 
set a.PropertyAddress = coalesce (a.PropertyAddress,b.PropertyAddress)	
	Where a.PropertyAddress is null
```



##  📌 Breaking out Address into induvidual columns (Address, City)

```sql
select PropertyAddress
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1
``` 

|PropertyAddress|
|---------------|
|1808  FOX CHASE DR, GOODLETTSVILLE|


The delimiter is a comma, so let's use the SUBSTRING_INDEX function to separate the address of the property from your city:

```sql
select PropertyAddress , SUBSTRING_INDEX(PropertyAddress,',',1) as PropertyAddress 
, SUBSTRING_INDEX(PropertyAddress,',',-1) as PropertyCity
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1 
```

|PropertyAddress|PropertyAddress|PropertyCity|
|---------------|---------------|------------|
|1808  FOX CHASE DR, GOODLETTSVILLE|1808  FOX CHASE DR| GOODLETTSVILLE|


Now let's create new columns for the address and city and update the table with the data:

```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning -- Address
add PropertySplitAdress varchar(255);
UPDATE dataCleaning.nashville_housing_data_for_data_cleaning nh
SET PropertySplitAdress = SUBSTRING_INDEX(PropertyAddress,',',1)
```

```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning -- City
ADD PropertySplitCity NVARCHAR(255);
UPDATE dataCleaning.nashville_housing_data_for_data_cleaning
SET PropertySplitCity = SUBSTRING_INDEX(PropertyAddress,',',-1)
```


```sql
select PropertyAddress, PropertySplitAdress, PropertySplitCity
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1 
```

|PropertyAddress|PropertySplitAdress|PropertySplitCity|
|---------------|-------------------|-----------------|
|1808  FOX CHASE DR, GOODLETTSVILLE|1808  FOX CHASE DR| GOODLETTSVILLE|



##  📌 Breaking Owner Address into induvidual columns (Address, City and State)

```sql
select OwnerAddress
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1 
```

|OwnerAddress|
|------------|
|1808  FOX CHASE DR, GOODLETTSVILLE, TN|


As with <code>PropertyAddress</code>, the address, city and state are separated by a comma, so let's use the SUBSTRING_INDEX function to separate.


```sql
select OwnerAddress 
,SUBSTRING_INDEX(OwnerAddress,',',1) as OwnerAddress
,SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress,',',2), ",",-1) as OwnerCity
,SUBSTRING_INDEX(OwnerAddress,',',-1) as OwnerState
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1
```

|OwnerAddress|OwnerAddress|OwnerCity|OwnerState|
|------------|------------|---------|----------|
|1808  FOX CHASE DR, GOODLETTSVILLE, TN|1808  FOX CHASE DR| GOODLETTSVILLE|TN|


Now let's create new columns for the address, city and state and update the table with the data:

```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning-- Address
ADD OwnerSplitAdress VARCHAR(255);
UPDATE dataCleaning.nashville_housing_data_for_data_cleaning
SET OwnerSplitAdress = SUBSTRING_INDEX(OwnerAddress,',',1)
```

```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning -- City
ADD OwnerSplitCity VARCHAR(255);
UPDATE dataCleaning.nashville_housing_data_for_data_cleaning
SET OwnerSplitCity = SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress,',',2), ",",-1)
```


```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning -- State
ADD OwnerSplitState VARCHAR(255);
UPDATE dataCleaning.nashville_housing_data_for_data_cleaning
SET OwnerSplitState =SUBSTRING_INDEX(OwnerAddress,',',-1)
```


```sql
select OwnerAddress, OwnerSplitAdress, OwnerSplitCity, OwnerSplitState
from dataCleaning.nashville_housing_data_for_data_cleaning nh
limit 1 
```

|OwnerAddress|OwnerSplitAdress|OwnerSplitCity|OwnerSplitState|
|------------|----------------|--------------|---------------|
|1808  FOX CHASE DR, GOODLETTSVILLE, TN|1808  FOX CHASE DR| GOODLETTSVILLE| TN|



##  📌 Change Y and N to Yes and No in "Sold as Vacant" field


The query below shows that we have 4 different outputs in the column `SoldAsVacant`

```sql
select distinct SoldAsVacant 
from dataCleaning.nashville_housing_data_for_data_cleaning nhdfdc 
```

|SoldAsVacant|
|------------|
|No|
|N|
|Yes|
|Y|


```sql
select SoldAsVacant, count(SoldAsVacant)  
from dataCleaning.nashville_housing_data_for_data_cleaning 
group by 1
```

|SoldAsVacant|count(SoldAsVacant)|
|------------|-------------------|
|N|399|
|No|51403|
|Y|52|
|Yes|4623|

```sql
Select SoldAsVacant
, CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
From dataCleaning.nashville_housing_data_for_data_cleaning nhdfdc
```

```sql
Update dataCleaning.nashville_housing_data_for_data_cleaning nhdfdc
SET SoldAsVacant = CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   end
```

```sql
select SoldAsVacant, count(SoldAsVacant)  
from dataCleaning.nashville_housing_data_for_data_cleaning 
group by 1	 
```

|SoldAsVacant|count(SoldAsVacant)|
|------------|-------------------|
|No|51802|
|Yes|4675|



##  📌 Removing duplicates


To remove duplicates in the table we will use DELETE, JOIN and the ROW_NUMBER window function. This last function will be partitioned by `ParcelID`,`SalePrice` and `LegalReference`, so that equal values ​​have different line numbers.

```sql
DELETE dataCleaning.nashville_housing_data_for_data_cleaning 
FROM dataCleaning.nashville_housing_data_for_data_cleaning 
INNER JOIN (
Select *, ROW_NUMBER() OVER ( PARTITION BY ParcelID,SalePrice,LegalReference ORDER by id) duplicate
From dataCleaning.nashville_housing_data_for_data_cleaning ) dup
ON nashville_housing_data_for_data_cleaning.LegalReference = dup.LegalReference
WHERE dup.duplicate > 1
```

```sql
select count(*) 
from dataCleaning.nashville_housing_data_for_data_cleaning 
```

|count(*)|
|--------|
|56230|

Initially we had 56477 rows in the table, and now we have 56230 rows, that is, 247 duplicates were deleted.


##  📌 Delete Unused Columns

To finish our data cleaning, let's remove some columns that are not needed from the table.

```sql
ALTER TABLE dataCleaning.nashville_housing_data_for_data_cleaning
DROP COLUMN OwnerAddress
,DROP column TaxDistrict
,DROP column PropertyAddress
,DROP column SaleDate
```

 😄 Thank you, hope you enjoyed!


















