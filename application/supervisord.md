#supervisord

##可以实现的功能
- 保证程序意外停止时自动再次启动,保证关键进程一直在运行
- 日志大小的限制，防止std,stderr日志过大，占满硬盘
- 提供web界面，方便快速重启。


##命令
- supervisord 启动命令，如果不用 -c 指定配置文件，则默认用/etc/supervisord.conf这个配置文件
- echo_supervisord_conf  输出配置文件，默认/etc/supervisord.conf是没有的，需要用 echo_supervisord_conf> /etc/supervisord.conf来生成默认配置文件.
- supervisorctl 
	- supervisorctl help 帮助
	- supervisorctl update all  在更新完配置文件后，执行。会把配置文件中删除的进程，从supervisord中删除，配置文件中添加的进程，会在配置文件中添加。
	- supervisorctl start <名字>  启动一个进程
	- supervisorctl restart <名字> 重启一个进程
	- supervisorctl stop  <名字> 停止一个进程
## 配置文件
全部配置在/etc/supervisord.conf下面是部分配置
>
web界面配置
[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9001        ; (ip_address:port specifier, *:port for all iface)
username=<username> ; (default is no username (open server))
password=<password>              ; (default is no password (open server))


>
添加进程配置
[program:skullcandy_frontend]
command=java -Xms512M -Xmx2G -Xdebug -Xrunjdwp:transport=dt_socket,address=10020,server=y,suspend=n -jar -Xloggc:gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:-UsePerfData -XX:+PrintVMOptions -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=70 -XX:ParallelGCThreads=4 -XX:SurvivorRatio=8 -XX:CMSMarkStackSize=8M  -XX:CMSMarkStackSizeMax=32M -XX:+UseLargePages -Djava.awt.headless=true /home/kmop/skullcandy/frontend/EC-Skullcandy-0.0.1-SNAPSHOT.jar	;定义启动命令
directory=/home/kmop/skullcandy/frontend	；工作目录



## 使用说明
