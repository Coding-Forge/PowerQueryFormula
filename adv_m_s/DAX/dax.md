```
Total Units Sold = SUM('Sales'[Units])
```
```
Category, Campaign = RELATED('ProductDim'[Category]) & ", " & RELATED('CampaignDim'[TrafficChannel])
```
```
Youth Units Sold = CALCULATE([Total Units Sold],FILTER(ProductDim,ProductDim[Segment]="Youth"))
```
```
Accessory Units Sold = CALCULATE([Total Units sold],FILTER(ProductDim,ProductDim[Segment]="Accessory"))
```
```
Rest of Company Units Sold = CALCULATE([Total Units Sold],FILTER(ALL(ProductDim),AND(ProductDim[Segment]<>"Accessory",ProductDim[Segment]<>"Youth")))
```
```
COGS = SUMX(Sales, Sales[Units] * RELATED(ProductDim[Unit Cost]))
```
```
Sales Amount = SUMX(Sales, Sales[Units] * RELATED(ProductDim[Unit Price]))
```
```
Profit = SUMX(Sales, Sales[Units] * (RELATED(ProductDim[Unit Price]) - RELATED(ProductDim[Unit Cost])))

```

## Performance Analysis
> Poor Performing Queries can often be attributed to poorly written queries which in turn have a major impact on usability of the report. It can have further repurcussions once the report is published to the Power BI Service.
```
_YTD Total Units Sold = TOTALYTD(SUM(Sales[Units]), DateDim[Date])
```
```
_PYTD Total Units Sold = CALCULATE(TOTALYTD([Total Units Sold], DateDim[Date]), SAMEPERIODLASTYEAR(DateDim[Date]))
```
```
_Growth % = (TOTALYTD(SUM(Sales[Units]), DateDim[Date]) - CALCULATE(TOTALYTD([Total Units Sold], DateDim[Date]), SAMEPERIODLASTYEAR(DateDim[Date]))) / TOTALYTD(SUM(Sales[Units]), DateDim[Date])
```
> Improved Performance Queries
How to improve performance by using already created measures and using best practices for formulas
```
YTD Total Units Sold = TOTALYTD([Total Units Sold], DateDim[Date])
```
```
PYTD Total Units Sold = CALCULATE([YTD Total Units Sold], SAMEPERIODLASTYEAR(DateDim[Date]))
```
```
Growth % = DIVIDE(([YTD Total Units Sold]-[PYTD Total Units Sold]) , [PYTD Total Units Sold])
```