Coding for Year to Date sales as compared to the same time in the previous year
```
PYTD = DIVIDE(SUM('Sales'[Price])-[PY Sales],SAMEPERIODASLAST(Date[Date]))
```