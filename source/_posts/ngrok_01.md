---
layout:     post                    # 使用的布局（不需要改）
title:      利用ngrok实现内网穿透         # 标题
subtitle:   xiangjie                       #副标题
date:       2018-03-19              # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - ngrok
---

实现内网穿透紫ngrok无法通过天墙之后，国内也出现了一批成熟的商业化实现方案，诸如花生壳、net123、Sunny-ngrok等。不过免费的极不稳定还有流量带宽限制，最后还是决定自己搭一个。本文利用ngrok搭建一个用于内网穿透的环境。需求是通过一层反向代理，实现通过一个外网域名访问一个部署在局域网上的服务。

> ## 准备

- #### 一个公网服务器（Linux系统）==> 阿里云，腾讯云之类的都行
  - 这个公网服务器主要用作反向代理，我们在本文中称之为VPS服务器
- #### 一个独立的域名
- #### 一个用于提供服务的本地PC

> ## 步骤

- ### GO语言环境搭建
  ngrok项目是用GO语言实现的，需要先安装GOLANG开发环境，系统不限，因为GO语言是跨平台的！安装过程很简单，参考[官网的教程](https://golang.org/dl/)即可！
- ### 获取ngrok源码
  ```bash
  # 下面是直接去ngrok的github地址下载（待会儿make的时候还会需要装几个其它的依赖，可能会出现很多问题）
  git clone https://github.com/inconshreveable/ngrok.git

  # 下面是笔者将代码clone下来，并添加了相应依赖之后的地址，如果用上面的方式出现错误，可以clone下面的地址
  git clone https://github.com/SunnyQjm/ngrok.git
  ```
- ### 解析域名
  因为我们自己搭建，需要使用自己的域名（以 test.j.cn ）为例，我们需要做以下解析：
  ```bash
  test.j.cn    ------------> A记录到你的VPS服务器的IP

  # ngrok可以指定子域名，下面的解析方式可以让任意子域名都能得到正确的解析
  *.test.j.cn  ------------> CNAME记录到 test.j.cn
  ```
- ### 生成签名证书
  > - 因为我们是自己搭建，就不能用ngrok官方的SSL证书，需要自己生成
  > - 下面**生成**的**证书**在编译项目的时候要用到，所以**务必要在编译之前生成**
  > - 需要注意的是，**客户端和服务器的证书必须是同一份**，这样在程序在认证的时候才能正确解析

  ```bash
  # 首先导出环境变量，将下面的值替换成你的域名
  export NGROK_DOMAIN="test.j.cn"

  #先进入到ngrok的根目录，生成证书的操作需要在根目录下进行
  cd ngrok

  # 下面的命令用于生成证书
  openssl genrsa -out rootCA.key 2048
  openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
  openssl genrsa -out device.key 2048
  openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
  openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

  # 下面的命令用于将我们生成的证书替换ngrok默认的证书
  cp rootCA.pem assets/client/tls/ngrokroot.crt
  cp device.crt assets/server/tls/snakeoil.crt
  cp device.key assets/server/tls/snakeoil.key
  ```
- ### 编译运行
  - #### 服务端（VPS服务器一端）
    - 首先指定一下环境变量，在不同的操作系统下需要指定不同的环境变量，才能正确编译(默认是Linux 64位的配置，如果你的服务器是64位的Linux系统，也可以不指定，直接用默认的就行)
      ```bash
      GOOS=linux GOARCH=amd64
      #如果是32位系统，这里 GOARCH=386
      #如果是windows系统，GOOS=windows
      ```
    - 然后make出服务端程序
      ```bash
      make release-server
      ```
    - 如果编译成功，你会在bin目录下看到ngrokd程序
      ```bash
      cd bin

      # 查看使用帮助
      ./ngrokd -h

      ```
    - 查看使用帮助
      ```bash
      # 查看使用帮助
      ./ngrokd -h
        -domain string
          	Domain where the tunnels are hosted (default "ngrok.com")
        -httpAddr string
          	Public address for HTTP connections, empty string to disable (default ":80")
        -httpsAddr string
          	Public address listening for HTTPS connections, emptry string to disable (default ":443")
        -log string
          	Write log messages to this file. 'stdout' and 'none' have special meanings (default "stdout")
        -log-level string
          	The level of messages to log. One of: DEBUG, INFO, WARNING, ERROR (default "DEBUG")
        -tlsCrt string
          	Path to a TLS certificate file
        -tlsKey string
          	Path to a TLS key file
        -tunnelAddr string
          	Public address listening for ngrok client (default ":4443")
      ```
    - 启动服务端
      ```bash
      # 如果不能执行，你可能需要用 sudo chmod +x ngrokd 给它执行权限
      # domain域输入之前生成证书时指定的域名
      # httpAddr 指定转发http协议的哪个端口
      # httpAddrs 指定转发https协议的哪个端口（如果不需要可以省略）
      ./ngrokd  -domain="$NGROK_DOMAIN" -httpAddr=":8000" -httpsAddr=":4433"
      ```
    - 如果执行成功，你会看到类似以下界面：
      ```bash
      [16:23:40 CST 2018/03/19] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [registry] [tun] No affinity cache specified
      [16:23:40 CST 2018/03/19] [INFO] (ngrok/log.Info:112) Listening for public http connections on [::]:9748
      [16:23:40 CST 2018/03/19] [INFO] (ngrok/log.Info:112) Listening for public https connections on [::]:443
      [16:23:40 CST 2018/03/19] [INFO] (ngrok/log.Info:112) Listening for control and proxy connections on [::]:4443
      [16:23:40 CST 2018/03/19] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [metrics] Reporting every 30 seconds
      ```
    - 自此，服务端算是配置好了
  - #### 客户端（部署了服务，需要内网穿透访问的主机）
    > 客户端要做的事情就是指定把本机的那个端口暴露给VPS服务器，客户端要在该端口上部署了服务（web服务或者tomcat服务等），这样VPS就能够将请求转发到该端口对应的服务上了

    ``` bash
    # 因为前面说过，服务端和客户端的证书要是同一份，这样认证才能通过，所以好的解决方案是在服务端上把客户端程序也编译出来，然后通过scp命令拷贝到客户端
    # 假设我要在mac上运行客户端，需要在编译命令前加上一些参数(如果客户端服务器也是Linux 64位，则不用指定环境变量)
    GOOS=darwin GOARCH=amd64 make release-client
    make release-client

    # 编译好后scp到本地
    scp xxx xxx

    # 下面开始本地配置（下面的配置在客户端进行）

    # 新建一个配置文件（在ngrok/bin目录下）
    vim ngrok.cfg

    # 添加一下两行
    # 第一行是将要绑定的域名+4443端口（因为ngrok服务端默认有一个服务是坚挺在4443端口的，客户端会通过这个端口与之相连）==> 记得将域名换成自己在生成证书时指定的
    server_addr: "test.j.cn:4443"
    trust_host_root_certs: true

    # 帮助信息
    ./ngrok -h
    Examples:
        ngrok 80
        ngrok -subdomain=example 8080
        ngrok -proto=tcp 22
        ngrok -hostname="example.com" -httpauth="user:password" 10.0.0.1

    # 80就是我们要转发的端口了
    ./ngrok -config=./ngrok.cfg 80

    # 指定协议和端口，不指定默认是 http+https
    ./ngrok -config=./ngrok.cfg -proto=tcp 22

    # 指定子域名，不指定就会随机生成
    ./ngrok -config=./ngrok.cfg -subdomain=test 80
    ```
  - 连接成功会显示如下状态：
    - {% img /img/ngrok_01_1.png 连接成功%}
    - 在浏览器中输入http://127.0.0.1:4040，就可以看到请求的具体信息了！是不是很神奇！

- ### 注意：
  - 上文中所有出现域名的地方都要统一，客户端和服务端的证书要是同一份，否则在连接的时候会出现bad certification

- ### 参考：
  - [自己建立ngrok服务器进行内网穿透](https://blog.phpgao.com/ngrok_how_to.html)
  - [CentOS上搭建ngrok服务端（内网穿透）](http://zpblog.cn/linux/run-ngrok-on-your-own-server.html)
