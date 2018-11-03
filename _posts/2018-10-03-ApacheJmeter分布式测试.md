## Jmeter分布式测试（二）

### 开始分布式测试
前面的文章，我们已经完成Jmeter分布式测试相关配置，现在准备开始做分布式测试（注意，不需要将testplan手动复制到每一个slave机器上，master会自动分发）。

1. 在GUI模式中准备脚本。
2. 在所有需要用到的slave机器上启动jmeter服务器模式   
	![服务器模式截图](/img/jmeter/start_jmeter_server.png "Jmeter5.0 服务器模式")
3. 在Master中以GUI方式启动jmeter
4. Master中添加聚合报告的listener
	![Master添加聚合报告的listener](/img/jmeter/jmeter_add_aggregate_listener.png "Jmeter5.0 添加聚合报告的listener")
	![Master添加聚合报告的listener](/img/jmeter/jmeter_aggregate_report.png "Jmeter5.0 添加聚合报告的listener")

5. 在Master中启动远程机器测试（也可以直接点击Remote Start All开启已经配置的所有Slave机器）   
	![Master开启Remote Test](/img/jmeter/jemter_start_remote_test.png "Jmeter5.0 启动远程测试")

6. 在Master中聚合报告中可以看到已经又测试数据过来   
	![Master聚合报告数据](/img/jmeter/jmeter_aggregate_report_data.png "Jmeter5.0 Master聚合报告数据")

7. 在Slave机器中应该可以看到下面这样的信息，表示测试已经启动   
	![Slave测试开启](/img/jmeter/remote_startedpng.png "Jmeter5.0 测试开始")