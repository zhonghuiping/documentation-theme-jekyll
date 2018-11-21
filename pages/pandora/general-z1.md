#搭建应用或者业务日志采集与分析


### 1.确保机器已经安装运行logkitpro


### 2.配置logkit日志采集器

点击增加按钮，添加一个日志采集器；

第一步：填写下面的内容，然后点击下一步；

```
数据源类型：选择从文件读取（file模式）
日志文件夹路径：填写您的业务日志所在路径
注意：
如果日志格式一样，并且分布在多个文件夹，并且一个文件夹下有多个日志文件，比如/var/logs/文件夹下面有多个应用的日志文件夹，文件夹下的日志都是XXX.log的形式存放，那可以使用通配符的方式，“通配符的日志切记一定要使用从文件读取（tailx模式），/var/logs/*/*.log
```


第二步：按照下面的讲解填写内容，然后下一步；
比如我们拿其中一种类型的应用日志：

```
2018-02-22 05:13:52,386 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:439) - Artifactory application context set to NOT READY by refresh
2018-02-22 05:13:52,387 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:219) - Refreshing artifactory: startup date [Thu Feb 22 05:13:52 EST 2018]; root of context hierarchy
2018-02-22 05:13:56,984 [art-init] [INFO ] (o.a.s.d.DbServiceImpl:218) - Connection URL: jdbc:mysql://cmbartifactory.chwyqybqnywr.rds.cn-north-1.amazonaws.com.cn:3306/cmbartifactory?characterEncoding=UTF-8&elideSetAutoCommits=true
```
**分析日志我们可以发现，一共有三条日志日志数据有些为一行，有些为多行，但是每条日志按照句首的时间可以做切分，那么我们在logkit "配置数据源" 页面*head_pattern*填写匹配`2018-02-22 05:13:52,386`的正则表达式`^(\S+-\S+-\S+ \S+:\S+:\S+)`做日志条目的切分,如下图所示：**
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:logkithead.png)

*接下来对日志的每个字段进行字段解析，可以通过如下两个方案：*
*(1)、借助Logkitpro的划词功能来配置日志解析*
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:grok1.png)
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:grok2.png)

*(2)、手动配置grok解析
在匹配日志的字段时候，grok表达式(grok_patterns)中填写：`%{MYLOG}`
在自定义pattern (grok_custom_patterns)中填写：*

```
MYLOG %{DATA:timestamp:date} \[%{GREEDYDATA:module}\] \[%{WORD:level} \] \(%{GREEDYDATA:logout}\) - %{GREEDYDATA:detaillog}
```
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:zidingyigrok.png)
有关 grok & pattern 的介绍，请参见：
`https://github.com/qiniu/logkit/wiki/Grok-Parser`

可以借助Grok这个工具调试：
`http://grokdebug.herokuapp.com/`

第三步：直接跳过，下一步；

第四步：按照下面的讲解填写内容，然后下一步；

填写工作流名称、数据源名称和仓库名称,然后定义内网发送地址即可；

![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:fasong2.png)


第五步：确认并提交

### 3.在控制台登录七牛平台账号就可以查看对应的数据
访问：
`http://12.16.5.148:8080/logdb`

![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:chakantest.png)

### 4.展示数据

在grafana中使用pandora logdb之前，我们需要先添加数据源。

1、首先，登陆grafana，点击菜单中的 Data Sources

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-1@2x.png)

2、点击 Add data source 按钮

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-0@2x.png)

3、填入相应参数，点击添加按钮即可


![img](http://p5bjfbphc.bkt.clouddn.com/zhaohang:datasource.png)
![img](http://p5bjfbphc.bkt.clouddn.com/zhaohang:datasource2.png)

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
