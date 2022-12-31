# SQL Server Table Partition Script

Attention !!!
- This script is created for SQL Server 2012 and above.
- The script is designed for creating table partitions based on date (monthly basis) as the primary key.
- This script will select the latest partition filegroup (if already created) and generate a script to create another filegroup.
- This will also create the partition schema name and schema function.
- In short, it will generate all the necessary scripts you just need to change before execution as per your requirement.

#Script:



USE [master]
GO

SELECT 
	[DB_File_Logical_Name]
	, FileGroupName
	, [DB_File_Physical_Name]
	, [FileName] = [DB_File_Logical_Name] + N'.ndf'
	, PartitionScheme 
	, PartitionFunction
	, PartitionFunctionValue		
	, Script = CONCAT(
	N'GO'
	, CHAR(13), CHAR(10) 
	,N'ALTER DATABASE [internaltoolset] ADD FILEGROUP [', FileGroupName, N'];'
	, CHAR(13), CHAR(10), CHAR(13), CHAR(10) 
	,N'ALTER DATABASE [internaltoolset] ADD FILE'
	, CHAR(13), CHAR(10) 
	,N'('
	, CHAR(13), CHAR(10), CHAR(13), CHAR(10)
	,N'NAME = N''', [DB_File_Logical_Name], N''''
	, CHAR(13), CHAR(10) 
	,N', FILENAME = N''', [DB_File_Physical_Name], N''''
	, CHAR(13), CHAR(10) 
	,N', SIZE = 8192KB , FILEGROWTH = 65536KB'
	, CHAR(13), CHAR(10) 
	,N') TO FILEGROUP [', FileGroupName, N'];'
	, CHAR(13), CHAR(10), CHAR(13), CHAR(10) 
	,N'ALTER PARTITION SCHEME '
	, CHAR(13), CHAR(10) 
	, PartitionScheme
	, CHAR(13), CHAR(10) 
	,N'NEXT USED [', FileGroupName, N'];'
	, CHAR(13), CHAR(10), CHAR(13), CHAR(10) 
	,N'ALTER PARTITION FUNCTION '
	, CHAR(13), CHAR(10) 
	, PartitionFunction
	, CHAR(13), CHAR(10) 
	,N' ()'
	, CHAR(13), CHAR(10) 
	,N'SPLIT RANGE (', PartitionFunctionValue, N'); '
	, CHAR(13), CHAR(10) 
	,N'GO '
	)
FROM 
	(
		SELECT DISTINCT 
			ps.Name AS PartitionScheme
			, pf.name AS PartitionFunction
			, fg.name AS FileGroupName
			, N'''' + FORMAT(CONVERT(DATE, rv.value), N'yyyy-MM-dd 00:00:00.000') + N'''' AS PartitionFunctionValue
			, RANK() OVER(PARTITION BY ps.Name, pf.name ORDER BY rv.value DESC) AS RK
		FROM 
			sys.indexes i  
			JOIN sys.partitions p ON i.object_id = p.object_id AND i.index_id = p.index_id  
			JOIN sys.partition_schemes ps ON ps.data_space_id = i.data_space_id  
			JOIN sys.partition_functions pf ON pf.function_id = ps.function_id  
			LEFT JOIN sys.partition_range_values rv ON rv.function_id = pf.function_id AND rv.boundary_id = p.partition_number
			JOIN sys.allocation_units au  ON au.container_id = p.hobt_id   
			JOIN sys.filegroups fg  ON fg.data_space_id = au.data_space_id  
		--WHERE i.object_id = object_id('[schema].[tablename]') 
	) AS T
	LEFT JOIN (
		SELECT 
			[Filegroup_Name] = fg.[name],
			[DB_File_Logical_Name] = df.[name],
			[DB_File_Physical_Name] = df.[physical_name]
		FROM sys.filegroups fg
		JOIN sys.database_files df ON df.[data_space_id] = fg.[data_space_id]
	) AS DF ON T.FileGroupName = DF.[Filegroup_Name]
WHERE
	RK = 1
ORDER BY 
	FileGroupName
	, PartitionFunctionValue
  
  
  
