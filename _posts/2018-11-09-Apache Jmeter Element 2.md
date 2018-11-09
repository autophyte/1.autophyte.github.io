## ApacheJmeter各Element介绍（二）：Thread Group

[TestPlan]:https://autophyte.github.io/2018/11/08/ApacheJmeter%E5%85%83%E4%BB%B6%E4%BB%8B%E7%BB%8D.html


### Thread Group（线程组）介绍
1. Thread Group是测试的起点，也就是说，[TestPlan]下面必须要有Thread Group才可以开始测试。   
2. Thread Group中设置的Thread的数量，也就是Jmeter模拟的用户数量，所以，Thread Group可以被看成是虚拟用户池。   
8. Thread Group中每一个Thread完全独立的运行线程组下面的所有测试。多个线程模拟并发访问。   
3. 测试所需要的controller和sampler都必须放在Thread Group下。   
4. 其他的，如Listener可以直接放在[TestPlan]下（listener放在Test Plan下和放在Thread Group下时，作用于不同）   
6. 右键点击[TestPlan]->Add->Threads(Users)->Thread Group，可以给[TestPlan]添加一个Thread Group     
![添加ThreadGroup](/img/jmeter/JmeterElements_ThreadGroup_add.png)   
7. 每一个[TestPlan]下可以放多个Thread Group，这些Thread Group的执行顺序可以在[TestPlan]中控制   


### Thread Group配置

1. Numbers of Threads(users): 设置线程组需要开启的线程的数量（也就是需要模拟的用户的数量）
![ThreadGroup配置](/img/jmeter/JmeterElements_TestPlan_config.png)
2. Ramp-Up Period(in seconds): 增加压力持续时间（可以理解成，多长时间以后所有线程开始全力加压）
	+ 这一条是指Numbers of Threads中设定的所有线程，在多长时间内全部开始给服务器加压
	+ 如果设定需要启动100个线程，而Ramp-Up Period设定为20，那么意味着，每秒钟有5个线程开始工作
	+ Ramp-Up Period需要设置的足够大，以保证能够将大量的线程完全开始工作，同时必须设置的足够小，以确保最后一个线程开始工作时，第一个线程的工作还没有完全做完
	+ 在模拟大用户量测试时，如果Ramp-Up Period设置过小，短时间内会给服务器较大的压力，有可能会导致服务器过载。
3. Loop Count：设置**每一个**线程的循环次数
	+ 勾选Forver时，启动一个死循环测试
	+ 注意的是，这个设置的是每一个线程的循环次数，所以总的requests数量因该是：“线程总数 * 循环次数 * 每个线程组请求数量”
4. Delay Thread creation until needed：勾选这个选项以后，只有在线程需要开始工作的时候，才会开启线程
![ThreadGroup配置](/img/jmeter/JmeterElements_TestPlan_config_delay.png)
	+ 如果不勾选这个选项，那么线程组启动的时候，就会将所有需要的线程全部创建（但是否给全部开始工作，需要看Ramp-Up Period的设置）
	+ 勾选这个选项，因为延缓了线程创建时间，所以，在开始的时候资源占用明显小于未勾选时
	+ 勾选这个选项以后，在后台可以看到，随着时间的推移，jmeter的线程数量呈线性增加的，而不勾选时，测试启动时，后台中已经存在大量jmeter线程。
	+ 这个选项默认是未勾选的
5. Scheduler：Jmeter还提供了一个调度器，可以用来控制线程在测试开始后，延迟多长时间启动，持续多长时间
![ThreadGroup配置](/img/jmeter/JmeterElements_TestPlan_config_schedule.png)
6. Thread Group中，也可以配置当取样器出错时的动作，默认为继续测试
![ThreadGroup配置](/img/jmeter/JmeterElements_TestPlan_config_onerror.png)
	+ Continue：如果出错，继续执行。线程会将后续的动作执行完成（如果后面的动作不依赖当前出错的动作，后面的测试仍然有意义的）
	+ Start Next Thread Loop：线程会忽略后续动作，之际开始下一个循环
	+ Stop Trhead：直接停止当前线程
	+ Stop Test：在执行完后续动作以后，结束测试。意味着线程动作列表中，错误动作后面的动作还会做，直到当前循环完成，再结束测试
	+ Stop Test Now：立刻停止测试，错误的动作后面的动作也不要做了
