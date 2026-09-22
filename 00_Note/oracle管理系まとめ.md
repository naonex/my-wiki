#### 以下コマンドを実行しておくと、sqlplusでバックスペースが使える

```
stty erase ^H
```

#### PLSQL内にsqlトレースを埋め込む

```sql
BEGIN

DBMS_SESSION.SESSION_TRACE_ENABLE(waits => true, binds => true);

BEGIN

--【メイン処理】

END;

DBMS_SESSION.SESSION_TRACE_DISABLE;

END;
```

sqlトレースファイル解析コマンド例
`/opt/app/oracle/product/12.1.0/dbhome_1/bin/tkprof ora_xxxxx_xxxxx.trc test`

---

##### 表領域削除（データファイルごと）

```sql
DROP TABLESPACE FXITNINDX_TBSE INCLUDING CONTENTS AND DATAFILES;
```

---

## スキーマ移行
##### sqlplus 接続先確認

```sql
select instance_name from v$instance;
show parameter instance_name

--エクスポート先確認（DATA_PUMP_DIR,EXPORT_DIR）
SELECT * FROM DBA_DIRECTORIES;
```


##### linux上での確認

```bash
printenv ORACLE_SID

export ORACLE_SID=USERDB
export ORACLE_SID=USERDBDEV

# ※oracleDB内で定義されているディレクトリしか使えないので注意
expdp SYSTEM/password DIRECTORY=EXPORT_DIR DUMPFILE=yyyymmdd.dmp LOGFILE=yyyymmdd.log schemas=USER1

impdp SYSTEM/password directory=DATA_PUMP_DIR dumpfile=yyyymmdd.dmp logfile=yyyymmdd.log schemas=USER1
```

※オブジェクト権限は手動で与えなければならない

```sql
GRANT EXECUTE ON DBMS_AQ TO USER1;
GRANT EXECUTE ON DBMS_AQADM TO USER1;
GRANT EXECUTE ON DBMS_CRYPTO TO USER1;
GRANT EXECUTE ON DBMS_LOCK TO USER1;

-- ユーザー削除
drop user USER1 cascade;
```

---


#### データファイル内のDBデータ確認

```sql
select e.owner, e.segment_name, e.segment_type, count(1) cnt
	from dba_data_files f, dba_extents e
	where f.file_name = '/xxx/oradata/orcl/user01.dbf'
		and f.file_id = e.file_id
	group by e.owner, e.segment_name, e.segment_type;
```

##### DBインスタンス確認

```sql
SELECT INSTANCE_NAME, HOST_NAME, VERSION, STARTUP_TIME FROM V$INSTANCE@LNK_APPDB;
```

##### DB内のソースで使用されている文字を検索する
```sql
  SELECT *
  FROM user_source
  where text like '%target1%'
     or text like '%target2%'
     or text like '%target3%'
```

##### ログインユーザー情報を調べる

```sql
select * from user_users@LNK_USERDB
```

##### INVALID確認

```sql
SELECT OBJECT_TYPE, OBJECT_NAME FROM USER_OBJECTS WHERE STATUS = 'INVALID' ORDER BY OBJECT_TYPE, OBJECT_NAME;
--全体用
SELECT OBJECT_TYPE, OBJECT_NAME FROM DBA_OBJECTS WHERE STATUS = 'INVALID' ORDER BY OBJECT_TYPE, OBJECT_NAME;
```

##### 上から5行まで取得

```sql
select * from (select * from TABLE1 where COL1 >= to_date('2018/12/15','yyyy/mm/dd') order by COL1) where ROWNUM <= 5;
select * from (select * from TABLE1 order by COL1 desc) where rownum <= 5;

--【ケース1】
select * from table1 
where ASEQNO between
(select max(ASEQNO) -4 from table1)
and (select max(ASEQNO) from table1)
order by ASEQNO desc;

--【ケース2】
select *
from (select * from table1 
 order by ASEQNO desc)
WHERE ROWNUM <= 5;
```

##### テーブルロック

```sql
LOCK TABLE TABLE1
   IN EXCLUSIVE MODE 
   NOWAIT; 
```

#### 現在実行中のSQLを調べる

```sql
select 
    a.INST_ID,
    a.SCHEMANAME,
    a.SQL_ID,
    a.WAIT_CLASS,
    a.EVENT,
--  b.SQL_FULLTEXT,
    trunc((sysdate - a.SQL_EXEC_START) * 24 * 60 * 60,2) as "経過",
--  a.WAIT_TIME,
    substr(b.SQL_TEXT,1, 60) as sql_head
from 
    gV$SESSION a,
    gV$sqlarea b
where 1 = 1
and status = 'ACTIVE'
and a.INST_ID = b.INST_ID
and a.SQL_ID = b.SQL_ID
order by a.INST_ID,a.SQL_EXEC_START 
;

select * from V$SQL_MONITOR where status = 'EXECUTING'
select * from V$SQL_PLAN_MONITOR where status = 'EXECUTING'
```

#### 累計調査用SQLサンプル

```sql
 select
  A.日付,
  A.集計,
  sum(B.集計) as 累計
 from (select trunc(updatedate) as 日付 ,count(updatedate) as 集計 from table1 group by trunc(updatedate) order by 日付) A
 left outer join (select trunc(updatedate) as 日付 ,count(updatedate) as 集計 from table1 group by trunc(updatedate) order by 日付) B
  on B.日付 <= A.日付
 group by A.日付
```

#### 表領域系

```sql
--ALTER DATABASE RENAME FILE '変更前のデータファイル名' TO '変更後のデータファイル名';
ALTER DATABASE RENAME FILE '/opt/app/oracle/product/12.1.0/dbhome_1/test1/TESTUSER.dba'  TO '/opt/app/oracle/product/12.1.0/dbhome_1/test2/TESTUSER.dba' ;

--手動拡張
alter database datafile '/opt/app/oracle/product/12.1.0/dbhome_1/test1/TESTUSER.dba'  resize 30G;

--自動拡張OFF
alter database datafile '/opt/app/oracle/product/12.1.0/dbhome_1/test1/TESTUSER.dba'  autoextend off;
```

NOTE: 同一インスタンス内の別スキーマのパッケージを使用する際は、そいつの実行権限を実行するスキーマが持ってないとエラーになる
