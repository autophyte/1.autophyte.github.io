### 启动
Jmeter测试时有3种启动方式：GUI模式，非GUI模式，服务器模式。另外还有1种使用现有jtl文件（csv文件）专门用来生成报告的模式

+ GUI模式
	- 多用于编写/修改测试计划，debug测试计划，或者查看报告，不要用于实际测试，因为其会消耗更多的资源
	- windows下运行Jmeter\bin\jmeter.bat命令启动GUI模式
	- linux运行Jmeter/bin/jmeter.sh命令
	![GUI模式截图](/img/jmeter/jmeter_gui.png "Jmeter5.0 GUI模式")

+ 非GUI模式
	- 测试时推荐使用非GUI模式，资源占用率最小
	- 运行下面命令

	```bash
	# -n 使用非GUI模式式启动jmeter
	# -t 指定需要运行的测试计划的脚本，后面需紧根脚本名，如testplan.jmx
	# -l 指定生成的日志（产生报告需要用到的日志文件）名称，后面需要接日志文件名，如testresult.jtl
	jmeter -n -t testplan.jmx -l testresult.jtl
	```
	![非GUI模式截图](/img/jmeter/jmeter_non_gui.png "Jmeter5.0 非GUI模式")

+ 服务器模式   
很多时候我们受负载机性能限制，负载机没法提供足够的压力，这时候我们需要使用多台机器同时给目标机提供压力，这时就需要用到服务器模式启动
	+ 在Slave机器上启动Jmeter/bin/jmeter-server
	+ 在Master机器上启动Jmeter\bin\jmeter.bat
	+ 在Master机器，JmeterGUI中，Run->Remote Start菜单中会列出配置的所有远程机器（Slave机器）
	+ 配置服务器模式，可以参照商议篇：[Apachejmeter分布式测试配置](https://autophyte.github.io/2018/11/02/ApacheJmeter%E5%88%86%E5%B8%83%E5%BC%8F%E6%B5%8B%E8%AF%95%E9%85%8D%E7%BD%AE.html)   

	![服务器模式截图](/img/jmeter/start_jmeter_server.png "Jmeter5.0 服务器模式")

+ 生成HTML报告
	- 生成HTML报告，使用下面的命令
	```bash
	# -g 指定使用现有的结果文件（testresult.jtl），参数后面紧跟结果文件名称
	# -o 指定存放报告的目录，目录可以不存在。如果目录存在，目录必须时空目录
	jmeter -g testresult.jtl -o reportPath
	```