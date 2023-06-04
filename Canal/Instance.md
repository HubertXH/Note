```properties
#mysql集群配置中的serverId概念，需要保证和当前mysql集群中id唯一 (v1.1.x版本之后canal会自动生成，不需要手工指定)
canal.instance.mysql.slaveId=0
# 是否启用mysql gtid的订阅模式 默认false
canal.instance.gtidon=false
```

#### position info
```properties
# mysql主库链接地址	
canal.instance.master.address=127.0.0.1:3306
# mysql主库链接时起始的binlog文件	可不配置 自动获取当前位点
canal.instance.master.journal.name=
# mysql主库链接时起始的binlog偏移量	 可不配置
canal.instance.master.position=
# mysql主库链接时起始的binlog的时间戳 可不配置
canal.instance.master.timestamp=
# 是否启用mysql gtid的订阅模式 默认false	
canal.instance.master.gtid=

# mysql从库链接地址
canal.instance.standby.address = 192.168.0.1:3306
# mysql从库链接时起始的binlog文件	
canal.instance.standby.journal.name =
# mysql从库链接时起始的binlog偏移量	
canal.instance.standby.position =
# mysql从库链接时起始的binlog的时间戳
canal.instance.standby.timestamp =
# 是否启用mysql gtid的订阅模式 默认false	
canal.instance.standby.gtid=
```

#### rds oss binlog
```properties
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=
```

##### table meta tsdb info
````properties
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal
````

##### username/password
```properties
# mysql数据库帐号	
canal.instance.dbUsername=userName
# mysql数据库密码	
canal.instance.dbPassword=password
# mysql 数据解析编码	
canal.instance.connectionCharset = UTF-8/BGK
```

##### enable druid Decrypt database password
```properties
canal.instance.enableDruid=false
# mysql 默认schema
canal.instance.defaultDatabaseName=databaseName
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==
```
##### tableConfig
````properties
#  mysql 数据解析关注的表，Perl正则表达式.多个正则之间以逗号(,)分隔，转义符需要双斜杠
canal.instance.filter.regex=schema1.tableName1

# mysql 数据解析表的黑名单，表达式规则见白名单的规则
canal.instance.filter.black.regex=

# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
# mysql 监听表的字段
canal.instance.filter.field=databaseName.tableName:field1/field2

# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
# 监听表字段的黑名单
canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch
````

##### mq config

```properties
# mq config mq里的topic名	
canal.mq.topic=myCanalTopic
# mq里的动态topic规则, 1.1.3版本支持	
canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
# 单队列模式的分区下标，
canal.mq.partition=
# 散列模式的分区数	
canal.mq.partitionsNum=24
# 散列规则定义
canal.mq.partitionHash=schema1.tableName1:field1
# mq里的动态队列分区数,比如针对不同topic配置不同partitionsNum	
canal.mq.dynamicTopicPartitionNum=test.*:4,mycanal:6
```

