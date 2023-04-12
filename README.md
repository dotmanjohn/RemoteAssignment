Remote Assignment

Readme Github link - https://github.com/dotmanjohn/RemoteAssignment/blob/main/README.md
****************************************************************************
**Database Connection**

A Postgres database was created on local and connection was made to the database via a pgadmin client.

Host: 172.17.0.2

Username: postgres

Password: postgresEnter
****************************************************************************
**Scope**

There are 4 sheets on the spreadsheet provided for this assignment i.e., 4 tables to be created within the database, DimCustomer, DimProduct, DimSalesTerritory and FactResellerSales
****************************************************************************
**Table Creation Scripts**
1.	DimCustomer
```
CREATE TABLE public."DimCustomer"
(
    "Customerkey" bigint NOT NULL,
    "GeographyKey" bigint,
    "CustomerAlternateKey" varchar,
    "Title" varchar,
    "FirstName" varchar,
    "MiddleName" varchar,
    "LastName" varchar,
    "NameStyle" varchar,
    "BirthDate" varchar,
    "MaritalStatus" varchar,
    "Suffix" varchar,
    "Gender" varchar,
    "EmailAddress" varchar,
    "YearlyIncome" numeric,
    "TotalChildren" bigint,
    "NumberChildrenAtHome" bigint,
    "EnglishEducation" varchar,
    "SpanishEducation" varchar,
    "FrenchEducation" varchar,
    "EnglishOccupation" varchar,
    "SpanishOccupation" varchar,
    "FrenchOccupation" varchar,
    "HouseOwnerFlag" boolean,
    "NumberCarsOwned" bigint,
    "AddressLine1" varchar,
    "AddressLine2" varchar,
    "Phone" varchar,
    "DateFirstPurchase" varchar,
    "CommuteDistance" varchar,
    CONSTRAINT "DimCustomer_pkey" PRIMARY KEY ("Customerkey")
)
```
2.	DimProduct
```
CREATE TABLE public."DimProduct"
(
    "ProductKey" bigint NOT NULL,
    "ProductAlternateKey" varchar,
    "ProductSubcategoryKey" varchar,
    "WeightUnitMeasureCode" varchar,
    "SizeUnitMeasureCode" varchar,
    "ProductName" varchar,
    "StandardCost" varchar,
    "FinishedGoodsFlag" boolean,
    "Color" varchar,
    "SafetyStockLevel" bigint,
    "ReorderPoint" bigint,
    "ListPrice" varchar,
    "Size" varchar,
    "SizeRange" varchar,
    "Weight" varchar,
    "DaysToManufacture" bigint,
    "ProductLine" varchar,
    "DealerPrice" varchar,
    "Class" varchar,
    "Style" varchar,
    "ModelName" varchar,
    "StartDate" varchar,
    "EndDate" varchar,
    "Status" varchar,
    CONSTRAINT "DimProduct_pkey" PRIMARY KEY ("ProductKey")
)
```

3.	DimSalesTerritory
```
CREATE TABLE public."DimSalesTerritory"
(
   "SalesTerritoryKey" bigint NOT NULL,
    "SalesTerritoryAlternateKey" bigint,
    "SalesTerritoryRegion" varchar,
    "SalesTerritoryCountry" varchar,
    "SalesTerritoryGroup" varchar,
    CONSTRAINT "DimSalesTerritory_pkey" PRIMARY KEY ("SalesTerritoryKey")
)
```
4.	FactResellerSales
```
CREATE TABLE IF NOT EXISTS public."FactResellerSales"
(
   "ProductKey" bigint NOT NULL,
    "OrderDateKey" bigint,
    "DueDateKey" bigint,
    "ShipDateKey" bigint,
    "ResellerKey" bigint,
    "EmployeeKey" bigint,
    "PromotionKey" bigint,
    "CurrencyKey" bigint,
    "SalesTerritoryKey" bigint,
    "SalesOrderNumber" varchar,
    "SalesOrderLineNumber" bigint,
    "RevisionNumber" bigint,
    "OrderQuantity" bigint,
    "UnitPrice" numeric,
    "ExtendedAmount" numeric,
    "UnitPriceDiscountPct" numeric,
    "DiscountAmount" numeric,
    "ProductStandardCost" numeric,
    "TotalProductCost" numeric,
    "SalesAmount" numeric,
    "TaxAmt" numeric,
    "Freight" numeric,
    "CarrierTrackingNumber" varchar,
    "CustomerPONumber" varchar,
    "OrderDate" timestamp,
    "DueDate" timestamp,
    "ShipDate" timestamp
)
```
<img width="216" alt="01  Tables" src="https://user-images.githubusercontent.com/63157768/231432638-eccd8a1a-6ccd-4f88-bf5d-ed617dc02f70.png">

****************************************************************************

**Exploratoty Data Analysis**

**Brief Notes**

•	DimCustomer has 18,484 records with 18,484 unique records and no null in the Customerkey column, Customerkey is assigned primarykey.
DimProduct and DimSalesTerritory also have 569 records and 11 records respectively, with ProductKey and SalesTerritoryKey records all unique and without nulls, hence ProductKey and SalesTerritoryKey are assigned primarykey on both tables.

•	FactResellerSales does not have a unique identifier e.g. FactResellerSalesKey which could also be considered a transaction reference for easier identification or querying of individual record. A fix for this will be to concatenate “SalesOrderNumber” and “SalesOrderLineNumber” for a unique reference.

•	Date columns on DimCustomer (“BirthDate”and “FirstPurchasedate”) and DimProduct (“StartDate” and “EndDate”) are not in the right SQL date format and as such could not be imported as date columns into the Postgres database, the columns had to be created as varchar type. The date columns can be transformed accordingly using a combination of the split_part(), concat() and to_date() functions, the birthdate is transformed while solving for question 3.

•	DimCustomer doesn’t have a “LastPurchaseDate”, this is useful for analyzing Customer behavior for various purposes for example marketing, calculating retention rate, etc.
N:B: It is assumed that “FirstPurchaseDate” is also “Registration or Sign up Date”, where customer information is captured when they make their first purchase, otherwise if customers are able to sign up without necessarily making a purchase, then it is important to provide the actual sign up date whuch is an important information for analysis.

•	There are 33 Products (ProductKey) on FactResellerSales that do not exist on the DimProduct table i.e., DimProduct is incomplete as this should be the primary directory for all products.

•	FactResellerSales contains a “CurrencyKey” column signifying sales were made in different currencies but the table does not contain an exchange rate column (or provision for an exchange rate table) for conversion of sales to a unitary currency for unified sales reporting. This means that for the questions that follow, where amount is concerned, it will be assumed that all sales are completed in the same currency.

•	On FactResellerSales, order date recorded only has a day or two for each month across the years which is mostly at month end (28,29,30,31) and a few instances of orders made on the 1st of March, May, July and August of 2011. This suggests orders are only made on these 5 dates or the data is incomplete.

•	There is no relationship between DimCustomer and FactResellerSales therefore there is no way to analyze sales by customers such as identifying the most profitable customers.

•	Missing relationship between DimCustomer and DimSalesTerritory, GeographyKey and AddressLine columns on DimCustomer are insufficient in establishing customer country, this prevents analyzing customers by country.

****************************************************************************

**EDA Scripts and Findings**

********************************************************************
•	House Ownership Distribution
```
SELECT "HouseOwnerFlag" , count(1) num_owners
FROM "DimCustomer"
group by "HouseOwnerFlag";
```
<img width="278" alt="02  House owner distribution" src="https://user-images.githubusercontent.com/63157768/231433018-629354b2-d83a-47b8-80e4-12292adc1c48.png">

Finding: Majority of the customers own a house, 12,502 against 5,982.
********************************************************************

•	House Ownership Distribution by Yearly Income
```
with base as
(SELECT "YearlyIncome" , "HouseOwnerFlag" , count(1) owner_count 
FROM "DimCustomer"
group by "YearlyIncome" , "HouseOwnerFlag"),
 
tot as (select "YearlyIncome", count(1) total_count
from "DimCustomer" group by 1)

select b.* , round((owner_count::numeric/total_count::numeric)*100,2) pct
from base b 
left join tot t on t."YearlyIncome" = b."YearlyIncome"
order by 1,3 desc ;
```
<img width="335" alt="03  House owner by income" src="https://user-images.githubusercontent.com/63157768/231433136-7191df58-6268-424e-a626-0155119a714f.png">

Finding: Across all income levels except those who earn 160,000, there are more house owners, we can assume that owing a house is essential to the customers and income level does not prevent them from doing so. 
********************************************************************

•	Commute distance distribution
```
SELECT "CommuteDistance" , count(1) num_owners
FROM "DimCustomer"
group by "CommuteDistance";
```
<img width="200" alt="17  Commute Distance" src="https://user-images.githubusercontent.com/63157768/231437153-3fe0f3bc-e671-41a6-a25f-75b769e8c15c.png">

Finding: The largest group of customers commute within 0-1 miles, followed by 2-5 miles while the least group commute 10+ miles.
********************************************************************

•	Level of education distribution
```
SELECT "EnglishEducation" , count(1) cust_count
FROM "DimCustomer"
group by 1;
```
<img width="182" alt="04  Education Distribution" src="https://user-images.githubusercontent.com/63157768/231433271-cfcd550d-f420-4be3-8c6e-bd12f85d099b.png">

Finding: The customers are fairly distributed across the various education levels except those who have a partial high school education and make up the least of the distribution
********************************************************************

•	Occupation distribution
```
SELECT "EnglishOccupation" , count(1) cust_count
FROM "DimCustomer"
group by 1;
```
<img width="186" alt="05  Occupation distribution" src="https://user-images.githubusercontent.com/63157768/231433309-639233be-6c04-4c37-8931-db25880aa22f.png">

Finding: The customer set is dominated by Professionals followed by Skilled Manual, while manual workers are the least represented.
********************************************************************

•	Level of education and occupation distribution
```
with base as (SELECT "EnglishEducation" , "EnglishOccupation" , count(1) cust_count
FROM "DimCustomer"
group by 1, 2),

tot as (SELECT "EnglishEducation" , count(1) total_count
FROM "DimCustomer"
group by 1)

select b."EnglishEducation", b."EnglishOccupation", b.cust_count, 
round((cust_count::numeric/total_count::numeric)*100,2) pct
from base b 
 left join tot on tot."EnglishEducation" = b."EnglishEducation"
 order by 1, 3 desc;
```
<img width="332" alt="06  Education and Occupation distribution" src="https://user-images.githubusercontent.com/63157768/231433394-9e4b35b4-ca60-4783-8cc3-aea27749b4c5.png">

Finding: The 5 occupation types are represented across all education levels but at varying proportion as it should be expected, with the more educated levels (Bachelors and Graduate Degree) having a larger proportion of Professionals and Management while those with lower education profile barely have representation in these senior roles. Those who only have partial high school education are mostly clerical or manual workers.
********************************************************************

•	YearlyIncome distribution by EnglishOccupation
```
SELECT "EnglishOccupation", avg("YearlyIncome") avg_income , max("YearlyIncome") max_income, 
min("YearlyIncome") min_income, count(1) cust_count
FROM "DimCustomer"
group by "EnglishOccupation"
order by 2;
```

<img width="418" alt="07  Yearly Income by Occupation" src="https://user-images.githubusercontent.com/63157768/231433572-2510c893-a22c-4af8-9141-634e9884f07c.png">

Finding: We can expect that as the occupation hierarchy increases, the average income/ income range of the customers will also increase. The manual customers earn the least, and the clerical customers earn higher than manual customers on average, followed by the skilled manual customers, the professionals and then management who earn the highest on average.
********************************************************************

•	YearlyIncome distribution by EnglishEducation
```
SELECT "EnglishEducation", avg("YearlyIncome") avg_income , max("YearlyIncome") max_income, 
min("YearlyIncome") min_income, count(1) cust_count
FROM "DimCustomer"
group by "EnglishEducation"
order by 2;
```
<img width="424" alt="08  Yearly Income by Education" src="https://user-images.githubusercontent.com/63157768/231433629-c4ebc3a5-89d1-4404-878d-dcc75105dc78.png">

Finding: Those with the least education level i.e., partial high school earn the least on average, while those with the highest levels, a bachelors or graduate degree earn the highest with a close average as these levels mostly comprises of professional and management customers.
********************************************************************

•	YearlyIncome distribution by NumberCarsOwned
```
SELECT "NumberCarsOwned", avg("YearlyIncome") avg_income , max("YearlyIncome") max_income, 
min("YearlyIncome") min_income, count(1) cust_count
FROM "DimCustomer"
group by "NumberCarsOwned"
order by "NumberCarsOwned";
```
<img width="433" alt="09  Yearly Income by Car Ownership" src="https://user-images.githubusercontent.com/63157768/231433673-e8706f96-7dc5-4dbe-b886-a5f1c38bb9ce.png">

Finding: The more the customers earn on average, the more cars they own.
********************************************************************

•	YearlyIncome distribution by TotalChildren
```
_SELECT "TotalChildren", avg("YearlyIncome") avg_income , max("YearlyIncome") max_income, 
min("YearlyIncome") min_income, count(1) cust_count
FROM "DimCustomer"
group by "TotalChildren"
order by "TotalChildren";
```
<img width="418" alt="10  Yearly Income by Children" src="https://user-images.githubusercontent.com/63157768/231433700-9bef593b-ebac-4c9b-b2d2-e99a9dbef30f.png">

Finding: Customers who earn higher on average have more children.
********************************************************************

•	Top 10 ordered products by OrderQuantity
```
select rs."ProductKey", "ProductName", count(1) num_times_ordered, sum("OrderQuantity") order_qty
FROM public."FactResellerSales" rs
left join "DimProduct" dp on dp."ProductKey" = rs."ProductKey"
group by 1,2
order by sum("OrderQuantity") desc
limit 10;
```
<img width="394" alt="11  Top 10 ordered Products by Quantity" src="https://user-images.githubusercontent.com/63157768/231433736-b9aeb3d9-f3b1-445d-8619-c85d02561763.png">

Finding: The most ordered product by quantity is “Classic Vest, S” with productkey 471, followed by “Short-Sleeve Classic Jersey, XL” with product key 491 and so on.
********************************************************************

•	Top 10 ordered products by SalesAmount and ProductKey

(ASSUMING ALL ORDERS WERE MADE IN THE SAME CURRENCY)
```
select rs."ProductKey", "ProductName", count(1) num_times_ordered, sum("OrderQuantity") order_qty, sum("SalesAmount") salesamount
FROM public."FactResellerSales" rs
left join "DimProduct" dp on dp."ProductKey" = rs."ProductKey"
group by 1,2
order by sum("SalesAmount") desc
limit 10;
```
<img width="440" alt="12  Top 10 Ordered Product Key by SalesAmount" src="https://user-images.githubusercontent.com/63157768/231433853-5e78bbbd-423c-46ac-b1f8-d1a4a0962cee.png">

Finding: Going by the above assumption, “Mountain-200 Black, 38” with productkey 359 is the most ordered product by sales amount, “Mountain-200 Black, 38” also comes next but under a different productkey (358). However, taking the product name and productalternatekey into consideration, to avoid having the same product, even though differentiated by productkey, appear multiple times, we can reference the ProductAlternateKey for a more instructive result.

(By Product Name)
```
select dp."ProductAlternateKey", "ProductName", count(1) num_times_ordered, sum("OrderQuantity") order_qty, sum("SalesAmount") salesamount
FROM public."FactResellerSales" rs
left join "DimProduct" dp on dp."ProductKey" = rs."ProductKey"
group by 1,2
order by sum("SalesAmount") desc
limit 10;
```
<img width="481" alt="13  Top 10 Ordered Product Name by SalesAmount" src="https://user-images.githubusercontent.com/63157768/231433918-f42b8800-f007-4b2f-af3e-c9f888966c5b.png">

The output shows “Mountain-200 Black, 38” as the second most ordered product followed by “Mountain-200 Black, 42”
Note: The top ordered product(s) appearing as null constitutes the 33 identified productkeys that do not exist on DimProduct, hence we can say “Mountain-200 Black, 38” is the most ordered product.
********************************************************************

•	Products not listed on DimProduct
```
select distinct rs."ProductKey"
from "FactResellerSales" rs
where rs."ProductKey" not in (select "ProductKey" from "DimProduct")
```
<img width="152" alt="16  Missing Products from DimProduct" src="https://user-images.githubusercontent.com/63157768/231433978-77bb620a-f8cc-4b34-92b7-9b13140fa6e9.png">

Finding: The above script returns 33 productkeys from FactResellerSales that are not found on DimProduct, this establishes the incompleteness of DimProduct
********************************************************************

•	CurrencyKey distribution across SalesTerritoryCountry
```
select distinct st."SalesTerritoryKey", "SalesTerritoryCountry", "CurrencyKey", count(1)
FROM public."FactResellerSales" rs
left join "DimSalesTerritory" st on st."SalesTerritoryKey" = rs."SalesTerritoryKey"
group by 1,2,3
order by 1;
```
<img width="360" alt="15  CurrencyKey by Country" src="https://user-images.githubusercontent.com/63157768/231434015-71e115eb-8048-4a08-a3aa-92cac7a65db0.png">

Finding: Based on the output of this script, it will be safe to assume key 100 is USD as it is mostly recorded across United States territories and a few times in France, 98 as British Pounds as it only has record in the UK, 36 is Euro recorded in France and Germany and both countries spend Euro, 19 is CAD recorded only in Canada, and 6 is AUD recorded only in Australia.
********************************************************************

•	SalesTerritoryCountry by SalesAmount

(ASSUMING ALL ORDERS WERE MADE IN THE SAME CURRENCY)
```
select st."SalesTerritoryCountry", count(1) num_times_ordered, sum("OrderQuantity") order_qty, sum("SalesAmount") salesamount
FROM public."FactResellerSales" rs
left join "DimSalesTerritory" st on st."SalesTerritoryKey" = rs."SalesTerritoryKey"
group by 1
order by sum("OrderQuantity") desc
limit 10;
```
<img width="364" alt="14  Top Ordered Products by Country" src="https://user-images.githubusercontent.com/63157768/231434064-baa2ae33-82ad-41cd-83e9-0476d6fe7def.png">

Finding: The US is the most active Country in quantity and sales, followed by Canada while Australia is the least active Country.
********************************************************************


**SQL QUESTIONS (Scripts and Explanation)**
********************************************************************
**1.	What is the highest transaction of each month in 2012 for the product Sport-100 Helmet, Red?**

•	Tables containing the information are DimProduct, FactResellerSales

•	Do not use max() SQL function

**Script:**
```
select Month_Year, SalesAmount, OrderDate 
from
(select to_char("OrderDate",'MM-YYYY') Month_Year,
"SalesAmount" SalesAmount, date("OrderDate") OrderDate,
row_number() over (partition by to_char("OrderDate",'MM-YYYY')
				  order by "SalesAmount" desc) rn
from "FactResellerSales" rs
left join "DimProduct" dp on dp."ProductKey" = rs."ProductKey"
where "ProductName" = 'Sport-100 Helmet, Red'
and date("OrderDate") between '2012-01-01' and '2012-12-31'
)pp
where rn =1
```
<img width="242" alt="Question 1" src="https://user-images.githubusercontent.com/63157768/231440853-547812af-9e6d-4bb3-b54d-043308bf3c00.png">

**Explanation:**

To return the highest transaction for each month in the year, data is partitioned by month to return a rank by SalesAmount in descending order i.e., every transaction is ranked on a monthly basis with the highest transaction ranked as 1. The conditions in the where clause is as instructed, date range is 2012 and ProductName is “Sport-100 Helmet, Red”.
This is then nested as a subquery and the outer query selects the desired columns with a condition to only return the transactions ranked 1.
The result is the highest transaction for the desired product as specified in the where clause for each month in 2012.
********************************************************************

**2.	Find all the products and their total sales amount by month. Only consider products that had at least one sale in 2012.**

•	Tables containing the information are DimProduct, FactResellerSales

•	Expected output columns are ProductKey, SalesAmount, OrderMonth

**Script:**
```
select "ProductKey", 
 sum("SalesAmount") SalesAmount,
 to_char("OrderDate",'MM-YYYY') Month_Year
from "FactResellerSales"
where date("OrderDate") between '2012-01-01' and '2012-12-31'
group by 1,3
```
<img width="267" alt="Question 2" src="https://user-images.githubusercontent.com/63157768/231440997-1f4a7b98-edcd-4ccd-b88f-65bbe5821adb.png">

**Explanation:**

The script is written with an aggregation on SalesAmount grouped by ProductKey and Month to return the total sales amount by month for all products, the condition given is to return a result that only considers products that were sold in 2012 which is passed in the where clause.
The result is the monthly total sales amount of all the products sold in 2012.
********************************************************************

**3.	What are the age groups of customers categorized by marital status and gender?**

•	Tables containing the information is DimCustomer

•	Build the query under the assumption that current date is 2070-01-01

•	Bucket them by Age Groups: a) Less than 35, b) Between 35 and 50, c) Greater than 50. Segregate the Number of Customers in each age group by Marital Status and Gender.

**Script:**
```
with base as 
(select "Customerkey", "MaritalStatus", "Gender",
to_date(
	concat(split_part("BirthDate", '/', 3),
		   '-', 
		   case when length(split_part("BirthDate", '/', 2))<2 then
		   	concat('0',split_part("BirthDate", '/', 2)) else split_part("BirthDate", '/', 2) end,
		   '-',
			case when length(split_part("BirthDate", '/', 1))<2 then
		   	concat('0',split_part("BirthDate", '/', 1)) else split_part("BirthDate", '/', 1) end), 
	'yyyy-mm-dd') birthday,
	to_date('2070-01-01', 'yyyy-mm-dd') curr_date
FROM public."DimCustomer"
 ),

age as 
	(select *, age(curr_date, birthday) age
		from base),
		
age_grp as
(select *, 
case when extract( year from age) < 35 then 'AgeBelow35'
	when extract( year from age) between 35 and 50 then 'AgeBetween35and50'
 	else 'AgeAbove50' end age_group
 from age
	)

select "MaritalStatus", "Gender",
case when age_group = 'AgeBelow35' then count(1) else 0 end AgeBelow35,
case when age_group = 'AgeBetween35and50' then count(1) else 0 end AgeBetween35and50,
case when age_group = 'AgeAbove50' then count(1) else 0 end AgeAbove50
 from age_grp
 group by "MaritalStatus", "Gender",age_group
 order by "MaritalStatus", "Gender"
```
<img width="438" alt="Question 3" src="https://user-images.githubusercontent.com/63157768/231435005-69ae679e-aac6-493f-9426-90636c354cda.png">

**Explanation:**

To solve for this question, CTEs were used, the first CTE (base) returns all the relevant columns. There is a need to group customers by age and to get customer age which is not readily available, the age has to be calculated using the birthdate column, however the birthdate column is not in the right date format hence there is a need to transform the column into the right format, this is done using the split_part function and then concatenating the date parts (year, month and date) to get the right format.

The transformed and newly created birthday column is then used to calculate customers age in the second CTE (age), for this, the age() function is used to get the interval between current date (‘2070-01-01’) and birthday.

The third CTE (age_grp) then groups customers into the 3 age buckets required by extracting customers age using the extract() function on the age interval gotten in age CTE. This is completed with a case statement.

Finally the age_grp CTE is leveraged to query the desired result returning the MaritalStatus and Gender of customers alongside the 3 age groups and the number of customers in each bucket.
********************************************************************

**4.	What is the combination of product & country with the lowest amount of sales per month in 2012?**

•	Tables containing the information are DimProduct, DimSalesTerritory, FactResellerSales

•	SalesTerritoryCountry & ProductName should be included

**Script:**
```
with base as (
	select dp."ProductName", 
	to_char("OrderDate",'MM-YYYY') Month_Year,
	"SalesTerritoryCountry", sum("SalesAmount") amount
	from "FactResellerSales" rs
	left join "DimProduct" dp on dp."ProductKey" = rs."ProductKey" 
	left join "DimSalesTerritory" st on st."SalesTerritoryKey" = rs."SalesTerritoryKey"
	where date("OrderDate") between '2012-01-01' and '2012-12-31'
	group by 1,2,3
),

sales_rank as (
	select *, row_number() over (partition by Month_Year order by amount) rn
	from base
	)

select Month_Year, "SalesTerritoryCountry", "ProductName", amount SalesAmount
from sales_rank
where rn = 1 
```

<img width="396" alt="Question 4" src="https://user-images.githubusercontent.com/63157768/231441229-5e86a915-4c4c-4a2d-b236-3a0ca21ca7e5.png">

**Explanation:**

Similar to question 3, CTEs are also used in solving for this question. 
The base CTE returns the relevant columns to this question, an aggregated (sum) SalesAmount column grouped accordingly by SalesTerritoryCountry, ProductName and Month.

The sales_rank CTE returns the columns from the base CTE and a rank column by the sum of SalesAmount on a monthly basis in ascending order i.e., the least salesamount for each month is ranked 1.

Finally, from the sales_rank CTE, the final output is queried as required, a combination of Country and Product with the least amount of sales for each month of 2012 by specifying in the where clause “rn = 1”.
********************************************************************
