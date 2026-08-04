## Part 1 - Data Exploration & Modeling

Get data -> folder -> Transform -> Extension (.csv) -> Content: Binary click: Open content without generating configure files.

![[Pasted image 20260730224648.png]]

VS. content - expand icon: Transform File from source (Helper Queries) folder generated - if you want to rollback this step; then ungroup folder and delete unnecessary files. 

![[Pasted image 20260730224814.png]]

file structure
1. load folder -> `Source Table/source`
2. open binary; single data file -> `Source Table/people_data_source`
3. people_data_source reference table; it will use for report, dash -> `Production Table/people_fact`
4. Nomalize data to fact & dim table -> `Production Table/department_dim`

fact table
- if it is granualar as record; the data describe single record then leave it in fact table. otherwise, it is repeated across table; then seperated as dim table.

Numeric categorical column (ex. active_status)
- for easier understanding, 0, 1 categorical value might lead confusing
- 1. numeric -> text 2. replace 1 with Yes, 0 with No

Fact & Dim table
- people_employment_source -> people_fact
- people_employment_source -> department_dim
- people_fact -> department_dim (X) -> **it cause circular reference error**
- 1. dim table: keep department, sub-department, add index, rename to department_key
- 2. fact table: merging queries using  department, sub-department; keep department_key; delete department, sub-department
- 3. relationship: left outer join using department_key (many to one)

![[Pasted image 20260730230517.png]]

![[Pasted image 20260730230546.png]]

![[Pasted image 20260730230650.png]]

Fact & Dim Table Structure
- Source Table
	- source: folder info
	- people_data_source
	- people_employment_source
- Production Table
	- people_employment_source -> people_fact
		- cols: employment_id, first_name, last_name, employment_status, salary, hire_date, term_date, active_status + keys (FK)
	- dim from people_employment_source
		- department_dim
		- job_level_dim
		- term_dim
		- manager_dim
			- merge with people_fact for name
	- dim from people_fact + people_data_source
		- demographic_dim
		- educiation_dim
		- location_dim
		- marital_status_dim

![[Pasted image 20260731170005.png]]

![[Pasted image 20260731165857.png]]

## Part 2 - Measure Tables

Cards - values - 4K -> Visual / Callout / Value / Display Units = None

![[Pasted image 20260731232029.png]]

## Part 3 - Headcount Calculation


![[Pasted image 20260731235424.png]]

```EXCEL
All Employees = DISTINCTCOUNT(people_fact[employee_id]) -- 4,138


Headcount = CALCULATE(
    [All Employees], -- expression
    FILTER(
        people_fact, -- table expression
        people_fact[hire_date] <= LASTDATE(Dates[Date]) && 
        -- hired before last day
        (
            people_fact[term_date] > LASTDATE(Dates[Date]) || 
            -- if termniated then, not terminiated before last day
            people_fact[term_date] = BLANK() 
            -- not terminiated yet
        )
    )
)
```

##  Part 3 - Employee Retention Calculation

```EXCEL
-- Employee Retention = The Number of the Same Employees We Ended With / The Number of Employees We **Started With**

Starting Headcount = CALCULATE(
  [All Employees], 
  FILTER(
    people_fact, 
    people_fact[hire_date] < FIRSTDATE(Dates[Date]) 
    && (
      people_fact[term_date] = BLANK() 
      || people_fact[term_date] >= FIRSTDATE(Dates[Date]) 
      -- include ppl leave company after start of period
    )
  )
)

Ending Headcount = CALCULATE(
  [All Employees], 
  FILTER(
    people_fact, 
    people_fact[hire_date] < FIRSTDATE(Dates[Date]) 
    && (
      people_fact[term_date] = BLANK() 
      || people_fact[term_date] >= LASTDATE(Dates[Date]) 
  	  -- include ppl leave company after end of period 
    )
  )
)

Retention = DIVIDE([Ending Headcount], [Starting Headcount], 0)
```


![[Pasted image 20260801002652.png]]

`Employee Turnover = Count of Employees Who Left the Company / (
  Starting # of Employees + Ending # of Employees
) / 2`

```EXCEL
Avg # of Employees = ([Starting Headcount] + [Headcount])/2

Departing Employees = CALCULATE([All Employees], FILTER(
  people_fact,
  people_fact[term_date] >= FIRSTDATE(Dates[Date]) &&
  people_fact[term_date] <= LASTDATE(Dates[Date])
))

Turnover % = DIVIDE(
  [Departing Employees],
  [Avg # of Employees]
)
// 3% - 5%
```

##  Part 4 - Visualizing Headcount

