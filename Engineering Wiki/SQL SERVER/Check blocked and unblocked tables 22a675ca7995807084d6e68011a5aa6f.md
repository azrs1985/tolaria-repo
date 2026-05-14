# Check blocked and unblocked tables

Owner: Nam Tran
Last edited time: August 28, 2025 1:37 PM

```sql
-- Get the request session id by executing following SQL statement
SELECT  
    OBJECT_NAME(p.[object_id]) BlockedObject
FROM    sys.dm_exec_connections AS blocking
    INNER JOIN sys.dm_exec_requests blocked
        ON blocking.session_id = blocked.blocking_session_id
    INNER JOIN sys.dm_os_waiting_tasks waitstats
        ON waitstats.session_id = blocked.session_id
    INNER JOIN sys.partitions p ON SUBSTRING(resource_description, 
        PATINDEX('%associatedObjectId%', resource_description) + 19, 
        LEN(resource_description)) = p.partition_id

-- Get the request session id by executing following SQL statement
SELECT
    OBJECT_NAME(P.object_id) AS TableName,
    Resource_type,
    request_session_id
FROM
    sys.dm_tran_locks L
JOIN
    sys.partitions P ON L.resource_associated_entity_id = p.hobt_id
WHERE
  OBJECT_NAME(P.object_id) = 'table_name'

-- We can check or get closer look or find all blocking or locking on database tables by following script:
SELECT
     blocking_session_id AS BlockingSessionID,
     session_id AS VictimSessionID,
 
     (SELECT [text] FROM sys.sysprocesses
      CROSS APPLY sys.dm_exec_sql_text([sql_handle])
      WHERE spid = blocking_session_id) AS BlockingQuery,
 
     [text] AS VictimQuery,
     wait_time/1000 AS WaitDurationSecond,
     wait_type AS WaitType,
     percent_complete AS BlockingQueryCompletePercent
FROM sys.dm_exec_requests
CROSS APPLY sys.dm_exec_sql_text([sql_handle])
WHERE blocking_session_id > 0

-- Kill the request session id which has kept lock on the table ‘table_name‘. Assume its request_session_id is 148. Execute following query:
Kill 148
```