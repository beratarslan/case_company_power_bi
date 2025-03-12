### Creating an Empty Table
```DAX
measures_table = GENERATESERIES(-1, 1, 1)
```

### Calendar Table to Store Dates
```DAX
date_table = ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q YYYY"),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMM")
)
```

## Aggregation Functions

### Total Revenue
```DAX
total_revenue = SUM(sales[net_sale])
```

### Total Profit
```DAX
total_profit = SUMX(
    sales,
    sales[net_sale] - sales[qty] * RELATED(products[cogs_usd])
)
```

### Total Orders
```DAX
total_orders = DISTINCTCOUNT(sales[order_id])
```

### Total Returns
```DAX
total_returns = DISTINCTCOUNT(returns[order_id])
```

### Return Rate %
```DAX
return_rate = 
VAR TotalReturns = DISTINCTCOUNT(returns[order_id])
RETURN DIVIDE(TotalReturns, [total_orders], 0) * 100
```

### Total Customers
```DAX
total_customers = DISTINCTCOUNT(customers[customer_id])
```

## Time Intelligence Functions

### Last Year Revenue (SAMEPERIODLASTYEAR)
```DAX
last_year_revenue = CALCULATE([total_revenue], SAMEPERIODLASTYEAR(sales[sale_date]))
```

### Target Revenue
```DAX
target_revenue = 
VAR LastYearSale = CALCULATE([total_revenue], SAMEPERIODLASTYEAR(sales[sale_date]))
RETURN IF(ISBLANK(LastYearSale), 500000, LastYearSale * 1.5)
```

### Total Revenue YTD
```DAX
total_revenue_YTD = CALCULATE([total_revenue], DATESYTD(sales[sale_date]))
```

### Last Refresh Date
```DAX
last_refresh = MAX(date_table[Date])
```

## KPI Calculations

### KPI Orders
```DAX
kpi_orders = 
VAR CurYTD = CALCULATE([total_orders], DATESYTD(date_table[Date]))
VAR LastYTD = CALCULATE([total_orders], SAMEPERIODLASTYEAR(date_table[Date]))
VAR Diff = CurYTD - LastYTD
VAR AbsDiff = ABS(Diff) / 1000
VAR ResultText = IF(ISBLANK(LastYTD), "No sale in last year", IF(Diff > 0, FORMAT(AbsDiff, "0.0") & "K More than last year (▲)", FORMAT(AbsDiff, "0.0") & "K Less than last year (▼)"))
RETURN ResultText
```

### KPI Profit
```DAX
kpi_profit = 
VAR CurYTD = CALCULATE([total_profit], DATESYTD(date_table[Date]))
VAR LastYTD = CALCULATE([total_profit], SAMEPERIODLASTYEAR(date_table[Date]))
VAR Diff = CurYTD - LastYTD
VAR DiffRatio = DIVIDE(Diff, LastYTD, 0)
VAR AbsDiff = ABS(DiffRatio)
VAR ResultText = IF(ISBLANK(LastYTD), "No sale in last year", IF(DiffRatio > 0, FORMAT(AbsDiff, "0.0%") & " More than last year (▲)", FORMAT(AbsDiff, "0.0%") & " Less than last year (▼)"))
RETURN ResultText
```

### KPI Return Rate
```DAX
kpi_return_rate = 
VAR CurYTD = CALCULATE(DIVIDE([total_returns], [total_orders], 0), DATESYTD(date_table[Date]))
VAR LastYTD = CALCULATE(DIVIDE([total_returns], [total_orders], 0), SAMEPERIODLASTYEAR(date_table[Date]))
VAR Diff = CurYTD - LastYTD
VAR AbsDiff = ABS(Diff)
VAR ResultText = IF(ISBLANK(LastYTD), "No sale in last year", IF(Diff > 0, FORMAT(AbsDiff, "0.0%") & " More than last year (▲)", FORMAT(AbsDiff, "0.0%") & " Less than last year (▼)"))
RETURN ResultText
```

## Best Performing Products

### Best Selling Product
```DAX
topN_product = CALCULATE(FIRSTNONBLANK(products[product_name], 1), TOPN(1, VALUES(products[product_name]), [total_orders], DESC))
```

### Best Return Product
```DAX
topN_returned_product = CALCULATE(FIRSTNONBLANK(products[product_name], 1), TOPN(1, VALUES(products[product_name]), [total_returns], DESC))
```
