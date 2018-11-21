## 产品简介
### 招行宁夏区AWS机器安装Logkit-pro文档

### logkit Pro 安装方法

宁夏区的项目安装请访问：**[http://55.18.5.171](http://55.18.5.171)**

进入 **[logkit Pro 官网](http://55.18.5.171)**，点击右上角登录，登录用户名密码提前找七牛申请。*（默认用户名为工作空间名，需要提前找七牛这边激活这个工作空间名哦）* 点击【机器管理】-> 【添加机器】-> 【手动安装】，可以通过“手动安装”模块，可以获取我们提供的一键式安装脚本：

### 针对Linux 机器

**第一步：**

登录后点击添加机器，选择Linux安装，复制地址，如下图
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:anzhuang1.png)

在复制出来的地址前**加上**：
```
export SMARTELF_HOST=http://55.18.5.171:80/v1;export LOGDB_HOST=http://55.18.5.171:9997;export PIPELINE_HOST=http://55.18.5.171:9999;export DOWNLOAD_URL=http://55.18.5.171:80/v1/archives;
```

同时将IP：`12.16.5.50`修改为`55.18.5.171` 即为安装地址。



这里测试的最后安装地址为：

```
export SMARTEL_HOST=http://55.18.5.171:80/v1;
export LOGDB_HOST=http://55.18.5.171:9997;
export PIPELINE_HOST=http://55.18.5.171:9999;
export DOWNLOAD_URL=http://55.18.5.171:80/v1/archives;
export QINIU_ACCESS_KEY=KuNhy……nqcK; 
export QINIU_SECRET_KEY=tbHrck……Lunnf; 
export QINIU_EMAIL=qiniu;
curl -L 'http://55.18.5.171:80/v1/install/5dd495b924fc1ebc872bea143/sh?tags=' | bash
```



**第二步：**

在linux服务器上执行以上构造好的命令即可安装成功,
效果如下：
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:installsuccess.png)
上述步骤安装好以后,访问：[http://55.18.5.171](http://55.18.5.171)点击右上角机器管理就可以看到您的机器。



### 针对Windows机器
**第一步：**

下载安装脚本到Windows机器
![](http://p5bjfbphc.bkt.clouddn.com/zhaohang:windows.jpeg)


**第二步：**

在windows机器上设置四个环境变量即可


> SMARTELF_HOST  设置为   http://55.18.5.171:80/v1
> 
> LOGDB_HOST   设置为  http://55.18.5.171:9997
> 
> PIPELINE_HOST设置为 http://55.18.5.171:9999
> 
> DOWNLOAD_URL设置为 http://55.18.5.171:80/v1/archives

`Windows设置环境变量的方法见下文附录`



**第三步：**
设置完环境变量后执行脚本即可



#### 附录：Windows的环境变量设置方法

Windows 中设置环境变量

第一步:
![](http://p5bjfbphc.bkt.clouddn.com/1.jpeg)
第二步:
![](http://p5bjfbphc.bkt.clouddn.com/2.jpeg)
第三步:
![](http://p5bjfbphc.bkt.clouddn.com/3.jpeg)
第四步:
![](http://p5bjfbphc.bkt.clouddn.com/4.jpeg)
第五步:
![](http://p5bjfbphc.bkt.clouddn.com/5.jpeg)
第六步:
![](http://p5bjfbphc.bkt.clouddn.com/6.jpeg)

