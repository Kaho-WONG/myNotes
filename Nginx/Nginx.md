# Nginx 

> 涵盖Nginx基本概念，软件的安装和基本操作命令，Nginx的各种应用场景的配置以及Nginx的原理。
>
> 视频教程地址：[尚硅谷Nginx教程由浅入深（一套打通丨初学者也可掌握）](https://www.bilibili.com/video/BV1zJ411w7SV?share_source=copy_web)



[toc]



# 1. Nginx 简介



## 1.1 什么是 Nginx？ 

Nginx 是一个**高性能**的 **HTTP 和反向代理服务器**，特点是**占有内存少，并发能力强**。

> 事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，能经受高负载的考验，有报告表明能支持高达 50,000 个并发连接数。



## 1.2 Nginx 作为 web 服务器

Nginx 可以作为**静态页面的 web 服务器**，同时还**支持 CGI 协议的动态语言**，比如 perl、php 等。但是**不支持 java**，Java 程序只能通过与 tomcat 配合完成。



## 1.3 正向代理

Nginx 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。 

**正向代理：** 是一个位于客户端和原始服务器(origin server)之间的服务器。为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端（目的是为了保护客户端用户） 。

> **需要在客户端（浏览器）配置代理服务器进行指定网站访问。**

![1637140544284](./imgs/1637140544284.png)

![1637141619818](./imgs/1637141619818.png)



## 1.4 反向代理

**反向代理：**是在客户端和服务器端之间的一个代理服务器，但是这个代理服务器的目的是**用来保护服务器端的**，当外网客户端来访问我们的服务器时，为了避免暴露应用服务器的实际 ip 地址，我们会让客户端访问代理服务器，然后代理服务器再根据客户端的请求去实际的应用服务器获取响应内容并返回客户端。 **此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。** 

![1637141232479](./imgs/1637141232479.png)

![1637141766318](./imgs/1637141766318.png)



> 正向隐藏客户端，反向隐藏服务器。



## 1.5 负载均衡

单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的**负载均衡**。

![img](./imgs/1064872-20190822222926586-494023196.png) 

![1637148104353](./imgs/1637148104353.png)



## 1.6 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

![动静分离](./imgs/b6761c5472d24959a87c526c22e93a1c.png) 

> Tomcat 服务器存储动态资源（jsp、servlet ……）





# 2. Nginx 安装（Linux）

## 2.1 进入 nginx 官网下载

官网地址：http://nginx.org/

![1637149424834](./imgs/1637149424834.png)

本教程（尚硅谷）需要的素材：

<img src="./imgs/1637149562943.png" alt="1637149562943" style="zoom:50%;" />



## 2.2 安装 nginx

### 2.2.1 安装 pcre 依赖 

**第一步** 

联网下载 pcre 压缩文件依赖

```shell
wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz 
```

在 `/usr/src/` 目录下：
![1637150995023](./imgs/1637150995023.png)

**第二步** 

解压压缩文件，使用命令：

```shell
tar –xvf pcre-8.37.tar.gz
```

**第三步** 

进入解压得到的 `pcre-8.37` 文件夹，执行 `./configure` ；

完成后，继续在 pcre 目录下执行 `make`，最后执行 `make install`；也可以直接一步到位执行这一条命令 `make && make install`。

安装成功后查看安装的版本号：

![1637150736041](./imgs/1637150736041.png)



### 2.2.2 安装 openssl、zlib、gcc 依赖

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```



### 2.2.3 安装 nginx

**第一步**

跟 pcre 一样，直接把已经下载好的压缩包 `nginx-1.12.2.tar.gz` 放到同一目录  `/usr/src/` 下。然后使用命令解压：

![1637151363172](./imgs/1637151363172.png)

```shell
tar -xvf nginx-1.12.2.tar.gz
```

**第二步**

进入解压得到的 `nginx-1.12.2` 文件夹，执行 `./configure` ；

**第三步**

同样在本目录下执行 `make && make install` 进行编译并安装。



## 2.3 启动 Nginx

安装成功后，在 `/usr/local/nginx/sbin/` 下可以找到 nginx 的启动脚本：

![1637151865165](./imgs/1637151865165.png)

如何启动？

进入到这个目录后输入命令：

```shell
./nginx
```

可以看到，已经启动了 nginx 的进程：

![1637152144855](./imgs/1637152144855.png)



nginx 默认监听 **80** 端口，可以直接通过 **虚拟机 ip 地址 + 端口号** 在浏览器尝试访问~

> 我的 linux 虚拟机 ip：192.168.200.130

![1637152447536](./imgs/1637152447536.png)



**注意：**

在 windows 系统中访问 linux 中 nginx，默认不能访问的，因为**防火墙**问题，可以通过下面两种方法解决：

（1）关闭防火墙 

（2）开放访问的端口号，80 端口 



**查看开放的端口号** 

```shell
firewall-cmd --list-all
```

![1637152819684](./imgs/1637152819684.png)

设置开放的端口号 

```shell
firewall-cmd --add-service=http --permanent 
或
firewall-cmd --add-port=80/tcp --permanent 
```

重启防火墙 

```shell
firewall-cmd --reload
```





# 3. Nginx 常用命令

 首先要进入 nginx 的目录中

```shell
cd /usr/local/nginx/sbin
```



## 3.1 查看 nginx 版本号

```shell
./nginx -v
```

![1637153482397](./imgs/1637153482397.png)



## 3.2 启动 nginx

```shell
./nginx
```

![1637152144855](./imgs/1637152144855.png)



## 3.3 关闭 nginx

```shell
./nginx -s stop
```

![1637153698070](./imgs/1637153698070.png)



## 3.5 重新加载 nginx

不需要重启服务器，使得修改过的配置文件（nginx.conf）生效。

```shell
./nginx -s reload
```





# 4. Nginx 的配置文件

nginx 安装目录下，其默认的配置文件都放在这个目录的 conf 目录下，而主配置文件 `nginx.conf` 也在其中，后续对 nginx 的使用基本上都是对此配置文件进行相应的修改。



## 4.1 nginx 配置文件的位置

**目录：**`/usr/local/nginx/conf/nginx.conf`

![1637154592538](./imgs/1637154592538.png)



## 4.2 配置文件的内容

配置文件中有很多 **#** 开头的表示注释内容，去掉所有以 # 开头的段落，精简之后的内容如下：

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
  
    sendfile        on;
   
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }  
}
```

> 未去注释的原生配置文件见：[nginx.conf](./配置文件/nginx.conf)



根据上述文件，可以将 nginx.conf 配置文件分为三部分：

### 4.2.1 全局块

从配置文件开始到 events 块之间的内容，主要会设置一些**影响 nginx 服务器整体运行的配置指令**，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。 

比如上面第一行配置的：

```
worker_processes  1;
```

这是 Nginx 服务器并发处理服务的关键配置，`worker_processes` 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约 。



### 4.2.2 events  块

比如上面的配置：

```
events {
    worker_connections  1024;
}
```

events 块涉及的指令主要**影响 Nginx 服务器与用户的网络连接**，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

上述例子就表示每个 work process 支持的最大连接数为 1024。

> 这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。 



### 4.2.3 http 块

```
http {
    include       mime.types;
    default_type  application/octet-stream;
  
    sendfile        on;
   
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }  
}
```

这算是 Nginx 服务器配置中最频繁的部分，**代理、缓存和日志定义**等绝大多数功能和**第三方模块**的配置都在这里。 

需要注意的是：http 块也可以包括 **http 全局块**、**server 块**。



#### 4.2.3.1 http 全局块

http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。



#### 4.2.3.2 server 块

这个块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。 

**每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。** 

而每个 server 块也分为**全局 server 块**，以及可以同时包含多个 **locaton 块**。

**1、全局 server 块**

最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

**2、location 块**

一个 server 块可以配置多个 location 块。 

 这块的主要作用是**基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。**地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。 





# 5. Nginx 配置实例—反向代理



## 5.1 反向代理实例一

### 5.1.1 实现效果

打开浏览器，使用 nginx 反向代理，在浏览器访问 `www.123.com` 直接跳转到 `127.0.0.1:8080`（linux 系统的 tomcat 主页）。

![1637238151750](./imgs/1637238151750.png)

最终实现效果图：

![1637238216042](./imgs/1637238216042.png)





### 5.1.2 准备工作

（1）在 linux 系统中安装 tomcat，使用默认端口号 8080

> 可以直接将准备好的 tomcat 安装文件压缩包放到 linux 系统中 `/usr/src/` 目录下，通过命令 `tar -xvf apache-tomcat-7.0.70.tar.gz` 解压。
>
> 解压后进入 tomcat 的 bin 目录中，执行 `./startup.sh` 启动 tomcat 服务器。
>
> 注意：运行 tomcat 需要有 jdk 环境。

启动 tomcat 服务器流程：

![1637237302806](./imgs/1637237302806.png)



（2）对外开放访问的端口

```shell
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

**查看开放的端口号** 

```shell
firewall-cmd --list-all
```

![1637152819684](./imgs/1637152819684.png)



（3）在 windows 系统中通过浏览器访问 linux 中的 tomcat 服务器

使用 **linux 系统 ip 地址 + 8080 端口号** 进行访问：

![1637237676138](./imgs/1637237676138.png)

出现上面的页面就证明准备工作圆满成功了！



### 5.1.3 具体配置

#### （1）在 windows 系统的 hosts 文件中进行域名和 ip 对应关系的配置

我的 hosts 文件位置：`C:\Windows\System32\drivers\etc`

![1637238424755](./imgs/1637238424755.png)

进入 hosts 文件中添加一条映射：

<img src="./imgs/1637238577277.png" alt="1637238577277" style="zoom: 80%;" />

> 注意这里用的是自己的 linux 主机 ip 地址。

配置完成后可以在这里初步测试，如下图：

![1637239040158](./imgs/1637239040158.png)



#### （2）在 nginx 进行请求转发的配置（反向代理配置）

找到 nginx 的配置文件（`/usr/local/nginx/conf/nginx.conf`），在其中进行如下配置：

1. 找到 server 块，需要修改的是下面红框中的两项：

   ![1637239363018](./imgs/1637239363018.png)

2. 修改成如下图：

   ![1637239484453](./imgs/1637239484453.png)

**原理：**当在 windows 主机的浏览器中访问 `www.123.com` 时，由于 hosts 文件配置了域名 `www.123.com` 和 ip `192.168.200.130` 的映射，所以请求会发送到我的 linux 主机；而 linux 主机中的 nginx 服务器配置了反向代理的路径为 `http://127.0.0.1:8080` （由于 tomcat 就在 linux 本机中），所以请求会被转发到 tomcat 服务器中。

> **注意：修改完配置文件后需要进入 nginx 的 sbin 目录中 `cd /usr/local/nginx/sbin` 执行重新加载 nginx 的命令 `./nginx -s reload`。**



#### （3）最终测试

![1637238216042](./imgs/1637238216042.png)





## 5.2 反向代理实例二

### 5.2.1 实现效果

使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中。

nginx 监听端口为 `9001`：

访问 `http://192.168.200.130:9001/edu/` 直接跳转到 `127.0.0.1:8080` 

访问 `http://192.168.200.130:9001/vod/` 直接跳转到 `127.0.0.1:8081` 



### 5.2.2 准备工作

（1）准备两个 tomcat 服务器，一个监听 8080 端口，一个监听 8081 端口。

在 `/usr/src/` 文件夹下建立两个文件夹 `tomact8080` 和 `tomcat8081` 分别放置两个 tomcat 服务器。

![1637470848147](./imgs/1637470848147.png)

把之前准备好的 tomcat 安装文件压缩包分别放置进这两个文件夹，再执行命令 `tar -xvf apache-tomcat-7.0.70.tar.gz` 解压。

> 注意：需要进入 tomcat 安装目录下的配置文件 `server.xml` 中修改其中一个 tomcat 的端口号为8081，同时还有几个其他端口号也要修改成与另一个 tomcat 服务器不同的端口号。

修改完成后分别使用 `./startup.sh` 启动两个 tomcat 服务器。

（2）创建文件夹和测试页面

在两个服务器文件夹中的 webapps 文件夹中分别建立两个文件夹 `/vod/` 和 `/edu/`，分别在里面加入一个不同的页面 html 文件，以便测试。



### 5.2.3 具体配置

#### （1）在 nginx 进行请求转发的配置（反向代理配置）

找到 nginx 的配置文件（`/usr/local/nginx/conf/nginx.conf`），在其中进行如下配置：

添加一个 server 块：

![1637471994427](D:\CodeLearning\ComputerScience\MyNote-git\Nginx\imgs\1637471994427.png)

#### （2）开放对外访问的端口号

```shell
firewall-cmd --add-port=9001/tcp --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=8081/tcp --permanent
```

> **注意：修改完配置文件后需要进入 nginx 的 sbin 目录中 `cd /usr/local/nginx/sbin` 执行重新加载 nginx 的命令 `./nginx -s reload`。**

#### （3）最终测试

![1637472283583](./imgs/1637472283583.png)

![1637472500245](./imgs/1637472500245.png)



### 5.2.4 location 指令说明

该指令用于匹配 URL。 

语法如下：

![1637472650949](./imgs/1637472650949.png)

 1、`=` ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。 

 2、`~`：用于表示 uri 包含正则表达式，并且区分大小写。 

 3、`~*`：用于表示 uri 包含正则表达式，并且不区分大小写。 

 4、`^~`：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。 

> 注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。 





