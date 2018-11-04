## ApacheJmeter5.0 录制脚本

Jmeter脚本录制，可以：
1. 用badboy录制后导出
2. 用jmeter自已直接录制
3. 通过BlazeMeter录制后导出。   

本篇只讲通过Jmeter自已录制。ApacheJmeter5.0 版本的录制方式比以前版本简单很多，本篇录制基于5.0版本。

### 录制过程
1. 以GUI模式启动Jmeter
2. 点击工具栏上模版按钮，打开模版选择页。     
	![模版按钮](/img/jmeter/templates_button.png "模版按钮")   
3. 选择Recording模版，用录制模版在新工程创建一个录制项目.   
	![模版页](/img/jmeter/templates.png "模版按钮")
4. 测试计划会变成这样子。   
	![模版计划](/img/jmeter/Jietu20181104-template_in_tesplan.jpg "录制脚本")
5.  然后在“HTTP Request Defaults”元件中填入网址（或者IP地址），路径等信息。   
	![填入基本信息](/img/jmeter/Jietu20181104-recording_set_parameter.jpg "http基本信息")
6. 如果“HTTP(S) Test Script Recorder”元件是被禁用的，需要启用.   
	![启用元件](/img/jmeter/Jietu20181104-enable_element.png "http基本信息")
7. 确认“HTTP(S) Test Script Recorder”中配置是不可用，主要看端口号是否有其他程户占用，开启代理服务器。   
	![开启代理服务器](/img/jmeter/Jietu20181104-record_proxy.png "开启代理服务器")
8. 点击启动后会生成一个证书（bin/ApacheJMeterTemporaryRootCA.crt），弹出一个对话框告诉我们证书生成了。   
	![证书](/img/jmeter/Jietu20181104-ca.png "开启代理服务器")
9. 在浏览器中安装这个证，以Firefox为例
	+ 在地址栏中输入“about:preferences#privacy”回车
	![证书](/img/jmeter/Jietu20181104-open_ca_manager.png "Firefox安装证书")
	+ 在打开的设置页面中，Certificates下面，点击“View Certificates”
	+ 点击import安装上刚才的证书   
	![证书](/img/jmeter/Jietu20181104-import_ca.png "Firefox安装证书")
	+ 确认安装   
	![证书](/img/jmeter/Jietu20181104-assert_install.png "Firefox安装证书")
10. 在浏览器中设置代理服务器为Jmeter
	+ 在地址栏中输入“about:preferences”回车
	+ 搜索“proxy”
	![代理](/img/jmeter/Jietu20181104-firefox_proxy.png "Firefox设置代理")
	+ 在弹出的框中设置IP、端口号，和Jmeter中想同即可。   
	![代理](/img/jmeter/Jietu20181104-firefox_set_prox.png "Firefox设置代理")
11. 浏览器中打开需要录制的网站，进行操作，jmeter会记录下所有的操作（step7中已开启代理服务器）
	![录制](/img/jmeter/Jietu20181104-firefox_recording.png "Firefox录制")
12. 录制动作完成后，点击“停止”按钮结束录制
13. 完成后，可以在Recing Controller下面查看录制结果，如果有不需要动作，也可以删除或修改。   
	![录制](/img/jmeter/Jietu20181104-recording_result.png "Firefox录制")


