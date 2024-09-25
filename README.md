# CTE Applications
## What is a CTE?  
![CTE Syntax](https://learnsql.com/blog/sql-common-table-expression-guide/cte_syntax_example.jpg)  
  
CTEs are temporary result sets that are defined before a SELECT statement and executed alongside it. They provide an efficient way to structure complex queries, improving code readability and maintainability. The temporary result sets created by CTEs can be referenced multiple times within the same query, making them especially useful when dealing with complex data. Unlike subqueries, which can only be used once in a query, CTEs offer the advantage of repeatability and clarity, making them easier to understand and work with.

## Inventory Replenishment Analysis CTE  
CTEs can range from simple to highly complex. For example, consider a scenario where we need to extract product inventory details to determine which products need to be reordered. A simple, single CTE defined as LowInventoryProducts can be used to aggregate all products with low inventory levels. The main SELECT statement would then filter out products whose stock levels are equal to or below their reorder point, helping businesses quickly identify items that require replenishment.  
```
--SQL
with LowInventoryProducts AS (
    SELECT 
        P.[EnglishProductName],
        SUM(PI.[UnitsBalance]) AS TotalUnitsInStock,
        P.[SafetyStockLevel],
        P.[ReorderPoint]
    FROM [AdventureWorksDW2022].[dbo].[FactProductInventory] AS PI
    JOIN [AdventureWorksDW2022].[dbo].[DimProduct] AS P
        ON PI.[ProductKey] = P.[ProductKey]
    GROUP BY 
        P.[EnglishProductName],
        P.[SafetyStockLevel],
        P.[ReorderPoint]
)
SELECT 
    EnglishProductName,
    TotalUnitsInStock,
    SafetyStockLevel,
    ReorderPoint
FROM LowInventoryProducts
WHERE TotalUnitsInStock <= ReorderPoint

```	
As you can see, a simple CTE can efficiently extract product inventory details in a modular and readable way. While this basic usage demonstrates how CTEs work and highlights their benefits, there are more complex scenarios where CTEs become essential.		

## Top 5 Selling Products by Category CTE

```
--SQL
with  
InternetSales AS
(
	SELECT 
		I.[ProductKey],
		I.[SalesAmount],
		P.[EnglishProductName],
		P.[Color],
		P.[Size],
		P.[SizeRange],
		P.[Weight],
		P.[ProductLine],
		P.[Class],
		P.[Style],
		P.[ModelName],
		PS.[EnglishProductSubcategoryName],
		PS.[ProductCategoryKey]
	FROM [AdventureWorksDW2022].[dbo].[FactInternetSales] AS I
	LEFT JOIN [AdventureWorksDW2022].[dbo].[DimProduct] AS P
	ON I.ProductKey = P.ProductKey
	LEFT JOIN [AdventureWorksDW2022].[dbo].[DimProductSubcategory] as PS
	ON P.ProductSubcategoryKey = PS.ProductSubcategoryKey
),
Ranking AS
(
SELECT 
	ROUND(SUM([SalesAmount]),2) AS TotalSales,
	RANK () OVER (PARTITION BY [ProductCategoryKey] ORDER BY SUM([SalesAmount]) DESC) AS SalesRanked,
	[ProductCategoryKey],
	[EnglishProductSubcategoryName],
	[ProductKey]
FROM InternetSales
GROUP BY [ProductCategoryKey], [EnglishProductSubcategoryName], [Color], [ProductKey]
)
SELECT DISTINCT
	R.[SalesRanked],
	R.[ProductCategoryKey],
	R.[EnglishProductSubcategoryName],
	I.[SalesAmount],
	R.[TotalSales],
	R.[ProductKey],
	I.[EnglishProductName],
	I.[Color],
	I.[Size],
	I.[SizeRange],
	I.[Weight],
	I.[ProductLine],
	I.[Class],
	I.[Style],
	I.[ModelName]
FROM InternetSales AS I
INNER JOIN Ranking AS R
ON I.[ProductKey] = R.[ProductKey]
WHERE SalesRanked <=5
ORDER BY ProductCategoryKey,SalesRanked
```
