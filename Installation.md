##BigBlueButton 1.1笔记
BigBlueButton是一套开源的视频会议系统，适用于视频会议、远程教育，可以理解为常见的直播平台。

###Installation

**这套系统对于硬件的要求较高。**
- 4GB的内存
- CPU主频2.6GHZ以上
- 500G的空余硬盘空间

**对于系统的要求如下：**
- Ubuntu 16.04 64-bit OS

**其他要求**
- 100M的带宽
- 80端口不被占用

**本地化设置**
```shell
$ cat /etc/default/locale
LANG="en_US.UTF-8"
```
如果没有如上的输出，那么可以通过下面的命令来设置本地化
``` shell
$ sudo apt install language-pack-en
$ sudo update-locale LANG=en_US.UTF-8
```
**检查本地的内存容量是够达到要求**
```
$ free -h
```
**查看系统信息**
```
$ cat /etc/lsb-release
```
**检查系统是否是64位**
```
$ uname -m
```

**设置主机名(hostname)和SSL证书(SSL certificate)**
(如果你是一个开发人员，只是在本地的虚拟环境中使用BBB，那么你可以跳过这一步)
官方文档建议给BBB设置一个完整的域名(FQDN)，比如`bigbluebutton.example.com`，并且配置一个SSL证书。这些配置才能使nginx(BBB自带)使用HTTPS协议提供服务。如果不使用HTTPS协议，有些浏览器可能不兼容(比如chrome)不能使用麦克风等。而且，不支持HTTPS协议，有些浏览器会报出关于内容安全性的警告。
简言之，任何服务上线后，都应该设置一个完整的域名和SSL证书
获取域名自不必说
获取SSL证书：在BBB系统安装完成后，[获取SSL证书模块](http://docs.bigbluebutton.org/install/install.html#obtain-an-ssl-certificate)有详尽的信息。

###进行安装

####第一步 升级服务
首先确保你的账号有足够的权限来做下面的事，账号无误的话，我们需要给系统添加更新源

先检查更新源中是否有`xenail multiverse`
`$ grep "multiverse" /etc/apt/sources.list`
如果输出结果中是否含有下面的内容
```
deb http://archive.ubuntu.com/ubuntu xenial multiverse
```
或者
```
deb http://archive.ubuntu.com/ubuntu xenial main restricted universe multiverse
```

如果没有，可以用如下语句添加：

```
$ echo "deb http:///archive.ubuntu.com/ubuntu/ xenial multiverse" | sudo tee -a /etc/apt/sources.list
```

###第二步 安装BigBlueButton仓库的app-key
所有的BigBlueButton的包都有一个公钥，所以要安装这个，命令如下：
```
$ wget http://ubuntu.bigbluebutton.org/repo/bigbluebutton.asc -O- | sudo apt-key add -
```
下面配置更新源
```
$ echo "deb http://ubuntu.bigbluebutton.org/xenial-110/ bigbluebutton-xenial main" | sudo tee /etc/apt/sources.list.d/bigbluebutton.list
```
然后更新系统，执行如下命令
```
$ sudo apt update
```

###第三部　安装BigBlueButton
使用包管理器apt进行安装，很简单
```
$ sudo apt install bigbluemindbutton
```
安装可能比较慢，文件大约一个G左右，耐心等待，安装完成后，执行如下命令，启动服务
```
$ sudo bbb-conf --restart
```
查看BigBlueButton的配置信息
```
$ sudo bbb-conf --restart
```
