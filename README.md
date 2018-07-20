# zabbix-grafana
docker-compose file for build up zabbix server and grafana

该文档用于部署docker集成的zabbix服务以及前端展示平台grafana

## 一、部署
### 1. 下载docker-compose文件以及所需目录,并更新grafana-zabbix插件
```
git clone git@github.com:fivestrong/zabbix-grafana.git
git clone https://github.com/alexanderzobnin/grafana-zabbix.git zabbix-grafana/grafana/plugins/grafana-zabbix

注意：配置文件中将数据库文件做了本地volumes,请根据自己的规划更改相关路径。

```
### 2.提前下载好docker所需镜像 
```
docker pull mysql:5.7
docker pull zabbix/zabbix-server-mysql:latest
docker pull zabbix/zabbix-web-nginx-mysql:latest
docker pull grafana/grafana:latest
```
### 3.利用docker-compose部署镜像
```
cd zabbix-grafana
docker-compose up -d
```

### 4.~~添加sendEmail插件~~
注:更新最新版本zabbbix-server-mysql之后该步骤会导致容器无限重启，具体原因没有细
研究，改为使用官方自带Email来发送，除了没研究怎么添加抄送，只能每个联系人各发送一份邮件，其他一切完美。
```
docker-compose down
cd zabbix-server-mysql-sendEmail
docker build -t zabbix/zabbix-server-mysql:latest .

cd ..
docker-compose up -d 
```
### 5. 利用官方Email插件发送告警邮件
(图片来自互联网，侵删)

管理-示警媒介类型-Email
![](https://s1.ax1x.com/2018/07/18/P11FyD.png)
设置Zabbix用户报警邮箱地址
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831121337421-1164876090.png)
```shell
名称：Email
类型：电子邮件
SMTP 服务器：zabbix.163.com
SMTP HELO：zabbix.163.com
SMTP电邮：zabbix@zabbix.163.com
已启用：勾选
```
配置-用户-Admin (Zabbix Administrator)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831123805468-1251886088.png)
切换到示警媒介
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831123834124-894032562.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831123919280-1633570748.png)
```shell
类型：Email
收件人：xxx@163.com
其他默认即可，也可以根据需要设置
状态：已启用
```
设置Zabbix触发报警的动作
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831124150999-1243269059.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831124612312-1898904334.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831125258218-1564250658.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831125329780-1879928416.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831125347030-1112819673.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831125403796-1942217419.png)
![](https://images2017.cnblogs.com/blog/1096467/201708/1096467-20170831125551374-1395860935.png)


## 二、配置

### 1 配置zabbix web
正常情况下部署顺利的话，镜像会自动生成所需要的数据库，直接登录web http://ip 初始用户名：Admin 密码：zabbix
如果出现网页打不开或者无法读取数据库配置等错误，可能是修改了docker-compose.yml中
```
DB_SERVER_HOST: "mysql"
links:
      - mysql:zabbix-mysql
      - server:zabbix-server
```
### 2.添加zabbix server本身服务器的监控

初始zabbix web添加了Zabbix server的主机监控选项，有人说zabbix docker镜像有docker agent,但是直接点enable开启会报错。
于是又在本机上安装了zabbix-agent的rpm包
```
rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm
yum install zabbix-agent zabbix-get #zabbix-get用于测试客户端
vim /etc/zabbix/zabbix_agentd.conf
修改参数
Server=172.19.0.4
ServerActive=172.19.0.4:10051
Hostname=Zabbix server
这里有个大坑，就是Server 这一项需要填成zabbix-server docker镜像的ip地址，填写本机真实ip或者127.0.0.1都会报错，
但是用zabbix_get命令测试通过这两个地址相关参数都没问，但web会显示error,查看zabbix-agent的log会发现172的有个地址
不被允许。其他非本机zabbix-agent直接配置zabbix-server真实地址即可

总结一下：这里Server和ServerActive地址要填写zabbix-server-mysql容器的地址。
```
### 3.配置grafana
登录http://ip:3000
默认用户名admin 密码admin
使zabbix插件生效：Plugins->app->Zabbix->Enable
添加Data Source
填写以下内容，此数据源为Zabbix的数据库，在第二个数据源中会用到。
```
注:因为是在docker容器操作，这里mysql地址填写为mysql容器地址,即172.19.0.x
```

```
Name: Zabbix
Type: MySQL
MySQL Connection Host: 172.19.0.x:10053
Database: zabbix
User: zabbix
Password: zabbix_pwd

Save & Test 如果登录成果会有提示
```
继续添加数据源。内容如下：
```
注:同上这里url地址填写zabbix-nginx-web容器的地址，端口默认为80
```
```
Name: zabbix
Type: Zabbix
url: http://172.19.0.x:80/api_jsonrpc.php
Access: Server(Default)
Basic Auth: √
Basic Auth Details User: admin
Basic Auth Details Password: zabbix
Zabbix API details Username: admin
Zabbix API details Password: zabbix
Direct DB Connection Enable: √
Direct DB Connection SQL Data Source: Zabbix
Alerting Enable alerting: √
Alerting Add thresholds: √

Save & Test 如果登录成果会有提示
```
### 4.导入zabbix数据
在添加zabbix数据源之后，Settings旁边有个Dashboards选项，里面有
```
Zabbix System Status                           import
Zabbix Template Linux Server                   import

导入这两个就可以看到zabbix监控的数据了，具体其他配置图表可以到官方文档中查找
```
里面有实例
http://play.grafana-zabbix.org/

## 小知识
通过iptables限制docker容器端口
```
docker 会在iptables上加上自己的转发规则，如果直接在input链上限制端口是没有效果的。这就需要限制docker的转发链上的DOCKER表。
# 查询DOCKER表并显示规则编号
iptables -L DOCKER -n --line-number 
# 修改对应编号的iptables 规则，这里添加了允许访问ip的限制
iptables -R DOCKER 5 -p tcp -m tcp -s 192.168.1.0/24 --dport 3000 -j ACCEPT
```
## 参考
https://liqiang311.github.io/linux/%E5%9F%BA%E4%BA%8EDocker%E7%9A%84Zabbix+Grafana%E7%9B%91%E6%8E%A7/

