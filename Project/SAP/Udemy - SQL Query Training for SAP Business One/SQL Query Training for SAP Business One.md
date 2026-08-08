## **SQL Query Training for SAP Business One**

https://www.udemy.com/course/sql-query-training-for-sap-business-one/learn/lecture/8332978#overview

## Finding Data in SAP B1

Master Data / Right Bottom has Data Info

Tools / Queires / Query Generator

Tabs - Tables

Bolds - Related Data

Use notepad2 to formatting

SQL Server Management Studio -- TO DO

Google Search ex. bin stock query sap bussiness one

SAP Business One - Database Tables - References - He only use it for data type reference - Use more query generator

## Process & Tools

Process

1. Spreadsheet - understading requirement before writing query. like plan, columns etc.
2. Find the table in query generator
3. Edit queries in notepad2 (Not Recomend to use Query Preview) ex. Formating
4. SQL Management Studio (benefit: auto complete, intellisense, color coding, etc)
5. Final testing in SAP B1 Query Generator

```SQL
SELECT * FROM OCRD
```

[Activity: Getting Your Tools Together](Activity%20Getting%20Your%20Tools%20Together.md)

[Article: Advanced Tips & Tricks](./Article%20Advanced%20Tips%20%26%20Tricks.md)

## SQL Query Basics

Basic SQL Syntax

```SQL
SELECT
  T0.[CardCode],
  T0.[CardName],
FROM
  OCRD T0 -- Business Partner Master Data
WHERE
  T0.[CardType] = 'C'
ORDER BY
  T0.[CardCode]

-- Column Aliases

SELECT
  -- T0.[CardCode] 'Customer Code',
  -- T0.[CardCode] AS 'Customer Code',
  [Customer Code] = T0.[CardCode],
  [BP Name]       = T0.[CardName],
FROM
  OCRD T0
WHERE
  T0.[CardType] = 'C'
ORDER BY
  T0.[CardCode]

-- Comments

/*
  Multiple line comments
*/

-- Single line comments
```

SQL Table Joins

- Query Generator
- Left click and hold on the column name and drag it to the other table to create a join

```SQL
/*
TABLE JOINS # 1
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]
FROM
	OINV T0 -- Invoices
ORDER BY T0.[DocEntry]

/*
TABLE JOINS # 2
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]
	,T1.[DocEntry]
	,T1.[LineNum]
	,T1.[ItemCode]
	,T1.[Quantity]
	,T1.[Price]
FROM
	OINV T0 -- Invoices
	INNER JOIN INV1 T1 ON T0.[DocEntry] = T1.[DocEntry] -- Invoice rows
ORDER BY T0.[DocEntry]

/*
TABLE JOINS # 3
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]
	,T1.[DocEntry]
	,T1.[LineNum]
	,T1.[ItemCode]
	,T1.[Dscription]
	,T1.[Quantity]
	,T1.[Price]
	,T2.[ItemCode]
	,T2.[ItemName]
	,T2.[AvgPrice]
FROM
	OINV T0 -- Invoices
	INNER JOIN INV1 T1 ON T0.[DocEntry] = T1.[DocEntry] -- Invoice rows
	LEFT OUTER JOIN OITM T2 ON T1.[ItemCode] = T2.[ItemCode]	-- Item master data
ORDER BY T0.[DocEntry]
```

SQL Query Filtering

```SQL
/*
BASIC QUERY FILTERING # 1

AND, OR, NOT, IN, LIKE
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]

FROM
	OINV T0 -- Invoices

WHERE
	(T0.[DocStatus] = 'O' AND T0.[CANCELED] = 'N') -- Valuate this first
	OR
	T0.[CardCode] IN ('C000100','C000101')
	AND
	T0.[CardCode] NOT IN ('C000100','C000101')
	AND
	T0.[U_Expedition_Comment] IS NOT NULL
	AND
	T0.[CardName] LIKE '%or%'
	AND
	NOT T0.[DocNum] = 81321

ORDER BY T0.[DocEntry]


/*
BASIC QUERY FILTERING # 2

=, <>, >, >=, <=, <
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]

FROM
	OINV T0 -- Invoices

WHERE
	T0.[DocTotal] = 0 -- Equal to
	AND
	T0.[DocTotal] <> 0 -- Not equal to
	AND
	T0.[DocTotal] > 0 -- Greater than
	AND
	T0.[DocTotal] >= 0 -- Greater than and equal to
	AND
	T0.[DocTotal] <= 0 -- Less than and equal to
	AND
	T0.[DocTotal] < 0 -- Less than

ORDER BY T0.[DocEntry]
```

SQL Query Filtering with Dates

```SQL
/*
QUERY DATE FILTERING
*/

SELECT
	 T0.[DocEntry]
	,T0.[DocNum]
	,T0.[DocType]
	,T0.[DocDate]
	,T0.[DocTotal]

FROM
	OINV T0 -- Invoices

WHERE
	T0.[DocDate] = GETDATE() -- Today
  AND
  T0.[DocDate] = '10/27/1983' -- Select a specific date
  AND
  T0.[DocDate] BETWEEN '01/01/2018' AND '12/31/2018' -- Between two dates
  AND
  T0.[DocDate] >= DATEADD(yyyy,-1,GETDATE()) -- Newer than one year ago
  AND
  DATEDIFF(d,T0.[DocDate],GETDATE()) > 30 -- Older than 30 days
  AND
  DATEPART(yyyy,T0.[DocDate]) = '2018' -- Any documents in the year 2018
  AND
  DATEPART(yyyy,T0.[DocDate]) = DATEPART(yyyy,GETDATE()) -- Year is the same as the current year

ORDER BY T0.[DocEntry]
```

SAP Business One Parameters

```SQL
/*
SAP PARAMETERS # 1
*/

SELECT TOP 5
   T0.[CardCode]
  ,T0.[CardName]
  ,T0.[CardType]
FROM
  OCRD T0
WHERE
  T0.[CardType] = '[%0]' -- only available in SAP query executor
ORDER BY T0.[CardCode]

/*
SAP PARAMETERS # 2
*/

SELECT
   T0.[DocNum]
  ,T0.[DocDate]
  ,T1.[ItemCode]
  ,T1.[Dscription]
  ,T1.[Quantity]
  ,T1.[Price]
  ,T1.[WhsCode]
  ,T2.[WhsName]
FROM
  OINV T0
  INNER JOIN INV1 T1 ON T0.[DocEntry] = T1.[DocEntry]
  INNER JOIN OWHS T2 ON T1.[WhsCode] = T2.[WhsCode]
WHERE
  T2.[WhsCode] = '[%0]'

/*
SAP PARAMETERS # 3
*/

Declare @Date1 Datetime
Declare @Date2 Datetime
Set @Date1 = (select min(S0.DocDate) from OINM S0 Where S0.Docdate >= '[%0]')
Set @Date2 = (Select max(S1.Docdate) from OINM S1 Where S1.DocDate <= '[%1]')
SELECT
   T0.[DocNum]
  ,T0.[DocDate]
  ,T1.[ItemCode]
  ,T1.[Dscription]
  ,T1.[Quantity]
  ,T1.[Price]
  ,T1.[WhsCode]
  ,T2.[WhsName]
FROM
  OINV T0
  INNER JOIN INV1 T1 ON T0.[DocEntry] = T1.[DocEntry]
  INNER JOIN OWHS T2 ON T1.[WhsCode] = T2.[WhsCode]
WHERE
  T0.[DocDate] BETWEEN @Date1 AND @Date2
  AND
  T2.[WhsCode] = '[%3]'
```

## Deploy Your Queries to SAP

Format in notepad2

Debug in query preview

Save as

- Query Name
- Category (Authorization group)

Modules -> Admin -> System Initialization -> Authoizations -> General Authorizations: authorization per query category

## Exporting Your Data to Excel

Query Manager -> Query Name -> Query -> right click; copy table -> paste to excel

Click excel button -> Save as Excel

## Alert Queries

```SQL
SELECT
	 TOP 10
	 T0.[DocNum] 'Doc. No.'
	,T0.[CardCode] 'Vendor Code'
	,T1.[CardName] 'Vendor Name'
	,T0.[DocDate]
	,T0.[DocDueDate] 'Delivery Date'

FROM
	OPOR T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]

WHERE
	T0.[DocStatus] = 'O' -- open
	AND
	DATEDIFF(d,T0.[DocDueDate],GETDATE()) >= 0 -- older than today

ORDER BY T0.[CardName]
```

## Formatted Searches

????

TO DO - Watch again later

```SQL
/*
FORMATTED SEARCHES
*/

-- HEADER SELECT

SELECT $[$4.0.0]

SELECT T0.[Phone1] FROM OCRD T0 WHERE T0.[CardCode] = $[$4.0.0]


-- HEADER UDF SELECT (No addiitonal $)

SELECT $[ORDR.U_ShippingNotes.0]


-- ROW SELECT

SELECT $[$38.1.0]

SELECT T0.[Price] FROM ITM1 T0 WHERE T0.[ItemCode] = $[$38.1.0] AND T0.[PriceList] = 4


-- SELECT LIST

SELECT TOP 10 T0.[CardCode], T0.CardName FROM OCRD T0 WHERE LEFT(T0.[CardName],8) = 'Customer' AND T0.[CardType] = 'C' ORDER BY T0.[CardName]
```

(???) Dynamic field value referencing other fields.

## CASE Satements

```SQL
/*
CASE STATEMENTS
*/

-- SELECT DISTINCT T0.[CardType] FROM OCRD T0
-- C
-- L

SELECT
	T0.[CardCode],
	CASE
	  WHEN T0.[CardType] = 'C' THEN 'Customer' 	-- WHEN 'C' THEN 'Customer'
	  WHEN T0.[CardType] = 'S' THEN 'Supplier'  -- WHEN 'S' THEN 'Supplier'
	  WHEN T0.[CardType] = 'L' THEN 'Lead' 			-- WHEN 'L' THEN 'Lead'
	ELSE 'Vendor' END 'CaseType',
	T0.[Phone1],
	T0.[Balance]

FROM
	OCRD T0 -- Business Partner Master Data

ORDER BY T0.[CardCode]
```

## Aggregate Functions and Grouping

```SQL
/*
AGGREGATE STATEMENTS #1
*/

SELECT
	T0.[CardType],
	SUM(T0.[Balance]) AS 'Balance', -- T0.[Balance]
FROM
	OCRD T0 -- Business Partner Master Data
GROUP BY T0.[CardType]

/*
AGGREGATE STATEMENTS #2
*/

SELECT
	T0.[DocDate],
	-- MAX(T0.[DocTotal])
	COUNT(T0.[DocTotal]) 'No. Sales Orders / Day'
FROM
	ORDR T0 -- Sales Orders
WHERE T0.[Docstatus] = 'O' -- Open
GROUP BY T0.[DocDate]

/*
AGGREGATE STATEMENTS #3
*/

SELECT
	T0.[CardCode],
	MAX(T0.[DocDate]) AS 'Last Order Date',
FROM
	ORDR T0 -- Sales Orders
GROUP BY T0.[CardCode]
ORDER BY T0.[CardCode]
```

1. `SUM()` - Adds up the column in question. Sometimes has issues with NULL values. Make sure you use COALESCE() if the result could be NULL. This would be the most commonly used aggregate function in my experience.
2. `COUNT()` - Counts the number of results from the query. Really good for volume of orders per day, counting records, etc.
3. `MAX()` - Returns the maximum from the dataset. I would say most commonly used for dates. Like the highest date in the summary. But you could also use TOP 1 in a subquery for this as well.
4. `MIN()` - Similar to MAX(), this returns the lowest date or value.
5. `AVG()` - Returns the average. I don't use this much but it's there.

## Coalesce Will Save the Day

```SQL

/*
COALESCE() #1
*/

SELECT NULL * 10 -- return null
SELECT COALESCE(NULL, 1) * 10 -- return 10

SELECT NULL + 10 -- return null
SELECT COALESCE(NULL, 5) + 10 -- return 15

/*
COALESCE() #2
*/

SELECT
	T1.[DocType],
	COALESCE(T0.[ItemCode], 'Service Type') '<BLANK>' -- callback if null
FROM
	INV1 T0
	INNER JOIN OINV T1 ON T0.[DocEntry] = T1.[DocEntry]
ORDER BY T0.[DocDate] DESC

/*
COALESCE() #3
*/

SELECT
	T0.[CardCode],
-- 	SUM(T1.[DocTotal])
  COALESCE(SUM(T1.[DocTotal]),0)

FROM
	OCRD T0
	LEFT OUTER JOIN OINV T1 ON T0.[CardCode] = T1.[CardCode]

GROUP BY T0.[CardCode]
```

## Subqueries and Derived Tables

```SQL
/*
SUBQUERIES AND DERIVED TABLES #1
*/

SELECT
	T0.[CardCode],
	[Total Orders]			= (SELECT SUM(S0.[DocTotal]) FROM ORDR S0 WHERE S0.[CardCode] = T0.[CardCode]),
	[Last Order]			= (SELECT MAX(S0.[DocDate]) FROM ORDR S0 WHERE S0.[CardCode] = T0.[CardCode]),
	[Total Invoices]		= (SELECT SUM(S0.[DocTotal]) FROM OINV S0 WHERE S0.[CardCode] = T0.[CardCode]),
	[Last Invoice]			= (SELECT TOP 1 S0.[DocDate] FROM OINV S0 WHERE S0.[CardCode] = T0.[CardCode] ORDER BY S0.[DocDate] DESC)
FROM
	OCRD T0
WHERE
	T0.[CardType] = 'C'

/*
SUBQUERIES AND DERIVED TABLES #2
*/

SELECT
	T0.[CardCode],
	SUM(T1.[DocTotal])

FROM
	OCRD T0
	LEFT OUTER JOIN (
		SELECT D0.[CardCode],D0.[DocTotal] FROM OINV D0
		UNION ALL
		SELECT D0.[CardCode],-D0.[DocTotal] FROM ORIN D0
	) T1 ON T0.[CardCode] = T1.[CardCode]
WHERE
	T0.[CardType] = 'C'
GROUP BY T0.[CardCode]
```


## SQL Views

```SQL
/* 
SQL VIEWS uv_BSC_SalesAnalysis #1
*/

SELECT 
	 'Invoice' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
	,T0.[TotalExpns] 'Freight (LC)'
	,T0.[VatSum] AS 'Tax (LC)'
	,T0.[DocTotal] AS 'Total (LC)'
	,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
	,T0.[TotalExpFC] 'Freight (FC)'
	,T0.[VatSumFC] AS 'Tax (FC)'
	,T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  OINV T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  
UNION ALL

SELECT 
	 'Credit' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
	,-T0.[TotalExpns] 'Freight (LC)'
	,-T0.[VatSum] AS 'Tax (LC)'
	,-T0.[DocTotal] AS 'Total (LC)'
	,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
	,-T0.[TotalExpFC] 'Freight (FC)'
	,-T0.[VatSumFC] AS 'Tax (FC)'
	,-T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  ORIN T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
```

```SQL
/* 
SQL VIEWS uv_BSC_SalesAnalysis #2
*/

Declare @Date1 Datetime
Declare @Date2 Datetime
 
Set @Date1 = (select min(S0.DocDate) from OINM S0 Where S0.Docdate >= '[%0]')
Set @Date2 = (Select max(S1.Docdate) from OINM S1 Where S1.DocDate <= '[%1]')

SELECT 
   T0.[Type]
  ,T0.[Doc. Date]
  ,T0.[DocNum]
  ,T0.[CardCode]
  ,T0.[Doc.Currency]
  ,T0.[Subtotal (LC)]
  ,T0.[Freight (LC)]
  ,T0.[Tax (LC)]
  ,T0.[Total (LC)]
  ,T0.[Subtotal (FC)]
  ,T0.[Freight (FC)]
  ,T0.[Tax (FC)]
  ,T0.[Total (FC)]
  ,T0.[Ship-To Overwritten]
  ,T0.[Address Line 1]
  ,T0.[Address Line 2]
  ,T0.[City]
  ,T0.[State]
  ,T0.[Country]
  ,T0.[Postal]

FROM 
  [uv_BSC_SalesAnalysis] T0
  
WHERE
  T0.[Doc. Date] BETWEEN @Date1 AND @Date2
  
ORDER BY T0.[Doc. Date],T0.[CardCode],T0.[DocNum]
```

```SQL
/* 
SQL VIEWS uv_BSC_SalesAnalysis #3
*/

USE [SAPBusinessObjects]
GO

/****** Object:  View [dbo].[uv_BSC_SalesAnalysis]    Script Date: 4/7/2018 5:13:47 PM ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE VIEW [dbo].[uv_BSC_SalesAnalysis]
AS
SELECT 
	 'Invoice' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
	,T0.[TotalExpns] 'Freight (LC)'
	,T0.[VatSum] AS 'Tax (LC)'
	,T0.[DocTotal] AS 'Total (LC)'
	,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
	,T0.[TotalExpFC] 'Freight (FC)'
	,T0.[VatSumFC] AS 'Tax (FC)'
	,T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  OINV T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  
UNION ALL

SELECT 
	 'Credit' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
	,-T0.[TotalExpns] 'Freight (LC)'
	,-T0.[VatSum] AS 'Tax (LC)'
	,-T0.[DocTotal] AS 'Total (LC)'
	,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
	,-T0.[TotalExpFC] 'Freight (FC)'
	,-T0.[VatSumFC] AS 'Tax (FC)'
	,-T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  ORIN T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'

GO

EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPane1', @value=N'[0E232FF0-B466-11cf-A24F-00AA00A3EFFF, 1.00]
Begin DesignProperties = 
   Begin PaneConfigurations = 
      Begin PaneConfiguration = 0
         NumPanes = 4
         Configuration = "(H (1[24] 4[19] 2[30] 3) )"
      End
      Begin PaneConfiguration = 1
         NumPanes = 3
         Configuration = "(H (1 [50] 4 [25] 3))"
      End
      Begin PaneConfiguration = 2
         NumPanes = 3
         Configuration = "(H (1 [50] 2 [25] 3))"
      End
      Begin PaneConfiguration = 3
         NumPanes = 3
         Configuration = "(H (4 [30] 2 [40] 3))"
      End
      Begin PaneConfiguration = 4
         NumPanes = 2
         Configuration = "(H (1 [56] 3))"
      End
      Begin PaneConfiguration = 5
         NumPanes = 2
         Configuration = "(H (2 [66] 3))"
      End
      Begin PaneConfiguration = 6
         NumPanes = 2
         Configuration = "(H (4 [50] 3))"
      End
      Begin PaneConfiguration = 7
         NumPanes = 1
         Configuration = "(V (3))"
      End
      Begin PaneConfiguration = 8
         NumPanes = 3
         Configuration = "(H (1[56] 4[18] 2) )"
      End
      Begin PaneConfiguration = 9
         NumPanes = 2
         Configuration = "(H (1 [75] 4))"
      End
      Begin PaneConfiguration = 10
         NumPanes = 2
         Configuration = "(H (1[66] 2) )"
      End
      Begin PaneConfiguration = 11
         NumPanes = 2
         Configuration = "(H (4 [60] 2))"
      End
      Begin PaneConfiguration = 12
         NumPanes = 1
         Configuration = "(H (1) )"
      End
      Begin PaneConfiguration = 13
         NumPanes = 1
         Configuration = "(V (4))"
      End
      Begin PaneConfiguration = 14
         NumPanes = 1
         Configuration = "(V (2))"
      End
      ActivePaneConfig = 0
   End
   Begin DiagramPane = 
      Begin Origin = 
         Top = 0
         Left = 0
      End
      Begin Tables = 
      End
   End
   Begin SQLPane = 
   End
   Begin DataPane = 
      Begin ParameterDefaults = ""
      End
      Begin ColumnWidths = 9
         Width = 284
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
         Width = 1500
      End
   End
   Begin CriteriaPane = 
      Begin ColumnWidths = 11
         Column = 1440
         Alias = 900
         Table = 1170
         Output = 720
         Append = 1400
         NewValue = 1170
         SortType = 1350
         SortOrder = 1410
         GroupBy = 1350
         Filter = 1350
         Or = 1350
         Or = 1350
         Or = 1350
      End
   End
End
' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'uv_BSC_SalesAnalysis'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_DiagramPaneCount', @value=1 , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'VIEW',@level1name=N'uv_BSC_SalesAnalysis'
GO
```

## Declaring SQL Variables

```SQL
/* 
VARIABLES
*/

-- Set Customer
DECLARE @CardCode VARCHAR(15) = 'C001000'

-- Set Date Range
DECLARE @Date1 DATETIME
DECLARE @Date2 DATETIME 

SET @Date1 = '01/01/2018'
SET @Date2 = '12/31/2018'

SELECT
	 T0.[CardCode]
	,[Total Orders]			= COALESCE((SELECT SUM(S0.[DocTotal]) FROM ORDR S0 WHERE S0.[CardCode] = T0.[CardCode] AND S0.[DocDate] BETWEEN @Date1 AND @Date2),0)
	,[Last Order]			= (SELECT MAX(S0.[DocDate]) FROM ORDR S0 WHERE S0.[CardCode] = T0.[CardCode])
	,[Total Invoices]		= COALESCE((SELECT SUM(S0.[DocTotal]) FROM OINV S0 WHERE S0.[CardCode] = T0.[CardCode] AND S0.[DocDate] BETWEEN @Date1 AND @Date2),0)
	,[Last Invoice]			= (SELECT MAX(S0.[DocDate]) FROM OINV S0 WHERE S0.[CardCode] = T0.[CardCode])

FROM
	OCRD T0

WHERE
	T0.[CardType] = 'C'
	AND
	T0.[CardCode] = @CardCode
```

##  Stored Procedures

```SQL
-- ================================================
-- Template generated from Template Explorer using:
-- Create Procedure (New Menu).SQL
--
-- Use the Specify Values for Template Parameters 
-- command (Ctrl-Shift-M) to fill in the parameter 
-- values below.
--
-- This block of comments will not be included in
-- the definition of the procedure.
-- ================================================
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		MIKE TAYLOR
-- Create date: SOMETIME 2018
-- Description:	Runs data from a date range
-- =============================================
CREATE PROCEDURE uSP_Udemy_Course_Example 
	-- Add the parameters for the stored procedure here
	@Date1 DATETIME, @Date2 DATETIME
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	/* 
SQL VIEWS uv_BSC_SalesAnalysis
*/

SELECT 
	 'Invoice' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
	,T0.[TotalExpns] 'Freight (LC)'
	,T0.[VatSum] AS 'Tax (LC)'
	,T0.[DocTotal] AS 'Total (LC)'
	,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
	,T0.[TotalExpFC] 'Freight (FC)'
	,T0.[VatSumFC] AS 'Tax (FC)'
	,T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  OINV T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  AND
  T0.[DocDate] BETWEEN @Date1 AND @Date2
  
UNION ALL

SELECT 
	 'Credit' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
	,-T0.[TotalExpns] 'Freight (LC)'
	,-T0.[VatSum] AS 'Tax (LC)'
	,-T0.[DocTotal] AS 'Total (LC)'
	,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
	,-T0.[TotalExpFC] 'Freight (FC)'
	,-T0.[VatSumFC] AS 'Tax (FC)'
	,-T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  ORIN T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  AND
  T0.[DocDate] BETWEEN @Date1 AND @Date2  

END
GO

```

```SQL
EXEC [dbo].[uSP_Udemy_Course_Example] '01/01/2018','12/31/2018'
```


## Views and Stored Procedures for Crystal Reports

```SQL
/* 
SQL VIEWS uv_BSC_SalesAnalysis #1
*/

SELECT 
	 'Invoice' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
	,T0.[TotalExpns] 'Freight (LC)'
	,T0.[VatSum] AS 'Tax (LC)'
	,T0.[DocTotal] AS 'Total (LC)'
	,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
	,T0.[TotalExpFC] 'Freight (FC)'
	,T0.[VatSumFC] AS 'Tax (FC)'
	,T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  OINV T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  
UNION ALL

SELECT 
	 'Credit' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
	,-T0.[TotalExpns] 'Freight (LC)'
	,-T0.[VatSum] AS 'Tax (LC)'
	,-T0.[DocTotal] AS 'Total (LC)'
	,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
	,-T0.[TotalExpFC] 'Freight (FC)'
	,-T0.[VatSumFC] AS 'Tax (FC)'
	,-T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  ORIN T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
```

```SQL
-- ================================================
-- Template generated from Template Explorer using:
-- Create Procedure (New Menu).SQL
--
-- Use the Specify Values for Template Parameters 
-- command (Ctrl-Shift-M) to fill in the parameter 
-- values below.
--
-- This block of comments will not be included in
-- the definition of the procedure.
-- ================================================
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		MIKE TAYLOR
-- Create date: SOMETIME 2018
-- Description:	Runs data from a date range
-- =============================================
CREATE PROCEDURE uSP_Udemy_Course_Example 
	-- Add the parameters for the stored procedure here
	@Date1 DATETIME, @Date2 DATETIME
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	/* 
SQL VIEWS uv_BSC_SalesAnalysis
*/

SELECT 
	 'Invoice' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
	,T0.[TotalExpns] 'Freight (LC)'
	,T0.[VatSum] AS 'Tax (LC)'
	,T0.[DocTotal] AS 'Total (LC)'
	,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
	,T0.[TotalExpFC] 'Freight (FC)'
	,T0.[VatSumFC] AS 'Tax (FC)'
	,T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  OINV T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  AND
  T0.[DocDate] BETWEEN @Date1 AND @Date2
  
UNION ALL

SELECT 
	 'Credit' AS Type
	,T0.[DocDate] AS 'Doc. Date'
	,T0.[DocNum]
	,T0.[CardCode]
	,T0.[DocCur] 'Doc.Currency'
	,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
	,-T0.[TotalExpns] 'Freight (LC)'
	,-T0.[VatSum] AS 'Tax (LC)'
	,-T0.[DocTotal] AS 'Total (LC)'
	,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
	,-T0.[TotalExpFC] 'Freight (FC)'
	,-T0.[VatSumFC] AS 'Tax (FC)'
	,-T0.[DocTotalFC] AS 'Total (FC)'
	,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
	,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
	,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
	,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
	,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
	,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
	,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'
  
FROM 
  ORIN T0
  INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
  LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]
  
WHERE
  T0.[CANCELED] = 'N'
  AND
  T0.[DocDate] BETWEEN @Date1 AND @Date2  

END
GO

```
## Business Partner Master Data Raw Export v1.0

```SQL
1. -- BSC: Business Partner Master Data Raw Export v1.0
2. -- To pull Customer and Lead Master Data

3. SELECT
4.     CASE T0.[CardType] WHEN 'L' THEN 'Lead' WHEN 'C' THEN 'Customer' WHEN 'S' THEN 'Supplier' END 'BP Type'
5.     ,T0.[frozenFor] 'Inactive'
6.     ,T0.[QryGroup10] 'Store is Closed'
7.     ,T0.[CardCode]
8.     ,T0.[CardName]
9.     ,T0.[CntctPrsn] 'Main Contact'
10.     ,T1.[GroupName]
11.     ,T0.[Phone1] 'Telephone 1'
12.     ,T0.[Phone2] 'Telephone 2'
13.     ,CASE 
14.       WHEN T0.[E_Mail] IS NOT NULL THEN T0.[E_Mail] 
15.       WHEN T0.[E_Mail] IS NULL THEN (SELECT TOP 1 S1.[E_MailL] FROM OCPR S1 WHERE T0.[CntctPrsn] = S1.[Name] AND T0.[CardCode] = S1.[CardCode] AND S1.[E_MailL] IS NOT NULL AND S1.[E_MailL] <> '')
16.       ELSE NULL END 'E-mail'
17.     ,T0.[Notes] 'Quick Remarks'
18.     ,T0.[Currency]
19.     ,T0.[U_Discount]
20.     ,T2.[PymntGroup]
21.     ,T6.[Descript] 'Payment Method'
22.     ,T3.[ListName]
23.     ,T4.[SlpName]
24.     ,T0.[CreditLine] 'Credit Limit'       
25.     ,T7.[TrnspName] 'Preferred Transportation'
26.     ,CASE 
27.       WHEN T0.[FatherType] = 'D' THEN 'Delivery'
28.       WHEN T0.[FatherType] = 'P' THEN 'Payment'
29.     ELSE NULL END 'Consolidation Type'

30.     ,T0.[FatherCard] 'Consolidation Partner Code'
31.     ,T5.[CardName] 'Consolidation Partner Name'

32.     ,T0.[ShipToDef] 'Ship-To Address Name'
33.     ,T0.[MailAddres] 'Ship-To Address Line 1'
34.     ,T0.[MailStrNo] 'Ship-To Address Line 2'
35.     ,T0.[MailCity]
36.     ,T0.[State2]
37.     ,T0.[MailZipCod]
38.     ,T0.[MailCountr]
39.     ,(SELECT TOP 1 S0.[TaxCode] FROM CRD1 S0 WHERE S0.[AdresType] = 'S' AND S0.[CardCode] = T0.[CardCode] AND S0.[Address] = T0.[ShipToDef]) 'Default Tax Code'

40.     ,T0.[BillToDef] 'Bill-To Address Name'
41.     ,T0.[Address] 'Bill-To Address Line 1'
42.     ,T0.[StreetNo] 'Bill-To Address Line 2'
43.     ,T0.[City]
44.     ,T0.[State1]
45.     ,T0.[ZipCode]
46.     ,T0.[Country]

47.     ,COALESCE((
48.       SELECT Sum(IsNull(S1.[DocTotal] + S1.[DiscSum] + S1.[DpmAmnt] - S1.[TotalExpns] - S1.[VatSum],0))
49.       FROM OCRD S0
50.       Left JOIN OINV S1 ON S0.CardCode = S1.CardCode 
51.       WHERE S0.CardType ='C' AND S1.[CANCELED] = 'N'
52.       And S0.CardCode = T0.CardCode AND DATEPART(yyyy,S1.DocDate) = 2018
53.     ),0)

54.     -

55.     COALESCE((
56.       SELECT Sum(IsNull(S1.[DocTotal] + S1.[DiscSum] + S1.[DpmAmnt] - S1.[TotalExpns] - S1.[VatSum],0))
57.       FROM OCRD S0
58.       Left JOIN ORIN S1 ON S0.CardCode = S1.CardCode 
59.       WHERE S0.CardType ='C' AND S1.[CANCELED] = 'N' 
60.       And S0.CardCode = T0.CardCode AND DATEPART(yyyy,S1.DocDate) = 2018
61.     ),0) 'YTD 2018 Sales'
62.     ,COALESCE((SELECT COUNT(S0.DocNum) FROM OINV S0 WHERE S0.CANCELED = 'N' AND DATEPART(yyyy,S0.DocDate) = 2018 AND S0.CardCode = T0.CardCode),0) 'Invoice Count 2018'
63.     ,COALESCE((SELECT COUNT(S0.DocNum) FROM ORIN S0 WHERE S0.CANCELED = 'N' AND DATEPART(yyyy,S0.DocDate) = 2018 AND S0.CardCode = T0.CardCode),0) 'Credit Count 2018'
64.     ,COALESCE((SELECT COUNT(S0.DocNum) FROM OINV S0 WHERE S0.CANCELED = 'N' AND DATEPART(yyyy,S0.DocDate) = 2018 AND S0.CardCode = T0.CardCode),0) - COALESCE((SELECT COUNT(S0.DocNum) FROM ORIN S0 WHERE S0.CANCELED = 'N' AND DATEPART(yyyy,S0.DocDate) = 2018 AND S0.CardCode = T0.CardCode),0) 'Invoice / Credit Net 2018'

65. FROM 
66.     OCRD T0  
67.     LEFT OUTER JOIN OCRG T1 ON T0.GroupCode = T1.GroupCode 
68.     LEFT OUTER JOIN OCTG T2 ON T0.GroupNum = T2.GroupNum 
69.     LEFT OUTER JOIN OPLN T3 ON T0.ListNum = T3.ListNum 
70.     LEFT OUTER JOIN OSLP T4 ON T0.SlpCode = T4.SlpCode 
71.     LEFT OUTER JOIN OCRD T5 ON T0.FatherCard = T5.CardCode 
72.     LEFT OUTER JOIN OPYM T6 ON T0.PymCode = T6.PayMethCod
73.     LEFT OUTER JOIN OSHP T7 ON T0.ShipType = T7.TrnspCode
74.     LEFT OUTER JOIN YTD_VIEW_2 T8 ON T0.CardCode = T8.CardCode

75. WHERE T0.[CardType] IN ('C','L')

76. ORDER BY T0.[CardName]
```

## Item Master Data + Price Lists Raw Export v1.0

```SQL
1. -- BSC: Item Master Data + Price Lists Raw Export v1.0
2. -- To dump some item master data into a spreadsheet or for comparing in a query

3. SELECT 
4.   CASE T0.[frozenFor] WHEN 'Y' THEN 'Inactive' ELSE '' END 'Active'
5.   ,T0.[ItemCode] 'Item No.'
6.   ,T0.[ItemName] 'Item Name'
7.   ,T1.[ItmsGrpNam] 'Collection'
8.   ,T2.[FirmName] 'Manufacturer'
9.   ,T4.[CstGrpName] 'Customs Group'
10.   ,T4.[TotalTax] 'Customs Rate'
11.   ,T0.[SuppCatNum] 'Mfr. Cat. No.'
12.   ,T0.[BVolume] 'P. Volume'
13.   ,T0.[BWeight1] 'P. Weight'
14.   ,T0.[SVolume] 'S. Volume'
15.   ,T0.[SWeight1] 'S. Weight'
16.   ,T0.[LeadTime] 'Lead Time'
17.   ,[Price List 1 Currency]    = (SELECT S0.[Currency] FROM ITM1 S0 WHERE S0.[ItemCode] = T0.[ItemCode] AND S0.[PriceList] = 1)
18.   ,[Price List 1 Price]        = (SELECT S0.[Price] FROM ITM1 S0 WHERE S0.[ItemCode] = T0.[ItemCode] AND S0.[PriceList] = 1)
19.   ,[Price List 2 Currency]    = (SELECT S0.[Currency] FROM ITM1 S0 WHERE S0.[ItemCode] = T0.[ItemCode] AND S0.[PriceList] = 2)
20.   ,[Price List 2 Price]        = (SELECT S0.[Price] FROM ITM1 S0 WHERE S0.[ItemCode] = T0.[ItemCode] AND S0.[PriceList] = 2)

21.   -- SELECT * FROM OPLN -- Use this query to find your price list number for each of the subqueries

22. FROM 
23.   OITM T0  
24.   INNER JOIN OITB T1 ON T0.ItmsGrpCod = T1.ItmsGrpCod 
25.   LEFT OUTER JOIN OMRC T2 ON T0.FirmCode = T2.FirmCode 
26.   LEFT OUTER JOIN OARG T4 ON T0.CstGrpCode = T4.CstGrpCode
```

## Sales Analysis by Date Range Export v1.0

```SQL
1. -- BSC: Sales Analysis by Date Range Export v1.0
2. -- To pull sales data for a period of time, includes invoices and credit memos

3. Declare @Date1 Datetime
4. Declare @Date2 Datetime

5. Set @Date1 = (select min(S0.DocDate) from OINM S0 Where S0.Docdate >= '[%0]')
6. Set @Date2 = (Select max(S1.Docdate) from OINM S1 Where S1.DocDate <= '[%1]')

7. SELECT 
8.   'Invoice' AS Type
9.     ,T0.[DocDate] AS 'Doc. Date'
10.     ,T0.[DocNum]
11.   ,T0.[CardCode]
12.     ,T0.[CardName]
13.     ,T0.[DocCur] 'Doc.Currency'
14. ,NULL
15.   ,T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum] AS 'Subtotal (LC)'
16.   ,T0.[TotalExpns] 'Freight (LC)'
17.   ,T0.[VatSum] AS 'Tax (LC)'
18.   ,T0.[DocTotal] AS 'Total (LC)'
19. ,NULL
20.   ,T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC] AS 'Subtotal (FC)'
21.   ,T0.[TotalExpFC] 'Freight (FC)'
22.   ,T0.[VatSumFC] AS 'Tax (FC)'
23.   ,T0.[DocTotalFC] AS 'Total (FC)'
24. ,NULL
25.   ,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
26.   ,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
27.   ,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
28.   ,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
29.   ,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
30.   ,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
31.   ,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'

32. FROM 
33.   OINV T0
34.   INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
35.   LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]

36. WHERE
37.   T0.[DocDate] BETWEEN @Date1 AND @Date2
38.   AND
39.   T0.[CANCELED] = 'N'

40. UNION ALL

41. SELECT 
42.   'Credit' AS Type
43.     ,T0.[DocDate] AS 'Doc. Date'
44.     ,T0.[DocNum]
45.   ,T0.[CardCode]
46.     ,T0.[CardName]
47.     ,T0.[DocCur] 'Doc.Currency'
48. ,NULL
49.   ,-(T0.[DocTotal] + T0.[DiscSum] + T0.[DpmAmnt] - T0.[TotalExpns] - T0.[VatSum]) AS 'Subtotal (LC)'
50.   ,-T0.[TotalExpns] 'Freight (LC)'
51.   ,-T0.[VatSum] AS 'Tax (LC)'
52.   ,-T0.[DocTotal] AS 'Total (LC)'
53. ,NULL
54.   ,-(T0.[DocTotalFC] + T0.[DiscSumFC] + T0.[DpmAmntFC] - T0.[TotalExpFC] - T0.[VatSumFC]) AS 'Subtotal (FC)'
55.   ,-T0.[TotalExpFC] 'Freight (FC)'
56.   ,-T0.[VatSumFC] AS 'Tax (FC)'
57.   ,-T0.[DocTotalFC] AS 'Total (FC)'
58. ,NULL
59.   ,CASE WHEN T0.[ShipToOW] = 'Y' THEN 'Yes' ELSE '' END AS 'Ship-To Overwritten'
60.   ,COALESCE(T2.[StreetS],T1.[MailAddres]) AS 'Address Line 1'
61.   ,COALESCE(T2.[StreetNoS],T1.[MailStrNo]) AS 'Address Line 2'
62.   ,COALESCE(T2.[CityS],T1.[MailCity]) AS 'City'
63.   ,COALESCE(T2.[StateS],T1.[State2]) AS 'State'
64.   ,COALESCE(T2.[CountryS],T1.[MailCountr]) AS 'Country'
65.   ,COALESCE(T2.[ZipCodeS],T1.[MailZipCod]) AS 'Postal'

66. FROM 
67.   ORIN T0
68.   INNER JOIN OCRD T1 ON T0.[CardCode] = T1.[CardCode]
69.   LEFT OUTER JOIN INV12 T2 ON T0.[DocEntry] = T2.[DocEntry]

70. WHERE
71.   T0.[DocDate] BETWEEN @Date1 AND @Date2
72.   AND
73.   T0.[CANCELED] = 'N'
```

## Inventory Audit Report Export v1.0

```SQL
1. -- BSC: Inventory Audit Report Export v1.0
2. -- Pulls data from the inventory audit tables for analysis

3. Declare @Date1 Datetime

4. Set @Date1 = (select min(S0.DocDate) from OINM S0 Where S0.Docdate >= '[%0]')

5. SELECT 
6.      T0.[ItemCode]
7.     ,T0.[Dscription]
8.     ,T0.[Warehouse]    
9.     ,[Total Quantity] = SUM(T0.[InQty] - T0.[OutQty])
10.     ,[Total Cost] = SUM(T0.[TransValue])

11. FROM 
12.     OINM T0 WITH (NOLOCK)

13. WHERE
14.     T0.[DocDate] <= @Date1
15.     AND
16.     T0.[Warehouse] = '01'

17. GROUP BY T0.[ItemCode],T0.[Dscription],T0.[Warehouse]
```

## Current Stock in Warehouse v1.0

```SQL
1. -- BSC: Current Stock in Warehouse v1.0
2. -- Pulls current stock available in your warehouses
3. SELECT 
4.      [Item Code] = T1.[ItemCode]
5.     ,[Item Name] = T1.[ItemName]
6.     ,[Item Group] = T2.[ItmsGrpNam]
7.     ,[Warehouse] = T0.[WhsCode]
8.     ,[In Stock] = T0.[OnHand]
9.     ,[Committed] = T0.[IsCommited]
10.     ,[Ordered] = T0.[OnOrder]
11.     ,[Available] = T0.[OnHand] - T0.[IsCommited] + T0.[OnOrder]
12. FROM 
13.     OITW T0  
14.     INNER JOIN OITM T1 ON T0.ItemCode = T1.ItemCode 
15.     INNER JOIN OITB T2 ON T1.ItmsGrpCod = T2.ItmsGrpCod
```