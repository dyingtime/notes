## STF 安装过程

### 安装相关依赖

- [Node.js](https://nodejs.org/) 8.x **required** (some dependencies don't support newer versions)
- [ADB](http://developer.android.com/tools/help/adb.html) properly set up
- [RethinkDB](http://rethinkdb.com/) >= 2.2
- [CMake](https://cmake.org/) >= 3.9 (for [node-jpeg-turbo](https://github.com/julusian/node-jpeg-turbo#readme))
- [GraphicsMagick](http://www.graphicsmagick.org/) (for resizing screenshots)
- [ZeroMQ](http://zeromq.org/) libraries installed
- [Protocol Buffers](https://github.com/google/protobuf) libraries installed
- [yasm](http://yasm.tortall.net/) installed (for compiling embedded [libjpeg-turbo](https://github.com/devicefarmer/node-jpeg-turbo))
- [pkg-config](http://www.freedesktop.org/wiki/Software/pkg-config/) so that Node.js can find the libraries

### 安装 NVM

```shell
# 下载脚本
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
# 根据情况修改 ~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
# 安装指定版本npm
nvm install 8.16.0
```

### 安装其它依赖

```shell
# mac brew 直接安装
brew install rethinkdb graphicsmagick zeromq protobuf yasm pkg-config cmake android-tools-adb
# ubuntu apt 安装
# 添加rethinkdb源
source /etc/lsb-release && echo "deb https://download.rethinkdb.com/repository/ubuntu-$DISTRIB_CODENAME $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/repository/raw/pubkey.gpg | sudo apt-key add -
sudo apt-get update
apt install  rethinkdb graphicsmagick yasm pkg-config cmake android-tools-adb libzmq3-dev
```

### 安装 STF

```shell
# 注意是@devicefarmer/stf,不是stf,stf是旧版,已经不维护了
npm install -g @devicefarmer/stf
# unrecognized option `-static' 错误解决方式
# 确定 libtool 路径是否是/usr/bin/libtool,如果不是则移除,https://github.com/nodejs/node/issues/2341
which libtool
brew unlink libtool
rm -rf /usr/local/bin/libtool
```

### 启动相关服务

```shell
# 先启动rethinkdb,--daemon后台启动
rethinkdb --daemon
# 再启动 stf local,--public-ip 设置访问的IP, --allow-remote 允许通过wifi连接adb
stf local --public-ip <your_internal_network_ip_here> --allow-remote
```

### 二次开发

```shell
# 获取代码
git clone https://github.com/DeviceFarmer/stf.git
# 安装依赖并链接到stf
cd stf && npm install && npm link
# 修改代码后重复发布,不用重启stf
npm run prepublish
```

### docker 安装 STF

```shell
# 先启动一个数据库
docker run -d --name rethinkdb -v /srv/rethinkdb:/data --net host rethinkdb rethinkdb --bind all --cache-size 8192 --http-port 8090
# 再启动 adb service
docker run -d --name adbd --privileged -v /dev/bus/usb:/dev/bus/usb --net host sorccu/adb:latest
# 再启动 stf 启动
docker run -d --name stf --net host openstf/stf stf local --public-ip 192.168.1.100
```

