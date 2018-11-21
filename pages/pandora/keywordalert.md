运维日志分析 -- 日志搜索和关键字报警


相信你经常会苦恼于线上大量服务器运维的压力，是不是出现某个机器故障，磁盘损坏亦或是内存超限等等。而这些异常现象发生之前，我们的运维日志中经常会出现一些提示日志。比如 `/var/log/syslog`，`/var/log/message`等等，那么如何快速指定日志文件（如：`/var/log/message`）中出现指定字段（如：`Stopping`,`Error`）后告警呢？

使用 Pandora，我们帮您轻松搞定这个事情。


## 采集数据

### 1. 根据您机器的操作系统版本下载 logkit

https://github.com/qiniu/logkit/wiki/Download

解压后您可以看到

```
logkit
logkit.conf
```

其中 `logkit.conf` 为主配置文件，用于配置启动监听的端口，启动后可以通过浏览器访问该端口进行 logkit 的配置。默认情况下为: "http://127.0.0.1:3000"。

### 2. 运行 logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1
```

下面我们将为您介绍如何配置 logkit 以收集您的运维日志。

### 3. 访问 logkit 配置页面

通过浏览器打开 logkit 的页面，在首页可以看到总体的运行情况。

![图1 logkit 首页](http://ou3jgt6kj.bkt.clouddn.com/logkitnginx1.png)

点击 【增加 Runner】就可以开始配置数据收集的 Runner 了。

首先就是填写您的日志路径。

![此处输入图片的描述][1]

然后选择原始日志按行解析方式(raw)。

![此处输入图片的描述][2]


### 4. 配置发送方式

![此处输入图片的描述][3]

在发送方式这边，最基本的只需要填写您的”数据源名称“，”七牛的公钥“，以及”七牛的私钥“ 三项即可。

点击【下一步】，【确认并提交】，至此就完成了您的所有配置。

更多 logkit 的高级用法，参见 [logkit wiki文档](https://github.com/qiniu/logkit/wiki)

### 5. 登录 Pandora 界面

运行后，在 [https://portal.qiniu.com/logdb](https://portal.qiniu.com/logdb) 上就已创建了日志仓库：`my_work`

![此处输入图片的描述][4]

点击右边的设置修改按钮，可以设置仓库的保存时长，logkit 自动创建的仓库数据默认只保存三天，以及设置不同类型的分词。

![此处输入图片的描述][5]

**注意**

* 修改仓库数据保存期限修改后即时生效
* 修改数据的分词，修改后配置在 T+1 时间后生效，对之前的数据不生效。
* 纯英文的日志建议设置为标准分词，以便快速检索数据；带有中文的数据建议设置为中文分词；如果字段都是关键字，类似"INFO"、"ERROR"，则建议设置为”不分词“。

Q：如何即时生效？
A：删除仓库 repo_test  → 重新新建仓库 repo_test → 将配置设置为上图一样

## 检索日志

以关键字 Stopping 为例，检索结果如下，此处"raw"字段的分词为”标准分词“

![此处输入图片的描述][6]

## Grafana 监控和告警

具体配置可参考 Pandora 文档站中的 [Grafana 简介](https://qiniu.github.io/pandora-docs-v2/#/grafana) 此处不再赘述，下面只给出示例配置。

### 1. 创建 Grafana App，如 图6 所示

![图6 Grafana APP 创建](http://op26gaeek.bkt.clouddn.com/newbuildGrafana.png)

### 2. 配置 Grafana LogDB 数据源，如 图7 所示，点击 logdb 使用指南，可以按照使用指南的指导在 Grafana 配置数据源。

![图7 Grafana 数据源配置](http://op26gaeek.bkt.clouddn.com/logdbGrafana.png)

**注意事项**

- Default Query Settings 中， Group by interval 填写时间 `10s`，注意单位为`s`,`m`等，不能漏掉，必须小写。
- Time Field Name 处填写您的 logdb 时间字段， 填您 nginx 配置的命名，在上述的截图示例中，是 `time_local` , 没有默认的 `$` 符号
- Index name中，模式固定为 `Daily` , 串固定为 `[reponame-]YYYY.MM.DD` , 将 reponame 字符串改为您的数据源名称即可。
- Version 固定为 `2.x`


### 3. 新建 Grafana 的 dashboard 并创建 Graph

### 4. 设置 metrics

![此处输入图片的描述][7]

### 5. 设置 Alert 告警
设置 condition，可以选择 sum(), count() 等，IS ABOUT 为设置的阈值

![此处输入图片的描述][8]

### 6. 设置告警 email 通知
首先，设置邮件通知组

![此处输入图片的描述][9]

### 7. 点击 sent test，测试是否可以收到邮件
![此处输入图片的描述][10]

### 8. 设置告警通知

![此处输入图片的描述][11]

### 9. 当超过设置的阈值后就出现告警，并发邮件，包含之前写好的 message，以及此刻 graph 截图。如下图所示：

![此处输入图片的描述][12]

以上就轻松完成利用 Pandora 配置关键字告警功能啦！


  [1]: http://ou3jgt6kj.bkt.clouddn.com/logkitsyslog.png
  [2]: http://ou3jgt6kj.bkt.clouddn.com/nginlog6.png
  [3]: http://ou3jgt6kj.bkt.clouddn.com/logkitnginx5.png
  [4]: http://ou3jgt6kj.bkt.clouddn.com/logdb1.png
  [5]: http://ou3jgt6kj.bkt.clouddn.com/logdb2.png
  [6]: http://ou3jgt6kj.bkt.clouddn.com/logdb3.png
  [7]: http://ou3jgt6kj.bkt.clouddn.com/logdb4.png
  [8]: http://ou3jgt6kj.bkt.clouddn.com/logdb5.png
  [9]: http://ou3jgt6kj.bkt.clouddn.com/logdb6.png
  [10]: http://ou3jgt6kj.bkt.clouddn.com/logdb7.png
  [11]: http://ou3jgt6kj.bkt.clouddn.com/logdb8.png
  [12]: http://ou3jgt6kj.bkt.clouddn.com/logdb9.png