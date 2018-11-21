#使用 logkit + 日志分析服务 + Grafana 10分钟搭建Nginx日志分析和实时监控系统。

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
数据源类型：选择从文件读取（fileauto模式）
日志文件夹路径：填写您的nginx日志所在路径
```

![](http://docs.qiniucdn.com/nginx1.png)

第二步：按照下面的讲解填写内容，然后下一步；

在匹配日志的grok表达式(grok_patterns)中填写：`%{NGINX_LOG}`

在自定义 pattern (grok_custom_patterns)中填写：

```
NGINX_LOG %{NOTSPACE:client_ip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:ts:date}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version:float})?|%{DATA})" %{NUMBER:resp_code} (?:%{NUMBER:resp_bytes:long}|-) (?:%{NUMBER:resp_body_bytes:long}|-) "(?:%{NOTSPACE:referrer}|-)" %{QUOTEDSTRING:agent} %{QUOTEDSTRING:forward_for} %{NOTSPACE:upstream_addr} (%{HOSTNAME:host}|-) (%{NOTSPACE:reqid}) %{NUMBER:resp_time:float}
```

有关 grok & pattern 的介绍，请参见：https://github.com/qiniu/logkit/wiki/Grok-Parser


![](http://docs.qiniucdn.com/nginx2.png)

第三步：直接跳过，下一步；

第四步：按照下面的讲解填写内容，然后下一步；

默认只需要填写账户的公钥和私钥即可；

![](http://docs.qiniucdn.com/nginx3.png)

第五步：确认并提交


### 3.新建并配置Grafana监控图表

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

现在将我们刚刚下载的json文件导入即可；

![](http://docs.qiniucdn.com/nginx45.png)

导入完成后，即可看到我们的实时nginx监控图；

![](http://docs.qiniucdn.com/nginx49.png)


### 4.配置报警信息

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
