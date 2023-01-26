

    Created by Ranjith Vijayan on May 08, 2022


Refer Office Hours session https://asktom.oracle.com/pls/apex/f?p=100:551::::551:P551_CLASS_ID,P551_STUDENT_ID:17190,&cs=103A48767C6612847920B49ED12BAA316 for the full details of this feature !


The code used in the above session didn't work in Oracle DB 19.3 as expected, the solution is given in this doc.

```
SQL> conn RESTUTIL@PDB1
Enter password:
Connected.
 
Connected.
SQL> select banner_full from v$version;
 
BANNER_FULL
----------------------------------------------------------------------------------------------------------------------------------
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.14.0.0.0
 
 
SQL>
SQL> create table t_objects as select * from all_objects;
 
Table created.
 
SQL> alter table t_objects add js varchar2(100) ;
 
Table altered.
 
SQL> alter table t_objects add constraint js_chk check (js is json);
 
Table altered.
 
 
 
SQL> create procedure PR_TXT_SEARCH
  2  (
  3      rid in rowid, tclob in out nocopy clob
  4  )
  5  is
  6  begin
  7      select json_arrayagg(json_object(*))
  8      into tclob
  9      from t_objects
 10      where rowid = rid;
 11  end;
 12  /
 
Procedure created.
 
SQL>
SQL> begin
  2    ctxsys.ctx_ddl.create_preference('RESTUTIL.REST_IDX_DST','USER_DATASTORE');
  3    ctxsys.ctx_ddl.set_attribute('RESTUTIL.REST_IDX_DST','PROCEDURE','RESTUTIL.PR_TXT_SEARCH');
  4  end;
  5  /
 
PL/SQL procedure successfully completed.
 
SQL>
SQL> create search index RESTUTIL.i_objects
  2  on RESTUTIL.t_objects (js) for json
  3  parameters('sync (on commit) datastore REST_IDX_DST');
 
Index created.
 
SQL>
SQL> alter table RESTUTIL.t_objects modify js invisible;
 
Table altered.
 
SQL>
SQL> select * from RESTUTIL.t_objects
  2  where json_textcontains(js, '$', 'DUAL');
 
OWNER                          OBJECT_NAME
------------------------------ ----------------------------------------
SUBOBJECT_NAME
--------------------------------------------------------------------------------------------------------------------------------
 OBJECT_ID DATA_OBJECT_ID OBJECT_TYPE             CREATED   LAST_DDL_ TIMESTAMP           STATUS  T G S  NAMESPACE
---------- -------------- ----------------------- --------- --------- ------------------- ------- - - - ----------
EDITION_NAME
--------------------------------------------------------------------------------------------------------------------------------
SHARING            E O A DEFAULT_COLLATION                                                                                    D S
------------------ - - - ---------------------------------------------------------------------------------------------------- - -
CREATED_APPID CREATED_VSNID MODIFIED_APPID MODIFIED_VSNID
------------- ------------- -------------- --------------
SYS                            DUAL
 
       143            143 TABLE                   30-MAY-19 05-NOV-21 2019-05-30:03:10:17 VALID   N N N          1
 
METADATA LINK        Y N USING_NLS_COMP                                                                                       N N
 
 
PUBLIC                         DUAL
 
       144                SYNONYM                 30-MAY-19 30-MAY-19 2019-05-30:03:10:17 VALID   N N N          1
 
METADATA LINK      N Y N                                                                                                      N N
 
 
SYS                            AV_DUAL
 
      1695           1695 TABLE                   30-MAY-19 30-MAY-19 2019-05-30:03:11:06 VALID   N N N          1
 
METADATA LINK        Y N USING_NLS_COMP                                                                                       N N
 
 
SYS                            GV_$DUAL
 
      4516                VIEW                    30-MAY-19 30-MAY-19 2019-05-30:03:15:05 VALID   N N N          1
 
METADATA LINK      N Y N USING_NLS_COMP                                                                                       N N
 
 
PUBLIC                         GV$DUAL
 
      4517                SYNONYM                 30-MAY-19 30-MAY-19 2019-05-30:03:15:05 VALID   N N N          1
 
METADATA LINK      N Y N                                                                                                      N N
 
 
SYS                            V_$DUAL
 
      4518                VIEW                    30-MAY-19 30-MAY-19 2019-05-30:03:15:05 VALID   N N N          1
 
METADATA LINK      N Y N USING_NLS_COMP                                                                                       N N
 
 
PUBLIC                         V$DUAL
 
      4519                SYNONYM                 30-MAY-19 30-MAY-19 2019-05-30:03:15:05 VALID   N N N          1
 
METADATA LINK      N Y N                                                                                                      N N
 
 
 
7 rows selected.
```


in 19.3 version, the above index creation didnt work as expected.


Upon investigation

```
select * from CTX_USER_INDEX_ERRORS ;
===below error
DRG-12604: execution of user datastore procedure has failed
DRG-50857: oracle error in drsinopen
ORA-00904: "RID": invalid identifier
ORA-06512: at "RESTUTIL.PR_TXT_SEARCH", line 7
ORA-06512: at line 1
```


Solution to the above problem is to use pl/sql  json_object_t and json_array_t instead of json_arrayagg.

```
create or replace procedure PR_TXT_SEARCH
(
    p_rowid in rowid, tclob in out nocopy clob
)
is
o json_object_t := new json_object_t;
a json_array_t := new json_array_t;
begin
    for i in (
    select * --json_arrayagg(json_object(*)) j
    --into tclob
    from t_objects a
    where a.rowid = p_rowid
    )
    LOOP
        o.put('OWNER',i.owner);
        o.put('OBJECT_NAME',i.object_name);
        --etc
    end loop;
    a.append(o);
    tclob := a.to_string();
end;
/
```




