# ETL Design and Data Model Documentation

- ETL flow for Products source file provided, is created by moving files from Products to staging and then staging to data warehouse
- ETL flow diagram and data model schema is added
- Tables Schemas, structures, relationships and stored procedures are added with SQL scripts
- SQL Scripts 


## 1. ETL Flow Diagram
https://github.com/lakshays175/AH-Techincal-Test/blob/31e8139101f2185d80474545203b6c28765c2e35/sql_etl_pipeline.png


## 2. Data Model Schema
https://github.com/lakshays175/AH-Techincal-Test/blob/31e8139101f2185d80474545203b6c28765c2e35/data%20model%20diagram.png

### Schema

- Staging
- data_quality
- logging
- dwh

### Tables

- **Staging.Product**: Contains the product source file with file date column
- **Data_quality.dqlog**: contains the data which was encountered on the staging table for the data quality checks
- **Logging.etllog**: It contians the log for failure and success of the staging and dwh tables
- **Dwh.dim_date**: Dimension tables which contains various fields related to date
- **Dwh.dim_Product**: Dimension table which takes the data from staging.Product using SC2 Type 2 logic
- **Dwh.fact_ProductSales**: Fact tables taking data from the staging.product, dim.product and dim_date

### Relationships

- **Dwh.Fact_ProductSales**: This table has the PK-FK relationship with dim_date and dim_Product

## 3. Implementation
## 4. Table/Schema Scripts
```sql
/****** Object:  Schema [DATA_QUALITY]    Script Date: 18-05-2025 22:11:15 ******/
CREATE SCHEMA [DATA_QUALITY]
GO
/****** Object:  Schema [DWH]    Script Date: 18-05-2025 22:11:15 ******/
CREATE SCHEMA [DWH]
GO
/****** Object:  Schema [LOGGING]    Script Date: 18-05-2025 22:11:15 ******/
CREATE SCHEMA [LOGGING]
GO
/****** Object:  Schema [STAGING]    Script Date: 18-05-2025 22:11:15 ******/
CREATE SCHEMA [STAGING]
GO
/****** Object:  Table [DATA_QUALITY].[dqlog]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [DATA_QUALITY].[dqlog](
	[ID] [int] IDENTITY(1,1) NOT NULL,
	[TableName] [nvarchar](128) NULL,
	[UniqueID] [varbinary](64) NULL,
	[DQCheck] [nvarchar](512) NULL,
	[loaddate] [date] NULL,
PRIMARY KEY CLUSTERED 
(
	[ID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [DWH].[dim_date]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [DWH].[dim_date](
	[DateKey] [int] NOT NULL,
	[Date] [date] NOT NULL,
	[Day] [int] NOT NULL,
	[Month] [int] NOT NULL,
	[Year] [int] NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[DateKey] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [DWH].[dim_Product]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [DWH].[dim_Product](
	[DimProductID] [int] IDENTITY(1,1) NOT NULL,
	[SKU] [int] NOT NULL,
	[Category] [nvarchar](100) NOT NULL,
	[name] [nvarchar](50) NOT NULL,
	[Description] [varchar](512) NOT NULL,
	[etl_startdate] [datetime] NOT NULL,
	[etl_enddate] [datetime] NULL,
	[IsCurrent] [bit] NOT NULL,
	[HashBytes] [varbinary](64) NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[DimProductID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [DWH].[fact_ProductSales]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [DWH].[fact_ProductSales](
	[ProductSalesID] [bigint] IDENTITY(1,1) NOT NULL,
	[ProductKey] [int] NOT NULL,
	[DateKey] [int] NOT NULL,
	[Sold] [int] NOT NULL,
	[Price] [float] NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[ProductSalesID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
/****** Object:  Table [LOGGING].[auditlog]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [LOGGING].[auditlog](
	[LogID] [int] IDENTITY(1,1) NOT NULL,
	[TableName] [nvarchar](100) NULL,
	[RowCount] [int] NULL,
	[LoadDate] [datetime] NULL,
	[Message] [nvarchar](max) NULL,
PRIMARY KEY CLUSTERED 
(
	[LogID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
/****** Object:  Table [STAGING].[Product]    Script Date: 18-05-2025 22:11:15 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [STAGING].[Product](
	[sku] [int] NOT NULL,
	[category] [nvarchar](50) NOT NULL,
	[name] [nvarchar](50) NOT NULL,
	[price] [float] NOT NULL,
	[sold] [smallint] NOT NULL,
	[description] [nvarchar](100) NOT NULL,
	[etlloaddate] [date] NULL
) ON [PRIMARY]
GO
ALTER TABLE [DATA_QUALITY].[dqlog] ADD  DEFAULT (getdate()) FOR [loaddate]
GO
ALTER TABLE [DWH].[dim_Product] ADD  DEFAULT ((1)) FOR [IsCurrent]
GO
ALTER TABLE [LOGGING].[auditlog] ADD  DEFAULT (getdate()) FOR [LoadDate]
GO
ALTER TABLE [DWH].[fact_ProductSales]  WITH CHECK ADD FOREIGN KEY([DateKey])
REFERENCES [DWH].[dim_date] ([DateKey])
GO
ALTER TABLE [DWH].[fact_ProductSales]  WITH CHECK ADD FOREIGN KEY([ProductKey])
REFERENCES [DWH].[dim_Product] ([DimProductID])
GO
```

## 5. Stored Procedures / SQL Logic

### a. SQL Logic to Load File into Staging
```sql

  BULK INSERT Staging.Product
  FROM 'C:\Data\Products_20250120.csv'
  WITH (
      FIRSTROW = 2,
      FIELDTERMINATOR = ';',
      ROWTERMINATOR = '\n',
      TABLOCK
  );

```

### b. Data Quality Procedure

```sql
ALTER PROCEDURE [dbo].[sp_DataQualityChecks]
@Table NVARCHAR(100),
@LoadDate NVARCHAR(10)
AS
/*
	Data Qaulity Checks
	1. Nulll Checks
	2. Invalid data (Negative or Zero Price)
	3. Duplicates Values
*/

BEGIN
    SET NOCOUNT ON;

	DECLARE @sql NVARCHAR(max);

	SET @sql=
	'
	DELETE FROM data_quality.dqlog WHERE TABLENAME = '''+ @Table +''' AND LOADDATE= '''+ @LoadDate +''';

    INSERT INTO data_quality.dqlog (TableName, UniqueID, DQCheck)
    SELECT '''+ @Table +''', HASHBYTES(''SHA2_256'', CONCAT_WS(''|'',SKU, category, [name],[Price],[Sold], [description])) , ''Null Values''
    FROM '+ @Table +'
    WHERE sku IS NULL OR category is null or [name] is null or price is null or sold is null or [description] is null;

    INSERT INTO data_quality.dqlog (TableName, UniqueID, DQCheck)
    SELECT '''+ @Table +''', HASHBYTES(''SHA2_256'', CONCAT_WS(''|'',SKU, category, [name],[Price],[Sold], [description]))  , ''Price is less than or equal to zero''
    FROM '+ @Table +'
    WHERE Price <= 0;

    INSERT INTO data_quality.dqlog (TableName, UniqueID, DQCheck)
    SELECT '''+ @Table +''',  HASHBYTES(''SHA2_256'', CONCAT_WS(''|'',SKU, category, [name],[Price],[Sold], [description]))  , ''Duplicate Values''
    FROM (
        SELECT SKU, category, [name],[Price],[Sold], [description]
        FROM '+ @Table +'
		GROUP BY SKU, category, [name],[Price],[Sold], [description]
		HAVING COUNT(*)>1

    ) dup;
	'

	EXEC (@SQL)
END;
```

### c. ETL Load Procedure

```sql
ALTER   PROCEDURE [dbo].[sp_etl_load_products]
@loaddate DATE=NULL,
@Table VARCHAR(50)=NULL
AS
/*
	ETL Data Load
	1.	Execute Data Qaulity Checks
	2.	Loading Dim date table based on the today's date
	3.	SCD2 type logic for dim_product using staging.product
	4.	Loading Fact Sales tables using staging.product
	5.	Audit log for failure or success
*/
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRANSACTION;

        DECLARE @DefaultDateKey INT = (SELECT TOP 1 DateKey FROM dwh.DIM_DATE WHERE [Date] = '1900-01-01'),
				@DefaultProductKey INT = (SELECT TOP 1 DimProductID FROM dwh.dim_product WHERE [name] = 'Unknown');


		EXEC [dbo].[sp_DataQualityChecks] 'STAGING.PRODUCT',@loaddate

		IF @loaddate is NULL
			SET @loaddate = CAST(GETDATE() AS [DATE])

        IF NOT EXISTS (SELECT 1 FROM dwh.dim_date WHERE [Date] = CAST(GETDATE() AS [DATE]))
        BEGIN
            INSERT INTO dwh.dim_date  (DateKey, [Date], [Day], [Month], [Year])
            SELECT
                CONVERT(INT, FORMAT(GETDATE() , 'yyyyMMdd')),
                CAST(GETDATE()  AS DATE),
                DATEPART(DAY, GETDATE() ),
                DATEPART(MONTH, GETDATE() ),
                DATEPART(YEAR, GETDATE() )
        END

		IF @Table= 'dwh.dim_product'
		BEGIN

			-- Create temp table for changed products BEFORE using it
			CREATE TABLE #Changed (
				[Action] VARCHAR(10),
				[SKU] int,
				[category] VARCHAR(128),
				[name] VARCHAR(128),
				[description] VARCHAR(128),
				[etl_startdate] DATETIME,
				[etl_enddate] DATETIME,
				IsCurrent BIT,
				[HashBytes] VARBINARY(64)
			);

			MERGE dwh.dim_product AS [target]
			USING (
				SELECT DISTINCT SKU, category, [name], [description],
				HASHBYTES('SHA2_256', CONCAT_WS('|',SKU, category, [name],[Price],[Sold], [description])) AS [HashBytes]
				FROM STAGING.[Product] sp
				WHERE sp.etlloaddate=@loaddate
				AND NOT EXISTS
				(SELECT 1
						FROM DATA_QUALITY.dqlog dq
						WHERE dq.TableName = 'Staging.Product'
						AND UniqueID= HASHBYTES('SHA2_256', CONCAT_WS('|',SP.SKU, SP.category, SP.[name],SP.[Price],SP.[Sold], SP.[description]))
						  )

			) AS [source]
			ON target.SKU = source.SKU AND target.isCurrent = 1

			WHEN NOT MATCHED BY TARGET THEN
				INSERT ([SKU], [category], [name], [description],etl_startdate,etl_enddate,[Hashbytes])
				VALUES (
					source.[SKU],
					source.[category],
					source.[name],
					source.[description],
					GETDATE(),
					NULL,
					source.[hashbytes]
				)

			WHEN MATCHED AND TARGET.[hashbytes] <> source.[hashbytes] THEN
				UPDATE SET
				target.isCurrent = 0, target.etl_enddate = GETDATE()
				OUTPUT
					$action,
					source.[SKU],
					source.[category],
					source.[name],
					source.[description],
					GETDATE(),
					NULL,
					1,
					source.[hashbytes]
				INTO #Changed;

			INSERT INTO dwh.dim_product(
						[SKU], [category], [name], [description],etl_startdate, [HashBytes]
					)
			SELECT [SKU], [category], [name], [description],etl_startdate, [HashBytes]
			FROM #Changed
			WHERE [action] = 'UPDATE';

			INSERT INTO [logging].auditlog(TableName, [RowCount], [Message])
			SELECT @Table, COUNT(*), 'ETL completed on ' + CAST(GETDATE() AS VARCHAR)
			FROM staging.[product] P
			WHERE p.etlloaddate=@loaddate

		-- Cleanup
		IF OBJECT_ID('tempdb..#Changed') IS NOT NULL
			DROP TABLE #Changed;
		END

		IF @Table= 'dwh.fact_ProductSales'
		BEGIN
			IF NOT EXISTS (SELECT 1 FROM dwh.fact_ProductSales WHERE [DateKey] = CONVERT(INT, FORMAT(GETDATE(), 'yyyyMMdd')))
			BEGIN
				INSERT INTO dwh.fact_ProductSales (DateKey, ProductKey, Sold, Price)
				SELECT
					ISNULL(d.DateKey, @DefaultDateKey),
					ISNULL(dp.DimProductID, @DefaultProductKey),
					P.Sold,
					P.Price
				FROM STAGING.[product] P
				LEFT JOIN dwh.DIM_DATE  D ON P.etlloaddate = D.[Date]
				LEFT JOIN dwh.dim_product  DP ON P.SKU = DP.SKU AND DP.IsCurrent = 1
				WHERE  p.etlloaddate=@loaddate
				AND NOT EXISTS
				(SELECT 1
						FROM DATA_QUALITY.dqlog dq
						WHERE dq.TableName = 'Staging.Product'
						  AND dq.UniqueID = HASHBYTES('SHA2_256', CONCAT_WS('|',P.SKU, P.category, P.[name],P.[Price],P.[Sold], P.[description]))
						  );
			END

			INSERT INTO [logging].auditlog(TableName, [RowCount], [Message])
			SELECT @Table, COUNT(*), 'ETL completed on ' + CAST(GETDATE() AS VARCHAR)
			FROM staging.[product] P
			WHERE p.etlloaddate=@loaddate
		END

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;

        DECLARE @ErrMsg NVARCHAR(MAX) = ERROR_MESSAGE();
        INSERT INTO [logging].auditlog(TableName, [RowCount], [Message])
        VALUES (@Table, 0, 'ETL Failed: ' + @ErrMsg);

        THROW;
    END CATCH
END;
```

### 6. Future Reports

```sql
--Most Sold Products Daily and Monthly
-- Daily
SELECT d.Date, p.name, SUM(f.Sold) AS TotalSold
FROM dwh.fact_ProductSales f
JOIN dwh.dim_date d ON f.DateKey = d.DateKey
JOIN dwh.dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY d.Date, p.name
ORDER BY d.Date, TotalSold DESC;

-- Monthly
SELECT d.Year, d.Month, p.name, SUM(f.Sold) AS TotalSold
FROM dwh.fact_ProductSales f
JOIN dwh.dim_date d ON f.DateKey = d.DateKey
JOIN dwh.dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY d.Year, d.Month, p.name
ORDER BY d.Year, d.Month, TotalSold DESC;

--Total Sales Per Category
SELECT p.Category, SUM(f.Price * f.Sold) AS TotalSales
FROM dwh.fact_ProductSales f
JOIN dwh.Dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY p.Category
ORDER BY TotalSales DESC;

--Most Expensive Category (based on average product price)
SELECT TOP 1 p.Category, sum(f.Price) AS SumPrice
FROM dwh.fact_ProductSales f
JOIN dwh.Dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY p.Category
ORDER BY SumPrice DESC;

--Category sales trend
--Monthly
SELECT d.Month, d.Year, p.Category, SUM(f.Price * f.Sold) AS MonthlySales
FROM dwh.fact_ProductSales f
JOIN dwh.dim_date d ON f.DateKey = d.DateKey
JOIN dwh.dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY d.Year, d.Month, p.Category
ORDER BY d.Year, d.Month, p.Category;

--Daily
SELECT d.Date, p.Category, SUM(f.Price * f.Sold) AS DailySales
FROM dwh.fact_ProductSales f
JOIN dwh.dim_date d ON f.DateKey = d.DateKey
JOIN dwh.dim_Product p ON f.ProductKey = p.DimProductID
GROUP BY d.Date, p.Category
ORDER BY d.Date, DailySales DESC;
```
