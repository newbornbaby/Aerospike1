# TOOLS（工具）

## 1 Management Console（管理工具）

### 1.1 Introduction(介绍)

​		Aerospike管理控制台（AMC）是一个基于web的工具，用于监视/管理Aerospike群集，它提供群集当前状态的实时更新，它有两个版本。

​		**1）AMC 社区版本**

​		社区版本对每个人都是免费使用的，他包括一些特性，让你一目了然的看到集群的吞吐量、存储的使用情况和集群的配置信息。

​		**2）AMC企业版本**

​		企业版本包括了一些额外的高级功能，去更好的监控管理aerospike集群。关于社区版与企业版的功能对比可以参考下图。

![image-20200103153531126](C:\Users\bangsun\AppData\Roaming\Typora\typora-user-images\image-20200103153531126.png)

### 1.2 Install（安装教程）

#### 1.2.1 Red Hat and Centos

**下载**

​		地址：https://www.aerospike.com/download/amc/4.0.27/

**安装依赖组件**

​		*python (2.6+)*

​		*gcc*

​		*python-devel*

​		rpm -qa | grep xxx 查看是否已经安装，若没有需要安装yum list | grep xxx yum install xxx

**安装rpm**

​		*sudo rpm -ivh aerospike-amc-<version>.rpm*

**备份**

​		*sudo cp /etc/amc/amc.conf /etc/amc/amc.conf.bac*

**启动**

​		*sudo /etc/init.d/amc start*

​		*sudo /etc/init.d/amc stop*

​		*sudo /etc/init.d/amc restart*

​		*sudo /etc/init.d/amc status*

​		you can check the error logs in /var/log/amc/error.log.

​		systemctl status amc

### 1.3 Configure （配置）

​		配置文件地址(Linux)：/etc/amc/amc.conf，配置文件遵循TOML语法，配置分为以下上下文。

#### 1.3.1 [amc]（必需）

​		与AMC运行时行为相关的配置。

```basic
[AMC]
update_interval             = 5
certfile                    = "/home/amc/cert.pem" #optional
keyfile                     = "/home/amc/key.pem"  #optional
database                    = "/home/amc/amc.db"
bind                        = ":8081"
loglevel                    = "info"
errorlog                    = "/home/amc/amc.log"
chdir                       = "/home/amc"
static_dir                  = "/home/amc/static"
cluster_inactive_before_removal = 1800
```

​		update_interval：AMC捕获正在监视的群集的统计信息的默认时间间隔（秒）。

​		certfile，keyfile：在https模式下运行AMC app server的公钥/私钥对。文件必须包含PEM编码的数据。证书文件可以包含叶证书之后的中间证书，以形成证书链。

​		database：用于保存AMC通知、备份/还原作业历史记录和其他内部状态的文件。如果要确保维护此数据，可以通过复制此文件来手动备份此文件。这个文件通常不应该超过100MB的大小。

​		bind：AMC绑定的端口号。

​		loglevel：日志级别，debug、warn、info、error。

​		errorlog：日志路径。

​		chdir：工作路径。

​		static_dir：静态资源路径。

​		cluster_inactive_before_removal：定义在该ms数下如果没有收到任何集群统计的信息，则从监控列表中删除对该集群的监控，但是，无论该值如何配置，配置文件中配置的需要监控的集群永远不会被删除。

#### 1.3.2 [amc.clusters]（可选）

​		启动时AMC始终监视的群集列表。

```bash
[amc.clusters]

[amc.clusters.clusterone] # unused
host         = "192.168.121.121"
port         = 3000
show_in_ui   = true
tls_name     = "clusteronetls"
user         = "admin"           
password     = "admin123"
alias        = "clusterone"

[amc.clusters.clustertwo] # unused
host         = "192.168.121.122"
port         = 3000
show_in_ui   = true
tls_name     = "clustertwotls"
user         = "admin"
password     = "admin123"
alias        = "clustertwo"
```

​		每个集群都有以下配置：

​		host、port：服务器ip、端口号。

​		user、password：对于允许进入as集群的用户名、密码，如果没配置可以配置。

​		alias：集群别名。

​		show_in_ui：当AMC加载到浏览器中时，在ui多群集视图中显示群集。

​		use_services_alternate： allows the use of services_alternate on the server to be able to connect from a public network to the cluster。暂时没试过这是用来做什么用的。

#### 1.3.3 [mailer]（可选）

​		AMC用于发送警报电子邮件的配置。

```bash
[mailer]
template_path             = "/home/amc/mailer/templates"
host                      = "smtp.outlook.com"
port                      = 587
user                      = "user"
password                  = "user123"
send_to                   = ["monitorone@gmail.com", "monitortwo@yahoo.com"]
accept_invalid_cert       = false
```

​		template_path：包含电子邮件模板的目录。

​		host、port：邮件服务的ip、端口号。

​		user、password：邮件服务的用户名、密码。

​		send_to：要向其发送警报的电子邮件列表。

​		accept_invalid_cert：如果为true，则继续发送邮件，即使证书无效。

#### 1.3.4 [basic_auth]（可选）

​		供AMC使用的HTTP基本身份验证凭据。

```bash
[basic_auth] 

user       = "user" 

password   = "user123"
```

​		user、password：用于HTTP基本身份验证的用户名和密码。

#### 1.3.5 [TLS]（可选）

​		AMC在验证服务器证书时使用的根证书颁发机构集。













