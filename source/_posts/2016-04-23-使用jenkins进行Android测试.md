---
title: 使用jenkins进行Android测试
date: 2016-04-23 09:02:45
tags: 
    - Android
    - Jenkins
category: Android
---
服务器使用的是DigitalOcean，Debian
# 安装Jenkins
添加apt key和source list
```sh
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add -
echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list
apt-get update
```
然后安装Jenkins
```sh
apt-get install jenkins
```
之后使用8080端口就可以打开jenkins了，第一步是设置密码，之后就是图像界面的操作了
在图形界面安装Git Plugin和Gradle Plugin这两个插件
为了安全起见，还要配置Jenkins的登录密码
# 配置主机环境
## 安装Git
```
sudo apt-get install git-core
```
## 安装Android SDK
在<http://developer.android.com/intl/zh-cn/sdk/index.html>找到command line tools下载地址，因是主机是Linux，所以下载Linux版本
```sh
cd /opt
wget <link you copied here>
```
然后解压
```
tar zxvf <filename of the just downloaded file>
```
得到android-sdk-linux目录
配置Android环境变量
```
export ANDROID_HOME="/opt/android-sdk-linux"
export PATH="$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"
source /etc/profile
```
至此，只是安装了SDK Manager，下面开始安装SDK
由于使用ssh所以只能在命令行下安装
使用
```sh
android update sdk --no-ui
```
可以安装所有的SDK，一只yes就可以，但是最低档的vps只有20G的空间，所有我们只需要安装最基本的SDK就可以
查看一下Android的帮助选择
```sh
# android -h

       Usage:
       android [global options] action [action options]
       Global options:
  -h --help       : Help on a specific command.
  -v --verbose    : Verbose mode, shows errors, warnings and all messages.
     --clear-cache: Clear the SDK Manager repository manifest cache.
  -s --silent     : Silent mode, shows errors only.

                                                                    Valid
                                                                    actions
                                                                    are
                                                                    composed
                                                                    of a verb
                                                                    and an
                                                                    optional
                                                                    direct
                                                                    object:
-    sdk              : Displays the SDK Manager window.
-    avd              : Displays the AVD Manager window.
-   list              : Lists existing targets or virtual devices.
-   list avd          : Lists existing Android Virtual Devices.
-   list target       : Lists existing targets.
-   list device       : Lists existing devices.
-   list sdk          : Lists remote SDK repository.
- create avd          : Creates a new Android Virtual Device.
-   move avd          : Moves or renames an Android Virtual Device.
- delete avd          : Deletes an Android Virtual Device.
- update avd          : Updates an Android Virtual Device to match the folders
                        of a new SDK.
- create project      : Creates a new Android project.
- update project      : Updates an Android project (must already have an
                        AndroidManifest.xml).
- create test-project : Creates a new Android project for a test package.
- update test-project : Updates the Android project for a test package (must
                        already have an AndroidManifest.xml).
- create lib-project  : Creates a new Android library project.
- update lib-project  : Updates an Android library project (must already have
                        an AndroidManifest.xml).
- create uitest-project: Creates a new UI test project.
- update adb          : Updates adb to support the USB devices declared in the
                        SDK add-ons.
- update sdk          : Updates the SDK by suggesting new platforms to install
                        if available.

```
我们要使用的是update sdk命令
```sh
# android -h update sdk

       Usage:
       android [global options] update sdk [action options]
       Global options:
  -h --help       : Help on a specific command.
  -v --verbose    : Verbose mode, shows errors, warnings and all messages.
     --clear-cache: Clear the SDK Manager repository manifest cache.
  -s --silent     : Silent mode, shows errors only.

                     Action "update sdk":
  Updates the SDK by suggesting new platforms to install if available.
Options:
     --proxy-port: HTTP/HTTPS proxy port (overrides settings if defined)
     --proxy-host: HTTP/HTTPS proxy host (overrides settings if defined)
  -s --no-https  : Uses HTTP instead of HTTPS (the default) for downloads.
  -a --all       : Includes all packages (such as obsolete and non-dependent
                   ones.)
  -f --force     : Forces replacement of a package or its parts, even if
                   something has been modified.
  -u --no-ui     : Updates from command-line (does not display the GUI)
  -p --obsolete  : Deprecated. Please use --all instead.
  -t --filter    : A filter that limits the update to the specified types of
                   packages in the form of a comma-separated list of
                   [platform, system-image, tool, platform-tool, doc, sample,
                   source]. This also accepts the identifiers returned by
                   'list sdk --extended'.
  -n --dry-mode  : Simulates the update but does not download or install
                   anything.
```
安装SDK的命令就是
```sh
android update sdk -u -a -t <type>
```
使用命令
```sh
android list sdk -u -a -e
```
可以查看所有SDK
开始安装
```sh
android update sdk -u -a -t tools
android update sdk -u -a -t platform-tools
android update sdk -u -a -t build-tools-23.0.3
android update sdk -u -a -t android-23
android update sdk -u -a -t extra-android-support
```
如果是64位系统
```sh
sudo apt-get install lib32stdc++6 lib32z1
```
赋予可执行权限
```sh
sudo chmod -R 755 /opt/android-sdk-linux
```
重启一下
```sh
sudo shutdown -r now
```
编译的时候遇到了 Could not find tools.jar 
所以要配置一下java
查看是32位还是64位的系统
```sh
uname -m
```
下载最新的jdk，地址到官网去找
```sh
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.tar.gz
mkdir /opt/jdk
tar -zxf jdk-8u5-linux-x64.tar.gz -C /opt/jdk
tar -zxf jdk-8u91-linux-x64.tar.gz -C /opt/jdk
update-alternatives --install /usr/bin/java java /opt/jdk/jdk1.8.0_91/bin/java 1052
update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk1.8.0_91/bin/javac 1052
```
如果主机上已经安装过其他的java版本
使用
```sh
update-alternatives --display java
update-alternatives --display javac
```
查看一下priority，确保update-alternatives --install后面的数字大于旧的的java

至此，Jenkins和Android SDK已经配置完毕，悲剧的是最终没有运行成功，服务器内存不足，最终编译失败。

参考：
<https://www.digitalocean.com/community/tutorials/how-to-build-android-apps-with-jenkins>
<https://www.digitalocean.com/community/tutorials/how-to-manually-install-oracle-java-on-a-debian-or-ubuntu-vps>

