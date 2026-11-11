<!-- 中文说明开始 -->
# 📖 中文详细说明（paperclip）

> **阅读顺序说明**：本文件最上面这一大段是**中文详细说明**，用大白话写给第一次接触这个项目的人；下面从 `---` 之后开始，是**上游项目原封不动的原始 README**，一字未删、未改写。
>
> 只想看官方原始文档，可以直接跳到下半部分。

---

## 1️⃣ 这是什么？

一句话说清楚：**一个"AI 公司管理系统"——让你像管理一家公司那样，管理一堆各自干活的 AI 智能体。**

先解释几个名词：

| 名词 | 大白话解释 |
|------|-----------|
| **AI 智能体（Agent）** | 能自己写代码、跑命令、查资料的 AI 程序。Claude Code、Codex、Cursor、Gemini CLI 这些都是 |
| **Paperclip（回形针）** | 项目名。官方一句比喻：**如果 OpenClaw 是个员工，Paperclip 就是这家公司** |
| **心跳（Heartbeat）** | 唤醒机制。智能体平时不动，有任务或定时器触发时才"醒来"去干活 |
| **组织架构图（Org Chart）** | 谁向谁汇报、谁是什么角色、能干什么、花多少钱 |
| **预算 / 硬停** | 给每个 agent、项目、公司设花钱上限，到线了报警或直接暂停 |

打个比方：**你手下有 20 个 AI 员工，但没个 Excel 记谁在干什么、花了多少钱、做完没有。** Paperclip 就是那个 Excel + 打卡机 + 财务部 + 会议室预定系统——**它自己不是一个 AI，它是一层管着这些 AI 的管理层。**

| 关键点 | 说明 |
|--------|------|
| **它是什么** | Node.js 服务端 + React 前端的**编排控制平面（control plane）** |
| **它不是什么** | 不是聊天机器人、不是 agent 开发框架、不是工作流编辑器、不是 prompt 管理器 |
| **支持接入** | OpenClaw、Claude Code、Codex、Cursor（本地 + Cloud）、Gemini CLI、OpenCode、Pi、Hermes + Gateway、Grok Build、Kimi Code，以及任何自定义 HTTP 端点 / CLI / 外部适配器包 |
| **默认端口** | `3100` |
| **数据库** | 本地自动起一个内嵌 PostgreSQL，不用自己装 |
| **技术栈** | TypeScript，pnpm monorepo（turbo），Node.js ≥ 24.11，pnpm ≥ 9.15 |
| **许可证** | MIT |
| **上游规模** | 113,000+ stars，已合并 2700+ PR |

> 💡 官方一句话概括：**「如果 OpenClaw 是员工，Paperclip 就是公司。」**

---

## 2️⃣ 适合谁用？能解决什么问题？

### ✅ 典型使用者

| 你是谁 | 你会得到什么 |
|--------|-------------|
| **同时开着十几个 Claude Code 终端的人** | 任务变成工单，谁在干什么一目了然，重启也不丢 |
| **想跑"自主 AI 组织"的人** | 给 agent 定目标、设角色、定预算，让它们自己协作 |
| **要控制成本的人** | 按公司 / agent / 项目 / 目标 / 任务 / 供应商 / 模型多维度算钱，超预算自动停 |
| **有重复性工作的人** | 定时任务（routines）自动生成工单并派给 agent，不用手动敲 |
| **要审阅 agent 产出的人** | 批准门、决策记录、产物（artifacts）可在文档具体段落上留评论 |
| **想在手机上看着它的人** | 移动端适配，随时查看和干预 |
| **想接自己写的 agent 的人** | 外部适配器包机制，能接心跳的都能接 |

### 🎯 它能帮你解决的具体麻烦

1. **「20 个 Claude Code 标签页，我根本记不住谁在干什么。」**
   → 任务变成工单（ticket），对话变成线程（thread），会话**跨重启持久化**。

2. **「每次开工我得手动把背景资料喂给 bot。」**
   → 上下文从任务往上穿过项目和公司目标自动流动——**agent 永远知道自己该干什么、为什么干**。

3. **「agent 跑飞了，一晚上烧掉几百美元还把额度打满。」**
   → 花费追踪 + 预算预警 + 到线自动暂停。

4. **「每周都要跑一次的报表，我得记得手动启动。」**
   → Routines 按 cron / webhook / API 触发，每次执行都生成一条可追溯的工单。

5. **「两个 agent 抢同一个任务，干出重复的活。」**
   → **原子任务签出（atomic checkout）**：一个负责人 + 执行锁，同一任务不会被两个 run 同时领走。

6. **「agent 改了配置，改错了回不去。」**
   → 治理带版本：批准门强制执行、配置改动有修订历史、坏改动能安全回滚。

7. **「我不知道这个 agent 到底代表谁在操作我的 GitHub。」**
   → **责任用户身份（responsible-user identity）**：受管 GitHub 操作可以用指挥者的账号。

### ❌ 不适合谁

- 只是想**跑一个 AI 编程助手**的（直接用 Claude Code 就够了，Paperclip 是给"管理多个"用的）
- 想**自己写 agent 框架**的（Paperclip 明确说"我们不告诉你怎么造 agent，我们告诉你怎么管一群 agent 组成的公司"）
- 需要**完整任务系统接管**的（自带 Jira / Linear / Asana 的团队，Paperclip 目前还只做"工单"这一层）

---

## 3️⃣ 核心功能

官方把系统拆成 12 块，下面按大白话逐条拆开：

### 🧱 四大支柱

| 支柱 | 干什么 | 大白话 |
|------|--------|--------|
| **Agentic Task Manager** | 声明意图 → agent 干活 → 你验收 | 像用任务管理器管 AI 员工 |
| **Org Chart for Agents** | 角色、权限、边界 | 给 AI 员工定岗位、定汇报关系、定权限范围 |
| **Agent Employee Training** | 设计、训练、评估 AI 员工 | 给 AI 员工做技能培训、搞考核 |
| **Agentic OS** | 让活儿能跑起来的基建 | 沙箱、集成、MCP 服务器、SSO、RBAC、成本控制 |

### ⚙️ 具体功能清单

| # | 功能 | 大白话 |
|---|------|--------|
| 1 | **自带任意 Agent（Bring Your Own Agent）** | 一个组织架构图装下所有 agent，官方说法：**"只要它能收到心跳，就算被录用了"** |
| 2 | **目标对齐（Goal Alignment）** | 任务和项目挂到组织目标上，agent 收到的是目标背景，不只是一个标题 |
| 3 | **心跳（Heartbeats）** | 有任务、后续消息或定时器时唤醒 agent；任务可以沿组织架构上下委派 |
| 4 | **成本控制** | 公司 / agent / 项目三级预算；跟踪已报花费、到阈值报警、超限自动暂停 |
| 5 | **多组织** | 一次部署管多个组织，任务、agent、权限、活动历史互相隔离 |
| 6 | **任务线程** | 对话、计划、阻塞项、文件、运行历史都挂在任务上，可以派给 agent 或人 |
| 7 | **治理** | 审查/批准阶段、批准招聘、暂停 / 改派 / 终止工作 |
| 8 | **组织架构图** | 层级、角色、汇报线，agent 有老板、有头衔、有 JD |
| 9 | **移动端** | 手机上随时监控和管理 |
| 10 | **应用与连接** | 接 GitHub、Notion、Railway，或自己的 MCP server；网关动作可设"允许 / 先问 / 关闭" |
| 11 | **共享 Agent、个人账号** | 决定谁能用某个连接、哪些 agent 能访问；受管 GitHub 操作可用指挥者的账号 |
| 12 | **技能与 Skill Studio** | 装或写共享技能、用保存的输入测试、看结果、回滚到旧版本 |
| 13 | **定时例行任务** | 按计划或 API/webhook 触发，每次运行都有工单、负责人和历史 |
| 14 | **产物与反馈** | 找 agent 产出的文件和文档，预览格式，在文档具体段落上留评论 |
| 15 | **现成团队模板** | 预览并安装带角色、技能、项目、例程的团队包 |
| 16 | **Agent Chat 与聊天/邮件连接器** | 实验特性：Slack、Discord、Telegram、AgentMail 里直接跟 agent 聊 |
| 17 | **插件系统** | 进程外 worker + 能力门控 host 服务 + 作业调度 + 工具暴露 + UI 贡献，**不改 Paperclip 源码就能扩展** |
| 18 | **密钥与存储** | 公司密钥和个人密钥值、加密凭据、本地或 S3 兼容存储、附件、产物 |
| 19 | **活动与事件** | 所有变更、心跳状态变化、成本事件、批准、评论都记成可审计的活动流水 |
| 20 | **组织可移植** | 导出 / 导入整个组织（agent、技能、项目、例程、任务、附件），带密钥脱敏和冲突处理 |
| 21 | **多模型多运行时** | 每个 agent 可以单独选模型和 harness |
| 22 | **深度规划** | 规划模式、带修订的计划、计划批准 |
| 23 | **结果强制** | 看门狗、恢复动作、审查门 |

### 🏗 底下的 12 个子系统

```
┌──────────────────────────────────────────────────────────────┐
│                       PAPERCLIP SERVER                       │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │Identity & │  │  Work &   │  │ Heartbeat │  │Governance │  │
│  │  Access   │  │   Tasks   │  │ Execution │  │& Approvals│  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Org Chart │  │Workspaces │  │  Plugins  │  │  Budget   │  │
│  │ & Agents  │  │ & Runtime │  │           │  │ & Costs   │  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Routines  │  │ Secrets & │  │ Activity  │  │  Company  │  │
│  │& Schedules│  │  Storage  │  │ & Events  │  │Portability│  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
└──────────────────────────────────────────────────────────────┘
         ▲              ▲              ▲              ▲
   ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐
   │  Claude   │  │   Codex   │  │   CLI     │  │ HTTP/web  │
   │   Code    │  │           │  │  agents   │  │   bots    │
   └───────────┘  └───────────┘  └───────────┘  └───────────┘
```

---

## 4️⃣ 怎么快速上手？

> ⚠️ 下面的命令全部来自本仓库真实文件（`README.md` 原文、`package.json`、`docker/docker-compose.quickstart.yml`、`docker/docker-compose.yml`、`.env.example`、`.nvmrc`），不是编的。

### 前置条件

| 依赖 | 版本 | 必需？ | 说明 |
|------|------|--------|------|
| Node.js | **24.11+** | ✅ | `.nvmrc` 里写的是 `24` |
| pnpm | **9.15+** | ✅ | |
| PostgreSQL | — | ❌ | 本地开发会自动起一个内嵌的 |
| Python 3 | — | ⚠️ | 只有用**本地 Claude / Codex 订阅登录**才需要（Docker 镜像已内置） |
| Rust 工具链 | — | ⚠️ | 源码开发要构建原生 Paperclip Runner；也可以设 `PAPERCLIP_RUNNER_BINARY` 指向预编译版 |

### 方式 A：最省事——让 agent 帮你装

把[安装指南](https://docs.paperclip.ing/reference/cli/installation/)发给你的 agent，让它照着做。

### 方式 B：npx 一条命令（推荐）

```bash
# 需要 Node.js 24.11 或更高版本
npx paperclipai@latest onboard --yes
```

CLI 从 npm 缓存运行，**实例配置和数据都留在本地**。

#### 先跑个试试（不装服务、不建任务）

```bash
# 用 Claude
ANTHROPIC_API_KEY=... npx paperclipai test-drive

# 用 Codex
OPENAI_API_KEY=... npx paperclipai test-drive --harness codex

# 用 OpenCode + OpenRouter 的模型
OPENROUTER_API_KEY=... npx paperclipai test-drive \
  --harness opencode \
  --model openrouter/anthropic/claude-sonnet-4.5
```

`test-drive` 的特点：

| 特性 | 说明 |
|------|------|
| **留在前台** | 不会装成系统服务，也不会创建首个任务 |
| **成功后才开浏览器** | 初始化失败不会留下半个开着的页面 |
| **每次独立目录** | 不带 `--data-dir` 时每次用一个新的、保留的临时目录，启动时打印绝对路径 |
| **复用目录** | 传 `--data-dir` 就能复用同一个 |
| **不开浏览器** | 加 `--no-browser` |
| **Git worktree** | 从已链接的 Git worktree 调用时，会在该 worktree 里启用任务执行 |

#### 绑定方式（决定要不要登录）

```bash
# 默认：本地回环可信模式，最快跑起来
npx paperclipai@latest onboard --yes

# 局域网
npx paperclipai@latest onboard --yes --bind lan

# Tailnet（私有网络）
npx paperclipai@latest onboard --yes --bind tailnet
```

已经配置过的实例，**重跑 `onboard` 会保留原配置**。要改设置用：

```bash
npx paperclipai configure
```

> ⚠️ **私有 npm registry 的坑**：如果你在全局 `~/.npmrc` 里配了私有源（比如 GitHub Packages），`npx` 会拿 `paperclipai` 去私有源找，结果 `E404`。
>
> 诊断：`npm config get registry`
>
> 绕过（强制走公共源）：
>
> ```bash
> npx --registry https://registry.npmjs.org paperclipai@latest onboard --yes
> ```

### 方式 C：手动源码运行（开发者）

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip
pnpm install
pnpm dev
```

- UI 和 API 都在 `http://localhost:3100`
- **内嵌 PostgreSQL 会自动创建**，不需要任何配置

### 方式 D：Docker 单容器（自带数据库）

`docker/docker-compose.quickstart.yml` 里有完整注释，先读一遍：

```bash
cd docker

# 在 .env 里设两个密钥，各用一次独立的 openssl rand -hex 32
# 重启时复用，不要替换已有部署的密钥
export ANTHROPIC_API_KEY="sk-ant-..."

docker compose -f docker-compose.quickstart.yml up --build
```

关键环境变量：

| 变量 | 作用 |
|------|------|
| `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` | LLM 供应商密钥，**至少要有一个**，否则 agent 没法干活 |
| `PAPERCLIP_DEPLOYMENT_MODE` | `authenticated` 需要登录 / `local_trusted` 跳过鉴权 |
| `PAPERCLIP_DEPLOYMENT_EXPOSURE` | `private` 仅局域网 / `public` 面向公网（**要配 TLS**） |
| `PAPERCLIP_PUBLIC_URL` | 用户访问 UI 的地址 |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | 除 public URL 主机名外额外接受的主机名（局域网或 Tailscale 别名），逗号分隔 |
| `BETTER_AUTH_SECRET` | **必填**，会话签名密钥，用 `openssl rand -hex 32` 生成 |
| `PAPERCLIP_TOOL_ACTION_SIGNING_SECRET` | 独立的密钥，用于签名需要审批的工具动作 |
| `PAPERCLIP_PORT` | 端口，默认 `3100` |
| `PAPERCLIP_DATA_DIR` | 持久化数据目录，默认 `../data/docker-paperclip` |

### 方式 E：Docker 双服务（自带 Postgres）

`docker/docker-compose.yml` 提供 `db`（`postgres:17-alpine`）+ `server` 两个服务：

```bash
cd docker

# BETTER_AUTH_SECRET 必须设，否则 compose 会直接报错退出
export BETTER_AUTH_SECRET="$(openssl rand -hex 32)"
export PAPERCLIP_PUBLIC_URL="http://localhost:3100"

docker compose -f docker-compose.yml up --build
```

### 常用开发命令

```bash
pnpm dev              # 完整开发环境（API + UI，watch 模式）
pnpm dev:once         # 完整开发环境，不监听文件变化
pnpm dev:server       # 只起服务端
pnpm dev:mobile       # 在 :3101 起预构建 UI 供手机/平板访问（把 /api 代理到 :3100）
pnpm dev:both         # 同时跑 dev 和 dev:mobile
pnpm build            # 构建全部
pnpm typecheck        # 类型检查
pnpm test             # 便宜的默认测试（只跑 Vitest）
pnpm test:watch       # Vitest watch 模式
pnpm test:e2e         # Playwright 浏览器测试
pnpm db:generate      # 生成数据库迁移
pnpm db:migrate       # 应用迁移
```

> `pnpm test` **不包含** Playwright。浏览器测试套件是独立的，通常只在做相关功能或在 CI 里跑。完整开发指南见 `doc/DEVELOPING.md`。

### 插件作者看这个

```bash
pnpm build:feature-catalog   # 生成功能目录
```

---

## 5️⃣ 常见问题 / 注意事项

### ❓ 会踩的坑

| 现象 | 原因 | 怎么办 |
|------|------|--------|
| **`npx` 报 `E404` 找不到 `paperclipai`** | 全局 `~/.npmrc` 配了私有 npm 源（GitHub Packages 等），`npx` 去私有源解析了 | `npm config get registry` 确认；用 `npx --registry https://registry.npmjs.org paperclipai@latest onboard --yes` |
| **agent 起来了但什么都干不了** | 没配 LLM 供应商密钥 | 至少配 `OPENAI_API_KEY` 或 `ANTHROPIC_API_KEY` |
| **compose 启动直接退出** | `BETTER_AUTH_SECRET` 没设（compose 里写的是 `:?` 必填校验） | `export BETTER_AUTH_SECRET="$(openssl rand -hex 32)"` |
| **本地订阅登录用不了** | 缺 Python 3 或对应的供应商 CLI | 装上；用 Docker 的话镜像里已经带了 |
| **源码开发报 Runner 相关错** | 需要构建原生 Paperclip Runner | 装 Rust 工具链，或设 `PAPERCLIP_RUNNER_BINARY` 指向预编译版 |
| **改了 `onboard` 的配置没生效** | 已配置过的实例重跑 `onboard` 会**保留原配置** | 用 `npx paperclipai configure` 改设置 |
| **`pnpm test` 里有 Playwright 相关的失败** | `pnpm test` 本来就不含浏览器套件 | 用 `pnpm test:e2e` 单独跑 |
| **重启后数据没了** | 数据目录没挂载 | Docker 场景检查 `PAPERCLIP_DATA_DIR` 那个 volume；源码场景检查 `PAPERCLIP_HOME` |
| **想从公网访问但没配 TLS** | `PAPERCLIP_DEPLOYMENT_EXPOSURE=public` 需要 TLS | 按 `doc/DEPLOYMENT-MODES.md` 和 `doc/DOCKER.md` 配反代；见 `deploy/` 相关的边缘安全文档 |
| **导入的组织包带出了本地路径** | 导出只脱敏引用的密钥值，普通环境值和本地路径会保留 | **分享前务必人工审阅导入包** |
| **agent 跑飞烧钱** | 没有预算上限 | 按公司 / agent / 项目设预算和硬停 |

### ⚠️ 必须知道的注意事项

1. **两种部署模式** —— `trusted local`（本地回环可信，跳过登录）和 `authenticated`（需要登录）。**公网暴露必须用 authenticated + TLS。**
2. **遥测默认开启** —— Paperclip 收集匿名使用遥测。**不收集**个人信息、issue 内容、prompt、文件路径或密钥；私有仓库引用会用每个安装的盐做哈希。关闭方式（任选其一）：

   | 方式 | 怎么设 |
   |------|--------|
   | 环境变量 | `PAPERCLIP_TELEMETRY_DISABLED=1` |
   | 标准约定 | `DO_NOT_TRACK=1` |
   | CI 环境 | `CI=true` 时自动关闭 |

3. **可观测性是可选的** —— OpenTelemetry 自动埋点（仅 trace）需要设 `OTEL_EXPORTER_OTLP_ENDPOINT`；Sentry 需要设 `SENTRY_DSN_FRONTEND` / `SENTRY_DSN_BACKEND`。这些 SDK 都是**可选 peer dependency**，不想要就别装。
4. **会话持久化** —— 任务、对话、会话状态存在 Paperclip 里，**支持适配器会跨 run 恢复保存的会话**。这和"只开一个标签页"的 agent 工具完全不同。
5. **组织隔离** —— 一次部署可以管多个组织，但每个组织的数据和访问检查是隔离的。
6. **本地 Claude / Codex 登录需要额外依赖** —— Python 3 + 对应的供应商 CLI。
7. **导入包不是完整实例备份** —— 组织包共享的是"一套配置"，**完整实例恢复要用备份**。
8. **它有内置的 PR review 流程** —— 但那只是其中一块能力，不是全部；做研究、运维、内容等其它工作也一样用。

---

## 6️⃣ 目录结构速览

```
paperclip/
├── cli/                           ← 独立的 CLI（paperclipai）
│   ├── src/
│   │   ├── commands/              # 各子命令
│   │   │   ├── onboard.ts         # 首次引导安装
│   │   │   ├── configure.ts       # 修改配置
│   │   │   ├── test-drive.ts      # 隔离试跑
│   │   │   ├── install.ts / uninstall.ts / update.ts
│   │   │   ├── doctor.ts          # 环境体检
│   │   │   ├── run.ts / heartbeat-run.ts
│   │   │   ├── worktree.ts / git-workspace.ts   # Git worktree 工作区
│   │   │   ├── routines.ts        # 例行任务
│   │   │   ├── channels.ts        # 聊天频道
│   │   │   ├── mcp.ts             # MCP 管理
│   │   │   ├── service.ts         # 服务管理
│   │   │   ├── db-backup.ts       # 数据库备份
│   │   │   ├── env.ts / env-lab.ts / allowed-hostname.ts
│   │   │   ├── managed-agent.ts / auth-bootstrap-ceo.ts
│   │   │   └── runtime.ts
│   │   ├── adapters/  client/  config/  services/  prompts/  utils/
│   │   ├── index.ts  onboard-service.ts  install-store.ts  runtime-info.ts
│   │   └── telemetry.ts  update-notice.ts
│   └── package.json
├── server/                        ← 服务端（TypeScript）
│   └── src/
│       ├── app.ts / index.ts      # 应用装配与入口
│       ├── adapters/              # 适配器调用层
│       ├── auth/                  # 鉴权（会话、API Key、运行 JWT）
│       ├── http/                  # HTTP 路由
│       ├── built-ins/             # 内置能力
│       ├── agent-auth-jwt.ts      # agent 短期运行 JWT
│       ├── agent-run-cancellation.ts
│       ├── embedded-postgres-supervisor.ts  # 内嵌 PG 进程管理
│       ├── first-admin-claim.ts   # 首个管理员认领
│       ├── config.ts / config-file.ts / env-file-policy.ts
│       ├── instrumentation.ts     # OTel 埋点
│       └── home-paths.ts          # PAPERCLIP_HOME 路径解析
├── ui/                            ← React 前端
│   ├── src/                       # 页面、组件、状态
│   ├── public/brands/             # 各 agent 品牌图标
│   ├── storybook/                 # 组件故事书
│   └── vite.config.ts
├── packages/                      ← monorepo 内部包
│   ├── db/                        # 数据库层（Prisma 风格 schema + 迁移）
│   ├── shared/                    # 共享类型与工具（含 telemetry 契约）
│   ├── adapters/                  # ★ 各 agent 适配器
│   │   ├── claude-local/          # Claude Code 本地
│   │   ├── codex-local/           # Codex 本地
│   │   ├── cursor-local/  cursor-cloud/
│   │   ├── gemini-local/  opencode-local/  pi-local/  kimi-local/
│   │   ├── grok-local/            # Grok Build
│   │   ├── hermes/  hermes-gateway/
│   │   ├── openclaw-gateway/      # OpenClaw
│   │   └── AUTHORING.md           # ★ 写新适配器的指南
│   ├── plugins/                   # 插件系统
│   │   ├── sdk/                   # 插件 SDK
│   │   ├── create-paperclip-plugin/  # 插件脚手架
│   │   ├── sandbox-providers/     # 沙箱提供方（e2b、Cloudflare、Daytona...）
│   │   ├── examples/              # 示例插件
│   │   ├── plugin-llm-wiki/  plugin-workspace-diff/
│   │   └── paperclip-plugin-fake-sandbox/
│   ├── paperclip-runner/          # 原生 Runner（Rust）
│   ├── paperclip-eval-kernel/     # 评测内核
│   ├── skills-catalog/            # 技能目录
│   ├── teams-catalog/             # 团队模板目录
│   ├── mcp-server/                # MCP 服务器
│   ├── google-sheets-mcp-server/  # Google Sheets MCP
│   ├── kv-demo-mcp-server/        # KV 示例 MCP
│   ├── adapter-utils/             # 适配器公共工具
│   └── tailscale-https-broker/    # Tailscale HTTPS broker
├── integrations/
│   └── assistant-plugins/         # 桌面助手插件（claude / openai / shared）
├── docker/
│   ├── Dockerfile.onboard-smoke   # 引导流程冒烟测试镜像
│   ├── docker-compose.yml         # 双服务版（db + server）
│   ├── docker-compose.quickstart.yml  # ★ 单容器版（自带数据库）
│   ├── docker-compose.untrusted-review.yml
│   ├── agent-runtime/  daytona-runner/  hermes-gateway-smoke/  openclaw-smoke/
│   ├── untrusted-review/          # 低信任 PR 评审环境
│   ├── quadlet/                   # systemd Quadlet 单元
│   └── ecs-task-definition.json   # AWS ECS 任务定义
├── cli/  server/  ui/  packages/  tests/  evals/  skills/  skills-releases/
├── doc/                           ← ★ 大量设计文档（140+ 个 .md）
│   ├── GOAL.md / PRODUCT.md       # ★ 目标与产品定义（改代码前必读）
│   ├── SPEC.md                    # 长期产品上下文
│   ├── SPEC-implementation.md     # ★ V1 具体实现契约
│   ├── DEVELOPING.md / DATABASE.md / DOCKER.md / INSTALLING.md
│   ├── DEPLOYMENT-MODES.md / LOW-TRUST-PRESETS.md
│   ├── CLI.md / AGENTS.md / AGENT-IDENTITY.md / AGENT-LIFECYCLE.md
│   ├── TASKS.md / GOAL.md / CHANNELS.md / MCP-ACCESS-GOVERNANCE.md
│   ├── observability.md / TELEMETRY_WORKFLOW.md / prompt-injection-prevention.md
│   ├── architecture/  design/  plans/  spec/  connections/  testing/
│   ├── benchmarks/  logs/  screenshots/  assets/
│   └── ...（还有大量 idle-*、runner-*、workspace-* 等专题文档）
├── docs/                          ← Mintlify 文档站（docs.paperclip.ing）
│   ├── docs.json  guides/  api/  cli/  deploy/  adapters/  companies/
│   └── plans/  specs/  performance/  start/  assets/
├── maps/                          # 架构地图
├── rules/                         # 分域规则（*.mdc，含 globs / alwaysApply）
├── .agents/ .claude/ .codex/ .devin/  # 各 AI 助手的项目级指令
├── scripts/                       # 构建、发布、冒烟测试脚本
├── tools/  evals/  cli/  screensh
├── AGENTS.md                      # ★ 给人和 AI 贡献者的总指引（先读这个）
├── CONTRIBUTING.md / SECURITY.md / ROADMAP.md / DESIGN.md
├── Dockerfile                     # 主镜像（内置 tini 作 PID 1）
├── docker-compose*.yml
├── package.json / turbo.json / pnpm-workspace.yaml / pnpm-lock.yaml
├── tsconfig.json / tsconfig.base.json
├── .env.example / .env.runner-e2e.example
├── .nvmrc                         # Node 版本（24）
├── .npmrc  .gitattributes  .mailmap
├── .github/                       # CI、发布、代码审查
├── .greptile/                     # 代码审查机器人配置
└── LICENSE                        # MIT
```

### 官方给贡献者的阅读顺序

`AGENTS.md` 里写得很明确，改代码前按这个顺序读：

1. `doc/GOAL.md`
2. `doc/PRODUCT.md`
3. `doc/SPEC-implementation.md`
4. `doc/DEVELOPING.md`
5. `doc/DATABASE.md`

> `doc/SPEC.md` 是长期产品上下文，`doc/SPEC-implementation.md` 才是具体的 V1 构建契约。

改 Apps 目录连接的话，还要额外读 `doc/connections/CONNECTOR-PLAYBOOK.md`。

编辑或评审文件前，读 `rules/*.mdc` 里作用域匹配的规则（`globs` 是逗号分隔的仓库相对路径模式，`alwaysApply: true` 表示全仓库生效）。

---

## 7️⃣ 本仓库说明（Fork 信息）

| 项目 | 说明 |
|------|------|
| **本仓库** | `aiwozhonghua81/paperclip` |
| **上游项目** | [`paperclipai/paperclip`](https://github.com/paperclipai/paperclip) |
| **项目名** | Paperclip |
| **关系** | **Fork（派生仓库）** |
| **当前分支** | `master` |
| **本地 HEAD** | `c0550e701 fix(server): preserve sessions and defer busy overlap recovery (#18552)`（2026-11-01） |
| **上游 HEAD**（拉取时） | `84fc67317 feat(runner): export authorized read responses as artifacts (#20606)`（2026-11-11） |
| **许可证** | MIT |
| **上游规模** | 113,068 stars |

### 关于这个 fork 的现状（实测数据）

用 `git rev-list --left-right --count upstream/master...master` 对比：

```
1377    0
 ↑       ↑
上游独有  本地独有
1377 个  0 个
```

并且 `git merge-base master upstream/master` 返回的就是**本地 HEAD 本身**（`c0550e701`）。

这说明：

| 事实 | 说明 |
|------|------|
| **本 fork 是上游的纯镜像** | 本地没有自己的独占提交，代码和上游**一模一样** |
| **本地落后上游 1377 个提交** | 上游从 2026-11-01 到 2026-11-11 这 10 天里又合了 1377 个 PR（Paperclip 迭代极快） |
| **本 fork 唯一新增的内容** | **README 顶部的这段中文详细说明**（以及仓库 description 里补的中文注解） |
| ⚠️ **上游迭代非常快** | 10 天 1377 个提交，平均每天上百个 PR。**本 fork 明显跟不上**，要用新功能请直接用上游 |

### 关于「克隆下来只有 1 个文件」这件事（已排查并修复）

初次克隆时这个仓库的 `git ls-files` 返回空、工作区只有一个 `README.md`，原因是 **clone 过程在检出（checkout）阶段被中断**，留下了：

- `.git/index` 文件缺失
- `.git/index.lock` 残留锁文件
- 工作区只落地了一部分文件（磁盘上只有 2749 个文件，而仓库实际有 13319 个）

已做的修复：

```bash
rm -f .git/index.lock      # 清掉残留锁
git read-tree HEAD          # 用 HEAD 重建索引
git checkout-index -a -f    # 把 13319 个文件全部检出到工作区
```

修复后 `git status` 干净（0 个改动），文件数与 `git ls-files` 一致（13319 个），仓库可正常使用。

> 📌 **这不是"空仓库"也不是"极简项目"**。Paperclip 是一个**完整的大型 TypeScript monorepo**（13000+ 文件，pnpm + turbo 多包结构），只是首次 clone 被中断了。

### 上游项目简介

> **Paperclip** 是一个开源的 AI agent 编排平台，用来管理工作中的 AI 智能体。
>
> 官方比喻：**如果 OpenClaw 是个员工，Paperclip 就是这家公司。**
>
> 它是一个 Node.js 服务端 + React UI，把一支 AI agent 队伍组织起来去完成一个业务目标。你可以自带各种 agent（Claude Code、Codex、Cursor、Gemini CLI、OpenClaw 等），给它们分配目标，并在一个仪表盘里跟踪工作进展和成本。每个 agent 可以单独选模型和 harness，而团队的任务、技能、权限和历史都集中在一处。
>
> 它看起来像个任务管理器，但底层是组织架构图、预算、治理、目标对齐和 agent 协作。开源、MIT 许可、可自托管，不需要 Paperclip 账号。

### 本 README 的结构

```
第 1 行
├── <!-- 中文说明开始 -->
│   ├── 1️⃣ 这是什么？
│   ├── 2️⃣ 适合谁用 / 能解决什么问题
│   ├── 3️⃣ 核心功能
│   ├── 4️⃣ 怎么快速上手
│   ├── 5️⃣ 常见问题 / 注意事项
│   ├── 6️⃣ 目录结构速览
│   └── 7️⃣ 本仓库说明（Fork 信息）  ← 你正在看的最后一段
├── <!-- 中文说明结束 -->
│
├── （分隔线）
│
└── ↑↑↑ 从这里开始，是上游项目 paperclipai/paperclip 的原始 README，一字未改 ↑↑↑
```

上游 README 里还有 **Roadmap**（完整版在 `ROADMAP.md`）、**「Paperclip 不是什么」对照表**、**系统架构图**、**FAQ**、**遥测与可观测性说明**，本 fork 一并保留。

---

<!-- 中文说明结束 -->

---

<p align="center">
  <img src="doc/assets/banner.jpg" alt="Paperclip is the app people use to manage AI agents for work." width="720" />
</p>

<p align="center">
  <a href="#quickstart"><strong>Quickstart</strong></a> &middot;
  <a href="https://docs.paperclip.ing"><strong>Docs</strong></a> &middot;
  <a href="https://github.com/paperclipai/paperclip"><strong>GitHub</strong></a> &middot;
  <a href="https://discord.gg/paperclipai"><strong>Discord</strong></a> &middot;
  <a href="https://x.com/papercliping"><strong>Twitter</strong></a> &middot;
  <a href="https://paperclip.ing"><strong>Website</strong></a>
</p>

<p align="center">
  <a href="https://github.com/paperclipai/paperclip/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="MIT License" /></a>
  <a href="https://github.com/paperclipai/paperclip/stargazers"><img src="https://img.shields.io/github/stars/paperclipai/paperclip?style=flat" alt="Stars" /></a>
  <a href="https://discord.gg/paperclipai"><img src="https://img.shields.io/badge/discord-join-7289da" alt="Discord" /></a>
  <a href="https://superagent.sh"><picture><source media="(prefers-color-scheme: dark)" srcset="https://www.superagent.sh/api/badge/paperclipai/paperclip/shield.svg?theme=dark&accent=teal&variant=scanned" /><img src="https://www.superagent.sh/api/badge/paperclipai/paperclip/shield.svg?theme=light&accent=teal&variant=scanned" alt="Security posture verified by Superagent" /></picture></a>
</p>

<p align="center">
  <a href="https://www.star-history.com/paperclipai/paperclip"><img src="https://api.star-history.com/badge?repo=paperclipai/paperclip" alt="Star History Rank" /></a>
</p>

<br/>

<div align="center">
  <video src="https://github.com/user-attachments/assets/773bdfb2-6d1e-4e30-8c5f-3487d5b70c8f" width="600" controls></video>
</div>

<p align="center">
  <a href="https://paperclip.ing/waitlist/"><strong>Sign up for the Paperclip Cloud waitlist →</strong></a>
</p>

<br/>

# Paperclip is the app people use to manage AI agents for work.

Open-source orchestration for teams of AI agents.

**If OpenClaw is an _employee_, Paperclip is the _company_.**

Paperclip is a Node.js server and React UI that orchestrates a team of AI agents to run a business. Bring your own agents, assign goals, and track work and costs from one dashboard. Choose models and harnesses per agent while keeping your team's tasks, skills, permissions, and history in one place.

It looks like a task manager. Under the hood: org charts, budgets, governance, goal alignment, and agent coordination.

**Manage business goals, not pull requests.**

|        | Step            | Example                                                            |
| ------ | --------------- | ------------------------------------------------------------------ |
| **01** | Define the goal | _"Build the #1 AI note-taking app to $1M MRR."_                    |
| **02** | Hire the team   | CEO, CTO, engineers, designers, marketers — any bot, any provider. |
| **03** | Approve and run | Review strategy. Set budgets. Hit go. Monitor from the dashboard.  |

<br/>

<div align="center">
<table>
  <tr>
    <td align="center" rowspan="2"><strong>Works<br/>with</strong></td>
    <td align="center" valign="top"><img src="doc/assets/logos/openclaw.svg" width="32" height="32" alt="OpenClaw" /><br/><sub>OpenClaw</sub></td>
    <td align="center" valign="top"><img src="doc/assets/logos/claude.svg" width="32" height="32" alt="Claude Code" /><br/><sub>Claude Code</sub></td>
    <td align="center" valign="top"><img src="ui/public/brands/codex-color.svg" width="32" height="32" alt="Codex" /><br/><sub>Codex</sub></td>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/adapters/cursor-dark.svg" /><img src="ui/public/brands/adapters/cursor.svg" width="32" height="32" alt="Cursor and Cursor Cloud" /></picture><br/><sub>Cursor<br/>+ Cloud</sub></td>
    <td align="center" valign="top"><img src="ui/public/brands/adapters/gemini-color.svg" width="32" height="32" alt="Gemini CLI" /><br/><sub>Gemini CLI</sub></td>
  </tr>
  <tr>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/opencode-logo-dark-square.svg" /><img src="ui/public/brands/opencode-logo-light-square.svg" width="32" height="32" alt="OpenCode" /></picture><br/><sub>OpenCode</sub></td>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/adapters/pi-dark.svg" /><img src="ui/public/brands/adapters/pi.svg" width="32" height="32" alt="Pi" /></picture><br/><sub>Pi</sub></td>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/adapters/hermesagent-dark.svg" /><img src="ui/public/brands/adapters/hermesagent.svg" width="32" height="32" alt="Hermes and Hermes Gateway" /></picture><br/><sub>Hermes<br/>+ Gateway</sub></td>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/adapters/grok-dark.svg" /><img src="ui/public/brands/adapters/grok.svg" width="32" height="32" alt="Grok Build" /></picture><br/><sub>Grok Build</sub></td>
    <td align="center" valign="top"><picture><source media="(prefers-color-scheme: dark)" srcset="ui/public/brands/adapters/kimi-color.svg" /><img src="ui/public/brands/adapters/kimi-color-light.svg" width="32" height="32" alt="Kimi Code" /></picture><br/><sub>Kimi Code</sub></td>
  </tr>
</table>

<em>If it can receive a heartbeat, it's hired.</em>

</div>

Custom processes, HTTP endpoints, and external adapter packages extend the roster. See the [adapter overview](https://docs.paperclip.ing/reference/adapters/overview/) for setup and capabilities.

<br/>

## Paperclip is right for you if

- ✅ You want to build **autonomous AI organizations**
- ✅ You **coordinate many different agents** (OpenClaw, Codex, Claude, Cursor) toward a common goal
- ✅ You have **20 simultaneous Claude Code terminals** open and lose track of what everyone is doing
- ✅ You want agents running **autonomously 24/7**, but still want to audit work and chime in when needed
- ✅ You want to **monitor costs** and enforce budgets
- ✅ You want a process for managing agents that **feels like using a task manager**
- ✅ You want to manage your autonomous businesses **from your phone**

<br/>

## The four pillars

Four things have to work for an organization of AI agents to actually produce: the tasks, the org, the training, and the infrastructure. Paperclip is built around exactly those four pillars.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/paperclipai/paperclip/1ec33ffd8b597f7e36aac3e2fbb4665b8c42dc3c/doc/assets/four-pillars-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/paperclipai/paperclip/1ec33ffd8b597f7e36aac3e2fbb4665b8c42dc3c/doc/assets/four-pillars-light.png">
  <img src="https://raw.githubusercontent.com/paperclipai/paperclip/1ec33ffd8b597f7e36aac3e2fbb4665b8c42dc3c/doc/assets/four-pillars-light.png" alt="The four pillars of Paperclip">
</picture>

| Pillar | Built for | What it covers |
| --- | --- | --- |
| **Agentic Task Manager** — Declare intent. Agents work. You verify the output. | Everyone, daily | Tasks, approvals & review gates · proactive agent coworkers · auditable routines & workflows · verify from diffs, screenshots & tests |
| **Org Chart for Agents** — Roles, permissions & boundaries for humans and agents. | Managers | Mixed human + agent org chart · responsibilities, delegation, specialization · governance: who can do what · scoped secrets & company boundaries · connection permissions & responsible-user identities |
| **Agent Employee Training** — Design, train & evaluate your AI employees. | Enablers | Skill Studio & shared org-wide skills · evals & saved test runs · active learning loops & quality metrics · performance reviews for agents · saved test inputs · skill version history & restore · reusable team templates |
| **Agentic OS** — The infrastructure that makes the work run. | IT & platform | Cross-provider runtime: any model, any agent · sandboxing, integrations & MCP servers · SSO, GRC, RBAC & cost controls · data privacy, internal trace collection, compounding data value · personal & shared app connections · run history & opt-in tracing |

<br/>

## Features

<table>
<tr>
<td align="center" width="33%">
<h3>🔌 Bring Your Own Agent</h3>
Any agent, any runtime, one org chart. If it can receive a heartbeat, it's hired.
</td>
<td align="center" width="33%">
<h3>🎯 Goal Alignment</h3>
Link tasks and projects to your organization goals. Agents receive the goal context behind their work.
</td>
<td align="center" width="33%">
<h3>💓 Heartbeats</h3>
Agents wake for assigned work, follow-up messages, or configured schedules. Delegation flows up and down the org chart.
</td>
</tr>
<tr>
<td align="center">
<h3>💰 Cost Control</h3>
Company, agent, and project <a href="https://docs.paperclip.ing/guides/day-to-day/costs/">budgets</a>. Track reported spend, get threshold alerts, and pause work at configured limits.
</td>
<td align="center">
<h3>🏢 Multi-Organization</h3>
One deployment, many organizations. Separate tasks, agents, permissions, and activity histories for each.
</td>
<td align="center">
<h3>🎫 Task Threads</h3>
Keep conversations, plans, blockers, files, and run history attached to the work. Assign tasks to agents or people.
</td>
</tr>
<tr>
<td align="center">
<h3>🛡️ Governance</h3>
Configure <a href="https://docs.paperclip.ing/guides/day-to-day/approvals/">review and approval stages</a>, approve hires, and pause, reassign, or stop work when needed.
</td>
<td align="center">
<h3>📊 Org Chart</h3>
Hierarchies, roles, reporting lines. Your agents have a boss, a title, and a job description.
</td>
<td align="center">
<h3>📱 Mobile Ready</h3>
Monitor and manage your autonomous businesses from anywhere.
</td>
</tr>
<tr>
<td align="center">
<h3>🔗 Apps & Connections</h3>
<a href="https://docs.paperclip.ing/connectors/">Connect services</a> such as GitHub, Notion, and Railway, or your own MCP server. Set gateway actions to Allowed, Ask first, or Off.
</td>
<td align="center">
<h3>👥 Shared Agents, Personal Accounts</h3>
Choose <a href="https://docs.paperclip.ing/connectors/access-model/">who can use a connection and which agents can access it</a>. Managed GitHub operations can use the account of the person directing the work.
</td>
<td align="center">
<h3>🧠 Skills & Skill Studio</h3>
Install or write <a href="https://docs.paperclip.ing/guides/org/skills/">shared skills</a>, test them with saved inputs, inspect results, and restore earlier versions.
</td>
</tr>
<tr>
<td align="center">
<h3>📅 Scheduled Routines</h3>
Run <a href="https://docs.paperclip.ing/guides/projects-workflow/routines/">recurring work</a> on a schedule or trigger it through an API or webhook. Each run has a task, an owner, and a history.
</td>
<td align="center">
<h3>📎 Artifacts & Feedback</h3>
Find the <a href="https://docs.paperclip.ing/guides/day-to-day/artifacts/">files and documents agents produce</a>. Preview supported formats and leave comments on specific passages in documents.
</td>
<td align="center">
<h3>📦 Ready-Made Teams</h3>
Preview and install <a href="https://docs.paperclip.ing/guides/org/team-catalog/">teams</a> with roles, skills, projects, and routines. Choose their runtimes and make the setup your own.
</td>
</tr>
</table>

Experimental **Agent Chat** and **chat/email connectors** add conversations with agents in Paperclip and through configured services such as Slack, Discord, Telegram, and AgentMail. Enable the relevant instance settings to try them.

<br/>

## Problems Paperclip solves

| Without Paperclip                                                                                                                     | With Paperclip                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| ❌ You have 20 Claude Code tabs open and can't track which one does what. On reboot you lose everything.                              | ✅ Tasks are ticket-based, conversations are threaded, sessions persist across reboots.                                                |
| ❌ You manually gather context from several places to remind your bot what you're actually doing.                                     | ✅ Context flows from the task up through the project and company goals — your agent always knows what to do and why.                  |
| ❌ Folders of agent configs are disorganized and you're re-inventing task management, communication, and coordination between agents. | ✅ Paperclip gives you org charts, ticketing, delegation, and governance out of the box — so you run a company, not a pile of scripts. |
| ❌ Runaway loops waste hundreds of dollars of tokens and max your quota before you even know what happened.                           | ✅ Spend tracking, budget alerts, and automatic pauses help you control the cost of ongoing work.                                       |
| ❌ You have recurring jobs (customer support, social, reports) and have to remember to manually kick them off.                        | ✅ Routines create assigned tasks on a schedule, with outputs and run history you can inspect.                                         |
| ❌ You have an idea, you have to find your repo, fire up Claude Code, keep a tab open, and babysit it.                                | ✅ Add a task in Paperclip. Your coding agent works on it until it's done. Management reviews their work.                              |

<br/>

## Why Paperclip is special

Paperclip handles the hard orchestration details correctly.

|                                   |                                                                                                               |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Atomic task checkout.**         | A single assignee and execution locks prevent competing runs from claiming the same task.                    |
| **Persistent work context.**      | Tasks, comments, and documents stay in Paperclip. Supporting adapters resume saved sessions across runs.       |
| **Runtime skill injection.**      | Agents can learn Paperclip workflows and project context at runtime, without retraining.                      |
| **Governance with rollback.**     | Approval gates are enforced, config changes are revisioned, and bad changes can be rolled back safely.        |
| **Accountable connections.**      | Human access, agent eligibility, and gateway action permissions are separate controls. Approve a call once or save a revocable rule. |
| **Goal-aware execution.**         | Linked tasks and projects carry goal ancestry so agents see the "why," not just a title.                      |
| **Portable company templates.**   | Export/import orgs, agents, and skills with secret scrubbing and collision handling.                          |
| **Organization boundaries.**      | Company-scoped access checks keep each organization's work, agents, and activity separate within one deployment. |

<br/>

## What's Under the Hood

Paperclip is a full control plane, not a wrapper. Before you build any of this yourself, know that it already exists:

```
┌──────────────────────────────────────────────────────────────┐
│                       PAPERCLIP SERVER                       │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │Identity & │  │  Work &   │  │ Heartbeat │  │Governance │  │
│  │  Access   │  │   Tasks   │  │ Execution │  │& Approvals│  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Org Chart │  │Workspaces │  │  Plugins  │  │  Budget   │  │
│  │ & Agents  │  │ & Runtime │  │           │  │ & Costs   │  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
│                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Routines  │  │ Secrets & │  │ Activity  │  │  Company  │  │
│  │& Schedules│  │  Storage  │  │ & Events  │  │Portability│  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
└──────────────────────────────────────────────────────────────┘
         ▲              ▲              ▲              ▲
   ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐
   │  Claude   │  │   Codex   │  │   CLI     │  │ HTTP/web  │
   │   Code    │  │           │  │  agents   │  │   bots    │
   └───────────┘  └───────────┘  └───────────┘  └───────────┘
```

### The Systems

<table>
<tr>
<td width="50%">

**Identity & Access** — Two deployment modes (trusted local or authenticated), human roles and permissions, agent API keys, short-lived run JWTs, company memberships, and invite flows. Responsible-user attribution follows work through delegation and supported managed connections.

</td>
<td width="50%">

**Org Chart & Agents** — Agents have roles, titles, reporting lines, permissions, and budgets. Adapter examples match the diagram: Claude Code, Codex, CLI agents such as Cursor/Gemini/bash, HTTP/webhook bots such as OpenClaw, and external adapter plugins. If it can receive a heartbeat, it's hired.

</td>
</tr>
<tr>
<td>

**Work & Task System** — Issues carry company/project/goal/parent links, atomic checkout with execution locks, first-class blocker dependencies, comments, documents, attachments, work products, labels, and inbox state. Search across work, review document revisions, and leave anchored feedback.

</td>
<td>

**Heartbeat Execution** — DB-backed wakeup queue with coalescing, budget checks, workspace resolution, secret injection, skill loading, and adapter invocation. Runs produce logs, usage records, and adapter-specific session state. Bounded recovery handles supported failures and surfaces cases that need human action.

</td>
</tr>
<tr>
<td>

**Workspaces & Runtime** — Project repositories and workspaces, optional isolated execution workspaces (git worktrees, operator branches), and runtime services (dev servers, preview URLs). Sandbox providers extend execution beyond the local host; availability depends on the configured environment and adapter.

</td>
<td>

**Governance & Approvals** — Board approval workflows, execution policies with review/approval stages, decision tracking, budget hard-stops, and agent pause/resume/terminate. Configured task reviews govern completion; connection action approvals govern calls through the tool gateway.

</td>
</tr>
<tr>
<td>

**Budget & Cost Control** — Token and cost tracking by company, agent, project, goal, issue, provider, and model. Scoped budget policies with warning thresholds and hard stops. Enforcement uses recorded spend; usage reporting and in-flight work can delay a stop.

</td>
<td>

**Routines & Schedules** — Recurring tasks with cron, webhook, and API triggers. Concurrency and catch-up policies. Each routine execution creates a tracked issue and wakes the assigned agent — no manual kick-offs needed.

</td>
</tr>
<tr>
<td>

**[Plugins](https://docs.paperclip.ing/administration/plugins/)** — Instance-wide plugin system with out-of-process workers, capability-gated host services, job scheduling, tool exposure, and UI contributions. Extend Paperclip without forking it.

</td>
<td>

**Secrets & Storage** — Company secrets and per-person secret values, encrypted credential storage, local or S3-compatible file storage, attachments, and work products. Secret references supply credentials to authorized runs without copying values into ordinary agent configuration.

</td>
</tr>
<tr>
<td>

**Activity & Events** — Mutating actions, heartbeat state changes, cost events, approvals, comments, and work products are recorded as durable activity so operators can audit what happened and why.

</td>
<td>

**[Company Portability](https://docs.paperclip.ing/guides/power/export-import/)** — Preview, export, and import organization packages with agents, skills, and optional projects, routines, tasks, and attachments. Referenced secret values are omitted; review packages before sharing because plain environment values and local paths can remain. Packages share an operating setup; full-instance recovery uses backups.

</td>
</tr>
</table>

<br/>

## What Paperclip is not

|                              |                                                                                                                      |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Not just a chatbot.**      | Conversations stay attached to tasks, plans, decisions, and outputs. Experimental Agent Chat can hand work off to assigned tasks.    |
| **Not an agent framework.**  | We don't tell you how to build agents. We tell you how to run a company made of them.                                |
| **Not just a workflow builder.** | Routines operate within an organization, with roles, goals, budgets, and governance.                  |
| **Not a prompt manager.**    | Agents bring their own prompts, models, and runtimes. Paperclip manages the organization they work in.               |
| **Not limited to one agent.** | Start with one agent and grow into a team with shared skills, delegation, and review.                                              |
| **Not only for code review.** | Coding and PR review fit alongside research, operations, content, and other work.                                                  |

<br/>

## Quickstart

Open source. Self-hosted. No Paperclip account required. Follow the [guided quickstart](https://docs.paperclip.ing/guides/getting-started/five-minute-path/) to set up your first agent.

### Just ask your agent to install Paperclip

Share the [installation guide](https://docs.paperclip.ing/reference/cli/installation/) with your agent.

### Or install it yourself

With **Node.js 24.11 or newer** installed:

```bash
npx paperclipai@latest onboard --yes
```

The CLI runs from npm's cache; your instance configuration and data persist locally.

See the [installation guide](https://docs.paperclip.ing/reference/cli/installation/) for managed installs, pinned versions, canary and git-ref installs, updates, rollback, service management, and uninstalling.

For an isolated manual test instance that is already initialized with a CEO
agent, use `test-drive`. It stays in the foreground, never installs a service
or creates a first task, and opens the browser only after setup succeeds:

```bash
ANTHROPIC_API_KEY=... npx paperclipai test-drive
OPENAI_API_KEY=... npx paperclipai test-drive --harness codex
OPENROUTER_API_KEY=... npx paperclipai test-drive \
  --harness opencode \
  --model openrouter/anthropic/claude-sonnet-4.5
```

Each run without `--data-dir` gets a unique, retained temporary directory; its
absolute path is printed at startup. Pass `--data-dir` to reuse one, or
`--no-browser` to leave the initialized instance unopened. When invoked from a
linked Git worktree, `test-drive` also enables task execution in that worktree.
See the [test-drive guide](https://docs.paperclip.ing/reference/cli/test-drive/) for credential and
reuse behavior.

> **Troubleshooting: private npm registry `.npmrc`**
>
> If this fails with an `E404` for `paperclipai` (or similar) and you use a private npm registry (for example GitHub Packages) via a global `~/.npmrc`, `npx` may be resolving `paperclipai` against that private registry instead of the public npm registry.
>
> Diagnostic:
>
> ```bash
> npm config get registry
> ```
>
> Workaround (cross-platform; force the public npm registry for this command):
>
> ```bash
> npx --registry https://registry.npmjs.org paperclipai@latest onboard --yes
> ```

That quickstart path now defaults to trusted local loopback mode for the fastest first run. To start in authenticated/private mode instead, choose a bind preset explicitly:

```bash
npx paperclipai@latest onboard --yes --bind lan
# or:
npx paperclipai@latest onboard --yes --bind tailnet
```

If you already have Paperclip configured, rerunning `onboard` keeps the existing config in place. Use `npx paperclipai configure` to edit settings.

Or manually:

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip
pnpm install
pnpm dev
```

This starts the UI and API at `http://localhost:3100`. An embedded PostgreSQL database is created automatically — no setup required.

> **Requirements:** Node.js 24.11+, pnpm 9.15+

Local Claude and Codex subscription sign-in also needs Python 3 and the
corresponding provider CLI on the Paperclip host. The Docker image includes them.

Source development also builds the native Paperclip Runner when enabled (the self-hosted default). Install a Rust toolchain, or set `PAPERCLIP_RUNNER_BINARY` to a compatible prebuilt runner.

<br/>

## FAQ

**Q: Is this project maintained or just slop?**

**A:** Paperclip is maintained by the [Paperclip team](https://paperclip.ing). We've merged [over 2,700 pull requests](https://github.com/paperclipai/paperclip/pulls?q=is%3Apr+is%3Amerged).

<br/>

**Q: What does a typical setup look like?**

**A:** Locally, a single Node.js process manages an embedded Postgres and local file storage. For production, point it at your own Postgres and deploy however you like. Configure projects, agents, and goals — the agents take care of the rest.

For remote access, use authenticated mode with a private-network bind such as Tailscale, or deploy the persistent server with Docker. See [deployment modes](https://docs.paperclip.ing/reference/deploy/deployment-modes/) and the [Docker guide](https://docs.paperclip.ing/reference/deploy/docker/).

<br/>

**Q: Can I run multiple companies?**

**A:** Yes. A single deployment can host multiple organizations with company-scoped data and access checks.

<br/>

**Q: How is Paperclip different from agents like OpenClaw or Claude Code?**

**A:** Paperclip _uses_ those agents. It orchestrates them into a company — with org charts, budgets, goals, governance, and accountability.

<br/>

**Q: Why should I use Paperclip instead of just pointing my OpenClaw to Asana or Trello?**

**A:** Agent orchestration has subtleties in how you coordinate who has work checked out, how to maintain sessions, monitoring costs, establishing governance - Paperclip does this for you.

(Bring-your-own-ticket-system is on the Roadmap)

<br/>

**Q: Do agents run continuously?**

**A:** Agents wake for assigned work and follow-up messages. Optional timer heartbeats let them check for work periodically; routines create recurring tasks on their own schedules. You can also connect externally running agents such as OpenClaw. A mention alone does not assign work or wake another agent.

<br/>

## Development

```bash
pnpm dev              # Full dev (API + UI, watch mode)
pnpm dev:once         # Full dev without file watching
pnpm dev:server       # Server only
pnpm dev:mobile       # Serve prebuilt UI on :3101 for phones/tablets (proxies /api → :3100)
pnpm dev:both         # Run `pnpm dev` and `pnpm dev:mobile` together
pnpm build            # Build all
pnpm typecheck        # Type checking
pnpm test             # Cheap default test run (Vitest only)
pnpm test:watch       # Vitest watch mode
pnpm test:e2e         # Playwright browser suite
pnpm db:generate      # Generate DB migration
pnpm db:migrate       # Apply migrations
```

`pnpm test` does not run Playwright. Browser suites stay separate and are typically run only when working on those flows or in CI.

See [doc/DEVELOPING.md](doc/DEVELOPING.md) for the full development guide.

<br/>

## Roadmap

- ✅ Plugin system (e.g. add a knowledge base, custom tracing, queues, etc)
- ✅ Get OpenClaw / claw-style agent employees
- ✅ companies.sh - import and export entire organizations
- ✅ Easy AGENTS.md configurations
- ✅ Skills Manager, Skill Studio & Skills Store
- ✅ Scheduled Routines
- ✅ Better Budgeting
- ✅ Agent Reviews and Approvals
- ✅ Multiple Human Users
- ✅ Cloud / Sandbox agents (e2b, Cloudflare, Daytona, Modal, Novita, boxd, self-hosted Kubernetes)
- ✅ Artifacts & Work Products
- ✅ Deep Planning (planning mode, revisioned plans, plan approvals)
- ✅ Enforced Outcomes (watchdogs, recovery actions, review gates)
- ✅ MCP Tool Gateway & Apps (governed tool access)
- ✅ Secrets Manager with per-agent access
- ✅ Activity log & action attribution
- ✅ Self-healing runs & automatic recovery
- ✅ Agent evals & feedback
- ✅ Connected Apps
- ✅ Personal & Shared AI Accounts
- ✅ Shared Agents Use Personal GitHub Identities
- ✅ Skill Version History & Restore
- ✅ Document Comments & Revision History
- ✅ Company-Wide Search
- ✅ Multi-Model & Multi-Harness Teams
- 🟡 Memory / Knowledge
- ⚪ MAXIMIZER MODE
- ⚪ Work Queues
- ⚪ Self-Organization
- ⚪ Automatic Organizational Learning
- 🟡 Agent Chat
- 🟡 Cloud deployments
- ⚪ Desktop App
- ⚪ Bring-your-own-ticket-system (Asana / Linear / Jira as on-ramps)

This is the short roadmap preview. See the full roadmap in [ROADMAP.md](ROADMAP.md).

<br/>

## Community & Plugins

Find Plugins and more at [awesome-paperclip](https://github.com/gsxdsm/awesome-paperclip)

## Observability

Paperclip ships with opt-in OpenTelemetry auto-instrumentation for the server (traces only). It activates when `OTEL_EXPORTER_OTLP_ENDPOINT` is set and supports `grpc`, `http/protobuf`, and `http/json` via the standard `OTEL_EXPORTER_OTLP_PROTOCOL` env var. `@opentelemetry/api` is a normal server dependency; the SDK, auto-instrumentation, and exporter packages are optional peer dependencies — install them only if you want tracing. See [doc/observability.md](doc/observability.md) for install commands and the full env-var reference.

Paperclip also ships with opt-in Sentry error monitoring for the server and the browser. Set `SENTRY_DSN_FRONTEND` to activate it for the browser and `SENTRY_DSN_BACKEND` to activate it for the server — each variable is optional, and the legacy `SENTRY_DSN` variable still works as a fallback for either component. The supported server SDK version is `@sentry/node@10.71.0`; it is an optional peer dependency for the server, so install it only if you want error monitoring. The browser SDK, `@sentry/browser`, is pinned to the same exact version. See [doc/observability.md](doc/observability.md#sentry-error-monitoring) for the install command, the privacy settings, and the full default capture set.

## Telemetry

Paperclip collects anonymous usage telemetry to help us understand how the product is used and improve it. No personal information, issue content, prompts, file paths, or secrets are ever collected. Private repository references are hashed with a per-install salt before being sent.

Contributors changing emitted telemetry events should follow the [Telemetry Data Contract](packages/shared/src/telemetry/README.md).
For proposed first-party events that are not in the generated contract yet, follow [Telemetry Workflow](doc/TELEMETRY_WORKFLOW.md).

Telemetry is **enabled by default** and can be disabled with any of the following:

| Method               | How                                                     |
| -------------------- | ------------------------------------------------------- |
| Environment variable | `PAPERCLIP_TELEMETRY_DISABLED=1`                        |
| Standard convention  | `DO_NOT_TRACK=1`                                        |
| CI environments      | Automatically disabled when `CI=true`                   |
| Config file          | Set `telemetry.enabled: false` in your Paperclip config |

## Contributing

We welcome contributions. See the [contributing guide](CONTRIBUTING.md) for details.

**[We're hiring](https://paperclip.ing/about/#careers)**

<br/>

## Community

- [Discord](https://discord.gg/paperclipai) — Join the community
- [Twitter / X](https://x.com/papercliping) — Follow updates and announcements
- [GitHub Issues](https://github.com/paperclipai/paperclip/issues) — bugs and feature requests
- [GitHub Discussions](https://github.com/paperclipai/paperclip/discussions) — ideas and RFC

<br/>

## License

MIT &copy; 2026 [Paperclip Labs, Inc](https://paperclip.ing)

## Star History

<a href="https://www.star-history.com/?repos=paperclipai%2Fpaperclip&type=date&legend=top-left">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=paperclipai/paperclip&type=date&theme=dark&legend=top-left&sealed_token=hFjuwFq41bQD5cevvXVv5cTru2swWRZujwJYKlHhtBh6n0H5-VvJZW2SAlcQKB8u4KxhyEB9JqFg1yccJ8WLv9wPBcoWpWcak4gx0MYTWu_pOs2jKOaDluH7KsLeTKt6DHGkHiN3LsqV9s--MTDQcC6Xl7zV51W0-YezQXo-pVPgoFDFAGf2CY5fiP5Q" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=paperclipai/paperclip&type=date&legend=top-left&sealed_token=hFjuwFq41bQD5cevvXVv5cTru2swWRZujwJYKlHhtBh6n0H5-VvJZW2SAlcQKB8u4KxhyEB9JqFg1yccJ8WLv9wPBcoWpWcak4gx0MYTWu_pOs2jKOaDluH7KsLeTKt6DHGkHiN3LsqV9s--MTDQcC6Xl7zV51W0-YezQXo-pVPgoFDFAGf2CY5fiP5Q" />
    <img src="https://api.star-history.com/chart?repos=paperclipai/paperclip&type=date&legend=top-left&sealed_token=hFjuwFq41bQD5cevvXVv5cTru2swWRZujwJYKlHhtBh6n0H5-VvJZW2SAlcQKB8u4KxhyEB9JqFg1yccJ8WLv9wPBcoWpWcak4gx0MYTWu_pOs2jKOaDluH7KsLeTKt6DHGkHiN3LsqV9s--MTDQcC6Xl7zV51W0-YezQXo-pVPgoFDFAGf2CY5fiP5Q" alt="Star History Chart" />
  </picture>
</a>

<br/>

---

<p align="center">
  <sub>Open source under MIT. Built for people who want to get work done, not babysit agents.</sub>
</p>
