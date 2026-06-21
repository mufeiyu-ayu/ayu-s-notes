# OrbStack / Docker 上手笔记

> 面向：从前端转后端的人。
> 目标：看完能独立启动、查看、停止本地数据库，看懂 OrbStack 界面，遇到问题会自己排查。
> 本笔记结合当前项目（`agent` 项目，本地跑一个 PostgreSQL）来写，命令直接可用。

---

## 1. 先建立正确认知

### 1.1 OrbStack 是什么

OrbStack = **Docker Desktop 的轻量替代品**（Mac 上更省电、更快）。

它解决的核心问题：你想在本地跑一个 PostgreSQL，但**不想真的在系统里安装 PostgreSQL**（难卸载、版本乱、污染系统）。容器让数据库跑在一个隔离的小盒子里，不用了直接删掉，系统干干净净。

### 1.2 "Docker" 不是一个软件，是一套标准

平时混着叫的 "Docker" 其实分三层：

| 层 | 是什么 | 类比前端 |
| --- | --- | --- |
| `docker` 命令行（CLI） | 终端里敲的那套指令 | `npm` / `pnpm` 命令 |
| 容器引擎（运行时） | 底层真正把容器跑起来的东西 | Node.js 运行时 |
| 图形界面 | Docker Desktop / OrbStack | npm 的可视化面板 |

`docker` 命令是**全行业事实标准**。OrbStack 换掉了底层引擎和界面，但**故意保留 `docker` 命令不变**——就像 `pnpm` 兼容 `npm` 的命令和 `package.json` 一样。

> 好处：你学的 `docker` 命令不绑定 OrbStack。换成 Docker Desktop、部署到云服务器，命令一模一样，是可迁移的通用技能。

### 1.3 三个核心概念（一定要分清）

| 概念 | 是什么 | 类比前端 |
| --- | --- | --- |
| **镜像 Image** | 打包好的只读模板，如 `postgres:16-alpine` | npm 包的某个发布版本 |
| **容器 Container** | 用镜像跑起来的运行实例 | `install` 后真正跑起来的进程 |
| **数据卷 Volume** | 容器的"持久化硬盘"，数据存这里 | 容器的外接硬盘 |

关键关系：

```txt
镜像（模板）  --运行-->  容器（实例）  --数据存到-->  数据卷
postgres:16-alpine        postgres            postgres-data
```

**重点**：删容器 ≠ 删数据。数据在 Volume 里，删掉容器再重建，数据还在。真要清空数据库，得去删那个 Volume。

---

## 2. 界面导览（OrbStack 左侧菜单）

### Docker 区（你只关心这块）

| 菜单 | 作用 | 你会用吗 |
| --- | --- | --- |
| **Containers** | 容器列表，看哪个在跑 | ✅ 最常用 |
| **Volumes** | 数据卷，数据库数据真正存的地方 | ✅ 偶尔，清空数据时用 |
| **Images** | 已下载的镜像模板 | 🔸 偶尔，删旧镜像省空间 |
| **Networks** | 容器间虚拟网络 | ❌ 现阶段不用管 |

### 其它区（现阶段全部忽略）

- **Kubernetes（Pods / Services）**：高级编排，学习项目用不到。
- **Machines**：Linux 虚拟机，用不到。
- **Activity Monitor**：看容器占用的 CPU / 内存。
- **Commands / Devices**：用不到，命令直接在终端敲更顺。

### 容器条目怎么看

```txt
agent                      ← 项目分组（来自 docker-compose.yml）
  └── postgres             ← 容器名；绿点=运行中，灰点=已停止
      postgres:16-alpine   ← 它用的镜像
```

右侧图标：🔗 连接信息 / ⬛ 停止 / 🗑 删除。

> 日常你在界面上其实只做一个动作：**看 postgres 那个圆点是不是绿的**。绿 = 后端能连数据库；灰 = 没启动，后端会报连接错误。

---

## 3. 你不需要手动"装镜像"

项目里有 `docker-compose.yml`，里面写了用哪个镜像：

```yaml
services:
  postgres:
    image: postgres:16-alpine   # 声明要用的镜像
```

当你 `docker compose up -d` 时，Docker 会**自动检查本地有没有这个镜像，没有就自动下载再启动**。

> `docker-compose.yml` 就是容器世界的 `package.json`：声明依赖，一条命令自动拉齐。
> 所以加新服务（如 Redis）的正确做法是**改 compose 文件**，而不是去界面手动拉镜像。这样配置即代码、能进 git、队友拉代码就能跑。

---

## 4. 常用命令（够用版）

> 都在**项目根目录**（有 `docker-compose.yml` 的地方）执行。
> 新版用 `docker compose`（中间空格），老教程里的 `docker-compose`（带横杠）是旧写法。

### 4.1 日常最高频

```bash
# 启动数据库（后台运行）。-d = detached 后台
docker compose up -d postgres

# 查看本项目容器状态（看到 healthy / running 就对了）
docker compose ps

# 停止（容器和数据都保留，下次 up 秒启）
docker compose stop postgres

# 重启
docker compose restart postgres
```

### 4.2 看日志（数据库连不上时第一时间看这个）

```bash
# 看日志
docker compose logs postgres

# 实时滚动看（Ctrl+C 退出）
docker compose logs -f postgres
```

### 4.3 进容器 / 连数据库

```bash
# 健康检查：返回 "accepting connections" 就是好的（只读，安全）
docker compose exec postgres pg_isready -U postgres -d agent_ai_seo

# 进入容器内的 psql 命令行（操作数据库）
docker compose exec postgres psql -U postgres -d agent_ai_seo
# 进去后常用：\dt 看所有表  \l 看所有库  \q 退出
```

### 4.4 收尾 / 清理

```bash
# 停止并删除容器（⚠️ 数据卷保留，数据还在）
docker compose down

# 停止并删除容器 + 数据卷（⚠️⚠️ 数据库数据会被清空，慎用）
docker compose down -v
```

### 4.5 通用命令（不依赖 compose 文件）

```bash
docker ps              # 所有正在跑的容器
docker ps -a           # 包括已停止的
docker images          # 本地所有镜像
docker pull redis:7    # 单独拉一个镜像
docker logs <容器名>   # 看某容器日志
docker stats           # 实时看各容器资源占用
```

---

## 5. 本项目实战流程

每天开始开发时，按顺序：

```bash
# 1. 启动数据库
docker compose up -d postgres

# 2. 确认健康（accepting connections）
docker compose exec postgres pg_isready -U postgres -d agent_ai_seo

# 3. 启动后端（具体以项目脚本为准）
pnpm dev:api
```

数据库连接信息（与 `.env` 里的 `DATABASE_URL` 对应）：

```txt
host:     localhost
port:     5432
user:     postgres
password: postgres
database: agent_ai_seo
连接串：  postgresql://postgres:postgres@localhost:5432/agent_ai_seo?schema=public
```

---

## 6. 常见问题排查

| 现象 | 可能原因 | 怎么办 |
| --- | --- | --- |
| 后端报连不上数据库 | 容器没启动 | `docker compose ps` 看状态，灰的就 `up -d` |
| `port 5432 already in use` | 本机已装了 PostgreSQL 或别的容器占了端口 | 关掉占用端口的进程，或改 compose 里的端口映射 |
| 数据莫名其妙没了 | 之前跑过 `down -v` 删了数据卷 | 数据卷一旦删除无法恢复，注意 `-v` 参数 |
| 改了 compose 文件不生效 | 容器还是旧的 | `docker compose up -d` 会自动按新配置重建 |
| 想从零干净重来 | — | `docker compose down -v` 再 `up -d`（会清空数据） |

排查口诀：**先 `ps` 看在不在跑，再 `logs` 看报什么错。**

---

## 7. 一句话总结

- OrbStack 只是"引擎 + 面板"，真正的技能是 `docker` 命令，到哪都通用。
- 日常就三件事：`up -d` 启动、`ps` 看状态、`logs` 查问题。
- 配置写进 `docker-compose.yml`，别在界面手动点——配置即代码。
- 删容器不丢数据，删数据卷（`-v`）才丢，小心这个参数。
