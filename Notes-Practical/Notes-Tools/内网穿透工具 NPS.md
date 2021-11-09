# NPS

[TOC]

## 介绍

基于微信平台开发时，在本地测试常常需要用到内网穿透工具。

### 什么是内网穿透

#### 内网穿透：

即 NAT 穿透，网络连接时术语，计算机是局域网内时，外网与内网的计算机节点需要连接通信，有时就会不支持通信。我们可以通过映射端口让外网的电脑找到内网的电脑，从而进行通信，简单来说就是让外网能够访问内网。

#### 内网：

所谓内网就是内部建立的局域网络或办公网络。

#### 外网：

所谓外网就是通过一个网关或网桥与其它网络系统连接，相对于自己的内网来说，连接的其它网络系统就称为外部网络，也叫外网。

### 内网穿透原理

内网穿透是基于 NAT 实现的

#### 什么是 NAT

NAT ( Network Address Translation ) 即网络地址转换，当在专用网内部的一些主机本来已经分配到了本地 IP 地址（即仅在本专用网内使用的专用地址），但现在又想和因特网上的主机通信（并不需要加密）时，可使用 NAT 方法。

#### NAT 的分类

NAT 的实现方式有三种：静态转换、动态转换、端口多路复用。

- **静态转换**是指将内部网络的私有 IP 地址转换为公有 IP 地址，IP 地址对是一对一。
- **动态转换**是指将内部网络的私有 IP 地址转换为公用 IP 地址时，IP 地址是不确定的，是随机的，所有被授权访问上 Internet 的私有 IP 地址可随机转换为任何指定的合法 IP 地址。
- **端口多路复用**（Port address Translation,PAT)，使内部网络的所有主机均可共享一个合法外部 IP 地址实现对 Internet 的访问，从而可以最大限度地节约 IP 地址资源。同时，又可隐藏网络内部的所有主机，有效避免来自 internet 的攻击。因此，目前网络中应用最多的就是端口多路复用方式。
- **应用程序级网关技术**（Application Level Gateway）ALG，传统的 NAT 技术只对 IP 层和传输层头部进行转换处理，但是一些应用层协议，在协议数据报文中包含了地址信息。为了使得这些应用也能透明地完成 NAT 转换，NAT 使用一种称作 ALG 的技术，它能对这些应用程序在通信时所包含的地址信息也进行相应的 NAT 转换。

## 搭建内网穿透

实现内网穿透的方式有很多，在没有一个合法 IP 的情况下可以使用[NATAPP](https://natapp.cn/)、[花生壳](https://hsk.oray.com/)等软件来实现内网穿透，也可以使用[nps](https://github.com/cnlh/nps)，[frp](https://github.com/fatedier/frp)之类的软件在自己的云服务器上进行搭建。

### 环境介绍

我使用的是 nps 在自己的云服务器上实现内网穿透，npc 是支持 tcp、udp 流量转发，支持内网 http 代理、内网 socks5 代理，同时支持 snappy 压缩、站点保护、加密传输、多路复用、header 修改等，还提供了一个可视化页面进行管理。[nps 的 Github 地址](https://github.com/cnlh/nps/releases)
  我的环境：云服务器 Linux（Ubuntu 20.2），本地主机 Windows10，nps 版本 0.26.10。

### 下载 NPS 

下载一个服务器端和一个客户端，版本一定要和自己的操作系统对应。[官网下载地址](https://github.com/cnlh/nps/releases)

### 服务器端

必须是拥有独立 ip 的公网服务器

#### 配置

将下载好的`linux_amd64_server.tar.gz`解压在自己的云服务器上。打开 conf 文件夹中的 nps.conf 配置文件，配置太多的我只展示需要更改的。

```java
 # 设置为云服务的内网 ip 地址，注意是云服务器的内网 ip 地址
 http_proxy_ip= xxx.xxx.xxx.xxx
 # http的默认是 80 端口，如果占用更换为其他
 http_proxy_port= 80
 # 确保防火墙的 443 端口是开放的
 https_proxy_port= 443
 
 # 设置 npc 的启动端口号，即服务端客户端通信端口
 bridge_port= xxx

# web管理页面的账号
web_username= admin
# web管理页面的密码
web_password= 123
# web管理端口，该端口和上面的 npc 启动端口不能相同，否则访问不到wen管理页面
web_port = xxx
```

##### 2）常用命令

  启动 nps

```java
1./nps start
```

  如果无需 daemon 运行或者打开后无法正常访问 Web 管理，去掉 start 查看日志运行即可

```java
1./nps
```

  服务端配置文件重载

```java
1./nps reload
```

  服务端停止或重启

```java
1 ./nps stop|restart
```

##### 3）Web 页面的配置

###### ① 登录

  账号密码都在配置文件里，这里不多做解释，登录进去后可以看到一个非常棒的管理界面。

![image.png](https://img.hacpai.com/file/2019/12/image-d5103247.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

###### ② 添加客户端

  点击客户端的新增按钮

![image.png](https://img.hacpai.com/file/2019/12/image-cde41625.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

  备注选填，认证和验证信息可以不填会自动生成，然后将压缩和加密设置成 yes，点击新增。

![image.png](https://img.hacpai.com/file/2019/12/image-3f5e6e15.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

  点击右边的 tunnel 按钮，进去后继续点击新增按钮。

![image.png](https://img.hacpai.com/file/2019/12/image-7f4d2a09.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

  按照提示完善信息，点击添加即可（window 在 DOS 窗口通过 ipconfig 命令获取本地 ip 地址）。

![image.png](https://img.hacpai.com/file/2019/12/image-a0785cdd.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

#### 4、客户端

  将下载好的`win_amd64_client.tar`解压在自己的电脑上。官网是通过`npc.exe`运行，而我使用`npc.exe`运行就直接闪退，所以我们这里采用 DOS 命令启动。
  打开 DOS 命令窗口切换到`npc.exe`所在的文件夹，执行以下命令启动。

```
1npc.exe -server=服务器域名/ip:端口 -vkey=密钥 -type=tcp
```

  命令参数可以在 Web 管理页面查看。

![image.png](https://img.hacpai.com/file/2019/12/image-04521429.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

------



