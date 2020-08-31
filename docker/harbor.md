# Harbor 安装过程

### 安装 `docker-compose`

```shell
# 下载对应版的 docker-compose 到指定目录
curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 添加可执行权限
chmod +x /usr/local/bin/docker-compose
# 把 /usr/local/bin 添加到root用户的PATH
export PATH=$PATH:/usr/local/bin
```

`curl -L` 跟随重定向

`uname -s` 输出系统内核版本

`uname -m` 输出系统主机的硬件架构名称

### 下载安装包

```shell
# 在线安装包,从docker hub下载镜像,需要联网
wget https://storage.googleapis.com/harbor-releases/harbor-online-installer-v1.6.1.tgz
# 离线安装包,包含预构建的镜像,不需要联网
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.6.1.tgz
```

[更多版本下载地址](https://github.com/goharbor/harbor/releases) 

`Harbor 1.6.1`版本支持`docker version 1.13.1`(yum 安装的版本),更高的版本可能需要更新`docker`版本

### 解压安装包

```sh
# 解压并进入目录
tar -xzf harbor-online-installer-v1.9.1.tgz && cd harbor
```

### 配置安装的相关参数

```properties
# vim harbor.cfg
# 配置私有仓库的访问IP或者域名,不能是localhost/127.0.0.1
# 若docker-compose.yml中端口修改,这里需要指定端口
hostname=192.168.187.128
```

### 修改`docker`配置

```json
// vim /etc/docker/daemon.json
// 需要先关闭docker, 修改完毕重新启动docker
{
	"insecure-registries":[
        // 若端口修改,这里需要加端口
        "192.168.187.128"
    ]
}
```

### 运行`install.sh`安装

```shell
# 需要root权限?
./installsh
```

### 登录到私有仓库

```shell
# 若端口修改,这里需要加端口
docker login 192.168.187.128
```

