# SQLite file dabases

Up until now, we've been using SQLite to work with Excel tables in-memory, but we can also connect to sqlite files. 

This enables us to move data from the workbook into the database in order to minimize the amount of data in Excel. This is useful when dealing with large Excel files, since Excel behaves much better with less data inside the xls/xlsx file.

## Connect to a SQLite file 

To connect to a sqlite file, we must define a custom connection. To do so, click the ***Connect with*** button in the QueryStorm ribbon. This will open up the following dialog:

![Create new connection](https://i.imgur.com/bsxnWn1.png)

You can select an existing .sqlite file, or you can create a new one by specifying a file name that does not yet exist.

Once connected, we can query permanent tables alongside Excel tables.

## Moving data from Excel into the SQLite file

Let's move the `movies` table from Excel into the sqlite file.

```sql
create table movies as -- create permanent table 'movies'
select * from movies -- using data from in-memory table 'movies'
```
This might seem like a name collision, since there's an Excel table called `movies` and a permanent table called `movies`, but the workbook table's full name is `temp.movies` and the name of the new permanent table is `main.movies`. We can refer to the permanent tables explicitly using the `main.` prefix like so:

```sql
select * from main.movies
```
 
More importantly, we can now delete the table from Excel since the data is now in the sqlite file. Rather than keeping the data in the workbook, we use the workbook to present results and graphs to the user and use the .sqlite file for storage. This makes Excel behave much better when dealing with large amounts of data, since it doesn't have to load it all into memory.

## Moving data from the SQLite file into Excel
All SQL engines in QueryStorm can use the preprocessor to redirect the results of queries into Excel tables or ranges.

For example, we can write the contents of the `actors` database table into a new or existing Excel table called `act` like so:

```sql
@Table TableName="act" IfExists="Update" DefaultAddress="B2"
select * from actors
```

You can read more about the preprocessor [here](../misc/preprocessor "Preprocessor docs")