### docker部署网页镜像一般流程

#### 一、前置准备

##### （一）docker安装

1、卸载旧版docker

`apt-get remove docker docker-engine docker.io containerd runc`

这一步无论之前是否安装过都必做。

2、更新软件包

`sudo apt update`

`sudo apt upgrade`

3、安装docker依赖

`apt-get install ca-certificates curl gnupg lsb-release`

4、添加GPG秘钥

`curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`

5、添加docker源

```cmd
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

6、安装

`apt-get install docker-ce docker-ce-cli containerd.io`

7、启动

`systemctl start docker`

8、安装工具并重启

`apt-get -y install apt-transport-https ca-certificates curl software-properties-common`

`service docker restart`

##### （二）docker-compose安装

注意，用传统方法安装可能会装到1版本的compose，导致报错：TypeError: kwargs_from_env() got an unexpected keyword argument ‘ssl_version‘，建议手动下载二进制包：

```
curl -L "https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 或版本1：

curl -L "https://github.com/docker/compose/releases/download/1.28.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

赋予执行权限：

```
chmod +x /usr/local/bin/docker-compose sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

检查安装是否成功：

`docker-compose -v`

#### 二、部署

##### （一）换源

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://yxzrazem.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

##### （二）从dockerhub拉取镜像

```bash
docker pull tikazyq/crawlab:latest
```

拉取后可以通过：

`docker images`

检查所有已拉取的镜象。

##### （三）编写docker-compose.yml

视具体要求编写，写在哪无所谓。

##### （四）启动

```bash
docker-compose up -d
```

##### （五）常用的命令

1、结束运行

`docker-compose down`

2、查看正在运行的容器

`docker ps -a`

3、删除容器

`docker rm <container id>`

4、删除镜像

`docker rmi <image id>`

5、进入容器内部

`docker exec -it <container id> /bin/bash`

