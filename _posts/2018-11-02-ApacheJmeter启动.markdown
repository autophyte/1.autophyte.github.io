### 启动
Jmeter有3种启动方式：GUI模式，非GUI模式，服务器模式

+ GUI模式
	- 多用于编写/修改测试计划，debug测试计划，或者查看报告，不要用于实际测试，因为其会消耗更多的资源
	- windows下运行Jmeter\bin\jmeter.bat命令启动GUI模式
	- linux运行Jmeter/bin/jmeter.sh命令
+ 非GUI模式
	- 测试时推荐使用非GUI模式，资源占用率最小
	- 运行下面命令

	```bash
	# -n 使用非GUI模式式启动jmeter
	# -t 指定需要运行的测试计划的脚本，后面需紧根脚本名，如testplan.jmx
	# -l 指定生成的日志（产生报告需要用到的日志文件）名称，后面需要接日志文件名，如testresult.jtl
	jmeter -n -t testplan.jmx -l testresult.jtl
	```

	- 生成HTML报告，使用下面的命令

	```bash
	jmeter -g testresult.jtl -e -o reportPath
	```
+ 服务器模式   
很多时候我们受负载机性能限制，负载机没法提供足够的压力，这时候我们需要使用多台机器同时给目标机提供压力，这时就需要用到服务器模式启动
	+ 在Slave机器上启动Jmeter/bin/jmeter-server
	+ 在Master机器上启动Jmeter\bin\jmeter.bat
	+ 在Master机器，JmeterGUI中，Run->Remote Start菜单中会列出配置的所有远程机器（Slave机器）