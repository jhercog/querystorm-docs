# User defined scalar functions

QueryStorm lets you write functions in C# or VBA, and call them from SQL. This can be especially useful if some team members know C# but others only know SQL. You can use this to perform custom calculations, cryptography, business rules, looking up data on the web, etc...

!!! Tip
	**Writing functions in C# is recommended** (rather than in VBA) since it offers much better performance, as well as a better user experience when calling the function from SQL (auto-complete and tooltips).

Suppose we have a table containing a list of people. The table has a `CreditCard` column and we want to check if the values it contains represent valid credit card numbers. Credit card number validation is usually done via the [Luhn algorithm](https://en.wikipedia.org/wiki/Luhn_algorithm). Not surprisingly, there is no such function in SQLite, and QueryStorm does not add one either, but we can easily add one to the workbook ourselves.
	
## Custom function via C# #

Let's connect via C#, and paste the following implementation of the Luhn algorithm into the code editor:

```csharp
///<summary>Returns true if input value is a valid credit card number, otherwise returns false.</summary>
///<param name="digits">The input value to check</params>
[SQLFunc]
public static bool Luhn(string digits)
{
    return digits.All(char.IsDigit) && digits.Reverse()
        .Select(c => c - 48)
        .Select((thisNum, i) => i % 2 == 0
            ? thisNum
            :((thisNum *= 2) > 9 ? thisNum - 9 : thisNum)
        ).Sum() % 10 == 0;
}
```  
In order for this function to be callable from SQLite, we need to decorate it with the `[SQLFunc]` attribute, and embed it into the workbook. 

![defining c# Luhn function](../Images/cs_luhn_def.png "defining c# Luhn function")

We can now disconnect from C# and connect via SQLite to test the function:
``` sql
select Luhn('1234123412341234') --returns 0 (0=invalid, 1=valid)
```
![calling c# Luhn function](../Images/cs_luhn_call.png "calling c# Luhn function")

The function shows up in autocomplete and we also get function insights based on the XML comments on the function.

We can use the function to verify a list of credit card numbers like so:

```sql
select 
	name, creditcardnumber, Luhn(CreditCard) as looksValid
from 
	people
``` 

### Modifying functions

The SQLite engine compiles any C# functions that it finds in the workbook while it's connecting. If you modify a C# function while connected via SQLite, you'll have to **reconnect** in order for the change to be visible to SQLite.

### XML comments
The XML comments on the function are not mandatory, but are useful as they will appear in autocomplete when using the function.

### Function scope
The function is only visible in the context of the workbook it is embedded in. Currently it isn't possible to define a function at the user/machine level, but this is likely to be allowed in the future. A central repository for sharing functions is also planned for a future release.

## Custom function via VBA

Using C# is recommended for implementing custom functions, but if you're not comfortable with C# or you already have VBA functions you'd like to use, here's how to call them from SQL...

Let's assume we have the Luhn function in VBA. In order for the function to be able to return a value, it needs to be **defined in a module** rather than at the workbook/sheet level. Otherwise, it will always return null (this is a technical limitation).

```VB
Public Function Luhn(ByVal CardNumber As String) As Boolean
	'...
	'actual implementation can be found here: 
	'http://www.freevbcode.com/ShowCode.asp?ID=3132
	'...       
End Function
```

Once the function in place, we can call it from SQL using the [vba function](./sqlite_functions/#vba) like so:

```sql
select 
	name, creditcardnumber, vba('Luhn', creditcardnumber) as cardValid 
from 
	people
```

!!! Tip
	Any breakpoints you set within the function will be respected. When hit, they will pause the execution of the SQL command. You can use them to inspect function parameters or debug the function itself. 