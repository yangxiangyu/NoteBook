#mongodb迁移到TokuMX

##查看oplog的时间戳
	rs0:PRIMARY> rs.status()
	{
		"set" : "rs0",
		"date" : ISODate("2014-10-08T11:52:42Z"),
		"myState" : 1,
		"members" : [
			{
				"_id" : 7,
				"name" : "z004.kmtongji.com:8140",
				"health" : 1,
				"state" : 1,
				"stateStr" : "PRIMARY",
				"uptime" : 17668,
				"optime" : Timestamp(1412769162, 21),
				"optimeDate" : ISODate("2014-10-08T11:52:42Z"),
				"electionTime" : Timestamp(1412751541, 1),
				"electionDate" : ISODate("2014-10-08T06:59:01Z"),
				"self" : true
			},
			{
				"_id" : 8,
				"name" : "z005.kmtongji.com:8140",
				"health" : 1,
				"state" : 2,
				"stateStr" : "SECONDARY",
				"uptime" : 2331,
				"optime" : Timestamp(1412769161, 3),
				"optimeDate" : ISODate("2014-10-08T11:52:41Z"),
				"lastHeartbeat" : ISODate("2014-10-08T11:52:41Z"),
				"lastHeartbeatRecv" : ISODate("2014-10-08T11:52:42Z"),
				"pingMs" : 0,
				"syncingTo" : "z004.kmtongji.com:8140"
			}
		],
		"ok" : 1
	}


##dump mongodb数据库
>mongodump -h localhost:8140 \ 
-d wom_production \
-u mongodb -p kmsocialmongodb2012 \
-o  wom_production

##恢复mongodb
>mongorestore -d <数据库名>  --drop <备份数据的目录>

##其他操作
1. 强制关闭mongodb
  > db.shutdownServer({fource:true})

 2. 修改复制集的主机名（解决开始其他节点无法初始化）
> cfg = rs.conf()
cfg.members[1].host = "mongodb1.example.net:27017"
rs.reconfig(cfg)

3. 添加rs复制集
>rs.add()

4. 指定认证数据库
>--authenticationDatabase <DBname>