#### canal 修改position

canal会自动刷新position信息到ZK中

##### canalZK的存储路劲为：

/otter/canal/destinations/${instance_name}/1001/cursor

##### 存储内容：

{"@type":"com.alibaba.otter.canal.protocol.position.LogPosition","identity":{"slaveId":-1,"sourceAddress":{"address":"dbaddress","port":3306}},"postion":{"gtid":"","included":false,"journalName":"mysql-bin.033017","position":162831532,"serverId":12136,"timestamp":1675914626000}}
需要修改的信息有：

- journalName
- position

##### 从mysql上获取position信息：

show master status 获取当前运行的position
show binary logs 查看有哪些binlog
show binlog events in "mysql-bin.033017"  查看具体文件中的position

替换需要修改的信息，使用set path "" 直接修改信息。重启动canal即可。

