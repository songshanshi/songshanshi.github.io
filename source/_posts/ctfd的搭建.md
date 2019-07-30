---
title: 记一次将ctfd搭建到校园网的centos主机上
---

看见其他学校的大佬都给学校搭建训练平台，羡慕不已，故我也搭一个练练手 嘻嘻。。
### ctfd的搭建
#### 环境准备
```powershell
yum install -y git nginx mariadb mariadb-server Mysql-python python-pip 
```
打开mariadb
```powershell
systemctl start mariadb
```
设置密码
```sql
mysql_secure_installation
```
#### 下载设置ctfd框架
```powershell
git clone https://github.com/isislab/CTFd.git
```
这里可能需要翻墙。。

下载完后进入到CTFd目录
```powershell
./prepare.sh
python3 serve.py  # 如果出现缺少库的话pip install安装相应的库
```
我看其他教程还要：
- 找到SQLALCHEMY_DATABASE_URI这一参数，然后更改为：SQLALCHEMY_DATABASE_URI = 'mysql://root:你的密码@localhost/CTFd?charset=utf8'找到HOST参数，更改为HOST = "你的服务器IP"

但是我并没有做好像直接接就成功了 ~\(≧▽≦)/~。。
然后可以在MySQL里看到出现CTFd的数据库

然后在当前目录下运行
```powershell
gunicorn --bind 0.0.0.0:4000 -w 1 "CTFd:create_app()"
```
在成功后，访问你的IP:4000就可以看到网页了，如果发现访问不了，看下你的防火墙，或者安全组配置里是否开启了相应端口


#### 配置nginx

弄完CTFd后访问，我发现，巨卡，是真的卡，于是想到把最近学到nginx加上，好了许多
```powershell
vim /etc/nginx/nginx.conf
```
修改
```powershell
location /{
proxy_pass http://localhost:4000;

proxy_set_header Host $host;

proxy_set_header X-Real-IP $remote_addr;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_redirect off;
}
```
设置完成
```powershell
nginx -t # 检测语法
nginx -s reload # 重载
systemctl enable nginx
systemctl enable mariabd
systemctl start nginx 
gunicorn --bind 0.0.0.0:4000 -w 1 "CTFd:create_app()"
```
最后一定要配置要防火墙安全策略之类的东西
firewall-cmd --add-port=4000/tcp --permanent     ##永久添加80端口 
流畅了吧，兄弟 ●０● 。这个gunicorn的命令一定要进CTFd目录输入
ojbk  ^O^ 
