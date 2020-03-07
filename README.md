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
                      If(Oject_id('DimEmployee') is not null)
                       Drop Table DimEmployee
                       Go
                       Create Table DimEmployee(
                       EmployeeID int Primary Key,
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
                       Insert Into Employee(FirstName,LastName,LoginID)
                                      Values('Foxy','Brown','Foxy@adventure-works.com')
                                            ('Aka','Benson','Aaka0@adventure-works.com')
                        Go
"Run insert Code"       ```
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
                                   
                                   ``` ```SQL
                                    Create Procedure pFlushAndFillDimEmployee
                                     As
                                     Delete From DimEmployee
                                     Insert into DimEmployee(EmployeeID,FirstName,LastName,EmailAddress)
                                     Select EmployeeID,FirstName,LastName,EmailAddress From Employee
                                     Go
                                      ```
                                     
