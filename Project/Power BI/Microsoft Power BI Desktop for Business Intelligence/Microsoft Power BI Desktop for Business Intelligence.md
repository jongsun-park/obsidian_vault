## 1. Getting Started
## 2. Introducing Microsoft Power Bl Desktop

### Adjusting Settings

1. Global - Select all available preview features

![[Pasted image 20260809190249.png]]

2. Current - Data load - for educational purpose
	1. Relationships: uncheck update or delete relationship when refresh data, autodetect new relationships after data is loaded
	2. Time intelligence: uncheck auto date / time

![[Pasted image 20260809190404.png]]

3. Current - Regional Settings - English (US)

![[Pasted image 20260809190633.png]]


### Power BI Desktop Interface & Workflow

Back-End
- Power Query Editor: Date is loaded & transformed 

Front-End
- Model View: Data models are configured
- Data View: Table features & calculations are added
- Report View: Visuals & reports are designed
### Helpful Resources

[Power BI Updates Blog](https://community.fabric.microsoft.com/t5/Power-BI-Updates-Blog/bg-p/fbc_pbiupdatesblog)

[Guy in a Cube](https://www.youtube.com/guyinacube)

[Power Platform User Group](https://dynamicscommunities.com/category/ug/power-platform-ug/)

## 3. Connecting & Shaping Data

Storage & Connection Modes
- Import
- DirectQuery
- Composite Model (=Combined Model)
- Live Connection (Single source data model)

[Semantic model modes in the Power BI service](https://learn.microsoft.com/en-us/power-bi/connect-data/service-dataset-modes-understand)

Data Profiling Tools
- Column quality: % of valid, errors, empty
	- use case: keep errors -> check error message -> remove steps -> remove errors & empty
- Column distribution: # of distinct, unique
	- unique: only appear once
- Column profile: a more holistic; details view (ex. count, distinct count, unique, min & max)

Date & Time Tools
- Age
- Date Only
- Year/Month/Quarter/Week/Day
- Earliest/Latest (in "Transform" menu)

Date Table
- Single column (Date) -> Add column (ex. Day Name, Start of Week, Start of Month, Month Name)

Change Type with Locale

![[Pasted image 20260809220743.png]]

Rolling Calendars

![[Pasted image 20260809221159.png]]

![[Pasted image 20260809221219.png]]

```M Code
= List.Dates(
    Source,
    Number.From(DateTime.LocalNow()) - Number.From(Source),
    #duration(1, 0, 0, 0)
)
```


Power Query M Code - Create a calendar for the last three years

```EXCEL
-- Source: Create single date (now)
= DateTime.Date(DateTime.LocalNow())

-- Create Date List from Today - 3 Y to Today
= List.Dates(
    Date.AddYears(Source, -3),
    Duration.Days(Source - Date.AddYears(Source, -3)) + 1,
    #duration(1, 0, 0, 0)
)

-- Convert to Date
-- Rename Col & Add Cols
```
![[Pasted image 20260809222632.png]]

### Data Source Parameters

Parameters
- New Parameters
- Name, Type, Value (default: any)
- Default & current value

Data source settings
- Text -> Parameter 

Power Query Editor
- View: Parameters -> Always allow

