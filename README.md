# Creating-an-Incremental-Loading-Versus-Flush-and-Fill-on-SQL-Server-For-ETL-Processing
Using Flush and Fill Versus Incremental Loading by Creating some Script Code

//In this short Project,

we will look at the differences between using flush and

fill versus incremental loading to perform ETL processing.

To highlight the differences between using flush and fill and

increment loading, I've created a simple demonstration script code down below:

//

FIRST OF ALL I AM GOING TO DEMONSTRATE HOW TO USE AN INCREMENT LOADING PROCESS: 
The incremental loading process extract only new or changed rows from the database-In SSIS
comparing the target table(destination) against the source data based on Id or Date Stamp or Time Stamp. If there are any New records in Source data, then we have to insert those records in the target table..


1.Use TempDb
==(Setup Code)== Reference Database:AdvendureWorksDW2012
To Start i am  creating a Source and Destination table:

                                                  --Source Table: 
```SQL
If ( Object_id('Employee') is not null )
Drop Table Employee
Go
Create Table Employee(
EmployeeID int Primary Key,
FirstName nvarchar(50),
LastName nvarchar(50),
EmailAddress nvarchar(50) Unique
)
```
"Run code"
                                                    --Destination Table:

```SQL 
 If(Object_id('DimEmployee') is not null)
 Drop Table DimEmployee
 Go
 Create Table DimEmployee
 (EmployeeID int Primary Key,
  FirstName nvarchar(50) ,
  LastName nvarchar(50),
  EmailAddress nvarchar(50)
  )
  ```
                 
"Run code"
--Now i am adding some data to my first table created-this will be my source table.
--To add my dummy data i am going to use the Insert Statement 
(Setup Code)- here i am just filling up my new data with two rows so save time ..

  
  ```SQL
    Insert Into Employee(FirstName,LastName,EmailAddress)
   Values('Foxy','Brown','Foxy@adventure-works.com')
         ('Aka','Benson','Aaka0@adventure-works.com')
  Go
  ```
    
    
    "Run insert Code above"      
--Since i want to find out the similarities and difference i am going to inspect the both data by briging a Store Procedure, so i am basically creating two store procedures;one to capture the difference and another store procedure to cleanse and fill the data stored in these two tables..
(Setup Stored Procedure code )


```SQL
Create Procedure pCompareDiff
 As
                                        --compare the difference with two simple select Stmts
 Select EmployeeID,FirstName,LastName,EmailAddress From Employee
 Select EmployeeID,FirstName,LastName,EmailAddress From DimEmployee
 Go
```
  "Run StoreProcedure Code"
--Also i am creating another store procedure to transfer the data using flush and fill process like this one below:
   I am creating a store procedure that will perform my fulsh and fluid action by using my Dimension tables
Here i am trying to delete the data in my DimEmployee table,and load the fresh data from the sources table employee

                                
                                   
                                   
  ```SQL
   Create Procedure pFlushAndFillDimEmployee
  As
  Delete From DimEmployee
  Insert into DimEmployee(EmployeeID,FirstName,LastName,EmailAddress)
  Select EmployeeID,FirstName,LastName,EmailAddress From Employee
  Go
  ```
  Now let me implement the second method using flush and fill concept
  *******************Using Flush and Fill**********************

Now i am goign to synchronize the tables using my fulsh and fill procedure like this:

  ```SQL
Execute pFlushAndFillDimEmployee
Execute pCompareDiff
Go
  ```
  --Now i am adding a new row to compare the values between the two tables
  
  ```SQL
  Insert Into Employee(FirstName,LastName,EmailAddress)
   Values( 'Rebecca','Fox','Beca430@Demo.com')
    ```
   Exec pCompareDiff
   Go
   
    ```
   --After adding a new row , i will synchronize the destination using Flush and Fill procedure below:
```SQL
Execute pFlushAndFillDimEmployee
Execute pCompareDiff
Go
```

--Let say later if somebody updates the Employee table's data , i will use the rows i want to change inside an Update statement and then reexcecute the procedure:

  ```SQL
 Use TempDb
 Update Employee
 Set FirstName = 'Molly'
    ,Lastname = 'Brown'
    ,EmailAddress = 'Molly@adventure-works.com'
 Where EmployeeID = 1
 Exec pCompareDiff
 Go
   ```
 --Then run the Flush and Fill Excecute command procedure
 
         ```SQL
Execute pFlushAndFillDimEmployee
Execute pCompareDiff
Go

--After i want to use the same code for the delete statment to see if the data has been completed deleted and if it works as i needed:
  ```SQL
 Delete 
 From Employee
 Where EmployeeID = 2
 Exec pCompareDiff
 Go
   ```
  Now i am just going to run one more last time the flush and fill execute command to make sure that EmployeeID =2 is gone from the source table and also in the destination table: notice below by running the execute command flush and fill below;
 ```SQL
Execute pFlushAndFillDimEmployee
Execute pCompareDiff
Go
``` 

By validating all these 3 commands i was able to perform a Insert,Delete and  ,Update incrementally in my DimEmployee table
this is how individual transaction works and how the fundamental of increment loading works as well  in MSSQL...
