# Prisma 从零上手教程

> 面向：会前端 TypeScript、但完全没用过数据库 ORM 的人。
> 目标：看完能看懂项目里的 Prisma 代码，能自己定义表、查数据、改数据。
> 全程用前端思维类比，结合当前 `agent` 项目（PostgreSQL + NestJS）来讲。

---

## 0. 心智模型：一句话先记住

> **Prisma 让你用"写 TypeScript 对象"的方式操作数据库，不用手写 SQL。**

类比：
- 没有 ORM：手写 SQL 字符串 `SELECT * FROM messages WHERE ...`，像手写原生 `XMLHttpRequest`。
- 有 Prisma：写 `prisma.message.findMany({ where: {...} })`，像用封装好的 `axios`——有类型、有补全、不易写错。

它对 TS 开发者最大的好处：**数据库表结构会自动变成 TS 类型**，查出来的数据全程类型提示，字段名写错直接报红。

---

## 1. 几个基础名词（先扫盲）

如果你连数据库都没接触过，先建立这几个概念：

| 名词 | 是什么 | 类比 |
| --- | --- | --- |
| 数据库 Database | 存数据的"仓库"，本项目用 PostgreSQL | 一个 Excel 文件 |
| 表 Table | 仓库里的一类数据，如"消息表" | Excel 里的一张 sheet |
| 字段 Column | 表的一列，如 `id`、`content` | sheet 的列头 |
| 记录 Row | 表里的一行数据 | sheet 的一行 |
| 主键 Primary Key | 每行唯一的身份证，通常是 `id` | 学号 |
| 外键 Foreign Key | 指向另一张表某行的"引用" | "这条消息属于哪个会话" |
| ORM | 把数据库表映射成代码对象的工具 | Prisma 就是 ORM |

```txt
数据库 agent_ai_seo
 └── 表 Message
      ├── 字段：id | role | content | conversationId
      ├── 记录：abc-1 | user      | 你好     | conv-1
      └── 记录：abc-2 | assistant | 你好呀   | conv-1
```

---

## 2. Prisma 的四个组成部分

```txt
1. schema.prisma   →  定义"数据库长什么样"（表结构）      ← 你主要写这个
2. prisma 命令      →  把 schema 同步到数据库 + 生成代码
3. Prisma Client    →  自动生成的、带类型的查询工具         ← 不用手写
4. PrismaService    →  在 NestJS 里把 Client 包成可注入服务  ← 业务里用
```

这四块对应项目里的文件：

| 部分 | 项目文件 |
| --- | --- |
| schema | `prisma/schema.prisma` |
| 配置 | `prisma.config.ts` |
| 生成的 Client | `apps/api/src/generated/prisma/`（自动生成，已 gitignore） |
| NestJS 服务 | `apps/api/src/prisma/prisma.service.ts`、`prisma.module.ts` |
| 命令 | `package.json` 里的 `prisma:generate` / `prisma:migrate` / `prisma:studio` |

下面逐块详讲。

---

## 3. schema.prisma —— 数据的"设计图纸"

这是你写得最多的文件，分三段。

### 3.1 文件结构

```prisma
// ① generator：告诉 Prisma 生成什么样的 Client
generator client {
  provider = "prisma-client"
  output   = "../apps/api/src/generated/prisma"  // 生成代码放哪
}

// ② datasource：连哪个数据库
datasource db {
  provider = "postgresql"     // 数据库类型
  url      = env("DATABASE_URL")  // 连接串从环境变量读
}

// ③ model：定义表（核心，你主要写这部分）
model Conversation {
  id    String @id @default(uuid())
  title String
}
```

> `generator` 和 `datasource` 配好后基本不用动，**你日常只在下面加/改 `model`。**

### 3.2 model 怎么写

一个 `model` = 一张表 = 一个 TS 类型。语法：`字段名  类型  属性修饰`。

```prisma
model Conversation {
  id        String    @id @default(uuid())   // 主键，自动生成 uuid
  title     String                            // 普通字符串字段
  createdAt DateTime  @default(now())         // 时间，默认创建时刻
  updatedAt DateTime  @updatedAt              // 每次更新自动刷新
  messages  Message[]                         // 关系：一个会话有多条消息
}
```

类比：**这就像写 TS `interface`**，只不过它会真的变成数据库的表。

### 3.3 常用字段类型

| Prisma 类型 | 含义 | 对应 TS |
| --- | --- | --- |
| `String` | 字符串 | `string` |
| `Int` | 整数 | `number` |
| `Float` | 小数 | `number` |
| `Boolean` | 布尔 | `boolean` |
| `DateTime` | 日期时间 | `Date` |
| `Json` | JSON 数据 | `object` |

### 3.4 常用属性修饰（@ 开头）

| 修饰 | 作用 |
| --- | --- |
| `@id` | 标记为主键 |
| `@default(...)` | 默认值，如 `@default(now())`、`@default(uuid())` |
| `@unique` | 该字段值不可重复（如 email） |
| `@updatedAt` | 每次更新自动写入当前时间 |
| `?`（写在类型后） | 可空，如 `title String?` 表示可以没有 |
| `@relation(...)` | 定义表之间的关系（见下） |

### 3.5 表关系（最容易懵的部分）

最常见是「一对多」：一个会话有多条消息。需要在**两张表里都写**：

```prisma
model Conversation {
  id       String    @id @default(uuid())
  messages Message[]                         // 一方：数组，表示"有多条"
}

model Message {
  id             String       @id @default(uuid())
  role           String
  content        String
  conversationId String                                              // 外键字段：存所属会话的 id
  conversation   Conversation @relation(fields: [conversationId], references: [id])  // 关系定义
}
```

怎么理解：
- `Conversation` 里的 `messages Message[]` —— "我有一堆消息"（虚拟的，数据库里不存这列）。
- `Message` 里的 `conversationId` —— **真正存在数据库里的外键列**，存它属于哪个会话。
- `@relation(fields: [conversationId], references: [id])` —— 告诉 Prisma：用本表的 `conversationId` 去对应 `Conversation` 表的 `id`。

类比前端：就像两个对象互相引用，`conversation.messages` 能拿到所有消息，`message.conversation` 能拿到所属会话。

---

## 4. Prisma 命令 —— 把图纸变成现实

光写 schema 没用，要让"数据库"和"代码"都知道这个结构。项目 `package.json` 已配好：

```jsonc
"prisma:generate": "prisma generate",     // 生成 Client 代码
"prisma:migrate":  "prisma migrate dev",  // 把表结构同步到数据库
"prisma:studio":   "prisma studio"        // 打开可视化数据库管理界面
```

### 4.1 `prisma generate` —— 生成代码

```bash
pnpm prisma:generate
```

- 作用：根据 `schema.prisma` 生成带类型的 `PrismaClient` 代码（到 `apps/api/src/generated/prisma`）。
- **只生成代码，不碰数据库。**
- 什么时候跑：**每次改完 schema** 都要跑，不然 TS 类型还是旧的。
- 类比：像 `openapi-typescript` 根据接口定义生成 TS 类型。

### 4.2 `prisma migrate dev` —— 同步数据库（建表）

```bash
pnpm prisma:migrate
# 会提示你给这次改动起个名字，如 init / add_message_table
```

- 作用：把 schema 的结构**真正建到数据库里**（建表/改字段），同时生成一份迁移记录存到 `prisma/migrations/`。
- 这个迁移记录**会进 git**，像"数据库结构的 commit 历史"，记录数据库怎么一步步演变的。
- `migrate dev` 通常会**顺便帮你 generate**，所以日常改完 schema 跑一次 migrate 基本就够。
- 类比：像 `git commit`，但提交的是"数据库结构的变化"。

> 记法：
> - `generate` 管**代码类型**（不改数据库）
> - `migrate` 管**数据库表**（顺便也 generate）

### 4.3 `prisma studio` —— 可视化看数据

```bash
pnpm prisma:studio
```

打开浏览器界面，可以直接看表里的数据、手动增删改，调试时非常好用。类比：像一个网页版的数据库 Excel 编辑器。

---

## 5. Prisma Client —— 增删改查（CRUD）

`generate` 后就能用 `PrismaClient` 操作数据库了。所有 model 的方法名通用，**学一次到处用**。

### 5.1 增 Create

```ts
// 建一条会话
const conv = await prisma.conversation.create({
  data: { title: '我的第一个会话' },
})

// 建会话时顺便建关联的消息（嵌套创建）
await prisma.conversation.create({
  data: {
    title: '带消息的会话',
    messages: {
      create: [
        { role: 'user', content: '你好' },
        { role: 'assistant', content: '你好呀' },
      ],
    },
  },
})
```

### 5.2 查 Read

```ts
// 查全部
const all = await prisma.conversation.findMany()

// 按主键查一条
const one = await prisma.conversation.findUnique({ where: { id: 'conv-1' } })

// 带条件查
const filtered = await prisma.conversation.findMany({
  where: { title: { contains: '会话' } },   // title 包含"会话"
  orderBy: { createdAt: 'desc' },           // 按时间倒序
  take: 10,                                  // 只取 10 条（分页）
})

// 查会话时把关联的消息一起查出来
const withMessages = await prisma.conversation.findUnique({
  where: { id: 'conv-1' },
  include: { messages: true },              // 关键：include 关联数据
})
```

### 5.3 改 Update

```ts
await prisma.conversation.update({
  where: { id: 'conv-1' },
  data: { title: '改个名字' },
})
```

### 5.4 删 Delete

```ts
await prisma.message.delete({ where: { id: 'msg-1' } })

// 批量删
await prisma.message.deleteMany({ where: { conversationId: 'conv-1' } })
```

### 5.5 常用查询选项速查

| 选项 | 作用 |
| --- | --- |
| `where` | 过滤条件 |
| `select` | 只取某些字段 |
| `include` | 带出关联数据 |
| `orderBy` | 排序 |
| `take` / `skip` | 分页（取几条 / 跳过几条） |

---

## 6. 在 NestJS 里使用（项目实际结构）

### 6.1 PrismaService —— 把 Client 包成服务

`apps/api/src/prisma/prisma.service.ts`：

```ts
@Injectable()
export class PrismaService extends PrismaClient
  implements OnModuleInit, OnModuleDestroy {
  async onModuleInit()    { await this.$connect() }    // 应用启动时连库
  async onModuleDestroy() { await this.$disconnect() } // 应用关闭时断开
}
```

- `extends PrismaClient`：让这个 Service **本身就是**一个 PrismaClient，拥有全部查询方法。
- `onModuleInit` / `onModuleDestroy`：NestJS 生命周期钩子，类比 Vue 的 `onMounted` / `onUnmounted`——启动就连库，关闭就优雅断开。

### 6.2 PrismaModule —— 导出服务给别人用

`apps/api/src/prisma/prisma.module.ts`：

```ts
@Module({
  providers: [PrismaService],
  exports: [PrismaService],   // 关键：导出，别的模块才能注入
})
export class PrismaModule {}
```

### 6.3 业务里注入使用

```ts
@Injectable()
export class ConversationService {
  constructor(private prisma: PrismaService) {}   // 注入

  findAll() {
    return this.prisma.conversation.findMany()    // 直接用，全程有类型提示
  }

  create(title: string) {
    return this.prisma.conversation.create({ data: { title } })
  }
}
```

> 想用 `PrismaService` 的模块，要在自己的 `@Module({ imports: [PrismaModule] })` 里导入 `PrismaModule`。

---

## 7. 完整工作流（以后固定这个节奏）

每次加表 / 改字段：

```txt
1. 改 prisma/schema.prisma           ← 改图纸（加 model 或字段）
2. pnpm prisma:migrate               ← 同步到数据库（建表）+ 自动 generate
   （会让你输入本次迁移名字，如 add_conversation）
3. 在 Service 里用 this.prisma.xxx    ← 写业务查询，享受类型提示
```

> 前提：数据库容器要先启动（`docker compose up -d postgres`），不然 migrate 连不上。

---

## 8. 前端类比速查表

| Prisma 概念 | 前端类比 |
| --- | --- |
| `schema.prisma` 的 `model` | TS 的 `interface` / `type` |
| `prisma generate` | 根据接口生成 TS 类型（openapi-typescript） |
| `prisma migrate` | git commit，但提交的是数据库结构 |
| `PrismaClient` 的方法 | 封装好的 axios（带类型的请求） |
| `include` / `select` | 接口返回字段裁剪 / 关联展开 |
| `PrismaService` 注入 | Vue 里 inject 一个全局服务 / composable |
| `onModuleInit` | Vue 的 `onMounted` |

---

## 9. 常见问题排查

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| migrate / 启动报连不上库 | 数据库容器没起 | `docker compose up -d postgres` |
| 报 `DATABASE_URL` 未设置 | `.env` 没配 | 按 `.env.example` 在根目录建 `.env` 填 `DATABASE_URL` |
| 改了 schema 但代码没有新类型 | 没重新生成 | `pnpm prisma:generate`（或跑一次 migrate） |
| 想看数据库里到底有啥 | — | `pnpm prisma:studio` 打开可视化界面 |
| 想从零重置数据库结构 | — | `prisma migrate reset`（⚠️ 会清空所有数据，慎用） |

---

## 10. 一句话总结

- Prisma = 用 TS 对象操作数据库的 ORM，表结构自动变成类型。
- 主要写 `schema.prisma` 里的 `model`，像写 interface。
- 改完 schema 跑 `pnpm prisma:migrate`（建表 + 生成类型）。
- 查数据用 `prisma.模型.findMany/create/update/delete`，全程类型提示。
- NestJS 里注入 `PrismaService` 即可使用。
- 改 schema → migrate → 写查询，这是固定三步。
