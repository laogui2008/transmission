# transmission-3.00_skip-hash-chek

这是基于 [Transmission 3.00](https://github.com/transmission/transmission/tree/3.00) 源码同时仿照 [blackyau/Transmission_SkipHashChek](https://github.com/blackyau/Transmission_SkipHashChek) 制作而成的可跳过校验的修改版。请注意:**使用修改版客户端 跳过校验 可能会导致您的账号被封禁 ，请你务必了解 跳过校验 后带来的风险**。

正在使用手机浏览本页面的用户 [请点击这里](https://github.com/wenyutools/transmission/blob/skip-hash-check/README.md)，查看完整的说明文档。

## 下载编译

[想要手动修改的看这里(较复杂)](https://github.com/blackyau/Transmission_SkipHashChek/blob/master/ModifyCodeYourself.md)

### 安装依赖

#### CentOS 7

```Shell
yum install gcc gcc-c++ m4 make automake libtool gettext openssl-devel libcurl-devel libevent-devel intltool gtk3-devel wget vim zip unzip
```

#### Ubuntu/Debian 18.04
```Shell
apt-get install -y build-essential automake autoconf libtool pkg-config intltool libcurl4-openssl-dev libglib2.0-dev libevent-dev libminiupnpc-dev libgtk-3-dev libappindicator3-dev ca-certificates libssl-dev pkg-config cmake openssl libssl1.0-dev zip unzip vim
```

#### Ubuntu/Debian 20.04
```Shell
apt-get install -y build-essential automake autoconf libtool pkg-config intltool libcurl4-openssl-dev libglib2.0-dev libevent-dev libminiupnpc-dev libgtk-3-dev libappindicator3-dev ca-certificates libssl-dev pkg-config cmake openssl libssl-dev zip unzip vim
```

### 下载源码

```Shell
git clone -b skip-hash-check https://github.com/wenyutools/transmission.git transmission-3.00_skip-hash-chek
```

### 解压编译

autogen.sh的编译配置说明
这里autogen.sh末尾处使用的configure选项是启用cli, 禁用gtk client：
```Shell
 ./configure --enable-cli --enable-daemon --with-inotify --enable-nls --without-gtk
```
如果你是编译给内存和cpu处理资源比较少的设备，那么可以修改成下面这条，以节省内存和cpu资源：
```Shell
 ./configure --enable-lightweight --enable-cli --enable-daemon --with-inotify --enable-nls --without-gtk
```

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' '/usr/local/share' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.

默认的安装prefix是 /usr/local/
所以自己编译出来的安装和启动位置是 
/usr/local/bin/transmission-daemon
/usr/local/share/transmission-web
可以给configure加参数 --prefix来修改安装位置。
configure的全部选项可以用./configure --help查看 

开始编译
```Shell
cd transmission-3.00_skip-hash-chek
git submodule update --init
chmod +x autogen.sh
./autogen.sh
make
sudo make install
```

检查编译安装的版本：
```Shell
/usr/local/bin/transmission-daemon -V
```
你应该得到的是
transmission-daemon 3.00 (commit-hash)

另外需要指出的是自己刚编译好的文件，还带有开发调试符号，因此文件会比官方发行版本大很多，可以用strip命令来移除这些开发调试符号
编译版:
-rwxr-xr-x 1 root root 4.1M Jan 13 11:10 /usr/local/bin/transmission-daemon
发行版:
-rwxr-xr-x 1 root root 531K Mar 25  2020 /usr/bin/transmission-daemon
这样就和发行版一样了:
```Shell
sudo strip /usr/local/bin/transmission-daemon
```

## 如何配置

在你编译安装完毕后，还需要一定的配置才能够使用。请注意:以下配置只适用于 PT([Private Tracker](https://www.baidu.com/s?wd=pt%E4%B8%8B%E8%BD%BD)) 不适用于 BT 。

```Shell
说明:
如果你是非systemd的老设备而且懒得自己写启动脚本，开机启动之类的，最简单的配置方法是:
sudo apt install transmission-daemon
sudo /etc/init.d/transmission-daemon stop
sudo vi /etc/init.d/transmission-daemon
把 DAEMON=/usr/bin/$NAME
改成 DAEMON=/usr/local/bin/$NAME
这样相当于使用了官方的启动脚本，开机它这个脚本是启动自己编译的版本，而不是apt安装的版本。
如果你这样做，就可以省去下面自己写脚本之类的。
```

systemd启动模式的配置:

### 创建启动脚本

```Shell
sudo vim /etc/systemd/system/transmission.service
写入以下内容
-g 是指定transmission-daemon的配置文件路径

[Unit]
Description=Transmission BitTorrent Daemon
After=network.target
 
[Service]
User=root
LimitNOFILE=100000
ExecStart=/usr/local/bin/transmission-daemon -f --log-error -g /usr/local/transmission
 
[Install]
WantedBy=multi-user.target
```

### 设置脚本权限并设置开机自启

```Shell
sudo chmod +x /etc/systemd/system/transmission.service
sudo systemctl daemon-reload
sudo systemctl enable transmission
```

### 启动 Transmission 生成默认配置文件

```Shell
sudo systemctl start transmission.service
```

### 关闭 Transmission 否则配置文件修改不会生效

```Shell
sudo systemctl stop transmission.service
```

### 修改配置文件

```Shell
sudo vim /usr/local/transmission/settings.json
```

### 根据自己情况修改以下选项

```json
"cache-size-mb": 512, #缓存大小，单位MB，建议设置内存大小的1/6~1/4
"dht-enabled": false, #启用DHT网络（通过tracker寻找节点）
"download-dir": "/var/lib/transmission/Downloads",  #下载完后文件存放目录
"incomplete-dir": "/var/lib/transmission/Downloads",  #正在下载的文件目录
"incomplete-dir-enabled": true, #启用正在下载的文件的保存路径
"peer-limit-global": 100000, #全局种子最大连接数
"peer-limit-per-torrent": 100, #每个种子最多连接数(根据你的带宽自行决定)
"peer-port": 51413, #传入端口号(建议调成和默认的不一样 10000-65535 皆可)
"pex-enabled": false, #节点交换
"preallocation": 0, #预分配文件磁盘空间，0=关闭，1=快速，2=完全，默认取1这里选0是为了能够快速完成下载
"rpc-authentication-required": true, #远程连接密码验证 不开的话谁都可以进后台
"rpc-password": "password", #这里是远程连接的密码 保存配置文件后这里会被加密不显示明文
"rpc-port": 9091, #网页GUI的端口
"rpc-username": "whsir",#远程连接的用户名
"rpc-whitelist-enabled": false,  #这里一定要禁用白名单 启用的话就只有在白名单里面的 ip 才能访问网页
"script-torrent-done-enabled": false, #在torrent完成时运行脚本，默认关闭
"script-torrent-done-filename": "", #脚本路径
"speed-limit-down": 100, #下载速度限制默认100KB/s
"speed-limit-down-enabled": false,#启用下载速度限制。默认关闭
"speed-limit-up": 100, #上传速度限制，默认100KB/s
"speed-limit-up-enabled": false, #启用上传速度限制。默认关闭
"start-added-torrents": true,#添加种子文件后，自动开始，如果为false，添加种子后不会自动开始
"trash-original-torrent-files": false,#是否删除监控目录添加的种子文件，也就是说在watch-dir监控的目录下添加种子文件后，任务开始后会自动删除添加的种子文件。watch-dir需要手动添加，在最下面。
"upload-slots-per-torrent": 30,#每个种子上传连接数(根据你的带宽自行决定)
"utp-enabled": true #UTP传输是否启用
#下面两个需要手动添加的选项，注意每行配置参数都是以逗号结尾，最后一行参数没有逗号（添加下面的参数一定要注意上面最后一行要以逗号结尾，例如"utp-enabled": true,）
#"watch-dir": "/root/test", #自动监控种子目录，将种子文件下载或放在此文件夹下，会自动开始下载文件
#"watch-dir-enabled": true #是否开启自动监控种子目录
```

配置文件中的更多参数设置可参考:

[GitHub - 官方wiki (英语)](https://github.com/transmission/transmission/wiki/Editing-Configuration-Files)

[吴昊博客 - Transmission 2.92 配置文件参数中文解释](https://blog.whsir.com/post-1182.html)

[360doc - 超详细的Transmission 2.31 |迅雷离线PT下载交流](http://www.360doc.com/content/14/0311/08/15024809_359678743.shtml)

如果你懒得一个一个调，这是我推荐的完整设置。对于一个100M的盒子来说，应该比较合适。

```json
{
    "alt-speed-down": 50,
    "alt-speed-enabled": false,
    "alt-speed-time-begin": 540,
    "alt-speed-time-day": 127,
    "alt-speed-time-enabled": false,
    "alt-speed-time-end": 1020,
    "alt-speed-up": 50,
    "bind-address-ipv4": "0.0.0.0",
    "bind-address-ipv6": "::",
    "blocklist-enabled": false,
    "blocklist-url": "http://www.example.com/blocklist",
    "cache-size-mb": 512,#缓存大小 单位MB,建议设置内存大小的1/6~1/4.如果你在同时运行多个PT程序,请酌情调低.
    "dht-enabled": false,
    "download-dir": "/root/Downloads",
    "download-queue-enabled": true,
    "download-queue-size": 3,
    "encryption": 1,
    "idle-seeding-limit": 30,
    "idle-seeding-limit-enabled": false,
    "incomplete-dir": "/root/Downloads",
    "incomplete-dir-enabled": true,
    "lpd-enabled": false,
    "message-level": 1,
    "peer-congestion-algorithm": "",
    "peer-id-ttl-hours": 6,
    "peer-limit-global": 100000,
    "peer-limit-per-torrent": 100,
    "peer-port": 51413,
    "peer-port-random-high": 65535,
    "peer-port-random-low": 49152,
    "peer-port-random-on-start": false,
    "peer-socket-tos": "default",
    "pex-enabled": false,
    "port-forwarding-enabled": true,
    "preallocation": 0,
    "prefetch-enabled": true,
    "queue-stalled-enabled": true,
    "queue-stalled-minutes": 30,
    "ratio-limit": 2,
    "ratio-limit-enabled": false,
    "rename-partial-files": true,
    "rpc-authentication-required": true,
    "rpc-bind-address": "0.0.0.0",
    "rpc-enabled": true,
    "rpc-host-whitelist": "",
    "rpc-host-whitelist-enabled": true,
    "rpc-password": "kJrk15kLooWBzP3", #这里是远程连接的密码,保存配置文件后这里会被加密不显示明文
    "rpc-port": 9091, #网页GUI的端口
    "rpc-url": "/transmission/",
    "rpc-username": "user", #远程连接的用户名
    "rpc-whitelist": "127.0.0.1",
    "rpc-whitelist-enabled": false,
    "scrape-paused-torrents-enabled": true,
    "script-torrent-done-enabled": false,
    "script-torrent-done-filename": "",
    "seed-queue-enabled": false,
    "seed-queue-size": 10,
    "speed-limit-down": 100,
    "speed-limit-down-enabled": false,
    "speed-limit-up": 100,
    "speed-limit-up-enabled": false,
    "start-added-torrents": true,
    "trash-original-torrent-files": false,
    "umask": 18,
    "upload-slots-per-torrent": 50,
    "utp-enabled": true
}
```

### 启动 Transmission

```Shell
systemctl start transmission.service
```

### 开启防火墙端口

**下面开启的端口分别为上面配置中的 rpc-port 和 peer-port 如果自己修改过配置，也依葫芦画瓢改改命令**

CentOS 7:

```Shell
firewall-cmd --permanent --add-port=9091/tcp --add-port=51413/tcp --add-port=51413/udp
firewall-cmd --reload
```

Ubuntu 18.04/20.04:

```Shell
sudo ufw allow 9091
sudo ufw allow 51413
```

### 安装增强 Transmission GUI

如果安装有问题或不会请移步至 [https://github.com/ronggang/transmission-web-control](https://github.com/ronggang/transmission-web-control)

```Shell
wget -N https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control-cn.sh --no-check-certificate
chmod +x install-tr-control-cn.sh
bash install-tr-control-cn.sh
```

### 如何跳过校验

打开网页GUI `http://vps 的 ip:9091` 
在使用增强 Transmission GUI 的环境下,你只需要在左边勾选(只能单选)你需要跳过校验的种子,然后点击 获取更多Peer 即可跳过校验。
在原版 Transmission GUI 的环境下你可以右键任意种子，然后点击 Ask tracker for more peers 即可跳过校验。

## 参考

[GitHub - transmission/transmission](https://github.com/transmission/transmission)

[GitHub - blackyau/Transmission_SkipHashChek](https://github.com/blackyau/Transmission_SkipHashChek)

[GitHub - superlukia/transmission-2.92_skiphashcheck](https://github.com/superlukia/transmission-2.92_skiphashcheck)

[GitHub - transmission Wiki Building Transmission](https://github.com/transmission/transmission/wiki/Building-Transmission)

[GitHub - transmission Wiki Editing Configuration Files](https://github.com/transmission/transmission/wiki/Editing-Configuration-Files)

[一曲长歌辞烟雨 - CentOS编译安装Transmission](https://archive.ls12.me/centos-install-transmission.html)

[吴昊博客 - Transmission2.94最新版完整编译安装并汉化](https://blog.whsir.com/post-2881.html)

[吴昊博客 - Transmission2.92配置文件参数中文解释](https://blog.whsir.com/post-1182.html)

[360doc - 超详细的Transmission 2.31 |迅雷离线PT下载交流](http://www.360doc.com/content/14/0311/08/15024809_359678743.shtml)

[91yun - 在VPS上离线下载PT/BT/磁力链:CentOS下transmission安装教程](https://www.91yun.co/archives/31861)

[简书 - 源码编译安装Transmission 2.93（debian 7)](https://www.jianshu.com/p/551ed5464e81)

The original English readme:

## About

Transmission is a fast, easy, and free BitTorrent client. It comes in several flavors:
  * A native Mac OS X GUI application
  * GTK+ and Qt GUI applications for Linux, BSD, etc.
  * A headless daemon for servers and routers
  * A web UI for remote controlling any of the above

Visit https://transmissionbt.com/ for more information.

## Command line interface notes

Transmission is fully supported in transmission-remote, the preferred cli client.

Three standalone tools to examine, create, and edit .torrent files exist: transmission-show, transmission-create, and transmission-edit, respectively.

Prior to development of transmission-remote, the standalone client transmission-cli was created. Limited to a single torrent at a time, transmission-cli is deprecated and exists primarily to support older hardware dependent upon it. In almost all instances, transmission-remote should be used instead.

Different distributions may choose to package any or all of these tools in one or more separate packages.

## Building

Transmission has an Xcode project file (Transmission.xcodeproj) for building in Xcode.

For a more detailed description, and dependencies, visit: https://github.com/transmission/transmission/wiki

### Building a Transmission release from the command line

    $ tar xf transmission-2.92.tar.xz
    $ cd transmission-2.92
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make
    $ sudo make install

### Building Transmission from the nightly builds

Download a tarball from https://build.transmissionbt.com/job/trunk-linux/ and follow the steps from the previous section.

If you're new to building programs from source code, this is typically easier than building from Git.

### Building Transmission from Git (first time)

    $ git clone https://github.com/transmission/transmission Transmission
    $ cd Transmission
    $ git submodule update --init
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make
    $ sudo make install

### Building Transmission from Git (updating)

    $ cd Transmission/build
    $ make clean
    $ git pull --rebase --prune
    $ git submodule update
    $ cmake ..
    $ make
    $ sudo make install

## Contributing

### Code Style

You would want to setup your editor to make use of uncrustify.cfg and .jsbeautifyrc configuration files located in the root of this repository.

If for some reason you are unwilling or unable to do so, there is a shell script which you could run either directly or via docker-compose:

    $ ./code_style.sh
    or
    $ docker-compose build --pull
    $ docker-compose run --rm code_style
