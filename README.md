# Snippets
## _I store here useful snippets that im probably going to forget_
 ---
## SQL Server

- Enable cdc on table
```sql
USE Rhdb;

EXECUTE sys.sp_cdc_enable_table
    @source_schema = N'Rhhh',
    @source_name = N'Employee',
    @supports_net_changes = 1,
   	@role_name = N'cdc_admin';
```
- List all cdc active tables
```sql
USE TestDb;

GO
SELECT 
    s.name AS Schema_Name, 
    tb.name AS Table_Name, 
    tb.object_id, tb.type, 
    tb.type_desc, 
    tb.is_tracked_by_cdc
FROM sys.tables tb
INNER JOIN sys.schemas s on s.schema_id = tb.schema_id
WHERE tb.is_tracked_by_cdc = 1
```
