##### Jmeter基本操作

##### 响应码汇总

| 响应码 | 解释                     |
| ------ | ------------------------ |
| 415    | 请求头的content-type有误 |

##### webservice接口测试

###### 接口介绍

```python
案例接口来源：http://ws.webxml.com.cn/WebServices/MobileCodeWS.asmx?op=getMobileCodeInfo
```

![01](/Users/coco/Documents/performance/img/01.png)

###### Jmeter请求

1、请求头

![02](/Users/coco/Documents/performance/img/02.png)

2、取样器

![03](/Users/coco/Documents/performance/img/03.png)

##### TCP取样器

```python
TCPClient classname:TCP报文格式,默认前缀：org.apache.jmeter.protocol.tcp.sampler.
	1、TCPClientImpl：普通文本传输，可设置他的编码格式（eg：json串）
  2、BinaryTCPClientImpl：十六进制报文（常用）
  3、LengthPrefixedBinaryTCPClientImpl：继承BinaryTCPClientImpl类，并在BinaryTCPClientlmpl前面增加两个字节数据长度。

Re-use connection:是否重用连接，如果选择，同一个线程执行的所有请求都会使用一个tcp连接。类似keep-live
Re-use connection+close connection:每次请求结束后关闭连接。类似短连接，一般不选择
end of line byte value:返回数据的结尾表示符，是ascii码
```

![04](/Users/coco/Documents/performance/img/04.png)

##### JDBC操作

1、驱动下载

```
https://downloads.mysql.com/archives/c-j/
```

![05](/Users/coco/Documents/performance/img/05.png)

2、加载驱动

![06](/Users/coco/Documents/performance/img/06.png)

3、配置连接

![07](/Users/coco/Documents/performance/img/07.png)

![08](/Users/coco/Documents/performance/img/08.png)

4、请求

```shell
数据库的字符类型与java的字符类型对照表：http://www.doc88.com/p-7704219368296.html
```

![09](/Users/coco/Documents/performance/img/09.png)

##### Beanshell

###### Beanshell内置变量

```
log: 写日志到控制台和jmeter.log。如log.info("xx");
vars: 操作jmeter变量
		String name = var.get("变量名"): 获取变量的值
		var.put("变量名","变量值"): 将变量值保存到变量中
prev: 获取前面sampler返回的消息
		prev.getResponseDataAsString(): 获取响应消息
		prev.getResponseCode(): 获取响应code
```

###### Beanshell调用外部源代码

```
source(xx/xx/xx.java)
Md5Util.getMd5Hex(xxxx)  # 直接类.方法调用
```

###### Beanshell调用外部jar包

```
在测试计划里导入.jar包
import com.stu.Md5Util
Md5Util.getMd5Hex(xxx)
```

###### Beanshell断言

```java
内置变量
Failure:是否失败，flase：成功；true：失败
FailureMessage:失败日志，在断言失败时显示

String code = vars.get("code");
log.info(new String(ResponseData));     // 打印出响应
log.info(SamplerData);         // 打印出请求
if (code != 10000){
	Failure = true;           // 断言
	FailureMessage = "登陆失败";
}
```

###### Beanshell写入数据到文件

```java
String message = vars.get("message");
try{
	BufferedWriter writer = new BufferedWriter(new FileWriter("/Users/coco/Documents/performance/script/token.txt",true));
	writer.write(message);
	writer.newLine();
	writer.close();
}catch(IOException e){
	e.printStackTrace();
}
```

#### 性能测试

##### jmeter运行模式

```
1、按照运行次数
2、按照运行时间（推荐）
	> 线程组设置永远循环
	> 勾选调度器，设置持续时间
```

##### 线程组和请求运行逻辑

```
一个线程组内的请求是顺序执行的
不同线程里的请求是并发执行的

如果接口之间有依赖关系，就设置在一个线程组内
```

##### 部署远程Jmeter

###### JDK安装

```shell
vim /etc/profile
# jdk
export JAVA_HOME=/usr/local/jdk1.8
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```

###### Jmeter安装

```shell
vim /etc/profile
# jmeter
# jmeter
# jmeter
export JMETER_HOME=/usr/local/jmeter5.4
export CLASSPATH=$JMETER_HOME/lib/ext/ApacheJMeter_core.jar:$JMETER_HOME/lib/jorphan.jar:$CLASSPATH
export PATH=$PATH:$JMETER_HOME/bin

source /etc/profile

验证：jmeter -v
```

###### 编写脚本传到远程,单机执行

```shell
jmeter -n -t xxx.jmx -l result.jtl
-n:命令行模式
-t:jmx脚本路径
-l:jtl结果文件保存路径
```

![10](/Users/coco/Documents/performance/img/10.png)

```shell
默认每30s刷新一次
修改刷新时间：
vim jmeter.properties
summariser.interval = 10

00:00:30代表: 00:00:17之后的30s里tps的平均值
00:00:47代表: 过去的47s的tps的平均值
AVG: 平均响应时间
Min: 最小响应时间
Max: 最大响应时间
Err: 错误率
Active: 正在活动的线程数,并发数
Start: 已经启动的线程数
Finished: 已经结束的线程数
# 注意，这里展示的是线程组里所有接口的和
```

###### 聚合报告查看结果

```
将远程的result.jtl文件传输到本地，使用聚合报告打开
```

![11](/Users/coco/Documents/performance/img/11.png)

###### html报表生成

```shell
vim reportgenerator.properties

jmeter.reportgenerator.overall_granularity=1000 # 每一秒

执行命令：
jmeter -g result.jtl -o ./output

-g:指定jtl文件路径
-o:执行html报表保存在哪里
```

重要的图表：

```
Over Time:
	Response Times Over Time 响应时间
	Active Threads Over Time 并发数
Throughput:
	Transactions Per Second 每秒传输的事物处理个数（并发数/平均响应时间）
	Hits Per Second 每秒点击数
```

###### 执行过程中生成报告

```
jmeter -e -o report.html(不推荐)
```

##### 分布式

```
1、配置hosts文件
# 注意：主压力机和从压力机都要配置
vim /etc/hosts
10.0.4.15 VM-4-15-centos
122.51.127.167 VM-4-15-centos
192.168.0.103 coco’s MacBook Air

2、jmeter部署：主压力机和从压力机都要配置

3、jmx脚本在主压力机，参数文件放在每一个压力机上，并且位置要相同。

4、修改每台压力机的jmeter.properties文件，去掉ssl.disable=true的注释

5、每台压力机进入bin目录下，都启动 nohup ./jmeter-server & 

6、在主压力机的jmeter/bin目录下修改jmeter.properties，将remote_hosts修改为主次两台压力机的IP

7、在主压力机执行  jmeter -n -t xxx.jmx -l result.jtl -r
-r：代表分布式

补充：
第6步不修改也可以，执行jmeter -n -t xxx.jmx -l result.jtl -R 远程ip
如果中途要停止，执行shoutdown.sh
```
