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

> Get the data from the Excel Workbook. Looking at the function you can break things down into the respective variables and the call to get the data.
<ol>
    <li>FilePath is set to the variable Path. The value being stored in Path is the root folder for the Excel Workbook</li>
    <li>FileName is being set to the value being stored in Actuals_File, which is the Excel Workbook</li>
    <li>PathSlash is a function that determines the origin of the file. If the file starts with Http then it will add the corresponding character according to the protocol required for navigating a file path on the web or locally</li>
    <li>FullPath takes the value from FilePath, concatenates with the appropriate character for the protocol being used and appends on the FileName at the end</li>
    <li>Last, is providing the "Source" of the data for Power BI to consume. If the FilePath has "http" then it will use Web.Contents to read the file from a URL else it will use File.Contents to read a file from a local or network drive</li>
</ol>

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

> Get the data from the txt/csv file. We can take the same process from the above cell block and apply it to our txt/csv files. Now we can dynamically put into a table where all our data is coming from, and we can easily add more resources or take them away simply by adding or removing a row from the table.

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

> Create a custom function that can be called in a table. This custom function is going to be used to figure out how many days have passed from the beginning of the year as provided by a date column from the table. A custom column will be created using the `invoke function` when adding an addition column. Looking at the function we can see that a "DATE" is being passed in as parameter. We are then making three variables `YearStart, DateDiff, and NumberDays`. The variable `YearStart` is extracting the year from the passed in "Date" value and recombinging it to make a new date that is the first day of the first month of the corresponding year. `DateDiff` is looking at the amount of time elapsed from the the `TransactionDate` and the `YearStart`. `NumberDays` is converting the time to actuals days and adding 1. The function is then returning the value of `NumberDays` as the source which can then be used as the response to any record that is passing in a "Date" value.

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