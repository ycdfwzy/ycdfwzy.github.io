# ros-android环境配置踩坑

***本文写于2019年4月10日，记录了我在配置ros-android环境的时候遇到的各种问题***

## 环境

* 操作系统：Ubuntu16.04
* Android Studio版本：
* ROS版本：kinetic

## 搭梯子

我使用了Shadowsocks，这个教程比较多，可自行Google。

Shadowsocks使用socks5协议，而终端很多工具目前只支持http和https等协议，所以笔者为终端设置Shadowsocks的思路就是将socks5协议转换成http协议。

1. 安装Polipo：`sudo apt-get install polipo`

2. 打开配置文件：`sudo vim /etc/polipo/config`。向config中添加如下语句

   ```
   # Uncomment this if you want to use a parent SOCKS proxy:
   # Note that 1080 is the port of shadowsocks.
   socksParentProxy = "127.0.0.1:1080"
   socksProxyType = socks5
   ```

   现在只要在某个命令前加上`http_proxy="http://127.0.0.1:8123"`设置代理

   **注：8123是Polipo的默认端口，如有需要，可以修改成其他有效端口。**

3. 如果要为当前会话设置全局代理，输入`export http_proxy="http://127.0.0.1:8123"`回车即可。输入`unset http_proxy`可撤销当前会话http_proxy代理。如果你觉得每次设置比较麻烦，可以将配置写到~/.bashrc中去（用zsh得小伙伴写到~/.zshrc中）

   ```
   export http_proxy="http://127.0.0.1:8123"
   export https_proxy="http://127.0.0.1:8123"
   export all_proxy="http://127.0.0.1:1080"
   ```

   这里，all_proxy 是指 socks5 协议的代理地址。

4. 接下来，重新启动polipo

   ```
   sudo service polipo stop
   sudo service polipo start
   ```

   

## 安装Android Studio

首先需要安装Java，ROS官网上推荐的是openjdk8，不过笔者装的是[Oracle JDK8](https://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html)，目前来看使用中并没有遇到什么问题。

Oracle JDK的安装网上也有很多，就不赘述了（记得安装好后配置好环境变量）。

然后安装Android Studio。