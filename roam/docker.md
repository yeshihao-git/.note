---
tags:
  - 工具
---
# docker

**what**：
开源的容器化平台，用于快速构建、部署和运行应用程序及其依赖项，实现环境隔离。三大关键组件：
1. 仓库
2. 镜像
3. 容器

概念
- 分层机制：镜像由多个只读层（Dockerfile中每个RUN指令对应一层）构成，每层只存储与上一层的差异部分，优化镜像大小的方法：合并多个RUN为一个 或 多阶段构建。容器就是镜像上加了一层可写层，容器停止后可写层丢失。
- 多阶段构建：在单个Dockerfile文件中定义多个FROM指令，分离构建和运行，只保留必要的文件和依赖，优化镜像大小
- 构建上下文：构建镜像时，传给docker build命令的目录或URL，用于构建镜像

docker的3个默认网络
- bridge(桥接网络)   ：可以将容器接入这个虚拟子网，容器间通过IP或容器名通信
- host(宿主机网络)   ：创建容器时通过 --network host 共享宿主机的网络（IP和端口）
- none(禁用所有网络) ：只有lo（本地回环），无法与外界通信

## docker配置镜像源

```bash
# 配镜像源影响 docker pull / docker push
vim /etc/docker/daemon.json # 输入以下内容

{
        "registry-mirrors": [
                "https://docker.lms.run",
                "https://hub.rat.dev",
                "https://docker.1panel.live",
                "https://mirror.ccs.tencentyun.com",
                "https://docker.mirrors.ustc.edu.cn",
                "https://hub-mirror.c.163.com"
        ]
}
```

## docker走代理

```bash
# 影响 docker build 的速度
docker build --network host \
  --build-arg http_proxy=http://127.0.0.1:7890 \
  --build-arg https_proxy=http://127.0.0.1:7890 \
  -t <镜像名> .
```

# Dockerfile

**what**：
包含一系列指令的文本文件，用于自动化构建 docker镜像

|命令|作用|
|---|---|
|FROM|指定基础镜像|
|WORKDIR|设置工作目录 (若不存在自动创建)|
|RUN|执行命令，并创建新的镜像层|
|COPY <宿主机路径> < 容器路径 >|复制文件|
|ENV|设置环境变量|
|CMD|指定容器创建时的默认命令 (可以覆盖)|
|SHELL|覆盖 Docker 中默认的 shell，用于 RUN、CMD、ENTRYPOINT|

## Dockerfile构建自动执行sql文件的镜像

mysql官方镜像在第一次启动时，会自动执行 /docker-entrypoint-initdb.d目录 下的 .sql文件（SQL命令的集合）
```bash
# Dockerfile文件
FROM mysql:8.0
COPY llfc.sql /docker-entrypoint-initdb.d/
ENV MYSQL_ROOT_PASSWORD=123456
ENV MYSQL_DATABASE=llfc
```

# docker compose

**what**：
docker官方的容器编排工具，简化对多个容器的操作（一键启动、停止、销毁），通过yml文件定义多个关联容器的配置

```yaml
# 示例：docker-compose.yml
redis:
    image: redis:latest              # 镜像
    container_name: redis-container  # 容器名
    pull_policy: never               # 拉取策略（默认从远程仓库拉取，设置never优先拉取本地）
    volumes:                         # 数据卷
      - "/home/yoshiki01/docker/redis/redis.conf:/usr/local/etc/redis/redis.conf"
    command: redis-server /usr/local/etc/redis/redis.conf         # 覆盖默认命令
    networks:                        # 网络（docker network中定义的）
      - server
mysql:
    image: mysql_new:latest
    container_name: mysql-container
    pull_policy: never
    ports:
      - "3303:3306"
    environment:                     # 环境变量
      - MYSQL_ROOT_PASSWORD=123456
    networks:
      - server
    healthcheck:                     # 健康检查：确保容器正常运行后再进行再后续操作（eg：依赖启动）
      test: >                        # （必须）检查命令或脚本
        mysql -uroot -p123456 -e "SELECT 1 FROM llfc.user LIMIT 1"
        && mysqladmin ping -uroot -p123456
      interval: 5s                   # 检查间隔
      timeout: 3s                    # 单次检查的超时时间
               retries: 20
gate:
    image: gateserver_new:latest
    container_name: gateserver-container
    pull_policy: never
    ports:
      - "8080:8080"
    volumes:
      - "/home/yoshiki01/docker/GateServer/config.ini:/config.ini"
    depends_on:                      # 启动依赖
      mysql:
        condition: service_healthy   # 健康检查服务是否通过
      redis:
        condition: service_started   # 只检查服务是否启动
    networks:
      - server
```