

This article covers an example of how to easily access JSON documents in Oracle DB without loading! This approach may be useful for some integration use-cases wherein json data needs to be processed/uploaded into traditional tables. Instead of writing complicated parsing & loading routines, the "schema-less" datamodel using JSON will be a natural fit for RESTful development techniques, and the power of SQL would come in handy for analytic requirements.

Oracle 21c introduced a new JSON data type. You should use this in preference to other data types. More details here.


The JSON document file.
        The example given in this doc is based on Oracle DB JSON Developer's Guide. A sample file is given in Github that is a typical json dump format of NOSQL databases. i.e. its JSON objects separated by CRLF
        The use-case I was working on was for ingesting data from external database (MSSQL) - The queries used to produce JSON output in desired format is given here for reference. We use a docker container of mssql for this example.

```
docker run --name mssql -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Y0urStr0ngPassw0rd" -p 1433:1433 -v /scratch/db-docker/obpm-db-14-6/share/:/share -d mcr.microsoft.com/mssql/server:2019-latest
docker exec -it mssql bash -c "/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Y0urStr0ngPassw0rd' -Q 'set nocount on;SELECT [value] FROM OPENJSON((select * from sys.databases for json PATH))' -y0 -o /share/data.json"
```

        The above query using sqlcmd will output the table sys.databases into data.json with following content. query with "for JSON PATH" produces the output in JSON format. -y0 is for not truncating the data. "set nocount on" is basically equivalent of "set feed off" in sql*plus.

    Lets "load" this in Oracle without loading. The file we created is in a directory that Oracle DB can access (if not move this file to make it available for Oracle database)


* Create Directory
```
        create or replace directory JSON_DATA as '/share';
```

* Create an external table

```
        CREATE TABLE json_dump_file_contents (json_document BLOB)
          ORGANIZATION EXTERNAL (TYPE ORACLE_LOADER DEFAULT DIRECTORY JSON_DATA
                                 ACCESS PARAMETERS
                                   (RECORDS DELIMITED BY 0x'0A'
                                    DISABLE_DIRECTORY_LINK_CHECK
                                    FIELDS (json_document CHAR(32000)))
                                 LOCATION (JSON_DATA:'data.json'))
          PARALLEL
          REJECT LIMIT UNLIMITED;
```
* Read the data
```
        select * from json_dump_file_contents;
```
            All the data placed in the file is available for the DB engine to read now
        You can also read the JSON elements easily using the "simple dot notation". E.g.
        select t.json_document.name, t.json_document.database_id, t.* from json_dump_file_contents t;


    The fun doesn't end there. Lets use Oracle JSON features to parse it and make sense of the data

        JSON Data Guide to "describe" the data
````
        SELECT JSON_DATAGUIDE(json_document,DBMS_JSON.pretty) dg_doc
        FROM   json_dump_file_contents;
```

            Database will read the document contents and make a dictionary out of it.
        Since the data is not really loaded to the DB, every time when you query the data, it reads the file directly. But for better performance if you have multiple queries that target different rows, you can load an ordinary database table from the data in the external table. This also offers other benefits such as you can index the data, and make automatic views representing the JSON document in traditional RDBMS structure

            Create "normal" table. A check constraint is put to ensure document is JSON. With Oracle 21c JSON datatype this is not required.
```
            CREATE TABLE t_json_dump_file_contents
              (id          VARCHAR2 (32) NOT NULL PRIMARY KEY,
               date_loaded TIMESTAMP (6) WITH TIME ZONE,
               json_document BLOB
               CONSTRAINT ensure_json CHECK (json_document IS JSON))
              LOB (json_document) STORE AS (CACHE);

            Load the data
            INSERT INTO t_json_dump_file_contents (id, date_loaded, json_document)
              SELECT SYS_GUID(), SYSTIMESTAMP, json_document FROM json_dump_file_contents
                WHERE json_document IS JSON;
            Now you can query the data coming from the normal table.
            select t.json_document.name, t.json_document.database_id, t.* from t_json_dump_file_contents t;
```

            Create Search Index
```
            CREATE SEARCH INDEX json_docs_search_idx1 ON t_json_dump_file_contents (json_document) FOR JSON;
```
            We can now create a view - that automatically understands the data model as per JSON documents present in the table
```
            BEGIN
              DBMS_JSON.create_view(
                viewname  => 'json_documents_v1',
                tablename => 't_json_dump_file_contents',
                jcolname  => 'json_document',
                dataguide =>  DBMS_JSON.get_index_dataguide(
                                't_json_dump_file_contents',
                                'json_document',
                                DBMS_JSON.format_hierarchical));
            END;
            /
            Select from the view
            select * from json_documents_v1;
```
            You would notice that the JSON data is now automatically represented in RDBMS structure, with rows and columns - and any SQL can be written for analytics purposes
    TIP: DBMS_JSON.create_view cannot be done directly on the external table. But if you dont want to have this "normal" table in the picture, and want the data always from external table, but in the RDMS view format... Then you can temporarily create this normal table and view like above, and then the DDL of the view to point to the external table. you can also rename the column to remove "JSON_DOCUMENT$" prefix if not needed.

        E.g.
         Expand source
        Now the data will be directly fetched from the external file and presented to you in RDBMS format.


* SODA Access *

    *** Literature to be added later ***
```
        Create collection
        DECLARE
          l_collection  SODA_COLLECTION_T;
        BEGIN
          l_collection := DBMS_SODA.create_collection('TestCollection1');
         
          IF l_collection IS NOT NULL THEN
            DBMS_OUTPUT.put_line('Collection ID : ' || l_collection.get_name());
          ELSE
            DBMS_OUTPUT.put_line('Collection does not exist.'); 
          END IF;
        END;
        /
        Load the data to collection... you can use REST APIs for a full NOSQL style loading. In the below example I am loading using PLSQL API
        DECLARE
          l_collection  SODA_COLLECTION_T;
          l_document    SODA_DOCUMENT_T;
          l_status      NUMBER;
        BEGIN
            delete "TestCollection1";
            l_collection := DBMS_SODA.open_collection('TestCollection1');
            for i in (select * from json_dump_file_contents)
            loop
                l_document := SODA_DOCUMENT_T(
                            b_content => i.json_document
                            );
                l_status := l_collection.insert_one(l_document);
            end loop;
            DBMS_OUTPUT.put_line('status    : ' || l_status);
          COMMIT;
        END;
        /
        Now the data will be available in JSON collection for query using SODA APIs
            ** to be added later ***
    Cleanup
        DECLARE
          l_status  NUMBER := 0;
        BEGIN
          l_status := DBMS_SODA.drop_collection('TestCollection1');
        END;
        /
         
        drop view vw_json_dump_file_contents;
        drop view json_documents_v1;
        drop table t_json_dump_file_contents;
        drop table json_dump_file_contents;

```
