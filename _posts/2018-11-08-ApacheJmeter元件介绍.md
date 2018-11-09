## ApacheJmeter各Element介绍（一）：TestPlan

### TestPlan介绍
TestPlan是Jmeter测试用例的“根”，我们所有的测试过程都在TestPlan中进行，每一个TestPlan可以看作是一个“项目”，其它所有element都必须在TestPlan下。Jmeter启动时，我们所看到的就是TestPlan。   
![TestPlan](/img/jmeter/JmeterElements_TestPlan.png "TestPlan")   

* TestPlan中有一个选项“Functional Test Mode”，这个选项默认时未被选中的。
	+ 如果是作功能测试，或者进行调试的话，可以将这个选项打开。
	+ 当这个选项被选中的时候，Jmeter会记录所有request请求值等数据（需要记录哪些数据，可以在listener人Configuration中设置），这些数据会被写入到你的listener中设置的文件中。
	+ 这个功能被打开是会严重影响性能，且随着requests数量增加，日志占用空间会骤增，所以，在做性能测试，压力测试以及负载测试的时候，最好不要打开这个选项
	+ 如果在listener中没有设置存储的文件的话，即便勾选了这个选项，这个选项也是不起作用的   

![TestPlan](/img/jmeter/JmeterElements_TestPlan_checkbox.png "TestPlan")      
* TestPlan中可以添加多个线程组
	+ 在TestPlan中如果没有勾选“Run Thread Groups consecutively”（默认未勾选），所有线程组是并行执行的。
	+ 如果期望一个Thread Group执行完成后，再执行另一个，确保已经勾选了这个选项。
	+ 如果一个TestPlan中只有一个线程组，可以忽略这条   
* TestPlan中间还有一个区域是用户自定义变量区域，在这里可以定义一些需要的变量，这些变量都是全局变量（对当前TestPlan来说）   

![TestPlan](/img/jmeter/JmeterElements_TestPlan_variable.png "TestPlan")     
* 如果有额外的一些jar库需要导入到测试中，在TestPlan下面有一个“Add directory or jar to classpath”的功能，可以在这里添加所需要的额外的包
