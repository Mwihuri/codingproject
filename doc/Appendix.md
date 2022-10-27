
Appendix
#################
Glossary
****************
Recommended Content
**********************

Change Tracker
********************

Approvals
**********************


Example
-----------------

USE master 

GO 
 
1.	**Database**

CREATE DATABASE [MemoryOptimzed] 

 ON  PRIMARY 

(NAME = N'MemoryOptimzed', FILENAME = N'C:\MSSQL\Data\MemoryOptimzed.mdf') 

 LOG ON 

(NAME = N'MemoryOptimzed_log', FILENAME = N'C:\MSSQL\Log\MemoryOptimzed_log.ldf') 

GO 
  
-- Add memory optimized filegroup and a file 

ALTER DATABASE [MemoryOptimzed] ADD FILEGROUP [Optimized_FG] CONTAINS MEMORY_OPTIMIZED_DATA 

GO 

ALTER DATABASE MemoryOptimzed ADD FILE (NAME = N'Optimized_Data', FILENAME = N'C:\MSSQL\Data\Optimized_Data.ndf') TO FILEGROUP [Optimized_FG] 

GO 
  
-- Turn memory optimization feature on at the database level 

ALTER DATABASE MemoryOptimzed SET MEMORY_OPTIMIZED_ELEVATE_TO_SNAPSHOT = ON 

GO 
  
-- Create a Customer temporal table without Memory optimization and insert 3 rows 

2.	**Table**

USE MemoryOptimzed 

GO 
  
CREATE TABLE Customer
( 
CustomerId INT IDENTITY(1, 1) PRIMARY KEY 

,FirstName VARCHAR(30) NOT NULL

,LastName VARCHAR(30) NOT NULL 

,Amount_purchased DECIMAL NOT NULL 

,StartDate datetime2 generated always as row START NOT NULL 

,EndDate datetime2 generated always as row END NOT NULL

,PERIOD FOR SYSTEM_TIME (StartDate, EndDate) 

) 

WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.CustomerHistory)) 

GO 
  
INSERT INTO dbo.Customer(FirstName, LastName, Amount_Purchased)

VALUES ('Frank', 'Sinatra', 20000.00),

('Shawn', 'McGuire', 30000.00),

('Amy', 'Carlson', 40000.00) 

GO 
  
-- Now create a memory optimized temporal table CustomerOptimzed and insert the same 3 rows 

CREATE TABLE CustomerOptimzed

( 

CustomerId INT IDENTITY(1, 1)  

,FirstName VARCHAR(30) NOT NULL 

,LastName VARCHAR(30) NOT NULL 

,Amount_purchased DECIMAL NOT NULL 

,StartDate datetime2 generated always as row START NOT NULL 

,EndDate datetime2 generated always as row END NOT NULL 

,PERIOD FOR SYSTEM_TIME (StartDate, EndDate), 

CONSTRAINT [PK_CustomerID] PRIMARY KEY NONCLUSTERED HASH (CustomerId) WITH (BUCKET_COUNT = 131072) 
) 

WITH(MEMORY_OPTIMIZED = ON, DURABILITY = SCHEMA_AND_DATA,

SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.CustomerOptimzedHistory))  

GO 
  
Create Temporal Table [dbo].[CustomerOptimzed] with a named History table [dbo].[CustomerOptimzedHistory]:

INSERT INTO dbo.CustomerOptimzed (FirstName, LastName,    Amount_Purchased) 

VALUES ('Frank', 'Sinatra', 20000.00),

('Shawn', 'McGuire', 30000.00),

('Amy', 'Carlson', 40000.00) 

GO 
  
Note: The History Table is used for versioning, whereby it stores the versions of each and every Row/Instance in the Main Memory Table. Example

DECLARE @StartDate DATETIME = '2015-12-29 07:50',

    @EndDate DATETIME = '2015-12-29 07:54'

SELECT Id, Name, StartTime, EndTime 

FROM dbo.Customer

FOR SYSTEM_TIME FROM @StartDate TO @EndDate

3.	**User-Defined Table Variable**

CREATE TYPE MemoryOptTabVar as TABLE

(

   [ID] INT IDENTITY(1,1) NOT NULL ,

   [First_Name] nvarchar(10) NULL,

   [Last_Name] nvarchar(10) NULL,

   INDEX IX_MemoryOptTabVar_ID NONCLUSTERED (ID)

) WITH (MEMORY_OPTIMIZED = ON)

GO

CREATE PROCEDURE dbo.MemoryOptVarTable --- Memory Optimized variable Table

AS

BEGIN

SET NOCOUNT ON;

    DECLARE @MemOptVarTab MemoryOptTabVar;

    INSERT @MemOptVarTab (First_Name, Last_Name) VALUES ('Sanya', 'J')

END

GO

4.	**Procedure**
CREATE PROCEDURE dbo.AddCustomer

(
    @CustomerId INT,

    @FirstName NVARCHAR(50),

    @LastName NVARCHAR(50)

)

WITH NATIVE_COMPILATION

        ,SCHEMABINDING

        ,EXECUTE AS OWNER

AS

BEGIN ATOMIC WITH (TRANSACTION ISOLATION LEVEL = SNAPSHOT,

                LANGUAGE = 'us_english')

    INSERT INTO dbo.Customer

    VALUES (@CustomerId, @FirstName, @LastName)

END
