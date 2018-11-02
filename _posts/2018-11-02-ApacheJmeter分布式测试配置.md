## Jmeter分布式测试
搭建Jmeter完成分布式测试，需要机器满足下面几个条件：
+ 确保客户端（提供负载的机器）可以正常连接服务器（防火墙/网段设置等）
+ 确保所有客户端的java/jemeter版本相同。版本不同，有可能会出错
+ Jmeter RMI默认使用动态端口，所以Slave上如果有防火墙，最好将端口号设置为固定端口
+ Jmeter 4.0以后RMI默认使用SSL传输，需要配置或者禁用SSL
![主从模型](/img/distributed.png "主从模型：原于jmeter管网")

### 名词解释
+ Master：控制其他测试机器的机器。用户之际操作的机器。在master上以GUI模式运行Jmeter，一般不直提供负载
+ Slave：远程机器，提供负载的机器，运行jmeter-server。可以有多个。和master通过RMI通讯
+ Target：测试目标机器
![名词解释](/img/distributed_names.png "名词解释：原于jmeter管网")

### 配置
+ 禁用RMI的SSL，修改Slave机器上“jmeter\bin\user.prorerties”文件

	```
	# user.prorerties
	server.rmi.ssl.disable=true
	```

+ 让Slave机器RMI使用固定端口，修改修改Slave机器上jmeter\bin\user.prorerties文件
	```bash
	# user.prorerties
	server.rmi.localport=44242
	```

+ 配置RMI SSL
	- 运行Master上面“jemter\bin\create-rmi-keystore.bat”(linux上create-rmi-keystore.sh)
	- 回复一些问题（可以使用默认值）
	- 脚本会在Master的“jemter\bin\”下面生成一个key文件：rmi_keystore.jks
	- 将这个文件复制到所有Slave机的“jemter\bin\”目录下

+ 在Master上配置远程机器，修改“jmeter\bin\user.prorerties”文件，
	```ini
	# 添加下面这一行,ip地址为Slave机器ip地址,如果有多台,用英文逗号分隔
	remote_hosts=172.16.3.138
	```

+ 多个网卡配置   
	有多个网卡时，必须指定网卡，否则会很可能会出现RMI通讯时并不是你期望的网卡   

	- windows下修改jmeter\bin\jmeter.bat
	```
	# 172.16.2.39为需要使绑定的ip地址
	set rmi_host=-Djava.rmi.server.hostname=172.16.2.39
	# 在ARGS后面追加%rmi_host%
	set ARGS=%ARGS% %rmi_host%
	```   

	- linux服务器下修改jmeter/bin/jmeter_server
	```bash
	# 去掉下面这一行前#符号，并修改为自已想要的ip地址
	RMI_HOST_DEF=-Djava.rmi.server.hostname=xxx.xxx.xxx.xxx
	```

