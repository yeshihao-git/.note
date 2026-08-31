---
tags:
  - cicd
  - docker
  - mysql
  - nginx
  - kkrepo
  - gitea
---
# CICD 工作流搭建
## 环境搭建
### docker 部署

1. 安装
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
curl -fsSL https://get.docker.com | sudo sh
```

2. 设置开机自启
```bash
sudo systemctl enable --now docker # 设置为开机自启
```

3. 配置镜像源
```bash
vim /etc/docker/daemon.json

# 配置镜像源
{
  "registry-mirrors": [
	"https://docker.m.daocloud.io"
  ]
}

sudo systemctl daemon-reload  # 重新加载 systemd 的配置文件
sudo systemctl restart docker # 重启 docker 守护进程
```

4. 运维命令
```bash
sudo docker ps     # 查看运行的容器
sudo docker images # 查看镜像
```

### mysql 部署（docker）

1. 拉取 docker 镜像
```bash
sudo docker pull mysql:8.0.46
```

2. 启动（端口映射、数据卷挂载、环境变量）
```bash
sudo docker run -d --name mysql --restart unless-stopped \
  -p 3306:3306 \
  --env-file /opt/mysql/mysql.env \
  -v /opt/mysql/data:/var/lib/mysql \
  mysql:8.0.46
```

3. 登录并进入 mysql
```bash
sudo docker exec -it mysql mysql -uroot -p
```

4. 创建两个数据库
```sql
CREATE DATABASE gitea
  CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;

CREATE DATABASE kkrepo
  CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

5. 创建账号并授权
```mysql
CREATE USER 'gitea'@'%' IDENTIFIED BY 'Gitea数据库密码'; # % 是主机名，表示用户 gitea 可以从任意IP地址连接数据库 
GRANT ALL PRIVILEGES ON gitea.* TO 'gitea'@'%';        # 对 gitea 数据库下的所有表（gitea.*）拥有全部权限（ALL PRIVILEGES）

CREATE USER 'kkrepo'@'localhost' IDENTIFIED BY 'KKRepo数据库密码'; # localhost 允许从数据库服务器本机（通过Unix socket或127.0.0.1）连接
GRANT ALL PRIVILEGES ON kkrepo.* TO 'kkrepo'@'localhost';

CREATE USER 'kkrepo'@'172.17.0.1' IDENTIFIED BY 'KKRepo数据库密码'; # 172.17.0.1 允许从Docker默认网桥的网关IP连接，其他运行在宿主机的docker可以通过这个IP访问数据库
GRANT ALL PRIVILEGES ON kkrepo.* TO 'kkrepo'@'172.17.0.1';

FLUSH PRIVILEGES; # 刷新权限

EXIT; # 退出
```

6. gitea、kkrepo 数据库账号密码
```
账号：gitea
密码：gitea_db_2026_change_me

账号：kkrepo
密码：kkrepo_pass_change_me
```

### nginx 部署（docker）

1. 拉取 docker 镜像
```bash
sudo docker pull nginx:1.31.4-alpine
```

2. 启动（端口映射、数据卷挂载）
```bash
sudo docker run -d --name cicd-nginx --restart unless-stopped \
  -p 8080:80 \
  -v /home/test/cicd/runtime/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /home/test/cicd/runtime/nginx/projects:/etc/nginx/project.d:ro \
  -v /home/test/cicd/test:/usr/share/nginx/html:ro \
  nginx:1.31.4-alpine

# -d 后台运行
# --name cicd-nginx 容器命名为 cicd-nginx
# --restart unless-stopped 除非手动停止，否则自动重启
# -p 8080:80 将宿主机的 8080 端口映射到容器的 80 端口

# 配置文件挂载（:ro 只读，防止容器修改文件）
# -v /home/test/cicd/runtime/nginx/conf.d:/etc/nginx/conf.d:ro 挂载自定义的 Nginx 配置文件片段（如 default.conf）
# -v /home/test/cicd/runtime/nginx/projects:/etc/nginx/project.d:ro 挂载项目相关的 Nginx 配置（可能是多个站点的配置）

# 静态文件挂载
# -v /home/test/cicd/test:/usr/share/nginx/html:ro 将宿主机 /home/test/cicd/test 目录挂载为 Nginx 的默认 Web 根目录

```

### kkrepo 部署（docker）

1. 拉取 docker 镜像
```bash
sudo docker pull ghcr.io/klboke/kkrepo:0.9.0
```

2. 启动（数据卷挂载、环境变量）
```bash
sudo docker run -d --name kkrepo --restart unless-stopped \
  --network host \
  --env-file /opt/kkrepo/kkrepo.env \
  -v /opt/kkrepo/data:/var/lib/kkrepo \
  ghcr.io/klboke/kkrepo:0.9.0

# -d：后台运行
# --name kkrepo：容器命名为 kkrepo
# --restart unless-stopped：自动重启策略
# --network host 直接使用宿主机网络（端口），不需要 -p 端口映射

# 环境变量
# --env-file /opt/kkrepo/kkrepo.env 从文件读取环境变量（数据库连接、密钥、配置参数等）

# -v /opt/kkrepo/data:/var/lib/kkrepo 将容器内的 /var/lib/kkrepo 挂载到宿主机 /opt/kkrepo/data
```

3. 账号密码
```
账号：Local/admin
密码：12345678
```

### gitea 部署（docker）

1. 拉取 docker 镜像
```bash
sudo docker pull docker.gitea.com/gitea:1.27.0
```

2. 启动（端口映射、数据卷挂载）
```bash
sudo docker run -d --name gitea --restart unless-stopped \
  --add-host=host.docker.internal:host-gateway \
  -p 3000:3000 \
  -p 2222:22 \
  -v /opt/gitea/data:/data \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/timezone:/etc/timezone:ro \
  docker.gitea.com/gitea:1.27.0
  
# 1. --restart unless-stopped 除非手动停止容器，否则 Docker 会在容器退出或系统重启时自动重新拉起它

# 2. --add-host=host.docker.internal:host-gateway 在容器的 /etc/hosts 中添加一条记录，将 host.docker.internal 解析为宿主机的 IP 地址

# 3. 端口映射 
# -p 3000:3000 将宿主机的 3000 端口映射到容器的 3000 端口（Gitea Web 界面）
# -p 2222:22 将宿主机的 2222 端口映射到容器的 22 端口（Gitea SSH 克隆通道）

# 4. 数据卷挂载
# -v /opt/gitea/data:/data 持久化存储 Gitea 的所有数据（仓库、配置、数据库文件等）到宿主机的 /opt/gitea/data
# -v /etc/localtime:/etc/localtime:ro 将宿主机的时区文件以只读方式挂载，确保容器时间与宿主机一致
# -v /etc/timezone:/etc/timezone:ro 同上，确保时区设置正确

# 5. docker.gitea.com/gitea:1.27.0 镜像地址和版本标签，指定使用 Gitea 1.27.0 版本
```

### gitea runner 部署（systemd 服务）

1. 拉取 gitea runner 镜像，并从镜像中提取二进制
```bash
sudo mkdir -p /opt/act-runner/data
sudo docker pull docker.io/gitea/runner:latest
sudo docker create --name act_runner_extract docker.io/gitea/runner:latest # 创建新 docker 容器后，不启动它（常用于从镜像提取文件）
sudo docker cp act_runner_extract:/usr/local/bin/gitea-runner /usr/local/bin/gitea-runner # 从容器中复制文件到宿主机
sudo docker rm act_runner_extract # 清理临时容器
sudo chmod 755 /usr/local/bin/gitea-runner
```

2. 将 gitea runner 注册到 gitea 组织（使用 gitea 组织级 token 注册）
获取token：
![[Pasted image 20260828111837.png|851]]
运行 gitea runner：
```bash
sudo /usr/local/bin/gitea-runner register \
  --no-interactive \
  --instance http://192.168.42.129:3000 \
  --token <组织Token> \
  --name cicd-runner \
  --labels linux-amd64-host:host
  
# 注册信息保存在 /opt/act-runner/data/.runner

# --labels 用来给 Runner 设置“标签”，工作流通过标签选择在哪台 Runner 上执行
# --labels linux-amd64-host:host 对应工作流 runs-on: linux-amd64-host
```

3. 创建 gitea runner 配置
文件：
```bash
/opt/act-runner/data/.runner
```
内容：
```yaml
runner:
  labels:
    - linux-amd64-host:host
```

4. 创建 systemd 服务
文件：
```bash
/etc/systemd/system/gitea-act-runner.service
```
内容：
```ini
[Unit]
Description=Gitea Runner cicd-runner
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/act-runner/data
Environment=HOME=/home/test
Environment=PATH=/home/test/.nvm/versions/node/v22.23.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ExecStart=/usr/local/bin/gitea-runner daemon --config /opt/act-runner/data/config.yaml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

5. 启动服务
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gitea-act-runner.service
sudo systemctl status gitea-act-runner.service
```
查看日志：
```bash
sudo journalctl -u gitea-act-runner.service -f
```

## CICD 工作流搭建
### 创建 actions 仓库

1. 创建 actions 组织存放各类 actions
![[Pasted image 20260828105152.png|833]]

2. 迁移外部仓库
在==当前 gitea 的 app.ini== 中写入，允许从外部 gitea 迁移
```ini
[migrations]
ALLOW_LOCALNETWORKS = true
ALLOWED_DOMAINS = 10.8.254.129（外部的IP）
BLOCKED_DOMAINS =
```
点击 ==迁移外部仓库==
![[Pasted image 20260822140333.png]]
![[Pasted image 20260822140447.png|486]]

### 创建组织并设置

1. 创建组织 InteVue 存放代码仓库
![[Pasted image 20260828110208.png|831]]
2. 创建工作流仓库 workflows，按 gitea 规则创建文件夹：==yml 文件必须放在 .gitea/scoped_workflows 中==
![[Pasted image 20260828111354.png|828]]
3. InteVue 组织设置