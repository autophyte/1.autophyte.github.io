## ApacheJmeter��Element���ܣ�������Thread Group

[TestPlan]:https://autophyte.github.io/2018/11/08/ApacheJmeter%E5%85%83%E4%BB%B6%E4%BB%8B%E7%BB%8D.html


### Thread Group���߳��飩����
1. Thread Group�ǲ��Ե���㣬Ҳ����˵��[TestPlan]�������Ҫ��Thread Group�ſ��Կ�ʼ���ԡ�   
2. Thread Group�����õ�Thread��������Ҳ����Jmeterģ����û����������ԣ�Thread Group���Ա������������û��ء�   
8. Thread Group��ÿһ��Thread��ȫ�����������߳�����������в��ԡ�����߳�ģ�Ⲣ�����ʡ�   
3. ��������Ҫ��controller��sampler���������Thread Group�¡�   
4. �����ģ���Listener����ֱ�ӷ���[TestPlan]�£�listener����Test Plan�ºͷ���Thread Group��ʱ�������ڲ�ͬ��   
6. �Ҽ����[TestPlan]->Add->Threads(Users)->Thread Group�����Ը�[TestPlan]���һ��Thread Group     
![���ThreadGroup](/img/jmeter/JmeterElements_ThreadGroup_add.png)   
7. ÿһ��[TestPlan]�¿��ԷŶ��Thread Group����ЩThread Group��ִ��˳�������[TestPlan]�п���   


### Thread Group����

1. Numbers of Threads(users): �����߳�����Ҫ�������̵߳�������Ҳ������Ҫģ����û���������
2. Ramp-Up Period(in seconds): ����ѹ������ʱ�䣨�������ɣ��೤ʱ���Ժ������߳̿�ʼȫ����ѹ��
	+ ��һ����ָNumbers of Threads���趨�������̣߳��ڶ೤ʱ����ȫ����ʼ����������ѹ
	+ ����趨��Ҫ����100���̣߳���Ramp-Up Period�趨Ϊ20����ô��ζ�ţ�ÿ������5���߳̿�ʼ����
	+ Ramp-Up Period��Ҫ���õ��㹻���Ա�֤�ܹ����������߳���ȫ��ʼ������ͬʱ�������õ��㹻С����ȷ�����һ���߳̿�ʼ����ʱ����һ���̵߳Ĺ�����û����ȫ����
	+ ��ģ����û�������ʱ�����Ramp-Up Period���ù�С����ʱ���ڻ���������ϴ��ѹ�����п��ܻᵼ�·��������ء�
3. Loop Count������**ÿһ��**�̵߳�ѭ������
	+ ��ѡForverʱ������һ����ѭ������
	+ ע����ǣ�������õ���ÿһ���̵߳�ѭ�������������ܵ�requests��������ǣ����߳����� * ѭ������ * ÿ���߳�������������
4. Delay Thread creation until needed����ѡ���ѡ���Ժ�ֻ�����߳���Ҫ��ʼ������ʱ�򣬲ŻῪ���߳�
	+ �������ѡ���ѡ���ô�߳���������ʱ�򣬾ͻὫ������Ҫ���߳�ȫ�����������Ƿ��ȫ����ʼ��������Ҫ��Ramp-Up Period�����ã�
	+ ��ѡ���ѡ���Ϊ�ӻ����̴߳���ʱ�䣬���ԣ��ڿ�ʼ��ʱ����Դռ������С��δ��ѡʱ
	+ ��ѡ���ѡ���Ժ��ں�̨���Կ���������ʱ������ƣ�jmeter���߳��������������ӵģ�������ѡʱ����������ʱ����̨���Ѿ����ڴ���jmeter�̡߳�
	+ ���ѡ��Ĭ����δ��ѡ��
5. Scheduler��Jmeter���ṩ��һ�����������������������߳��ڲ��Կ�ʼ���ӳٶ೤ʱ�������������೤ʱ��
6. Thread Group�У�Ҳ�������õ�ȡ��������ʱ�Ķ�����Ĭ��Ϊ��������
	+ Continue�������������ִ�С��̻߳Ὣ�����Ķ���ִ����ɣ��������Ķ�����������ǰ����Ķ���������Ĳ�����Ȼ������ģ�
	+ Start Next Thread Loop���̻߳���Ժ���������֮�ʿ�ʼ��һ��ѭ��
	+ Stop Trhead��ֱ��ֹͣ��ǰ�߳�
	+ Stop Test����ִ������������Ժ󣬽������ԡ���ζ���̶߳����б��У�����������Ķ�����������ֱ����ǰѭ����ɣ��ٽ�������
	+ Stop Test Now������ֹͣ���ԣ�����Ķ�������Ķ���Ҳ��Ҫ����

![ThreadGroup����](/img/jmeter/JmeterElements_TestPlan_config.png)
