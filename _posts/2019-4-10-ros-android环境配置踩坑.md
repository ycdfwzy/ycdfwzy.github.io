***本文写于2019年4月10日，记录了笔者在配置ros-android环境的时候遇到的各种问题***

## 环境

* 操作系统：Ubuntu16.04
* Android Studio版本：3.3.2
* ROS版本：kinetic

## 搭梯子

我使用了Shadowsocks，这个配置教程比较多，可自行Google。

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

5. 可以使用命令`curl ip.gs`检查是否代理成功：如果输出类似于如下文字，则成功
   ```
   Current IP / 当前 IP: ×××.×××.×××.×××
   ISP / 运营商:  choopa.com
   City / 城市: San Jose California
   Country / 国家: United States
   ...
   ```
   

## 安装Android Studio

首先需要安装Java，ROS官网上推荐的是openjdk8，不过笔者装的是[Oracle JDK8](https://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html)，目前来看使用中并没有遇到什么问题。

Oracle JDK的安装网上也有很多，就不赘述了（记得安装完后配置好环境变量）。

然后安装Android Studio。

安装完成后，在终端打开Android Studio前，先将确保该会话已翻墙（用上一节的方法，这点很重要，因为Android Studio会尝试联网下载Gradle等工具包，但是很不幸这些网站被墙了，下载失败的话Android Studio无法正常编译运行代码）。在Android Studio中可设置HTTP代理：在File>Settings页面找到http_proxy界面设置代理模式为`Manual Proxy Configuration`，Host为`127.0.0.1`，端口为`8123`（即交由polipo代理）。设置完后可点击下方的`check connection`检查是否成功。

**在安装过程中似乎也会弹出窗口要求设置代理，使用上面类似的方法设置即可。**

其余步骤可以按照ROS[官网](http://wiki.ros.org/android/kinetic/Android%20Studio/Download)完成。

## 安装Ros Kinetic

请按照[官网](http://wiki.ros.org/kinetic/Installation/Ubuntu)的步骤安装，注意两点

1. 推荐安装Desktop-Full版本，笔者一开始装的Desktop版，官网上的开发环境安装Test依赖的Compressed包在Desktop版里没有，结果排查了好久才找到原因。
2. 注意用zsh的小伙伴配置环境的时候选择`ros/kinetic/setup.zsh`，而不是`ros/kinetic/setup.bash`。

## 安装Ros Development Environment

按照[官网](http://wiki.ros.org/android/Tutorials/kinetic/Installation%20-%20ROS%20Development%20Environment)安装core android libraries，最后一句话命令无法执行成功

   ```
   catkin_make
   ```

执行后会报这样的错误

> Could not find method google() for arguments [] on repository container of type org.gradle.api.internal.artifacts.dsl.DefaultRepositoryHandler.

最后笔者在github的[issue](https://github.com/rosjava/android_core/issues/292#issuecomment-465554790)里找到了解决方案：用` ./gradlew build --debug`代替`catkin_make`。

这样就装完了core android libraries。

最后在Testing过程中，如果出现打开摄像头就闪退的情况，大概率是应用没有打开摄像头的权限（虽然代码里已经申请过了，但是笔者试验了两台机器都没有成功拿到权限），去系统的权限管理中开一下就行了。