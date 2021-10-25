# Power Query (M Language)
## L2 Create Customer Dimension

### Formulas
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
