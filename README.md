# Shrinking Activity

## What and Why Shrinking:- 
Shrinking is a database activity which de-allocate the unused spaced used by SQL database server. Database does not released automatically occupied spaces for the records.

SQL database does not released occupied spaces automatically. 
Shrinking operation is most effective after an operation that creates lots of unused space such as TRUNCATE, DROP a table or DELETE data from table.

Shrinking database files recovers space moving pages of data from end of the file to unoccupied space closer to the front of file. When enough free space is created at the front of the file, data pages at the end of the file can be de-allocated and renamed to the file system.

## Limitation and restriction:- 
The database cannot be made smaller than the minimum size of the database.
You cannot shrink a database while the database is being backed up and also you cannot take the backup a database while a shrink operation on the database is in process.
Unless you have a specific requirement, do not set the AUTO_SHRINK database option to ON. 

## Display Data and Log space Information in a Database:-
A) Using Management studio:-

1) Connect the database engine.
2) Select a database from management studio which you want to check the space.
3) Right-click on the database, point to Reports, point to Standard Reports, and then click Disk Usage.

B) Using T-SQL:- 
1) Connect the database engine 
2) Open new query and fire below system stored procedure.
```
Use #Database_name
Sp_spaceused
```
-- To know the space of tables
Use #Database_name
Sp_spaceused N’#Table_Name’

Perform Shrinking:-

Using Management Studio:- 
1) In Object Explorer, connect to an instance of the SQL server database engine.
2) Expand the database and then right click on the database that you want to shrink.
3) Point to Task, pont ti Shrink, and then click File.
It will display the name of the database, 

Current allocated space- Display the total used and unused space for the selected database.

Available free space- Display the sum of free space in the log and data files of the selected database name.

Shrink Action- In this section you can choose what action you want to take while shrinking the database. In this section choose Reorganize pages before releasing unused space and give the 1GB at a time for the shrink.

Using T-SQL:-
```
USE [#DataBase_Name]
GO
DBCC SHRINKFILE (N'#DataFile_Name' , MinimumSpace_ToBeShrink)
GO
```
Please do not shrink all unused space the database file at one time.

Use below script to shrink the data taking 1000 MB at a time.

After executing this script take the output of the script and run in another window.

Script 1
```
declare @id int, @cnt int;
set @id = 270000  -- Set the size of database (maxSize-10000)
set @cnt = 284200  -- Set the maximam size in MB of database here
while @cnt >= @id
begin
	DECLARE @SSQL NVARCHAR(MAX)
	SET @SSQL = ' Use [#Database_name]
	GO

	DBCC SHRINKFILE (#DataFile_name, ' + cast(@cnt as nvarchar) + ') 
	Go'

	print (@SSQL)
	SET @cnt = @cnt - 1000   -- We are breaking the shrinking database in 1000 MB in one time
END
```
After completion of script one time please reorganize the indexes using below script.

Script2
```
SELECT 'ALTER INDEX '+b.name+' ON '+C.name+' REORGANIZE;'
FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, NULL) AS ps
INNER JOIN sys.indexes AS b ON ps.OBJECT_ID = b.OBJECT_ID
inner join sys.tables as c ON b.object_id = c.object_id
AND ps.index_id = b.index_id
WHERE ps.database_id = DB_ID()
and b.name is not null
and ps.avg_fragmentation_in_percent between 5 and 29
ORDER BY ps.avg_fragmentation_in_percent desc
```
When you will execute this script then take the output and run in another window.


Now again go script 1 and set the size of the shrinking 10000 MB. Then again execute the script 2 again. Repeat these steps again and again until Current allocated space does not become equal to Available free space.

## Why Shrinking is not recommended?
Most databases requires some free space to be available for regular day-to-day operations. If you shrink the database repeatedly and notice that the database size growth again, this indicates that the space that was shrunk is required for regular operations. In this cases, repeatedly shrinking the database is a wasted operation.

A shrink operation does not preserve the fragmentation state of indexes in the database, and generally increases fragmentation to a degree. The most fragmentation of indexes on a table is decrease the performance of the database. This is another reason not to repeatedly shrink the database.

## After Shrink the Database:- 
Data that is move moved to shrink a file can be scattered to any available location in the file. This causes index fragmentation and can slow the performance of queries that search a range of the Index. To eliminate the fragmentation, consider rebuilding or reorganizing the indexes on the file after shrinking.

### Script of reorganize indexes

In the out put of this script will make the query to reorganize all the indexes in database.

Take that query and execute all in another query window.

```
SELECT 'ALTER INDEX '+b.name+' ON '+C.name+' REORGANIZE;'
FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, NULL) AS ps
INNER JOIN sys.indexes AS b ON ps.OBJECT_ID = b.OBJECT_ID
inner join sys.tables as c ON b.object_id = c.object_id
AND ps.index_id = b.index_id
WHERE ps.database_id = DB_ID()
and b.name is not null
and ps.avg_fragmentation_in_percent between 5 and 29
ORDER BY ps.avg_fragmentation_in_percent desc
```

