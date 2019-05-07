# cassandra
## keyspace操作
1. 查看所有keyspace属性--> select * from system.schema_keyspaces;
2. 创建keyspace，有两个属性，REPLICATION 和 DURABLE_WRITES，--> CREATE KEYSPACE “KeySpace Name” WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’} AND durable_writes = ‘Boolean value’;
3. 修改keyspace--> ALTER KEYSPACE “KeySpace Name” WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’};
4. 删除keyspace --> drop keyspace test;
## table操作
1. 
## CAP理论
1. C(consistency 一致性)
    所有的客户端使用同样查询都可以得到同样的结果，即使并发很高
2. A(availability 可用性)
    所有的数据库客户端总是可以读写数据
3. P(partition tolerance 分区容错性)
    数据库可以分散到多台机器，即使发生网络故障，被分成多个额区，依然可以提供服务

