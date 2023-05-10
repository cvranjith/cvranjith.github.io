The functionality provided by the DBMS_PARALLEL_EXECUTE package enables the division of a workload related to a base table into smaller fragments that can be executed in parallel. This article utilizes a basic update statement as an example, although in practice, it is more efficient to employ a single parallel DML statement. Nonetheless, this simple example effectively illustrates how to use the package. The DBMS_PARALLEL_EXECUTE package is particularly useful in situations where straight parallel DML is not suitable. It entails a series of distinct stages to implement.


The task at hand is linked to a base table, which can be fragmented into subsets or "chunks" of rows. The workload can be split into these chunks using one of three methods:

    * CREATE_CHUNKS_BY_ROWID
    * CREATE_CHUNKS_BY_NUMBER_COL
    * CREATE_CHUNKS_BY_SQL

Once the chunks have been created and associated with a task, they can be removed using the DROP_CHUNKS procedure.


Following is an example of creating chunks by ROWID and executing them in parallel


``` sql

DECLARE
  l_task     VARCHAR2(30) := 'stat_task';
  l_sql_stmt VARCHAR2(32767);
BEGIN
 
  execute immediate 'truncate table t_stat';
 
  insert /*+ append */ into t_stat select branch_code, cust_ac_no, account_class, null from sfcubs.sttm_cust_account where record_stat = 'O';
  commit;
 
  DBMS_PARALLEL_EXECUTE.create_task (task_name => l_task);
 
  DBMS_PARALLEL_EXECUTE.create_chunks_by_rowid(task_name   => l_task,
                                               table_owner => user,
                                               table_name  => 'T_STAT',
                                               by_row      => TRUE,
                                               chunk_size  => 10000);
 
  l_sql_stmt := 'UPDATE t_stat t set stat = sfcubs.fn_ac_stat(branch_code, cust_ac_no, account_class) WHERE rowid BETWEEN :start_id AND :end_id';
 
  DBMS_PARALLEL_EXECUTE.run_task(task_name      => l_task,
                                 sql_stmt       => l_sql_stmt,
                                 language_flag  => DBMS_SQL.NATIVE,
                                 parallel_level => 60);
 
  DBMS_PARALLEL_EXECUTE.drop_task(l_task);
END;
/
```



Below is the explanation:

    * Truncating the table: The execute immediate 'truncate table t_stat' statement truncates the t_stat table to remove any previous data. This ensures that the table is clean before new data is inserted.
    * Inserting data into the table: The insert /*+ append */ statement inserts data into the t_stat table. This data is selected from the sttm_cust_account table in the sfcubs schema, where record_stat is equal to 'O'. The /*+ append */ hint directs the database to bypass any space management checks and write directly to the end of the table, making the insert operation faster.
    * Creating a parallel task: The DBMS_PARALLEL_EXECUTE.create_task procedure creates a parallel task named l_task that can be used to execute SQL statements in parallel.
    * Creating chunks: The DBMS_PARALLEL_EXECUTE.create_chunks_by_rowid procedure creates chunks of data from the t_stat table based on the ROWID of each row. The chunk_size parameter specifies the number of rows in each chunk. In this example, each chunk contains 10,000 rows.
    * Defining the SQL statement: The l_sql_stmt variable is set to an update statement that updates the stat column of the t_stat table using the sfcubs.fn_ac_stat function. The :start_id and :end_id bind variables are used to specify the range of ROWIDs for each chunk.
    * Running the parallel task: The DBMS_PARALLEL_EXECUTE.run_task procedure executes the l_sql_stmt SQL statement in parallel using the l_task parallel task. The parallel_level parameter specifies the number of parallel threads to use. In this example, 60 parallel threads are used.
    * Dropping the parallel task: The DBMS_PARALLEL_EXECUTE.drop_task procedure drops the l_task parallel task once the parallel operation is complete


The status of the processing can be checked using below sqls

```sql
SELECT status, COUNT(1) FROM user_parallel_execute_chunks GROUP BY status;
 
STATUS                 COUNT(1)
-------------------- ----------
PROCESSED                   621
 
 
 
 
SELECT job_prefix FROM user_parallel_execute_tasks WHERE task_name = 'stat_task';
 
JOB_PREFIX
------------------------------
TASK$_245
```
