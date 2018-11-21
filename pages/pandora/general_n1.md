#搭建应用或者业务日志采集与分析

在开始场景实践之前，请确保您已经完整体验了我们的[快速开始](prepare_the_data.md)下所有内容，对我们的产品有一个基础的了解。

### 1.下载并运行logkit
``` 
|-------|------|
Logkit获取方式：
sftp -P 2222 share01@12.16.7.178
centos使用：sftp -oPort=2222 share01@12.16.7.178  访问
账号:share01   
密码:1q2w3e4r
ls可以看到不同版本的logkit,根据不同的操作系统通过：
get logkit1.4.8_****.tar.gz
下载不同的logkit进行数据采集
``` 

将下载的logkit解压后，在命令行中将路径指向logkit文件夹；

输入 `nohup ./logkit -f logkit.conf 2>&1 &`

!> 注意：如需修改端口或其他信息，请参考logkit产品详解。

在浏览器输入 `EC2内网ip:8081` 访问logkit主页。

![](http://docs.qiniucdn.com/data_collection_logkit_4.png)

### 2.配置logkit日志采集器

点击增加按钮，添加一个日志采集器；

第一步：填写下面的内容，然后点击下一步；

```
数据源类型：选择从文件读取（file模式）
日志文件夹路径：填写您的业务日志所在路径
注意：
如果日志格式一样，并且分布在多个文件夹，并且一个文件夹下有多个日志文件，比如/var/logs/文件夹下面有多个应用的日志文件夹，文件夹下的日志都是XXX.log的形式存放，那可以使用通配符的方式，“通配符的日志切记一定要使用从文件读取（tailx模式），/var/logs/*/*.log
```

![](http://p59ej0qjj.bkt.clouddn.com/datasource.jpg)

第二步：按照下面的讲解填写内容，然后下一步；
比如我们拿其中一种类型的应用日志：

```
2018-02-22 05:13:52,386 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:439) - Artifactory application context set to NOT READY by refresh
2018-02-22 05:13:52,387 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:219) - Refreshing artifactory: startup date [Thu Feb 22 05:13:52 EST 2018]; root of context hierarchy
2018-02-22 05:13:56,984 [art-init] [INFO ] (o.a.s.d.DbServiceImpl:218) - Connection URL: jdbc:mysql://cmbartifactory.chwyqybqnywr.rds.cn-north-1.amazonaws.com.cn:3306/cmbartifactory?characterEncoding=UTF-8&elideSetAutoCommits=true
```
分析日志我们可以发现，一共有三条日志日志数据有些为一行，有些为多行，但是每条日志按照句首的时间可以做切分，那么我们在logkit "配置数据源" 页面*head_pattern*填写匹配`2018-02-22 05:13:52,386`的正则表达式`^(\S+-\S+-\S+ \S+:\S+:\S+)`来做日志条目的切分，在匹配日志的字段时候，grok表达式(grok_patterns)中填写：`%{MYLOG}`
在自定义pattern (grok_custom_patterns)中填写：

```
MYLOG %{DATA:timestamp:date} \[%{GREEDYDATA:module}\] \[%{WORD:level} \] \(%{GREEDYDATA:logout}\) - %{GREEDYDATA:detaillog}
```
![](http://p59ej0qjj.bkt.clouddn.com/peizhigrok.png)
![](http://p59ej0qjj.bkt.clouddn.com/grok2.png)
有关 grok & pattern 的介绍，请参见：
`https://github.com/qiniu/logkit/wiki/Grok-Parser`

可以借助Grok这个工具调试：
`http://grokdebug.herokuapp.com/`



第三步：直接跳过，下一步；

第四步：按照下面的讲解填写内容，然后下一步；

填写账户的公钥\私钥、仓库名称,然后定义内网发送地址即可；

![](http://p5bjfbphc.bkt.clouddn.com/fasong.png)

![](http://p5bjfbphc.bkt.clouddn.com/fasong2.png)

第五步：确认并提交

### 3.在控制台就可以查看对应的数据
访问：
`http://12.16.5.148:8080/logdb`

![](http://p59ej0qjj.bkt.clouddn.com/logdb.png)

### 4.展示数据

我们准备好了一个nginx监控的dashboard模板，点击右侧下载 json http://op26gaeek.bkt.clouddn.com/logdbgrafana.json

在`http://12.16.5.148:8090/login`访问Grafana；

进入Grafana之后，首先创建一个数据源；

![](http://docs.qiniucdn.com/nginx42.png)

在创建数据源时，请按照注意进行配置，注意User和Password填写账户的公钥和私钥，同时Index name中的中文替换成日志分析服务的仓库名称，若您所有配置都是根据本教程填写的，那么直接在框中填写 `[my_work-]YYYY.MM.DD`即可。

![](http://docs.qiniucdn.com/nginx44.png)
`	
注意： 
`
`
	1. url 必须填入 http://12.16.5.148:8999/logdb
`
`
	2. index 名字填写时候必须选择Daily，同时[repoName-]YYYY.MM.DD，用实际的数据仓库名字替换repoName。注意中括号内不能包含任何诸如空格、制表位等空白和特殊字符。
`
`
	3. Time field name 是指定数据仓库中的时间字段，默认值timestamp，必须替换为实际的字段名字，要注意字段不能带@符号，否则将无法生效。
`
`
	4. ES 版本固定选择2.x `
`	

绘制对应的grafana图表，具体可以参考：
https://qiniu.github.io/pandora-docs/#/grafana

### 5.配置报警信息

配置好监控dashboard之后，我们可以配置报警信息，进入notification页面，进行报警配置；

![](http://docs.qiniucdn.com/nginx59.png)

以邮件报警为例，按下图进行配置即可；

![](http://docs.qiniucdn.com/nginx60.png)

配置好接警方式以及接警人后，我们对需要报警提示的图表进行报警设置，以下图为例，设置了响应时间大于1000ms的报警；

![](http://docs.qiniucdn.com/nginx70.png)

最终的报警内容如下图所示：

第一张是右键报警，第二张是slack报警。

![](http://docs.qiniucdn.com/nginx50.png)

![](http://docs.qiniucdn.com/nginx51.png)
