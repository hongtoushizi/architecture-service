# swarm集群服务

> 现在只把web服务和job服务这种计算**无状态服务**迁移到swarm集群上
> mysql/redis等存储服务尽量使用云服务
> http://www.dockerinfo.net/4374.html

### 创建集群
init swarm
```bash
docker swarm init --advertise-addr <MANAGER-IP>
```

添加进集群
```bash
docker swarm join \
--token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
192.168.99.100:2377
```

显示token cmd
```bash
docker swarm join-token manager
```

集群节点
```bash
docker node ls
```

### 管理服务
在Swarm集群上部署服务，必须在Manager Node上进行操作。先说明一下Service、Task、Container（容器）这个三个概念的关系，如下图（出自Docker官网）非常清晰地描述了这个三个概念的含义：
![](http://img.dockerinfo.net/2017/03/20170315210902.jpg)

创建Docker服务，可以使用docker service create命令实现
```bash
docker service create --replicas 2 --name my_web nginx
```
扩容缩容服务
```bash
docker service scale my_web=3
```
删除服务
```bash
docker service rm my_web
```
滚动更新(**具体参考服务持续发布**)
```bash
docker service update my_web
```

### 构建管理系统  
> ** 等待shipyard-swarm版本发布 **

使用shipyard作为docker及docker service管理工具
> Shipyard enables multi-host, Docker cluster management. It uses Docker Swarm for cluster resourcing and scheduling.

shipyard 安装配置
```bash
curl -sSL https://shipyard-project.com/deploy | TLS_CERT_PATH=$(pwd)/certs bash -s
```