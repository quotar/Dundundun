---
title: "ssh隧道与代理"
date: 2020-03-08T11:50:19+08:00
draft: false
---

#### ssh代理和ssh隧道的区别：  

---

ssh隧道：是利用ssh本身的加密传输，在两台机器间打通隧道，让两台机器各自局域网内其他服务走22端口，所以实际上可以看作是开代理让别的服务用  
ssh代理：在两台机器间已有的端口上，建立ssh隧道，实际上是走代理

---

#### ssh 代理(ssh走代理)
ssh 隧道/tunnel+ ssh config 实现快速跳板机  
###### 直接命令方式隧道
ssh -p23000 thor-uc2-was-master-01 -o ProxyCommand="ssh -W %h:%p vechain@thor-uc2-was-proxy-01 -p23000"
注：thor-uc2-was-proxy-01 上的/etc/hosts 里要有thor-uc2-was-master-01 的IP 记录
######  ～/.ssh/config 文件    
>Host thor-uc2-was-master-*       
    ProxyCommand    ssh -W %h:%p vechain@thor-uc2-was-proxy-01 -p23000
#### ssh 隧道（ssh开代理）：

###### 本地隧道
将10.0.1.2上的3306端口  映射到10.0.1.1上的9906端口，也可以理解为将10.0.1.1的9066端口转发到10.0.1.2上的3306端口
ssh -gfNL   9066:10.0.1.2:3306  root@10.0.1.2
-L  建立ssh隧道
-N  不执行远程命令，仅仅用在端口转发上
-f   强制将此次ssh回话放到后台运行
-g   不光监听 127.0.0.1 的9066端口，监听所有本地IP的9066端口

###### 远程隧道（内网穿透）
ssh -fNR  remoteip:remoteport:localip:localport   user@remoteip
将本地内网监听的端口，转发到外网机器上

###### 动态隧道
通过 -D 参数，我们可以实现一个简单的代理服务器。

>ssh -D [bind_address:]port root@服务器ip

在平常，我们的电脑是无法访问谷歌的，但是现在我们手上有一台服务器A可以访问谷歌。现在我们就可以通过ssh -D来让服务器A变成一个代理服务器，从而让我们的电脑可以访问谷歌。

绑定本地的8080端口，后面发往8080端口的请求都会通过ssh隧道发给服务器A，由服务器Alain进行请求后再将响应返回给我们   
>ssh -D 127.0.0.1:8080 root@服务器A的ip


之后我们需要设置一下浏览器的代理，地址使用127.0.0.1:8080。之后浏览器的请求都会走向8080端口，到8080端口后，经过ssh隧道发向服务器A，服务器把这个请求发往目标服务器，之后拿到响应内容后返回给我们。整个过程就是一个正向代理的过程。