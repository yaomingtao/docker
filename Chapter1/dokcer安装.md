# 第1节：dokcer安装

## centos安装

### 系统要求

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 overlay2 存储层驱动）无法使用，并且部分功能可能不太稳定。

### 卸载旧版本

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 使用 yum 安装

安装依赖包

```shell
$ sudo yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。

执行下面的命令添加 yum 软件源：

```shell
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo


# 官方源
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装 Docker CE

更新 yum 软件源缓存，并安装 docker-ce。

```shell
$ sudo yum makecache fast
$ sudo yum install docker-ce
```

### 使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装，另外可以通过 --mirror 选项使用国内源进行安装：

```shell
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
# $ sudo sh get-docker.sh --mirror AzureChinaCloud
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的稳定(stable)版本安装在系统中。

### 启动 Docker CE

```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

### 建立 docker 用户组

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

建立 docker 组：

```shell
$ sudo groupadd docker
```

​	将当前用户加入 docker 组：

```shell
$ sudo usermod -aG docker $USER
```

### 测试 Docker 是否安装正确

```shell
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 镜像加速

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。国内很多云服务商都提供了国内加速器服务，例如：

	*  Azure 中国镜像 https://dockerhub.azk8s.cn
	*  阿里云加速器(需登录账号获取)
	*  七牛云加速器 https://reg-mirror.qiniu.com

本节我们以 Azure 中国镜像 https://dockerhub.azk8s.cn 为例进行介绍。

对于使用 systemd 的系统(Ubuntu 16.04+、Debian 8+、CentOS 7)，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
  ]
}
```

之后重新启动服务。

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### 检查加速器是否生效

执行 $ docker info，如果从结果中看到了如下内容，说明配置成功。

```shell
Registry Mirrors:
 https://dockerhub.azk8s.cn/
```

