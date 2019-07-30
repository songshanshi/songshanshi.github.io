---
title: 使用ssh反向代理+nginx实现外网连接内网服务器
---

### 一.准备条件
机器		-ip-用户-主机名-备注
A	-10.10.10.128     -    root        www    -    位于内网的目标服务器
B	-123.123.123.123 -root  vultr	- 位于公网的服务器
C - 192.168.1.1 - root - 123- 可以 访问公网的客户机

### 二.方案
简单的说：在A机器上做到B机器的反向代理，将A机器上的服务端口映射到B机器的本地端口，然后使用nginx实现将本地端口转发到80(http)服务端口。
#### 1.前提
在AB机器上安装好ssh客户端和服务端 ，机器c上装上浏览器
在B机器上装好nginx，测试使用完好
#### 2.需要使用的ssh参数
反向代理 ```ssh -fCNR ```
正向代理```ssh -fCNL ```
```powershell
-f 后台执行ssh指令
-C 允许压缩数据
-N 不执行远程指令
-R 将远程主机(服务器)的某个端口转发到本地主机指定的端口
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口
-p 指定远程主机的端口
```
#### 3.建立A到B的反向代理
```
ssh -fCNR [B机器IP或省略]:[B机器端口]:[A机器的IP]:[A机器的sshd端口] [登录B机器的用户名@B机器的IP] -p [B机器的sshd端口]
```
A机器的服务端口是4000，反向代理到B机器的4001端口，在A机器上操作：
```
ssh -fCNR 4001:localhost:4000 -o ServerAliveInterval=60 root@123.123.123.123 -p 22
```
- 注：-o ServerAliveInterval=60表示服务器B每隔60秒发送一次数据给服务器A，以便ssh连接不会因为超时而断开连接.
然后这里之前要先实现A机器到B机器的ssh免密码登录，检查是否成功建立连接可以在A机器上```curl localhost:4001```或者是可以```netstat -antpul |grep '127.0.0.1:4001'```来确认机器A到B的反向连接是否建立成功。

#### 4.使用nginx实现本地端口的转发
在A机器上 ```vim /etc/nginx/nginx.conf```
将下面配置信息添加到上面
```powershell
 location / {
        proxy_pass http://localhost:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        }

```
然后启动nginx，就可以成功将内网服务搭载公网了，使用C机器访问公网ip，这里还有就是一定要注意配置防火墙规则等。
```
systemctl start nginx
systemctl enable nginx
```
[这个博客]:https://blog.csdn.net/u012843189/article/details/79522738
本文主要引用[这个博客]