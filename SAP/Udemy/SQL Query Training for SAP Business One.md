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

[Activity: Getting Your Tools Together](./Activity%20Getting%20Your%20Tools%20Together.md)

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
WHERE TO.[Docstatus] = 'O' -- Open
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
