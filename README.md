# zabbix-grafana
docker-compose file for build up zabbix server and grafana

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

```
### 3.配置grafana
登录http://ip:3000
默认用户名admin 密码admin
使zabbix插件生效：Plugins->app->Zabbix->Enable
添加Data Source
```
Name: zabbix
Type: Zabbix
url: http://ip:80/api_jsonrpc.php
Access: direct
Zabbix API details Username: Admin    #zabbix 登录用户名
Zabbix API details Password: zabbix   #zabbix 登录密码

Save & Test 如果登录成果会有提示
```
## 参考
https://github.com/liqiang311/zabbix-grafana
由于以上配置在实际测试环境跑不通，所以我进行了修改以及把遇到的坑填了一下，在此感谢原作者。
