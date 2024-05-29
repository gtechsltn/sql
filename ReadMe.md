# SQL

## MS SQL Tips
https://www.mssqltips.com/

## System Stored Procedures (Transact-SQL)
```
exec sp_spaceused
exec sp_help 'dbo.sites' --Table
exec sp_helptext 'dbo.spGetAllUsers'; --SP
exec sp_MSforeachtable 'SELECT "?", COUNT(*) FROM ?' --All Tables
exec sp_depends 'dbo.Sites' --Table
```

## Show SQL Server Table Attributes

Alt + F1

```
Click to select table and then press Alt + F1
```

```
--Show all table info
exec sp_help 'dbo.Address';
```

```
--Displays the definition that is used to create an object in multiple rows
exec sp_helptext 'dbo.spGetAllUsers';
```
## List of Logins in current SQL Server instance

```
select sp.name as login,
       sp.type_desc as login_type,
       sl.password_hash,
       sp.create_date,
       sp.modify_date,
       case when sp.is_disabled = 1 then 'Disabled'
            else 'Enabled' end as status
from sys.server_principals sp
left join sys.sql_logins sl
          on sp.principal_id = sl.principal_id
where sp.type not in ('G', 'R')
order by sp.name;
```

## Retrieve all Logins in SQL Server
```
SELECT name as SQLServerLogin, SID as SQLServerSID FROM master.sys.syslogins
SELECT * FROM master.sys.server_principals
SELECT * FROM master.sys.sql_logins;
SELECT * FROM sysusers
EXEC sp_helpuser
```

## TRY CATCH RAISERROR

https://www.mssqltips.com/sqlservertip/7997/sql-server-try-catch-raiserror-throw-error-handling/

```
BEGIN TRY
    RAISERROR ('An error occurred in the TRY block.', 16, 1);
END TRY
BEGIN CATCH
    DECLARE @ErrorMessage NVARCHAR(2048),
            @ErrorSeverity INT,
            @ErrorState INT;
 
    SELECT @ErrorMessage = ERROR_MESSAGE(),
           @ErrorSeverity = ERROR_SEVERITY(),
           @ErrorState = ERROR_STATE();
 
    RAISERROR(@ErrorMessage, @ErrorSeverity, @ErrorState);
END CATCH;
```

## SQL Convert Examples for Dates, Integers, Strings and more

https://www.mssqltips.com/sqlservertip/8004/sql-convert-examples-dates-integers-strings/

```
DECLARE @DateAndTime DATETIME = '2024-01-01 08:00:00.000'
SELECT CONVERT(VARCHAR(30), @DateAndTime) AS [String];
GO

DECLARE @MyString VARCHAR(10) = '123'
SELECT CONVERT(INT, @MyString) AS [Integer];
GO

DECLARE @Num INT = 5
SELECT CONVERT(DECIMAL(3,2), @Num) AS [Decimal];
GO

```

# Git

```
echo "# asp-net-core" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M master
git remote add origin https://github.com/gtechsltn/sql.git
git push -u origin master
```

## Mastering SQL Server: Advanced Features Every DBA Should Know

https://medium.com/@nile.bits/mastering-sql-server-advanced-features-every-dba-should-know-bb0c28031c6f

https://www.nilebits.com/blog/2024/04/sql-server-advanced-features-dba/

## .NET 8 Web API project with an example of Offset and Keyset Pagination with EF Core

https://henriquesd.medium.com/pagination-in-a-net-web-api-with-ef-core-2e6cb032afb7

https://github.com/henriquesd/PaginationDemo/
