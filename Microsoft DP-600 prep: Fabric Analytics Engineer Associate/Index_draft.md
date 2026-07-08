<!--
 TODOs
 1. Restructure heading
 2. Remove image links
 3. Check transcript for earlier chapters
-->

# 1. Introduction

# 2. Installing and using Power BI Desktop and Service

# 3. Creating calculated columns, including Logical functions

## Creating calculated columns and using basic operators

Easiest way: Table → Date → Icon → New Column

```sql
AmountUSD = FactFinance[Amount] * 1.3
Scenario = "Scenario: " & FactFinance[ScenarioKey] & " " & FactFinance[Date].[Year]
```

## The IF, BLANK and ISBLANK functions

```sql
AmountActual = IF(
  (FactFinance[ScenarioKey] = 'Actual'), // condition
  FactFinance[Amount], // true
  BLANK(), // false
);

AmountAutalPlus1 = IF(
  ISBLANK(FactFinance[AmountActual]), // if it's blank
  BLANK(), // then blank
  FactFinance[AmountActual] + 1, // or + 1
);
```

## The AND, OR and NOT functions

```sql
AmountResult = IF(
  NOT(
    OR(FactFinance[AccountKey] < 60, FactFinance[AccountKey] >= 70),
    // FactFinance[AccountKey] < 60 || FactFinance[AccountKey] >= 70
    // AND: &&
  ),
  FactFinance[Amount],
  BLANK(),
);
```

## The SWITCH function

```sql
Location = IF(FactFinance[OrganizationKey] < 8, 'US', 'Other');

Location = SWITCH(
  FactFinance[OrganizationKey],
  3,
  'NE',
  4,
  'NW',
  5,
  'Central',
  6,
  'SE',
  7,
  'SW',
  'OTHER', // optional, default
);
```

## Other logical functions, together with the DIVIDE function

```sql
IsUnitedStates = IF(FactFinance[OrganizationKey] < 8, TRUE(), FALSE());

AverageAmountPerOrder = IFERROR(FactFinance[Amount] / 0, BLANK());

AverageAmountPerOrder = DIVIDE(FactFinance[Amount], 0, BLANK());
```

## Practice Activity

```sql
LocationName = DimGeography[StateProvinceName] & ", " & DimGeography[EnglishCountryRegionName]

IsUS = IF(DimGeography[CountryRegionCode] = "US", "In US", "Outside of US")

IsTexasOrCalifornia = IF(DimGeography[StateProvinceCode] = "CA" || DimGeography[StateProvinceCode] = "TX", "Yes", "No")

Continent = SWITCH(
    DimGeography[CountryRegionCode],
    "US", "North America",
    "CA", "North America",
    "GB", "Europe",
    "FR", "Europe",
    "DE", "Europe",
    "AU", "Oceania",
    "Unknown"
)

**Continent =
SWITCH( TRUE(),
    DimGeography[CountryRegionCode] IN {"US", "CA"}, "North America",
    DimGeography[CountryRegionCode] IN {"GB", "FR", "DE"}, "Europe",
    DimGeography[CountryRegionCode] = "AU", "Oceania",
    "Unknown"
)**

NewKey = IF(ISBLANK(DimGeography[SalesTerritoryKey]), BLANK(), DimGeography[SalesTerritoryKey] - 1)

Division = DIVIDE(DimGeography[GeographyKey], DimGeography[NewKey], BLANK())
```

# 4. Statistical functions

## Measures - an introduction, with standard aggregations including Count

Mean: 평균

Median: 중간값 (Not skewed by outliners)

AVERAGEA

- power bi: aggregating values only same data type
- function name + a: implicitly converse type then aggregating

```jsx
AmountActualBlank = COUNTBLANK(FactFinance[AmountActual]);

// Explicit aggregation
AmountActutalSum = SUM(FactFinance[AmountActual]);
```

## Aggregation of calculations (iterator functions)

SUM(<column>)

```jsx
AmountActutalSum = SUM(FactFinance[AmountActual]);
```

SUMX(<table>, <expression>)

```jsx
// For debugging; checking staging values
AmountAcualPlus1 = FactFinance[AmountActual] + 1;
AmountAcualPlus1Sum = SUM(FactFinance[AmountAcualPlus1]);

// SAME
// For final values; consuming less memories
AmountActutalSum = SUMX(FactFinance, FactFinance[AmountActual] + 1);
```

Iterator functions = extended functions

## Other statistical functions

**Table Expansion & Creation**

| **Function**   | **Formula Placeholder**                                            | **Description**                                                                     |
| -------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| **ADDCOLUMNS** | `ADDCOLUMNS(<table>, "<name>", <expression>, ...)`                 | Adds new calculated columns to an existing table or table expression.               |
| **ROW**        | `ROW("<name>", <expression>, ...)`                                 | Returns a table with a single row containing values from the specified expressions. |
| **SAMPLE**     | `SAMPLE(<n_value>, <table>, <orderBy_expression>, [<order>], ...)` | Returns a sample of $N$ rows from a table, often used for data exploration.         |
| **TOPN**       | `TOPN(<n_value>, <table>, <orderBy_expression>, [<order>], ...)`   | Returns the top $N$ rows of a table based on a specific sort order.                 |

**Counting & Aggregation**

| **Function**      | **Formula Placeholder**   | **Description**                                                             |
| ----------------- | ------------------------- | --------------------------------------------------------------------------- |
| **COUNTROWS**     | `COUNTROWS(<table>)`      | Counts the total number of rows in the specified table or filtered context. |
| **DISTINCTCOUNT** | `DISTINCTCOUNT(<column>)` | Counts the unique values in a column, ignoring duplicates.                  |

**Percentiles**

| **Function**        | **Formula Placeholder**                       | **Description**                                                                                    |
| ------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **PERCENTILE.EXC**  | `PERCENTILE.EXC(<column>, <k>)`               | Returns the $k$-th percentile of values in a column ($0 < k < 1$), **excluding** endpoints.        |
| **PERCENTILE.INC**  | `PERCENTILE.INC(<column>, <k>)`               | Returns the $k$-th percentile of values in a column ($0 \le k \le 1$), **including** endpoints.    |
| **PERCENTILEX.EXC** | `PERCENTILEX.EXC(<table>, <expression>, <k>)` | Evaluates an expression for each row in a table and returns the $k$-th percentile (**exclusive**). |
| **PERCENTILEX.INC** | `PERCENTILEX.INC(<table>, <expression>, <k>)` | Evaluates an expression for each row in a table and returns the $k$-th percentile (**inclusive**). |

**Ranking**

| **Function** | **Formula Placeholder**                                        | **Description**                                                                                          |
| ------------ | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **RANK.EQ**  | `RANK.EQ(<value>, <column>, [<order>])`                        | Returns the rank of a number in a column relative to other numbers in that column.                       |
| **RANKX**    | `RANKX(<table>, <expression>, [<value>], [<order>], [<ties>])` | Returns the rank of an expression evaluated in the current context against a list of values for a table. |

```jsx
Rank = RANK.EQ(FactFinance[Amount], FactFinance[Amount], ASC); // small to big
// what are you looking for, where are you looking for, order
// EQ, same value, same rank, skip # of same rank
// EESC // big to small
```

## Practice Activity

```jsx
ProcessingFeeColumn = SWITCH(
  DimGeography[EnglishCountryRegionName],
  'United States',
  1,
  'Canada',
  2,
  5,
);

ProcessingFeeAverage = AVERAGEX(
  DimGeography,
  SWITCH(
    DimGeography[EnglishCountryRegionName],
    'United States',
    1,
    'Canada',
    2,
    3,
  ),
);

RankEQ = RANK.EQ(
  DimGeography[SalesTerritoryKey],
  DimGeography[SalesTerritoryKey],
  DESC,
);
```

# 5. Expanding our data model and information Functions

## Getting additional tables and creating relationships

Import Data → Relationship; Edit relationship → Use it as visualizations

!image.png

!image.png

## Inactive relationships and cross-filter direction

Use single as possible, then use both when it required

Both is slow & Will have missing value

Or you can use DAX

!image.png

!image.png

!image.png

Location (Fact) → Organization Name (Dim)

Fact 테이블에서 필터를 하는 경우, Both; Cross-filter 혹은 DAX 필요

## Using information functions

```jsx
Divide = **IF(ISERROR**(4 / 0), BLANK(), 4/0)
Divide = **IFERROR**(4 / 0, BLANK())

OrganizationName = LOOKUPVALUE(
  DimOrganization[OrganizationName], // search this // rel table
  DimOrganization[OrganizationKey], // search using key // rel table
  FactFinance[OrganizationKey] // this info in current table
) // return single value
```

## Practice Activity

```jsx
Group = LOOKUPVALUE(
  DimSalesTerritory[SalesTerritoryGroup],
  DimSalesTerritory[SalesTerritoryKey],
  DimGeography[SalesTerritoryKey],
);

Country = IFERROR(
  LOOKUPVALUE(
    DimSalesTerritory[SalesTerritoryCountry], // retrieves this in rel table
    DimSalesTerritory[SalesTerritoryGroup], // using this in rel table
    DimGeography[Group], // using this in current table
  ),
  BLANK(),
);
```

# 6. Filter and Value Functions

## The RELATED function

```jsx
OrganizationName2 = RELATED(DimOrganization[OrganizationName]);
// = OrganizationName = LOOKUPVALUE(DimOrganization[OrganizationName], DimOrganization[OrganizationKey], FactFinance[OrganizationKey])
// No need to state join key in current & rel table
// it only work when it has **active relationship**
// many to one

// inactive reltionship then you should use LOOKUPVALUE function

// When do you need RELATED?
// When you want to add more or change it
OrganizationName2 =
  RELATED(DimOrganization[OrganizationName]) &
  ' (' &
  FactFinance[ScenarioKey] &
  ')';

// When you want to hide field in dimension table
// Fact table: use RELATED function
// Dimension table: hide the field
```

## Using the RELATEDTABLE and COUNTROWS functions

```jsx
// Many to one; Return one value; RELATED; Fact -> Dim

// One to Many; Return manay value; RELATEDTABLE; Dim -> Fact

// RELATEDTABLE; return table expression
// Used in Fact table; Using Dim talbe to aggregate date
Answer = SUMX(
  // aggregate to single value
  RELATEDTABLE(FactFinance), // table expression
  FactFinance[Amount], // expression
);

Answer = COUNTX(
  // not include blank
  RELATEDTABLE(FactFinance),
  FactFinance[AmountActual],
);

Answer = COUNTROWS(RELATEDTABLE(FactFinance)); // only accept table expression
```

## Context

```jsx
Answer = SUMX(RELATEDTABLE(FactFinance), FactFinance[Amount]);
// RELATEDTABLE(FactFinance)
// Use Current Context

Answer = SUMX(FactFinance, FactFinance[Amount]);
// FactFinance
// Entire table
// Iterate each row
// Can use expression

Answer = SUM(FactFinance[Amount]);
// Entire table
// Faster

Answer4 = DimOrganization[Answer1] / DimOrganization[Answer2];
// % of total
```

!image.png

Context for measure - Layout (row & col)

!image.png

## The ALL function

!image.png

Problem: Answer4 (Calc field); only has organization name context, not date context, it show all same value for date columns

```jsx
PercentOfTotal =
  SUM(FactFinance[Amount]) / SUMX(ALL(FactFinance), FactFinance[Amount]);
// SUM(FactFinance[Amount]): measure; no need related or iterate function
// ALL(FactFinance); no conext; no layout context
// SUMX(table expression, expression)
```

!image.png

## Filter

```jsx
// Add context
Amount2028 = SUMX(
  FILTER(
    FactFinance, // table
    FactFinance[Date].[Year] = 2028 // condition
  ), // return table
  FactFinance[Amount]
)
```

!image.png

## The CALCULATE function

`ALL` - Remove context

`FILTER` - Add filter

`CALCULATE(expression, filter, ... )`

```jsx
PercentOfTotal =
  SUM(FactFinance[Amount]) /
  CALCULATE(
    SUMX(FactFinance, FactFinance[Amount]), // expression
    ALL(FactFinance), // filter
  ); // don't need to modify table
// SUMX(ALL(FactFinance), FactFinance[Amount]
// modify table
```

```jsx
// Amount2028 = SUMX(FILTER(FactFinance, FactFinance[Date].[Year] = 2028), FactFinance[Amount])
Amount2028 = CALCULATE(SUMX(FactFinance, FactFinance[Amount]), FILTER(FactFinance, FactFinance[Date].[Year] = 2028))
```

!image.png

## The ALLEXCEPT function

All function but except the column you specific mention

```jsx
ActualExcept = CALCULATE(
  SUM(FactFinance[Amount]),
  ALLEXCEPT(
    FactFinance, // table; remove all context
    DimOrganization[OrganizationName], // except 1; add context if there have
    FactFinance[Date].[Year] // except 2; add context if there have
  )
)
```

!image.png

if they don’t have context (ex. organization name) then filter doesn’t do anything; don’t throw the error

## The AllSELECTED function

```jsx
AllSelected = CALCULATE(
  SUM(FactFinance[Amount]), // all
  ALLSELECTED(DimOrganization[OrganizationName]), // remove filters
  // same value for this fields
  // same total amount for organization name
);
```

ALL(tablename) = ALLSELECTED(tablename)

| **비교 항목**                | **ALL**                                      | **ALLSELECTED**                                              | **ALLEXCEPT**                                                    |
| ---------------------------- | -------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------------- |
| **필터 제거 범위**           | **모든** 필터를 무시                         | 시각적 개체 **내부(행/열)** 필터만 무시                      | 지정한 열을 **제외한 모든** 필터 무시                            |
| **슬라이서(외부 필터) 영향** | 슬라이서 조건도 **무시함**                   | 슬라이서 조건은 **유지함**                                   | 지정된 열의 슬라이서만 **유지함**                                |
| **핵심 목적**                | 절대적인 **전체 총계** (Absolute Total)      | 화면에 선택된 데이터 내의 **소계** (Visual Total)            | 특정 기준(그룹)에 대한 **고정값** (Fixed Total)                  |
| **비율 계산 시나리오**       | "우리 회사 역대 총매출 중 이 제품의 비중은?" | "네가 방금 검색한 조건(예: 2023년) 내에서 이 제품의 비중은?" | "다른 조건 상관없이, 이 '제품군' 전체 매출 중 이 제품의 비중은?" |

## Other filter functions

### 1. `CROSSFILTER(col1, col2, direction)`

- **기능:** 두 테이블 간의 **필터 방향을 일시적으로 변경**합니다. (데이터 모델 자체를 바꾸지 않고, 해당 측정값을 계산할 때만 방향을 조작합니다.)
- **Direction(방향) 옵션:** `Both`(양방향), `None`(관계 없음), `OneWay`(단방향)
- **예시 상황:** '제품' 테이블에서 '매출' 테이블로 단방향 필터가 걸려 있는데, 역으로 매출 데이터를 기준으로 제품을 필터링해야 할 때.

```
양방향_매출 =
CALCULATE(
    [총 매출],
    CROSSFILTER('Sales'[ProductID], 'Product'[ProductID], Both)
)
```

- **의미:** 이 수식을 계산할 때만 `Sales` 테이블과 `Product` 테이블의 필터 방향을 양방향(Both)으로 열어줍니다.

### 2. `DISTINCT(col)`

- **기능:** 지정한 열(Column)에서 중복을 제거한 고유한 값들의 목록(테이블)을 반환합니다.
- **예시 상황:** 영수증 데이터(매출 테이블)에 고객 ID가 수백만 줄 찍혀 있을 때, "실제로 물건을 사간 순수 고객은 몇 명이지?"를 알고 싶을 때.

```
순수_구매_고객수 =
COUNTROWS(
    DISTINCT('Sales'[CustomerID])
)
```

- **의미:** `Sales` 테이블의 고객 ID에서 중복을 싹 다 지우고 고유한 ID만 남긴 뒤(`DISTINCT`), 그 줄 수를 세어서(`COUNTROWS`) 순수 고객 수를 구합니다. _(참고: `DISTINCTCOUNT` 함수와 같은 역할을 할 때가 많습니다.)_

### 3. `HASONEVALUE(col)`

- **기능:** 현재 필터 컨텍스트에서 특정 열의 값이 **딱 1개만 남아있는지 확인**합니다. (1개면 `TRUE`, 2개 이상이거나 없으면 `FALSE`). 주로 표(Matrix)의 **'총계(Total)' 행에서 다른 계산을 하거나 빈칸을 만들 때** 씁니다.
- **예시 상황:** 표의 각 행(커피, 차, 디저트)에는 단가가 나오게 하고, 맨 아래 '총계' 행에는 단가들의 합이 나오면 이상하니까 그냥 빈칸으로 두고 싶을 때.

```
단가_표시 =
IF(
    HASONEVALUE('Product'[Category]),
    [평균 단가],
    BLANK()
)
```

- **의미:** 현재 계산하는 줄이 '단일 카테고리(값 1개)'라면 평균 단가를 보여주고, '총계' 행처럼 여러 카테고리가 섞여서 값이 1개가 아니라면 빈칸(`BLANK()`)을 반환하라는 뜻입니다.

### 4. `KEEPFILTERS(expression)`

- **기능:** `CALCULATE` 함수 안에서 필터를 적용할 때, **기존의 외부 필터를 덮어쓰지 않고 "교집합(AND)"으로 겹쳐서 적용**합니다. (이전 노트에서 `FILTER` 함수 대신 쓰면 성능이 좋아진다고 언급된 바로 그 함수입니다!)
- **예시 상황:** 매트릭스 표에 빨강, 파랑, 검정 제품들이 나열되어 있습니다. 특정 측정값에서 "빨간색 매출"만 콕 집어서 구하고 싶을 때.

```
// 일반 CALCULATE (덮어쓰기)
빨간색_매출_일반 = CALCULATE([매출], 'Product'[Color] = "Red")

// KEEPFILTERS 사용 (교집합)
빨간색_매출_보존 = CALCULATE([매출], KEEPFILTERS('Product'[Color] = "Red"))
```

- **두 방식의 차이점:**
  - **일반:** 사용자가 '파란색' 행을 보고 있어도, 무조건 기존 필터를 무시하고 '빨간색' 매출을 보여줍니다. (파란색 줄에 빨간색 매출액이 나오는 이상한 현상 발생)
  - **`KEEPFILTERS`:** 기존 필터를 존중합니다. '파란색' 행에서는 (색상=파랑 AND 색상=빨강)이 되어 조건이 안 맞으므로 빈칸이 됩니다. 오직 원래부터 '빨간색'인 행에서만 값을 보여줍니다.

**💡 한 줄 요약:**

- `CROSSFILTER`: 이 계산 할 때만 필터 **방향 바꿔줘!**
- `DISTINCT`: 중복 빼고 **고유값만 남겨줘!**
- `HASONEVALUE`: 지금 이 칸에 데이터가 **딱 1개 항목만 있는 거 맞아?**
- `KEEPFILTERS`: 원래 있던 사용자 필터 무시하지 말고 **합쳐서(교집합) 계산해줘!**

## Practice Activity

```jsx
// DimCustomer
Region = RELATED(DimSalesTerritory[SalesTerritoryRegion]);

// DimSalesTerritory
NumberOfCustomers = COUNTROWS(RELATEDTABLE(DimCustomer));

// DimCustomer
NumberSingle = CALCULATE(
  COUNT(DimCustomer[CustomerKey]),
  FILTER(DimCustomer, (DimCustomer[MaritalStatus] = 'S')), // fixed value
);

GrandTotal = CALCULATE(
  COUNT(DimCustomer[CustomerKey]),
  ALL(), // total; same value for each row
);

NumberContinent = CALCULATE(
  COUNT(DimCustomer[CustomerKey]),
  ALLSELECTED(DimSalesTerritory[SalesTerritoryRegion]),
  // visual filter; same value for selected filter
);
```

!image.png

# 7. Date, Time and Time Intelligence functions

## Date and Time functions and creating a Date Table

Create a new table

```jsx
DataTable = CALENDAR(DATE(2026, 1, 1), DATE(2031, 12, 31));
// recommended way
// specific end & start date

DataTable = CALENDARAUTO(12); // default: 12
// fiscal year end
// automatically genderate earliest and latest date
// cons: include not neccessary date
```

| **Parameter**      | **Fiscal Year End** | **Table Start Date (if data starts Aug 2022)** |
| ------------------ | ------------------- | ---------------------------------------------- |
| `CALENDARAUTO(12)` | December 31         | January 1, 2022                                |
| `CALENDARAUTO(6)`  | June 30             | July 1, 2021                                   |
| `CALENDARAUTO(3)`  | March 31            | April 1, 2021                                  |

Mark as a date table

!image.png

Create a new column

```jsx
Year = YEAR(DateTable[Date]);
Month = MONTH(DateTable[Date]);
Day = DAY(DateTable[Date]);
```

Create a relationship

!image.png

## Using the USERELATIONSHIP, EDATE and CROSSFILTER functions

!image.png

```jsx
DateDue = EDATE(FactFinance[Date], 1); // + 1 month

DateDueYear = CALCULATE(
  MIN(DateTable[Year]), // force to return single value
  USERELATIONSHIP(
    DateTable[Date], // col 1
    FactFinance[DateDue], // col 2
  ),
  CROSSFILTER(
    // modifity individual field's corss filter
    DateTable[Date],
    FactFinance[DateDue],
    Both, // direction
  ),
);

DateDueMonth = CALCULATE(
  MIN(DateTable[Month]),
  USERELATIONSHIP(DateTable[Date], FactFinance[DateDue]),
  // CROSSFILTER(DateTable[Date], FactFinance[DateDue], Both)
  // when you edit cross filter in model
);
```

## Time Intelligence Functions

```jsx
AmountCalc = CALCULATE(SUM(FactFinance[Amount]))

AmountCalc = CALCULATE(SUM(FactFinance[Amount]), **DATESYTD**(FactFinance[Date]))
// Reset every year (quater, month)
// expression, filter

AmountCalc2 = **TOTALYTD**(SUM(FactFinance[Amount]), FactFinance[Date])
// expression, cols
// same
```

!image.png

```jsx
AmountCalc3 = CALCULATE(SUM(FactFinance[Amount]), **DATEADD(FactFinance[Date], -12, MONTH**))
// get -12 mont before value

// alternative
AmountCalc4 = CALCULATE(SUM(FactFinance[Amount]), **SAMEPERIODLASTYEAR(FactFinance[Date])**)

// same as DATEADD, but more flexible
// return whole month, instead of extact date
AmountCalc5 = CALCULATE(SUM(FactFinance[Amount]), **PARALLELPERIOD(FactFinance[Date], 12, MONTH))**
```

```jsx
// next month
AmountCalc6 = CALCULATE(SUM(FactFinance[Amount]), **NEXTMONTH(FactFinance[Date])**)
```

!image.png

---

Power BI의 시계열(Time Intelligence) 함수는 크게 누적(Cumulative)과 **이동(Shift)** 두 가지 패턴으로 나뉩니다. 보내주신 노트를 바탕으로 실무에서 헷갈리지 않게 그룹화하여 정리해 드릴게요.

### 1. 누적 데이터 계산 (YTD, QTD, MTD)

특정 시점부터 현재까지 쌓인 값을 계산할 때 사용합니다. 매년/매분기/매월 초에 값이 **0으로 리셋**되는 것이 특징입니다.

| **함수명**   | **특징**                              | **노트 기반 예시**                        |
| ------------ | ------------------------------------- | ----------------------------------------- |
| **DATESYTD** | `CALCULATE`의 필터로 사용 (가장 정석) | `CALCULATE([AmountCalc], DATESYTD(Date))` |
| **TOTALYTD** | `CALCULATE` 없이 단독 사용 (간결함)   | `TOTALYTD(SUM(Amount), Date)`             |

> **Tip:** `TOTALYTD`는 내부적으로 `CALCULATE`를 실행하는 "Shortcut(지름길)" 함수입니다. 결과는 동일하지만, 복잡한 필터 조건이 추가될 때는 `CALCULATE + DATESYTD` 조합이 더 유리합니다.

### 2. 시간 이동 및 비교 (Shift)

과거 혹은 미래의 데이터를 가져와 현재와 비교(YoY, MoM 등)할 때 사용합니다.

**A. 작년 동일 기간 비교**

- **SAMEPERIODLASTYEAR**: 딱 **1년 전** 데이터만 필요할 때 쓰는 직관적인 함수입니다.
- **DATEADD**: 숫자(`12`)와 단위(`MONTH`)를 직접 지정하므로 훨씬 **유연**합니다. (2년 전, 1달 전 모두 가능)

**B. 기간의 범위 차이 (DATEADD vs PARALLELPERIOD)**

이 부분이 실무에서 가장 많이 헷갈리는 포인트입니다.

- **DATEADD**: **날짜 단위**로 이동합니다. (오늘이 3월 15일이면, 작년 3월 15일 데이터를 찾음)
- **PARALLELPERIOD**: 전체 기간(Full Period)을 반환합니다. (오늘이 3월 15일이라도, 작년 3월의 1일부터 31일까지 전체 합계를 가져옴)

### 3. 특정 시점 이동 (Next/Previous)

현재 문맥에서 바로 다음 혹은 이전 기간의 데이터를 가져옵니다.

- **NEXTMONTH**: 현재 보고 있는 달의 바로 다음 달 데이터를 반환합니다. (예산 대비 실적 비교 등에 활용)
- **PREVIOUSMONTH**: (노트에는 없지만 세트 메뉴) 바로 이전 달의 데이터를 반환합니다.

### 💡 한눈에 보는 요약 테이블

| **카테고리** | **함수**             | **핵심 용도**                       | **유연성**    |
| ------------ | -------------------- | ----------------------------------- | ------------- |
| **누적**     | `DATESYTD`           | 연초부터 오늘까지 합계              | 보통          |
| **누적**     | `TOTALYTD`           | 위와 동일 (함수 하나로 끝낼 때)     | 낮음          |
| **비교**     | `SAMEPERIODLASTYEAR` | 정확히 1년 전 동일 기간             | 낮음          |
| **비교**     | `DATEADD`            | 자유로운 시점 이동 (-1년, +3월 등)  | **매우 높음** |
| **범위**     | `PARALLELPERIOD`     | 해당 월/분기/연도의 **전체** 데이터 | 높음          |
| **단일**     | `NEXTMONTH`          | 다음 달의 전체 값                   | 낮음          |

**실무 활용 팁**

노트의 `AmountCalc3`(-12, MONTH)과 `AmountCalc4`(`SAMEPERIODLASTYEAR`)는 결과가 같습니다. 하지만 실무에서는 **`DATEADD`** 하나만 제대로 익혀두는 것을 추천드려요. 숫자만 바꾸면 전월(MoM), 전분기(QoQ), 전년(YoY)을 모두 만들 수 있기 때문입니다.

혹시 이 함수들을 활용해서 전년 대비 성장률(YoY %)을 구하는 계산식도 함께 정리해 드릴까요?

## Pra

```jsx
YearStart = STARTOFYEAR(DimCustomer[DateFirstPurchase])

RunningTotal = **TOTALMTD**(COUNT(DimCustomer[CustomerKey]), DimCustomer[DateFirstPurchase])
RunningTotal = CALCULATE(COUNT(DimCustomer[CustomerKey]), **DATESMTD(DimCustomer[DateFirstPurchase]))**

PrevYear = CALCULATE(COUNT(DimCustomer[CustomerKey]), **DATEADD(DimCustomer[DateFirstPurchase], -12, MONTH))** // -4, QUATER // -1, YEAR
PrevYear = CALCULATE(COUNT(DimCustomer[CustomerKey]), **SAMEPERIODLASTYEAR(DimCustomer[DateFirstPurchase]))**

CurrentMonth = CALCULATE(COUNT(DimCustomer[CustomerKey]), **PARALLELPERIOD(DimCustomer[DateFirstPurchase], 0, MONTH))**
```

# 8. Other PL 300 exam topics needed for the DP 600 exam

## Performance improvements to report visual and DAX

**1. Monitoring & Discovery: The Diagnostic Phase**

Before making changes, identify the actual bottleneck. Most "slowness" is caused by either a heavy DAX engine load or a visual rendering bottleneck.

- **Performance Analyzer:** Use it to record how long each visual takes.
  - **DAX Query:** Time spent calculating.
  - **Visual Display:** Time spent rendering lines, bars, and colors.
  - **Other:** Time spent waiting on other visuals or network lag.
- **DAX Query View:** A built-in sandbox to test and optimize specific queries without leaving the report.
- **DAX Studio:** The power-user tool. Use it to check **Server Timings** and see if your query is hitting the **Formula Engine (FE)** (slow/single-threaded) or the **Storage Engine (SE)** (fast/multi-threaded).

**2. Data Modeling: The Foundation**

A bad model can't be "fixed" with clever DAX. It is the most common cause of poor performance.

- **Star Schema > Snowflake:** Power BI is built for Star Schemas (central Fact table surrounded by Dimension tables). Snowflake models (Dimensions connected to other Dimensions) create longer relationship chains that slow down the engine.
- **Relationship Management:**
  - **Many-to-Many:** Avoid these. They require a "bridge" in the background that consumes high memory. Convert to One-to-Many whenever possible.
  - **Bi-directional Filters:** Use with extreme caution. They can cause circular dependencies and force the engine to work twice as hard to resolve filter contexts.
- **Cardinality Reduction:**
  - **Delete, Don't Hide:** Hidden columns still occupy RAM. If you don't need a column for a visual or a calculation, **delete it** in Power Query.
  - **High Cardinality:** Columns with unique values (like "Transaction ID" or "Timestamp with Seconds") prevent the VertiPaq engine from compressing data.

**3. Visual & UI Optimization: Reducing the "Noise"**

Even with fast DAX, a page with too many elements will feel sluggish to the end-user.

- **Visual Density:** Every visual sends at least one query. If you have 30 visuals on a page, the browser will queue them, causing a "spinning" effect.
  - **Solution:** Use **Buttons** and **Bookmarks** to swap visuals, or **Drill-through** to move details to a separate page.
- **Chart Granularity:** Avoid showing 10,000 data points on a single scatter or line chart. Use "Top N" filters to limit the display to the most important data.
- **Slicers:** Slicers are "expensive" visuals. Consider using the **Filter Pane** for complex filtering to reduce the number of queries triggered on page load.

**4. DAX Optimization: Writing Efficient Code**

Optimized DAX minimizes the work the Formula Engine has to do.

- **Variables (`VAR`):** Variables evaluate once and store the result.
  - _Bad:_ `IF([Measure] > 0, [Measure], 0)` (Calculates the measure twice).
  - _Good:_ `VAR val = [Measure] RETURN IF(val > 0, val, 0)` (Calculates once).
- **`DIVIDE` vs. `/`:** Always use `DIVIDE(a, b)` as it handles "divide by zero" errors more efficiently than a manual `IF` check.
- **Avoid `FILTER(Table)`:** Avoid filtering an entire fact table. Instead, filter specific columns using `KEEPFILTERS` or `CALCULATE`.
- **Use `COUNTROWS`:** It is significantly faster than `COUNT` because it only looks at the table structure rather than checking every row for non-blank values.

**5. Advanced: DAX Studio & VertiPaq Analyzer**

When you reach the "pro" level, you optimize the physical storage of the data.

- **VertiPaq Analyzer:** Check which columns take up the most % of your database. You’ll often find that one "Long Text" column is taking up 90% of your file size.
- **Clear Cache:** When testing DAX speed, always clear the cache. Otherwise, Power BI will return the result from memory, making a "bad" query look "fast."

**Quick Tip:** If a report is slow, **Performance Analyzer** usually tells you the "Who," and **DAX Studio** tells you the "Why."

Would you like me to dive deeper into how to use **DAX Studio** to identify if a query is "FE-heavy" or "SE-heavy"?

## Choose a storage mode

**1. Import Mode: The "Speed King"**

This is the default and most common mode. Power BI takes a "snapshot" of your data and loads it into its internal memory (RAM).

- **How it works:** Data is compressed and stored in the Power BI cache using the VertiPaq engine.
- **Performance:** ⚡ **Extremely Fast.** All calculations and filters happen instantly in RAM.
- **The "Cost":** \* **Memory:** Limited by your workspace capacity (e.g., 1GB for Pro).
  - **Freshness:** Data is only as current as the last **Scheduled Refresh**.
  - **Load Time:** The initial import and subsequent refreshes take time for large datasets.

**2. DirectQuery: The "Live Link"**

Instead of importing data, Power BI stays connected to the source and sends "live" queries whenever a user interacts with a visual.

- **How it works:** Only the **schema** (table structures, names, relationships) is stored. No actual data lives in the Power BI file.
- **Performance:** 🐢 **Slower.** Every click sends a request to the source database. Speed is limited by the database's hardware and network latency.
- **The Benefits:** \* **Real-time:** You always see the latest data available in the source.
  - **Massive Scale:** Ideal for "Big Data" that simply won't fit in memory.

**3. Dual Mode: The "Hybrid"**

A table in Dual mode can behave as either Import or DirectQuery depending on the context of the visual.

- **The Logic:** Usually applied to **Dimension tables** (like Date or Product) that are shared between an Imported fact table and a DirectQuery fact table.
- **The Benefit:** It prevents "Cross-source" joins. If you visualize an Imported Fact with a Dual Dimension, it uses the cached data. If you use a DirectQuery Fact, it stays in DirectQuery mode to avoid a performance penalty.

**4. Switching Storage Modes (The "One-Way" Rule)**

You can change this in **Model View > Properties > Advanced > Storage Mode**.

- **DirectQuery → Import (O):** Allowed. Power BI will download the data and cache it.
- **Import → DirectQuery (X):** **Impossible.** Once data is imported, you cannot convert the table back to DirectQuery. You would have to delete the table and re-connect.

**5. Composite Models: Mix & Match**

A Composite Model is a single report that combines different storage modes.

- **Example:** You **Import** your "Sales Targets" but use **DirectQuery** for your "Transaction Records" (billions of rows).
- **Optimization Tip:** This provides high-level summary speed (Import) with the ability to "drill through" into real-time details (DirectQuery).

**Summary Table for Quick Memory**

| **Feature**       | **Import**        | **DirectQuery**     | **Dual**      |
| ----------------- | ----------------- | ------------------- | ------------- |
| **Data Location** | Power BI RAM      | Source Database     | Both          |
| **Speed**         | ⚡ Very Fast      | 🐢 Slower           | Fast/Variable |
| **Freshness**     | Scheduled Refresh | **Always Latest**   | Mixed         |
| **Changeable?**   | No (to DQ)        | **Yes** (to Import) | Yes           |

## Improve query performance by using query

**1. Reduce Data Volume (Rows & Columns)**

- **The Concept:** Power BI stores data in memory using a columnar database engine (VertiPaq). This means the number of columns, and the uniqueness of the data inside them, impacts performance more than just the raw row count.
- **Reduce Columns:** Always delete unnecessary columns before loading data into the model. Pay special attention to **high-cardinality columns** (columns with many unique values, like transaction IDs, exact timestamps, or primary keys). These columns cannot be compressed efficiently and will consume massive amounts of memory and slow down performance.
- **Reduce Rows:** Filter the data as early as possible. If your business only reports on the last 3 years of data, do not import 10 years of history.

**2. Query Folding (The Core Concept)**

- **What it is:** Query Folding is the ability of Power Query to translate your data transformation steps into a single native query (like a SQL statement) and push the computation back to the source database.
- **Power BI vs. SQL Server Processing:** \* _Without Query Folding:_ Power BI must download all the raw data (e.g., millions of rows) over the network and process the filters and aggregations locally using its own memory and CPU. This is slow and inefficient.
  - _With Query Folding:_ The SQL Server does the heavy lifting. It applies the filters and aggregations on the server side and _only sends the final outcome_ (e.g., a few thousand rows) to Power BI.
- **How to Verify:** In the Power Query Editor, look at the **Applied Steps** pane on the right.
  - Right-click a step and look for **View Native Query**.
  - **If it is clickable:** Query folding is active. The step has been translated to SQL and will be handled by the server.
  - **If it is greyed out:** Query folding is broken. The source cannot process this step, meaning Power BI will handle this step (and all steps after it) locally.
- **Best Practice:** Always place steps that support query folding (like removing columns, filtering rows, and grouping) at the _top_ of your Applied Steps list. Place non-folding steps (like complex custom M code or changing certain data types) at the very _bottom_ so you don't break the folding chain prematurely.

**3. Group By (Pre-Aggregation)**

- **The Concept:** Summarizing data at the source level using the "Group By" feature in Power Query.
- **Why it matters:** If your final report only requires viewing total sales by _Month_ and _Product Category_, there is no need to load millions of individual daily receipt rows into your semantic model.
- **The Benefit:** By grouping and aggregating the data in Power Query (ideally while Query Folding is active), you drastically reduce the physical size of the `.pbix` file. This results in much faster scheduled refreshes and instantly responsive report pages.

**4. Performance Monitoring Tools**

- **Performance Analyzer (Power BI Desktop):**
  - _What it does:_ It is a built-in tool found under the "Optimize" ribbon that records exactly how long it takes for report elements to load when a user interacts with the page (e.g., clicking a slicer).
  - _How to use it:_ It breaks down the loading time in milliseconds for each visual into three categories:
    - **DAX Query** (how long the calculation took)
    - **Visual Display** (how long it took to render the chart)
    - **Other** (time spent waiting in the queue).
  - _Actionable Insight:_ Use this to pinpoint exactly which visual or DAX measure is the bottleneck. As a general rule, any DAX query taking longer than 120 milliseconds is a prime candidate for optimization.

## Exploring the Power BI Service Interface

!image.png

## Implement workspace-level access controls

Power BI Pro license to create new workspace. (Free - only for my workspace; can’t share with other)

Manage Access

- Workspace Access
  - Admin
    - Update and delete the workspace
    - Add/remove people, including other admins
    - Allow Contributors to update the app for the workspace
    - Manage subscriptions created by others
  - Member (or above)
    - Add members or others with lower permissions.
    - Publish, unpublish, and change permissions for an app
    - Update an app.
    - Share an item (including semantic models) or share an app.
    - Allow others to reshare items.
    - Feature apps on colleagues’ Home
    - Manage semantic model permission
  - Contributor (or above)
    - **Update the app (if allowed)**
    - Feature dashboards and reports on colleagues’ Home
    - Publish, create, edit and delete content in the workspace.
    - Create a report in another workspace based on a semantic model in this workspace.
    - Copy a report (if you have Build permission for the semantic model)
    - Create subscriptions for others to a report.
    - Analyze the semantic model in Excel.
    - If you have permissions on the Gateway:
      - Schedule data refreshes via the on-premises gateway
      - Modify gateway connection settings
  - Viewer (or above)
    - View and interact with an item
      - Need Power Pro BI license($40 per user), or items in a workspace which has a Power BI Premium capacity ($8k per capacity)
    - Read data stored in workspace dataflows (gen 1)
    - Create subscriptions for yourself to report
    - Analyze the semantic model in Excel (if you have Build Permission)

| **Role**        | **Core Purpose**            | **Defining Capabilities**                                                                                                                                        |
| --------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Admin**       | **Total Control**           | The only role that can update or delete the workspace itself, manage other Admins, and control subscriptions created by anyone.                                  |
| **Member**      | **App & Access Management** | Focused on distribution. They can add lower-level users, fully manage apps (publish, unpublish, change permissions), and manage semantic model permissions.      |
| **Contributor** | **Content Creation**        | The primary developers. They can create, edit, delete, and publish content (reports/dashboards), as well as schedule data refreshes via gateways.                |
| **Viewer**      | **Read-Only Consumption**   | Limited strictly to viewing and interacting with finished items. They can only create subscriptions for themselves (requires a Pro license or Premium capacity). |

**The Hierarchy of Access**

Each tier inherits the permissions of the tiers below it, adding a specific layer of responsibility:

- **Viewers** consume the data.
- **Contributors** build and edit the data and reports.
- **Members** decide who gets to see the reports and manage the Apps.
- **Admins** manage the entire workspace infrastructure and rule over all users.

## Implement incremental refresh for semantic models

**What is Incremental Refresh?**

Instead of truncating and reloading the entire dataset (e.g., gigabytes of data) every time you refresh your report, incremental refresh partitions the data. The Power BI Service stores the massive historical data (Archive) and **only queries the source system for the newest data** (Incremental).

- **The Result:** Refreshes take seconds/minutes instead of hours, consume vastly fewer resources on your SQL Server, and keep your reports highly responsive.
- **CRITICAL PREREQUISITE:** **Query Folding** _must_ be active for the table. If query folding is broken, Power BI will download the entire historical database just to filter it locally, which destroys the entire purpose of incremental refresh and will crash your gateway.

**Step 1: Set up Parameters in Power Query**

You must tell Power Query how to slice the data using parameters.

1. **Create Parameters:** Go to _Home → Manage Parameters → New Parameter_.
2. **Naming Convention:** You **MUST** name them exactly `RangeStart` and `RangeEnd` (case-sensitive). If you misspell them, Power BI will not recognize them for incremental refresh.
3. **Data Type:** They **MUST** be set to the `Date/Time` data type. (Even if your column is just a Date, these parameters must be Date/Time). Set a temporary current value (e.g., a 1-month range from last year) just to filter your preview data.
4. **Apply the Filter:** Go to your fact table (e.g., Sales), find your Date column (e.g., OrderDate), and choose _Date/Time Filters → Custom Filter_.
5. **Set the Logic:**
   - `is greater than or equal to (>=)` **RangeStart**
   - `AND`
   - `is strictly less than (<)` **RangeEnd**

   _(Note: Using >= and < ensures that if a transaction happens exactly at midnight, it isn't counted twice across two different days)._

6. **Close & Apply:** Load the filtered data into the Desktop model.

**Step 2: Define the Policy in Power BI Desktop**

Now you define the rules of how the data will be archived and updated.

1. **Access the Menu:** In the Model view or Data view, right-click the table name and select **Incremental Refresh**.
2. **Archive Data Starting (Historical Range):**
   - _Example: "Store rows from the last 3 Years."_
   - This tells the Power BI Service how much historical data to keep in memory.
3. **Incrementally Refresh Data (Rolling Window):**
   - _Example: "Refresh rows from the past 10 Days."_
   - This means every time the schedule triggers, Power BI drops the last 10 days of data and re-queries the SQL server for _only_ the last 10 days.
4. **Optional - Only refresh complete days:** Check this if you don't want today's partial data loaded until the day actually ends at midnight.
5. **Optional - Detect data changes:** If your SQL database has a `LastUpdateDate` column, you can select it here. Power BI will only refresh days where the `LastUpdateDate` has changed, saving even more processing power.

**Step 3: Publish to Power BI Service (The Point of No Return)**

Incremental refresh **does not work in Power BI Desktop**. Desktop will only ever show the data between the temporary `RangeStart` and `RangeEnd` values you manually typed in.

- **The First Refresh:** After you publish to the Service, you must manually trigger the first refresh. **This first refresh will take a very long time** because it is building the entire 3-year historical archive from scratch.
- **Subsequent Refreshes:** The next scheduled refreshes will be lightning-fast because they only fetch the 10-day incremental window.
- **⚠️ Important Limitation:** As your notes mentioned, once you publish a dataset with an incremental refresh policy applied, the dataset becomes heavily partitioned in the cloud. You generally **cannot download the `.pbix` file back to Power BI Desktop** (unless you are using advanced XMLA endpoint tools in Premium capacity). Always keep a backup of your original Desktop file!

## Implement item-level access controls

**(Report) Share Link**

Sharing a link is the most common way to grant individuals access to a specific report without adding them to the entire workspace.

**1. Who (Target Audience)**

When you click "Share," you must choose who the link will work for:

- **People in your organization:** Creates a shareable link that works for anyone inside your company's tenant. _Caution: This link can be forwarded, and anyone in the company who clicks it will gain access._
- **People with existing access:** This does not grant _new_ permissions. It simply generates a clean URL to send to colleagues who already have access via workspace roles or app permissions.
- **Specific People:** The most secure method. You must type in specific email addresses or Azure Active Directory security groups. The link will only work for those exact users.

**2. Settings (Permissions granted via the link)**

- **Allow recipients to share this report:** Grants the user the ability to forward the link or share the report with others. (Best practice is to uncheck this for sensitive data to maintain tight control).
- **Allow recipients to build content with the data associated with this report:** This grants **Build Permission** on the underlying semantic model. It allows the user to connect to the dataset via Excel (Analyze in Excel), build their own reports using this model, or export summarized data.

**3. How (Distribution Methods)**

- **Copy link:** Generates a standard URL to paste anywhere.
- **Mail:** Opens your default email client with the link and a generic message.
- **Teams:** Allows you to seamlessly share the report directly into a Microsoft Teams channel or direct message, where users can view the interactive report without leaving Teams.
- **PowerPoint:** Embeds the report as a "live" interactive slide within a PowerPoint presentation.

**4. Limits**

- **Pro License Requirement:** Sharing is a premium feature. To share a report, the sender **must** have a Power BI Pro (or Premium Per User) license. Additionally, the recipient **must also** have a Pro/PPU license to view it, _unless_ the workspace hosting the report is backed by a Premium Capacity (diamond icon).

**5. Manage Permissions**

- **Advanced (from Workspace):** If you click the three dots (...) and select "Manage Permissions" -> "Advanced," it takes you out of the report view and into the Workspace permission settings.
- **Why use Advanced?** This is where admins go to clean up access. You can see a detailed list of exactly who has direct access, who has access via links, and who has access via Apps. From here, you can manually revoke specific links, remove individuals, or strip away "Build" permissions without deleting the report.

# 9. Design and Build semantic models

## Creating semantic model

!image.png

!image.png

## Normal form (1st, 2nd, 3rd)

How to design data model

1. Snow flake schema: Fact → Dim → … → Dim (Link into other table)
2. Start schema: Fact → Multiple Dim tables (Descriptive)

Denormalize data

- 1st Normal Form (1NF)
  - Requirements
    - The values in each column must be atomic (indivisible)
    - Each value contains only a single value
  - Actions
    - Eliminate repeating groups in individual tables
    - Create a separate table for each set of related data
    - Identify each set of related data with **a primary key**
  - ex. Order Table
- 2nd Normal Form (2NF)
  - Requirements
    - Is in 1st Norm Form
    - Reduce repeating information
  - Actions
    - Create separate tables for values that apply to multiple records
    - Relate tables with **a foreign key**
  - ex. Product, Address, Person Table
- 3rd Normal form (3NF)
  - Requirements
    - Is in 2nd Normal Form
    - Values that are not part of a record’s key are to be removed from the table
  - Action
    - Remove fields that are not dependent on the key
  - ex. Subcategory, Category Table (from Product Table)

## Implementing a star schema for a semantic model

- **3NF (Third Normal Form - Highly Normalized):**
  - _Purpose:_ Good for **writing** data (OLTP - Online Transaction Processing).
  - _How it works:_ Data is split into many separate, smaller tables to completely eliminate duplicate data.
  - _Pros:_ If a product name changes, the database only has to update it in one single row.
  - _Cons for Power BI:_ Terrible for analytics. To read a simple sales report, the engine has to jump through 5 different table relationships, which is very slow.
- **"2NF" / Star Schema (Denormalized Dimensions):**
  - _Purpose:_ Good for **reading** data (OLAP - Online Analytical Processing).
  - _How it works:_ This is the gold standard for Power BI. You have a central **Fact** table (numbers/transactions) surrounded by flattened **Dimension** tables (descriptions/attributes).
  - _Pros:_ Highly efficient for relationships. It balances fast read times with manageable file sizes. Power BI's VertiPaq engine is specifically optimized to compress and read this exact structure.
- **1NF / Flat Table (One Big Table):**
  - _Purpose:_ Entire data in one massive spreadsheet-like table.
  - _How it works:_ No relationships are needed because every single detail (Customer Name, Product Color, Sales Amount) is repeated on every single row.
  - _Cons for Power BI:_ While DAX is easy to write, this creates massive file sizes and terrible memory performance due to the lack of columnar compression efficiency.

### Normalization vs. Denormalization in Power BI

- **Normalization (1NF → 2NF → 3NF):** The process of breaking large tables into smaller, related tables to reduce redundancy. (Usually done by database engineers before the data gets to you).
- **Denormalization (3NF → 2NF → 1NF):** The process of combining smaller tables into fewer, wider tables to speed up read performance. **(This is what Data Analysts do in Power Query).**

### The Power Query Merge (Snowflake to Star Schema)

- **The Snowflake Problem:** If your source database gives you three separate tables for `Product`, `SubCategory`, and `Category`, this forms a "Snowflake" schema. To filter a sale by Category, Power BI must travel: _Category → SubCategory → Product → Sales_. This chain degrades report performance.
- **The Star Schema Solution:** In Power Query, you should **Merge** these three tables into a single, wide `DimProduct` table.
- **The Result:** You end up with a single `Product` table that has a direct **One-to-Many (1:\*) relationship** with the `Sales` (Fact) table. Filtering becomes instant, DAX calculations become simpler, and the report renders much faster.

# 10. Calculation groups/items and field parameters

## Enabling the calculation groups preview feature

```jsx
OrderQuantity YTD = **TOTALYTD**(
  SUM(FactInternetSales[OrderQuantity]),
  **FactInternetSales[OrderDate].[Date]
)**

SalesAmount YTD = **TOTALYTD(**
  SUM(FactInternetSales[SalesAmount]),
  **FactInternetSales[OrderDate].[Date]
)**
```

Models / Calculation group

!image.png

## Creating our first calculation group and item

Auto Date - Disable & Create Date table

```jsx
DimDate = ADDCOLUMNS(
  CALENDARAUTO(),
  'CalendarYear',
  YEAR([Date]),
  'MonthNumber',
  MONTH([Date]),
  'MonthName',
  FORMAT([Date], 'MMMM'),
  'Quater',
  'Q' & QUARTER([Date]),
  'DayOfWeekNumber',
  WEEKDAY([Date]),
  'DayOfWeekName',
  FORMAT([Date], 'dddd'),
);
```

Calculation Group: Time Intelligence / Time Calculation / YTD

```jsx
YTD = CALCULATE(SELECTEDMEASURE(), DATESYTD(DimDate[Date]));
```

Metrics

- Rows: Date
- Columns: Time Calculation

!image.png

## Adding additional calculation items

Model / Calculation Group / Calculation Items

!image.png

```jsx
Total = SELECTEDMEASURE()

YTD = CALCULATE(SELECTEDMEASURE(), DATESYTD(DimDate[Date]))

// Previous Year
PY = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR(DimDate[Date]))

// Previous Year Year to Date
PY YTD = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR(DimDate[Date]), TimeIntelligence[Time Calculation] = "YTD")

// Year Over Year
YOY = SELECTEDMEASURE() - CALCULATE(SELECTEDMEASURE(), TimeIntelligence[Time Calculation] = "PY")

// Year Over Year%
// Calculate order by "Precedence"
YOY% = DIVIDE(
    CALCULATE(SELECTEDMEASURE(), TimeIntelligence[Time Calculation] = "YOY"),
    CALCULATE(SELECTEDMEASURE(), TimeIntelligence[Time Calculation] = "PY"),
    0
)
```

!image.png

## Using our calculation group, including dynamic strings

Add slicer to choose which calculation group items are presented.

!image.png

Matrics (Rows \* Cols) → Table (Rows) → Create measure for calculation group time

```jsx
SalesAmountYOY% = CALCULATE(
  [SumOfSalesAmount],
  TimeIntelligence[Time Calculation] = "YOY%"
)
```

!image.png

## Implement field parameters

Model / New Parameters / Fields

!image.png

Edit Parameters → Parameter Name & Value & Index

Calculation group → only explicit measure work

```jsx
/*
Field chooser = {
    ("SalesAmount", NAMEOF('FactInternetSales'[SalesAmount]), 0),
    ("OrderQuantity", NAMEOF('FactInternetSales'[OrderQuantity]), 1),
    ("ExtendedAmount", NAMEOF('FactInternetSales'[ExtendedAmount]), 2)
}
*/

Field chooser = {
    ("SalesAmount", NAMEOF('FactInternetSales'[SumOfSalesAmount]), 0),
    ("OrderQuantity", NAMEOF('FactInternetSales'[OrderQuantitySum]), 1),
    ("ExtendedAmount", NAMEOF('FactInternetSales'[SumExtendedAmount]), 2)
}

// SumOfSalesAmount = SUM(FactInternetSales[SalesAmount])
// OrderQuantitySum = SUM(FactInternetSales[OrderQuantity])
// SumExtendedAmount = SUM(FactInternetSales[ExtendedAmount])
```

# 11. DAX Functions

### Using DAX variables

```jsx
SalesAmountCalculation =
  (CALCULATE(
    SUM(FactInternetSales[SalesAmount]),
    DATESMTD(FactInternetSales[DueDate]),
  ) -
    SUM(FactInternetSales[SalesAmount])) /
  CALCULATE(
    SUM(FactInternetSales[SalesAmount]),
    DATESMTD(FactInternetSales[DueDate]),
  );
```

```jsx
SalesAmountCalculation =
VAR MyCalculation =
(
    CALCULATE(
      SUM(FactInternetSales[SalesAmount]),
      DATESMTD(FactInternetSales[DueDate])
    ) - SUM(FactInternetSales[SalesAmount])
)
/
(
   CALCULATE(
     SUM(FactInternetSales[SalesAmount]),
     DATESMTD(FactInternetSales[DueDate])
   )
)
RETURN MyCalculation
```

```jsx
SalesAmountCalculation =
VAR MonthToDate = CALCULATE(
  SUM(FactInternetSales[SalesAmount]),
  DATESMTD(FactInternetSales[DueDate])
)
VAR CurrentSales = SUM(FactInternetSales[SalesAmount]
RETURN (MonthToDate-CurrentSales)/MonthToDate
-- COMMENT
```

Benefits of using variables

- Readability
- Efficiency

## ROWNUMBER

```jsx
RowNumberColumn = ROWNUMBER(
  ALLSECTED(
    DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductSubcategoryName], ASC),
);
```

열이 화면에 없더라도 row number 은 유지 된다.

Order by 에 포함 되지 않은 열이 추가 되는 경우, NULL 로 값을 채운다.

```jsx
RowNumberColumn = ROWNUMBER(
  ALLSECTED(
    DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductSubcategoryName], ASC),
  PARTITIONBY(DimProduct[EnglishProductSubcategoryName]),
  // EnglishProductSubcategoryName에 따라 row number 새로 시작
);
```

## RANK

```jsx
RowNumberColumn = ROWNUMBER(
	ALLSECTED(
	  DimProduct[EnglishProductSubcategoryName],
	  DimProduct[EnglishProductCategoryName]
	),
  ORDERBY(DimProduct[EnglishProductSubcategoryName], ASC),
  PARTITIONBY(DimProduct[EnglishProductSubcategoryName]
)
```

```jsx
RowNumberColumn = ROWNUMBER(
  ALLSECTED(
    DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductCategoryName], ASC),
);
```

```jsx
RowNumberColumnDense = RANK(
  DENSE, // 1, 2, 2, ..., 3, 3, ...
  ALLSECTED(
    DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductCategoryName], ASC),
);

// ALLSECTED 에서 SubCategory 를 제외한 것과 동일하다.
RowNumberColumn = ROWNUMBER(
  ALLSECTED(
    // DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductCategoryName], ASC),
);

RowNumberColumnSkip = RANK(
  SKIP, // 1, 2, 2, ..., 14, 14, ...
  ALLSECTED(
    DimProduct[EnglishProductSubcategoryName],
    DimProduct[EnglishProductCategoryName],
  ),
  ORDERBY(DimProduct[EnglishProductCategoryName], ASC),
);
```

## INDEX

```jsx
SummaryTable = SUMMARIZECOLUMNS(
  DimDate[CalendarYear],
  DimProduct[ProductSubcategoryKey], // Category Name
  'SalesAmount',
  SUM(FactInternetSales[SalesAmount]),
);
```

```jsx
IndexCalculation = INDEX(
  2, // index your looking for
  ALL(SummaryTable[CalendarYear]), // relation, it should be unique value
  ORDERBY(SummaryTable[CalendarYear], ASC), // order by
);
```

## OFFSET

```jsx
TheOtherYearSales = CALCULATE(
  SUM(SummaryTable[SalesAmount]),
  OFFSET(-1, , ORDERBY(SummaryTable[CalendarYear]))
)
```

## WINDOW

Retrieves a range of rows within the specified partition, sorted by the specified order or on the axis specified.

```jsx
WINDOW ( <From> [, <FromType>], <To> [, <ToType>] [, <Relation>] [, <OrderBy>] [, <Blanks>] [, <PartitionBy>] [, <MatchBy>] [, <Reset>] )
```

```jsx
RunningTotal = CALCULATE(
  SUM(FactInternetSales[SalesAmount]), // calculation based on window
  WINDOW(0, ABS, 0, REL, ORDERBY(DimDate[CalendarYear])),
  // WINDOW(-1, REL, 0, REL, ORDERBY(DimDate[CalendarYear])) // Previous Row
);
```

# 12. Optimize enterprise scale semantic models

## Identify use cases for DAX Studio and Tabular Editor 2 (TE2), installing TE2

**Tabular Editor**

https://tabulareditor.com/

- v2: Free
- v3: Premium

https://github.com/TabularEditor/TabularEditor/blob/master/VersionHistory.md

**DAX Studio**

https://daxstudio.org/

**ALM Toolkit**

https://www.sqlbi.com/tools/alm-toolkit/

**Metadata Translator**

https://github.com/microsoft/metadata-translator

## Optimize a semantic model by using Tabular Editor 2 (BPA)

BPA = Best Practice Analyzer

Tools → Manage Best Practice Rules → Add → Enter Rule File URL

https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json

Can do

- Copy & Paste & Edit rules
- URL: Read only
- Script: C#

## Implement object-level security (OLS)

Table or Columns

Create Role

!image.png

Role → Table

!image.png

Table → Role

!image.png

You can’t choose object level security (visuals)

- question and answers
- quick insights
- smart narrative

## **Improve DAX performance by using DAX Studio**

https://daxstudio.org/

## Using the VertiPaq Analyzer

Part of DAX studio

more unique value will cost more memory (high cardinality)

- if col doesn’t need it, then remove
- split date and time (sperate column to remove high cardinality)

## ALM Toolkit

Application Lifecycle Management

source version (power bi desktop) → target version (power bi service)

target version (power bi service, development) → target version (power bi service, production)

check version, and choose how to deal with.

git-like interface

## **Translating metadata**

Localization

Automatic translation

# 13. Relationships and performance improvements

## Resolve many-to-many relationships - Joint Bank Accounts

Joining Key - BankAcount - Many to Many Cardinality - Cross filter direction (Both)

Single

- Dim → Fact: Fine
- Fact → Dim: only work with specific col (ex. Currency in example)

## Implement dynamic format strings

Preview features - Dynamic format strings for measures

only work for explicit measures

```jsx
// explicit measures
TransactionTotal = SUM(MtoMTransactions[Transaction]);
```

Format - Dynamic

```jsx
IF((MtoMTransactions[Currency] = 'USD'), '$#,##0', '#,##0');
```

```jsx
IF(MIN(MtoMTransactions[Currency])<>MAX(MtoMTransactions[Currency]), "Multiple",
  IF(MIN(MtoMTransactions[Currency])="USD", "$#,##0",
    IF(MIN(MtoMTransactions[Currency])="EUR", "€#,##0",
      IF(MIN(MtoMTransactions[Currency])="GBP",
        "£#,##0",
        (MIN(MtoMTransactions[Currency]) & " #,##0" // fallback
      )
    )
  )
)
```

## Resolve many-to-many relationships - creating a Bridge Table

Actual → Bridge → Target

Actual

| Actual | **Country** | Location |
| ------ | ----------- | -------- |

Target

| **Country** | Target | Type |
| ----------- | ------ | ---- |

Bridge

- Append Query - New
- Keep Country
- Remove Duplicate

| Country |
| ------- |

## The Optimize menu

Performance analyzer

- Paused visual
- Refresh visual
- Optimization presets
  - default: interactive
  - query reduction: inactive highlight, apply button
- Apply all slicers button

Insert → Button → Clear all slicers

# 14. Manage the analytics development lifecycle

## Creating a Premium Per User (PPU) workspace and Azure DevOps repos

version control for workspace

- Git - Azure DevOps repos
- need premium workspace

## **Implement version control for a workspace**

workspace setting → git integration → select organization & project & repository & branch & folder

status: synced, uncommitted

source control → select item → commit

permission: at least contributor

## Create and manage a Power BI Desktop project (.pbip)

.pbix

- default extension
- encoded for machine use

.pbip

- project file
- encoded for json file
- text readable

# 15. Other Power BI Analytics topics

Here are your organized, structured notes from the video transcript on creating and managing Composite Models, formatted for quick reference with no icons in the headers.

## Introduction to Composite Models

A **Composite Model** is a Power BI data model that combines multiple data connections from different source groups, where at least one of those connections uses **DirectQuery** (instead of everything being imported).

- **Use Case:** Ideal when dealing with a massive fact table that is too large to import (kept on DirectQuery) alongside smaller, manageable dimension tables (set to Import).
- **Benefits:** Breaks the historical limitation where a DirectQuery report could only connect to a single data source. It also makes establishing many-to-many relationships across different sources much easier.
- **Storage Status:** The Power BI Desktop status bar will display **"Mixed Storage Mode"** when a composite model is active.

## Distinguishing Storage Modes in Model View

You can visually identify how data is being stored and accessed by looking at the tables in the Model View:

- **DirectQuery Tables:** Formatted with a solid blue/colored bar across the top of the table diagram.
- **Import Tables:** Formatted with no colored bar at the top.

## Security & Performance Considerations

### 1. Data Privacy Cross-Source Risk

When combining independent data sources (e.g., an Azure SQL Database and a local Excel workbook), Power BI will trigger a potential security risk warning.

- **The Risk:** Queries sent to one data source may include literal data values retrieved from the other data source.
- **Best Practice:** Only combine sources if you fully trust the administrators and owners of both data systems with the combined data.

### 2. Performance Implications & Literal Values

Cross-source filtering can sometimes negatively impact report performance.

- When you filter a visual using a dimension from an Import table (e.g., filtering by specific "Heads of Department"), Power BI must pass those explicit values directly into the DirectQuery SQL statement.
- If you extract the underlying query using the **Performance Analyzer** and view it in **DAX Studio**, you will see these values hardcoded (literal values) directly into the query clause.
- If a filter contains dozens of items, the query can become massive, run slowly, or be split into multiple sub-queries, degrading execution speeds.

## Advanced Concept: Limited Relationships

When you build a relationship that spans across two completely different data source groups, Power BI classifies this as a **Limited Relationship** (as opposed to a regular relationship).

- **Visual Indicator:** In the Model View, the relationship line will display **brackets or parentheses `( )`** around the cardinality numbers.
- **Behavioral Restrictions:**
  - **Join Behavior:** Evaluation behaves strictly like an `INNER JOIN` in SQL terms. Data is only processed where rows explicitly match across both sides; it does not support full left/right unmatched row retention.
  - **DAX Restriction:** You cannot use the `RELATED()` DAX function to pull values from the "one" side of a limited relationship to the "many" side.

## Power BI Service Configuration Settings

To successfully host and run a composite model on the Power BI Service, specific administrative configurations must be turned on in the **Admin Portal > Tenant Settings**:

### Integration Settings

- **Allow XMLA Endpoints and analyze in Excel with on-premises semantic models:** Absolutely required to support external DirectQuery pass-through connections.
- **Users can work with Power BI semantic models in Excel using a live connection:** Necessary for handling live relational connections.
- **Allow DirectQuery connections to Power BI semantic models:** Must be explicitly enabled.

### Capacity & Workload Settings

- For Premium or Premium Per User (PPU) workspaces, ensure the **XMLA Endpoint workload setting** is actively configured to either **Read Only** or **Read Write**.

# Design and build a large semantic model storage format

pro workspace: 10 gb → premium: XMLA endpoint, 10+ gb, 8 m rows, (PPU)

large semantic model - you can download

semantic model < capacity size 50%

large semantic model should stay in same region

XMLA right operation

# Perform impact analysis of downstream dependencies

Workspace → Lineage view → to check how semantic data impact on difference target source (ex. data warehouse, report, dashboard)

# Deploy and manage semantic models by using the XMLA endpoint

## Introduction to XMLA Endpoints

**XMLA (XML for Analysis)** allows you to connect to, manage, and query Power BI semantic models directly from the Power BI Service.

- **Supported Workspace Types:** Power BI Premium, Premium Per User (PPU), and Power BI Embedded.
- **Connection Link Format:** `PowerBI://API.powerbi.com/v1.0/[tenant_name]/[workspace_name]`
- **Initial Catalog:** If using a tool like SQL Server Profiler that requires an initial catalog/database, use the connection link **plus** the specific semantic model name.

## Configuration & Setup

### 1. Enable Tenant Settings (Admin Portal)

By default, these settings are typically enabled, but they must be verified in the Admin Portal under **Tenant Settings > Integration Settings**:

- Enable **"Allow XMLA endpoints and analyze in Excel with on-premises semantic models"**.
- To allow Excel usage, also ensure **"Users can work with semantic models in Excel using a live connection"** is enabled.

### 2. Change Read-Only to Read-Write

By default, XMLA endpoints are **Read Only** (restricted to querying data, metadata, events, and schemas). To perform management, governance, debugging, and advanced modeling, you must switch to **Read-Write**:

- **For Premium Capacity:** Go to _Admin Portal > Capacity Settings > Power BI Premium tab > [Your Capacity Name] > Workloads_ $\rightarrow$ Change XMLA Endpoint to **Read Write**.
- **For Premium Per User (PPU):** Go to _Workspace Settings > Premium Per User tab > Semantic Model Workload Settings_ $\rightarrow$ Change XMLA Endpoint to **Read Write**.

> ⚠️ **Critical Warning:** If you write/commit changes to a semantic model using write-access via the XMLA endpoint, you will **no longer be able to download** that model as a `.pbix` file from the Power BI Service.

## Supported Client Applications

| **Connection Mode**                         | **Allowed Operations**                             | **Supported Tools**                                                                                                                    |
| ------------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Read Only**                               | Query operations only                              | Microsoft Excel, Power BI Report Builder, DAX Studio                                                                                   |
| **Read Write**                              | Full management, scripting, and deployment         | Visual Studio (with Analysis Services Projects), SQL Server Profiler (v18.9+), Analysis Services Deployment Wizard, PowerShell cmdlets |
| **Both (Read/Write determines capability)** | Queries (Read) OR Scripting/Metadata edits (Write) | SQL Server Management Studio (SSMS v18.9+), Tabular Editor, ALM Toolkit                                                                |

## How to Connect

### In Power BI Desktop

- **Method 1:** Go to _Home > Get Data > Power BI Semantic Model_.
- **Method 2:** Go to _Get Data > Analysis Services_. Paste the XMLA endpoint workspace name, select **DirectQuery** (do not use Import), sign in with your Microsoft account, and select your model.

### In External Tools

- **DAX Studio:** Go to _File_, paste the workspace URL into the **Tabular Server** field, and connect.
- **Tabular Editor:** Go to _File > Open from Database_, paste the endpoint into the **Server** field, and log in using Microsoft Entra ID (Azure AD).
- **SQL Server Management Studio (SSMS):** Connect to **Analysis Services**, paste the endpoint in the Server Name field, change authentication to **Microsoft Entra multifactor authentication**, and log in.

## Data Refresh: Processing Modes in SSMS

When connected via SSMS, you can right-click an individual database, table, or partition to process it. The available processing modes include:

- **Process Default:** Fully processes the object by loading data for empty tables/partitions and recalculating hierarchies, calculated columns, and relationships.
- **Process Full:** Completely drops all existing data and structural schemas, then completely rebuilds the object. _Requires the most resources; necessary for structural changes._
- **Process Data:** Loads data only. It does **not** rebuild hierarchies/relationships or recalculate measures and calculated columns.
- **Process Clear:** Removes all data from the database, table, or partition.
- **Process Defrag:** Defragments and reorders table indexes.
- **Process Recal:** (Semantic models only) Updates and recalculates hierarchies, relationships, and calculated columns without reloading underlying data.
- **Process Add:** (Partitions only) Incrementally updates partitions with new data, helping implement incremental refreshes.

## Querying the Model via DAX

You can run DAX queries directly against your hosted semantic models.

- **In SSMS:** Right-click the semantic model, navigate to **New Query**, and explicitly choose **DAX** (avoiding clicking generic "New Query" which defaults to MDX).
- **In DAX Studio:** Write your evaluation statement directly.

**Example Query:**

```
EVALUATE DIM Product
```

# Creating and updating Power BI template (.pbit) files

## Introduction to Power BI Templates

A Power BI template file (**`.pbit`**) is a reusable asset designed to help you start fresh with different data while maintaining your existing report structure.

- **What it contains:** Report pages, visual layouts, data model definitions (schemas, relationships, measures), and query definitions (including query parameters).
- **What it excludes:** **Templates do not contain any data.** This makes the file size significantly smaller and safer for sharing sensitive structures without sharing the actual data.

## How to Create a Template File

To export your current Power BI report as a template, follow these steps:

1. Navigate to **File > Export > Power BI template**.
2. Enter a **Description** if desired (e.g., _"This is my fact internet sales model"_), then click **OK**.
3. Choose a save location. Notice the file type defaults to **Power BI template file (`.pbit`)**.
4. Click **Save**.

## How to Open and Use a Template File

Because template files contain no data, opening one will prompt you to connect to a data source to populate the visuals.

### Methods to Open a Template:

- **Method 1:** Open Power BI Desktop, go to **Open > Browse This Device**, select the `.pbit` file, and click **OK**.
- **Method 2:** Double-click the `.pbit` file directly within Windows Explorer.
- **Method 3:** Go to **File > Import > Power BI template** (Note: This requires you to already have a Power BI file session open).

### Loading the Data:

- Upon opening, Power BI will warn you that some tables have incomplete or missing data, and calculated objects may need a manual refresh.
- If your template includes **Query Parameters**, you will be prompted to enter the required values or specify new file locations for your data sources.
- Click **Refresh** and provide the necessary credentials to pull the new data into the report structure.

# Creating and updating Power BI data source (.pbids) files.

## Introduction to PBIDS Files

A Power BI Data Source file (**`.pbids`**) is a lightweight file with a JSON structure that points directly to a single, specific Power BI data source.

- **Purpose:** It streamlines the connection process for report creators by pre-configuring the data source path, meaning users don't have to manually look up server names or file paths.
- **Security:** **PBIDS files do not contain authentication credentials** (such as usernames or passwords). Users must provide their own credentials when they connect.
- **Limitation:** You cannot generate a PBIDS file if columns within the underlying data source are encrypted; attempting to do so will result in an error.

## How to Create a PBIDS File

To export a data source connection from Power BI Desktop into a reusable `.pbids` file:

1. Navigate to **File > Options and settings > Data source settings**.
2. Select the specific data source you want to share.
3. Click **Export PBIDS**.
4. Choose your save location and save the file.

## Under the Hood: The JSON Structure

When opened in a text editor like Notepad, a `.pbids` file reveals a structured JSON schema outlining the connection details:

- **Connection Path:** It explicitly lists details like the server, database, or file path.
- **Escape Characters:** File paths within the JSON will use double backslashes (`\\`). This is because a single backslash acts as an escape character in JSON formatting (e.g., `\n` represents a new line), so a double backslash is required to specify a literal, standard backslash.

## How to Open and Use a PBIDS File

Unlike standard Power BI files, you cannot open a `.pbids` file using the traditional "File > Open" menu inside Power BI Desktop.

### Connection Workflow:

1. Locate the `.pbids` file in **Windows Explorer**.
2. **Double-click** the file. This automatically launches Power BI Desktop and initializes the pre-configured connection.
3. If prompted, enter your personal **credentials** (username/password) to authenticate.
4. Select the database, tables, or connection mode (if they weren't explicitly hardcoded into the PBIDS file structure).
5. Load or transform the data via Power Query as you normally would to begin building your model.

# Creating and updating shared semantic models

## Introduction to Shared Semantic Models

Sharing a semantic model allows other users in your organization to build their own reports, modify the model, or further share the asset without duplicating the underlying data source.

- **Where to initiate sharing:**
  - Directly from the semantic model's detail page in the Power BI Service via the top menu.
  - Within the **OneLake data hub** by clicking the ellipsis (`...`) next to the specific semantic model.

## Configuring Share Permissions

When you share a semantic model, you can explicitly grant or restrict the following capabilities via checkboxes in the sharing dialog box:

- **Modify:** Allows recipients to edit the semantic model itself.
- **Reshare:** Allows recipients to share the semantic model with other users.
- **Build:** Allows recipients to build new content (such as their own independent reports) using the data associated with the model.
- **Email Notification:** Includes an option to send an automated email to the recipient, along with an optional custom message.

## Managing and Removing Permissions

If you need to view, adjust, or revoke user access, Power BI provides multiple routes to the permission management interface.

### Accessing the Manage Permissions Page

You can reach the primary permission management screen through several areas:

- **From the Workspace:** Click the ellipsis (`...`) next to the semantic model → Select **Manage permissions**.
- **From the Semantic Model Page:** Go to **File > Manage permissions**.
- **From the OneLake Data Hub:** Click the ellipsis (`...`) next to the model → Select **Manage permissions**.

### Direct Access Pane vs. Advanced Settings

- If you go through **Share > Ellipsis (`...`) > Manage permissions**, it opens a simplified side pane. This pane allows you to add people to direct access, but it does **not** allow you to remove them.
- To revoke access, you must click **Advanced** within that pane to open the full management page.

### Modifying or Revoking Access

On the advanced management page, clicking the ellipsis (`...`) next to a user's name allows the owner to:

- Remove the ability to reshare.
- Remove the ability to build reports.
- Remove the ability to modify the model.
- **Remove access entirely** to the semantic model.
- View related reports built on top of the model (if you have the required permissions).

## Discovering and Connecting to a Shared Semantic Model

Once a semantic model is shared, other users can connect to it and start building reports using either the web service or the desktop application.

### In Power BI Service:

1. Navigate to the **OneLake data hub**.
2. Locate and click on the shared semantic model.
3. Select the option to create a new report.

### In Power BI Desktop:

- **Method 1:** Go to **Home > OneLake data hub > Power BI semantic models**, select your model, and connect.
- **Method 2:** Go to **Home > Get Data > Power BI semantic models**, select the desired model from the list, and click **Connect**.

# Create an Azure SQL Database, and Design and build a composite model

## Introduction to Composite Models

A **Composite Model** is a Power BI data model that combines multiple data connections from different source groups, where at least one of those connections uses **DirectQuery** (instead of everything being imported).

- **Use Case:** Ideal when dealing with a massive fact table that is too large to import (kept on DirectQuery) alongside smaller, manageable dimension tables (set to Import).
- **Benefits:** Breaks the historical limitation where a DirectQuery report could only connect to a single data source. It also makes establishing many-to-many relationships across different sources much easier.
- **Storage Status:** The Power BI Desktop status bar will display **"Mixed Storage Mode"** when a composite model is active.

## Distinguishing Storage Modes in Model View

You can visually identify how data is being stored and accessed by looking at the tables in the Model View:

- **DirectQuery Tables:** Formatted with a solid blue/colored bar across the top of the table diagram.
- **Import Tables:** Formatted with no colored bar at the top.

## Security & Performance Considerations

### 1. Data Privacy Cross-Source Risk

When combining independent data sources (e.g., an Azure SQL Database and a local Excel workbook), Power BI will trigger a potential security risk warning.

- **The Risk:** Queries sent to one data source may include literal data values retrieved from the other data source.
- **Best Practice:** Only combine sources if you fully trust the administrators and owners of both data systems with the combined data.

### 2. Performance Implications & Literal Values

Cross-source filtering can sometimes negatively impact report performance.

- When you filter a visual using a dimension from an Import table (e.g., filtering by specific "Heads of Department"), Power BI must pass those explicit values directly into the DirectQuery SQL statement.
- If you extract the underlying query using the **Performance Analyzer** and view it in **DAX Studio**, you will see these values hardcoded (literal values) directly into the query clause.
- If a filter contains dozens of items, the query can become massive, run slowly, or be split into multiple sub-queries, degrading execution speeds.

## Advanced Concept: Limited Relationships

When you build a relationship that spans across two completely different data source groups, Power BI classifies this as a **Limited Relationship** (as opposed to a regular relationship).

- **Visual Indicator:** In the Model View, the relationship line will display **brackets or parentheses `( )`** around the cardinality numbers.
- **Behavioral Restrictions:**
  - **Join Behavior:** Evaluation behaves strictly like an `INNER JOIN` in SQL terms. Data is only processed where rows explicitly match across both sides; it does not support full left/right unmatched row retention.
  - **DAX Restriction:** You cannot use the `RELATED()` DAX function to pull values from the "one" side of a limited relationship to the "many" side.

## Power BI Service Configuration Settings

To successfully host and run a composite model on the Power BI Service, specific administrative configurations must be turned on in the **Admin Portal > Tenant Settings**:

### Integration Settings

- **Allow XMLA Endpoints and analyze in Excel with on-premises semantic models:** Absolutely required to support external DirectQuery pass-through connections.
- **Users can work with Power BI semantic models in Excel using a live connection:** Necessary for handling live relational connections.
- **Allow DirectQuery connections to Power BI semantic models:** Must be explicitly enabled.

### Capacity & Workload Settings

- For Premium or Premium Per User (PPU) workspaces, ensure the **XMLA Endpoint workload setting** is actively configured to either **Read Only** or **Read Write**.

# Improving DAX performance by adding aggregations

## Introduction to Aggregations

Pre-calculated **aggregations** are used to significantly speed up data retrieval from massive **DirectQuery** datasets. Instead of querying millions of raw rows in a remote database for every visual interaction, Power BI intercepts the query and diverts it to a smaller, pre-aggregated local table.

- **The Problem:** DirectQuery tables must query the live data source every time a visual changes. For large datasets, this can take seconds or minutes instead of milliseconds.
- **The Solution:** Create a smaller, summarized table, change its storage mode to **Import**, and map it to the large detailed table using **Manage aggregations**. Power BI automatically chooses the fastest path.

## Storage Modes & Performance Comparison

The video demonstrates how drastically performance changes depending on the storage mode configuration, measured via the **Performance Analyzer**:

| **Configuration**                                                      | **Query Type**                     | **Observed Performance**                                |
| ---------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------- |
| **Detailed Table Only** (DirectQuery)                                  | Live Database Query                | ~94 milliseconds                                        |
| **Aggregated Table Only** (DirectQuery)                                | Live Database Query                | ~87 milliseconds                                        |
| **Aggregated Table Only** (Import)                                     | Local In-Memory Cache              | **~4 milliseconds**                                     |
| **Detailed Table + Aggregated Import Table** (Mapped via Aggregations) | Local Cache (Hit) / Live DB (Miss) | **~5 milliseconds** (without hitting the live database) |

## Step-by-Step Configuration

### 1. Create the Aggregation Table in Power Query

1. In the Power Query Editor, **Duplicate** your detailed DirectQuery table.
2. Select only the columns you need to group by and aggregate (e.g., _Color_ and _List Price_), and remove all other columns.
3. Click **Group By** to define your groupings and aggregations (e.g., Group by _Color_, aggregate using `SUM` of _List Price_).
4. **Critical Data Type Check:** The data type of the aggregated column in your new table must _exactly_ match the data type of the column in the original detailed table (e.g., both must be _Fixed Decimal Number_). If they don't match, you won't be able to map them later.
5. Click **Close & Apply**.

### 2. Change Storage Mode to Import

1. In the Power BI Desktop Model View, select your new aggregated table.
2. Expand the **Advanced** section in the Properties pane.
3. Change the **Storage Mode** from DirectQuery to **Import** (Note: This is an irreversible operation).

### 3. Set Up Manage Aggregations

1. Right-click the aggregated Import table and select **Manage aggregations**.
2. Map the aggregation columns to the detailed table columns:
   - **GroupBy Columns:** Set Summarization to _GroupBy_, select your detailed table, and select the matching base column.
   - **Measure Columns:** Set Summarization to your calculation type (e.g., _Sum_), select your detailed table, and select the base column to be calculated.
3. Click **Apply All**.

> 💡 **Note:** Once applied, Power BI automatically **hides** the aggregation table from the Report View. It can now only be viewed or edited from the **Model View**.

## Handling Averages (Sum and Count)

The _Manage Aggregations_ user interface does not feature a direct option for an `AVERAGE` summarization.

To overcome this constraint, configure your aggregation table to include both a **Sum** column and a **Count** column for that field. Power BI is intelligent enough to background-calculate an average ($Average = \frac{Sum}{Count}$) using your imported aggregation table without needing to trigger a live DirectQuery.

## Rules and Best Practices for Aggregations

- **Detail Table Requirement:** The detailed base table must remain in **DirectQuery** storage mode.
- **Aggregated Table Requirement:** For the best performance gains, the aggregation table should be set to **Import** mode.
- **Dual Storage Mode for Dimensions:** Any dimension tables that connect to both the Import aggregation table and the DirectQuery detail table should typically be set to **Dual** storage mode. This allows them to act seamlessly as local tables or remote tables depending on the query path.
- **No Duplications:** You cannot map multiple aggregations that share the exact same combination of summarization type, detail table, and detail column.
- **Row-Level Security (RLS) Exception:** Aggregation tables are bypassed if a user has read-only access limited by certain Row-Level Security rules. In those cases, Power BI safely defaults back to querying the slower detailed table directly to ensure security integrity.
- **Composite Model Constraints:** Be cautious when using limited relationships across a composite model, as they can sometimes prevent the aggregation engine from triggering correctly.

# Quiz

1. What is the maximum size for semantic models that **do not** use large format semantic models?
   **10 GB**
2. Which of the following applications require a read-write XMLA endpoint?
   **SQL Server Profiler**
3. Which of the following does a Power BI template (.pbit) file not include?
   **Data**
4. How do you create a .pbids (Power BI Data Source) file in Power BI Desktop?
   **Go to File - Options and settings - Data source settings.**
5. Which of the following can you NOT directly set up in an aggregation table?
   **Average**

**Semantic model connectivity with the XMLA endpoint**

https://learn.microsoft.com/en-us/fabric/enterprise/powerbi/service-premium-connect-tools

# 16. Advanced Power BI practice test

Question 1:

**I have a table called FactInternetSales. It is fairly slow for using in visuals.
To solve this, I want to set out an aggregation table called AggregationTable.
Which of the two statements is normally the case for aggregation tables?**

Select all that apply.

A.

- **The FactInternetSales table must use the DirectQuery storage mode.**
  - This is because it allows Power BI to query the data in real-time. This is important for ensuring that the aggregated data is always up-to-date and reflects the latest changes in the underlying data source.
- **AggregationTable would normally use the Import storage mode.**
  - This is because it pre-aggregates and stores the summarized data in the Power BI model. This helps improve query performance and reduces the need to query the underlying data source repeatedly.

Please look at this formula:

```jsx
RankNumberColumn = RANK(
  DENSE,
  ALLSELECTED(tblTable[Color], tblTable[MeasureCol]),
  ORDERBY(tblTable[MeasureCol]),
);
```

Please look at the below result:

```jsx
Color   MeasureCol   RankNumberColumn
Black   3,000
Blue    5,000
Green   7,000
Red     5,000
```

What answer does the formula give?

A. **1, 2, 3, 2**

- 1, 2, 4, 2 would be correct if the formula used `RANK(SKIP`.
- 1, 2, 3, 4 would be correct if the formula used `ROWNUMBER` using `ORDERBY(tblTable[Color]).`
- 1, 2, 4, 3 would be correct if the formula used `ROWNUMBER` using `ORDERBY(tblTable[MeasureCol]).`

Question 3:

**Which of these is NOT a "deployment rule" for a deployment solution?**

A. **Default warehouse rule**

Deployment rules typically involve defining how data is sourced, transformed, and stored, whereas a default warehouse rule is not a standard deployment rule in this scenario.

- "**Default lakehouse rule**". It involves defining how data is stored in a lakehouse architecture, which is a combination of data lake and data warehouse technologies.
- "**Data source rule**". It involves defining how data is sourced, connected, and integrated into the analytics solution.
- "**Parameter rule**". It involves defining how parameters are used to customize and configure the deployment process for different environments or scenarios.

Question 4:

**In order to reduce the number of DAX queries, I want to add a "Clear all slicers" button.
Where do I go to add this?**

A. **Insert - Buttons - Clear all slicers**

Question 5:

**I am using a calculation group in a matrix.
It has the following calculation items: YTD, QTD, MTD, YOY and YOY%.
The YOY% should be expressed as a percentage, and the others as a number.
How should I start?**

A. **In the Model view, go to the Data pane - Model tab.**

**Click on the YOY% calculation item, change "Dynamic format string" to Yes. Click Edit and change the Format to "0.00%".**

Question 6:

**I want to save my Power BI file in a way that I can use source control and Tabular Model Scripting Language (TMSL).
What file type do I save it in?**

A. **.pbip file**

- The .pbip (Power BI Desktop project) file format is in a text-readable format, and allows you to use source control and Tabular Model Scripting Language (TMSL). This file format includes both the Power BI report and the data model, making it suitable for version control and scripting purposes.
- .pbix file. This is the standard Power BI report structure, which cannot be easily used to use source control or TMSL.
- .pbit file. This is a Power BI template file.
- .pbids file. This is a Power BI data source file, and points towards a data source.

Question 7:

I want to compare two stages in a deployment pipeline.

What are the minimum permission(s) that I need?

A, **Pipeline Admin and Workspace Contributor**

- the pipeline configuration and settings.
- Additionally, you need the Workspace Contributor role to have access to the workspace where the pipeline and its artifacts are stored.

Question 8:

\*\*I am using DAX Studio, and have just run a query with Server Timings on.
Please look at the below image.

Which of the following shows the time spent processing data, but not retrieving data?\*\*

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2024-05-14_09-28-49-833245a521794ea538c18af0a75b5abe.png)

A. **FE: 154 ms**

- FE stands for the Formula Engine. It can process data, but cannot retrieve data from the tables. It is single-thread.
- SE: 3 ms. SE stands for Storage Engine. It retrieves data. It is multi-threaded.
- Total: 157 ms. The Total is a combination of FE and SE.

Question 9:

**I have two tables, Employee and Salary. The Salary table has a "Pay" column which totals $1,000,000.
Currently I have a table visual showing details from the Employee table and the Pay from the Salary table.
However, the table visual has been filtered, so it is only showing the HR department, and a total of $200,000.
I want to create a calculation based on this visual total of $200,000. What formula do I use?**

A. **CALCULATE(SUM(Salary[Pay]), ALLSELECTED())**

- Using ALLSELECTED removes all row and column filters, but keeps explicit filters and contexts. With nothing in the brackets/paratheses, this calculates the visual total of $200,000.
- `CALCULATE(SUM(Salary[Pay]), ALL())` . This remove all filters, including those in the table visual. This would result in a figure of $1,000,000, not $200,000.
- `CALCULATE(SUM(Salary[Pay]), ALLEXCEPT())` . The ALLEXCEPT function requires at least two arguments - the table and the column(s) to be removed.
- `CALCULATE(SUM(Salary[Pay]), ALLVISUAL())` . There is no such function as ALLVISUAL.

Question 10:

**I have a snowflake schema - a table related to another table which is related to another table and so on.
I want to convert it into a star schema - which is based on a fact table directly related to dimension tables.
Which of these DAX functions would be the most appropriate to convert the snowflake schema to a star schema?**

A. **RELATED**

- This can be used to relationships which have a One-To-Many cardinality. It can retrieve data from the "One" side of the relationship.
- `FILTER`. This can be used to reduce the number of rows in a dataset.
- `SAMEDATELASTPERIOD`. This is a time intelligence function which looks at the parallel period a year ago.
- `SUMX`. This can be used to calculate the total of a table, based on a calculation or a field.

Question 11:

What is the first argument in the INDEX formula?

A. **The row number**

- INDEX(row_number, table, order_by)

Question 12:

**I want to allow the end user to change which measure or dimension is being used in your visuals.
I need to use a slicer. What other feature do I use?**

A. **A field parameter.**

- A calculation group and items allow you to create similar calculations for multiple measures.
- Dynamic strings are used in calculation items to create a different format, and for a measure to allow multiple formats for the same measure.
- Aggregations are used to more rapidly calculate sum or count of measures.

Question 13:

**I want to use an external app to change the properties of multiple columns in a semantic model at once.
What should I use?**

A. **Tabular Editor 2**

- It provides a user-friendly interface to modify column properties, such as data types, formatting, and relationships, across multiple tables simultaneously.
- DAX Studio is primarily used for writing and analyzing DAX queries in Power BI. While it is a powerful tool for DAX development, it is not designed for changing column properties in bulk.
- VertiPaq Analyzer is a tool used for analyzing and optimizing the data model in Power BI, particularly focusing on the VertiPaq engine. It does not have the capability to change column properties across multiple tables.
- ALM Toolkit (Application Lifecycle Management Toolkit) is a tool designed for managing the application lifecycle of Power BI projects, including version control, deployment, and collaboration. It does not have the functionality to change column properties in bulk.

Question 14:

**I have an Activity table which includes the following columns: Cost and Duration.
I want to create a measure which calculate the total costs.
What DAX formula should I use?**

A. **`TotalCost = SUMX([Activity], [Cost] * [Duration])`**

- SUM is used when you are totalling a field - for example, `SUM([CostPerActivity])`. However, it cannot be used for a calculation.
- SUMX needs two arguments - the table, and the calculation.

Question 15:

**I want to create a relationship between two tables: Employee and Payroll. One employee can be paid multiple times (in different months). As shown below, I create the relationship, with Payroll as the first table and Employee as the second table.**

**What cardinality should I use?**

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2023-08-30_10-07-16-34c3b7a32d7427ba74aeb987ae88bf6b.png)

What cardinality should I use?

A. **Many to one (\*:1)**

Question 16:

**Which of these external apps should I use to implement object-level security (OLS)?**

A. **Tabular Editor 2**

- You create roles, and then click on a table and change the Table Permissions, or click on a column and change the Object Level Security.
- It allows you to define and manage roles and permissions at the object level, such as tables, columns, and measures, providing fine-grained control over access to data and functionality.
- **DAX Studio** is a tool primarily used for writing and analyzing DAX queries in Power BI. While it is a valuable tool for optimizing DAX calculations and queries, it is not specifically designed for implementing object-level security (OLS) in Power BI.
- **VertiPaq Analyzer** is a tool used for analyzing and optimizing the VertiPaq engine, which is the in-memory data storage technology used by Power BI. While it can help improve performance and efficiency of Power BI models, it is not the tool for implementing object-level security (OLS).
- **ALM Toolkit (Application Lifecycle Management Toolkit)** is a tool designed to help manage the application lifecycle of Power BI projects, including version control, deployment, and collaboration. While it is essential for managing the development and deployment process, it is not the tool for implementing object-level security (OLS) in Power BI.

Question 17:

**I want to compare a source and target semantic model, and find the differences between them.
Which external app should I use?**

A. **ALM Toolkit**

- It can compare a source and target semantic model and identify the differences between them. It provides features for managing and deploying Analysis Services models, including the ability to compare models and highlight variances.
- **Tabular Editor 2** - It is a tool primarily used for editing and managing individual Tabular models in Power BI and Analysis Services.
- **DAX Studio** - It is a tool used for writing, executing, and analyzing DAX queries in Power BI and Analysis Services models.
- **VertiPaq Analyzer** - It is a tool used for analyzing and optimizing the data model in Power BI and Analysis Services.

Question 18:

**I want to create a calculated column in DAX which retrieves the previous row.
Which DAX function do I use?**

A. **OFFSET**

- The OFFSET function in DAX is used to retrieve a value from a previous or subsequent row in a table. It allows you to specify the number of rows to offset and the column to retrieve the value from, making it suitable for creating a calculated column that retrieves the previous row.
- The RANK function in DAX is used to assign a rank to each row in a table based on a specified expression
- The ROWNUMBER function in DAX is used to return the number of the current row in a table.
- The INDEX function in DAX is used to retrieve a value from a specific row and column in a table.

Question 19:

**Which of the following means that you can no longer download a semantic model over 50 Gigabytes to Power BI Desktop?**

A. **Use "Large dataset storage format"**

- It means that the semantic model is stored in a format that is optimized for large datasets and cannot be downloaded to Power BI Desktop if it is over 50 Gigabytes.

Question 20:

**Which of the following can you NOT use Git integration in a Power BI Workspace?**

A. **Dashboards**

- Git integration in Power BI Workspace is primarily used for version control and collaboration on Power BI reports and datasets. Dashboards in Power BI are visual representations of data from reports and cannot be directly version controlled using Git integration.
- The other options (Paginated reports, semantic models and reports) can be version controlled and managed using Git integration. This allows users to track changes, manage versions, and collaborate on data models effectively.

Question 21:

**I have a dotted line between two tables in a model.
What does this dotted line represent?**

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2024-04-26_12-45-18-2f45790d7a42762e908f6e7b139096e2.png)

What does this dotted line represent?

A. **An inactive relationship**

Question 22:

**I want to create a running total of SalesAmount in DAX. It should be calculated from the beginning of the visualization to the current row, and should be ordered by CalendarYear.
I am using this code:**

```jsx
RunningTotal = CALCULATE(
  SUM(FactInternetSales[SalesAmount]),
  WINDOW(___________, ORDERBY(DimDate[CalendarYear])),
);
```

What should I use to create a window from the beginning to the current row?

A. **`WINDOW(1, ABS, 0, REL, ORDERBY(DimDate[CalendarYear])`**

- 1, ABS (which is short for "absolute") returns the first row.
- 0, REL (which is short for "relative") returns the current row.
- `ORDERBY(DimDate[CalendarYear])` will then order the table by CalendarYear.
- `WINDOW(0, ABS, 1, REL, ORDERBY(DimDate[CalendarYear])`. This returns from the beginning of the table to the row after the current row.
- `WINDOW(0, REL, 1, REL, ORDERBY(DimDate[CalendarYear])`. This returns the current row and the next row.

Question 23:

**I want to save my Power BI Desktop file with schema, relationships, measures, report pages and visuals, and queries set up, but without any data.
What type of file should I save it as?**

A. **.pbit file**

- .pbit (Power BI template) file.
- This format allows you to create a template that can be reused with different datasets.

Question 24:

**What is the ideal type for a Power BI semantic model which uses multiple tables?**

A. **A centralised Fact table, with dimensions directly connected to the Fact Table.**

Question 25:

**In the below graphic, you can see that there are two tables: FactInternetSales and DateTable.**

**There is an inactive relationship between DateTable and FactInternetSales[DueDate].**

**What function do I need to use the inactive relationship?**

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2023-09-05_09-53-24-8b8af02cc253120c2b89181c804ca711.png)

A. **USERELATIONSHIP**

- `USERELATIONSHIP(FactInternetSales[DueDate], DateTime[Date])`
- `SAMEPERIODLASTYEAR`. This allows you to calculate a measure based on a parallel period last year.
- `CALENDARAUTO`. This creates a new table, based on the earliest and latest dates in the data.
- `UNION`. This allows you to put the rows of two tables in a single table

Question 26:

**I have created a semantic model with two tables, "MtoMPeople" and "MtoMTransactions".
As shown below, the tables are connected with a solid line, and an up arrow between them.
I have created a visualization with Owner and Sum of Transaction. However, the "Sum of Transaction" for every Owner is the same.
What should I do to resolve this?**

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2024-05-14_10-13-45-4a30a2fcc87232dcbbb3fa9a98c62daf.png)

![](https://img-c.udemycdn.com/redactor/raw/practice_test_question/2024-05-14_10-13-45-52faf920a0e3d1533808181f38b7a84b.png)

What should I do to resolve this?

A. **Change the cross-filter direction to "Both".**

Question 27:

**I have created a calculation group.
I want to create an additional calculation item in that group.
What do I do?**

A. **In the Model view, Data pane, Model tab, click on the Calculation group.. Then go to Properties and click on "+New calculation item".**

Question 28:

**I am using the VertiPaq analyzer, and it shows the Cardinality for tables and individual columns.
What does the cardinality for a table mean?**

A. **It counts the number of different values in a column.**

Question 29:

**I want to create a measure which uses different formats - for example, different currency.
I click on the relevant column in the Fields/Data pane and go to Measure tools - Formatting.
What format should I use?**

A. **Dynamic**

Question 30:

**I have created a calculation group.
What can I no longer do in Power BI Desktop?**

A. **Create implicit measures.**

Question 31:

**I have two tables, Employee and Salary, which have a One-to-Many relationship. The Salary tables has a Pay column.
I want to create a measure in the Employee table which calculates the total of the Pay in all of the rows in the Salary.
I am going to use the CALCULATE function.
What DAX formula do I use?**

A. **`CALCULATE(SUMX([Salary], Pay), ALL([Salary]))`**

# 17. A look around Fabric

# Creating a Fabric capacity and configure Fabric-enabled workspace settings

## Microsoft Fabric Overview & Workloads

Microsoft Fabric acts as an enterprise-wide extension of Power BI, integrating multiple data disciplines into a single SaaS platform.

### Key Fabric Workloads

Beyond **Power BI**, the platform includes several distinct workloads that integrate seamlessly to import, manipulate, and visualize data:

- **Data Factory:** Used for data integration (Extract, Transform, Load / ETL processes).
- **Data Engineering:** Allows the creation of **Lakehouses** and the execution of Notebooks.
- **Data Warehouse:** Provides enterprise SQL-based data warehousing capabilities.
- **Real-Time Intelligence:** Optimized for managing and querying streaming data.

> **Exam Tip:** Objects and items are frequently duplicated across multiple workloads. Focus on the object you need rather than worrying about which workload interface you pull it from.

## Interface Comparison: Power BI vs. Fabric

You can access the environment via two primary URLs, which offer slightly tailored user experiences:

| **Feature / UI Element** | **app.powerbi.com (Power BI Portal)**                                        | **app.fabric.microsoft.com (Fabric Portal)**                            |
| ------------------------ | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Primary Focus**        | Power BI artifacts and standard reporting                                    | Broad Fabric data workloads                                             |
| **Navigation Items**     | Includes separate _Create_, _Browse_, _Apps_, _Metrics_, and _Learn_ buttons | Hides Power BI-specific buttons to reduce clutter for non-BI developers |
| **Workspaces Button**    | Located on the left-hand navigation rail                                     | Relocated to the top panel (just under _Home_)                          |

## Fabric Capacity & Workspaces

To utilize any Fabric item (such as Lakehouses, Notebooks, or Warehouses), you must establish a **Fabric Capacity** and assign it to a **Workspace**.

- **Fabric Workspace:** Used to _store_ Fabric items and manage metadata.
- **Fabric Capacity:** Provides the shared **compute power (CPU)** and infrastructure required to _run_ and execute those items.
- **Licensing & Execution:** Unlike traditional Power BI sharing (which requires Pro or Premium Per User licenses), **Fabric Free Users** can run Fabric items, provided those items reside within a workspace backed by an active Fabric Capacity.

### Potential Trial Bottlenecks

- Microsoft offers a free 60-day Fabric Trial (typically provisioned at an **F64** capacity tier, providing 64 Capacity Units).
- **Restriction:** If you are using a freshly created `.onmicrosoft.com` tenant email address, you may face a **90-day waiting period** before becoming eligible for the free trial.

### Expiration Warning

When a Fabric trial or capacity expires:

- Standard Power BI items remain unaffected and viewable.
- Non-Power BI items (Lakehouses, Warehouses, Notebooks) become immediately unusable.
- **Data Loss Risk:** Non-Power BI items will be **permanently deleted 7 days after capacity expiration**.

## Provisioning Fabric Capacity in Azure

If a free trial is unavailable, an alternative is to purchase an Azure-backed Fabric capacity utilizing the $200 Azure free credit (if eligible). This can sustain a lower tier (F2/F4) for roughly 30 days if managed efficiently.

### Step-by-Step Creation Path

1. Navigate to **`portal.azure.com`**.
2. Search for **Microsoft Fabric** and click **Create**.
3. Configure the following required properties:
   - **Subscription & Resource Group**
   - **Capacity Name:** Must be entirely lowercase letters and numbers, and globally unique within the selected region.
   - **Region:** Choose the region geographically closest to your data or users.
4. Select the **Size (SKU - Stock Keeping Unit):**
   - Costs scale linearly (doubling the SKU tier doubles the capacity units and the estimated hourly cost).
   - For entry-level testing, select **F2** instead of production-level tiers like F64.
5. Click **Review + Create**, then **Create**.

### Cost Management (Pausing Capacity)

Fabric capacities are billed on a pay-as-you-go model while running.

- To prevent ongoing charges when not in use, click the **Pause** button in the Azure portal and wait for the status confirmation.
- You can **Resume** the capacity whenever you need to resume development.

## Linking Capacity to a Workspace

Once your capacity is created in Azure, you must map it back to the Fabric/Power BI service:

**1.Refresh the Service:**

Refresh `app.powerbi.com` to ensure the platform detects your newly provisioned Azure Fabric capacity.

**2.Create a New Workspace:**

Go to **Workspaces** and click **New Workspace**. Name your workspace appropriately.

**3.Configure Advanced Capacity Settings:**

Expand the **Advanced** settings section of the workspace creation pane.

**4.Assign the Azure Capacity:**

Under License Context, select **Fabric capacity** (or Trial if applicable), select your specific capacity name from the dropdown list, and click **Apply**.

# Identify requirements for a Fabric solution and manage Fabric capacity

## Fabric SKUs and Compute Architecture

In Microsoft Fabric, compute power is measured in **Capacity Units (CUs)** and purchased via specific Stock Keeping Units (SKUs).

- **F-SKUs:** The naming convention corresponds directly to the number of Capacity Units provided (e.g., F2 provides 2 CUs, F64 provides 64 CUs).
- **Scalability:** You can scale your capacity up or down directly in the Azure Portal (from F2 up to F2048). Doubling the SKU tier doubles both the compute power and the cost.
- **Legacy SKUs:** Fabric F-SKUs replace older legacy Power BI capacities.
  - **P-SKUs (Premium Per Capacity):** e.g., P1, P2. _Note: Microsoft will not issue new P-SKUs after July 2024._
  - **A / EM SKUs:** Used specifically for embedded Power BI objects.

| **Fabric SKU** | **Legacy Equivalent** | **Capacity Units (CUs)** | **Target Audience**                      |
| -------------- | --------------------- | ------------------------ | ---------------------------------------- |
| **F2**         | N/A                   | 2                        | Single developer / testing (Lowest tier) |
| **F64**        | **P1**                | 64                       | Enterprise / Production                  |

!image.png

!image.png

## The Fabric Licensing Model

Understanding the interplay between user licenses and capacity SKUs is a core concept for the DP-600 exam.

### User License Types

- **Free Fabric License:** Can use "My Workspace" and run non-Power BI Fabric items (like Notebooks or Lakehouses) if they have workspace permissions. **Limitation:** Cannot view shared Power BI objects by default.
- **Pro & Premium Per User (PPU):** Required to build, share, and consume Power BI objects across standard workspaces.

### The F64 Threshold Rule

The most critical licensing rule in Fabric revolves around the **F64 capacity tier** (the equivalent of the old Premium P1 tier).

> **Exam Tip:** If a workspace is assigned to an **F64 capacity or higher**, users with only a **Free Fabric License** CAN view shared Power BI objects. If the workspace is on **F32 or lower**, viewers strictly require a Pro or PPU license to see shared Power BI content.

## Storage Architecture & Costs

Fabric separates compute billing from storage billing.

- **OneLake:** All Fabric storage utilizes OneLake. This unified architecture ensures that different workloads (Lakehouses, Warehouses, Power BI) communicate using the exact same data source without duplication or data movement.
- **Storage Cost:** Billed separately at approximately **$0.026 per GB per month**.
- **Disaster Recovery:** Enabling Business Continuity and Disaster Recovery (BCDR) nearly doubles the storage cost per GB.

## Cost Optimization Strategies

Because Fabric capacities (like the F2 developer tier) run in Azure, you have several mechanisms to control your compute spend:

1. **Pay-As-You-Go (Hourly):** You are billed an hourly rate as long as the capacity is running.
2. **Pause and Resume:** For non-production capacities (like an F2 used for learning), you can manually click **Pause** in the Azure Portal when you are not actively developing (e.g., overnight). This completely stops the hourly compute charges. You simply click **Resume** when you are ready to work again.
3. **Capacity Reservations:** For stable, long-term workloads, you can purchase a 1-year Capacity Reservation in the Azure Portal. This provides approximately a **41% discount** compared to pay-as-you-go pricing. If you need temporary spikes in compute beyond your reservation, you can add pay-as-you-go capacity on top of it.

# A quick tour of Fabric

## Fabric Workspace & Item Creation

Fabric offers a unified environment, but the navigation experience shifts slightly depending on whether you are using the **Fabric View** or the **Power BI View**.

- **Workspace Navigation:** In the Fabric view, you must select the workspace from the list and then click it again to enter it. The Power BI view allows direct entry with a single click.
- **UI Updates:** Microsoft frequently updates the UI. When clicking **New Item**, the menu may appear across the full screen or as a side panel. Items might open in tabs or a left-hand navigation pane.
- **Item Categorization:** Some items (like Notebooks) appear under multiple categories (e.g., "Get Data" and "Prepare Data") because they serve multiple stages of the data engineering lifecycle.

## Core Fabric Data Engineering Items

These are the primary objects you will create and configure to ingest, transform, and store data.

- **Lakehouse:** A unified architecture for storing, cleaning, querying, and reporting. It supports both structured tables and unstructured files (e.g., raw text). It includes a read-only SQL Analytics Endpoint.
- **Data Warehouse:** A traditional, strictly structured SQL-based environment. It supports full Data Manipulation Language (DML) operations (Insert, Update, Delete).
- **Notebook:** An authoring environment used to manipulate data using Apache Spark (primarily PySpark and Spark SQL).
- **Dataflow Gen2:** Provides a Power Query interface (similar to Power BI desktop) to extract, transform, and load (ETL) data into destinations like a Lakehouse or Warehouse.
- **Data Pipeline:** An orchestration tool that strings multiple actions together. For example, a single pipeline can copy data, trigger a Dataflow Gen2, and then run a Notebook.
- **Shortcuts:** Virtual links that point to data stored elsewhere (either within OneLake or in external external storage) without physically moving the data.
- **Semantic Models:** The datasets generated from a Lakehouse or Warehouse that power the final Power BI reports.

## Lakehouse vs. Data Warehouse Comparison

Understanding the technical distinctions between a Lakehouse and a Warehouse is heavily tested on the DP-600. Use this table to memorize the boundaries.

| **Feature**                  | **Lakehouse**                                | **Data Warehouse**                                                       |
| ---------------------------- | -------------------------------------------- | ------------------------------------------------------------------------ |
| **Primary Skillset**         | PySpark, Spark SQL, Scala, R                 | T-SQL                                                                    |
| **Data Types**               | Unstructured (files/folders) & Structured    | Structured only (Databases, schemas, tables)                             |
| **Read Operations**          | Spark & T-SQL (via Analytics Endpoint)       | T-SQL                                                                    |
| **Write Operations**         | Spark only (T-SQL is read-only)              | T-SQL (Supports full DML: Insert/Update/Delete)                          |
| **Multi-Table Transactions** | Not supported                                | Supported                                                                |
| **Primary Interface**        | Spark Notebooks                              | SQL Scripts / Visual Queries                                             |
| **Direct Shortcuts**         | Yes (can access data directly via shortcuts) | Indirect access only                                                     |
| **Shortcut Source**          | Can act as a source for Files and Tables     | Can act as a source for Tables only                                      |
| **Security Granularity**     | Row-level and Table-level                    | **Highly granular:** Object, Column, Row-level, and Dynamic Data Masking |

> **Exam Tip:** If a scenario requires modifying existing records using `UPDATE` statements, or demands Column-Level Security/Dynamic Data Masking (e.g., hiding the first 12 digits of a credit card), you **must** choose a Data Warehouse. If the scenario involves processing raw unstructured files using Python, choose a Lakehouse.

# 18. Using data pipelines and dataflows

# Ingest data by using a dataflow

## Dataflows Gen2 & Data Ingestion

Dataflows Gen2 provides a cloud-based Power Query experience to extract, transform, and load (ETL) data into Fabric destinations (like a Lakehouse or Warehouse). You can initiate a Dataflow Gen2 directly from the central Fabric Workspace or from within a specific Lakehouse.

### Getting Data

Fabric supports over 100 data connectors. Key ingestion methods include:

- **Standard Sources:** Uploaded files (Excel, Text/CSV), SQL Server, and existing Dataflows.
- **Power Platform & Azure:** Native connections to Dataverse, Azure SQL, and other Azure online services.
- **OneLake Data Hub:** Grants direct access to internal organizational data, endorsed datasets, and personal favorites.
- **Blank Query:** Allows you to create a query from scratch by pasting raw **M code** (the Power Query Formula Language).

> **Exam Tip:** When connecting to external data sources (like an Azure SQL database that is not in your immediate environment), you must configure and provide the correct authentication credentials before the connection can be established.

## Power Query Interface & Views

The online Power Query interface in Fabric includes several views to help you manage and understand your data transformations.

| **View Type**    | **Purpose & Key Features**                                                                                                                                             |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data View**    | The standard tabular preview of your data. Allows you to highlight columns (using Shift-click for ranges) to apply transformations like _Remove other columns_.        |
| **Schema View**  | Hides the raw data and displays only a list of column names and their associated data types. Highly efficient for managing wide tables with hundreds of columns.       |
| **Diagram View** | Provides a visual, node-based map of your queries and applied steps. Includes a **Minimap** to easily navigate complex dataflows with multiple interconnected queries. |
| **Script View**  | Toggles the visibility of the M code. You can choose to view the formula for the current step or the script for the entire query.                                      |

## Data Transformation & Profiling

Dataflows Gen2 tracks every transformation you make and provides tools to assess data health.

### Applied Steps & Query Folding

- **Applied Steps:** Located in the **Query Settings** pane on the right. Every action (e.g., removing a column) creates a sequential step. You can undo an action by clicking the **X** next to that specific step.
- **Query Folding Indicators:** Small icons next to the applied steps indicate whether a step will be folded. Hovering over these icons tells you if the transformation will be pushed back to the source database (query folding) or processed locally in the Fabric engine.

### Data Profiling Tools

Data profiling helps identify data quality issues before you load the data into a Lakehouse or Warehouse. It can be enabled via the View menu or the bottom half of the Data View:

- **Column Quality:** Displays the percentage of valid, error, and empty (null) values in a column.
- **Value Distribution:** Shows the count of distinct and unique values, helping identify cardinality.
- **Column Profile:** Provides deeper statistical metrics (minimum, maximum, average, standard deviation) for the selected column.

# Add a destination to a dataflow

## Data Destinations in Dataflow Gen2

Once you have extracted and transformed your data using the Power Query interface, you must define where the data will flow. This is done by adding a **Data Destination**.

### Supported Destinations

You can add a destination via the Query Settings pane or the Home ribbon. Currently, there are five supported destination types:

1. **Fabric Lakehouse**
2. **Fabric Warehouse**
3. **Fabric SQL Database**
4. **Azure SQL Database**
5. **Azure Data Explorer**

> **Exam Tip:** Data destinations are configured on a **per-query basis**. If your Dataflow contains four separate queries, you must configure a data destination four separate times. However, if you initiate the Dataflow Gen2 creation directly _from within a Lakehouse_, that Lakehouse is automatically populated as the default destination.

## Configuring the Target Table

When mapping your query to a destination, you must select the target workspace, the specific destination item (e.g., which Lakehouse), and choose to either load into an **existing table** or create a **new table**.

### Schema Mapping and Column Types

During the setup, you must validate the data types being loaded into the destination.

- **The "Type Any" Error:** Fabric cannot load a column categorized as `type any`. This often happens with text containing accents or special characters.
- **The Fix:** To resolve this, switch off the **automatic settings** toggle in the destination configuration and manually assign the correct data type (e.g., `Text`, `Decimal Number`, `Boolean`).
- **Excluding Columns:** You do not have to load every column from your query. You can easily toggle off specific columns in the destination mapping screen if they are not needed in the final table.

## Data Load Strategies: Append vs. Replace

When loading data into a destination, you must define how it interacts with existing data. Understanding the difference between schema options during a "Replace" is critical for the exam.

- **Append:** Adds the new data rows to the bottom of the existing table. No schema changes are made.
- **Replace:** Overwrites the existing data. You must choose how the schema is handled:
  - **Fixed Schema:** Drops only the existing rows, but keeps the underlying table structure intact. **Crucial:** Because the structure remains, _any measures you have already created on this table will be retained._
  - **Dynamic Schema:** Drops the entire table completely and recreates it from scratch based on the current Dataflow run. **Crucial:** Because the table is dropped, _any existing measures built on that table will be permanently lost._

## Publishing and Post-Publishing Behavior

Once your destinations are mapped, you must publish the Dataflow to execute the data load.

### Publishing Interfaces

- **Standard Workspaces:** You will see a **Publish Now** or **Publish Later** button.
- **Version-Controlled Workspaces:** If your workspace is integrated with source control (like GitHub), the interface changes. You will not see a publish button; instead, you will use **Save** or **Save and Run**.

### Viewing the Data

- Once the refresh (indicated by circular loading icons) completes, you can navigate to your Lakehouse or Warehouse to view the tables.
- The data preview in the Fabric UI will only display a **maximum of 1,000 rows**, regardless of how large the underlying dataset is.

### Editing Restrictions After Publishing

If you need to edit a Dataflow after it has been published:

- You **can** change the destination settings (e.g., column mapping, switching from Append to Replace, changing from Dynamic to Fixed schema).
- You **cannot** change the destination _type_ (e.g., you cannot flip an existing query's destination from a Lakehouse to a Warehouse). You would need to recreate the destination mapping to do so.

# **Saving as a template and scheduling a dataflow**

> **Correction on File Formats:** The auto-generated transcript contains a slight terminology error regarding the export file type. It mentions exporting as a "Parquet" or ".py / PCB" file. In reality, exporting a Power Query template generates a **`.pqt` (Power Query Template)** file. While Fabric does store _actual data_ in Delta Parquet format within OneLake, the template file itself only contains the M code script, not the data.

## Power Query Templates (.pqt)

Exporting a Dataflow Gen2 as a template allows you to reuse complex data transformations across different workspaces or projects without having to rewrite the M code.

### Exporting & Importing

- **To Export:** Open the Dataflow, navigate to the **Home** ribbon, and select **Export Template**. You will provide a name and description.
- **File Characteristics:** The resulting `.pqt` file is extremely lightweight (often just a few kilobytes) because it exclusively stores the logic from the Advanced Editor, not the underlying data.
- **To Import:** Create a new item and select **Import from a Power Query template**.
- **Authentication Requirement:** Because credentials are intentionally excluded from the template file for security, you must manually re-enter your connection credentials when importing a template before the data will load.

## Scheduling Dataflow Refreshes

Scheduling automates the ETL process, pulling fresh data into your Lakehouse or Warehouse. The scheduling interface and granularity differ significantly depending on whether your workspace is integrated with source control (CI/CD).

### Configuration Comparison

| **Feature**               | **Standard Workspace (No CI/CD)**                            | **Version-Controlled Workspace (CI/CD)**      |
| ------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| **Navigation Path**       | `...` (Ellipsis) > Settings > Refresh                        | `...` (Ellipsis) > Schedule                   |
| **Available Frequencies** | Daily, Weekly (select specific days)                         | By the minute, Hourly, Daily, Weekly, Monthly |
| **Time Granularity**      | On the hour or half-hour (e.g., 1:00 AM, 3:30 AM)            | Down to the single minute                     |
| **Start/End Dates**       | Not explicitly configured                                    | Can define strict Start and End dates/times   |
| **Notifications**         | Can email failure alerts to the owner or additional contacts | Handled via standard scheduling interface     |

### Setting Up a Standard Schedule

To configure a standard schedule:

1. Locate the Dataflow Gen2 in your workspace list.
2. Click the **Ellipsis (...)** and select **Settings**.
3. Expand the **Refresh** section and toggle the schedule to **On**.
4. Set the frequency, specify your time zone, and add one or multiple specific run times.
5. Define who receives failure notifications and click **Apply**.

# Ingest data by using a data pipeline, and adding other activities

## Data Pipelines Overview

Data pipelines in Microsoft Fabric act as orchestration engines. They allow you to automate, schedule, and link a series of coordinated events—called **activities**—into a cohesive workflow.

### Supported Activities

Pipelines can trigger a wide variety of Fabric and Azure operations, including:

- **Dataflows (Gen2):** For ETL processes.
- **Notebooks:** For Apache Spark data manipulation.
- **Stored Procedures:** Triggered from a Data Warehouse or external SQL store.
- **Other Pipelines:** Allowing you to nest or chain workflows.
- **Notifications:** Integrating with Office 365 Outlook or Microsoft Teams.

## Configuring Activities

When you add an activity (like a Dataflow) to the canvas, you must explicitly configure its underlying settings.

### General Configuration

- **Naming:** The name of the activity on the canvas (e.g., "Import Dataflow") is just a label. You must navigate to the **Settings** tab to link it to the actual underlying item in your workspace.
- **Activity State:** You can toggle an activity to be active or inactive (skipped) during a run without deleting it from the canvas.

### Execution Parameters

- **Timeouts:** Controls how long an activity can run before it automatically times out and fails. The default is **12 hours** (minimum 10 minutes, maximum 7 days).
- **Retries:** You can set the number of times an activity will attempt to rerun upon failure, as well as the delay interval (in seconds) between retry attempts.
- **Logging Security:** Toggling **Secure Input** or **Secure Output** ensures that sensitive data passed into or out of the activity is masked and not captured in the pipeline's execution logs.

## Control Flow and Dependencies

If you place multiple activities on the canvas without connecting them, they will run **concurrently** (at the same time). To create a sequential process, you must link activities using connection conditions.

You can drag connection lines from one activity to another based on four distinct states:

| **Condition**     | **Trigger Requirement**                                           | **Common Use Case**                                                   |
| ----------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| **On Success**    | The upstream activity must complete without any errors.           | Triggering a Notebook only after a Dataflow finishes loading data.    |
| **On Failure**    | The upstream activity must encounter an error.                    | Triggering an alert email if an ETL process breaks.                   |
| **On Completion** | The upstream activity finishes, regardless of success or failure. | Triggering a final cleanup task that must run no matter what happens. |
| **On Skip**       | The upstream activity was bypassed.                               | Triggering an alternative path if a specific condition wasn't met.    |

## Automated Notifications & Dynamic Content

You can use the **Office 365 Outlook** activity to send automated emails (e.g., success or failure alerts) directly from the pipeline.

- **Authentication:** You must sign in and authorize the connection before configuring the email.
- **Dynamic Content:** Instead of static text, you can inject runtime metadata into your email subject or body. By clicking the **lightning bolt icon**, you can use basic Power Fx to insert variables like the Workspace ID, Pipeline ID, or the specific Pipeline Run ID.

## Validation and Execution

Before executing a pipeline, it is best practice to use the **Validate** tool on the Home ribbon. This checks for logical errors, such as forgetting to map a Dataflow activity to an actual Dataflow.

### Monitoring Execution

When you click **Run** (or Save and Run), the **Output** pane at the bottom of the screen populates with real-time telemetry. You can track:

- Activity status (Queued, In Progress, Succeeded, Failed).
- Start time and total duration.
- Detailed JSON inputs and outputs for each specific activity step (unless secured).

# Copy data by using a data pipeline

## Data Pipeline Copy Operations Overview

While Dataflows (Gen2) are powerful for applying complex transformations (ETL), you do not _have_ to use them just to move data. If your goal is to perform a direct copy from a source to a destination (EL - Extract and Load), you can do this natively within a Data Pipeline.

Fabric provides two primary methods to build a copy operation:

1. **The Copy Data Assistant:** A guided wizard.
2. **The Copy Activity:** A manual configuration added directly to the pipeline canvas.

## Method 1: The Copy Data Assistant

The Assistant is a step-by-step wizard and is the fastest way to move data, especially when dealing with multiple tables.

### Key Features & Capabilities

- **Multi-Table Support:** This is the critical distinction. The Assistant allows you to select **multiple tables** from the source (e.g., Lakehouse) and map them all to the destination (e.g., Warehouse) in a single pass.
- **Under the Hood:** When you finish the wizard, Fabric automatically generates a `ForEach` loop on your pipeline canvas. This loop contains an underlying Copy Activity that iterates through all the tables you selected.
- **Data Types Supported:** You can copy structured data (Tables) or semi-structured/unstructured data (Files).
- **In-Flight Adjustments:** Within the wizard, you can rename destination columns, delete specific columns from the import, or reset schema mappings.
- **Staging:** You have the option to enable staging (loading data into an intermediate location before the final write), which can improve performance for large data volumes.

## Method 2: The Manual Copy Activity

Adding the **Copy Activity** directly to the canvas gives you more granular control over the data transfer settings, but requires manual configuration.

> **Exam Tip:** A single manual Copy Activity can only transfer **one table at a time**. If you need to copy multiple tables manually, you must configure separate Copy Activities and chain them together, or build your own dynamic `ForEach` loop.

### Configuration Tabs

When you select the Copy Activity on the canvas, you must configure the properties pane:

| **Tab**         | **Key Settings**                                                                                                                                                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **General**     | Name, description, activity state, timeouts, retries, and secure input/output logging.                                                                                                                                               |
| **Source**      | Define the connection, dataset, and the specific single table.                                                                                                                                                                       |
| **Destination** | Define the target location. You can choose to load to an existing table or auto-create a new one. Advanced settings control append vs. overwrite behaviors and max concurrent connections.                                           |
| **Mapping**     | Map source columns to destination columns. Contains critical **Type Conversion Settings** (e.g., handling data truncation, treating Booleans as 1/0, formatting DateTimes, and defining culture codes like English U.S. vs. French). |
| **Settings**    | Controls performance tuning and fault tolerance.                                                                                                                                                                                     |

### Advanced Settings Details

The Settings tab holds parameters crucial for performance tuning and resilience:

- **Intelligent Throughput Optimization:** Controls how resources are allocated to the copy job (Auto, Standard, Balanced, Maximum).
- **Degree of Copy Parallelism:** Dictates how many concurrent connections the pipeline can open to speed up the transfer.
- **Fault Tolerance:** Defines what happens if the pipeline encounters bad data. You can configure it to **skip incompatible rows**, missing files, or invalid names instead of failing the entire pipeline run.
- **Logging:** Allows you to capture a log of skipped files and rows for auditing purposes.

## Execution and Troubleshooting

Before running the pipeline, always **Validate** to check for missing configurations (such as forgetting to name an auto-created destination table).

### Monitoring Outputs & Errors

When the pipeline runs, the **Output** tab at the bottom of the screen provides real-time telemetry.

- If you used the Assistant, you will see the individual table runs executing inside the `ForEach` loop.
- **Error Handling:** If an activity fails (e.g., trying to copy a `large photo` column into a Warehouse that does not support that specific data type), you can click the output icon to view the raw JSON error log to diagnose the issue. You must then adjust the mapping to exclude the incompatible column.
- **Run History:** You can view the full run history and export the logs (top 1000 rows or current page) to a CSV file for further analysis.

# Schedule data pipelines and monitor data pipeline runs

## Scheduling Data Pipelines

While Dataflows and Pipelines both support automated refresh schedules, Data Pipelines offer significantly more granular control over execution timing.

### Accessing the Schedule

You can access pipeline scheduling via:

- The **Home** or **Run** ribbon while editing the pipeline.
- The **Workspace** list by clicking the ellipsis (`...`) next to the pipeline and selecting **Schedule**.

### Scheduling Granularity

This is a key differentiator for the exam. Unlike standard Dataflows (which can only be scheduled on the hour or half-hour), Data Pipelines support highly frequent execution:

- **Daily / Weekly:** Standard frequency, allowing selection of specific days.
- **Hourly:** Can be set to run at specific hourly intervals (e.g., every 2 hours).
- **By the Minute:** Can be scheduled down to the minute (e.g., every 2 minutes).
- **Boundaries:** You can establish strict **Start and End dates/times** for the schedule to remain active.

## Monitoring Pipeline Runs

Monitoring is critical for identifying failures, bottlenecks, and performance issues in your ETL architecture.

### The Monitoring Hub

To view execution details, you can navigate to **View Run History** (via the workspace or pipeline canvas), which provides a link to the central **Monitoring Hub**.

- **Filtering:** You can filter pipeline runs by Status (Succeeded, Failed, Queued), Start Time (last 24 hours, 7 days, 30 days, custom), or Submitter.
- **Customization:** You can add additional metadata columns to the view, such as Refresh Type and Duration.
- **Actions:** From the hub, you can view deep details, open the pipeline, or click **Retry** to immediately rerun a failed pipeline.

### The Gantt View

When investigating a specific pipeline run, you can toggle from the standard text list of activities to a **Gantt View**.

- **Use Case:** The Gantt chart visualizes the duration and overlap of activities. It is highly useful for diagnosing concurrency issues (e.g., identifying if a pipeline failed because too many heavy tasks were executing concurrently rather than sequentially).

### Inspecting Activity Details

Just like during live execution, you can click the input/output icons next to any completed activity to view the raw **JSON input and output** payloads. If you identify a structural error in the JSON, you can click **Update Pipeline** to return directly to the editing canvas.

## Exporting Run Data

For external auditing or advanced reporting on pipeline performance, you can export the run history.

- **Format:** Exports to a Comma Separated Values (.csv) file.
- **Row Limit:** You can export a maximum of **1,000 records** at a time.
- **Privacy Restriction:** Personally Identifiable Information (PII), such as the submitter's name and email address, is automatically stripped and **will not be included** in the exported CSV.

# Exploring sample data (including copy data assistant) + data pipeline templates

## Exploring Sample Data in Pipelines

When learning or testing pipeline functionalities in Fabric, you do not need to connect your own data sources immediately. Fabric provides built-in sample datasets accessible directly when creating a pipeline.

### Available Sample Datasets

You can access these via the **Copy Data Assistant**. The available datasets cover different data engineering and data science scenarios:

- **COVID-19 Data Lake:** Comprehensive public health data.
- **New York City Taxi:** Standard large-scale dataset for performance testing. _(Note: Clicking the dedicated "Practise With Sample Data" button defaults directly to this dataset)._
- **Diabetes Dataset:** Contains 442 samples with 10 specific features, specifically designed for testing machine learning workloads.
- **Public Holidays Worldwide:** Contains holiday data spanning from 1970 to 2099.
- **Worldwide Importers:** A simulated transactional database for a fictitious importer and distributor based in the San Francisco Bay Area.

## Utilizing Pipeline Templates

Pipeline templates offer pre-configured workflows for common data engineering tasks. Reviewing these templates is an excellent way to understand how to architect pipelines for real-world scenarios.

### Common Template Use Cases

Templates are categorized by their primary function and can be accessed right from the pipeline creation screen.

**1. Data Ingestion & Migration**

- Copy data from an Azure Data Lake Storage (ADLS) Gen2 account.
- Copy data from an Azure SQL Database directly into a Lakehouse (as either a structured table or an unstructured file).
- Copy predefined sample data into a Lakehouse or Data Warehouse.
- Copy multiple file containers from external file stores (like Azure Blob Storage).

**2. Filtered & Incremental Copying**

- Copy files based on specific metadata filters, such as extracting only the files based on their **last modified date**.

**3. Automated File Management**

- Move files seamlessly from one folder to another.
- Perform routine cleanup by automatically deleting files that are older than 30 days.

# Implement Fast Copy when using dataflows

## Fast Copy Overview

While the Data Pipeline Copy Activity is the fastest way to move massive amounts of data, it lacks the graphical user interface needed for even simple column adjustments. Conversely, standard Dataflows Gen2 provide a great GUI but can introduce processing overhead for massive, straight data loads.

**Fast Copy** bridges this gap. It allows you to utilize the underlying speed and engine of the pipeline copy activity while working within the intuitive graphical user interface of a Dataflow Gen2.

## Technical Constraints and Requirements

Fast Copy cannot be used with every connection type or complex transformation. To pass the DP-600 exam, you must memorize these strict boundaries.

### 1. Supported Data Sources

Fast Copy only supports a specific subset of incoming data connectors:

- **Azure Data Lake Storage (ADLS) Gen2** _(CSV or Parquet files only)_
- **Azure Blob Storage** _(CSV or Parquet files only)_
- **Azure SQL Database**
- **Fabric Lakehouse**
- **PostgreSQL**
- **On-Premises SQL Server**

> **Exam Tip:** Standard Excel workbooks, web sources, or general local file uploads are **not** supported. If an unsupported source is selected, the Power Query window will explicitly display the warning: _"This source is not supported by Fast Copy."_

### 2. Supported Data Destinations

- Currently, the data destination **must be a Fabric Lakehouse**.
- **Workaround:** If you need the final destination to be something else (like a Data Warehouse), you can use the Fast Copy query as a **staging query**, and then reference that query's results in a second, standard Power Query step.

### 3. Allowed Transformations

If you apply advanced transformations, Fabric will automatically drop out of Fast Copy and fall back to the standard, slower evaluation engine. You are strictly limited to these basic structure-altering transformations:

- Combine Files
- Choose Columns / Remove Columns
- Rename Columns
- Change Data Types

## Activation and Threshold Limits

Enabling the setting does not mean Fast Copy runs automatically on every query. It is designed purely for large-scale data ingestion.

### How to Enable Fast Copy

1. Inside your Dataflow Gen2, navigate to the **Home** ribbon and click **Options**.
2. Go to the **Scale** tab.
3. Check the box for **"Allow use of Fast Copy connectors"** and click OK.
4. When successfully configured and supported, the UI will state: _"This step is going to be evaluated with Fast Copy."_

### Default Evaluation Thresholds

To prevent unnecessary resource overhead, the Fabric engine bypasses Fast Copy for small files and small datasets unless it clears these minimum file size or row count limits:

| **Supported Source**                | **Minimum Threshold Required to Trigger Fast Copy**  |
| ----------------------------------- | ---------------------------------------------------- |
| **ADLS Gen2 / Azure Blob Storage**  | Data size must be **greater than 100 MB**            |
| **Azure SQL Database / PostgreSQL** | Dataset must contain **greater than 5 million rows** |

### Overriding the Threshold

If you want to force Fabric to process a smaller dataset using the Fast Copy engine:

1. Ensure Fast Copy is enabled in your Global Options first.
2. Right-click the specific query in the left-hand navigation pane.
3. Check the option for **Require Fast Copy**.

# Quiz

!image.png

!image.png

!image.png

!image.png

!image.png

# 19. Transforming data using a Dataflow Gen2

# The first part of the Home menu, including converting column data types

## The Core Engine: Applied Steps and M Language

When you ingest data into Dataflows Gen2, the system records your actions sequentially in the **Query Settings** pane on the right side of the screen as **Applied Steps**.

- **Declarative Logic:** Instead of altering the source data directly, these steps represent a recipe. Every time the dataflow is refreshed, the data engine automatically re-executes these exact steps in order, ensuring a consistent final output.
- **The M Language:** Underneath the graphical user interface, every single applied step is coded in **M** (the Power Query Formula Language).
- **Exam Boundary:** For the DP-600 exam, you are **not** required to manually write or modify raw M code. You only need to know how to navigate and manage the visual interface to apply the correct configurations.

## Data Ingestion and Step Re-ordering

When importing multi-sheet spreadsheets (like an Excel workbook), Fabric creates an individual, isolated **Query** for each spreadsheet selected.

### Automated Step Sequence

Upon initial ingestion, the system typically generates three default steps:

1. **Source:** Establishes the connection to the underlying file container.
2. **Navigation:** Points to the specific table, sheet, or folder path.
3. **Changed Type:** Attempts to evaluate the columns and apply data types.

## Column Transformation Fundamentals

### Promoting and Demoting Headers

Often, structural problems with raw files require you to fix table headers manually:

- **Use First Row as Headers:** Promotes the very first row of your data grid to become the official column text headers.
- **Use Headers as First Row:** Demotes the column titles back down into Row 1 of your data grid.

> **Exam Tip / Real-World Scenario:** If a source system outputs files with unnecessary metadata rows or blank rows at the very top, the automated system might mistakenly promote a blank row or the wrong metadata row to the header. To fix this, you must click the **X** to delete the automated _Promoted Headers_ and _Changed Type_ steps, use **Remove Top Rows** to clear out the junk metadata, and then manually click **Use First Row as Headers**.

### Managing Data Types

Every column must have an explicitly assigned data type before it can be securely mapped to a target.

- **The "Type Any" Catch-All:** Raw data often defaults to a data type of **Any** (indicated by an `ABC 123` icon). This means it can hold any structural format, but it **cannot** be written directly into a Fabric Lakehouse table as "Any."
- **Why Change Type Comes Last:** Fabric waits to change data types automatically until _after_ headers are promoted. If it attempted to set a column to a numeric data type while the text header (e.g., "ProductKey") was still in Row 1, the data validation would fail because of conflicting text and numbers.

### Changing Types with Locale

If you receive datasets formatted using cultural or geographical standards different from your environment, standard data type conversion will fail.

- _Example:_ A German data file uses commas as a decimal separator ($1.234,56$ instead of $1,234.56$).
- _The Solution:_ Right-click the column or click the data type icon, choose **Using Locale...**, choose your target data type (e.g., Decimal Number), and select the originating culture/locale (e.g., German). The engine will then parse the delimiters correctly without throwing conversion errors.

## Home Ribbon: Left-Pane Reference

The left portion of the **Home** ribbon hosts several vital workspace and data architectural controls:

| **Control Feature**      | **Operational Function**                                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Manage Connections**   | Central gateway to view, update, edit, or clear security credentials for local or global project database connections.                                       |
| **Advanced Editor**      | Opens a code window showing the complete script of the active query written in M.                                                                            |
| **Add Data Destination** | Duplicates the visual function of the bottom-right corner menu, allowing you to route your transformed query directly into an authorized Fabric destination. |

### Duplicate vs. Reference

Understanding the architectural difference between these two cloning methods is frequently tested:

- **Duplicate:** Creates a complete, separate copy of the existing query along with **all of its historical applied steps**. Modifying the original query will have absolutely no impact on the duplicated query.
- **Reference:** Creates a brand new, dependent query that points directly to the **final output stage** of the source query. If you add or alter a step in the original query, those changes automatically flow through down to the referenced query.

# Removing rows/columns, and filtering and sorting data

## Column Selection and Navigation

Managing columns effectively is the first step in optimizing dataflow performance, as removing unnecessary columns early reduces memory overhead.

### Selection Shortcuts

- **Disparate Columns:** Hold `Ctrl` while clicking specific column headers to select columns that are not next to each other.
- **Range of Columns:** Click the first column header, hold `Shift`, and click the last column header to select all columns in between.
- **Reordering:** You can move a column simply by dragging and dropping it to a new position. This automatically records a **Reordered Columns** applied step.

### Choose Columns vs. Remove Columns

There are two ways to eliminate unwanted columns. They achieve the same visual result but create different applied steps:

- **Remove Columns / Remove Other Columns:** Triggers a command based explicitly on what you selected. If you choose "Remove Other Columns," only the selected columns stay.
- **Choose Columns:** Opens a checklist window of all available columns. This is highly recommended for tables with a high column count.
- **Go To Column:** A navigation shortcut. If a table is too wide to scroll through, use `Home > Choose Columns > Go To Column`. You can sort the columns alphabetically ($A$ to $Z$) to find and jump directly to the column without creating a new applied step.

## Row Filtering and Sorting

Unlike Excel, where filtering merely hides rows temporarily, filtering in a Fabric Dataflow physically removes the filtered-out records from that point onward in the data stream. To get the data back, you must delete the corresponding **Filtered Rows** applied step.

### Top/Bottom Row Maintenance

- **Remove Top Rows:** Commonly used to strip out hardcoded text titles or blank lines that appear at the very top of a raw data file before the actual data grid starts.
- **Keep Top Rows:** Retains a specified number of rows from the top and drops everything else.

> **Exam Tip / Pattern:** To get a "Top 10" or "Top 20" list based on a metric (e.g., the highest sales amounts), you must apply a two-step pattern: **First**, sort the target column in descending order ($Z$ to $A$). **Second**, use _Keep Top Rows_ and specify the count.

### Type-Specific Filtering Options

The available filtering logic adapts automatically depending on the column's data type:

- **Text Columns:** Offers text-specific rules like _Equals_, _Begins With_, _Ends With_, and _Contains_.
- **Numeric Columns:** Offers math-specific rules like _Greater Than_, _Less Than_, _Equal To_, or _Between_.
- **Date/Time Columns:** Offers chronological rules like _Before_, _After_, _Between_, or dynamic relative windows (_In the Next X Days_, _In the Previous X Months_), as well as _Is Earliest_ and _Is Latest_.

### The Case-Sensitivity Trap

Text filtering in Power Query and Dataflows Gen2 is strictly **case-sensitive**.

- _Example:_ Filtering for rows that contain `"washer"` (lowercase) will completely skip rows containing `"Washer"` (uppercase).
- _UI Quirk:_ If you click the **Gear/Wheel icon** next to a filtered step to correct a casing mistake, the interface may occasionally fail to register the update if you only change the capitalization. To force the engine to register the change, type a temporary placeholder character (e.g., `"washer2"`), then go back in and change it to the correct string (`"Washer"`).

## Structural Column Alterations

### Splitting Columns

When a column contains conjoined data elements (such as an alphanumeric ID like `ETA12345`), you can isolate components using **Split Column**:

- **By Delimiter:** Splits data by a character like a comma, space, or hyphen.
- **By Number of Characters:** Splits after a strict count of characters. Under advanced options, you can specify to split _Once, as far left as possible_ (e.g., splitting after the first 3 letters to separate `ETA` from `12345`), _Once, as far right as possible_, or _Repeatedly_.

### Replacing Values

- Allows you to search for a specific text string or numeric value across a column and swap it out for another (e.g., replacing `"ball"` with `"sphere"`). This generates a **Replaced Value** applied step.

# Grouping and aggregating data, and duplicating and referencing queries

## Data Aggregation via Group By

The **Group By** feature allows you to collapse a detailed table into a summarized dataset based on specific attributes. You can access it via `Home > Group By` or `Transform > Group By`.

### Technical Implications

- **Data Loss in Current Query:** When you apply a Group By operation, it is recorded as an applied step. From that point forward in the query engine, **the detailed, row-level records are permanently summarized and no longer accessible** within that specific query.
- **The "Incomplete List" UI Warning:** When inspecting a column's values via the filter dropdown before grouping, Fabric may display a warning: _"This list may be incomplete."_ By default, the UI only loads a small preview sample of rows. You must click **Load more** to force the engine to scan the entire dataset and reveal all unique values.

## Group By Configurations: Basic vs. Advanced

The Group By configuration window adapts depending on how many dimensions and metrics you need to summarize.

### 1. Basic Mode

- Used when grouping by a **single column** to produce a **single metric** (e.g., grouping by `Safety Stock Level` to calculate the total row count).

### 2. Advanced Mode

Advanced mode unlocks multi-dimensional analysis and complex metric definitions:

- **Multi-Column Grouping:** Click **Add grouping** to segment your data across multiple attributes simultaneously (e.g., group by both `Safety Stock Level` _and_ `Reorder Point`).
- **Multi-Metric Aggregations:** Click **Add aggregation** to compute several distinct summary statistics at the same time.

### Available Aggregation Operations

- **Standard Math/Stats:** Sum, Average, Minimum, Maximum.
- **Median:** Reorders the data sequentially and extracts the absolute middle value (or the average of the two middle values if the row count is even).
- **Percentile:** Extracts the value below which a given percentage of observations fall (e.g., entering `30` calculates the 30th percentile).
- **Row Counting:** \* _Count Rows:_ Counts every record in the group.
  - _Count Distinct Rows:_ Counts unique rows while ignoring exact duplicates.
- **All Rows:** Instead of a scalar number, this returns a nested sub-table containing the raw row-level records for that specific group, allowing you to retain detail alongside summaries.

### Fuzzy Grouping

When grouping text columns that contain typos, inconsistent casing, or spelling variations, you can enable **Fuzzy Grouping**.

- This leverages an intelligent matching algorithm to bind similar values together.
- You can configure parameters like the _similarity threshold_ or toggle _Ignore case_ to ensure strings like `"washer"` and `"Washer"` collapse into the same group.

## Architectural Best Practices: Duplicate vs. Reference

Because a Group By operation destroys the underlying detailed data stream in that query, you often need to clone your main table before aggregating it. Choosing the right cloning mechanism is heavily tested on the DP-600 exam.

```jsx
DUPLICATE METHOD (Isolated Copies):
Raw Data ──> Query A (Includes Steps 1-4)
         ──> Query B (Copy of Steps 1-4) ──> Group By Step Added

REFERENCE METHOD (Lineage / Dependency):
Raw Data ──> Query A (Steps 1-4) ───(Final Output)───> Query B ──> Group By Step Added
```

### 1. Duplicate

- **How it works:** Creates an entirely separate, decoupled copy of the original query, replicating **every historical applied step** into the new query.
- **Pros:** Complete isolation. If you modify or delete a column in the original query, it will **not** affect or break the duplicated query.
- **Cons:** Inefficient. The Fabric compute engine must process the upstream data transformations twice (once for the original query, once for the duplicate).

### 2. Reference

- **How it works:** Creates a new query whose absolute **Source** step points directly to the final output of the original query.
- **Pros:** Highly efficient performance. The Fabric engine executes the upstream transformations only once, passing the final result directly to the referenced query.
- **Cons:** Rigid dependency. If you drop a column or rename a step in the original query that the referenced query relies on for its aggregation, **the referenced query will break immediately**.

# Denormalize data by Joining data together using Merge Queries

## Merging Queries in Fabric Dataflows

**Merging** combines two existing queries horizontally by matching rows based on one or more common identifier columns (keys). This is the Power Query equivalent of a relational database `JOIN`.

### Execution Methods

- **Merge Queries:** Overwrites the currently selected query. It appends the matching columns from the second table directly into the active query's existing schema.
- **Merge Queries as New:** Leaves the original two queries completely untouched and creates a **brand new, third query** to house the combined output dataset.

## The Merge Interface and Configuration

When configuring a merge, you must define the relationship between the two tables:

### 1. Mapping Key Columns

- To establish the join, click on the matching column in the first table, and then click on the corresponding column in the second table (e.g., mapping `ProductCategoryKey` to `ProductCategoryKey`).
- **Composite Keys (Multi-Column Join):** If a single column is not unique enough to join the data, you can match multiple columns. Hold `Ctrl` while clicking the key columns in sequential order. The UI will display small indicators ($1, 2, 3$, etc.) next to the headers to confirm the matching order is identical in both tables.

### 2. The 6 Join Kinds

The **Join Kind** determines which rows are retained or excluded from the final output grid.

| **Join Kind**   | **Technical Behavior**                                                                                         | **Core Use Case**                                                   |
| --------------- | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Inner**       | Retains **only** the rows where the key value matches perfectly in both tables.                                | Extracting records that have complete, bidirectional relationships. |
| **Left Outer**  | Retains **all** rows from the first (left) table, and only the matching records from the second (right) table. | Enforcing standard dimension-to-fact relationships.                 |
| **Right Outer** | Retains **all** rows from the second (right) table, and only the matching records from the first (left) table. | Swapping priority to the second dataset.                            |
| **Full Outer**  | Retains **all** rows from both tables, filling unmatched fields with nulls.                                    | Auditing lists or combining overlapping datasets.                   |
| **Left Anti**   | Retains rows from the first (left) table **only** if they have **no match** in the second table.               | Identifying missing keys or isolating orphan records.               |
| **Right Anti**  | Retains rows from the second (right) table **only** if they have **no match** in the first table.              | Finding records in a secondary system missing from the primary.     |

## Expanding the Merged Columns

Immediately after a merge step is applied, the rightmost column will display structured `Table` values instead of scalar data. This occurs because one record in your primary table might match multiple records in your secondary table.

### The Expansion Process

1. Click the **Expand Icon** (two arrows pointing outward) in the header of the new table column.
2. De-select columns you do not need. _Best Practice:_ Uncheck the join key column from the secondary table since it already exists in the primary table.
3. **Use original column name as prefix:** * If *checked\*, columns are renamed as `TableName.ColumnName`.
   - If _unchecked_, columns retain their raw names. If a column name conflicts with an existing column, Fabric automatically appends a numeric suffix (e.g., `ProductCategoryKey` becomes `ProductCategoryKey.1`).

> **Granularity Impact:** Expanding a table can drastically alter the grain of your data. For example, expanding 4 rows of a `Category` table by a `Subcategory` table can cause the dataset to explode out into 37 rows to accommodate every underlying subcategory record.

## Architectural Context: Normalization vs. Denormalization

Understanding when to merge tables relates directly to data warehousing design concepts on the DP-600.

```jsx
NORMALIZED MODEL (Snowflake / Multi-Table):
[DimProduct] ──> [DimProductSubcategory] ──> [DimProductCategory]

DENORMALIZED MODEL (Star Schema / Unified Table):
[DimProductExpanded] ──> Includes (Product + Subcategory + Category attributes)
```

### Normalized Data

- **Structure:** Separated out into highly distinct tables to minimize redundancy (e.g., separating Categories, Subcategories, and Products).
- **Strength:** Optimized for transactional data ingestion, write operations, and storage efficiency.

### Denormalized Data

- **Structure:** Merged flat tables where a single row contains data spanning multiple operational subjects.
- **Strength:** Significantly faster for analytical reporting and processing. It eliminates the overhead of running real-time, complex joins when running queries.

# Unioning data using Append Queries

## Append vs. Merge: The Critical Distinction

Understanding when to use an Append versus a Merge operation is fundamental to relational data modeling in Microsoft Fabric.

- **Merge Queries (Horizontal Combination):** Combines tables by adding **columns** from different queries based on a matching key column.
- **Append Queries (Vertical Combination):** Combines tables by adding **rows** from two or more queries right underneath each other. In SQL and other programming languages, an append operation is known as a **UNION**.

## Appending Tables with Mismatched Schemas

When you append two tables that do not share identical column names, the query engine expands the schema to include _all_ unique columns from both tables.

### The Problem of Unexpected Nulls

If you append two entirely different structural tables (such as a 4-row `Category` table and a 37-row `Subcategory` table):

- The resulting table size will equal the sum of both sets of rows ($37 + 4 = 41$ rows).
- Any columns that exist in Table A but not Table B will be padded with **`null` values** (representing missing data) for Table B's rows, and vice versa.
- The engine will only collapse data into a single column if the column names match **exactly**.

## Schema Alignment and the Column Name Trap

Column matching in Fabric Appends is strictly **case-sensitive and character-sensitive**. If column headers do not match perfectly across all target tables, the engine creates separate columns, causing structural layout issues and data gaps.

### The Speling/Casing Trap Scenario

Consider appending two manually entered data tables containing tracking information:

- **Table 1 (`Append1`):** Contains columns named `ID`, `Description`, and `color` (US spelling).
- **Table 2 (`Append2`):** Contains columns named `ID`, `Description`, and `colour` (UK/Irish spelling).

### The Execution Result

When appending `Append1` into `Append2`:

1. The `ID` and `Description` data will align perfectly into single columns because their header text matches exactly.
2. The engine will generate **two separate columns** for color: one named `color` and one named `colour`.
3. Rows 1 and 2 will contain valid data in `color` but show `null` in `colour`. Rows 3 and 4 will show `null` in `color` but contain valid data in `colour`.

> **Exam Tip / Best Practice:** To avoid unexpected `null` columns during an execution, always audit and rename column headers across your datasets so they align perfectly **before** triggering the append step. If you need to preserve the original tables un-altered, **Duplicate** or **Reference** them first to adjust the schemas safely.

## Append Configurations: Two Tables vs. Multi-Table

The Append configuration interface offers two operational layouts depending on how many datasets you need to combine:

- **Two Tables Mode:** A simple dropdown menu to pick a primary table and a secondary table to stack underneath it.
- **Three or More Tables Mode:** Opens an advanced shifting window where you can add multiple tables into a linear collection queue.

> **Architectural Advantage:** Unlike a **Merge** operation (which can only join two tables at a time and must be chained sequentially in multiple steps to join three or more tables), an **Append** can stack **three or more tables simultaneously in a single step**.

## Architectural Workflow: Modifying vs. Generating New

- **Append Queries:** Directly alters the active, selected query by pushing the rows of the secondary table into it.
- **Append Queries as New:** Preserves both source queries in their original states and generates an entirely separate, **third query** to store the stacked results.

# Identify and resolve duplicate data, missing data, or null values

## Technical Definitions: Distinct vs. Unique

Understanding the strict semantic difference between **Distinct** and **Unique** counts is critical when profiling your datasets.

- **Distinct Values:** The count of all completely different values in a column, regardless of how many times they repeat. Duplicates are essentially compressed down into a single representative value.
- **Unique Values:** The count of values that appear **exactly once** in that specific column. Any value that repeats two or more times is excluded from this count.

> _Example:_ Consider a column containing the sequence: `[1, 2, 3, 3, 3, 3, 4, 5]`
>
> - **Distinct Count = 5** (The values are `1`, `2`, `3`, `4`, and `5`)
> - **Unique Count = 4** (The values `1`, `2`, `4`, and `5` occur only once; `3` is excluded because it repeats)

## Data Profiling via the View Tab

Instead of scrolling through millions of records manually, Fabric provides automated graphical tools in the **View** ribbon under the **Data View** section. To unlock these features, you must check the box next to **Enable Column Profile**.

### 1. Show Column Quality Details

Displays a high-level summary at the top of every column split into three precise percentages:

- **Valid:** Clean data that conforms properly to the column's assigned data type.
- **Error:** Data that failed validation (e.g., trying to parse a text string like `"No Information"` inside a Whole Number column).
- **Empty:** The percentage of rows containing `null` (the formal absence of data, rendered in lowercase).

### 2. Show Column Value Distribution

Displays a small inline bar chart directly under the header of every column. It provides a visual snapshot of the frequencies of specific data entries alongside the exact counts of distinct and unique values.

### 3. Show Column Profile in Details Pane

Unlike the first two options (which appear across all columns simultaneously), this option opens a dedicated split-pane layout at the bottom of the screen focused **only on the single column you have currently selected**.

- **Column Statistics:** Displays exact counts for Row Count, Errors, Nulls, Distinct, Unique, Empty Strings, Minimum, and Maximum values.
- **Value Distribution:** Shows an expanded chart ranking every individual value in that column by its exact frequency count (e.g., showing that `null` occurs 209 times, while ID `14` occurs 70 times).

## Resolving Nulls and Duplicates

### Replacing Nulls

If your data targets do not permit null values, you can swap them out globally for placeholder records:

1. Go to `Home > Replace Values`.
2. Type `null` (lowercase, no quotes) into the **Value to Find** field.
3. Type your placeholder text (e.g., `"No Information"`) into the **Replace With** field.

### Deduplicating and Isolation Patterns

You can clean or isolate rows based on structural anomalies by highlighting a target column and navigating to `Home > Remove Rows` or `Home > Keep Rows`:

- **Remove Duplicates:** Drops all subsequent repeating entries of a value, retaining only the very first instance it encounters. For example, deduplicating a `ProductSubcategoryKey` column with 1 to 37 and nulls collapses the entire table down to exactly 38 rows.
- **Isolating Errors Workflow:** A common exam pattern involves handling dirty data without losing records:
  1. **Duplicate** your dirty query into two separate paths.
  2. In **Query A**, choose `Remove Rows > Remove Errors`. This gives you a clean dataset ready to load immediately.
  3. In **Query B**, choose `Keep Rows > Keep Errors`. This isolates _only_ the broken rows into a dedicated target table for troubleshooting or auditing.

## Identifying Missing Data with Joins

You can leverage relational joins to find records that exist in your master dimensions but are completely missing from your operational fact transactional tables.

### Right Outer Join Discovery

- If you join `FactInternetSales` (Left Table) with `DimProduct` (Right Table) using a **Right Outer Join**, the engine retains 100% of your product catalog rows.
- If you expand the sales columns and filter for rows where the sales keys are `null`, you visually isolate every single product that exists in inventory but has **never been ordered**.

### Right Anti-Join Optimization

- Instead of executing a heavy outer join and then applying filters, you can use a **Right Anti-Join** to accomplish this in a single step.
- A Right Anti-Join explicitly excludes any product records that find a matching key inside the sales table. The final output generates a lean table containing _only_ the unsold products.

### Architectural Remediations

If an anti-join uncovers missing records that _should_ be there, the engineering remediation typically involves:

1. Verifying if the data belongs to an older period or a separate ledger system.
2. Ingesting that separate source file as a new dataflow query.
3. Using an **Append (Union)** operation to stack the historical rows underneath your active data stream to complete the dataset.

# Transforming existing data or adding new columns

## Architectural Rule: Transform vs. Add Column

Understanding the architectural distinction between these two menus is fundamental when manipulating schemas in Fabric.

- **Transform:** Operates directly on the **existing column**. The original data is overwritten by the output of the transformation step, maintaining the exact same column count in your schema.
- **Add Column:** Generates a **brand new column** at the rightmost end of your table to house the output. The original source column remains completely untouched.

## The Transform Menu: In-Place Operations

The Transform menu modifies your active columns. The available tools adapt dynamically based on whether you select text, numeric, or date/time data types.

### 1. Table and Schema Basics

- **Transpose:** Flips the entire data grid, turning your rows into columns and your columns into rows.
- **Reverse Rows:** Inverts the row order of the table (the last row becomes the first row, independent of sorting rules).
- **Count Rows:** Returns a single scalar value representing the total rows in the dataset (e.g., `606`).
- **Mark as Key:** Designates a column as the table's **Primary Key**, placing a small key icon next to the header. This signals that values are strictly unique.
  - **Performance Impact:** When updating a destination table over multiple scheduled runs, marking a key forces Fabric to perform in-place updates. Without a designated key, Fabric deletes the older record before executing a fresh insert, which introduces extra processing overhead.

### 2. Structural and Text Operations

- **Fill (Up / Down):** Replaces consecutive `null` values with the nearest valid cell entry. For example, applying _Fill Down_ to a sequence of `[1, null, null, 5, null]` converts it into `[1, 1, 1, 5, 5]`.
- **Format:** Changes string casings (Lower Case, Upper Case, Capitalize Each Word) or inserts static text as a **Prefix** (at the start) or **Suffix** (at the end).
- **Merge Columns:** Joins two or more selected text columns into a single column using a specified delimiter (e.g., a comma, space, or colon).

### 3. Number, Date, and Time Operations

- **Standard / Scientific Math:** Applies arithmetic operations (Add, Multiply, Divide). Advanced functions include **Absolute Value** (stripping out negative signs) and rounding configurations (_Round Up_, _Round Down_, or rounding to specific decimal boundaries).
- **Date Metrics:** Extracts components from a date cell, including the Year, Month, Quarter, or Week. It can pinpoint the exact _Start of Year_ or calculate the total _Days in Month_.
- **Age:** Computes a duration representing the exact time elapsed between the current system timestamp and the date recorded in the cell.
- **Durations:** Measures the elapsed chronological block between two distinct date/time values.

## The Add Column Menu: Structural Expansion

Use the Add Column menu when you need to derive new variables while preserving your original source fields.

### 1. Intelligent Creation

- **Column from Examples:** An AI-assisted modeling tool. By manually typing a few desired output values based on adjacent columns (e.g., typing `"1998"` next to a full date of `12/05/1998`), the engine automatically reverse-engineers the correct transformation logic and populates the remaining rows.
- **Custom Column:** Opens a formula canvas where you can write custom **M code** expressions to generate advanced row calculations.

### 2. Logic and Indexing

- **Conditional Column:** A visual UI wizard for building **If-Then-Else** conditional statements.
  - _Example:_ **IF** `StartDate` is less than `01/01/1999`, **THEN** output `"Legacy"`, **ELSE** output `"Modern"`.
- **Index Column:** Appends a sequential counter column down the length of the table. You can choose to start the index from `0`, `1`, or input a custom starting number and increment step.
- **Rank Column:** Analyzes a numeric column and evaluates where each specific row value ranks relative to the rest of the dataset.
- **Duplicate Column:** Creates an exact, structural clone of the selected column to let you safely apply separate operations later on.

# 20. Loading and saving data using notebooks

# Ingesting data into a lakehouse using a local upload

## The Lakehouse Storage Architecture

A Microsoft Fabric Lakehouse splits its underlying storage into two distinct, isolated storage zones in the Explorer pane. Understanding how files behave in these zones is highly tested.

```jsx
LAKEHOUSE STORAGE STRUCTURE:
└── [Lakehouse Name]
    ├── 📁 Tables (Managed Delta Lake tables; accessible via SQL Analytics Endpoint)
    └── 📁 Files  (Raw storage zone; any format: CSV, JSON, Parquet, images, etc.)
```

### 1. The Files Section (Raw/Unmanaged)

- This is a standard object storage layer. You can upload files of **any structural format** (CSV, TSV, JSON, Parquet, PDF, etc.).
- Files stored here are **not structured tables** and **cannot** be queried natively using the SQL Analytics Endpoint or standard SQL statements.

### 2. The Tables Section (Managed Delta Lake)

- This zone contains managed tables stored exclusively in **Delta Parquet** format.
- Only data in this section appears in the SQL Analytics Endpoint schema and can be queried using T-SQL.

## Method 1: Ingestion via the Fabric UI Portal

The most direct way to ingest small to medium datasets is through the SaaS web interface.

### Step-by-Step File Ingestion

1. In the Lakehouse Explorer pane, hover over the **Files** folder and click the ellipsis (**...**).
2. Select **Upload > Upload files**.
3. Browse and select your target file (e.g., `purchases.csv`). You can hold `Shift` to select multiple files simultaneously.
4. Check **Overwrite if files already exist** if you need to replace an older version of the file with the same name.
5. Click **Upload**.

### Converting Raw Files to Tables

To expose a raw CSV file from the _Files_ section to the SQL engine, you must explicitly parse it into a table:

1. Hover over the uploaded file in the Files directory, click the ellipsis (**...**), and select **Load to Tables > New table**.
2. **Configure Load Settings:**
   - **Table Name:** Set the target destination table name (defaults to the file name, e.g., `purchases`).
   - **First Row as Column Names:** Check this box if your file contains text headers in row 1.
   - **Separator:** Define the structural delimiter. For a standard CSV, ensure **Comma** is selected.
3. Click **Load**. The engine automatically runs an ingestion job, converts the data into Delta Parquet format under the hood, and displays it in the **Tables** section.

## Method 2: Ingestion via OneLake File Explorer

For bulk operations, automated scripts, or local developer workflows, Microsoft provides a native client application called the **OneLake File Explorer for Windows**.

### Installation & Authentication

1. Download the app installer from the official Microsoft portal.
2. Run the installation package and launch the client.
3. **Sign In:** You must authenticate using your exact Fabric/Power BI organizational credentials.

### File Explorer Integration

Once authenticated, the app seamlessly maps the entire cloud-based Fabric OneLake storage directly into your native Windows File Explorer as a virtual drive.

### Local Deployment Workflow

1. Locate your local file (e.g., `purchases.csv`) and copy it (`Ctrl + C`).
2. Navigate to the **OneLake** drive partition on the left sidebar of your Windows File Explorer.
3. Drill down through the hierarchical folders: `[Workspace Name] > [Lakehouse Name] > Files`.
4. Paste the file (`Ctrl + V`).
5. **Synchronization Status:** The app displays an inline status icon next to the file:
   - **Syncing Symbol:** Data is actively streaming from your local machine up to the cloud.
   - **Synchronized Symbol (Green Check):** The file is fully uploaded and saved securely in OneLake.
6. **Fabric Portal Verification:** To see the newly synced file in the web workspace UI, you must click the ellipsis (**...**) next to the Lakehouse **Files** directory and click **Refresh**.

# Choose an appropriate method for copying to a Lakehouse or Warehouse

## Data Ingestion Decision Matrix

Choosing the correct ingestion tool in Microsoft Fabric depends primarily on three factors: **data volume/scale**, **the source location**, and the **complexity of the required transformations**.

| **Ingestion Method**                          | **Data Volume**  | **Transformation Capabilities**     | **Primary Use Case**                                                       |
| --------------------------------------------- | ---------------- | ----------------------------------- | -------------------------------------------------------------------------- |
| **Local File Upload / OneLake File Explorer** | Small            | None                                | Ad-hoc uploading of flat files (CSV, JSON, etc.) from a local machine.     |
| **Dataflow Gen2**                             | Small to Medium  | Low to Medium (Power Query UI)      | Visual, low-code data transformation using 200+ native connectors.         |
| **Data Pipeline (Copy Activity)**             | Large to Massive | None (Schema Mapping Only)          | High-throughput, rapid raw data movement without transformation.           |
| **Notebooks (Apache Spark)**                  | Large to Massive | High (Code-based PySpark/Scala/SQL) | Complex, scalable programmatic transformations and data science workflows. |

## 1. Local File Copying (Ad-hoc Ingestion)

Best suited for migrating small desktop files directly into cloud object storage.

- **Fabric Portal Upload:** Directly upload files or folders to the `Files/` directory of a Lakehouse. Includes an option to overwrite existing target files.
- **OneLake File Explorer:** A virtual drive client mapping Fabric directly to Windows File Explorer. It utilizes three primary status icons:
  - **Green Check Mark:** The file is fully downloaded and synchronized locally on your computer.
  - **Circular Sync Arrows:** Synchronisation is currently pending or actively in progress.
  - **Blue Cloud Icon:** The file is stored "Online Only" in the cloud to save local disk space.

## 2. Dataflow Gen2 (Low-Code ETL)

Best suited when dealing with small to medium-sized data volumes that require structural cleaning.

- **Connectors:** Access to over 200 specific out-of-the-box source system connectors.
- **Engine:** Leverages the familiar graphical **Power Query** interface to perform transformations (renaming, filtering, appending, merging) without writing code.

## 3. Data Pipelines & The Copy Tool (High-Scale Orchestration)

Best suited for moving massive volumes of data as fast as possible.

- **Performance:** The most rapid method for ingesting large scale enterprise data sources.
- **Transformations:** Restricted entirely to basic column/schema mapping between the source and destination. It **does not** support data-level transformations.

## 4. Notebooks (Code-First Big Data ETL)

Best suited for high-volume enterprise data requiring advanced engineering logic.

- **Capabilities:** Uses Apache Spark (PySpark, Spark SQL, Scala) to execute highly complex programmatic data transformations.
- **Development Flow:** Allows engineers to execute code blocks step-by-step and inspect the immediate tabular output preview at each stage.
- **Orchestration:** Once written, Notebook activities can be seamlessly dragged into a **Data Pipeline** canvas alongside Dataflows and Copy tools to form a fully automated, schedule-driven ETL workflow.

# Querying a table using a notebook

## Technical Overview: Core Approaches to Table Previews

There are three main code-free or low-code methods to inspect the contents of a Lakehouse table within Microsoft Fabric:

1. **Lakehouse Explorer Preview:** Open the Lakehouse artifact and click directly on any table in the left explorer pane to instantly load a top-row data grid preview.
2. **SQL Analytics Endpoint:** Switch to the SQL Analytics endpoint interface to run read-only previews and inspect schemas.
3. **Notebook Integration:** Load data frames natively into a Spark notebook to iteratively query, clean, or visualize tables.

## Fabric Notebook Architecture: Disconnected vs. Connected

Notebooks execute repeatable code block instructions interactively using logical chunks called **cells**. How a notebook connects to your data infrastructure depends on how it was created.

```jsx
CONNECTED CREATION PATH:
Lakehouse Portal ──> Click "Open Notebook" ──> Auto-attached to Context Lakehouse

DISCONNECTED CREATION PATH:
Workspace View ──> Click "New Item" ──> Blank Notebook ──> Manually Add Lakehouse Source
```

### Attaching Data Sources Manually

If you create a blank notebook directly from a workspace (`New Item > Notebook`), it has no context data connection.

- To link it, click **Add a data source** in the left notebook explorer and choose **Lakehouses**.
- Select **Existing lakehouse** to map your target environment into the notebook session.
- _Exam Tip:_ For standard Delta tables created in Fabric Lakehouses, you attach them as an **Existing lakehouse without a schema**.

## Notebook Cell Anatomy: Code vs. Markdown

A notebook canvas is composed of a vertical stack of cells. You can execute individual cells independently or change their processing order.

### 1. Markdown Cells (Documentation)

Markdown cells are non-executable blocks dedicated strictly to rich-text notes, inline documentation, and structural layout headers.

- They support formatting text inside the cell editor like **Bold**, _Italics_, headers (`#`, `##`), and structural bulleted or numbered loops.

### 2. Code Cells (Execution)

Code cells process logic against the attached cluster compute environment.

- When you execute your very first code cell, Fabric spins up an automated back-end Spark compute block called a **Session**. A typical session initializes quickly (often within 5 to 10 seconds), after which all subsequent cells run immediately.
- **Cell Control Commands:** Clicking the ellipsis (`...`) or contextual options on an individual code cell allows you to change it to markdown, clear its historical output, lock it to prevent accidental typing edits, or freeze it so it cannot be executed.

## Multi-Language Execution and Cell Magic Prefixes

Fabric Notebooks support multi-language environments. While the primary global engine default language can be selected via the bottom-right status bar configuration, you can override the language cell-by-cell.

### The Role of Cell Magics

By default, the global environment for Fabric Lakehouses defaults to **PySpark** (Python for Spark). However, you can write native query statements inside specific cells by changing the runtime language environment or adding a cell magic line identifier.

- **Spark SQL (`%%sql`):** Changes the specific code cell engine behavior to interpret statements using Spark SQL layout instead of Python script.
- **The Warehouse/Lakehouse Language Rule:**
  - **Spark SQL** is utilized for querying data stored within a **Lakehouse**.
  - **T-SQL** is utilized for querying data stored within a Data **Warehouse**.
  - _Warning:_ Attempting to manually force a Lakehouse notebook query cell into a T-SQL schema interpreter type triggers validation errors (represented by a red error warning indicator) and prevents the cell from processing.

## Code-Free Data Ingestion & Tabular Queries

You can instantly generate script blocks without typing by using the Explorer UI shortcuts:

1. Expand the attached Lakehouse tree on the left sidebar to show the **Tables** or **Files** list.
2. Click the ellipsis (`...`) next to your target table (e.g., `purchases`) and select **Load data > Spark**.
3. Fabric automatically injects a pre-configured PySpark template script block into a new cell:

Python

```
# Auto-generated PySpark template block
df = spark.sql("SELECT * FROM demo_lakehouse.purchases LIMIT 1000")
display(df)
```

Executing this statement runs the query and displays an interactive data preview grid containing up to the first 1,000 records directly below the cell.

## Native Data Visualizations and Chart Editing

Once a dataframe or database query returns data successfully in a notebook cell, you can visually explore the results directly within the output area.

### Inline Chart Adjustments

- **Chart Mode:** Switch the output grid view toggle button from **Table** view to **Chart** view.
- **Start Editing Wizard:** Click **Start Editing** on the workspace preview row to open the visualization panel. This menu allows you to map specific dimensions and numerical measures to your plot axes.

### Native Chart Formats Available

The built-in chart editor supports multiple common analytical visualization styles:

- **Comparison & Trend plots:** Bar Chart, Column Chart, Area Chart, Line Chart.
- **Distribution & Relationship plots:** Scatter Chart, Histogram, Box Plot.
- **Proportional & Structural layout grids:** Pie Chart, Pivot Table, Word Cloud.

## Quiz

!image.png

!image.png

!image.png

# 21. Creating SQL queries using the SELECT and FROM clauses

# Creating our first query, and exploring the output

## Accessing the SQL Analytics Endpoint

While Fabric Notebooks allow you to write Spark SQL queries against a Lakehouse, you can also use a dedicated, read-only relational database interface called the **SQL Analytics Endpoint**.

There are two primary methods to access it:

1. Navigate to your central workspace list and click directly on the **SQL Analytics Endpoint** artifact.
2. Open your Lakehouse portal, go to the top-right corner switcher dropdown, and flip the environment view from **Lakehouse** to **SQL Analytics Endpoint**.

## Writing Foundational SELECT Statements

Once inside the endpoint interface, you can click **New SQL Query** to open an empty statement editor.

### 1. Minimal Clauses

A valid `SELECT` statement can compute standalone expressions without targeting an actual table schema (e.g., `SELECT 1 + 1;` returns an analytical grid containing the scalar value `2`).

### 2. The Select All Asterisk ()

To extract every column and every row from a dataset, use the asterisk wildcard:

```
SELECT * FROM purchasers;
```

- **Intellisense Integration:** The editor features automated syntax autocomplete list menus. Use your keyboard arrow keys to scroll items and hit the `Tab` key to complete the table or column text instantly.
- **Semicolon Rules:** Semicolons are optional if you are executing a single query block. However, if you place multiple independent SQL queries inside a single execution cell, **semicolons are strictly required** to separate the statements.

### 3. Restricting Columns & Creating Aliases

To reduce network and memory overhead, narrow down your query to fetch only the essential data fields by explicitly naming them. You can rename the resulting column output headers using an **Alias**:

```
SELECT
    date,
    subtype,
    purchase_method AS method
FROM purchasers;
```

- **The `AS` Keyword:** The keyword `AS` explicitly declares a column alias. While some SQL dialects allow you to omit `AS` and simply space-separate the alias name, explicitly including `AS` is an engineering best practice that significantly improves code readability.

### 4. Handling Column Names with Spaces

If an output alias name contains spaces or special characters, you must escape the text formatting depending on your interface:

- **In the SQL Analytics Endpoint:** Enclose the alias text inside **single quotes (`' '`)** or **double quotes (`" "`)**.
- **In a Spark Notebook:** Neither single nor double quotes will work for column identifiers. You must enclose column names with spaces inside **backtick characters (` `)**.

## Interacting with Query Results

The results grid at the bottom of the SQL interface offers advanced, code-free data inspection and ad-hoc visualization tools.

### 1. Ad-Hoc Grid Adjustments

- **Filtering & Sorting:** Clicking the ellipsis (`...`) on any returned column header lets you sort or filter rows dynamically. You can also use the global search bar on the right side of the panel to instantly isolate rows matching a specific term (e.g., searching for `"paper"`).
- **Copying Layouts:** You can copy data grids directly out of the portal with or without the associated text header metadata.

### 2. Open in Excel Integration

Clicking **Open in Excel** downloads a dynamic spreadsheet linked directly back to your cloud data layer.

- _Prerequisite:_ You must explicitly highlight/select the text of a single `SELECT` statement in your script window before clicking the button.
- _Live Data Sync:_ This does not export a static copy of the data. It establishes an active external data connection. Once you authenticate using your Microsoft account, clicking **Refresh** in Excel reaches back to the SQL endpoint to stream the latest data into your spreadsheet.

### 3. Native Data Exploration & Visualizations

- **Explore This Data:** Generates a temporary split-pane workbook where you can quickly drag fields into structural matrices, add global filters (e.g., `Purchase Method = 'Cash'`), and pin them.
- **Visualize Results:** Launches a built-in Power BI reporting interface directly on top of your SQL statement's output. You can immediately build interactive bar, column, or line charts without leaving the SQL workspace, and save your visual layout directly to the workspace as a report.

# Adding new columns by using String Functions

## 1. String Concatenation

Combining multiple columns or static text into a single column can be handled in three distinct ways within Fabric SQL.

- **The Plus Operator (`+`):** The simplest method to join strings and **String Literals** (static text values, which must always be enclosed in **single quotes (`' '`)**).
  ```
  SELECT subtype + ' ' + purchase_method FROM purchasers;
  ```
- **The `CONCAT` Function:** Achieves the exact same result as the plus operator but uses commas to separate strings instead of arithmetic operators.
  ```
  SELECT CONCAT(subtype, ' ', purchase_method) FROM purchasers;
  ```
- **The `CONCAT_WS` Function (Concatenate With Separator):** The most efficient choice when chaining many columns together with the same delimiter. You pass the **separator first**, followed by all target columns. It automatically places the delimiter between fields and intelligently ensures no trailing separator is added at the very end of the string.
  ```
  SELECT CONCAT_WS(' ', subtype, purchase_method, subtype) FROM purchasers;
  ```

## 2. String Extraction and Length

When isolating parts of a text column, you use position-based extraction functions.

> **Exam Tip / Indexing Rule:** In SQL syntax, **string indexing begins at 1**. The first character of a string is always position `1` (unlike other programmatic languages like Python or JavaScript where indexing starts at `0`).

- **`LEFT(column, n)`:** Extracts the specified number of characters (`n`) starting from the far-left position of the string.
- **`RIGHT(column, n)`:** Extracts the specified number of characters (`n`) starting from the far-right position of the string.
- **`SUBSTRING(column, start_position, length)`:** Extracts a custom segment from the middle of a string. It takes the target column, the exact character position to start the extraction, and how many characters to pull.
  ```
  -- Starts at character 2 and extracts a block 3 characters long
  SELECT SUBSTRING(subtype, 2, 3) FROM purchasers;
  ```
- **Handling Out-of-Bounds Extraction:** If you attempt to extract characters beyond the natural boundaries of a string (e.g., asking for character 9 of a 5-character string), SQL will **not** crash or append blank spaces; it will return an empty result with a length of zero.
- **`LEN(column)`:** Returns a scalar integer representing the total character count of a string.

## 3. Whitespace Cleaning (Trimming)

Hidden spaces at the beginning or end of raw data text cells can corrupt data joins and string matches. SQL provides cleaning operations to strip out this trailing junk:

- **`LTRIM(column)`:** Left-trim; removes all leading whitespace from the beginning (left side) of the text.
- **`RTRIM(column)`:** Right-trim; removes all trailing whitespace from the completion (right side) of the text.
- **`TRIM(column)`:** Global trim; strips out hidden whitespaces from **both** the beginning and the end of the target string simultaneously.

## 4. Case Modification and Replacements

- **`UPPER(column)`:** Evaluates the string and forces all alphabetical characters into uppercase (e.g., `"paper"` becomes `"PAPER"`).
- **`LOWER(column)`:** Forces all alphabetical characters into lowercase.
- _Note:_ Fabric SQL does **not** feature native `PROPER` or `TITLE` functions to automatically capitalize only the first letter of words.
- **`REPLACE(column, string_to_find, replacement_string)`:** Scans a text column, isolates a specific substring match, and replaces it globally with a new value.
  ```
  -- Replaces abbreviation 'misc' with full word 'miscellaneous'
  SELECT REPLACE(subtype, 'misc', 'miscellaneous') FROM purchasers;
  ```

## Query Window Execution Pro-Tip

When writing and executing multiple distinct `SELECT` statements inside a single SQL script window, the engine runs them sequentially. To view the distinct outputs without clashing, look at the bottom results pane where a dropdown selector menu appears, allowing you to flip between the results grids of each individual query.

# Adding new columns by using Mathematical Functions

## 1. Basic Arithmetic and Power Functions

Fabric SQL supports all standard arithmetic operations directly on numeric columns (such as the `out` column from the `purchases` table) using standard operators: `+` (addition), `-` (subtraction), `*` (multiplication), and `/` (division).

For exponential and advanced geometric calculations, use the following built-in functions:

- **`SQUARE(column)`:** Multiplies a number by itself (e.g., `SQUARE(50)` returns `2500`).
- **`SQRT(column)`:** Calculates the square root of a number—the value that, when multiplied by itself, yields the original number (e.g., `SQRT(50)` returns approximately `7.07`).
- **`POWER(column, exponent)`:** Raises a number to a specified power or exponent.
  ```
  -- Multiplies the 'out' value by itself 3 times (cubed)
  SELECT POWER(out, 3) FROM purchasers;
  ```

## 2. Precision and Rounding: Round, Floor, and Ceiling

When shifting data from decimal values to integers or specific decimal places, choose your function based on the exact directional rounding logic your business rules require.

- **`FLOOR(column)`:** Rounds a number **down** to the nearest integer, moving toward the lower boundary (the "floor" of the room). For example, `FLOOR(14.88)` evaluates to `14`.
- **`CEILING(column)`:** Rounds a number **up** to the nearest integer, moving toward the upper boundary (the "ceiling" of the room). For example, `CEILING(14.88)` evaluates to `15`.
- **`ROUND(column, length)`:** Rounds a value to a specified number of decimal places based on standard rounding rules (0–4 round down, 5–9 round up).
  - **To specific decimals:** `ROUND(14.88, 1)` rounds to one decimal place, yielding `14.90`.
  - **To the nearest integer:** Pass `0` as the length argument. `ROUND(14.88, 0)` evaluates to `15`, while `ROUND(22.09, 0)` evaluates to `22`.
  - **To the nearest ten/hundred:** Pass a negative integer. `ROUND(14.88, -1)` rounds to the nearest ten, yielding `10`.

### Eliminating Floating-Point Errors

Computers occasionally introduce tiny rounding inaccuracies due to floating-point math binary translations (e.g., calculating `7.99 - 7.9` might return `0.089999999999`). To clean this data before storing or visualizing it, wrap your arithmetic formulas entirely inside a `ROUND` function set to your target precision:

```
SELECT ROUND(out - 7.9, 2) FROM purchasers;
```

## 3. Absolute Discrepancies and Numerical Signage

- **`ABS(column)`:** Absolute value; strips away any negative sign preceding a number and returns its positive magnitude.
  - _Use Case:_ Ideal for calculating variance or distance from a target. If an item is $1.9$ units below a baseline (returning `1.9`), wrapping the calculation in `ABS(-1.9)` converts it to a clean, positive `1.9`.
- **`SIGN(column)`:** Analyzes a numeric column and returns an indicator flag classifying the value's mathematical sign. It yields only one of three possible outputs:
  - **`1`** if the value is positive.
  - **`1`** if the value is negative.
  - **`0`** if the value is absolute zero.

## Advanced Numeric Reference Bound

For data science or specialized engineering workloads, the underlying Fabric SQL compiler supports advanced mathematical categories detailed in the course documentation:

- **Logarithmic Operations:** Calculating natural logs, base-10 logs, and exponential steps.
- **Trigonometric Tiers:** `SIN`, `COS`, `TAN`, their corresponding inverses, the exact mathematical value of `PI()`, and functions to convert angles back and forth between **Radians** and **Degrees**.

# Adding new columns by using Date and Time Functions

## 1. Date Arithmetic and Adjustments

Date manipulation in Fabric SQL allows you to add or subtract specific chronological blocks. Unlike string values, date parts used inside these functions are passed as **keywords** (not enclosed in quotes).

### The `DATEADD` Function

Increments a target date by a specific interval. It takes three arguments: the date part (e.g., `day`, `month`, `year`), the number to add, and the column.

```
-- Adds 1 day to the date column
SELECT DATEADD(day, 1, date) FROM purchasers;

-- Adds 1 month to the date column
SELECT DATEADD(month, 1, date) FROM purchasers;
```

> **Exam Tip / Substitution Rule:** There is no structural `DATESUBTRACT` function in SQL. To subtract a time frame (e.g., to go back exactly one day), you must use `DATEADD` and pass a **negative integer** as the second argument: `DATEADD(day, -1, date)`.

### Finding the Start and End of Periods

- **`DATETRUNC(datepart, column)`:** Short for date truncate. It strips away smaller time increments to return the absolute **start** of a designated period.
  - `DATETRUNC(week, date)` rolls back to the beginning of the week (Defaulting to Sunday).
  - `DATETRUNC(month, date)` rolls back to the **first day of the month** (e.g., `2027-01-01`).
- **`EOMONTH(column)`:** End of Month. Automatically calculates and returns the absolute **last day of the month** for the given date. It does not require a separate date part argument.

## 2. Calculating Durations with `DATEDIFF`

To measure the gap between two chronological values, use the **`DATEDIFF`** function (spelled with two "F"s). It evaluates the span using three parameters: `DATEDIFF(datepart, start_date, end_date)`.

```
-- Calculates the day count between the first and last day of the month
SELECT DATEDIFF(day, DATETRUNC(month, date), EOMONTH(date)) + 1 FROM purchasers;
```

> **Inclusive Calculation Trap:** `DATEDIFF` evaluates raw difference by subtraction (`End - Start`). For instance, `DATEDIFF(day, '2027-01-01', '2027-01-31')` yields `30`. To find the true total number of inclusive days in a month, you must manually add **`+ 1`** to the final scalar output (yielding `31`).

## 3. Extracting Date Components

You can break a complex timestamp column down into smaller scalar values using two different styles of functions.

### Direct Numeric Extraction

- **`DAY(column)` / `MONTH(column)` / `YEAR(column)`:** Returns the distinct component as a standard integer.

### Advanced Component Parsing

When extracting parts like hours, minutes, or weekdays, use these universal functions:

- **`DATEPART(datepart, column)`:** Returns the target component as a **number** (e.g., extracting `weekday` returns `5`).
- **`DATENAME(datepart, column)`:** Returns the target component as a descriptive text **string** (e.g., extracting `weekday` returns `"Thursday"`).

_Available Date Parts include:_ `year`, `quarter`, `month`, `dayofyear`, `day`, `week`, `weekday`, `hour`, `minute`, `second`, `millisecond`, and `microsecond`.

## 4. System Timestamps and Date Construction

### Extracting the Present System Time

Fabric supports three common functions to retrieve the live current date and time from the system cluster:

1. **`CURRENT_TIMESTAMP`:** ANSI-standard function. Returns the current date and time.
2. **`GETDATE()`:** SQL Server dialect function. Returns the exact same timestamp and millisecond precision as `CURRENT_TIMESTAMP`.
3. **`SYSDATETIME()`:** Returns the system date and time but with **higher fractional millisecond precision** than the previous two functions.

### Constructing Dates From Parts

If you have raw integer columns representing separate temporal blocks, you can stitch them into clean date objects:

- **`DATEFROMPARTS(year, month, day)`:** Requires three parameters to produce a standard date object (e.g., `DATEFROMPARTS(2026, 2, 3)` outputs `2026-02-03`).
- **Advanced Constructors:** For high-precision requirements, the documentation outlines granular methods like `DATETIMEFROMPARTS`, `DATETIME2FROMPARTS`, `SMALLDATETIMEFROMPARTS`, and `TIMEFROMPARTS` which accept additional arguments for hours, minutes, seconds, and fractional sub-seconds.

## Complete Core Date Functions Cheat Sheet

| **Function**        | **Sample Syntax**                             | **Example Output Scenario**           |
| ------------------- | --------------------------------------------- | ------------------------------------- |
| **`DATEADD`**       | `DATEADD(month, -1, '2027-01-15')`            | `'2026-12-15'`                        |
| **`DATETRUNC`**     | `DATETRUNC(month, '2027-01-15')`              | `'2027-01-01'`                        |
| **`EOMONTH`**       | `EOMONTH('2027-01-15')`                       | `'2027-01-31'`                        |
| **`DATEDIFF`**      | `DATEDIFF(month, '2026-01-01', '2027-01-01')` | `12`                                  |
| **`DATENAME`**      | `DATENAME(weekday, '2026-06-23')`             | `'Tuesday'`                           |
| **`DATEFROMPARTS`** | `DATEFROMPARTS(2026, 6, 23)`                  | `'2026-06-23'`                        |
| **`SYSDATETIME`**   | `SYSDATETIME()`                               | High-precision system clock timestamp |

# Converting between date and string data types

## Implicit vs. Explicit Conversion

When writing SQL queries, moving data between strings (text) and chronological dates is a frequent architectural task.

- **Implicit Conversion:** The Fabric SQL query engine automatically detects when a date-specific function (like `DAY()`) is wrapped around a string column that contains formatted text. It attempts to automatically convert the text to a date behind the scenes.
- **Explicit Conversion:** The engineer manually invokes conversion functions to strictly enforce data types, maintain schema integrity, and control formatting.

## Core Conversion Functions: CAST and CONVERT

To manually convert a string column into a formal date or time data type, use either `CAST` or `CONVERT`.

```
-- CAST Syntax: CAST(column AS target_type)
SELECT **CAST**(date **AS** date) FROM purchasers;
SELECT **CAST**(date **AS** datetime2) FROM purchasers;

-- CONVERT Syntax: CONVERT(target_type, column)
SELECT **CONVERT**(date, date) FROM purchasers;
```

- **`date` Data Type:** Strips away any time metrics to store strictly the year, month, and day (`YYYY-MM-DD`).
- **`datetime2` Data Type:** Formats the column to retain both the core date and a highly precise fractional timestamp.

## Fault-Tolerant Conversions: TRY_CAST and TRY_CONVERT

If you run a standard `CAST` or `CONVERT` on a column containing corrupt data or unparseable text (e.g., trying to force `'2027-02-31'` into a date schema), **the entire query will fail and throw an error**.

To safely handle dirty rows without crashing production data pipelines, use **`TRY_CAST`** or **`TRY_CONVERT`**:

```
SELECT **TRY_CAST**('2027-02-31' AS date); -- Returns NULL instead of an error
```

### Understanding NULL vs. Empty String

When fault-tolerant functions encounter an evaluation error, they output **`NULL`**.

- **`NULL`:** Represents the total and absolute _absence of data_ or an _unknown_ state across any data type (strings, numbers, or dates).
- **Empty String (`''`):** A valid text string asset that simply contains exactly zero characters. It is not a null value.

## Culture-Aware Conversions: PARSE and FORMAT

Regional differences in date formatting (such as US `MM/DD/YYYY` vs. UK/Irish `DD/MM/YYYY`) can easily lead to invalid data parsing or inverted months and days during ingestion.

### 1. Ingesting Strings with `PARSE`

`PARSE` acts similarly to `CAST`, but features an explicit `USING` clause to define the target culture or locale.

```
-- Parses '12/11/2027' safely as November 12th using the British/Irish standard
SELECT **PARSE**('12/11/2027' **AS** date **USING** 'en-GB');
```

### 2. Exporting Dates to Strings with `FORMAT`

You cannot use the string concatenation plus operator (`+`) to join a hardcoded string literal directly to a formal `date` object without throwing a data type clash error. You must first convert the date back to a string using **`FORMAT`**.

```
-- Formats a date using the Spanish standard long date template
SELECT **FORMAT**(**CAST**(date **AS** date), 'D', 'es-ES');
-- Output: "14 de enero de 2027"
```

### Custom Format Specifiers

By passing specific character strings to the `FORMAT` function, you can fine-tune your text extraction layouts:

```
-- Precise structural layout extraction
SELECT **FORMAT**(**CAST**(date **AS** date), 'dddd, dd MMMM yyyy');
-- Output: "Thursday, 14 January 2027"
```

- **`d` Tokens:** `dd` extracts a zero-padded day number (`14`). `ddd` extracts the abbreviated weekday name (`Thu`). `dddd` extracts the full text weekday name (`Thursday`).
- **`M` Tokens:** `MM` extracts the zero-padded month number (`01`). `MMMM` extracts the full text month name (`January`).
- **`y` Tokens:** `yyyy` extracts the full 4-digit numeric year calendar block (`2027`).

## Technical Summary Matrix

| **Function**   | **Core Purpose**                    | **Distinct Feature**                    | **Error Handling**        |
| -------------- | ----------------------------------- | --------------------------------------- | ------------------------- |
| **`CAST`**     | General type conversion             | ANSI-standard syntax (`AS`)             | Fails query on error      |
| **`CONVERT`**  | General type conversion             | Dialect-specific layout                 | Fails query on error      |
| **`TRY_CAST`** | Safe type conversion                | Returns `NULL` if string is unparseable | **Does not fail query**   |
| **`PARSE`**    | Localized string-to-date conversion | Uses the `USING [Culture]` clause       | Fails query on error      |
| **`FORMAT`**   | Localized date-to-string layout     | Supports custom masks (`dd/MM/yyyy`)    | Returns `NULL` if invalid |

# Converting between number and string data types

## 1. Advanced Numeric Formatting with `FORMAT`

The `FORMAT` function allows you to transform numeric columns into strings for presentation, using either custom masks or standard format specifiers.

### Custom vs. Standard Format Strings

- **Custom Formats:** Use hardcoded structural tokens like `0` (force character display even if it is zero) or `#` (optional digit placeholder), along with standard decimal (`.`) and group separators (`,`).
  ```
  -- Custom example: Forces leading/trailing zeros
  SELECT FORMAT(out, '000.00') FROM purchasers;
  -- Result for 7.9: "007.90"
  ```
- **Standard Formats:** Single-character shortcuts that adapt dynamically to a user's target culture/locale.
  - **`C` (Currency):** Automatically adds the appropriate currency symbol and digit delimiters based on the culture (e.g., `$` for `en-US`, `€` for `es-ES`).
  - **`D` (Decimal), `F` (Fixed Point), `G` (General), and `N` (Number)** are other common standard specifiers.

> **Exam Tip:** Custom format strings produce an identical visual structure across all locales. Standard format strings (like `C`) change their output dynamically depending on the target culture argument passed to the function.

## 2. Numeric Conversions and Integer Sub-Types

You can explicitly change data types using `CAST`, `CONVERT`, or `PARSE` (and their fault-tolerant `TRY_` variants) to handle numbers.

```
-- Casts a decimal dollar amount to a whole number integer
SELECT **CAST**(out **AS** int) FROM purchasers;
```

- **Truncation Behavior:** Casting a decimal number (like `7.9` or `14.88`) directly to an integer `int` data type does **not** follow rounding rules; it strictly truncates the decimal portion, returning `7` and `14` respectively.

### Summary of Integer Schema Sizes

While exact boundaries aren't heavily tested, understanding the scale hierarchy of whole-number schemas is critical for optimizing table memory in OneLake:

- **`tinyint`:** Stores very small ranges (0 to 255).
- **`smallint`:** Stores whole numbers from approximately -32,000 to +32,000.
- **`int` (Integer):** Standard storage capacity spanning from roughly -2 billion to +2 billion.
- **`bigint`:** Extends significantly further for massive big-data row counts or metrics.
- _Note:_ For floating-point decimal storage, systems rely on `decimal`, `numeric`, `float`, and `real` structures.

## 3. String Truncation and Fixed Character Padding

When casting columns to text data types, you must choose between variable or fixed-width strings:

- **`char(n)`:** Fixed-length character string. If you cast a column to `char(20)`, the database strictly allocates 20 characters of storage space. If the data string itself (like `"Paper"`) is shorter than 20 characters, the database implicitly **pads the right side with blank spaces** until it hits exactly 20 characters.
- **The Length Trap:** While the Fabric UI portal grid often visually strips out trailing empty spaces in the preview pane, checking the length via a formula exposes the structural padding:SQL
  ```
  -- Appending a dot to a char(20) column reveals a fixed length of 21
  SELECT LEN(CAST(subtype AS char(20)) + '.') FROM purchasers;
  ```

## 4. The Numeric `STR` Function Shortcut

Fabric SQL provides a specific legacy shortcut function called **`STR`** designed purely to flip numbers over into clean string assets.

- **Unique Behavior:** Unlike standard casting to an integer (which truncates), the `STR` function **automatically rounds to the nearest whole integer** during the conversion process.
  - `7.9` converted via `STR` outputs the string character `"8"`.
  - `3.19` converted via `STR` outputs the string character `"3"`.

## Conversion Functions Reference Table

| **Function Syntax**                 | **Core Parameters**                | **Unique Architectural Trait**                                  |
| ----------------------------------- | ---------------------------------- | --------------------------------------------------------------- |
| **`CAST(col AS type)`**             | `column`, `target_type`            | Standard ANSI SQL structure. Truncates integers.                |
| **`CONVERT(type, col)`**            | `target_type`, `column`            | Dialect-specific layout.                                        |
| **`PARSE(col AS type USING cult)`** | `column`, `target_type`, `culture` | Best for parsing regional text strings into numbers.            |
| **`FORMAT(col, mask, cult)`**       | `column`, `mask_string`, `culture` | Best for transforming outputs to stylized presentation strings. |
| **`STR(col)`**                      | `numeric_column`                   | Quick numeric-to-string swap; **rounds to nearest integer**.    |

# Quiz: Enrich data by adding new columns or tables

!image.png

!image.png

!image.png

!image.png

!image.png

# 22. Using the WHERE, GROUP BY, HAVING and ORDER BY clauses

# Filtering data using the WHERE clause

## Row-Level Filtering with WHERE

While the `SELECT` clause handles vertical filtering by restricting columns, the **`WHERE`** clause provides horizontal filtering by restricting rows based on specific logical conditions.

### 1. Equality and Standard Comparison Operators

Fabric SQL uses standard mathematical operators to evaluate rows.

- **Equality:** Uses a **single equal sign (`=`)**. Double equal signs (`==`), common in programming languages like Python, will result in a syntax error.
- **Comparison Suite:** Supports `>` (greater than), `>=` (greater than or equal to), `<` (less than), and `<=` (less than or equal to).

### 2. Not Equal To Syntax Variants

Fabric SQL is highly flexible and accepts three different syntax layouts to define a non-matching condition:

- **The Traditional Operator:** `<>` (literally meaning "less than or greater than," which effectively excludes the exact boundary value).
- **The Modern Operator:** `!=` (the standard programmatic exclamation mark equal symbol).
- **The Logical `NOT` Operator:** Prefacing the entire column evaluation statement with the word `NOT`. Wrapping the statement in parentheses is a best practice for clean readability.

```
SELECT * FROM purchasers WHERE out <> 7.99;
SELECT * FROM purchasers WHERE out != 7.99;
SELECT * FROM purchasers WHERE NOT (out = 7.99);
```

## Combining Conditions: Multiple Values & Ranges

When evaluating multiple logic steps, use logical conjunctions (`OR`, `AND`) or built-in shorthand operators.

### 1. `OR` vs. `IN` (Discrete Multi-Value Matching)

- **`OR` Logic:** Evaluates multiple separate conditions where **at least one** must be true for the row to be retained.
- **`IN` Shorthand:** Instead of chaining continuous `OR` strings, you can group discrete values together inside parenthesis using `IN`. This makes the code shorter and more readable.

```
-- Chained OR approach
SELECT * FROM purchasers WHERE out = 7.99 OR out = 7.9;

-- Optimized IN approach
SELECT * FROM purchasers WHERE out IN (7.99, 7.9);
```

### 2. `AND` vs. `BETWEEN` (Boundary Range Matching)

- **`AND` Logic:** Enforces a rigid boundary where **both** surrounding conditions must simultaneously evaluate to true.
- **`BETWEEN` Shorthand:** Provides a clean range limit filter.
  - **Crucial Rule:** The `BETWEEN` operator is **strictly inclusive** of both its upper and lower boundaries.

```
-- Standard AND layout
SELECT * FROM purchasers WHERE out >= 7.9 AND out <= 15.0;

-- Optimized BETWEEN layout (Inclusivity maintained)
SELECT * FROM purchasers WHERE out BETWEEN 7.9 AND 15.0;
```

## Data Type Sensitivities in the WHERE Clause

Applying filters without checking the underlying storage schemas can lead to broken or deceptive query results.

### The Text Date Trap

If a column named `date` is stored across OneLake as a plain **string/text** data type rather than a formal date type, standard chronological evaluations will fail.

- **Exact Match Failure:** Comparing text to an inverted text layout (e.g., `WHERE date = '2027-02-25'`) returns zero rows if the raw text is formatted as `'25 February 2027'`.
- **Chronological Sorting Failure:** Evaluating a text string using comparison operators (e.g., `WHERE date >= '25 February 2027'`) triggers an **alphabetical sort** rather than a chronological one. This means January records could appear while March records are dropped simply because `"J"` comes after `"F"` alphabetically.
- **The Fix:** Explicitly warp the column using conversion functions like `CAST` inside the `WHERE` clause to force valid date math:

```
SELECT * FROM purchasers WHERE CAST(date AS date) >= '2027-02-25';
```

## Pattern Matching with the LIKE Operator

When you need to search for text snippets rather than exact matches, use the **`LIKE`** operator in combination with pattern wildcards.

### Core Wildcard Tokens

- **The Percent Sign (`%`):** Represents any block of characters, spanning from zero characters, a single character, up to an infinite string length.
- **The Underscore (`_`):** Represents exactly **one single character position**. For example, `'_book'` matches `"ebook"`, `"gbook"`, or `"qbook"`.

### Wildcard Structural Patterns

- `LIKE 'ebook%'` $\rightarrow$ Finds all items that **begin with** the word "ebook".
- `LIKE '%book'` $\rightarrow$ Finds all items that **end with** the word "book".
- `LIKE '%book%'` $\rightarrow$ Finds the word "book" **anywhere** inside the column string, regardless of its position.

### Case Sensitivity in the Analytics Endpoint

> **Exam Tip / Architectural Guardrail:** The `LIKE` operator executing inside the Fabric SQL Analytics Endpoint is **strictly case-sensitive**. Searching for `LIKE '%s%'` will completely miss uppercase characters like `"Sequel"`.
>
> To bypass this restriction safely, normalize both sides of the evaluation to a consistent casing format using the `UPPER()` or `LOWER()` functions:

```
SELECT * FROM purchasers WHERE LOWER(subtype) LIKE '%s%';
```

## Core Comparison Operators Summary Reference

| **Operator / Clause** | **Example Expression**        | **Functional Ingestion Behavior**                                     |
| --------------------- | ----------------------------- | --------------------------------------------------------------------- |
| **`=`**               | `WHERE subtype = 'Paper'`     | Evaluates exact case-sensitive text or exact values.                  |
| **`<>` or `!=`**      | `WHERE out != 7.99`           | Retains all rows except those that match the criteria.                |
| **`IN`**              | `WHERE out IN (10, 20, 30)`   | Matches any value inside the discrete list.                           |
| **`BETWEEN`**         | `WHERE out BETWEEN 5 AND 10`  | Inclusive filter spanning from 5 to 10.                               |
| **`LIKE`**            | `WHERE subtype LIKE '%book%'` | Executes wildcard text pattern matches.                               |
| **Functions**         | `WHERE LEN(subtype) > 5`      | You can wrap columns in functions directly inside the `WHERE` clause. |

# Aggregating and filtering data using the GROUP BY and HAVING clauses

> Group by - filtering using input field - ex. `out > 30`
>
> Having - filtering using output - ex. `SUM(out) > 30`

## Data Aggregation and the GROUP BY Rule

When you want to transition a dataset from transactional, row-level records down to a summarized view (such as finding total sales per payment type), you must use **Aggregate Functions**.

### The Core Aggregate Functions

- **`SUM()`**: Calculates the total numeric combined mass of a column.
- **`COUNT()`**: Returns the number of items or records.
- **`MIN()`**: Isolates the smallest value in the dataset.
- **`MAX()`**: Isolates the largest value in the dataset.
- **`AVG()`**: Computes the arithmetic mean (`SUM` divided by `COUNT`).

### The Non-Aggregated Column Rule

> **Exam Tip / Strict Syntax Rule:** If your `SELECT` clause contains a mix of **aggregated columns** (like `SUM(out)`) and **non-aggregated columns** (like `purchase_method`), you **must** include every single non-aggregated column inside the `GROUP BY` clause. Forgetting to do so will cause the query compiler to fail immediately and throw an evaluation error.

```
-- This syntax will FAIL because 'purchase_method' is missing from the GROUP BY clause:
SELECT purchase_method, SUM(out) FROM purchasers;

-- This syntax is CORRECT:
SELECT
    purchase_method,
    SUM(out) AS sum_out
FROM purchasers
GROUP BY purchase_method;
```

- **Alias Best Practice:** Aggregated mathematical columns return without a default header title (labeled as `untitled` or `No column name` in results panes). Always provide a meaningful header using the `AS` keyword (e.g., `AS sum_out`).

## Multi-Dimensional Grouping

You can group data by more than one descriptive attribute to break down summaries further.

```
SELECT
    purchase_method,
    subtype,
    SUM(out) AS total_out
FROM purchasers
GROUP BY purchase_method, subtype;
```

- If your database contains 12 raw rows, grouping by both columns might condense the output to 11 rows. This occurs because recurring dual combinations (e.g., two separate sales that both used `'Credit Card'` and `'eBook'`) collapse into a single unified row representing that combination's total sum.

## Counting Nuances: `COUNT(*)` vs. `COUNT(column)`

The way the `COUNT` function behaves shifts depending on the parameters you pass into its parentheses:

- **`COUNT(out)`**: Explicit Column Counting. The engine scans the specific column and counts a row **only if it contains a valid value**. Any row where the cell is `NULL` is completely ignored by the count.
- **`COUNT(*)`**: Wildcard Row Counting. This commands the database engine to count every row that passes the filter criteria, regardless of whether individual columns contain `NULL` values.

## Filtering Groups: WHERE vs. HAVING

A common point of confusion when writing SQL is knowing when to filter data using a `WHERE` clause versus a `HAVING` clause. They operate at completely different execution stages.

### 1. The `WHERE` Clause (Pre-Aggregation)

- Evaluates data **before** any rows are grouped or aggregated.
- It operates directly on the raw row attributes of the table.
- **Strict Constraint:** You can **never** place an aggregate function inside a `WHERE` clause (e.g., `WHERE SUM(out) > 40` will crash immediately).

### 2. The `HAVING` Clause (Post-Aggregation)

- Evaluates data **after** the `GROUP BY` clause has processed and compressed the rows.
- It is used specifically to filter out summarized groups based on calculated aggregate metrics.

## Combining WHERE, GROUP BY, and HAVING

You can safely combine both filtering methods within a single query block to build highly refined data pipelines.

### The Standard Query Clause Order

To maintain correct syntax, your query clauses must follow this specific order:

1. `SELECT`
2. `FROM`
3. `WHERE` (Filters raw transactional rows first)
4. `GROUP BY` (Collapses the remaining rows into summary metrics)
5. `HAVING` (Filters out summarized groups that do not meet aggregate criteria)

### Comprehensive Scenario Example

Consider a dataset containing 12 transactional records. We want to find the payment methods where the total spend exceeds $40, but we want to ignore any individual transaction smaller than $6.00:

```
SELECT
    purchase_method,
    SUM(out) AS total_amount
FROM purchasers
**WHERE out > 6.00**                -- Step 1: Drops small individual receipts first
GROUP BY purchase_method        -- Step 2: Clusters remaining records by payment type
**HAVING SUM(out) > 40.00;**        -- Step 3: Drops any summarized cluster totaling under $40
```

- **Execution Impact:** Adding the `WHERE` filter dynamically reduces the internal values _before_ they are summed up. For example, a `Credit Card` total might drop from $47.17 down to $37.98 because its small transactions were filtered out. Consequently, that payment method would be eliminated by the subsequent `HAVING` clause because it no longer exceeds the $40.00 threshold.

## Summary Reference Table

| **Clause**     | **Structural Purpose**             | **Allowed Expressions**                           | **Execution Timing**           |
| -------------- | ---------------------------------- | ------------------------------------------------- | ------------------------------ |
| **`WHERE`**    | Filters raw input records          | Raw column names, comparisons, text patterns      | **Before** `GROUP BY` runs     |
| **`GROUP BY`** | Segments data into summarized rows | Every non-aggregated column present in `SELECT`   | Intermediate stage             |
| **`HAVING`**   | Filters output summary blocks      | Aggregate functions (`SUM`, `COUNT`, `MAX`, etc.) | **After** `GROUP BY` completes |

# Ordering data and retrieving a LIMITed number of rows

> notebook: `LIMIT5`
>
> SQL endpoint: `TOP 5`

## Sorting Datasets with ORDER BY

The `ORDER BY` clause controls the sorting order of rows returned by your query. By default, queries return rows in no guaranteed structural sequence unless an explicit ordering clause is defined.

### 1. Ascending vs. Descending Order

- **Ascending (`ASC`):** Sorts numbers from smallest to largest and text alphabetically (A to Z). This is the automatic default behavior; explicitly typing `ASC` is optional.
- **Descending (`DESC`):** Sorts numbers from largest to smallest and text in reverse-alphabetical order (Z to A).

### 2. Multi-Column Sorting Hierarchy

You can pass multiple comma-separated columns into a single `ORDER BY` clause to create sequential sorting tiers. The engine sorts by the first designated column first, and then breaks ties by sorting within those subsets using the second column:

```
SELECT purchase_method, subtype, out
FROM purchasers
ORDER BY purchase_method DESC, subtype ASC;
```

- In this example, the rows are cleanly grouped together by `purchase_method` in descending order (e.g., Electronic, then Credit Card, then Cash). Within the Credit Card subset, individual entries are neatly listed by `subtype` in alphabetical ascending order.

## Restricting Row Counts: TOP vs. LIMIT

When you need to isolate extreme boundaries—such as extracting only the top 5 highest transactional amounts—you must combine `ORDER BY` with a row-limiting keyword.

> **Exam Tip / Architectural Split:** The specific keyword required to restrict rows depends entirely on the Fabric environment you are executing queries in.

| **Query Interface**                | **Keyword Used** | **Structural Placement**                               | **Sample Syntax**                                       |
| ---------------------------------- | ---------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| **SQL Analytics Endpoint** (T-SQL) | **`TOP`**        | Placed inside the `SELECT` clause before column lists. | `SELECT TOP 5 out FROM purchasers ORDER BY out DESC;`   |
| **Fabric Notebook** (Spark SQL)    | **`LIMIT`**      | Placed at the very end of the complete query block.    | `SELECT out FROM purchasers ORDER BY out DESC LIMIT 5;` |

- _Note:_ Attempting to use `LIMIT` in the SQL Analytics Endpoint or `TOP` inside a Spark SQL notebook cell will cause validation errors and fail execution.

## The Six Principal Clauses of a SELECT Statement

When building complete end-to-end relational data pipelines, your queries must strictly follow this specific architectural clause order:

```jsx
1. SELECT   ──> Defines output columns or scalar values
2. FROM     ──> Connects to the data source artifact
3. WHERE    ──> Filters individual raw transactions (Pre-aggregation)
4. GROUP BY ──> Clusters rows based on non-aggregated attributes
5. HAVING   ──> Filters aggregated group rows (Post-aggregation)
6. ORDER BY ──> Arranges the final dataset visually
```

### The Alias Visibility Rule

Understanding where column **Aliases** (assigned using the `AS` keyword in the `SELECT` clause) can be referenced is a highly tested foundational concept:

- **The Restriction:** You **cannot** use an alias inside the `WHERE`, `GROUP BY`, or `HAVING` clauses. Because of how the database engine sequentially compiles the query, it evaluates those filtering and grouping steps _before_ it processes the names inside the `SELECT` line. You must use the full underlying calculation or column name instead.
- **The Exception:** You **can** freely use an alias inside the `ORDER BY` clause, as it is processed at the very end of execution.

### Comprehensive Example Implementing All Clauses:

```sql
**SELECT**
    purchase_method,
    SUM(out) AS sum_out              -- Alias definition
**FROM** purchasers
**WHERE** out > 6.00                     -- Clause 3: Raw transactional filter
**GROUP BY** purchase_method             -- Clause 4: Grouping rule
**HAVING** SUM(out) > 38.00              -- Clause 5: Group-level aggregate filter (Alias NOT allowed here)
**ORDER BY** sum_out DESC;               -- Clause 6: Final sort (Alias IS allowed here)
```

## Memory Shortcut: Remembering the Syntax Order

To easily memorize the structural compilation flow of a full SQL statement, you can look directly down at your computer keyboard. Omitting the `WHERE` clause, the first letters of the remaining core clauses track sequentially from left to right across the standard home row and surrounding keys:

- **S**elect $\rightarrow$ **F**rom $\rightarrow$ [Where] $\rightarrow$ **G**roup by $\rightarrow$ **H**aving $\rightarrow$ **O**rder by
- **S** $\rightarrow$ **F** $\rightarrow$ **G** $\rightarrow$ **H** $\rightarrow$ **O**

### Mandatory vs. Optional Clauses

- **`SELECT`** is the only single clause that is mathematically mandatory to form a valid query (e.g., executing `SELECT 1+1` functions perfectly on its own).
- To pull database values, you must pair it with a **`FROM`** clause.
- All other components (`WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`) are optional structural building blocks added sequentially as needed. You can only deploy a `HAVING` clause if an active `GROUP BY` operation is present.

# Quiz: Creating SQL queries using the six principle clauses of the SQL statement

!image.png

!image.png

!image.png

!image.png

# 23. Transform data in a lakehouse

# Identifying and resolving duplicate data

## 1. Resolving Duplicates via SQL: `DISTINCT` vs. `GROUP BY`

When a query evaluates a column containing recurring operational categories (e.g., pulling `purchase_method` from the `purchases` table), it returns every transaction row by default. To strip out repeating duplicates and compress the output to show unique entities, you can deploy two different structural SQL approaches:

### Option A: The `DISTINCT` Modifier

Placing the keyword `DISTINCT` directly after `SELECT` evaluates rows and drops matching duplicates.

```sql
SELECT **DISTINCT** purchase_method FROM purchasers;
```

- **Multi-Column Evaluation Behavior:** If you include multiple columns after the modifier (e.g., `SELECT DISTINCT purchase_method, subtype`), the engine does **not** look for distinct payment methods alone. Instead, it evaluates the **combined uniqueness** of the entire row.
- If your dataset has 12 raw records, but two rows contain the exact same pairing (e.g., two entries that both say `'Credit Card'` and `'eBook'`), the query compresses those two records into a single row, returning 11 distinct rows total.

### Option B: The `GROUP BY` Suffix

If you write a `GROUP BY` statement without attaching any summary aggregate calculations to the columns, it functions exactly like `DISTINCT` by grouping matching entries onto a single row:

```sql
SELECT purchase_method, subtype
FROM purchasers
**GROUP BY** purchase_method, subtype;
```

## 2. Auditing and Identifying Duplicates with `HAVING`

Before globally deleting records, an analyst must verify which combinations are duplicated and inspect how many times they repeat. To build an auditing pipeline, combine `GROUP BY`, `COUNT(*)`, and `HAVING`.

```sql
SELECT
    purchase_method,
    subtype,
    COUNT(*) AS occurrence_count
FROM purchasers
GROUP BY purchase_method, subtype
**HAVING COUNT(*) > 1;**                  -- Post-aggregation filter to find duplicates
```

### Explanation of the Auditing Sequence

1. **`GROUP BY purchase_method, subtype`** clusters the table into unique structural combinations.
2. **`COUNT(*)`** scans each cluster and calculates the exact number of times that specific pairing occurs in the raw source data.
3. **`HAVING COUNT(*) > 1`** runs at the very end of compilation. It drops any clean, well-behaved row that only appears once (`COUNT = 1`) and isolates **only** the broken rows that repeat multiple times (e.g., revealing that the combination of `'Credit Card'` and `'eBook'` has an occurrence count of `2`).

## 3. Resolving Duplicates via Low-Code Dataflows Gen2

If you prefer clean visual workflows or are developing early data transformation steps inside a Dataflow Gen2 canvas, you can mirror these exact SQL methods using two visual UI paths:

- **Path 1: The Reduce Rows Tool (The `DISTINCT` Mirror)**
  1. Highlight the target columns in the data preview grid.
  2. Navigate to the **Home** tab ribbon.
  3. Locate the **Reduce Rows** section, click **Remove Rows**, and choose **Remove Duplicates**. Fabric records this as a clean step that immediately drops repeating rows.
- **Path 2: The Transform Group By Tool (The `GROUP BY` Mirror)**
  1. Go to either the **Home** or **Transform** ribbons and click **Group By**.
  2. Map out your group-by schema columns. This operation collapses duplicate rows down to a single unique combination row.

## Technical Comparison Summary

| **Technical Approach**                 | **Data Environment**               | **Core Analytical Goal**                           | **Unique Action**                                            |
| -------------------------------------- | ---------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **`SELECT DISTINCT`**                  | SQL Analytics Endpoint / Notebooks | Rapid, code-based cleanup of presentation results. | Evaluates combined row uniqueness.                           |
| **`GROUP BY` empty aggregation**       | SQL Analytics Endpoint / Notebooks | Relational database processing.                    | Collapses matches down into a single row.                    |
| **`GROUP BY` + `HAVING COUNT(*) > 1`** | SQL Analytics Endpoint / Notebooks | **Auditing and Discovery**.                        | Isolates and exposes _only_ the broken rows that repeat.     |
| **Reduce Rows > Remove Duplicates**    | Dataflow Gen2 (Power Query)        | Low-code ETL visual step building.                 | Drops duplicate rows sequentially based on selected columns. |

# Merging data with a UNION

## The SQL UNION Suite: Appending Rows Vertically

In relational databases, a **UNION** operation stacks the row outputs of two or more independent `SELECT` statements vertically into a single unified dataset. This is the direct code equivalent of the low-code **Append** operation in Power Query / Dataflows.

### Structural Requirements for a Valid Union

To execute any vertical join successfully, your underlying `SELECT` queries must strictly adhere to two structural laws:

1. **Column Count Equality:** Every `SELECT` statement in the chain must contain the **exact same number of columns**.
2. **Data Type Compatibility:** The columns occupying the same relative positions across the statements must share compatible data types (e.g., you cannot stack a text string column directly on top of an integer column without causing a mismatch error).

## UNION vs. UNION ALL

Choosing between `UNION` and `UNION ALL` alters both your final row count and the processing overhead on the Fabric engine.

```sql
UNION ALL (Fast & Retains Everything):
Table A (12 Rows) ──┐
                    ├──> Stacks Directly ──> 14 Rows Total
Table B (2 Rows)  ──┘

UNION (Evaluates and Deduplicates Row-by-Row):
Table A (12 Rows) ──┐
                    ├──> Scans for Identical Rows ──> Drops Duplicates ──> 11 Rows Total
Table B (2 Rows)  ──┘
```

- **`UNION ALL`:** Ingests all rows from all tables and stacks them directly. It **retains all duplicate records** and executes extremely fast because the engine does not inspect the underlying data. (e.g., combining 12 rows from `purchases` and 2 rows from `purchases_april` yields exactly `14` rows).
- **`UNION`:** Combines the data and then actively scans the entire output dataset to **remove completely duplicate rows**. Two rows must match identically across _every single column position_ to be flagged as a duplicate and stripped away. This deduplication pass adds significant computational overhead.

## Architectural Rules: Metadata and Conversions

### 1. Column Header Rule

The final output dataset will exclusively inherit its column header names from the **first `SELECT` statement** in the code string.

- _Scenario:_ If Table 1's fourth column is named `out` and Table 2's fourth column is named `amount`, the resulting union column header will be named **`out`**.

### 2. Implicit Data Type Resolution

If there is a slight data type discrepancy between columns in the same position (such as a formal `date` schema column in the first query and a `string/varchar` column in the second query), the Fabric engine initiates an **implicit conversion**. It automatically evaluates the entire vertical column stream and forces a matching, unified data type (like `date`) for all 14 rows, regardless of which table was placed first in the script text.

### 3. Handling Mismatched Table Schemas (The NULL Filler)

If Table 2 contains an extra column (such as a fifth column named `details`) that does not exist in Table 1, you will trigger a column mismatch failure if you attempt a straight `SELECT *`.

To bypass this schema barrier cleanly, you must manually explicitly declare a mock column inside the first query using the **`NULL`** keyword as a structural filler layout placeholder:

```sql
-- Query 1 (12 Rows): Injects manual NULL placeholder matching the 'details' schema
SELECT
    date,
    subtype,
    purchase_method,
    out,
    **NULL AS details**                 -- Filler column matching the schema
FROM purchasers

**UNION ALL**

-- Query 2 (2 Rows): Contains the actual operational string column
SELECT
    date,
    subtype,
    purchase_method,
    **amount**,                         -- Inherits header name 'out' from Query 1
    **details**
FROM purchasers_april;
```

## Low-Code Execution: Dataflows Gen2

If you are developing transformations visually within a Dataflow Gen2 canvas instead of writing SQL queries, you achieve this vertical union by navigating to:

1. Open the **Home** tab ribbon.
2. Locate the **Combine** cluster group.
3. Click **Append Queries** (or _Append Queries as New_). This visual wizard stacks your targeted Lakehouse tables vertically into a single output grid.

## Operator Summary Reference

| **SQL Operator** | **Performance Speed**                      | **Row-Level Behavior**                                  | **Column Header Source**            |
| ---------------- | ------------------------------------------ | ------------------------------------------------------- | ----------------------------------- |
| **`UNION ALL`**  | **Fastest** (No processing overhead)       | Retains 100% of rows, including duplicates.             | Inherited from the **first** query. |
| **`UNION`**      | **Slower** (Requires a deduplication pass) | Deletes rows that match identically across all columns. | Inherited from the **first** query. |

# JOINing data together

## Horizontal Combinations: Joins vs. Unions

When pulling multiple database entities together, you must architecturally distinguish between horizontal expansion and vertical stacking.

- **UNION (Vertical Stack):** Stacks records directly underneath one another, expanding the dataset's **row count** while keeping column counts matching.
- **JOIN (Horizontal Merge):** Merges tables side-by-side using a shared relational key column. This expands the dataset's schema by adding **new columns** of secondary attributes to existing rows.

## Table Aliasing and the Query Execution Order

When joining tables, columns with the same name across both sources (such as a shared identifier column like `subtype`) will cause your script to crash due to a **Column Ambiguity Error**. To bypass this layout obstacle, you must explicitly prefix the target columns using **Table Aliases**.

```sql
-- Explicit mapping using table aliases 'p' and 'pc'
SELECT
    p.date,
    p.subtype AS purchases_subtype,        -- Resolving ambiguity via alias
    p.purchase_method,
    p.out,
    pc.category,
    pc.subtype AS category_subtype
FROM purchases AS p
**INNER JOIN purchases_category AS pc
    ON p.subtype = pc.subtype;**
```

### The Selection Illusion

An alias assigned inside a `FROM` or `JOIN` clause is immediately visible to the preceding `SELECT` clause line. While `SELECT` is physically written first in your script, the back-end compute engine follows a strict sequential execution path:

$$
\text{FROM} \longrightarrow \text{JOIN} \longrightarrow \text{WHERE} \longrightarrow \text{GROUP BY} \longrightarrow \text{HAVING} \longrightarrow \mathbf{SELECT} \longrightarrow \text{ORDER BY}
$$

Because the data source context is parsed long before the target projection attributes are isolated, your `SELECT` statement can safely rely on data aliases declared down inside the `FROM` block.

## The 4 Relational Join Types

The chosen join modifier alters the total row granularity of your query output grid based on how it resolves missing, unmapped, or orphan data keys.

```sql
INNER JOIN (Matches Only):
Purchases (12 Rows) ──────┐
                          ├──> Retains perfect intersections only ──> 11 Rows
Purchases_Cat (11 Rows) ──┘    (Drops unmapped rows from both sides)

LEFT JOIN (Retains all primary rows):
Purchases (12 Rows) ──────┐
                          ├──> Keeps 100% Left Rows ──> 14 Rows Total
Purchases_Cat (11 Rows) ──┘    (Pads missing Right metrics with NULL)
```

### 1. `INNER JOIN` (The Implicit Default)

- **Ingestion Action:** Retains rows **only** when a perfect matching key is found simultaneously in both tables. If a transaction contains a corrupted placeholder key (like `?`) missing from the catalog table, that row is omitted.
- _Note:_ If you drop the descriptive word `INNER` and simply write `JOIN`, the query compiler defaults to an Inner Join.

### 2. `LEFT JOIN` (Left Outer)

- **Ingestion Action:** Preserves 100% of the rows from the first table declared in the query text (the left side), regardless of whether they find a corresponding match in the secondary lookup table (the right side). Any row missing a matching right-side mapping has its empty target columns padded with `NULL`.

### 3. `RIGHT JOIN` (Right Outer)

- **Ingestion Action:** Preserves 100% of the rows from the second table declared in the query text (the right side). Unmatched attributes originating from the transactional left table are systematically populated with `NULL` rows.
- _Architectural Identity:_ Any `LEFT JOIN` can be re-engineered into an identical `RIGHT JOIN` simply by inverting the physical sequencing order of the target tables inside your SQL text block.

### 4. `FULL JOIN` (Full Outer)

- **Ingestion Action:** Captures the entire structural layout of both systems. It merges matching indices perfectly, retains unmapped rows from the left table (padded with right-side nulls), and appends unmapped categories from the right table (padded with left-side nulls).

## The Cartesian Boundary: `CROSS JOIN`

A **`CROSS JOIN`** computes the full mathematical Cartesian product of two tables.

- **Behavior:** It binds every single individual row from Table A directly to every single individual row in Table B. Because this maps every record globally against everything else, it **does not utilize an `ON` clause constraint statement**.
- **Row Granularity Multiplier:** The row footprint scales via straight multiplication:
  $$\text{Total Output Rows} = (\text{Rows in Table A}) \times (\text{Rows in Table B})$$
- _Scenario:_ Cross-joining a 12-row transaction table against an 11-row category table generates a massive output footprint of $12 \times 11 = 132$ rows. This operation is rarely implemented in enterprise production analytics workloads due to severe memory overhead.

## Analytical Ingestion Syntax Chain

When scaling your analytical queries to pull data across three or more disparate entities simultaneously, you chain sequential join operations linearly. To keep your syntax clean, follow this structural execution chain order:

```sql
SELECT columns
FROM table_1 AS t1
[INNER/LEFT/RIGHT] JOIN table_2 AS t2 ON t1.key = t2.key
[INNER/LEFT/RIGHT] JOIN table_3 AS t3 ON t2.alt_key = t3.alt_key
WHERE raw_filters
GROUP BY grouping_attributes
HAVING aggregate_filters
ORDER BY visual_sorting;
```

## Low-Code Equivalency: Dataflows Gen2

To combine tables horizontally without writing relational SQL statements, build this transformation step visually inside a Dataflow Gen2 asset canvas:

1. Open the **Home** tab ribbon.
2. Locate the **Combine** cluster group and select **Merge Queries**.
3. Map the matching structural key columns between your primary and secondary grid visualizations.
4. Select your target **Join Kind** (Left Outer, Right Outer, Full Outer, Inner, Left Anti, or Right Anti) from the visual dropdown selector interface.

# Handling missing data and Null values

## The Nature of NULL: The Equality Trap

In relational databases and Microsoft Fabric's execution engines, **`NULL` represents the absolute absence of data or an unknown state**. It is structurally distinct from a zero (`0`) or an empty string text field (`''`).

> **Exam Tip / Golden Rule:** Because `NULL` is mathematically treated as an unknown value, **you can never use the standard equality operator (`=`) or inequality operator (`<>`) to test for it.** Asking if an unknown value equals another unknown value results in an unknown state, causing your query to return zero rows.

```sql
-- This syntax will FAIL to find null records:
SELECT * FROM purchasers WHERE category = NULL;

-- This syntax is CORRECT:
SELECT * FROM purchasers **WHERE category IS NULL;**
```

## Filtering Rows: `IS NULL` vs. `IS NOT NULL`

To horizontally isolate or filter out records based on missing data points within your `WHERE` clause, rely on explicit relational testing operators:

- **`IS NULL`:** Isolates rows where a cell is explicitly empty or unmapped. For example, filtering a joined dataset `WHERE category IS NULL` isolates all transactional product lines that completely lack a matching category record in your master data lookup catalog.
- **`IS NOT NULL`:** Retains only clean, completely populated rows, filtering out any missing records.
- _Alternative Syntax:_ While you can technically write `WHERE NOT (purchase_method IS NULL)`, using the dedicated **`IS NOT NULL`** operator is the industry standard and significantly improves execution clarity.

## Coalescing Data: Resolving Gaps and Backups

When merging tables horizontally, you frequently end up with two separate version tracks of a similar column—such as a `purchases_subtype` from your transactional ledger and a `category_subtype` from your central dimension lookup, each containing scattered `NULL` values where data went unrecorded.

To collapse these columns down into a single, unified, gap-free column, use the **`COALESCE`** function.

```sql
SELECT
    p.subtype AS purchases_subtype,
    pc.subtype AS category_subtype,
    **COALESCE(p.subtype, pc.subtype, 'Completely Unknown') AS unified_subtype**
FROM purchases AS p
FULL JOIN purchases_category AS pc
    ON p.subtype = pc.subtype;
```

### The Coalesce Evaluation Protocol

The `COALESCE` function accepts an arbitrary number of comma-separated arguments and evaluates them sequentially from left to right. It strictly **returns the very first non-null value** it encounters in the sequence:

1. It evaluates argument 1 (`p.subtype`). If it contains data, it instantly outputs that value and moves to the next row.
2. If argument 1 is `NULL`, it immediately evaluates argument 2 (`pc.subtype`). If argument 2 has data, it outputs it.
3. If all column arguments evaluate to `NULL` for a row, you can pass a hardcoded fallback string literal at the very end (e.g., `'Completely Unknown'`) to serve as a clean global default placeholder.

## Null-Handling Logic Summary Reference

| **Syntax Operation**      | **Structural Placement** | **Primary Data Purpose**                                | **Core Architectural Impact**                              |
| ------------------------- | ------------------------ | ------------------------------------------------------- | ---------------------------------------------------------- |
| **`IS NULL`**             | `WHERE` / `HAVING`       | Identifies missing records or orphan join gaps.         | Retains only empty target records.                         |
| **`IS NOT NULL`**         | `WHERE` / `HAVING`       | Filters out incomplete transactions.                    | Retains only 100% complete records.                        |
| **`COALESCE(A, B, ...)`** | `SELECT`                 | Blends overlapping columns into a single unified field. | Sequentially grabs the first **non-null** value available. |

# Conditional functions

## 1. The Power of CASE Expressions

The `CASE` statement is SQL's most flexible conditional construct. It evaluates a list of conditions sequentially and returns a value as soon as it hits the **first true condition**. Once a condition is met, processing for that row stops.

There are two distinct structural types of `CASE` statements:

### Type A: Searched CASE (High Flexibility)

Allows you to use complex logical expressions, inequality operators (`>`, `<`, `<>`), or multiple conditions per branch using `AND`/`OR`.

```sql
SELECT
    out,
    **CASE
        WHEN out > 100.00 THEN 'Premium'
        WHEN out BETWEEN 40.00 AND 100.00 THEN 'Standard'
        WHEN out IS NULL THEN 'Unbilled'
        ELSE 'Budget'
    END** AS tier_classification
FROM purchasers;
```

> **Syntax Rules:** Every `CASE` block **must** begin with the keyword `CASE` and terminate explicitly with **`END`**. The `ELSE` branch handles the fallback logic if no prior condition evaluates to true. If you omit the `ELSE` branch entirely, an implicit `ELSE NULL` is automatically injected by the query engine.

### Type B: Simple CASE (Exact Match Only)

Evaluates a single target column or expression against specific, discrete static values. It cannot evaluate inequalities or range queries.

```sql
SELECT
    subtype,
    **CASE subtype
        WHEN 'misc'  THEN 'Miscellaneous'
        WHEN 'other' THEN 'Miscellaneous'
        ELSE subtype
    END** AS cleaned_subtype
FROM purchasers;
```

## 2. Low-Code/Shorthand Conditionals: `IIF`

For quick, binary branch choices (True or False operations), Fabric SQL supports the **`IIF`** function (spelled with a double "I"). This functions identically to an `IF` statement or the `IF()` formula in Microsoft Excel.

```sql
-- IIF Syntax: IIF(condition, value_if_true, value_if_false)
SELECT
    **IIF(subtype IN ('misc', 'other'), 'Miscellaneous', NULL)** AS group_type
FROM purchasers;
```

## 3. Positional Evaluation with `CHOOSE`

The `CHOOSE` function acts as a numeric index selector map. It accepts an integer value as its first argument and uses it to select the corresponding item from an ordered list of string or value parameters.

```sql
-- Returns the 3rd item in the list: 'Item C'
SELECT **CHOOSE(3,** 'Item A', 'Item B', 'Item C', 'Item D') AS selected_index;
```

- **Constraint:** The first parameter **must strictly evaluate to a whole number integer**. Floating-point decimals or fractional scales are not supported.

## 4. Scalar Boundary Scans: `GREATEST` and `LEAST`

When you need to perform an inline comparison across **multiple columns within the same row** (rather than compiling down the height of a single column), use `GREATEST` and `LEAST`.

```
-- Returns the highest and lowest scalar boundaries across the given variables
SELECT
    **GREATEST**(1, 2, 3) AS max_boundary,  -- Output: 3
    **LEAST**(1, 2, 3)    AS min_boundary   -- Output: 1
FROM purchasers;
```

- **Use Case:** They accept lists of numeric or date variables to instantly pick the maximum or minimum horizontal value per row (e.g., comparing `ship_date` vs `delivery_date`).

## Conditional Logic Shorthand Summary

| **Function Structure**   | **Input Requirements**                       | **Structural Behavior**                               | **Best Used For**                                                      |
| ------------------------ | -------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| **`CASE WHEN [Logic]`**  | Boolean conditional strings                  | Returns value on the **first** true match.            | Highly complex or inequality-based multi-tier routing.                 |
| **`IIF(Cond, T, F)`**    | Condition, True return, False return         | Evaluates a clean binary path layout.                 | Quick, low-code shorthand for true/false queries.                      |
| **`CHOOSE(Index, ...)`** | Non-fractional base integer, item list       | Looks up an item based on its list position index.    | Mapping static sequence lists (e.g., matching months 1–4 to quarters). |
| **`GREATEST` / `LEAST`** | Collection of numbers, dates, or expressions | Compares parameters horizontally across a single row. | Picking out extreme horizontal boundaries across columns.              |

# Quiz 19: Transform data in a lakehouse

!image.png

!image.png

!image.png

!image.png

!image.png

# 24. Transform data in data warehouse

# Creating tables in a data warehouse

## Architectural Guardrail: Lakehouse vs. Warehouse DDL

A fundamental structural distinction on the DP-600 exam is how the SQL engine interacts with a Lakehouse versus a Data Warehouse.

- **Lakehouse SQL Analytics Endpoint (Read-Only):** The SQL engine here is **strictly read-only**. Attempting to execute write commands like `CREATE TABLE`, `DROP TABLE`, or `CREATE SCHEMA` will fail. Neither the endpoint nor a Spark SQL notebook cell supports relational DDL statements for structural database adjustments.
- **Data Warehouse (Read/Write):** The Data Warehouse fully supports full transactional DDL and DML (Data Manipulation Language). You can dynamically create, modify, and drop schemas and tables using standard T-SQL scripts.

## Database Schemas and Namespace Isolation

By default, any table created without a specified prefix is assigned to the **`dbo`** (Database Owner) schema namespace. However, you can create custom schemas to cleanly isolate target tables by department, environment, or data layer.

### Creating a Schema

To generate a custom schema container, click the ellipsis (`...`) next to the Schemas folder in the Explorer pane or run a direct DDL query:

```sql
CREATE SCHEMA staging;
```

## Defensive Coding: Table Drop and Creation Workflow

When automating table creation inside pipeline scripts, running a raw `CREATE TABLE` command when the table already exists in the metadata catalog will cause the query to crash. Conversely, running a raw `DROP TABLE` on a table that does not exist will also throw an error.

To build an idempotent, fault-tolerant execution script, implement the **`DROP TABLE IF EXISTS`** pattern:

```sql
-- Step 1: Safe cleanup without throwing errors if the table is missing
**DROP TABLE IF EXISTS dbo.tblTarget**;

-- Step 2: Clean structural definition
**CREATE TABLE dbo.tblTarget** (
    country VARCHAR(20),
    type    VARCHAR(20),
    target  INT
);
```

### Text Storage Optimization: `CHAR(n)` vs. `VARCHAR(n)`

When choosing a character schema type, consider how OneLake allocates storage space under the hood:

- **`CHAR(20)` (Fixed-Length):** Consistently reserves exactly 20 characters of space for every single row. If an entry is shorter (e.g., `"Ireland"`), the engine right-pads the remaining 13 characters with trailing blank spaces.
- **`VARCHAR(20)` (Variable-Length):** Dynamically scales down to fit the exact length of the incoming text data string. It only uses storage space proportional to the characters provided, making it highly recommended for varying text fields.

## Data Type Comparison: SQL vs. PySpark

Data type boundaries can vary slightly between the Spark notebook environment and the Warehouse T-SQL relational engine.

### 1. The `TINYINT` Discrepancy

- **Fabric SQL Warehouse:** Follows the unsigned integer standard. It stores strictly non-negative numbers ranging from **`0` to `255`**.
- **PySpark (Notebooks):** Utilizes a signed data type model, allowing values to span from **`128` to `+127`**.

### 2. Precise Floating-Point Strategy

- For financial metrics or values requiring absolute precision, implement **`DECIMAL(precision, scale)`** or **`NUMERIC`**. For instance, `DECIMAL(10,2)` permits a maximum of 10 total digits, with exactly 2 digits allocated after the decimal point.
- Avoid using approximate types like `FLOAT` or `REAL` (SQL) / `DOUBLE` (PySpark) for precise columns, as their underlying binary estimation framework can introduce minor floating-point errors during massive aggregation steps.

### Data Type Mapping Reference

| **Architectural Classification** | **Fabric SQL Data Warehouse**                     | **Notebook PySpark Engine**                  |
| -------------------------------- | ------------------------------------------------- | -------------------------------------------- |
| **Whole Numbers**                | `BIGINT`, `INT`, `SMALLINT`, `TINYINT` (0 to 255) | `long`, `int`, `short`, `byte` (-128 to 127) |
| **Exact Decimals**               | `DECIMAL(p,s)` or `NUMERIC`                       | `DecimalType`                                |
| **Approximate Fractions**        | `FLOAT`, `REAL`                                   | `float`, `double`                            |
| **Boolean Flags**                | `BIT` (Stores `0` or `1`)                         | `boolean`                                    |
| **Text Strings**                 | `VARCHAR(n)`, `CHAR(n)`                           | `string`                                     |
| **Temporal Timestamps**          | `DATETIME2`, `DATE`, `TIME`                       | `timestamp`, `date`                          |

# Inserting data into tables and transforming data

## 1. DML Insert Operations: Populating Tables

Once a schema layout is defined via DDL (`CREATE TABLE`), you use Data Manipulation Language (DML) commands to inject transactional records into the Data Warehouse.

### The `INSERT INTO` Syntax Pattern

- The `INSERT INTO` statement identifies the target table and the specific structural columns you intend to populate.
- The `VALUES` clause maps the raw input data. You must wrap string literals (e.g., `'London'`, `'England'`) in **single quotation marks** while passing integer metrics directly.

```sql
-- Inserting a single record
**INSERT INTO** tblActual (country, location, actual)
**VALUES** ('England', 'London', 5000);

-- Bulk Inserting multiple records simultaneously
**INSERT INTO** tblTarget (country, type, target)
**VALUES**
    ('England', 'store', 10000),
    ('France',  'store',  8000),
    ('Italy',   'online', 3000);
```

> **Syntax Rule:** Terminating `INSERT` statements with a semicolon (`;`) is best practice but remains functionally optional unless you are chaining advanced CTEs (Common Table Expressions) using the `WITH` clause downstream.

## 2. Generating Persistent Output Tables (CTAS & INTO)

When you construct an aggregated summary view (such as grouping sales by country), you often need to save the output permanently as a new physical table rather than just displaying a temporary grid.

There are three ways to materialize a `SELECT` query into a physical table within Fabric:

### Method 1: The UI "Save As Table" Tool

1. Write and highlight a complete, functional `SELECT` statement in the SQL query editor.
2. Click the **Save as table** ribbon button above the query results.
3. Configure the destination name (e.g., `actual_sum`) and select the target workspace/warehouse. The engine will run a background job to orchestrate the table creation.

### Method 2: The UI Object Explorer (Auto-Generate Script)

If you want the system to write the code for you based on an existing table:

1. Locate the source table in the left Explorer pane.
2. Click the ellipsis (`...`) and select **Script as > Create table AS Select (CTAS)**.

### Method 3: The Native T-SQL `INTO` Clause (Fastest)

The most efficient, programmatic way to generate and populate a brand-new table in a single step is injecting the **`INTO`** keyword directly inside your `SELECT` query block (positioned strictly between the column selector and the `FROM` clause).

```sql
-- Creates a new table named 'target_sum' and populates it immediately
SELECT
    country,
    SUM(target) AS total_target
**INTO target_sum**
FROM tblTarget
GROUP BY country;
```

## 3. Native Data Exploration Tools

Once your tables are populated, you can interact with the results exactly as you would in the Lakehouse SQL Analytics Endpoint.

- **Grid Interaction:** Clicking directly on a table in the left explorer pane loads a top-row preview where you can visually sort descending/ascending or use the global text search filter.
- **Open in Excel:** Highlighting a `SELECT` statement allows you to push the dynamic results grid out to a live-connected Excel workbook.
- **Explore This Data:** Highlighting a query and clicking this option instantly spins up a temporary Power BI visual canvas. You can drag dimensions (like `country`) and aggregated facts (like `total_target`) onto the pane to build quick visual charts, which you can then save permanently as a standard Fabric Report.

## Quick UI Command Reference

| **Action**                 | **Left-Pane Object Explorer UI Steps** |
| -------------------------- | -------------------------------------- |
| **Drop/Delete Table**      | Click `...` > **Drop**                 |
| **Clear & Recreate Table** | Click `...` > **Drop and Create**      |
| **Query Top Rows**         | Click `...` > **Select Top 100 Rows**  |

# Creating a running total in SQL

Here is the detailed breakdown note on implementing **SQL Window Functions** to calculate running totals, using the `OVER`, `PARTITION BY`, and `ORDER BY` clauses, structured for your DP-600 exam preparation.

## The Concept of Window Functions and the `OVER` Clause

To calculate a cumulative metric (like a running financial total) across rows without collapsing the dataset down into a single summary row, you must use **Window Functions**.

A standard `SUM()` combined with a traditional `GROUP BY` collapses individual transactional rows into single category totals. To retain all original detail rows while dynamically computing a running calculation along them, replace the `GROUP BY` clause with the **`OVER`** clause.

```sql
-- Basic Window Function Syntax Structure
SELECT
    country,
    location,
    actual,
    **SUM(actual) OVER(ORDER BY country, location)** AS running_total
FROM tblActual;
```

## Controlling Calculation Flow: `ORDER BY` inside `OVER`

The `ORDER BY` clause nested inside the `OVER()` parentheses does **not** control the final visual arrangement of the final query results grid. Instead, it serves as the calculation anchor that teaches the execution engine how to step through the rows to accumulate data sequentially.

### 1. Determining the Sequence

The execution engine must know what constitutes the "first row," "second row," and so on, to accurately add values step-by-step. Without an inline `ORDER BY` statement inside the `OVER` clause, the engine cannot compute an incremental running total.

### 2. Resolving Ties (The Tiebreaker Rule)

If you only order your window function calculation by a high-level category with repeating entries (like `ORDER BY country`), the engine may encounter duplicate sorting ties. When multiple lines match exactly (e.g., three separate entries for `'England'`), the calculation can aggregate all tied rows simultaneously rather than line-by-line.

To prevent calculation ties and enforce a smooth, row-by-row cumulative step, always introduce a unique column attribute (such as a transactional `location` or a primary key ID) as a **Tiebreaker**:

```sql
-- Order calculation by country first, breaking ties using location
SUM(actual) OVER(ORDER BY country ASC, location ASC)
```

### 3. Separating Calculation Order from Presentation Order

The internal sorting flow of your window logic can run entirely independent of your query's final external display sort order.

- _Example:_ You can calculate a running total that loops alphabetically from A to Z, but visually display the final results pane upside down from Z to A.

```sql
SELECT
    country, actual,
    SUM(actual) **OVER(ORDER BY country ASC)** AS calculating_forward
FROM tblActual
**ORDER BY country DESC**; -- Controls final display order only
```

## Segmenting Calculations: The `PARTITION BY` Clause

By default, the `OVER()` function treats the entire dataset as one massive continuous block. If you want the running metric to automatically drop back to zero and **restart its calculation** whenever a new high-level category changes (e.g., creating a separate isolated running total for each distinct country), inject the **`PARTITION BY`** keyword.

```sql
SELECT
    country,
    location,
    actual,
    SUM(actual) OVER(
        **PARTITION BY country**   -- Steps 1: Split table into mini-country subsets
        **ORDER BY location**      -- Step 2: Accumulate step-by-step within each partition
    ) AS partitioned_running_total
FROM tblActual;
```

### Syntax Guardrails for the Partition Block

- **Ordering Sequence:** Inside the `OVER()` block, the `PARTITION BY` statement must **always** be declared before the `ORDER BY` statement.
- **No Commas Between Clauses:** Do _not_ place a comma separating the partition clause from the ordering clause. (Write: `PARTITION BY country ORDER BY location`).

## Visualizing Partitioned Running Totals

Below is a conceptual tracing grid demonstrating how the Fabric engine computes independent cumulative layers when combining both clauses on a sample dataset:

| **country (Partition)**   | **location (Sort Anchor)** | **actual (Row Value)** | **Running Total Output Behavior**          |
| ------------------------- | -------------------------- | ---------------------- | ------------------------------------------ |
| **England**               | Birmingham                 | 7,000                  | **7,000** _(Base Row)_                     |
| **England**               | London                     | 5,000                  | **12,000** _(7,000 + 5,000)_               |
| **England**               | Manchester                 | 11,000                 | **23,000** _(12,000 + 11,000)_             |
| `--- Partition Shift ---` | `--- Window Resets ---`    | `---`                  | `--- Calculations drop back to zero ---`   |
| **France**                | Paris                      | 4,000                  | **4,000** _(Restarts tracking for France)_ |
| **France**                | Lyon                       | 6,000                  | **10,000** _(4,000 + 6,000)_               |

## Window Clause Summary Component Matrix

| **Window Sub-Clause** | **Code Requirement**                 | **Core Ingestion Purpose**                                       | **Component Impact**                                   |
| --------------------- | ------------------------------------ | ---------------------------------------------------------------- | ------------------------------------------------------ |
| **`OVER()`**          | **Mandatory**                        | Declares that an expression is a row-preserving window function. | Prevents the rows from collapsing like a `GROUP BY`.   |
| **`ORDER BY`**        | **Mandatory** _(for running totals)_ | Dictates the exact row-by-row step sequence for math tracking.   | Evaluates which records are "prior" vs. "current".     |
| **`PARTITION BY`**    | _Optional_                           | Groups the dataset into isolated, independent mini-tables.       | Clears the accumulated buffer back to zero on changes. |

# Query a lakehouse/warehouse in Fabric by using the visual query editor

Here is the detailed breakdown note for navigating **Visual Queries** and configuring semantic model settings across Fabric Lakehouses and Warehouses, structured for your DP-600 exam preparation.

## 1. Underlying Settings & Global Configuration

Both the Fabric Lakehouse and the Data Warehouse share a core management framework within the workspace pane, containing specific settings and configuration options:

- **Endorsement Strategy:** Artifacts can be stamped with metadata labels to establish trust across an enterprise. Options include leaving an object **Non-promoted**, marking it as **Promoted** (to indicate it is ready for broad consumption), or elevating it to **Certified** (signifying strict organizational verification).
- **Sensitivity Labels:** Used to apply structural compliance tags to monitor, restrict, or track protected data.
- **Semantic Model Synchronization:** A toggle controls whether new schema tables, calculated views, or semantic changes are automatically synchronized down to the workspace's default Power BI semantic model.

## 2. Default Power BI Semantic Models

Whenever an engineer provisions a Lakehouse or a Data Warehouse in Microsoft Fabric, the platform automatically stands up a matching **Default Power BI Semantic Model**.

### Core Management and reporting

- **New Report Creation:** Clicking **New Report** on the primary ribbon creates a fresh Power BI report tied directly to the default semantic dataset. It automatically bundles every underlying table and view existing in that environment.
- **Model Customization:** To explicitly choose which production tables populate a report, select **Manage default Power BI semantic model**. This menu allows you to manually select the specific tables to expose or hide, preventing unnecessary raw columns or staging tables from confusing end users.

## 3. Visual Queries: Low-Code Matrix Architectures

A **Visual Query** provides a graphical interface built directly on top of the SQL Analytics Endpoint or Data Warehouse. It allows engineers and analysts to construct complex data retrieval steps using a drag-and-drop workflow rather than writing T-SQL text.

```sql
VISUAL QUERY INTERFACE (Restricted Mini-Pane)
   └── Drag in tables ──> Map links ──> Click "Expand/Open Power Query"
                                                   │
   ┌───────────────────────────────────────────────┘
   ▼
FULL-SCREEN POWER QUERY CANVAS (100% Core Functionality Unlocked)
```

### The Interface Transformation Best Practice

By default, initializing a New Visual Query loads a restricted "mini-pane" layout editor. Many advanced data cleaning operations are intentionally hidden or absent in this default layout view (such as _Remove Duplicates_).

> **Core Workflow Pro-Tip:** The very first action an engineer should execute after dragging source tables into a Visual Query canvas is clicking the **Expand/Open Power Query** button in the top corner. This launches a dedicated full-screen window, unlocking the complete suite of low-code Power Query transformation tools.

### Available Power Query Visual Transformations

Once fully expanded, the graphical diagram tool supports a wide array of data preparation and column manipulation features:

- **Column Management:** Use _Choose Columns_ to pick fields, _Go to Column_ for rapid navigation, and _Remove Columns_ to drop unwanted attributes.
- **Row Reduction:** Filter out unwanted records by keeping or removing top rows, bottom rows, matching ranges, data errors, or duplicate entries (_Home > Remove Rows > Remove Duplicates_).
- **Structural Joins:** Select _Combine > Merge Queries as New_ to join tables horizontally using specific key matches (such as binding an `actual_sum` and a `target_sum` table together using a **Full Outer Join** mapping). You can also use _Append Queries_ to stack datasets vertically.
- **Canvas Controls:** Built-in diagram layout tools allow you to toggle a structural _Mini-map_ overlay, adjust zoom scales, fit the flowchart to view, or fully collapse/expand the active query step tree.

## 4. Column Transformation Nuances

When right-clicking column headers or clicking the inline **Plus (`+`) Icon** inside a query diagram step, pay close attention to the distinction between transforming a column versus adding a new one:

- **Transform Tab Operations:** Changes the data type or layout directly inside the existing column. This includes formatting strings (Lowercase, Uppercase, Trim) and applying a structural **Prefix** or **Suffix**.
- **Add Column Tab Operations:** Generates a brand-new column to hold the processed output while preserving the original column intact. Supported actions include calculating string length, extracting text segments (First/Last characters), and generating _Columns from Examples_.
- **The Prefix/Suffix Rule:** Adding a text _Prefix_ or _Suffix_ is a unique operation that is exclusively available under the **Transform** function suite; it cannot be executed as a direct operation from the _Add Column_ tab.

## 5. Visual Queries vs. Hardcoded SQL

The Visual Query ribbon features an automated utility button called **View SQL** alongside an option to **Save as View**. This auto-generates the backend T-SQL statement representing your graphical steps.

> **Architectural Guardrail / Warning:** For complex data cleaning operations involving multi-table merges, column renames, or granular filters, the auto-generated code produced by the _View SQL_ translator is often highly nested, repetitive, and overly complex. For optimal production performance and ease of maintenance, manually engineering clean, structured T-SQL script files remains the best practice for complex enterprise workloads.

# Quiz 20: Transform data in a data warehouse

!image.png

!image.png

!image.png

# 25. Improving performance in SQL

# Create views, functions and stored procedures

Here is the detailed architectural breakdown note on encapsulating SQL queries using **Views**, **Stored Procedures**, and **Table-Valued Functions** within Fabric, structured for your DP-600 exam preparation.

## Code Encapsulation Objects: Fast Comparison

While these programming objects do not natively alter the physics of data storage on their own, they allow engineers to package optimized queries (e.g., restricted row/column scopes) into reusable modular components.

| **Object Type**        | **Ingestion Placement**    | **Supports Input Parameters?** | **Can Process Multiple Statements?**   |
| ---------------------- | -------------------------- | ------------------------------ | -------------------------------------- |
| **`VIEW`**             | Inside the `FROM` clause   | **No**                         | No (Single query only)                 |
| **`STORED PROCEDURE`** | Standalone call via `EXEC` | **Yes**                        | **Yes** (Full multi-statement scripts) |
| **`FUNCTION`** (TVF)   | Inside the `FROM` clause   | **Yes**                        | No (Single query output map)           |

## 1. Relational Views

A **View** acts as a virtual table. It saves a predefined `SELECT` query configuration within the system database catalog without duplicating physical rows in OneLake.

### Creation Paths in Fabric

- **The UI Path:** Highlight any functional query in the editor and click the **Save as view** ribbon button.
- **The T-SQL DDL Path:**
  ```sql
  CREATE VIEW dbo.view_address_data AS
  SELECT address_ID, address_line1, city, country_region
  FROM dbo.address_data;
  ```

### Usage and Architectural Bounds

Once compiled, you query a view identically to a standard physical table frame (`SELECT * FROM dbo.view_address_data`).

- **The Query Layer Chain:** You can easily write a query targeting View B, which itself depends on View A. This isolates messy base transformations from downstream analytics layers.
- **Strict Constraint:** Views **cannot accept dynamic parameters**. They represent static, unyielding logic masks.

## 2. Stored Procedures (`PROCEDURE`)

A **Stored Procedure** (`PROC`) encapsulates transactional execution code. It is designed to run complex multi-line operational routines, pass input parameters, and output multiple independent data tables concurrently.

### Structural Syntax Pattern

```sql
-- Batch separator required if dropping/altering in the same query tab script
GO

CREATE OR ALTER PROCEDURE dbo.proc_get_regional_addresses
    @country VARCHAR(20) -- Input Parameter (Prefixed with '@')
AS
BEGIN
    -- Batch Statement 1: Disconnected from the parameter
    SELECT COUNT(*) AS total_global_catalog FROM dbo.address_data;

    -- Batch Statement 2: Parameterized query block
    SELECT address_ID, city, country_region
    FROM dbo.address_data
    WHERE country_region = @country;
END;
```

### Execution and Batch Constraints

- **Invoking Procedures:** You execute a procedure standalone using the **`EXEC`** or `EXECUTE` command keyword along with its parameter criteria values. You _cannot_ select data directly from a stored procedure in a `FROM` clause.

```sql
EXEC dbo.proc_get_regional_addresses @country = 'Canada';
```

**The Batch Boundary Rule:** `CREATE PROCEDURE` statements must always be the very first statement processed in a query batch. If you are grouping cleanup commands (like dropping an old version) in the same script window tab, you must inject the **`GO`** keyword command to isolate execution batches.

## 3. Inline Table-Valued Functions (TVF)

A **Function** provides a middle ground: it supports dynamic input variables like a stored procedure, but it returns a tabular output dataset that can be dropped directly into a `FROM` clause like a View.

> **Fabric Analytical Constraint:** While standard SQL engines support both _Scalar_ functions (returning a single individual value) and _Table-Valued_ functions, Fabric SQL development environments **strictly restrict user-defined functions to Table-Valued Functions (TVFs)**.

### Structural Syntax Pattern

```sql
CREATE FUNCTION dbo.func_address_by_region ( @country VARCHAR(20) -- Input Parameter
)
RETURNS TABLE -- Plural return type specification
AS
RETURN
( -- A single structural SELECT statement SELECT address_ID, city, country_region FROM dbo.address_data WHERE country_region = @country
);
```

### Downstream Consumption Example

Because the function returns a valid relation table structure, you can drop it straight into a `FROM` clause and continue applying standard column selections, `WHERE` clauses, or sorting tiers on top of it:

```sql
SELECT address_ID, city
FROM dbo.func_address_by_region('Canada')
WHERE city LIKE 'V%'
ORDER BY city ASC;
```

# Choose a storage mode for a semantic model, including Direct Lake

Here is the detailed breakdown note on **Direct Lake Storage Mode** in Microsoft Fabric, structured for your DP-600 exam preparation.

## 1. What is Direct Lake Mode?

**Direct Lake** is a groundbreaking data connection engine exclusive to Microsoft Fabric semantic models. It serves as a structural bridge combining the high performance of **Import mode** with the real-time data freshness of **DirectQuery**.

!Power BI Data Connectivity Architecture. Source: radacad

Power BI Data Connectivity Architecture. Source: radacad

### Mechanics of the Storage Modes

- **Import Mode:** Completely duplicates and stores a snapshot of the source data inside the specialized Power BI in-memory VertiPaq engine cache. While queries execute fast, it introduces data latency because you must configure regular refresh schedules to sync changes.
- **DirectQuery Mode:** Never copies any records. It translates active visual queries into live relational database statements on the fly. This ensures real-time accuracy but incurs significant data extraction latency.
- **Direct Lake Mode:** The Power BI VertiPaq engine loads data **directly from raw Delta Parquet files in OneLake memory frames** without executing traditional T-SQL queries or copying files. It yields near-instant query speeds alongside live data freshness.

## 2. Structural Requirements and Limitations

To implement Direct Lake successfully, you must operate within strict engineering boundaries:

- **Capacity Prerequisites:** The underlying data tables must be hosted in either a **Fabric Lakehouse** or a **Fabric Data Warehouse** backed by an active Microsoft Fabric capacity.
- **Base Object Restriction:** Direct Lake mode supports only physical backend **tables**. It does _not_ support database **views**; any custom semantic model relying on a relational view will automatically default to DirectQuery mode instead.
- **Mixing Isolation Rule:** If a table uses Direct Lake mode, your semantic model cannot simultaneously utilize traditional modes like Import, DirectQuery, or Dual configurations.
- **Unsupported Operations:** Calculated columns and calculated tables built using DAX are natively unsupported inside the Direct Lake ecosystem. However, write operations are supported over the XMLA endpoint using modern tabular modeling software tools (such as SSMS, Tabular Editor, or DAX Studio).

## 3. On-Demand Column Paging & Memory Behavior

Direct Lake optimizes your operational memory allocation by pulling only the exact data assets required for your immediate visuals.

- **Cold Start Column Loading:** When a fresh semantic model is spun up or refreshed, its memory footprints are completely empty. If an end user opens a dashboard that requests data from _Column A_, the VertiPaq engine pages only the compressed Parquet data for _Column A_ directly into system memory.
- **Incremental Column Loading:** The initial query may experience a slight delay while column data loads into memory. Subsequent queries against _Column A_ execute instantly. If a visual later requests _Column B_, only that specific column is paged into the existing memory cache.
- **Coherent Time-Anchoring:** If a multi-column visual requests _Column B_ hours after _Column A_ was loaded, the engine pages _Column B_ using a strict snapshot time-anchor. This ensures data coherence, preventing a situation where columns display data from different points in time.
- **Eviction Strategy:** If a semantic model begins to approach system limits, infrequently accessed or aging columns are automatically evicted from memory. They remain safely stored in OneLake and can be dynamically re-paged if an end user accesses them.

## 4. Fabric Capacity Limits and Fallback Behavioral Configuration

Fabric capacities impose strict ceilings on maximum rows and uncompressed table sizes.

| **Scale Parameter Metrics**    | **Smaller Fabric SKUs (e.g., F2 to F8)** |
| ------------------------------ | ---------------------------------------- |
| **Max Row Count Limit**        | 300 Million Rows per individual table    |
| **Max Model File Size**        | 10 Gigabytes total file volume           |
| **Max Allocated Query Memory** | 3 Gigabytes active memory space          |

### Handling Capacity Overages

If your dataset breaches the maximum row counts or file sizes allowed by your assigned SKU capacity, **Direct Lake calculations will cease**. Instead, your model will experience an immediate architectural shift known as **Fallback**.

### Configuring Fallback Behavior

To adjust this setting, open your semantic model layout editor, click into the clear background space (ensuring no individual table is highlighted), and locate the **Direct Lake Behavior** property menu:

1. **Automatic (Default):** The engine leverages Direct Lake functionality for all compatible tables. If capacity limits are breached or compatibility boundaries fail, it gracefully falls back to **DirectQuery** mode to keep visuals running.
2. **DirectQuery Only:** Disables Direct Lake optimizations globally, forcing all metrics to fetch live records using standard DirectQuery pipelines.
3. **Direct Lake Only:** Demands absolute Direct Lake execution. If a table breaks a capacity rule or attempts to reference an unsupported view, the engine blocks the query entirely and throws a validation error instead of falling back.

> **Performance Note:** If you exceed the allocated **Query Memory** parameter (e.g., crossing the 3 GB line on an F2 SKU), Direct Lake continues to run without triggering a full DirectQuery fallback. However, query execution speeds will drop significantly because the engine must actively shuffle data frames in and out of system memory.

## 5. Keeping Data Current: Core Refresh Configurations

Direct Lake relies on two configuration paths within your model settings to align with your organization's ETL workflows:

### Configuration 1: Real-Time Automatic Updates

- **Mechanism:** A background process monitors your OneLake files. Any time an upstream pipeline writes modifications or inserts records into the Delta Parquet storage, Direct Lake captures the metadata change and updates the visual presentation layer within seconds.
- **When to Disable:** Turn this off during massive, multi-hour ETL data prep runs or large batch updates to prevent the system from constantly updating after every individual file insert. Once your pipeline completes, trigger a single, clean refresh programmatically using the Power BI REST API.

### Configuration 2: Hardcoded Refresh Schedules

If automatic updates are disabled, you must map out a fixed processing timetable:

- **Frequency Tiers:** Supports **Daily** or **Weekly** recurrences (Monthly scheduling is not supported).
- **Slots Allocations:** Standard licensing setups permit up to **8 time slots** per day, while enterprise Premium/Fabric capacities unlock up to **48 time slots** per day (configured on the hour or half-hour).
- **Alerting Pipelines:** Features automated checkbox routing to immediately dispatch refresh failure notifications to the primary dataset owner or secondary technical support contacts.

# Choose between Direct Lake on OneLake and Direct Lake on SQL endpoints

Here is the detailed comparative breakdown of the two modern pathways to access **Direct Lake** data—**Direct Lake via the SQL Analytics Endpoint** versus **Direct Lake via the OneLake Catalog**—structured for your DP-600 exam preparation.

## The Two Direct Lake Access Pathways

While Fabric originally required routing all Direct Lake connections strictly through the SQL Analytics Endpoint, the architecture includes a secondary native pathway: connecting **Direct Lake directly to the OneLake Catalog**.

Both methods deliver the ultra-fast in-memory performance of VertiPaq running over Delta Parquet files, but they differ significantly in tenant governance, schema flexibility, and advanced modeling capabilities.

## Shared Architecture (The Similarities)

Regardless of whether your semantic model points to the SQL Endpoint or the OneLake Catalog, the underlying engine enforces the following shared baseline rules:

- **Capacity Requirement:** Both methods strictly require an active Microsoft Fabric capacity subscription.
- **Authentication:** Both support **Single Sign-On (SSO)** out of the box, simplifying security configurations for end users.
- **Security & Scale:** Both natively support enterprise-grade semantic model **Row-Level Security (RLS)** and **Object-Level Security (OLS)**, and both easily handle massive big-data volumes.
- **Modeling Restrictions:** Neither pathway supports DAX _Calculated Columns_. Neither allows traditional _User-Defined Aggregations_ or native _Model Table Partitions_ (though row partitioning can be handled upstream directly inside the Delta tables themselves).
- **Advanced Features:** Both fully support Calculation Groups, What-If parameters, and Field Parameters.

## Deep-Dive Comparison Matrix

| **Architectural Feature**       | **Direct Lake on SQL Analytics Endpoint**                                    | **Direct Lake on OneLake Catalog**                                                      |
| ------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Tenant Availability**         | Enabled globally by default for all tenants.                                 | Requires explicit activation in the Fabric Admin Portal.                                |
| **Supported Data Sources**      | Restricted exclusively to Fabric **Lakehouse** or **Warehouse** tables.      | Any Fabric data source backed by formal **Delta tables**.                               |
| **Composite Model Support**     | **Forbidden.** Cannot mix Direct Lake tables with Import/DirectQuery tables. | **Allowed.** Direct Lake tables can be blended with Import, DirectQuery, or Dual modes. |
| **Calculated Tables (DAX)**     | Not supported.                                                               | Fully supported.                                                                        |
| **Auto Date/Time Intelligence** | Not supported inside Power BI Desktop.                                       | Fully supported inside Power BI Desktop.                                                |
| **Database View Support**       | Supported, but **forces an immediate fallback** to DirectQuery mode.         | Not supported.                                                                          |
| **Endpoint Security (CLS/OLS)** | Supports Endpoint-level Column-Level and Object-Level Security.              | Relies strictly on upstream semantic model security.                                    |
| **Lakehouse Shortcuts**         | Native data source support.                                                  | Restricted/Limited indirect support.                                                    |
| **Deployment Pipelines**        | Fully supports data source rebinding rules.                                  | Highly restrictive data source rebinding.                                               |

## Key Differentiators Explained

### 1. Tenant Governance and Administration

- **SQL Endpoint:** Universally accessible across all Fabric environments without administrative intervention.
- **OneLake Catalog:** Must be explicitly enabled by a Fabric Administrator. In the **Admin Portal**, administrators must toggle on the setting: _"Users can create Direct Lake on OneLake semantic models."_

### 2. Composite Modeling and Storage Mixing

- **The SQL Endpoint Barrier:** If you connect via the SQL Endpoint, your semantic model is locked into a rigid storage structure. You cannot mix these tables with external Import or DirectQuery tables.
- **The OneLake Catalog Advantage:** Unlocks true hybrid **Composite Models**. You can configure a high-performance Direct Lake table directly alongside legacy Import tables or live DirectQuery sources within the same unified semantic model.

### 3. Deployment Pipeline Rebinding Rules

When promoting data assets through lifecycle environments ( $Development \rightarrow Test \rightarrow Production$ ), your staging databases will point to different physical sources than your production environments.

- **SQL Endpoint Edge:** Fully supports automated **Deployment Pipeline Rules**. As an asset moves between workspaces, the pipeline rule automatically rebinds the semantic model's data source string from the Dev SQL endpoint to the Prod SQL endpoint.
- **OneLake Catalog Limitation:** Rebinding data sources during deployment pipeline transitions is significantly more restrictive and requires manual configuration or scripted workarounds.

### 4. Advanced Security Routing

If you need to implement **Column-Level Security (CLS)** or database-native **Object-Level Security (OLS)** directly within the relational SQL engine layer rather than higher up in the Power BI semantic model layer, you **must use the SQL Analytics Endpoint connector**.

> **Fallback Warning:** Keep in mind that referencing database **Views** or triggering engine-level security rules on the SQL Analytics Endpoint will cause the engine to immediately drop out of high-speed Direct Lake mode and **fall back to DirectQuery mode**.

# Identify and resolve data loading bottlenecks in SQL queries

## 1. Upstream Optimizations: Query Folding

When your data loading process pulls data from a relational SQL database via **Dataflows Gen2**, your first line of performance defense is **Query Folding**.

- **Mechanism:** Query folding occurs when the Power Query M engine translates your visual transformation steps into a single, native SQL statement and pushes it back to the source server for execution.
- **Impact:** This ensures the source database handles the heavy filtering, sorting, and data compression using its own optimized resources, delivering only the final, minimized dataset to Fabric. If a step breaks query folding, Fabric must download the entire raw dataset and process it locally, creating a massive data loading bottleneck.

## 2. Horizontal & Vertical Volume Reduction

A primary cause of slow queries in the SQL Analytics Endpoint or Data Warehouse is requesting unnecessary data volume.

### Vertical Reduction: Eliminating `SELECT *`

The wildcard operator (`SELECT *`) is a heavy engineering anti-pattern in production environments.

- **The Problem:** It commands the engine to scan and stream every single column in the table, increasing network latency and memory usage.
- **The Fix:** Explicitly name only the specific columns required for your downstream semantic models or report visuals.

### Horizontal Reduction: `TOP` vs. `LIMIT`

If you are inspecting a dataset or building a staging layer, never pull millions of records if a subset will suffice.

- **SQL Analytics Endpoint (T-SQL):** Use the `TOP` keyword immediately after the `SELECT` statement to restrict rows.
- **Fabric Notebooks (Spark SQL):** Place the `LIMIT` keyword at the very end of your query block.

## 3. Data Type Optimization & Storage Footprint

Because Microsoft Fabric stores data natively in columnar **Delta Parquet** formats inside OneLake, optimizing your underlying column data types directly impacts query speeds and I/O scan times. The fewer bytes used per row, the faster the column can be loaded into memory.

### Whole Numbers: Int vs. BigInt

- **The Bottleneck:** Developers frequently default to `BIGINT` for identification or surrogate keys out of habit.
- **The Fix:** Evaluate your true data scale. A standard integer (`INT`) can store whole numbers up to approximately **2 billion**. If your row count will never exceed this threshold, shifting from `BIGINT` to `INT`, `SMALLINT`, or `TINYINT` saves significant storage bytes per record and accelerates scans.

### Text Strings: Char vs. Varchar

- **The Bottleneck:** Defining columns with a fixed-length constraint like `CHAR(8000)` forces the engine to physically allocate a full 8,000 bytes of storage for every single row—even if a specific cell only contains a short 10-character comment. The remaining space is filled with empty padding.
- **The Fix:** Convert unstructured text, notes, or comment fields to variable-length character schemas (**`VARCHAR`**). A `VARCHAR(8000)` column sets the exact same safety ceiling but only consumes storage bytes proportional to the actual text string entered for that row.

## Performance Tuning Summary Matrix

| **Optimization Target** | **Anti-Pattern Example**    | **High-Performance Pattern**     | **Architectural Benefit**                                         |
| ----------------------- | --------------------------- | -------------------------------- | ----------------------------------------------------------------- |
| **ETL Ingestion**       | Local M Engine processing   | **Query Folding** enabled        | Offloads processing to the source database.                       |
| **Schema Projection**   | `SELECT * FROM table;`      | `SELECT id, city FROM table;`    | Minimizes vertical network transport and memory allocation.       |
| **Row Restraints**      | Scanning all raw records    | `SELECT TOP 1000` / `LIMIT 1000` | Drastically lowers execution times during ad-hoc analysis.        |
| **Numeric Schema**      | Using `BIGINT` universally  | Matching to `INT` or `SMALLINT`  | Reduces column memory footprint in OneLake storage.               |
| **Text Schema**         | `CHAR(8000)` for brief text | `VARCHAR(8000)` or lower         | Eliminates wasted byte padding on short or variable text strings. |

# Implement performance improvements in SQL queries

## 1. The Architectural Role of Indexes and Tradeoffs

An **Index** in a relational database acts like an index at the back of a textbook. Instead of scanning every single page (a full table scan) to find a specific keyword, the database engine lookups the pointer directly in the index object, drastically accelerating retrieval speeds.

### Where Indexes Optimize Queries

Creating indexes delivers massive performance boosts when targeted at columns frequently appearing in these specific SQL operations:

- **`JOIN` clauses:** Accelerates matching foreign keys between unified fact and dimension schemas.
- **`WHERE` clauses:** Speeds up row-level filtering by isolating specific boundaries instantly.
- **`GROUP BY` clauses:** Optimizes the clustering phase by having the distinct categorization rows already structured.
- **`ORDER BY` clauses:** Eliminates the computational step of sorting the output grid, as the index is pre-sorted.

### The Read vs. Write Tradeoff

You should not blindly index every column in a table. While indexes vastly improve **Read (SELECT)** performance, they heavily degrade **Write (INSERT, UPDATE, DELETE)** performance. Every time data is modified or added to a table, the database administrator's engine must immediately pause to recalculate and rewrite every index object attached to that table.

> **Fabric Exam Tip / Guardrail:** At present, you **cannot manually create indexes** directly inside native Microsoft Fabric Lakehouses or Data Warehouses. They rely on automated column-store optimizations and Delta Parquet file storage. However, understanding indexing is critical when your Fabric pipelines are ingestion-tuning from an upstream relational source like an **Azure SQL Database**.

## 2. Implementing SARGable Conditions

**SARGable** stands for **Search ARGument ABLE**. A query is considered SARGable if the SQL engine can directly utilize an underlying index to isolate rows, rather than processing every row one by one.

### The Non-SARGable Anti-Pattern (Function on a Column)

Wrapping an architectural database column inside a function inside the `WHERE` clause completely breaks index matching. The engine is forced to calculate the function for _every single row_ in the table before it can evaluate if it matches your criteria.

### Rewriting Non-SARGable Queries to SARGable

#### Scenario A: Date Filtering

- **Non-SARGable (Slow):** Evaluates the `YEAR()` function across the entire dataset.
  ```sql
  WHERE YEAR(order_date) = 2007;
  ```
- **SARGable Rewrite (Fast):** Converts the logic into a direct column comparison across a static date range window.
  ```sql
  WHERE order_date >= '2007-01-01' AND order_date < '2008-01-01';
  ```

#### Scenario B: Text Snippet Matching

- **Non-SARGable (Slow):** Evaluates the string splitting function on every row.
  ```sql
  WHERE SUBSTRING(sales_order_number, 1, 3) = 'SO5';
  ```
- **SARGable Rewrite (Fast):** Leverages a leading text string literal alongside a wildcard operator.
  ```sql
  WHERE sales_order_number LIKE 'SO5%';
  ```

### The SARGable Operator Toolkit

To ensure your workloads safely trigger index seeks rather than slow table scans, design your query filters around these SARGable operators:

- `=` (Equality) and inequalities (`>`, `<`, `>=`, `<=`)
- **`BETWEEN`**
- **`LIKE`** (Only when the wildcard is placed at the _end_ of the string, e.g., `'Text%'`)
- **`IS NULL` / `IS NOT NULL`**
- **`IN`**

## 3. Optimizing Cardinality and Data Compression

**Cardinality** refers to the uniqueness of data values contained in a column.

- **High Cardinality:** A column containing highly unique, distinct values (e.g., unique timestamps or Guid strings).
- **Low Cardinality:** A column containing heavily repeating categories, flags, or broad indicators (e.g., countries or product classes).

### Tuning Split Strategy: Splitting DateTime Columns

In modern columnar big-data systems like Delta Parquet inside OneLake, lowering a column's uniqueness score allows the built-in dictionary encoding and compression engines to shrink data storage sizes and accelerate analytical scans.

Consider a system logging a transaction every two minutes. Storing this as a single combined `DateTime` column forces almost every row value to be completely unique (High Cardinality).

```sql
COMBINED DATETIME (High Cardinality - Bad Compression):
[2026-07-01 14:02:00] ──> Unique Value
[2026-07-01 14:04:00] ──> Unique Value

SPLIT ARCHITECTURE (Low Cardinality - High Compression):
Date Column:        [2026-07-01] ──> Repeats 720 times a day (Compresses perfectly)
Time Column:        [14:02:00]   ──> Standardized repeating time matrices
```

By structurally breaking a combined stamp down into two dedicated columns—a distinct **Date** column and a distinct **Time** column—the Date column will now repeat the exact same text token 720 times a day. This uniform pattern enables high-ratio dictionary compression and significantly speeds up column-store index evaluations.

# Quiz 21: Create objects

!image.png

!image.png

!image.png

# 26. Creating an eventhouse

# Creating an eventhouse, exploring the environment, and getting data

## 1. Core Architecture: What is an Eventhouse?

An **Eventhouse** is an enterprise-grade telemetry engine designed for real-time intelligence workloads inside Microsoft Fabric. It provides a shared compute container capable of hosting multiple isolated **KQL (Kusto Query Language) databases** for a project.

> **Engine telemetry** is **the automated process of collecting and transmitting real-time performance, diagnostic, and operational data from an engine to a centralized monitoring system**

### Ideal Workload Use Cases

Unlike a standard Lakehouse or a Data Warehouse (which are built for batch-oriented relational analysis), an Eventhouse is highly optimized for **append-only, high-ingestion time-series and event-based logs**. Choose an Eventhouse when managing:

- **System Monitoring:** Security, infrastructure, and compliance log metrics.
- **IoT Telemetry:** Device, machinery, and smart-sensor data streaming continuously.
- **Temporal Records:** High-frequency financial tickers and stock market transactions.
- **Application Tracking:** User clickstreams and real-time gaming telemetry.

## 2. Navigating the Real-Time Intelligence Experience

To deploy an Eventhouse environment, you must pivot away from the traditional Data Factory or Power BI personas inside Fabric:

1. **Switch Experiences:** Click the experience selector icon in the bottom-left corner of the Fabric interface and select **Real-Time Intelligence**.
2. **Workspace Setup:** Provision or open a Fabric-enabled workspace configured with either a **Trial** capacity or a dedicated **Fabric Capacity** license mode.
3. **Artifact Provisioning:** Click _New Item_ or locate the _Eventhouse_ icon on the Real-Time home ribbon to initialize the compute layer.

> **Naming Best Practice:** When an Eventhouse is provisioned, Fabric automatically spawns a child KQL database sharing the exact same name as the host container. To maintain a clean metadata architecture, delete this default database and create a explicitly distinct container named something like `yourProject_Eventhouse_DB`.

## 3. Ingestion Topology: Getting Data Into a KQL Database

The Eventhouse supports two major ingestion pathways: batch historical loading and continuous event streaming.

### Pathway A: Batch and External Ingestion

Accessible by clicking the ellipsis (`...`) next to your custom KQL Database and choosing **Get Data**:

- **Local Files:** Uploading local structured files (CSV, JSON, Parquet).
- **Cloud Fabrics:** Pulling historical batches out of **OneLake**, Azure Blob Storage, or Amazon S3 buckets.

### Pathway B: Real-Time Continuous Ingestion

For active, live event streams, the engine hooks directly into the **Real-Time Hub**, supporting native listeners for:

- **Fabric Eventstreams**
- **Azure Event Hubs**
- **Data Factory Pipelines**

### Working with Sample Data

For rapid development testing and KQL syntax validation, Fabric offers instant sample analytical templates, including _Weather Analytics_ (capturing US storm event logs). Loading this sample auto-generates a structured table populated with time-series dimensions such as `State`, `EventType`, and explicit ingestion `Timestamps`.

## 4. UI Monitoring Interface & Contextual Menus

Once an Eventhouse table is populated, the user interface shifts contextually depending on what object is highlighted in the explorer tree:

- **The Data Activity Tracker:** Positioned directly above the data preview grid. It renders a live timeline graph tracking when rows hit the table. You can visually toggle the time granularity (e.g., viewing records bundled every 5 minutes versus aggregated by day) to track ingestion spikes.
- **Contextual Menu Scopes:**
  - **Table Highlighted:** Unlocks the _Table Menu_ ribbon to instantly run KQL queries, ingest additional records, delete rows, or immediately build a _Power BI Report_ on top of the raw event stream.
  - **Database Highlighted:** Unlocks the database control bar to provision clean tables, run cross-table queries, or map out real-time related items.
  - **Eventhouse Highlighted:** Governs cluster-level settings and allows you to spin up additional KQL databases.

# Creating sample KQL and SQL queries, and exploring the query environment

Here is the detailed breakdown note for querying Eventhouse data using **KQL (Kusto Query Language)**, understanding the `explain` operator, and leveraging real-time visualization features, structured for your DP-600 exam preparation.

## 1. Accessing and Initiating KQL Queries

Inside a Fabric KQL database, there are two primary methods to quickly generate and run analytical scripts:

- **The Query Set Workspace:** Located directly beneath your KQL database node in the explorer tree. Opening it automatically provides boilerplate queries requiring only your target table name.
- **Contextual Table Menu:** Clicking the ellipsis (`...`) next to a specific table (e.g., `weather`) provides quick-launch templates such as:
  - _Show any 100 records_ (Picks a random sample).
  - _Show the total count of records_.
  - _Summarize ingestion per hour_ (Audits continuous telemetry streams).
  - _Get table schema_ (Exposes structural column definitions and metadata types).

## 2. Core KQL Architecture: The Pipe Operator (`|`)

The foundation of KQL syntax relies heavily on a step-by-step linear flow.

```sql
[Base Table Name]
       │
       ▼
 ┌───────────┐
 │  Pipe (|) │ ──> Pass rows to next operation (e.g., filter, count, project)
 └───────────┘
```

- **The Data Pipeline Concept:** A KQL query **always starts with the name of the table** on line one.
- **The Pipe Symbol (`|`):** Every subsequent line of code must be prefaced by a pipe character. The pipe takes the complete output table generated from the previous row and pushes it forward into the next operational filter or clause.

### Basic KQL Syntax Examples

```sql
-- Example 1: Returns a swift, top-row data subset scan
weather
| take 100

-- Example 2: Aggregates total row height count (e.g., returning 59,066 records)
weather
| count
```

## 3. Translating SQL to KQL via the `explain` Utility

While the Fabric Eventhouse engine can interpret standard relational SQL queries (`SELECT TOP 10...`), it is natively built and optimized for KQL. Forcing a time-series telemetry layer to process SQL statements adds execution time overhead (e.g., a query taking 0.04 seconds in SQL versus 0.032 seconds in native KQL).

To effortlessly convert legacy SQL code into optimized native KQL, use the **`explain`** directive:

```sql
-- Prefix your T-SQL query with two hyphens and 'explain'
-- explain
SELECT TOP 10 state, COUNT(*) AS number_of_rows
FROM weather
GROUP BY state
ORDER BY number_of_rows DESC;
```

### The Resulting KQL Blueprint Translation

When you execute the block above, the engine returns a clean KQL script reflecting the sequence of operations:

```sql
weather
| summarize number_of_rows = count() by state
| project state, number_of_rows
| sort by number_of_rows desc
| take 10
```

1. **`summarize ... by`**: The immediate KQL equivalent of SQL's combined `GROUP BY` and aggregate functions (`COUNT`, `SUM`). It compresses data blocks and names the output metric in a single statement.
2. **`project`**: The equivalent of the SQL column list, specifying exactly which fields pass vertically to the final results pane.
3. **`sort by`**: Orders rows dynamically (defaults to descending if explicitly stated).
4. **`take`**: Restricts the horizontal row count, structurally identical to T-SQL's `TOP` or Spark SQL's `LIMIT`.

## 4. Advanced Real-Time Intelligence Features

Once a KQL dataset returns an active data pane, you unlock several dedicated streaming features:

### 1. Low-Code Visual Charts

You don't need a standalone reporting application to visually interpret your query output. Clicking **Add Visual** inside the results workspace allows you to map your axes instantly into native presentation formats:

- **Trend & Vectoring:** Bar Charts (Standard, Stacked, 100% Stacked), Column Charts, Line Charts, Area Charts.
- **Proportions & Density:** Pie/Donut Charts, Scatter Plots, Multi-Stat Blocks, and **Heat Maps** for identifying geographical ingestion anomalies.

### 2. Downstream Integration Actions

- **Pin to Dashboard:** Pins the live, self-refreshing query visual directly to a **Real-Time Intelligence Dashboard** (which updates much faster than traditional Power BI dashboards).
- **Set Alert:** Establishes a data monitoring threshold. If a continuous log table matches a problematic query condition, the engine triggers automated workflows to send an email, fire a message in **Microsoft Teams**, or run a Data Factory pipeline item.
- **Power BI Export:** Automatically builds a complete Power BI semantic model and report layer pre-mapped directly to your KQL statement.

## 5. Free External Training Sandbox Environment

If you want to practice your KQL syntax without utilizing your company's active Fabric capacity subscription limits, you can connect to Microsoft's global open-source sandbox cluster:

1. Navigate to the online log testing portal: **dataexplorer.azure.com**.
2. Locate the public samples folder: $Samples \rightarrow Tables \rightarrow StormEvents.$
3. _Note:_ The public `StormEvents` sample dataset features identical schemas and row configurations to the weather data used inside Microsoft Fabric, allowing you to validate your KQL logic completely for free.

# Choosing between a lakehouse, warehouse, or eventhouse

Here is the comprehensive architectural comparison between an **Eventhouse**, a **Lakehouse**, and a **Data Warehouse** within Microsoft Fabric, structured to optimize your DP-600 exam preparation.

## Architectural Comparison Matrix

Choosing the right artifact depends on your primary data structure, your development team's skillset, and the required security model.

| **Feature / Artifact**       | **Eventhouse**                                   | **Lakehouse**                                                 | **Data Warehouse**                                         |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------------------------- | ---------------------------------------------------------- |
| **Primary Data Purpose**     | Real-time, streaming, time-series, and IoT data. | Big data engineering, advanced data science, and raw staging. | Enterprise relational BI reporting and historic analytics. |
| **Supported Data Formats**   | Structured, Semi-structured, Unstructured.       | Structured, Semi-structured, Unstructured.                    | **Strictly Structured** (Relational tables).               |
| **Primary Language**         | **KQL** (Kusto Query Language)                   | **PySpark** (Python / Scala / Spark SQL)                      | **T-SQL** (Transactional SQL)                              |
| **SQL Write Capabilities?**  | No (Read-only subset support).                   | No (Read-only via SQL Endpoint).                              | **Yes** (Full DDL/DML support).                            |
| **Primary UI Dev Interface** | KQL Databases & KQL Query Sets.                  | Spark Notebooks & Spark Job Definitions.                      | SQL Query Scripts & Object Explorer.                       |

## Technical Deep Dive: Read and Write Mechanics

While all three artifacts reside on top of OneLake and can read data using standard SQL syntax, their ingestion and modification backends differ significantly.

### 1. Lakehouse Storage Layer

- **Capabilities:** Acts as a unified storage layer containing a `Files/` directory (for unstructured blobs like pictures, audio, or raw files) and a `Tables/` directory (for structured Delta Parquet tables).
- **Write Constraint:** The accompanying SQL Analytics Endpoint is **strictly read-only**. To insert, update, delete, or transform data within a Lakehouse, you must use a code-first approach via **PySpark Notebooks** or low-code tools like Dataflows Gen2.

### 2. Data Warehouse Layer

- **Capabilities:** Tailored for traditional relational database developers and data analysts.
- **Write Advantage:** Fully supports relational write statements. You can execute standard T-SQL scripts (`INSERT INTO`, `UPDATE`, `CREATE TABLE AS SELECT`) to modify both schemas and rows without touching Spark.

### 3. Eventhouse Layer

- **Capabilities:** Built for super-fast append-only real-time ingestion pipelines.
- **Write Advantage:** Optimized around **KQL**. While you can execute basic SQL reads, full engineering capabilities—including complex time-series aggregations, visual dashboards, and automated alert triggering—require KQL syntax.

## Security Model Frameworks

While all three environments support standard workspace permissions and **Row-Level Security (RLS)** to restrict row data based on a user's role, the Data Warehouse provides the most comprehensive security capabilities.

- **Column-Level Security (CLS):** Restricts access to specific high-sensitivity columns (such as salaries or identifiers) within a table. This is natively integrated and fully complete within the Data Warehouse engine.
- **Dynamic Data Masking (DDM):** A security feature unique to the Data Warehouse layer in Fabric. It obfuscates sensitive transactional data on the fly without altering the physical underlying table data in OneLake.
  - _Use Case:_ Masking a 16-digit credit card number column so that unauthorized data consumers only see `XXXX-XXXX-XXXX-1234`, while database administrators retain full visibility.

## Persona Alignment & Decision Guide

### Choose a Lakehouse if...

- You are a **Data Engineer** or **Data Scientist**.
- Your data tier consists of heavily mixed types (e.g., scraping raw PDFs alongside relational logs).
- You require the raw processing scale of Apache Spark to train Machine Learning models.

### Choose a Data Warehouse if...

- You are a **Data Warehouse Developer**, **Database Architect**, or **BI Analyst**.
- Your ecosystem is entirely composed of highly structured, transactional relational tables.
- Your development team possesses deep T-SQL skills and requires strict Column-Level Security and Dynamic Data Masking.

### Choose an Eventhouse if...

- You are a **Real-Time Data Analyst** or **Telemetry Operations Engineer**.
- Your data streams continuously from log metrics, system endpoints, or physical IoT smart sensors.
- You need to detect immediate anomalies, build real-time visual charts, and set rapid data-driven alerts.

!image.png

# 27.

# 28. Functions

Here is the detailed architectural breakdown note on advanced string manipulation, whitespace handling, and empty data testing functions in **KQL (Kusto Query Language)**, structured for your DP-600 exam preparation.

# Empty strings, concatenating and trimming strings

## 1. Testing for Blank Data: Empty vs. Null Strings

In time-series data streams, text fields frequently arrive with missing values. In KQL, you must distinguish between a field containing zero characters (an empty string `""`) and a field containing a completely absent value (`null`).

To filter for fields containing empty text, you can use a direct literal match or a built-in evaluation function:

```sql
-- Method A: Checking against an empty string literal
weather | where BeginLocation == ""

-- Method B: Using the optimized evaluation function
weather | where isempty(BeginLocation)

-- Method C: Isolating completely populated rows
weather | where isnotempty(BeginLocation)
```

### The Sequence Trap: Filter Placement Performance

The chronological order of your pipeline stages fundamentally changes the data profile returned. Consider these two variations:

```sql
-- Variation 1: Pre-Filter Top-N
weather
| take 20
| where isempty(BeginLocation)
-- Analysis: Grabs the top 20 rows of the table *first*, then drops any row where BeginLocation has text. Result: Variable height <= 20 rows.

-- Variation 2: Post-Filter Top-N
weather
| where isempty(BeginLocation)
| take 20
-- Analysis: Finds all empty rows across the entire dataset *first*, and then slices out exactly 20 pure rows. Result: Always exactly 20 rows.
```

## 2. Advanced String Concatenation

Unlike standard relational T-SQL or other programming languages, **KQL does not support string concatenation using the plus symbol (`+`)**. Attempting to write `col1 + col2` will trigger a compilation exception. Instead, use dedicated concatenation functions.

### Function 1: Standard `strcat()`

Blends an arbitrary number of parameters together. It handles mixed data types natively, allowing you to append numerical values (like `EventId`) directly to text strings without manual type casting.

```sql
-- Syntax: strcat(arg1, arg2, arg3, ...)
weather
| extend Calc = strcat(EventType, ": ", EpisodeNarrative)
```

### Function 2: Delimited `strcat_delim()`

When joining multiple data columns with a uniform separator (such as a colon-space or a comma), `strcat()` can become highly repetitive. The **`strcat_delim()`** function streamlines this by accepting a master delimiter as its very first parameter and automatically injecting it _between_ all subsequent fields.

```sql
-- Syntax: strcat_delim(delimiter, arg1, arg2, arg3, ...)
weather
| extend ComplexLabel = strcat_delim(": ", State, EventType, EpisodeNarrative, EventId)
```

## 3. Case Normalization Operators

To clean up data entry variances before joining text strings or evaluating matches, use standard casing transformations inside an `extend` block:

- **`toupper()`**: Forces an entire string variable into absolute UPPERCASE.
- **`tolower()`**: Normalizes a string variable into absolute lowercase.

```sql
weather
| extend UnifiedState = toupper(State)
```

## 4. Whitespace Trimming and Left-Side Padding

When columns are empty or text values contain trailing whitespace, concatenation tasks can easily introduce unwanted spacing layout artifacts. To resolve this, deploy targeted cleanup functions.

```sql
INPUT VALUE: "00012345"
   └──> | extend Out = trim_start("0", Input) ──> Output: "12345"

INPUT VALUE: "   Wisconsin   "
   └──> | extend Out = trim(" ", Input)       ──> Output: "Wisconsin"
```

### The `trim()` Function

Removes all instances of a target character from both the absolute beginning and absolute end of a string text block.

- _Syntax Configuration:_ `trim(CharacterToDrop, SourceColumn)`
- _Scenario:_ If an empty field is concatenated with trailing spaces using `strcat_delim()`, wrapping the result in `trim(" ", Source)` cleanly strips out the leading and trailing padding.

### Edge Anchored Trimming: `trim_start()` and `trim_end()`

- **`trim_start()`**: Removes target tokens strictly from the left hand side of a string.
  - _Use Case (Surrogate Keys):_ If system account values are imported as strings padded with leading tracking zeroes (e.g., `"000045612"`), calling `trim_start("0", AccountNum)` cleanly normalizes the value down to `"45612"` while maintaining its core text data type.

- **`trim_end()`**: Removes target tokens strictly from the right hand side of a string.

## String Utility Function Quick Reference

| **KQL Function**                  | **Data Input Parameter Requirements** | **Core Architectural Purpose**                                  |
| --------------------------------- | ------------------------------------- | --------------------------------------------------------------- |
| **`isempty()`**                   | Text column target                    | Returns true if the target cell string is empty (`""`).         |
| **`strcat()`**                    | Array of mixed strings/numbers        | Chains values sequentially into a single unified column.        |
| **`strcat_delim()`**              | `(Delimiter, col1, col2, ...)`        | Chains strings while auto-injecting a separator between them.   |
| **`toupper()`** / **`tolower()`** | Target column                         | Standardizes character casing properties.                       |
| **`trim()`**                      | `(Character, Source)`                 | Strips the designated token from both edges of the text string. |
| **`trim_start()`**                | `(Character, Source)`                 | Strips matching leading tokens strictly from the left edge.     |

# Manipulating strings

## 1. Zero-Based Indexing Architecture

When parsing and slicing text string vectors in KQL, the engine relies strictly on a **Zero-Based Indexing** framework. This means the very first character of a text string sits at position `0`, the second character at position `1`, and so on.

- **The Slicing Illusion:** If you locate a space character in the string `"High Wind"`, the lookup engine returns an index position of `4`. Even though there are exactly 4 letters before it (_H, i, g, h_), the space itself occupies the 5th logical position in memory.

## 2. Locating Positions with `indexof()` and the Padding Trick

The **`indexof()`** function scans an expression horizontally to find the starting location of a target character or phrase.

### Syntax Structure

```
indexof(LongerSourceString, TargetCharacterToFind)
```

### The Boundary Trap & The Ingest Padding Solution

If `indexof()` searches a column and fails to find a match (for example, searching for a space character inside the single-word event `"Hail"`), it returns a value of **`-1`**. Passing a negative boundary value directly into a downstream slicing function like `substring()` will cause the query to fail.

To build a defensive data pipeline that safely processes single-word entries without throwing errors, use **`strcat()`** to dynamically append a trailing safety padding space to the data during evaluation:

```sql
weather
-- Employs strcat() to ensure a space always exists, preventing a -1 error
| extend FindSpace = indexof(strcat(EventType, " "), " ")
```

- **Impact:** For the word `"Hail"`, the engine transforms the evaluation string internally to `"Hail "`. It finds the space at index position `4`, providing a clean, safe integer value for subsequent slicing steps.

## 3. String Extractions: `substring()` and `strlen()`

To extract distinct word fragments out of a unified column field, use the linear execution model of KQL to pass calculated position integers into the **`substring()`** function.

### Syntax Structure

```sql
substring(SourceString, StartPositionIndex, TotalLengthToExtract)
```

### Chaining Clean Left and Right Slices

Because you cannot reference a column alias inside the same expression block where it is declared, split your position mapping and extraction operations across sequential pipeline stages:

```sql
weather
| distinct EventType
-- Step 1: Map the safe index anchor point
| extend FindSpace = indexof(strcat(EventType, " "), " ")
-- Step 2: The pipe streams the position integer downward to separate evaluation boundaries
| extend
    BeforeSpace = substring(EventType, 0, FindSpace),
    AfterSpace  = substring(EventType, FindSpace + 1, 999)
```

- **Handling Right Edges Safely:** For the right-hand slice (`AfterSpace`), incrementing the start position by `+1` ensures you skip over the space itself. Passing an arbitrarily large length ceiling (like `999`) ensures you capture the remainder of the text without causing clipping errors.

### Exact Length Extraction Alternative

If you prefer to supply the exact mathematically precise character remainder length instead of a generic fallback number like `999`, pair your subtraction logic with the string length tool **`strlen()`**:

```sql
| extend RemainderLength = strlen(EventType) - FindSpace - 1
| extend AfterSpace = substring(EventType, FindSpace + 1, RemainderLength)
```

## 4. Text Substitutions: `replace_string()`

When you need to globally find and substitute specific text segments or words across a column without calculating index positions manually, use **`replace_string()`**.

### Syntax Structure

```sql
replace_string(SourceColumn, OldSegmentToFind, NewSegmentToInject)
```

```sql
weather
-- Swaps out the word "High" for the word "Big" (e.g., transforming "High Wind" to "Big Wind")
| extend ModifiedType = replace_string(EventType, "High", "Big")
```

> **Casing Guardrail:** The `replace_string()` function is **strictly case-sensitive**. Attempting to search for the word `"High"` using a lowercase query like `"high"` will fail to find a match and leave the column data completely unmodified.

## KQL String Manipulation Function Matrix

| **KQL Function**              | **Output Data Type** | **Casing Sensitivity** | **Core Operational Purpose**                                                                 |
| ----------------------------- | -------------------- | ---------------------- | -------------------------------------------------------------------------------------------- |
| **`indexof(S, T)`**           | Integer              | Case-Insensitive       | Returns the zero-based starting position index of a target pattern. Returns `-1` if missing. |
| **`substring(S, P, L)`**      | String Text          | N/A                    | Carves out a targeted portion of text starting from position `P` for a length of `L`.        |
| **`strlen(S)`**               | Integer              | N/A                    | Computes the absolute total character count of a string row.                                 |
| **`replace_string(S, O, N)`** | String Text          | **Case-Sensitive**     | Finds every exact instance of an old substring and replaces it with a new value.             |

# Other string functions

## 1. Advanced Structural Manipulations

While standard slicing handles primary data transformations, KQL contains an extended toolkit for specialized text engineering workloads:

- **`split()`**: Segments a single unified string into an ordered array of distinct elements based on a chosen delimiter.
  - _Use Case:_ Breaking a fully qualified domain path or file pathway string into separate components.
- **`strrep()`**: Repeats a target string literal a designated number of times.
  - _Use Case:_ Dynamically padding or aligning columns by prefixing text with a variable number of blank spaces.
- **`reverse()`**: Inverts the character order of a string completely (e.g., transforming `"ABC"` to `"CBA"`).

## 2. Character Counts and Extractions

- **`countof()`**: Scans a longer expression and returns an exact integer tracking how many times a smaller substring fragment occurs within it.
  - _Use Case:_ Counting the number of dots (`.`) inside an IP address or system configuration log line to validate structural formatting.
- **`extract()`**: Extracts a regex pattern match from a text string, pulling out specific alphanumeric shards from a larger, unstructured message block.

## Core KQL String Function Checklist

When preparing for the DP-600 exam, prioritize mastering the primary foundational string functions highlighted across this curriculum. These form the core architecture of most text-cleansing data pipelines:

```
                  ┌── Text Evaluations  ──> isempty() / isnotempty()
                  ├── Case Adjustments  ──> toupper() / tolower()
KQL CORE TOOLKIT  ├── Row Filtering     ──> has / contains / startswith / has_any
                  ├── Column Assembly   ──> strcat() / strcat_delim()
                  ├── Padding Cleanup   ──> trim() / trim_start() / trim_end()
                  └── Slicing & Lookup  ──> indexof() / substring() / replace_string()
```

## Technical Function Summary Reference

| **Advanced KQL Function**    | **Output Format** | **Core Processing Purpose**                                  |
| ---------------------------- | ----------------- | ------------------------------------------------------------ |
| **`countof(Source, Sub)`**   | Integer           | Counts instances of a small string inside a larger one.      |
| **`split(Source, Delim)`**   | Dynamic Array     | Breaks a single text string into a structured list.          |
| **`strrep(String, Count)`**  | String Text       | Repeats a specific text fragment `[Count]` times.            |
| **`extract(Regex, Target)`** | String Text       | Isolates a specific pattern match from an unmapped text row. |

# Quiz 24: String Functions

![[Pasted image 20260705213304.png]]
![[Pasted image 20260705213351.png]]
![[Pasted image 20260705213425.png]]
![[Pasted image 20260705213501.png]]

# Number Data Types

## 1. Operator Precedence and Evaluation Boundaries

When engineering calculations inside an `extend` or `project` operator, KQL evaluates standard mathematical symbols using standard algebraic priorities rather than reading linearly left-to-right.

$$\text{Parentheses ()} \longrightarrow \text{Multiplication (*)} \text{ and Division (/)} \longrightarrow \text{Addition (+)} \text{ and Subtraction (-)}$$

### The Precedence Trap

In the expression `EpisodeId - EventId * 5`, the engine evaluates `EventId * 5` first. It then subtracts that resulting product from the `EpisodeId`. If your design requires the subtraction to execute first, you must explicitly force the evaluation order using **parentheses `()`**:

```sql
-- Multiplies the product after subtraction
weather | extend Calc = (EpisodeId - EventId) * 5
```

## 2. The Integer Division Trap (Truncation)

A common point of failure on the DP-600 exam occurs when dividing whole numbers. In KQL, **dividing an integer by an integer yields an integer**, completely dropping any fractional decimal remainder. This is known as **Truncation**.

```sql
-- Truncated Result (Output: 0)
print 1 / 2

-- Fractional Result (Output: 0.5)
print 1 / 2.0
```

### The Inherent Typing Rule

- If a calculation involves _only_ fractionless data types (`int`, `long`), the engine truncates the remainder and outputs a whole number.
- To preserve decimals, at least one of the inputs must be explicitly typed or formatted as a fractional number type (`real`, `decimal`).

## 3. Numeric Data Types Matrix

KQL segments numbers into four core structural buckets based on memory allocation and scaling limits:

### Fractionless Whole Numbers

- **`int` (32-bit Integer):** Stores signed whole numbers ranging from approximately **$-2$ billion to $+2$ billion**.
- **`long` (64-bit Integer):** Stores much larger whole numbers (up to roughly 19 digits). **In KQL, un-suffixed whole number literals default to a `long` data type.**

  ```sq
  -- Explicitly casting types
  print ExplicitInt = int(5), ExplicitLong = long(5)
  ```

### Fractional Numbers

- **`real` (64-bit Floating-Point):** A double-precision floating-point type. **Un-suffixed decimal literals (like `5.0`) default to a `real` type.**
- **`decimal` (128-bit Decimal):** A high-precision, fixed-point decimal layout type.

## 4. Architectural Tradeoff: `real` vs. `decimal`

When choosing a data type to store or process fractional values, you must balance computational speed against absolute accuracy.

### The `real` Type (Speed over Precision)

The `real` type handles fractions using binary floating-point math. Because numbers are stored as binary approximations, massive calculation sweeps can introduce minor trailing floating-point errors (e.g., evaluating $5.0$ internally as $4.99999999999$ or $5.00000000001$).

- **Use Case:** Highly recommended for massive system infrastructure log streams, performance tracking, or IoT sensor feeds where **computational speed** is prioritized over microscopic precision.

### The `decimal` Type (Precision over Speed)

The `decimal` type utilizes a 128-bit architecture to store numbers with absolute mathematical accuracy, completely avoiding trailing rounding errors.

- **Use Case:** Mandatory for financial metrics, currency tracking, transactional auditing, or any business metric where floating-point rounding errors would corrupt reporting balances.
- **Performance Cost:** `decimal` math takes significantly longer to process than `real` calculations.

```sql
weather
-- Scenario: Eliminating trailing floating-point anomalies
-- Explicitly casting the divisor to a 128-bit decimal ensures clean, precise results
| extend PrecisionDivision = (EpisodeId - EventId) / decimal(5)
```

## KQL Numeric Data Types Quick Reference

| **KQL Data Type** | **Storage Size** | **Fractional Parts?** | **Underlying Accuracy Model**         | **Primary Target Use Case**                     |
| ----------------- | ---------------- | --------------------- | ------------------------------------- | ----------------------------------------------- |
| **`int`**         | 32-Bit           | No                    | 100% Precise Whole Numbers            | Compact IDs, small counts ($\le \pm2\text{B}$). |
| **`long`**        | 64-Bit           | No                    | 100% Precise Whole Numbers            | Large record counters, Default whole numbers.   |
| **`real`**        | 64-Bit           | **Yes**               | **Approximate** Binary Floating-Point | **High-speed** IoT telemetry & log metrics.     |
| **`decimal`**     | 128-Bit          | **Yes**               | **100% Exact** Fixed-Point            | Financial ledger auditing, cash balances.       |

# Other Math Functions

## 1. Scale Control and Rounding Mechanics

When working with fractional `real` or `decimal` values, you can control the decimal precision of your output metrics using two primary formatting functions:

### The `round()` Function

Truncates or elevates a number to the **nearest** specified decimal precision boundary.

- **Syntax Configuration:** `round(SourceColumn, DecimalPlaces)`
- **Behavior:** Passing `0` drops the entire fractional element, rounding to the nearest whole integer. Passing `1` locks the scale to exactly one position past the decimal point.

### The `ceiling()` Function

Forces a fractional value to always round **up** to the immediate next highest whole integer boundary.

- Unlike `round()`, `ceiling()` does not accept a second precision parameter—it automatically resolves to a whole number format.

```sql
-- Drop unnecessary precision cells cleanly
weather
| extend
    NearestTenth = round(RealCalculation, 1),
    NextWholeNum = ceiling(RealCalculation)
```

## 2. Sign Modification and Structural Signatures

When processing differential telemetry spans (such as calculating the variances between baseline configurations and active operational states), your calculations can yield erratic positive and negative outcomes. KQL provides targeted functions to normalize these scales:

### The `abs()` Function (Absolute Value)

Eliminates directional polarity entirely by stripping away negative signs (`-`), converting every negative integer or real number into its positive equivalent.

```sql
-- Ensures structural measurements evaluate as positive metrics
where abs(ValueDifference) > 10
```

### The `sign()` Function

Evaluates a column value and returns a standardized mathematical indicator mapping the source's structural signature:

- Returns **`-1`** if the evaluation value is strictly negative.
- Returns **`0`** if the evaluation value is exactly zero.
- Returns **`1`** if the evaluation value is strictly positive.

## 3. The Remainder Operator: Modulus (`%`)

To extract the fractional remainder left over following an integer division process, use the **percent symbol (`%`)**, which represents the **Modulus (Mod)** operator.

```sql
-- Captures the remainder left over from division
print Remainder = 9 % 4  -- Outputs: 1 (Since 4 goes into 9 twice, leaving a remainder of 1)
```

- **Use Case:** Modulus tracking is highly valuable when engineering cyclical pipeline loops, distributing row shards evenly across processing pools, or extracting time chunks out of raw second streams.

## 4. Exponential and Geometric Scaling Functions

To perform advanced physical scale shifts or apply polynomial transformations to raw data columns, utilize native geometric functions:

- **`pow()` (Power):** Elevates a number to a specified exponential power.
  - _Syntax Configuration:_ `pow(BaseValue, Exponent)`
  - _Example:_ Squaring an ID value (`ID` $\times$ `ID`) can be cleanly written as `pow(EpisodeId, 2)`.
- **`sqrt()` (Square Root):** Calculates the square root of a positive numeric column.
- **`rand()` (Random Number Generation):** Generates non-deterministic value distributions.
  - Calling `rand()` with no parameters outputs a random `real` fraction between `0.0` and `1.0`.
  - Passing an integer value ceiling argument (e.g., `rand(10)`) outputs a random whole integer falling strictly between **`0` and `9`**.

## KQL Advanced Math Function Reference

| **KQL Function**    | **Output Format Type**   | **Operational Objective**                                      |
| ------------------- | ------------------------ | -------------------------------------------------------------- |
| **`round(col, p)`** | Numeric (Real/Dec)       | Shifts a fraction to the nearest `[p]` decimal places.         |
| **`ceiling(col)`**  | Numeric (Whole)          | Automatically rounds a fractional cell up to the next integer. |
| **`abs(col)`**      | Numeric (Positive)       | Drops negative prefixes, returning absolute scalar magnitude.  |
| **`sign(col)`**     | Integer (`-1`, `0`, `1`) | Audits whether a data position is negative, zero, or positive. |
| **`%` (Operator)**  | Integer / Numeric        | Evaluates the leftover remainder from a division path.         |
| **`pow(B, E)`**     | Numeric                  | Multiplies the base value `[B]` by itself `[E]` times.         |
| **`rand(N)`**       | Integer (`0` to `N-1`)   | Injects a random distribution variable into the stream.        |

# datetime and timespan Data Types

## 1. The Temporal Data Types: Points vs. Durations

KQL splits time handling into two fundamentally distinct architectural data types:

- **`datetime`**: Represents a specific chronological point in time. It supports a wide storage range running from 0001-01-01 (1 C.E.) up to 9999-12-31.
- **`timespan`**: Represents an elapsed physical **duration** or interval of time. Subtracting one `datetime` from another automatically outputs a `timespan` object mapped in standard days, hours, minutes, and seconds (`d.hh:mm:ss.ffff`).

## 2. Declaring DateTime Points

KQL provides three native compilation pathways to explicitly declare a `datetime` point in time:

### Path A: The `datetime()` Literal

The most common and high-performance method to parse a standardized ISO 8601 date string literal.

```sql
-- Structure: datetime(YYYY-MM-DD hh:mm:ss.ffffff)
print TargetPoint = datetime(2030-02-03 01:23:45.6)
```

### Path B: The `todatetime()` Function

Dynamically casts an existing text string field into a true `datetime` format down the data pipeline.

```sql
print CastPoint = todatetime("2030-02-03 01:23:45.6")
```

### Path C: The `make_datetime()` Constructor

Assembles a point in time by passing individual, comma-separated integer variables representing each chronological segment.

```sql
-- Structure: make_datetime(year, month, day, hour, minute, second)
print ConstructedPoint = make_datetime(2030, 2, 3, 1, 23, 45.6)
```

## 3. Engineering Durations: TimeSpans

TimeSpans allow you to add offsets to static timestamps or evaluate operational processing intervals.

### Unit Suffix Shorthands

You can declare a `timespan` instantly by suffixing a number with an explicit time unit character literal:

- `d` $\rightarrow$ Days (e.g., `1d`, `3d`)
- `h` $\rightarrow$ Hours (e.g., `3h`, `23h`)
- `m` $\rightarrow$ Minutes (e.g., `45m`)
- `s` $\rightarrow$ Seconds (e.g., `17.8s`)
- `ms` / `microsecond` / `tick` $\rightarrow$ Sub-second intervals

### Mathematical Evaluation Rules

- **DateTime $\pm$ TimeSpan $=$ DateTime**: Appending a duration to a timestamp yields a shifted timestamp point.
  ```sql
  weather | extend RevisedEndTime = EndTime + 1d
  ```
- **TimeSpan $\times$ Number $=$ TimeSpan**: You can multiply a duration by a scalar number to scale its scale length (e.g., `3h * 3` evaluates to a `9h` duration).
- **TimeSpan / TimeSpan $=$ Real Number**: Dividing one duration by another calculates an exact frequency metric, telling you how many smaller intervals fit inside the larger period.
  ```sql
  -- Calculates how many 3-hour shift buckets exist inside a 23-hour duration
  print Result = 23h / 3h -- Outputs: 7.6666
  ```

## 4. Live Clock Functions: Evaluating the Present

To filter continuous streaming logs dynamically against your capacity server's active system clock, leverage real-time evaluation anchors:

### The `now()` Anchor Function

Returns the exact current UTC timestamp. You can pass an optional `timespan` argument straight into the parentheses to easily peek into future or past time windows.

```sql
print CurrentTime = now(), FutureTime = now(3h) -- now(3h) is equivalent to now() + 3h
```

### The `ago()` Tracking Function

A specialized, highly readable function designed to step backward into historical logs from the exact moment of execution.

```sql
-- These two filtering steps compile down to identical backend performance paths
weather | where StartTime > ago(3h)
weather | where StartTime > now() - 3h
```

## Temporal Operation Reference

| **Expression Input**      | **Resulting Data Type** | **Operational Purpose**                                                    |
| ------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| **`datetime - datetime`** | `timespan`              | Calculates the total duration between two dates.                           |
| **`datetime + timespan`** | `datetime`              | Offsets a date forward into the future.                                    |
| **`now()`**               | `datetime`              | Fetches the live system execution timestamp in UTC.                        |
| **`ago(12h)`**            | `datetime`              | Establishes a rolling lookback window anchor exactly 12 hours in the past. |
| **`timespan / timespan`** | `real`                  | Calculates the frequency ratio between two time intervals.                 |

# datetime and timespan Functions

## 1. Range Filtering and Boundary Anchors

### The `between` Operator

To filter rows within a specific chronological window in a `where` clause, KQL provides a highly optimized **`between`** operator. Instead of linking separate logic checking parameters with commas, KQL utilizes a **double-dot (`..`) syntax pattern** to establish range boundaries.

```sql
-- Evaluates an inclusive chronological boundary range
weather
| where StartTime between (datetime(2007-03-26) .. datetime(2007-04-15))
```

### Start/End Structural Alignment Truncation

Telemetry calculations frequently require rolling up hourly logs or sporadic events into unified boundaries (such as daily or monthly blocks). KQL features explicit point-anchoring rounding functions to trim raw timestamp precision:

- **`startofday()`** / **`endofday()`**
- **`startofweek()`** _(Note: The engine defaults the starting boundary of a week strictly to **Sunday**)._
- **`startofmonth()`** / **`endofmonth()`**
- **`startofyear()`** / **`endofyear()`**

```sql
weather
-- Instantly truncates a detailed time stamp down to 12:00:00 AM midnight of that day
| extend DailyBucket = startofday(StartTime)
```

## 2. Temporal Element Extractions

To pull a specific numeric extraction out of a `datetime` field, you can leverage standalone targeting functions or use a master evaluation utility.

### Standalone Extraction Matrix

- **`dayofweek()`**: Extracts a zero-based integer tracking the day index, where **`0` represents Sunday** and `6` represents Saturday.
- **`hourofday()`** / `dayofmonth()` / `dayofyear()` / `weekofyear()` / `monthofyear()`

### The Unified `datetime_part()` Engine Function

Instead of remembering multiple individual functions, you can pass a uniform string literal configuration parameter into **`datetime_part()`** to isolate an explicit metadata element:

```sql
-- Structure: datetime_part("TargetElement", DateTimeColumn)
weather
| extend ISOWeekNum = datetime_part("week_of_year", StartTime)
```

- _Supported Element Targets:_ `"year"`, `"quarter"`, `"month"`, `"day"`, `"day_of_year"`, `"hour"`, `"minute"`, `"second"`.

## 3. Directional Arithmetic and Date Differencing

While standard math symbols like `+ 2d` shift timestamps easily, enterprise schemas can implement advanced programmatic arithmetic wrappers.

### The `datetime_add()` Function

Appends a duration interval metric directly to an existing timestamp variable block.

```sql
-- Structure: datetime_add("UnitPeriod", ValueOffset, TargetColumn)
weather | extend ShiftedDate = datetime_add("day", 2, StartTime)
```

### The `datetime_diff()` Function (Truncated Interval Tracking)

When calculating the distance between two distinct points in time, standard column subtraction (`EndTime - StartTime`) returns a continuous precision `timespan` (e.g., `1 day, 11 hours`).

If your visualization reporting metrics only care about absolute elapsed boundaries, use **`datetime_diff()`**. This function calculates how many discrete boundary thresholds were crossed, ignoring smaller fractions.

```sql
-- Structure: datetime_diff("UnitPeriod", EndTimePoint, StartTimePoint)
weather
| extend DaysElapsed = datetime_diff("day", EndTime, StartTime)
```

> **Critical Execution Rule:** Unlike custom calculations, `datetime_diff()` requires passing the **newer end timestamp first** as the second argument, followed by the older start timestamp as the third argument. If a log spans across late night to the following morning (e.g., an 18-hour delta crossing from the 26th to the 27th), `datetime_diff` records this as exactly `1` full day elapsed.

## 4. Timezone Conversions and Multi-Region Mapping

By default, data ingested into Fabric Eventhouses maps to standard **UTC / GMT**. To shift the display to local regional matrices, use built-in conversion utilities:

- **`datetime_utc_to_local(Time, "ZoneName")`**: Shifts a UTC baseline timestamp to a targeted regional grid. (e.g., converting UTC midnight down to `8:00 PM` the previous evening for the `"US/Eastern"` zone).
- **`datetime_local_to_utc(Time, "ZoneName")`**: Normalizes local time metrics backward into standard UTC variables.

### Auditing Timezone Systems

To inspect the exact, string literal naming variables recognized by the KQL engine cluster, evaluate the **`datetime_list_time_zones()`** catalog array list. To expand that output array into clean, vertical lookup reference table records, pair it with the **`mv-expand`** operator:

```sql
print Timezones = datetime_list_time_zones()
| mv-expand Timezones
```

## 5. String Mask Formatting: Custom Display Layouts

To transform raw data markers into human-readable text strings for production dashboard presentations, use the `format_datetime()` and `format_timespan()` functions.

```sql
weather
| extend
    CustomDateStr = format_datetime(StartTime, "dd-MM-yyyy"),
    CustomSpanStr = format_timespan(EndTime - StartTime, "d 'days' hh:mm:ss")
```

### Critical Casing Syntax Mask Rules

| **Format Token Mask** | **Targeted Segment**                    | **Output Text Format Example**                |
| --------------------- | --------------------------------------- | --------------------------------------------- |
| **`yyyy`**            | Full Calendar Year                      | `2007`                                        |
| **`MM`**              | Month Token (Must be **UPPERCASE**)     | `03` _(Note: lowercase `mm` maps to minutes)_ |
| **`dd`**              | Day of Month                            | `26`                                          |
| **`HH`** / `hh`       | Hour Metric (24-Hour vs 12-Hour format) | `23` / `11`                                   |
| **`mm`**              | Minute Token (Must be **lowercase**)    | `45`                                          |
| **`ss`**              | Second Marker                           | `17`                                          |
| **`ff`** to `fffffff` | Fractional Millisecond Sub-elements     | Scales up to 7 decimal positions.             |
| **`tt`**              | Operational Meridian Marker             | `AM` or `PM`                                  |

# Quiz 25: Number and date data types and functions

![[Pasted image 20260705224215.png]]
![[Pasted image 20260705224244.png]]
![[Pasted image 20260705224349.png]]
![[Pasted image 20260705224434.png]]
![[Pasted image 20260705224518.png]]

# 29. Transforming data using KQL

# Merging data

## 1. KQL Pipeline Scope and Derived Tables

The **`union`** operator takes two or more tabular datasets and stacks their rows vertically into a single consolidated output stream.

### The Parentheses Boundary Trap

Because KQL interprets queries as a continuous linear data pipeline, failing to explicitly isolate a downstream query block will break your logic. The execution engine will treat trailing modifiers as though they apply to the entire combined stream rather than individual subsets.

To safely union a query with a **derived table** (the output of a separate, distinct query expression), you must wrap the downstream query completely inside **parentheses `()`**:

```sql
-- CORRECT: The parentheses explicitly isolate the second query block
weather
| project State, EventType
| union (weather | project State, EventNarrative)
```

## 2. Structural Schema Mapping: KQL vs. SQL Union

A major architectural difference between relational SQL and KQL is how they handle mismatched columns during a union operation.

- **The SQL Behavior:** SQL maps columns strictly by their relative ordinal position. If Query 1 outputs `State` and `EventType`, and Query 2 outputs `State` and `EventNarrative`, SQL forces `EventNarrative` into the second column spot, corrupting your data alignment.
- **The KQL Behavior:** KQL maps columns strictly by their **explicit header name strings**. If columns do not find an exact naming match, KQL expands the output schema horizontally to accommodate both fields, padding mismatched cells with empty values.

## 3. Merging Behavior Modes: `kind=inner` vs. `kind=outer`

You can control how the schema handles column mismatches by passing the optional **`kind`** parameter suffix to the operator:

### `kind=outer` (Default System Behavior)

Returns a global union containing **every unique column** found across all participating inputs. If a row originates from a query that lacks a specific column, that cell returns empty.

### `kind=inner`

Filters the schema tightly, returning **only the intersecting columns** that exist across every single table or query in the pipeline.

```sql
-- Returns ONLY the 'State' column because it is the only shared header string
weather
| project State, EventType
| union kind=inner (weather | project State, EventNarrative)
```

## 4. Auditing Data Lineage: The `withsource` Property

When debugging massive, multi-stream event telemetry networks, you often need to audit exactly which physical table or query block generated a specific row. To achieve this, pass the **`withsource=`** tracking parameter:

```sql
weather
| project State, EventType
-- Appends a tracking column named 'TableSource' populated with "Table0", "Table1", etc.
| union withsource=TableSource (weather | project State, EventNarrative)
```

- **Impact:** The engine dynamically creates a brand-new column matching your parameter string. Rows from the primary stream are marked as `Table0`, while rows from the parenthesized derived query block are labeled as `Table1`. If you are unioning named table assets directly, this column automatically captures the literal string names of those databases.

## 5. Advanced Bulk Scaling: Wildcard Table Unions

If your workspace ingests time-sliced chunks of data into separate tables matching a regular naming pattern (such as `weather_2026_Jan`, `weather_2026_Feb`), you do not need to type out dozens of manual union statements. KQL natively supports **Wildcard Pattern Unions**:

```sql
-- Stacks every single table in the database whose name begins with the prefix string "we"
union withsource=OriginalTable we*
```

## KQL Union Configuration Reference

| **Operator Configuration** | **Schema Output Type**  | **Shared Fields Only?** | **Best Used For**                                                  |
| -------------------------- | ----------------------- | ----------------------- | ------------------------------------------------------------------ |
| **`union`** (Naked)        | Outer Schema Map        | No                      | Standard, non-destructive row stacking.                            |
| **`kind=outer`**           | Outer Schema Map        | No                      | Explicit declaration of a full structural merge.                   |
| **`kind=inner`**           | Intersecting Schema Map | **Yes**                 | Stripping out mismatched columns to unify core dimensions.         |
| **`withsource=ColName`**   | Lineage Audited Map     | N/A                     | Adding debugging tracers to identify records across multiple logs. |
| **`union prefix*`**        | Wildcard Stacking Map   | No                      | Consolidating shards of rolling time-slice logs instantly.         |

# Joining data

## 1. Relational Matching: The `join` Operator

The **`join`** operator merges two tabular streams horizontally by matching row keys, appending columns from the right-side table onto the left-side stream.

### Explicit Multi-Table Key Referencing

Unlike SQL, where tables are joined using their catalog names (e.g., `ON weather.State = region.State`), KQL uses generalized positional variables:

- **`$left`**: References the upstream tabular data flowing into the join operator.
- **`$right`**: References the external table asset specified inside the join argument.

```SQL
weather
| join kind=inner (region) on $left.State == $right.State
```

### Joining Over Mismatching Casing

Because KQL functions are strictly case-sensitive, key matching will fail if one table uses uppercase strings (`"TEXAS"`) and the other uses proper-case strings (`"Texas"`).

You **cannot** embed a transforming function directly inside the `on` criteria clause. Instead, you must inject a **derived table** by executing an inline transformation on the right-hand asset using parentheses:

```sql
weather
-- Force the right-side state names to uppercase before the join evaluates
| join kind=inner (
    region
    | extend State = toupper(State)
) on $left.State == $right.State
```

## 2. KQL Join Flavors (`kind=`) and The Default System Trap

The most critical concept for the DP-600 exam is understanding KQL's default join behavior. In relational SQL, a standard `JOIN` defaults to an `INNER JOIN`. In KQL, **a naked `join` statement defaults to an `innerunique` join.**

### `kind=innerunique` (The System Default)

Deduplicates keys on the **left side** first, returning only the very first matching row instance for each unique key. Joining a 59,000-row telemetry table with a 50-row region dimension using `innerunique` will collapse your final dataset down to exactly **51 rows**, stripping out thousands of streaming events.

### `kind=inner`

The traditional relational join mapping. It preserves all row dimensions on both sides, outputting an explicit match for every intersecting record (e.g., yielding the full 57,500 streaming rows).

### `kind=leftouter` / `kind=rightouter` / `kind=fullouter`

- `leftouter`: Retains 100% of the rows from the left stream, filling the right-side columns with empty `null` blocks if a dimension key (like an offshore weather station code) is missing from the region lookup asset.
- `rightouter`: Retains all dimensions from the right asset (including entries like `"England"`, which are absent from the weather log).
- `fullouter`: Stacks all matching and mismatching boundaries from both layers concurrently.

## 3. The `lookup` Operator: Fast Dimension Appending

The **`lookup`** operator is a highly optimized, specialized version of the `join` operator. It is specifically designed to enrich high-frequency streaming fact rows with descriptors from a small, static dimension table.

```sql
weather
| lookup (region | extend State = toupper(State)) on State
```

### Major Architectural Differences: `lookup` vs. `join`

- **Default Flavor Mapping:** Unlike `join`, the default behavior for `lookup` is a **`leftouter`** join. It guarantees that no rows from your primary streaming data are dropped or accidentally deduplicated.
- **Cardinality Restrictions:** `lookup` explicitly **forbids many-to-many relationships**. It supports only 1-to-1 or many-to-1 mappings.
- **Memory Constraints:** The right-hand lookup table **must be small** (typically limited to a maximum scale of tens of megabytes). If the dimension table is too large, the engine triggers an allocation failure, and you must use `join` instead.

## 4. Query Performance Optimization Strategies

To maximize processing speeds across your Fabric capacity, memory placement must shift depending on whether you deploy a `join` or a `lookup`:

- **Join Performance Optimization Rule:** Place the **smaller table on the LEFT** side of the expression, and push the massive streaming data asset to the RIGHT side inside the parentheses.
- **Lookup Performance Optimization Rule:** Place the **massive streaming fact table on the LEFT**, and keep the compressed, smaller dimension table on the RIGHT side inside the parentheses.

## Horizontal Merging Comparison Reference

| **Feature Matrix Comparison** | **KQL join Operator**                     | **KQL lookup Operator**                 |
| ----------------------------- | ----------------------------------------- | --------------------------------------- |
| **System Default Flavor**     | `kind=innerunique` (Deduplicates Left)    | `kind=leftouter` (Preserves Left)       |
| **Supported Join Layouts**    | Inner, InnerUnique, Left/Right/Full Outer | LeftOuter, Inner                        |
| **Many-to-Many Support?**     | **Yes**                                   | **No** (Strictly 1:1 or Many:1)         |
| **Right-Side Size Limit**     | Unrestricted (Scales with capacity)       | Restricted strictly to a few Megabytes. |
| **Optimal Setup Layout**      | **Smaller table on the Left.**            | **Smaller table on the Right.**         |

# Merging and joining data

![[Pasted image 20260706211149.png]]
![[Pasted image 20260706211242.png]]
![[Pasted image 20260706211322.png]]
![[Pasted image 20260706211358.png]]

# Identify and resolve duplicate data, missing data, or null values

## 1. Auditing and Isolating Duplicate Records

While using `distinct` or a naked `summarize by` cleanly deduplicates a row stream, it hides the true volume of your duplicate data. To explicitly locate and measure duplicate entries, build an aggregation filter down the pipeline.

```sql
weather
| project State, Region
-- Step 1: Compress rows and calculate frequency per combination
| summarize RecordCount = count() by State, Region
-- Step 2: Act as a relational 'HAVING' clause to isolate duplicates
| where RecordCount > 1
```

## 2. Empty Strings vs. Null Values

When analyzing the output of mismatched horizontal operations (such as a `fullouter` join), missing values can present as either an empty text block (`""`) or a true computational **`null`** token.

### The Dual-Condition Approach

To build a bulletproof filter that catches both empty cells and true database nulls safely, pair your conditions with an `or` gate inside parentheses:

```sql
where TargetColumn == "" or isnull(TargetColumn)
```

### Streamlining with `coalesce()`

Instead of writing repetitive, multi-condition logic blocks, deploy the **`coalesce()`** function. `coalesce()` evaluates an ordered array of arguments from left to right and returns the **very first non-null value** it encounters.

```sql
-- If 'State1' is null, coalesce falls back to the secondary argument ("")
where coalesce(State1, "") == ""
```

## 3. High-Performance Isolation: The Anti-Join Family

While using a `fullouter` or `leftouter` join combined with a `where isnull()` clause successfully catches missing records, it forces the Fabric capacity engine to compile and stream all right-side column assets into memory before throwing them away.

To optimize performance, use an **Anti-Join**. Anti-joins discard matching rows during the compilation phase, and they only return columns belonging to the primary table.

### `kind=leftanti` (or `leftantisemi`)

Returns **only** the records from the left-hand table that fail to find any matching keys inside the right-hand table.

```sql
weather
-- Instantly isolates rows in weather that have no corresponding match in region
| join kind=leftanti (region) on State
```

- **Performance Benefit:** The output grid drops all right-side lookup columns entirely, stripping out empty cell noise and saving vertical network pipeline memory.

### `kind=rightanti` (or `rightantisemi`)

The exact positional inverse. It isolates records existing exclusively inside the right-hand reference table (e.g., catching lookup states like `"England"` that were never recorded in the active streaming fact tables).

## Technical Summary Matrix: Missing & Duplicate Filters

| **Optimization Target**   | **KQL Operational Pattern**                             | **Schema Output Footprint**        | **Best Architectural Use Case**                          |
| ------------------------- | ------------------------------------------------------- | ---------------------------------- | -------------------------------------------------------- |
| **Locate Duplicates**     | `summarize c=count() by...` $\rightarrow$ `where c > 1` | Aggregated metrics table           | Auditing telemetry integrity and profiling data quality. |
| **Coalesce Blanks**       | `coalesce(Column, "Fallback")`                          | Maintained column state            | Normalizing mixed `null` and empty text strings.         |
| **Isolate Missing Left**  |                                                         | `join kind=leftanti (RightTable)`  | Left table columns only                                  |
| **Isolate Missing Right** |                                                         | `join kind=rightanti (RightTable)` | Right table columns only                                 |

# iif/iff and case conditional functions

## 1. Inline Boolean Evaluation: The `iif` Function

When you need to execute a simple binary evaluation (true/false choice) to categorize streaming logs on the fly, deploy the inline if function.

### Structural Syntax Pattern

In KQL, the standard programming keyword `if` is invalid for standalone row transformations. Instead, you must use one of its twin functional aliases: **`iif()`** or **`iff()`**. They compile down to the exact same execution paths.

```sql
-- Structure: iif(LogicalPredicate, ResultIfTrue, ResultIfFalse)
weather
| extend StateGroup = iif(State in ("TEXAS", "FLORIDA"), "Texas/Florida", "Other")
| summarize EventCount = count() by StateGroup
```

### Argument Layout Mechanics

The function strictly expects exactly **three arguments**:

1. **The Predicate:** The boolean condition being evaluated (e.g., checking if the value belongs to a set or matches an expression).
2. **The True Branch:** The value injected into the column if the condition evaluates to true.
3. **The False Branch (The Fallback):** The value injected if the condition evaluates to false. Failing to provide this fallback parameter will trigger a compilation syntax error.

## 2. Multi-Branch Logical Routing: The `case` Function

When your data transformations scale past a basic binary choice and require multiple nested evaluation rules, switching from `iif` to the **`case`** function optimizes script legibility and engine efficiency.

### Structural Syntax Pattern

The `case` function behaves similarly to a `SWITCH` statement or a nested `IF...ELSE` loop in traditional programming languages. It evaluates an array of condition/result pairs sequentially from left to right.

```sql
weather
| extend RegionalPriority = case(
    State == "TEXAS",   "High Priority - South",
    State == "FLORIDA", "High Priority - East",
    State == "ALASKA",  "Special Window",
    "Standard Tracking" -- The mandatory final 'Else' fallback value
)
```

### The Odd-Argument Rule

A common troubleshooting trap tested on technical assessments is the **Odd-Argument Constraint** of the KQL `case` function:

- The function accepts an arbitrary number of conditions, but it **strictly requires an odd number of total arguments** to execute.
- This is because every condition parameter _must_ have a matching result parameter (even numbers), plus **one absolute final, unmatched argument acting as the default `else` fallback**.

> **Compilation Error Warning:** If you map three conditions to three results (totaling 6 arguments) without adding a final default text string or fallback token, the engine will block execution and return the error: _"The case() function expects an odd number of arguments."_

## KQL Conditional Logic Quick Reference

| **Function Name** | **Supported Aliases** | **Argument Count Rule**   | **Primary Architecture Use Case**                                                 |
| ----------------- | --------------------- | ------------------------- | --------------------------------------------------------------------------------- |
| **`iif()`**       | `iff()`               | **Exactly 3** arguments   | High-performance binary flags or true/false conditional tagging.                  |
| **`case()`**      | None                  | **Must be an Odd Number** | Complex, multi-layered bucketing and replacing deeply nested `iif` logic strings. |

# The OneLake data and real-time hubs + implementing OneLake integration

## 1. Data Discovery: OneLake Data Hub vs. Real-Time Hub

Microsoft Fabric provides two distinct structural planes for browsing, auditing, and consuming data assets across your enterprise:

- **OneLake Data Hub:** Acts as the central corporate repository for discoverable, high-level artifacts. It organizes data by workspace containers, allowing engineers to locate master **Eventhouses** and child **KQL Databases** to check settings or review metadata permissions.
- **Real-Time Hub:** Focuses strictly on live, moving data telemetry streams rather than abstract structural folders. Instead of displaying a parent Eventhouse container, it exposes the granular, underlying ingestion **tables** directly.

### Operational Actions in the Real-Time Hub

From the Real-Time Hub stream inventory, you can immediately execute low-code operational tasks:

- **Endorse Data:** Apply certifications or promotional tags to verify raw tables as authoritative data sources for the company.
- **Explore Data:** Launches an interactive scratchpad pre-populated with basic KQL sampling commands (`| take 10`) to let you quickly audit live columns.

## 2. Eventhouse OneLake Integration Mechanics

By default, an Eventhouse captures high-velocity, append-only real-time streams and holds them inside its own optimized Kusto ingestion cache. Activating **OneLake Integration** bridges this gap by automatically writing a parallel copy of incoming records out to OneLake as standard **Delta Parquet** files.

```
STREAMING INGESTION ──> KQL Database Ingestion Cache
                             │
                             └── (OneLake Integration Active) ──> Delta Parquet Export ──> Lakehouses, Warehouses, Notebooks
```

### Scope Levels for Activation

- **Table Scope:** Turns on mirroring strictly for that individual table object.
- **Database Scope:** Activates mirroring globally across the entire database workspace. When choosing this option, you can check **"Apply to existing tables"** to ensure all future structures mirror out automatically.
- _Note:_ If you encounter a bug attempting to toggle integration on an empty KQL database, simply seed a blank placeholder table first to unblock the configuration layer.

### The Delta Mirror Delay

Once active, the platform surfaces a direct file link copyable into the **OneLake File Explorer**. However, because real-time streaming inputs must be grouped and batched into optimized columnar Parquet shapes, **it can take up to several hours** for newly streamed rows to physically appear inside the downstream OneLake directory layout.

## 3. Strict Architectural Guardrails & Risks

Turning on OneLake integration introduces irreversible architectural constraints. **It should generally be left disabled during initial agile prototyping phases.**

> **Critical Exam Top-Priority Warnings:**
>
> 1. **Data Deletion Inversion:** Once integration is activated for a table, **data cannot be removed or deleted** from that integrated copy.
> 2. **Structural Lockdown:** You lose the ability to rename the table asset.
> 3. **Security Incompatibility:** Downstream **Row-Level Security (RLS)** configurations _cannot_ be applied to the mirrored Delta Parquet files sitting in OneLake.
> 4. **Historical Gap Rule:** Activating integration **does not backfill historical records**. Only new streaming data hitting the table _after_ the switch is turned on will export out to the OneLake folder.

## 4. Semantic Model OneLake Integration

Just as real-time KQL metrics can be exported to OneLake, traditional compressed **Import Mode Semantic Models** can also be exposed as open Delta tables, allowing engineers to query BI models using standard Spark Notebooks or SQL endpoints.

### Prerequisites for Semantic Model Export

- **Capacity Limit:** The parent workspace must be explicitly backed by a dedicated **Fabric Capacity** or **Power BI Premium Capacity**.
- **Tenant Governance Switch:** A Fabric Administrator must explicitly activate the governance toggle within the **Admin Portal** tenant settings under:
  > _“Semantic models can export data to OneLake”_

## OneLake Integration Configuration Checklist

| **Artifact Target**       | **Storage Output Format** | **Core Integration Benefit**                                                                   | **Major Engineering Tradeoff**                                                                  |
| ------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **KQL Database / Table**  | Delta Parquet             | Exposes real-time logs to Spark Notebooks, Lakehouses, and SQL Warehouses seamlessly.          | **Permanent change.** Prevents table renames, row deletions, and blocks active RLS enforcement. |
| **Import Semantic Model** | Delta Parquet             | Allows cross-functional data scientists to query curated Power BI datasets using Python/Spark. | Requires active Premium/Fabric SKU capacity and a dedicated Tenant Admin switch activation.     |

# Quiz 27: Transforming data using KQL

![[Pasted image 20260706222242.png]]
![[Pasted image 20260706222453.png]]

# 30. Configure security and governace, and deployment pipelines.md

# Apply sensitivity labels to items

## 1. Core Architecture and Information Protection Boundaries

Sensitivity labels leverage **Microsoft Purview Information Protection** to classify and safeguard enterprise data assets across the Fabric fabric.

> **Critical Exam Rule:** Sensitivity labels **do not control or restrict access** to content within the Fabric or Power BI service environments. Access is governed purely by Workspace Roles, Item Permissions, and Share Links. Instead, labels act as persistent metadata banners and enforcement anchors.

### Downstream Ingestion and Export Protection

The primary value of a sensitivity label occurs when data boundaries are crossed. The label—and its associated encryption or protection policies—automatically follows the data into physical file formats during the following operations:

- Exporting reports or grids to **Excel, PowerPoint, or PDF**.
- Launching **Analyze in Excel** pivots against a centralized Semantic Model.
- Downloading a report from the service as a local **`.pbix`** file.

## 2. In-Service Label Assignment and Downstream Inheritance

### How to Apply Labels

1. **Header Banner:** Open an asset (such as a Lakehouse, Report, or Warehouse) and select the label drop-down from the top-center menu bar.
2. **Item Settings:** Navigate to the workspace row, click the ellipses (`...`), select **Settings**, and expand the **Sensitivity Labels** tab section. (Mandatory approach for background items like Dataflows or Pipelines that lack a persistent viewing canvas).

### The Inherent Downstream Cascade

When you assign a label to a upstream root data source item, that label automatically flows forward to any child objects generated from it:

```
[Lakehouse] (Marked as Confidential)
    │
    └──> [Default Semantic Model] ──> Inherits "Confidential"
             │
             └──> [Power BI Report] ──> Inherits "Confidential"

```

### Distinguishing "None" vs. "No Sensitivity"

- **`None`**: Indicates the item has **not yet been assessed** by a data steward. It carries no classification payload.
- **`No Sensitivity` / `Public`**: Indicates a formal assessment has occurred, and the data has been verified as safe for low-level or general distribution.

## 3. Microsoft Purview Policy Configuration

Master label schemas are created, published, and ranked within the **Microsoft Purview Compliance Portal** (`purview.microsoft.com`).

### Label Publishing Policy Enforcements

When rolling out labels to Fabric users via a Publishing Policy, administrators can enforce two critical corporate compliance gates:

- **Mandatory Labeling:** Restricts users from saving new Fabric items or Power BI reports until they select a valid sensitivity level.
- **Downgrade Justification:** If a developer attempts to lower an item’s classification tier (e.g., changing a Semantic Model from _Highly Confidential_ down to _Confidential_), they **must provide a text justification reason** which is logged to the corporate audit trail.

## 4. Tenant-Level Governance Settings

Within the Fabric **Admin Portal Tenant Settings**, administrators control how security parameters react to upstream changes.

### Policy Switch 1: Upstream Source Inheritance

Allows Fabric Semantic Models to automatically inherit pre-existing Purview labels attached directly to cloud data sources. This is strictly supported for cloud-native storage assets, including **Excel files hosted on OneDrive/SharePoint Online**, **Azure Synapse Analytics**, and **Azure SQL Database** (on-premises sources behind local data gateways are excluded).

### Policy Switch 2: Downstream Update Automation

Controls what happens when an upstream label is _changed_ or updated:

- **Disabled (Default):** Downstream items only capture the label that existed when they were first created. Subsequent updates to the parent Lakehouse label will not cascade forward automatically.
- **Enabled:** Changes immediately ripple down the entire data lineage tree. Workspace administrators can be granted explicit override flags to block the automatic inheritance cascade if a specific child report requires unique handling.

## Sensitivity Label Governance Matrix

| Policy Feature Gate       | Managed Location                  | Requires Pro/PPU License? | Core Governance Impact                                                                     |
| ------------------------- | --------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------ |
| **Apply/View Labels**     | Fabric Service / Power BI Desktop | **Yes**                   | Displays metadata banners and encrypts data upon external file export.                     |
| **Mandatory Label Rules** | Microsoft Purview Portal          | N/A (Admin Side)          | Forces users to classify content before saving or publishing.                              |
| **Cloud Source Ingest**   | Fabric Tenant Admin Settings      | N/A (Admin Side)          | Automatically pulls labels into models from SharePoint Excel or Azure SQL.                 |
| **Downstream Cascade**    | Fabric Tenant Admin Settings      | N/A (Admin Side)          | Automatically pushes label updates down the lineage path (Lakehouse $\rightarrow$ Report). |

# Endorse items

## 1. The Item Endorsement Hierarchy

Fabric features a tiered quality-control system designed to help users identify accurate, high-quality data sources. Endorsements establish clear metadata tracking signs that surface directly inside the OneLake Data Hub and Real-Time Hub.

```
                      ┌─── [ MASTER DATA ] ───> Single Source of Truth (Databases Only) - Reviewers Only
Increasing Authority  ├─── [ CERTIFIED ]   ───> Org-Wide Quality Control - Authorized Reviewers Only
                      └─── [ PROMOTED ]    ───> General Shared Quality - Item Owners & Contributors

```

### Tier 1: Promoted

- **Scope:** Can be applied to any Fabric item layout asset **except Power BI Dashboards**.
- **Permissions:** Any user with write permissions (Contributor role or higher) can instantly apply a Promoted status tag.
- **Objective:** Acts as an internal advertisement signal, letting colleagues know a report or model is ready for general collaboration.

### Tier 2: Certified

- **Scope:** Can be applied to any Fabric item layout asset **except Power BI Dashboards**.
- **Permissions:** Restricted strictly to an explicit security group of **Authorized Reviewers** defined by Fabric Governance Administrators.
- **Objective:** Certifies that the underlying asset is heavily audited and completely safe for official corporate-wide business intelligence dependencies.

### Tier 3: Master Data

- **Scope:** Restricted exclusively to storage and semantic layers (**Lakehouses, Data Warehouses, and Semantic Models**). It is completely blocked for presentation layers like reports or dashboards.
- **Permissions:** Restricted strictly to dedicated governance security groups.
- **Objective:** Elevates a database asset to the absolute **Single Source of Truth** for a core business domain (e.g., the official corporate general ledger repository).

## 2. Model Discovery and OneLake Hub Integration

When you endorse a **Semantic Model**, the system automatically triggers its **Discovery Configuration** state.

- **The Blind Metadata Search:** Marking a semantic model as discoverable allows developers across the company to find its record container inside the OneLake Hub or Real-Time Hub, even if they do not yet have read access to the data cells.
- **Granular Indexing:** Users can search the catalog for matching table header paths, column names, or model descriptions. If they choose to build a report from it, they can use the system to instantly request the necessary access permissions from the owner.

## 3. Tenant Administration and URL Intercepts

Tenant Administrators govern the structural parameters of endorsements through the **Fabric Admin Portal Tenant Settings**.

### Restricting Endorsement Rights

Within the tenant configuration, admins can isolate who has the authority to grant Certified or Master Data badges by mapping those permissions to specific, corporate Azure Active Directory (AAD) Security Groups.

### Overriding the Documentation Link Block

By default, clicking the _"How do I get content certified?"_ or _"How do I get content endorsed as master data?"_ guidance hyperlinks inside an item’s settings pane redirects users to general Microsoft documentation.

To operationalize this for an enterprise workflow, Tenant Admins can override this behavior by pasting an internal company URL path into the settings:

> **Admin Portal Route:** Tenant Settings $\rightarrow$ Certification $\rightarrow$ _“Specify a URL web page for documentation”_

Providing a custom link (such as an internal SharePoint ticket intake form) replaces the default destination, directing developers straight to your company’s custom data audit submission process.

## Endorsement and Governance Compatibility Reference

| Endorsement State | Supported Item Scope Profiles                | Who Can Assign It?                | Hub Discovery Behavior                                |
| ----------------- | -------------------------------------------- | --------------------------------- | ----------------------------------------------------- |
| **Promoted**      | All assets (Excludes Power BI Dashboards)    | Anyone with Item Edit rights.     | Visible across general hub search feeds.              |
| **Certified**     | All assets (Excludes Power BI Dashboards)    | Authorized Reviewers Group only.  | Configurable via explicit Admin Tenant Switch.        |
| **Master Data**   | Lakehouses, Warehouses, Semantic Models only | Master Data Reviewers Group only. | Anchored as a master corporate data catalog priority. |

# Row-level security in a Data Warehouse

## 1. Fabric RLS Architecture: The Two-Component Model

Row-Level Security (RLS) in a Fabric Data Warehouse relies on an inline database security engine. It intercepts raw table queries and injects background filter criteria seamlessly without changing the front-end query structure.

RLS is built using a strict two-part architecture:

1. **The Security Predicate Function:** An inline Table-Valued Function (TVF) that evaluates the identity of the active session context (`USER_NAME()`) against data rows.
2. **The Security Policy:** The database wrapper that binds the function to a physical target table and turns the tracking state to `ON`.

## 2. Phase 1: Engineering the Security Predicate Function

The security function acts as a logical gateway. It accepts a column data parameter from a row, executes string validation matching, and must return a **single-cell row containing the integer `1`** for the row to pass the security filter.

### The System Username Validation Trick

In Fabric, cloud identities are passed down into the SQL engine as long User Principal Name (UPN) string emails (e.g., `susan@yourcompany.com`). If your data table stores short names (`"Susan"`), a standard direct equality match (`==`) will fail.

To overcome this, use the **`LEFT()`** and **`LEN()`** functions to dynamically slice the incoming system name down to match the exact string length of your data variable:

```sql
CREATE FUNCTION dbo.fn_securitypredicate (@UsernameVar VARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN
    SELECT 1 AS fn_securitypredicate_result
    WHERE
        -- Rule 1: Match row data by dynamically trimming the long UPN email string
        LEFT(USER_NAME(), LEN(@UsernameVar)) = @UsernameVar

        -- Rule 2: Super-user override (If the caller is 'Jane', return 1 globally)
        OR LEFT(USER_NAME(), 4) = 'Jane';

```

### Critical Syntax Rules for the Exam

- **`WITH SCHEMABINDING`**: This clause is **mandatory**. It locks the underlying table structure down, preventing changes to data types that would break the security function.
- **Two-Part Naming Schema (`dbo.`)**: You must explicitly declare the target schema prefix (Database Owner) when creating or altering database security structures.

## 3. Phase 2: Binding with the Security Policy

Once the function is compiled into memory, you must bind it to the physical table using a **Security Policy**. The policy monitors the table, captures the values from the designated security column, and passes them straight into the function's internal parameters as rows pass through the query stream.

```sql
CREATE SECURITY POLICY dbo.SecurityPolicy
ADD FILTER PREDICATE dbo.fn_securitypredicate(UserNameColumn)
ON dbo.TableActual
WITH (STATE = ON);

```

- **The Predicate Column Assignment:** The parameter variable inside the parentheses `(UserNameColumn)` must reference the actual column name housing ownership strings inside your fact table (`dbo.TableActual`).
- **`STATE = ON`**: Activates enforcement immediately. You can toggle this parameter to `OFF` during database migration windows or maintenance cycles.

## 4. Evaluation and Validation Matrix

Once RLS is activated, user experience splits down distinct identity paths despite everyone running the exact same `SELECT * FROM TableActual` script:

```
[SELECT * FROM TableActual]
       │
       ├──> Logged in as Jane  ──> Override Evaluates True  ──> Returns ALL Rows
       ├──> Logged in as Susan ──> Direct Email Slice Match ──> Returns 3 Rows
       └──> Logged in as Admin ──> No Rules Match Matrix    ──> Returns 0 Rows

```

### Shared Workspace Permission Nuance

When sharing a Data Warehouse object with a user or security group using the workspace interface, granting **Read all data (SQL Read)** permissions _does not_ bypass or override your active database RLS policies.

The security policy takes absolute precedence at the engine level—Susan will still be restricted to her 3 rows, and Jane will still see all 6 rows, maintaining strict compliance across your reporting tiers.

## RLS T-SQL Component Quick Reference

| SQL Security Component | Technical Data Type | Required Parameters/Modifiers         | Core Architectural Purpose                                                     |
| ---------------------- | ------------------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| **`USER_NAME()`**      | System Function     | None                                  | Fetches the active caller's Microsoft Entra ID UPN string.                     |
| **Inline TVF**         | Database Function   | `WITH SCHEMABINDING`                  | Evaluates filtering criteria; outputs a `1` on a successful match.             |
| **Filter Predicate**   | Policy Modifier     | `ADD FILTER PREDICATE`                | Binds the security function's input fields directly to a table's data columns. |
| **Policy State**       | Policy Setting      | `STATE = ON                   \| OFF` | Controls whether security tracking is actively enforced on the table.          |

# Column-level security in a Data Warehouse

Here is the detailed architectural breakdown note on implementing **Column-Level Security (CLS)** using granular T-SQL permissions within a Microsoft Fabric Data Warehouse, structured for your DP-600 exam preparation.

## 1. Column-Level Security (CLS) Architecture

Column-Level Security (CLS) allows data engineers to restrict column visibility based on a user's security profile. This is crucial for protecting personally identifiable information (PII), financial metrics, or sensitive business keys without shifting rows or splitting physical tables.

### The Explicit Share Permission Prerequisite

To implement CLS effectively in Fabric, you must manage how the object is shared at the workspace level first:

- **The Trap:** When sharing a Data Warehouse via the Fabric UI, checking the box for **"Read all data using SQL connection" (SQL Read)** grants an absolute database-level `SELECT` permission. This will completely bypass any granular restrictions you try to configure in T-SQL.
- **The Solution:** Share the warehouse _without_ checking the SQL Read permission box. This leaves the user with simple connectivity access, allowing the internal SQL engine to strictly enforce your specific T-SQL security schemas.

## 2. Granular Permission Masking via T-SQL

Once a user has connectivity access, you control their column visibility using the **`GRANT SELECT`** statement. KQL does not handle these types of schema restrictions natively, so these permissions must be applied directly inside the warehouse's SQL analytics endpoint.

### Scenario A: Full Table Clearance (The Baseline)

To grant a user access to every column inside a target table, apply the standard object-level permission:

```sql
GRANT SELECT ON dbo.TableActual TO [jane@yourcompany.com];
```

### Scenario B: Restricting to Specific Columns

To restrict a user's visibility, append a parenthesized list containing **only the approved column names** directly after the `SELECT` keyword:

```sql
-- Explicitly restricts Susan to only seeing 'Country' and 'Location'
GRANT SELECT ON dbo.TableActual(Country, Location) TO [susan@yourcompany.com];
```

## 3. Query Execution and Error Behaviors

Once CLS is enforced, a user's query syntax dictates whether the database engine allows it to run or blocks it completely.

### The `SELECT *` Failure Mode

If Susan attempts to execute a blanket wildcard query against the restricted table, the query engine will fail completely:

```sql
SELECT * FROM dbo.TableActual;
-- RESULT: Msg 230, Level 14, State 1
-- The SELECT permission was denied on the column 'Actual' of object 'TableActual'
```

- **Engine Processing Rule:** If a query references _at least one_ unauthorized column—even via an implicit wildcard expansion like `SELECT *`—the engine terminates the entire transaction and returns no data at all.

### The Explicit Column Selection Pathway

To query the data successfully, the restricted user must explicitly name only the columns they have permission to see:

```sql
SELECT Country, Location FROM dbo.TableActual;
-- RESULT: Executes perfectly, returning all rows for those two columns only.
```

## 4. Production Design: Combining Views and CLS

While applying `GRANT SELECT` directly onto physical staging tables works well for simple adjustments, it can quickly become difficult to maintain in a complex production environment.

### The Enterprise Best Practice

Instead of exposing underlying raw tables directly, construct a database **View** to serve as an intermediate security abstraction layer. You can then apply Column-Level Security to the view itself.

```
[Physical Table: TableActual] ──> [Database View: v_PublicLocation] ──> GRANT SELECT(Col1, Col2) TO User
  (Houses Raw PII Data)             (Abstracts Base Architecture)          (Applies Granular CLS Layer)
```

### Benefits of the View Architecture

- **Structural Isolation:** If the schema of the underlying physical table changes during a database migration, you only need to update the view definition. This ensures that downstream user queries and permissions remain completely unbroken.
- **Role-Based Access Control (RBAC):** You can grant a team of internal auditors access to the full view, while applying an additional layer of column-level restrictions to that same view for external contractors or general business users.

## CLS Permission Target Reference

| Security Enforcer Pattern         | Target Scope | Error Trigger Profile                                           | Best Production Use Case                                            |
| --------------------------------- | ------------ | --------------------------------------------------------------- | ------------------------------------------------------------------- |
| **`GRANT SELECT ON Table`**       | Table-Level  | Blocks all access if revoked completely.                        | Broad data access clearance for trusted analyst roles.              |
| **`GRANT SELECT ON Table(Cols)`** | Column-Level | Fails completely if an unauthorized column or `*` is requested. | Masking specific high-risk fields (such as salary figures or SSNs). |
| **`GRANT SELECT ON View(Cols)`**  | View-Level   | Fails completely if a restricted view column is queried.        | **Recommended standard** for managing data masking at scale.        |

# Object-level security in a Data Warehouse

Here is the detailed architectural breakdown note on the database security lifecycle, evaluating the priority interactions between **GRANT, DENY, and REVOKE**, and tracking directory identities inside a Fabric Data Warehouse, structured for your DP-600 exam preparation.

## 1. Item-Level vs. Object-Level Access Control

Fabric enforces a strict separation between workspace-level connectivity permissions and granular database object permissions:

- **Item-Level Control (The UI Layer):** Configured via the Fabric workspace interface or share panels. It manages permission bounds for a user or Microsoft Entra ID Security Group against the _entire artifact_ (e.g., granting connectivity to the warehouse container).
- **Object-Level Control (The Engine Layer):** Managed strictly through T-SQL commands inside the SQL analytics endpoint. It provides granular security definitions over individual database objects, including **Tables, Views, Scalar/Table Functions, and Stored Procedures**.

### Primary Object-Level Permission Metrics

- **`SELECT`**: Grants row/column reading visibility.
- **`INSERT` / `UPDATE` / `DELETE`**: Authorizes raw Data Manipulation Language (DML) writes.
- **`EXECUTE`**: Authorizes the physical processing run of compiled stored procedures.

## 2. The Granular T-SQL Permission Lifecycle Matrix

Understanding how the database engine resolves conflicts between access commands is a highly tested concept on technical examinations.

```
                  ┌─── [ DENY ]  ───> Absolute block. Strongest priority. Overrides GRANTS.
                 │
PERMISSION TIER  ├─── [ GRANT ] ───> Explicitly provides object access.
                 │
                  └─── [ REVOKE ] ───> Neutralizer. Wipes away either a GRANT or a DENY.

```

### The Three Core Access Commands

- **`GRANT`**: Explicitly provides access to an object.
- **`DENY`**: Injects an absolute security block. **`DENY` takes absolute priority over a `GRANT`.** If a user is granted table access explicitly but belongs to an Entra security group that is explicitly denied access, the engine blocks the query.
- **`REVOKE`**: Acts as a logical eraser. It does _not_ mean "deny access"—it simply removes an existing `GRANT` or `DENY` string modification from the object's security listing, returning the user to a neutral, unassessed state.

### Devolution of the `WITH GRANT OPTION` Suffix

Appending **`WITH GRANT OPTION`** to an assignment allows the target user to delegate and pass that specific permission flag forward to other corporate identities.

```sql
-- Grants table visibility and authorizes Jane to pass this permission forward
GRANT SELECT ON dbo.TableActual TO [jane@yourcompany.com] WITH GRANT OPTION;
```

## 3. Structural Security System Views: Database Principles

To trace, debug, and audit which internal database actors and external cloud directories have security footprints on your warehouse, query the system catalog view **`sys.database_principals`**:

```sql
SELECT name, type_desc, authentication_type_desc
FROM sys.database_principals;
```

### Identity Definitions Returned

- **Database Roles:** Standard security wrappers like `db_owner` or `public`.
- **Built-in System Identities:** Fixed profiles like `dbo`, `guest`, `INFORMATION_SCHEMA`, and `sys`.
- **EXTERNAL_USER**: Individual cloud identities authenticated natively via **Microsoft Entra ID** (formerly Azure Active Directory).
- **EXTERNAL_GROUPS**: Entra ID Security Groups or security-enabled Microsoft 365 groups synced down to the SQL engine database context. An `EXTERNAL_GROUP` principal will only appear in this catalog view _after_ it has been explicitly targeted by at least one T-SQL permission command.

## 4. The System Administrative Guardrail (Immunity Rule)

To prevent an engineering disaster where an administrator accidentally locks the entire company out of a data stream, the query engine implements an absolute restriction block: **You cannot apply `GRANT`, `DENY`, or `REVOKE` statements against master administrative or system-owned boundaries.**

> **Absolute Enforcement Guardrail:** The database engine will reject any statement attempting to alter permissions for the object creator, the Database Owner (`dbo`), `INFORMATION_SCHEMA`, `sys`, or yourself.

## Permission Precedence Resolution Cheat Sheet

| Existing User Permission | Group Membership State       | Downstream Command Run | Net Resulting User Access State                                                   |
| ------------------------ | ---------------------------- | ---------------------- | --------------------------------------------------------------------------------- |
| **`GRANT`**              | No Group Restrictions        | `REVOKE SELECT`        | **Access Denied** (Returned to unassessed baseline).                              |
| **`GRANT`**              | Explicit **`DENY`** on Group | None                   | **Access Denied** (`DENY` completely overrides `GRANT`).                          |
| None (Baseline)          | Explicit **`DENY`** on Group | `GRANT SELECT` on User | **Access Denied** (User-level `GRANT` cannot crack a group `DENY`).               |
| **`DENY`**               | No Group Restrictions        | `REVOKE SELECT`        | **Access Denied** (Blocks are cleared, but requires an explicit `GRANT` to read). |

# File-level access controls in a Lakehouse

Here is the detailed architectural breakdown note on configuring folder-level security using **OneLake Data Access Roles** inside a Fabric Lakehouse, structured for your DP-600 exam preparation.

## 1. OneLake Data Access Architecture

By default, users with access to a Fabric Lakehouse can read all the data inside it. **OneLake Data Access Roles** allow you to restrict visibility at a granular level, targeting specific folders within the two primary Lakehouse directories:

- **The `Files` Directory:** Contains unstructured or semi-structured raw files uploaded via pipelines or manual uploads.
- **The `Tables` Directory:** Contains the underlying delta-parquet folders that physically store the data for your managed tables.

### The Implicit Deny Security Model

Unlike standard SQL permissions that utilize explicit `DENY` commands, OneLake Data Access Roles operate on an **Implicit Deny** framework.

- There is no explicit "Deny" checkbox.
- The baseline default is that a user **cannot access any data** unless they are explicitly assigned to a role that grants them permission to a specific folder.

## 2. Activation and The Irreversible Guardrail

To begin configuring roles, a user with Workspace **Admin, Member, or Contributor** rights must click the _Manage OneLake Data Access_ button on the Lakehouse home ribbon.

> **Critical Exam Guardrail:** Activating OneLake Data Access Roles is a **permanent, irreversible architectural change**. Once enabled for a specific Lakehouse, this security feature cannot be turned off.

Upon activation, the system automatically generates a **`Default Reader`** role. This baseline role grants access to _all_ folders and is automatically assigned to the user who enabled the feature.

## 3. Creating Roles and Assigning Access

When creating a new custom role, you define its boundary by checking the exact `Files` folders or `Tables` folders the role should expose. Once the role's boundary is saved, you assign identities to it using one of two assignment methodologies:

| Assignment Method       | Target Identity                    | Operational Behavior                                                                                                                |
| ----------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Explicit Assignment** | Specific Users or Security Groups  | Directly typing in a user (e.g., `Susan`) or an Entra ID Security Group.                                                            |
| **Virtual Members**     | Dynamic Lakehouse Permission Tiers | Assigning access based on the user's existing baseline Lakehouse permissions (`Read`, `Write`, `Reshare`, `Execute`, or `ReadAll`). |

## 4. The Virtual Member Logic Trap (Exam Focus)

When assigning users via the **Virtual Members** (Lakehouse permissions) method, understanding how the system evaluates multiple selections is a highly tested concept.

If you check multiple permission boxes in the assignment pane—for example, checking both **`Write`** and **`Reshare`**—the system evaluates this using strict **`AND`** logic, not `OR` logic.

- **The Result:** The role will _only_ apply to users who possess **both** the Write permission AND the Reshare permission concurrently.
- **The Solution:** If your goal is to grant folder access to users who have _either_ the Write permission OR the Reshare permission, you must create **two separate roles** and assign one permission requirement to each.

# Creating a deployment pipeline

Here is the detailed architectural breakdown note on continuous integration, structural alignment testing, and environmental mapping rules for **Fabric Deployment Pipelines**, structured for your DP-600 exam preparation.

## 1. Lifecycle Environment Architecture

Fabric deployment pipelines provide an automated, low-code continuous integration and continuous delivery (CI/CD) framework. They allow you to isolate development testing tasks from operational production spaces.

```
                  ┌───────────────┐      ┌───────────────┐      ┌───────────────┐
PIPELINE STAGES   │  Development  │ ───> │  Test / QA    │ ───> │  Production   │
                  └───────────────┘      └───────────────┘      └───────────────┘
                  (Min: 2 Stages  ────────────────────────────> Max: 10 Stages)

```

### Architectural Sizing Rules

- **Stage Scale Constraints:** A pipeline strictly requires a **minimum of 2 workspaces** and can expand to handle a **maximum of 10 distinct sequential environmental stages**.
- **Stage Name Validation:** To lock in modifications made to stage naming attributes, you **must explicitly click the confirmation checkmark button**. Navigating or clicking away from the input field without doing so will immediately discard your changes.
- **The Backwards-Deployment Exception:** By default, deployment pipelines push changes forward from development to production. However, you can execute a backwards deployment _only if the preceding targeted stage is completely blank and contains no items_. This allows you to retroactively link a pipeline to an existing production asset and copy its structure backward into new testing and development workspaces.

## 2. Supported and Unsupported Item Profile Matrix

Not all Fabric items can pass through a deployment pipeline. Understanding what the engine can and cannot migrate is a high-priority topic for the DP-600 exam.

### Native Pipeline Support

- **Storage and Compute:** Data Pipelines, Lakehouses, Warehouses, SQL Databases, and Mirrored Databases.
- **Analytics and BI:** Reports, Dashboards (Activator), Environments, Eventstreams, Notebooks, and Power BI Dataflows (Gen1).
- **Semantic Models:** Fully supported **only** if they originally originated from a local `.pbix` file template upload (such as a model uploaded from Power BI Desktop).

### Unsupported Architecture (Strictly Excluded)

- **`Dataflows Gen2`**: Cannot be transported or tracked using a deployment pipeline.
- **`Eventhouses` and `KQL Databases`**: Completely blocked from native pipeline migration sweeps.

## 3. Metadata Drift and Structural Variance Auditing

Once workspaces are bound to pipeline stages, the deployment engine uses metadata signatures to continually analyze and flag any differences between them.

### Drifts and Discrepancies

The pipeline engine tracks differences using explicit visual state indicators:

| UI Status Indicator Icon       | Meaning   | Engine Evaluation Rule                                                                                   |
| ------------------------------ | --------- | -------------------------------------------------------------------------------------------------------- |
| **Green Checkmark**            | Identical | The schemas match exactly between the source and target.                                                 |
| **Not-Equal Symbol ($\ne$)**   | Modified  | The item exists in both stages, but internal schemas have changed (e.g., a table structure was altered). |
| **"Only in Source" Indicator** | New Item  | A completely new item has been added to the upstream stage that does not yet exist in the target.        |

### Structural Changes inside Database Engines

If you add an internal database object—such as executing a `CREATE TABLE` command inside a T-SQL query within a specific stage's Warehouse—the parent item container will be flagged as **Modified ($\ne$)**.

### Item Renaming and GUID Lineage Binding

If you rename a parent item in a downstream stage (e.g., renaming `Demo Lakehouse` to `Demo Lakehouse 2` in the Test stage), the deployment pipeline **will not lose track of it**.

The pipeline uses internal, persistent Globally Unique Identifiers (GUIDs) to maintain a permanent structural bond between the objects across stages. The engine will accurately pair them in the interface and show that they are the same item despite having different display names.

## 4. Execution Governance and Error Controls

When launching a deployment, you can check specific boxes to filter which items migrate forward or dictate how the engine handles processing faults:

```kql
-- Functional Flow of Execution Failure Controls
[Item Ingest Migration Phase]
             │
             ├──> Severe Error Encountered
                     │
                     ├── [Checkbox Cleared] ──> Aborts instantly, leaving remaining items uncopied.
                     └── [Checkbox Checked] ──> Skips failed item, continuing migration for the rest.

```

- **Selective Migration:** You do not have to deploy the entire workspace at once; you can select individual checkboxes to move only specific items down the pipeline.
- **Fault Tolerance ("Continue on Failure"):** If an error occurs during deployment, checking the _Continue deployment if deployment fails_ box forces the engine to skip the broken item and finish migrating the rest of the workspace, rather than halting the entire operation.

## Pipeline Configuration Matrix Summary

| Core Architectural Metric           | Pipeline Behavior / Setting Values     | High-Priority Exam Takeaway                               |
| ----------------------------------- | -------------------------------------- | --------------------------------------------------------- |
| **Workspace Capacity Boundaries**   | Minimum: 2 \| Maximum: 10 Stages       | Flexible multi-tier stage staging design support.         |
| **Semantic Model Limit Rule**       | Must originate from `.pbix` files      | Direct cloud-native web models are unsupported.           |
| **Dataflows Gen2 Integration**      | **Completely Unsupported**             | Must be migrated manually or via file exports.            |
| **Downstream Structural Additions** | Preserved (Items are not auto-deleted) | Deploying forward does not wipe out unique target assets. |

# Configuring a deployment pipeline

Here is the detailed architectural breakdown note on **Deployment Pipeline Permissions, Management Constraints, and Parameter Override Rules** inside Microsoft Fabric, structured for your DP-600 exam preparation.

## 1. Pipeline Architecture limits and Dependency Routing

When managing multi-environment item rollouts at an enterprise scale, Fabric enforces strict data connection rules to preserve cross-artifact dependencies.

- **Single Deployment Volumetric Cap:** A pipeline can process a maximum of **300 items concurrently** in a single execution sweep. For larger implementations, you can group or partition components across workspace folders.
- **Orphaned Report Dependency Routing:** If you deploy a Power BI Report forward _without_ including the semantic model it depends on, the execution engine evaluates the target environment:
- If that exact semantic model already exists in the destination workspace, the newly deployed report **automatically establishes a new hot link connection** to it.
- If the underlying semantic model does not exist in the destination stage, the deployment fails.

- **Real-Time Streaming Block:** You **cannot deploy semantic models configured with real-time data connectivity** or push datasets through the pipeline that have streaming direct-access connections. Additionally, local `.pbix` file downloads are blocked for reports after they pass through a pipeline.

## 2. Shared Security and Role Matrix Matrix

Pipeline access controls are separate from workspace security tiers. Granting a user access to a deployment pipeline does not automatically grant them access to the underlying storage boundaries.

```
                   ┌─── Create/Edit/Share/Delete Pipeline Strings
                   ├─── View Deployment History & Audit Item Lists
 PIPELINE ADMIN ───├─── Manage Pipeline Users & Access Matrix
                   └─── Unassign Workspaces from Pipeline Stages
                             │
                             └── + WORKSPACE CONTRIBUTOR TIER (Minimum requirement to:)
                                      │
                                      ├─── Compare Two Environmental Stages Side-by-Side
                                      ├─── Execute the Deploy Action to Push Items Forward
                                      └─── Configure, Edit, and Toggle Deployment Rules

```

### The Pipeline Administrator Privilege Level

A user must possess **Pipeline Admin** permissions within a Pro, Premium Per User (PPU), or Fabric capacity workspace to create, share, edit, or delete a pipeline asset. Pipeline Admins can unassign a workspace from a stage without needing to be a Workspace Administrator.

### The Deployment Execution Trigger

To actively execute a deployment or compare structural drift between two stages, a user must be a Pipeline Admin **AND hold at least a Workspace Contributor role** (or higher) across both the source and target workspaces.

## 3. Environmental Parameter Override Rules

To ensure a development environment safely queries mock staging assets while a production environment automatically connects to a high-capacity database cluster, you must configure **Deployment Rules** on the target stage.

### Rule Category 1: Default Lakehouse Routing

- **Target Profile:** Native Spark Notebooks.
- **Objective:** Allows you to change the target workspace's default Lakehouse destination ID string, redirecting notebook processing routines away from development storage and onto production files.

### Rule Category 2: Data Source Mapping Rules

- **Target Profile:** Dataflows (Gen1), Semantic Models, Datamarts, Paginated Reports, and Mirrored Databases.
- **Supported Connectors:** SQL Server, Azure SQL Database, Azure Synapse Analytics, Azure Analysis Services (AAS), SQL Server Analysis Services (SSAS), OData feeds, Oracle, SharePoint, and Teradata (using Import Mode).

### Rule Category 3: Parameter Override Rules

- **Target Profile:** Dataflows (Gen1), Semantic Models, and Datamarts.
- **Objective:** Intercepts and replaces internal Power Query parameters dynamically during migration to adjust API keys, query thresholds, or server locations.

## 4. Irreversible Rule Erasure Risks

When managing data objects across active pipeline stages, certain modifications will permanently delete your rule parameters:

> **Critical Exam Caution Warnings:**
>
> 1. **The Item Deletion Inversion:** If you physically delete an item from a target workspace, its associated deployment rules are **instantly destroyed** and cannot be restored.
> 2. **The Unassign Workspace Reset:** Unassigning a workspace from a pipeline stage immediately wipes out all custom Data Source and Parameter rules established for that stage. Even if you reassign the exact same workspace back to the same stage later, those configuration metrics must be rebuilt from scratch.

## Deployment Pipeline Access and Configuration Reference

| Feature Matrix Control       | Required Pipeline Role | Required Workspace Role      | Core Technical Boundary                               |
| ---------------------------- | ---------------------- | ---------------------------- | ----------------------------------------------------- |
| **View Pipeline Canvas**     | Anyone in Tenant       | None Required                | General structural visibility across the directory.   |
| **Unassign Workspace**       | Pipeline Admin         | None Required                | Severs the link between a workspace and a stage slot. |
| **Compare Stages & Deploy**  | Pipeline Admin         | **Contributor** (Both sides) | Syncs items forward to the next environmental stage.  |
| **Configure Override Rules** | Pipeline Admin         | **Contributor** (Both sides) | Configures environmental overrides on target stages.  |
| **Assign Initial Workspace** | Pipeline Admin         | **Workspace Admin**          | Establishes the baseline connection link for a stage. |

# Quiz 28: Configure security and governance, and deployment pipelines

![quiz](../images/Pasted%20image%2020260708150701.png)

![quiz](../images/Pasted%20image%2020260708150730.png)

![quiz](../images/Pasted%20image%2020260708150803.png)

![quiz](../images/Pasted%20image%2020260708150902.png)

![quiz](../images/Pasted%20image%2020260708150918.png)
