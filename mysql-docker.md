# Docker 安装
更详细的安装可以参考[官方文档](https://docs.docker.com/engine/install/centos/)
```shell script
# Older versions of Docker were called docker or docker-engine. If these are installed, uninstall them, 
# along with associated dependencies
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# Install the yum-utils package (which provides the yum-config-manager utility) 
# and set up the stable repository.
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install the latest version of Docker Engine and containerd, or go to the next step to install a specific version
yum install docker-ce docker-ce-cli containerd.io

# If prompted to accept the GPG key, verify that the fingerprint matches 
# 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

# Start Docker.
systemctl start docker
```

# 从`dockerhub`下载[mysql官方镜像](https://hub.docker.com/_/mysql)
```shell script
# 5.7
docker pull mysql:5.7

# 其他版本
docker pull mysql:8
docker pull mysql:5.6
```

# 创建并启动一个mysql容器
```shell script
# -p, -v ':' 左边为宿主机信息，':' 右边为容器内的信息，更多参数可以参考上面的官方镜像说明文档
docker run\
  --restart=on-failure:3\
  --name mysql-5.7\
  -p 3306:3306\
  -v /etc/localtime:/etc/localtime:ro\
  -v /etc/mysql/conf.d:/etc/mysql/conf.d\
  -v /var/lib/mysql:/var/lib/mysql\
  -v /var/log/mysql:/logs\
  -e MYSQL_ROOT_PASSWORD=allcom\
  -d mysql:5.7
```

# 启动容器命令行
```shell script
docker exec -it mysql-5.7  bash
# 后面还可以直接写命令
docker exec -it mysql-5.7 mysql -uroot -paxxxx database -e "select * from user"
```
