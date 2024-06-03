# SQL Server

https://docs.google.com/document/d/1TI1M1uBEM-wlhWN_9Q3LxJ_9RBeQgOTEALhFx48Zg6c/

+ master.sys.server_principals
+ master.sys.syslogins
+ master.sys.sql_logins;
+ master.dbo.sysdatabases
+ sys.all_objects
+ sys.system_objects
+ sys.schemas
+ sys.tables
+ sys.columns
+ sys.objects
+ sys.procedures
+ sys.triggers
+ sys.indexes
+ sys.views
+ sys.partitions
+ sys.key_constraints
+ sys.default_constraints
+ sys.check_constraints
+ sp_help
+ sp_helptext
+ sp_helprole
+ sp_helpdb

```
SELECT name, suser_sname(sid), convert(nvarchar(11), crdate),dbid, cmptlevel
FROM master.dbo.sysdatabases
```
+ sp_password
```
exec sp_password @old = 'Abc@123$', @new = 'Abcde@12345-', @loginame = 'sa'
```

## Changing a password
```
ALTER LOGIN [sa] WITH PASSWORD = 'Abcde@12345-' OLD_PASSWORD = 'Abc@123$'
GO
```

# All objects
```
SELECT  o.type_desc AS Object_Type
       ,  s.name AS Schema_Name
       ,  o.name AS Object_Name
    FROM  sys.objects o 
    JOIN  sys.schemas s
      ON  s.schema_id = o.schema_id
   WHERE  o.type NOT IN ('S'  --SYSTEM_TABLE
                        ,'PK' --PRIMARY_KEY_CONSTRAINT
                        ,'D'  --DEFAULT_CONSTRAINT
                        ,'C'  --CHECK_CONSTRAINT
                        ,'F'  --FOREIGN_KEY_CONSTRAINT
                        ,'IT' --INTERNAL_TABLE
                        ,'SQ' --SERVICE_QUEUE
                        ,'TR' --SQL_TRIGGER
                        ,'UQ' --UNIQUE_CONSTRAINT
                        )
ORDER BY  Object_Type
       ,  SCHEMA_NAME
       ,  Object_Name
```

# All columns in all tables
```
SELECT	syso.name [Table],
		sysc.name [Field], 
		sysc.colorder [FieldOrder], 
		syst.name [DataType], 
		sysc.[length] [Length], 
		sysc.prec [Precision], 
		CASE WHEN sysc.scale IS null THEN '-' ELSE sysc.scale END [Scale], 
		CASE WHEN sysc.isnullable = 1 THEN 'True' ELSE 'False' END [AllowNulls], 
		CASE WHEN sysc.[status] = 128 THEN 'True' ELSE 'False' END [Identity], 
		CASE WHEN sysc.colstat = 1 THEN 'True' ELSE 'False' END [PrimaryKey],
		CASE WHEN fkc.parent_object_id is NULL THEN 'False' ELSE 'True' END [ForeignKey?], 
		CASE WHEN fkc.parent_object_id is null THEN '-' ELSE obj.name END [RelatedTable],
		CASE WHEN ep.value is NULL THEN '-' ELSE CAST(ep.value as NVARCHAR(500)) END [Description]
 FROM [sys].[sysobjects] AS syso
 JOIN [sys].[syscolumns] AS sysc on syso.id = sysc.id
 LEFT JOIN [sys].[systypes] AS syst ON sysc.xtype = syst.xtype and syst.name != 'sysname'
 LEFT JOIN [sys].[foreign_key_columns] AS fkc on syso.id = fkc.parent_object_id and sysc.colid = fkc.parent_column_id  
 LEFT JOIN [sys].[objects] AS obj ON fkc.referenced_object_id = obj.[object_id]
 LEFT JOIN [sys].[extended_properties] AS ep ON syso.id = ep.major_id and sysc.colid = ep.minor_id and ep.name = 'MS_Description'
 WHERE syso.type = 'U' AND syso.name != 'sysdiagrams' 
 AND sysc.name = 'DocFiled'
ORDER BY 1,3,2,4,5,6
```

# SQL Server find tables with or without a certain property

https://www.mssqltips.com/sqlservertip/3402/over-40-queries-to-find-sql-server-tables-with-or-without-a-certain-property/

## SQL Server Tables without a Primary Key
```
SELECT [table] = s.name + N'.' + t.name 
  FROM sys.tables AS t
  INNER JOIN sys.schemas AS s
  ON t.[schema_id] = s.[schema_id]
  WHERE NOT EXISTS
  (
    SELECT 1 FROM sys.key_constraints AS k
      WHERE k.[type] = N'PK'
      AND k.parent_object_id = t.[object_id]
  );
```

## SQL Server Tables without a Unique Constraint
```
SELECT [table] = s.name + N'.' + t.name 
  FROM tables AS t
  INNER JOIN schemas AS s
  ON t.[schema_id] = s.[schema_id]
  WHERE NOT EXISTS
  (
    SELECT 1 FROM key_constraints AS k
      WHERE k.[type] = N'UQ'
      AND k.parent_object_id = t.[object_id]
  );
```

## SQL Server Tables without a Clustered Index (Heap)
```
SELECT [table] = s.name + N'.' + t.name 
  FROM tables AS t
  INNER JOIN schemas AS s
  ON t.[schema_id] = s.[schema_id]
  WHERE NOT EXISTS
  (
    SELECT 1 FROM indexes AS i
      WHERE i.[object_id] = t.[object_id]
      AND i.index_id = 1
  );
```

## SQL Server Tables with a Default or Check Constraint
```
SELECT [table] = s.name + N'.' + t.name
  FROM sys.tables AS t
  INNER JOIN sys.schemas AS s
  ON t.[schema_id] = s.[schema_id]
  WHERE EXISTS
  (
    SELECT 1 FROM sys.default_constraints AS d
      WHERE d.parent_object_id = t.[object_id]
    UNION ALL
    SELECT 1 FROM sys.check_constraints AS c
      WHERE c.parent_object_id = t.[object_id]
  );
```

## SQL Server Tables with More (or Less) Than X Rows
```
DECLARE @threshold INT;
SET @threshold = 100000;

SELECT [table] = s.name + N'.' + t.name
  FROM sys.tables AS t
  INNER JOIN sys.schemas AS s
  ON t.[schema_id] = s.[schema_id]
  WHERE EXISTS
  (
    SELECT 1 FROM sys.partitions AS p
      WHERE p.[object_id] = t.[object_id]
        AND p.index_id IN (0,1)
      GROUP BY p.[object_id]
      HAVING SUM(p.[rows]) > @threshold
  );
```

## DATETIME columns: Checking for rows where system_type_id = 61
```
SELECT DISTINCT
 QUOTENAME(OBJECT_SCHEMA_NAME([object_id])) 
     + '.' + QUOTENAME(OBJECT_NAME([object_id]))
 FROM sys.columns
 WHERE [system_type_id] = 61;
```
  
## SQL Server Index

https://www.mssqltips.com/sqlservertip/1337/building-sql-server-indexes-in-ascending-vs-descending-order/

```
SELECT TOP 10 OrderDate, SubTotal FROM Purchasing.PurchaseOrderHeader ORDER BY OrderDate asc, SubTotal desc
GO

DROP INDEX [IX_PurchaseOrderHeader_OrderDate] ON [Purchasing].[PurchaseOrderHeader] 
GO

CREATE NONCLUSTERED INDEX [IX_PurchaseOrderHeader_OrderDate]
ON [Purchasing].[PurchaseOrderHeader] ( [OrderDate] ASC, [SubTotal] DESC )
GO
```


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

## Table Row Count
```
DECLARE @tblRowCount AS TABLE (Counts INT,
                              TableName VARCHAR(255))

INSERT INTO @tblRowCount (Counts,TableName)
EXEC sp_MSforeachtable 'SELECT COUNT(1) As counts, "?" as tableName FROM ?'

SELECT * FROM @TblRowCount ORDER BY Counts desc
```

## Show SQL Server Table Attributes

```
--Click to select table and then press Alt + F1

--Show all table info
exec sp_help 'dbo.Address';

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

## SQL Server TRY CATCH, RAISERROR and THROW for Error Handling

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
DECLARE @Date DATE = '2024-01-01' -- date value
SELECT CONVERT(VARCHAR(10), @Date, 101) AS [MM/DD/YYYY];
GO

DECLARE @Date DATE = '2024-01-01' -- current date example
SELECT CONVERT(VARCHAR(10), @Date, 1) AS [MM/DD/YY];
GO

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
