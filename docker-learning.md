# Docker 全栈核心指南：从基础到多容器集群编排

本指南旨在为开发者提供一份系统化、面向实战的 Docker 学习与查阅手册。内容涵盖 Docker 的核心设计哲学、四大支柱（镜像、容器、网络、数据卷）的底层逻辑与高频命令、Dockerfile 精准编写规范，以及 Docker Compose 多容器分布式编排的最佳实践。

---

## 1. Docker 的背景与核心优势

### 1.1 传统运维的痛点
在容器技术普及之前，传统的应用部署长期受到以下问题的困扰：
* **“在我的电脑上明明是好的”**：由于开发、测试、生产环境的操作系统版本、依赖库、环境变量、配置不一致，导致应用在线上运行时频繁崩溃。
* **资源利用率低下**：传统虚拟机（VM）需要搭载完整的客户操作系统（Guest OS），即使只运行一个几兆大小的微服务，也需要划分 GB 级别的内存和磁盘空间，造成极大的硬件浪费。
* **启动与扩展缓慢**：虚拟机启动过程包含完整的 OS 引导阶段，通常需要数分钟，无法做到秒级弹性扩容以应对突发流量。

### 1.2 Docker 的诞生与容器化技术
Docker 是基于 Linux 内核的 **Namespace（命名空间）** 实现资源隔离、**CGroups（控制组）** 实现资源限制，以及 **UnionFS（联合文件系统）** 实现分层镜像的高级封装工具。

它将应用程序及其所有依赖项（代码、运行时环境、系统工具、系统库）打包到一个轻量级、可移植的容器中，使应用可以平滑地运行在任何安装了 Docker Engine 的宿主机上。

### 1.3 Docker 的核心优势
| 优势维度 | 传统虚拟机 (VM) | Docker 容器 (Container) |
| :--- | :--- | :--- |
| **架构** | 虚拟化硬件，运行完整的 Guest OS | 共享宿主机内核，利用内核级空间隔离 |
| **启动时间** | 分钟级 | 秒级 / 毫秒级 |
| **资源消耗** | 巨大（通常 GB 级内存起步） | 极小（通常 MB 级内存，几乎无额外 CPU 开销） |
| **存储占用** | 巨大（数十 GB 级磁盘镜像） | 极小（分层复用，一般数百 MB） |
| **密度** | 单台物理机一般只能运行十几个 VM | 单台物理机可并发运行数百个甚至上千个容器 |
| **弹性扩展** | 困难，扩容周期长 | 极易，支持通过 K8s 或 Compose 秒级横向扩展 |

---

## 2. Docker 核心概念与常用指令

Docker 的工作流紧紧围绕四个核心概念展开：**镜像 (Image)、容器 (Container)、网络 (Network)、数据卷 (Volume)**。

### 2.1 镜像 (Image) —— 只读的系统模板
Docker 镜像是一个特殊的**只读**文件系统，它包含了容器运行时所需的程序、库、资源、配置及一些高级元数据。镜像采用 **UnionFS（联合文件系统）** 的分层结构，每一层都是对前一层做出的修改。这种设计使得镜像层可以被高度复用。

#### 2.1.1 镜像常用指令与参数
* **检索镜像**：`docker search <image_name>`
* **下载镜像**：`docker pull [OPTIONS] <name>[:TAG]`
* **查看本地镜像**：`docker images [OPTIONS]`
* **删除本地镜像**：`docker rmi [OPTIONS] <image_id/name>`
* **修改镜像标签**：`docker tag <source_image>[:tag] <target_image>[:tag]`
* **构建镜像**：`docker build [OPTIONS] <path>`
* **导出镜像**：`docker save -o <output_file.tar> <image_name>`
* **导入镜像**：`docker load -i <input_file.tar>`

#### 2.1.2 镜像命令核心参数详析
```bash
# 下载指定版本的 Golang 镜像
docker pull golang:1.22-alpine

# 查看本地所有镜像（包含隐藏的中间层），并仅展示镜像 ID
docker images -a -q

# 强制删除一个正在被容器使用的镜像
docker rmi -f alpine:latest

# 基于当前目录的 Dockerfile 构建镜像，并打上生产环境标签
docker build -t my-app:v1.0.0 .
# 参数说明:
#  -t, --tag: 指定镜像的名称和标签 (name:tag)
#  .        : 指定上下文路径 (Context)，Docker 会将该目录下的文件发送给守护进程
```

---

### 2.2 容器 (Container) —— 镜像的运行实例
容器是镜像运行时的实体。容器可以被创建、启动、停止、删除和暂停。容器层在镜像层之上增加了一个**可写层（Writable Layer）**，应用在运行期间产生的所有修改都只会记录在可写层中。

#### 2.2.1 容器常用指令
* **创建并启动容器**：`docker run [OPTIONS] <image> [COMMAND] [ARG...]`
* **查看容器列表**：`docker ps [OPTIONS]`
* **停止容器**：`docker stop <container_id>`
* **启动已有容器**：`docker start <container_id>`
* **重启容器**：`docker restart <container_id>`
* **删除容器**：`docker rm [OPTIONS] <container_id>`
* **进入容器内部执行命令**：`docker exec [OPTIONS] <container_id> <command>`
* **查看容器日志**：`docker logs [OPTIONS] <container_id>`
* **查看容器详细元数据**：`docker inspect <container_id>`

#### 2.2.2 容器命令核心参数详析
```bash
# 启动一个后台运行、带有名字、端口映射、环境配置及自动重启的 Redis 容器
docker run -d \
  --name my-redis \
  -p 6379:6379 \
  -v /opt/redis/data:/data \
  -e REDIS_PASSWORD=secret_pass \
  --restart always \
  redis:7.0-alpine --requirepass "secret_pass"

# 参数剖析:
#  -d, --detach      : 后台运行容器，并返回容器 ID。
#  --name            : 为容器指定一个自定义别名，避免随机生成怪异名称。
#  -p, --publish     : 端口映射，格式为「宿主机端口:容器内端口」。
#  -v, --volume      : 挂载数据卷或宿主机目录。
#  -e, --env         : 设置容器内的环境变量。
#  --restart always  : 容器退出策略。always 表示无论因何种原因退出（或 Docker 重启），均自动重启。
#  --rm              : 容器退出时自动删除容器文件系统（不可与 -d 同时配合用于长期服务）。

# 动态查看容器日志（类似于 tail -f）并显示时间戳
docker logs -f --tail 100 --timestamps my-redis

# 交互式进入一个正在运行的 Linux 容器执行 shell
docker exec -it my-redis sh
# 参数剖析:
#  -i, --interactive : 保持标准输入打开，允许接收用户命令。
#  -t, --tty         : 分配一个伪终端 (Pseudo-TTY)，提供标准的交互式 Shell 界面。

# 查看本地正在运行以及已经停止的所有容器
docker ps -a
```

---

### 2.3 网络 (Network) —— 容器间的通信桥梁
Docker 通过网络驱动为容器之间、容器与宿主机之间提供隔离与互通能力。

#### 2.3.1 四大核心网络模式
1.  **Bridge (网桥模式)**：默认模式。Docker 会在宿主机创建一个名为 `docker0` 的虚拟网桥，每个容器都会分配独立的 Namespace 和 IP，通过 NAT 规则与外网通信。
2.  **Host (宿主机模式)**：容器不隔离网络命名空间，直接使用宿主机的网卡和端口。性能最高，但存在端口冲突风险。
3.  **None (无网络模式)**：容器拥有独立的网络命名空间，但不进行任何网络配置（无网卡、IP）。适用于高安全性的离线计算任务。
4.  **Overlay (覆盖网络模式)**：用于跨多台物理主机的容器间通信（通常在 Docker Swarm 或多主机分布式环境中使用）。

#### 2.3.2 网络常用指令
* **创建自定义网络**：`docker network create [OPTIONS] <network_name>`
* **列出所有网络**：`docker network ls`
* **查看网络详情**：`docker network inspect <network_name>`
* **将容器连接到网络**：`docker network connect <network_name> <container_name>`
* **将容器断开网络**：`docker network disconnect <network_name> <container_name>`
* **删除网络**：`docker network rm <network_name>`

#### 2.3.3 网络命令实战
```bash
# 创建一个名为 app-net的自定义桥接网络
docker network create --driver bridge app-net

# 在启动容器时直接加入该自定义网络
docker run -d --name biz-service --network app-net my-biz-app:latest

# 优势：加入同一个自定义 bridge 网络的容器，可以直接通过「容器名」作为域名进行内部高效互访，无需关心动态变化的 IP。
```

---

### 2.4 数据卷 (Volume) —— 数据的持久化与共享
由于容器的可写层生命周期与容器绑定，容器一旦被销毁，内部产生的数据也会全部丢失。为了解决**数据持久化**以及**容器间数据共享**的问题，Docker 引入了数据卷机制。它将宿主机的特定目录挂载到容器中，绕过了 UnionFS。

#### 2.4.1 数据挂载的三种方式
1.  **Named Volumes（命名数据卷）**：由 Docker 完全管理的卷，存储在宿主机的固定目录（如 `/var/lib/docker/volumes/`）。推荐在生产环境使用。
2.  **Bind Mounts（绑定挂载）**：直接将宿主机上任意绝对路径的目录或文件挂载到容器中。适合开发环境（如将本地源码挂载到容器中实现热更新）。
3.  **Tmpfs Mounts**：数据保存在宿主机的内存中，不会写入磁盘。适合存放敏感且无需持久化的临时凭证。

#### 2.4.2 数据卷常用指令
* **创建数据卷**：`docker volume create <volume_name>`
* **列出所有卷**：`docker volume ls`
* **查看卷详情**：`docker volume inspect <volume_name>`
* **删除特定卷**：`docker volume rm <volume_name>`
* **清理无主孤儿卷**：`docker volume prune`

#### 2.4.3 数据卷挂载实战对比
```bash
# 1. 使用命名数据卷持久化 pgsql 数据 (推荐)
docker run -d --name pg-db -v pg_data:/var/lib/postgresql/data postgres:15-alpine

# 2. 使用绝对路径的 Bind Mount 挂载本地配置文件 (适合配置、源码同步)
docker run -d --name nginx-server -v /home/user/nginx.conf:/etc/nginx/nginx.conf:ro -p 80:80 nginx:alpine
# 注: 末尾加上 :ro 表示 container 内对此挂载文件「只读 (Read-Only)」，增强安全性。
```

---

## 3. Docker 其他高频实用指令与系统运维

在日常开发与高级运维中，以下系统级指令能帮助我们进行快速排查、监控与瘦身。

```bash
# 1. 实时监控容器资源消耗 (CPU、内存、网络 I/O、磁盘 I/O)
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# 2. 查看某个容器内正在运行的进程树
docker top my-redis

# 3. 在宿主机与容器之间进行文件/目录的双向复制
docker cp /path/to/local/file.txt my-redis:/data/imported_file.txt # 复制到容器
docker cp my-redis:/data/dump.rdb ./local_dump.rdb                  # 从容器拷出

# 4. 深度清理系统（极其高频的瘦身命令）
# 该命令会删除：所有已停止的容器、所有未被至少一个容器使用的网络、所有未打标签的镜像(虚悬镜像)、所有构建缓存
docker system prune -f

# 如果想连同「未被使用的卷」和「所有未运行容器的镜像」一起彻底干掉，可追加参数：
docker system prune -a --volumes -f
```

---

## 4. Dockerfile 配置文件编写详解

**Dockerfile** 是一个文本文件，其中包含了一组用户可以调用其命令行以组装镜像的指令。它是自动化构建的核心。

### 4.1 核心指令全景表
| 指令 | 含义说明 | 最佳实践防坑指南 |
| :--- | :--- | :--- |
| **FROM** | 指定基础镜像 | 尽量采用官方的、轻量级的 `alpine` 或 `slim` 版本镜像。 |
| **WORKDIR** | 设置工作目录 | 为后续的 `RUN, CMD, ENTRYPOINT, COPY` 指令设置绝对路径，若不存在会自动创建。 |
| **COPY** | 复制文件/目录 | 仅复制本地文件到镜像中。支持 `--chown` 在复制时修改文件属主。 |
| **ADD** | 高级复制文件 | 功能比 COPY 强，能解压本地 tar 包，能下载远程 URL（但不推荐，会增加体积）。 |
| **RUN** | 执行构建命令 | 在镜像构建阶段执行。多个命令务必用 `&&` 及换行符 `\` 合并成一条，减少镜像层数。 |
| **ENV** | 设置持久环境变量 | 镜像运行时及构建时均有效。 |
| **ARG** | 构建期临时变量 | 仅在 `docker build` 阶段有效，容器运行时不留痕迹。可通过 `--build-arg` 传入。 |
| **EXPOSE** | 声明暴露端口 | 仅起声明/文档作用，并不会在宿主机自动绑定物理端口。 |
| **VOLUME** | 挂载匿名数据卷 | 强制将容器内指定路径配置为匿名卷，防止用户因忘记挂载而导致核心数据写入可写层。 |
| **CMD** | 容器启动默认命令 | 指定容器启动时执行的命令，可以被 `docker run` 命令行尾部追加的参数直接**覆盖**。 |
| **ENTRYPOINT**| 容器启动入口命令 | 指定容器运行的实体。不易被覆盖，尾部追加的参数会被当作参数传递给它。 |

### 4.2 CMD 与 ENTRYPOINT 的黄金组合
在编写生产级 Dockerfile 时，推荐使用 **Exec 格式** (`["executable", "param1"]`)，并组合两者：用 `ENTRYPOINT` 固定执行脚本或主程序，用 `CMD` 提供可变动的默认参数。

### 4.3 生产级实战：Go 微服务的多阶段构建 (Multi-stage Build)
多阶段构建是优化镜像体积、确保生产环境安全的关键技术。它允许我们在一个 Dockerfile 中使用多个 `FROM` 语句，将前一阶段编译生成的二进制产物拷贝到后一个精简的运行时镜像中，从而抛弃所有编译依赖。

```dockerfile
# ==========================================
# 第一阶段：构建/编译阶段 (Build Stage)
# ==========================================
FROM golang:1.22-alpine AS builder

# 安装编译所需的依赖（如 gcc 等，根据项目实际决定是否需要）
RUN apk add --no-cache git gcc musl-dev

# 设置容器内的构建工作目录
WORKDIR /app

# 利用 Docker 构建缓存机制：先拷贝依赖描述文件，再进行 download
COPY go.mod go.sum ./
RUN go mod download

# 拷贝全量业务源码
COPY . .

# 执行 CGO 禁用编译，构建出极致静态绑定的二进制可执行文件
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o server ./cmd/main.go


# ==========================================
# 第二阶段：生产运行阶段 (Final Stage)
# ==========================================
FROM alpine:3.19

# 设置时区与基础安全证书
RUN apk add --no-cache ca-certificates tzdata \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone

WORKDIR /app

# 核心：从名为 builder 的上层阶段中，仅仅将编译完的无依赖可执行文件拷贝过来
COPY --from=builder /app/server .
# 拷贝静态资源或配置文件
COPY --from=builder /app/config ./config

# 使用非 root 安全用户运行服务
RUN adduser -D appuser
USER appuser

# 声明端口
EXPOSE 8080

# 启动容器
ENTRYPOINT ["./server"]
```
*使用该模式，镜像体积可以从原本包含 Go 环境的 800MB+ 直降至 20MB 左右。*

---

## 5. Docker Compose 多容器集群编排

### 5.1 什么是 Docker Compose？
在实际的企业级微服务应用中，一个业务系统往往由多个模块组成（例如：API 网关、后端微服务、中间件 Redis、数据库 PostgreSQL）。如果全部手动通过 `docker run` 去逐个管理网络、数据卷和容器启动顺序，运维工作将变成灾难。

**Docker Compose** 是一个用于定义和运行多容器 Docker 应用程序的工具。它通过一个独立的 `docker-compose.yml` 配置文件，允许用户将一组相互关联的容器作为一个完整的应用服务栈（Stack）进行统一建模、一键启停和弹性伸缩。

### 5.2 核心指令集
Docker Compose 命令需要在包含 `docker-compose.yml` 的目录下执行：
* **一键后台构建并启动整个服务栈**：`docker-compose up -d`
* **强制重新构建镜像并启动**：`docker-compose up -d --build`
* **停止并删除服务栈（释放容器、网络、挂载点）**：`docker-compose down`
* **停止并删除服务栈，同时连同命名数据卷一起彻底清理**：`docker-compose down -v`
* **查看服务栈内的容器状态**：`docker-compose ps`
* **实时聚合查看所有容器的日志输出**：`docker-compose logs -f`
* **水平扩容某个微服务实例**：`docker-compose up -d --scale web-service=3`

---

### 5.3 生产级多容器编排模板：`docker-compose.yml`
以下提供一个开箱即用的工业级全栈多容器编排模板，包含了一个后端 API 微服务（支持源码动态构建）、一个高性能可定制的 Nginx 网关、一个高可用 PostgreSQL 数据库以及一个 Redis 缓存实例。

```yaml
version: '3.8'

# 定义全局复用的共享网络
networks:
  app-tier:
    driver: bridge

# 定义需要持久化的命名数据卷
volumes:
  pg_backend_data:
    driver: local
  redis_cache_data:
    driver: local

services:
  # ----------------------------------------------------
  # 1. 数据库服务 (PostgreSQL)
  # ----------------------------------------------------
  db-server:
    image: postgres:15-alpine
    container_name: compose-postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: my_secure_password_123
      POSTGRES_DB: core_biz_db
    volumes:
      - pg_backend_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-tier
    healthcheck: # 健康检查：确保数据库真正初始化完毕后才允许下级依赖服务连接
      test: ["CMD-SHELL", "pg_isready -U admin -d core_biz_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  # ----------------------------------------------------
  # 2. 缓存服务 (Redis)
  # ----------------------------------------------------
  cache-server:
    image: redis:7.0-alpine
    container_name: compose-redis
    restart: always
    command: redis-server --requirepass "redis_secure_pass"
    volumes:
      - redis_cache_data:/data
    networks:
      - app-tier

  # ----------------------------------------------------
  # 3. 后端业务微服务 (基于本地 Dockerfile 构建)
  # ----------------------------------------------------
  api-service:
    build:
      context: ./backend-app    # 本地源码路径
      dockerfile: Dockerfile    # 对应的 Dockerfile 路径
    container_name: compose-api-app
    restart: on-failure
    environment:
      - DB_HOST=db-server       # 直接利用服务名进行容器间 DNS 域名解析
      - DB_PORT=5432
      - DB_USER=admin
      - DB_PASSWORD=my_secure_password_123
      - DB_NAME=core_biz_db
      - REDIS_HOST=cache-server
      - REDIS_AUTH=redis_secure_pass
    depends_on:
      db-server:
        condition: service_healthy # 强依赖：等待 db-server 的健康检查通过
      cache-server:
        condition: service_started # 弱依赖：Redis 启动即可
    networks:
      - app-tier

  # ----------------------------------------------------
  # 4. 反向代理与网关 (Nginx)
  # ----------------------------------------------------
  web-gateway:
    image: nginx:alpine
    container_name: compose-nginx-gateway
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - api-service
    networks:
      - app-tier
```

### 5.4 编排深度技巧提示
1.  **容器间通信原理**：在上述的 `docker-compose.yml` 中，`api-service` 连接数据库时，主机名配置的是 `db-server` 而不是 `localhost` 或具体的 IP。Docker Compose 会在启动时自动创建一个属于该项目范围的默认网桥，并将服务名作为 DNS 条目注册，各容器间直接通过服务名即可完成稳定通信。
2.  **启动顺序控制 (`depends_on`)**：普通的 `depends_on` 仅能保证容器的**启动（Start）**先后，并不能保证容器内服务**就绪（Ready）**。结合上文中的 `healthcheck` 块与 `condition: service_healthy` 约束，可以完美解决因微服务比数据库先拉起而导致的连接超时、崩溃退出的典型生产痛点。
