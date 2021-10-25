# Power Query (M Language)

## Formulas
### L2 Create Customer Dimension
We want to extract text from the Email Address and make three new columns, First Name, Last Name and Full Name. To achieve this we are going to make a helper column that will provide the position of a character that can be used as a "delimiter/separator" of the data

>First we are going to create the Separator column that will tell us the position of the comma. For the name of this new column enter `Separator` as the value
```
=Text.PositionOf([Email Name.2],",")
```
>This function works by providing the column you want to search in followed by the character that you want to find. It returns the numerical position of the character found for each record.


>Next, we want to extract out of the text the Last Name. To do this we will be using a substring type function. Name the following column `Last Name`
```
=Text.Start([Email Name.2],[Separator])
```
>What this function did was to extract all the text from the start of the column and carry over up to the position of the `Separator`


>Now that we have the `Last Name` we need to extract out the First Name from the email address. To do this we will use another substring type function. Name the following column `First Name`
```
=Text.Range([Email Name.2],[Separator]+2)
```

>Now we want to create the `Full Name` column by combining the columns `First Name` and `Last Name`. When working with string values and concatenating I like to remove any whitespace before or after the string value. You can use the `Trim.Text()` function to remove all whitespace. Next, combine the two columns together using and ampersand with double quotes containing a space and another ampersand.

```
=Trim.Text([First Name]) & " " & Trim.Text([Last Name])
```

### Budget Fact Table

>Creating a date by combining values from multiple columns using the `Date.From()` function.
```
=Date.From([Column3] & [Column2])
```

>Creating a custom column using conditional statements
```
if Text.Length([Column3])>3 then [Column3] else [Column1] & "~" & Date.ToText([Budget Month], "MM/dd/yy")
```

### Dynamic Tables

> Get the data from the Excel Workbook
```
let
    FilePath = Path,  //External reference to text query = FilePath
    FileName = Actuals_File,  /*  Wrapping */
   
    PathSlash = if Text.StartsWith(FilePath,"http") then "/" else "\",
    FullPath = FilePath & (if Text.EndsWith(FilePath, PathSlash) then "" else PathSlash) & FileName,

    Source = if Text.StartsWith(FilePath,"http")
           then Excel.Workbook(Web.Contents(FullPath), null, true)
           else Excel.Workbook(File.Contents(FullPath), null, true)
in
    Source
```

> Get the data from the txt/csv file
```
let
    FilePath = Path,  //External reference to text query = FilePath
    BudgetFilename= Budget_File,  /*  Wrapping comment line */
   
    PathSlash = if Text.StartsWith(FilePath,"http") then "/" else "\",
    FullPath = FilePath & (if Text.EndsWith(FilePath, PathSlash) then "" else PathSlash) & BudgetFilename,


    Source = if Text.StartsWith(FilePath,"http")
             then Csv.Document(Web.Contents(FullPath ),[Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.None])
             else Csv.Document(File.Contents(FullPath),[Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.None])							
in
    Source
```

> Create a custom function that can be called in a table
```
let
    Source = (TransactionDate as date) => let
        YearStart = #date(Date.Year(TransactionDate),1,1),
        #"DateDiff" = Duration.From(TransactionDate-YearStart),
        #"NumberDays" = Duration.Days(#"DateDiff")+1
    in
        #"NumberDays"
in
    Source
```