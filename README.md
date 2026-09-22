<div align="center">

# 知忆 MindVault

**个人知识库系统 · Personal Knowledge Base**

基于 **Vue 3 + Spring Boot 3** 的全栈知识管理平台，集成三级笔记本、富文本笔记、标签、全文搜索、附件与回收站，并内置基于 **DeepSeek** 的 AI 对话与写作助手（SSE 流式输出）。

![Java](https://img.shields.io/badge/Java-17-007396?style=flat-square&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.16-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Vue](https://img.shields.io/badge/Vue-3.5-42B883?style=flat-square&logo=vuedotjs&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-8-646CFF?style=flat-square&logo=vite&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-8-FF4438?style=flat-square&logo=redis&logoColor=white)
![Element Plus](https://img.shields.io/badge/Element%20Plus-2.14-409EFF?style=flat-square&logo=element&logoColor=white)
![DeepSeek](https://img.shields.io/badge/DeepSeek-Chat-4D6BFE?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

</div>

---

## 目录

- [功能特性](#功能特性)
- [技术栈](#技术栈)
- [目录结构](#目录结构)
- [快速开始](#快速开始)
- [配置说明](#配置说明)
- [API 概览](#api-概览)
- [数据库设计](#数据库设计)
- [License](#license)

---

## 功能特性

### 知识管理

- **三级笔记本**：树形结构，支持新建 / 重命名 / 删除，含路径面包屑与返回上级
- **富文本笔记**：基于 wangEditor 5，支持图片上传、代码块、AI 结果一键回填
- **标签体系**：笔记标签管理，按标签统计分布
- **附件上传**：图片 / 文档，按年 / 月分目录存储，50MB 上限
- **全文搜索**：按关键字检索笔记标题与正文
- **置顶与归档**：笔记置顶、归档、逻辑删除与回收站（支持一键清空）
- **数据看板**：概览统计、笔记增长趋势（ECharts）、标签分布环形图

### 账号与权限

- 注册 / 登录，JWT **双 Token**（access 2h + refresh 7d 一次性轮换），前端 40101 自动单飞刷新
- **个人中心**：头像、昵称、生日、个人简介、联系方式、邮箱、修改密码
- **管理后台**：用户管理（禁用 / 修改）、每名用户 AI 对话消耗 token 用量统计

### AI 助手（DeepSeek）

- **多轮对话**：SSE 流式输出、打字机效果、会话历史管理（新建 / 切换 / 删除）
- **写作四动作**：总结、润色、续写、起标题，结果直接插入笔记正文
- **断线续传**：网络中断后按 taskId 自动恢复，任务缓冲 30 分钟
- **分钟限流**：Redis Lua 原子限流（默认 20 次 / 分钟 / 用户），管理员可查看用量

---

## 技术栈

| 端 | 技术 | 版本 |
|---|---|---|
| 前端 | Vue 3（Composition API + `<script setup>`） | 3.5 |
| 前端 | Vite | 8.x |
| 前端 | Element Plus / Pinia / Vue Router | 2.14 / 4.x / 5.x |
| 前端 | ECharts / wangEditor 5 / marked + DOMPurify | 6.x / 5.1 / 18 |
| 前端 | @microsoft/fetch-event-source（SSE 客户端） | 2.x |
| 后端 | Spring Boot | 3.5.16 |
| 后端 | MyBatis-Plus（逻辑删除、雪花 ID、分页） | 3.5.17 |
| 后端 | Spring AI（OpenAI 兼容协议 → DeepSeek） | 1.1.8 |
| 后端 | jjwt / jsoup / hutool | 0.13 / 1.23 / 5.8 |
| 存储 | MySQL 8 | - |
| 缓存 | Redis（双 Token、jti 黑名单、SSE 缓冲、限流） | 6+（8.x 实测） |

---

## 目录结构

```
MindVault1.0/
├─ mindvault-web/                  # 前端 Vue 3 + Vite
│  └─ src/
│     ├─ api/                      # axios 接口封装
│     ├─ components/               # 通用组件（编辑器 / AI 面板 / Markdown 渲染等）
│     ├─ composables/              # SSE 流式封装等组合式函数
│     ├─ layout/                   # 主布局与顶栏
│     ├─ stores/                   # Pinia（AI 会话、应用状态等）
│     ├─ views/                    # 页面（登录、笔记、AI 对话、个人中心、管理后台）
│     └─ router/                   # 路由与守卫
├─ mindvault-server/               # 后端 Spring Boot
│  └─ src/main/
│     ├─ java/com/mindvault/
│     │  ├─ controller/            # REST / SSE 接口
│     │  ├─ service/               # 业务逻辑（AI 流式、回收站、统计等）
│     │  ├─ sse/                   # SSE 任务缓冲、心跳、限流
│     │  ├─ config/                # Jackson / MyBatis-Plus / 异步线程池等
│     │  ├─ entity/ dto/ vo/       # 实体、请求、出参
│     │  └─ common/                # 统一返回、错误码、全局异常
│     └─ resources/
│        ├─ application.yml        # 配置（支持环境变量覆盖）
│        └─ mapper/                # 自定义 SQL（回收站等）
└─ sql/                            # 建表与初始化脚本
   ├─ schema-mini.sql              # 8 张表结构
   ├─ data-mini.sql                # 初始数据（管理员账号、默认笔记本）
   ├─ upgrade-admin.sql            # 管理后台升级脚本
   └─ upgrade-profile.sql          # 个人中心字段升级脚本
```

---

## 快速开始

### 前置要求

| 组件 | 版本 |
|---|---|
| JDK | 17+ |
| Maven | 3.8+ |
| MySQL | 8.0+ |
| Redis | 6+（8.x 实测可用） |
| Node.js | 18+ |

### 1. 初始化数据库

```bash
mysql -u root -p < sql/schema-mini.sql
mysql -u root -p < sql/data-mini.sql
```

`data-mini.sql` 创建初始管理员账号：

| 账号 | 密码 | 说明 |
|---|---|---|
| `admin` | `123456` | 上线前请务必修改 |

### 2. 启动 Redis

```bash
redis-server
redis-cli ping    # 期望输出 PONG
```

### 3. 配置后端

编辑 `mindvault-server/src/main/resources/application.yml`，所有敏感项均可用环境变量覆盖（见 [配置说明](#配置说明)）：

```yaml
spring:
  datasource:
    username: root
    password: 123456        # 改成你的 MySQL 密码
  ai:
    openai:
      api-key: ${DEEPSEEK_API_KEY}   # AI 功能需要；不配置也能启动，仅 AI 接口返回 50010
```

### 4. 启动后端

```bash
cd mindvault-server
mvn spring-boot:run
```

服务地址 `http://localhost:8080`。

### 5. 启动前端

```bash
cd mindvault-web
npm install
npm run dev
```

访问 `http://localhost:5173`，Vite 已将 `/api` 代理到 `localhost:8080`。

---

## 配置说明

以下环境变量可覆盖 `application.yml` 中的默认值：

| 环境变量 | 默认值 | 说明 |
|---|---|---|
| `MYSQL_USERNAME` / `MYSQL_PASSWORD` | `root` / `123456` | MySQL 连接 |
| `REDIS_HOST` / `REDIS_PORT` / `REDIS_PASSWORD` | `127.0.0.1` / `6379` / 空 | Redis 连接 |
| `JWT_SECRET` | 内置开发密钥 | **生产环境必须覆盖**，HS256 需 32 字节以上 |
| `DEEPSEEK_API_KEY` | 无 | DeepSeek API Key，未配置时 AI 接口返回 50010 |
| `DEEPSEEK_BASE_URL` | `https://api.deepseek.com` | DeepSeek 网关地址 |
| `DEEPSEEK_MODEL` | `deepseek-chat` | 模型名 |
| `UPLOAD_DIR` | `./uploads` | 附件存储根目录 |

---

## API 概览

共 35 个 REST / SSE 端点，统一返回 `Result{ code, message, data }`：

| 模块 | 端点 |
|---|---|
| 认证 | `POST /api/auth/login`、`/register`、`/refresh`、`/logout`；`GET/PUT /api/auth/profile`；`PUT /api/auth/password` |
| 笔记本 | `GET /api/notebooks/tree`；`POST /api/notebooks`；`PUT/DELETE /api/notebooks/{id}` |
| 笔记 | `GET /api/notes`；`GET/POST/PUT/DELETE /api/notes/{id}`；`PUT /api/notes/{id}/top`、`/status`、`/restore`；`DELETE /api/notes/trash` |
| 标签 | `GET/POST /api/tags`；`PUT/DELETE /api/tags/{id}` |
| 搜索 | `GET /api/search/keyword` |
| 文件 | `POST /api/files/upload`；`GET /api/files/preview/{fileToken}`；`GET /api/files/{id}/download` |
| AI | `POST /api/ai/{chat,summarize,polish,continue,title}/stream`（SSE）；`GET /api/ai/stream/resume/{taskId}`；`GET /api/ai/conversations`、`/api/ai/conversations/{id}/messages`；`DELETE /api/ai/conversations/{id}` |
| 统计 | `GET /api/stats/overview`、`/note-trend`、`/tag-distribution` |
| 管理 | `GET/PUT/DELETE /api/admin/users`；`GET /api/admin/tokens` |

### 错误码约定

| code | HTTP | 含义 |
|---|---|---|
| 200 | 200 | 成功 |
| 40000 / 40001 | 400 | 参数 / 业务错误 |
| 40100 / 40101 / 40102 | 401 | 未认证 / access 过期（前端自动刷新）/ refresh 失效 |
| 40400 | 404 | 不存在或越权 |
| 42900 | 429 | AI 分钟限流 |
| 50000 | 500 | 系统异常（响应含 traceId 前 8 位） |
| 50010 | SSE error 帧 | AI 上游异常，可重试 |

---

## 数据库设计

共 8 张表：`user`、`notebook`、`note`、`tag`、`note_tag`、`attachment`、`ai_conversation`、`ai_message`。

- 主键为**雪花 ID**，后端 Jackson 全局序列化为字符串下发，前端全程按字符串处理，避免 JS 精度丢失（SSE 帧同样以字符串下发）
- 业务表含 `is_deleted` 逻辑删除；`note_tag`、`ai_message` 不做逻辑删除
- 时间统一 `yyyy-MM-dd HH:mm:ss`（GMT+8），由 `JacksonConfig` 全局控制
- 不建物理外键，越权访问统一返回 404（40400）

---

## 界面预览

> 截图占位，可自行补充：

```
docs/screenshots/
├─ login.png
├─ dashboard.png
├─ note-editor.png
└─ ai-chat.png
```

```markdown
![登录页](docs/screenshots/login.png)
![笔记编辑](docs/screenshots/note-editor.png)
```

---

## License

[MIT](LICENSE)
