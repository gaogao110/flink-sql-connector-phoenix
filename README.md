# flink-sql-connector-phoenix

## 简介
flink-sql-connector-phoenix 可以使用flink sql 读写phoenix
## 特点


## 原理
基于flink-sql-connector-jdbc改造而成

##使用方式
--创建flinksql  phoenix表

`CREATE TABLE pv (
     sid INT,
     ucount BIGINT,
     PRIMARY KEY (sid) NOT ENFORCED
 ) WITH (
   'connector.type' = 'phoenix', 
   'connector.url' = 'jdbc:phoenix:xxxxxx:2181', 
   'connector.table' = 'TEST.PV',  
   'connector.driver' = 'org.apache.phoenix.jdbc.PhoenixDriver', 
   'connector.username' = '', 
   'connector.password' = '',
   'phoenix.schema.isnamespacemappingenabled' = 'true',
   'phoenix.schema.mapsystemtablestonamespace' = 'true',
   'connector.write.flush.max-rows' = '1'
  );`
  
参数解释：

phoenix-connector中拓展了
‘phoenix.schema.isnamespacemappingenabled’ = ‘true’,
‘phoenix.schema.mapsystemtablestonamespace’ = ‘true’
两个参数 用于连接开启schema配置的phoenix ，如果未开启则设置为false。
‘connector.write.flush.max-rows’ 参数为写入的数据批次条数，如果想提升写入phoenix性能可以设置较大。

##flink sql connector phoenix的使用样例
--设置flink参数

` SET 'table.exec.mini-batch.enabled' = 'true';
 SET 'table.exec.mini-batch.allow-latency' = '5 s';
 SET 'table.exec.mini-batch.size' = '5000';
 SET 'table.optimizer.agg-phase-strategy' = 'TWO_PHASE';
 `
 --模拟数据源
 
` CREATE TABLE datagen (
  userid int,
  proctime as PROCTIME()
 ) WITH (
  'connector' = 'datagen',
  'rows-per-second'='10',
  'fields.userid.kind'='random',
  'fields.userid.min'='1',
  'fields.userid.max'='100'
 );`
 
 --创建mysql lookup维表
 
` CREATE TABLE student (
     sid INT,
     name STRING,
     PRIMARY KEY (sid) NOT ENFORCED
 ) WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://xxxx:3306/test',
    'table-name' = 'student',
    'username' = 'xxxx',
    'password' = 'xxxxx',
    'lookup.max-retries' = '1',
    'lookup.cache.max-rows' = '1000',
    'lookup.cache.ttl' = '60s' 
 );`
 
 --创建 phoenix pv表
 
` CREATE TABLE pv (
     sid INT,
     ucount BIGINT,
     PRIMARY KEY (sid) NOT ENFORCED
 ) WITH (
   'connector.type' = 'phoenix', 
   'connector.url' = 'jdbc:phoenix:xxxxx:2181', 
   'connector.table' = 'TEST.PV',  
   'connector.driver' = 'org.apache.phoenix.jdbc.PhoenixDriver', 
   'connector.username' = '', 
   'connector.password' = '',
   'phoenix.schema.isnamespacemappingenabled' = 'true',
   'phoenix.schema.mapsystemtablestonamespace' = 'true',
   'connector.write.flush.max-rows' = '30'
  );`
 
 --写入phoenix中
 
` insert into pv select student.sid as sid ,count(student.sid) as ucount from datagen left join student FOR SYSTEM_TIME AS OF datagen.proctime on student.sid = datagen.userid group by student.sid having student.sid is not null;
`
