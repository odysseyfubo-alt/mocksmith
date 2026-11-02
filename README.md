<div align="center">

**MockSmith** —— AI 模拟面试与私有知识库问答平台

<sub>Skill-driven AI mock interview · RAG knowledge base · Java 25 + Spring AI + pgvector</sub>

[![Java](https://img.shields.io/badge/Java-25-orange?logo=openjdk)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1-green?logo=springboot)](https://spring.io/projects/spring-boot)
[![Spring AI](https://img.shields.io/badge/Spring%20AI-2.0-brightgreen?logo=spring)](https://spring.io/projects/spring-ai)
[![React](https://img.shields.io/badge/React-18.3-blue?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-blue?logo=typescript)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-pgvector-336791?logo=postgresql)](https://www.postgresql.org/)

</div>

---

## 项目介绍

MockSmith 是一个集**简历分析、模拟面试（文字 / 语音）、面试日程管理、RAG 知识库问答、题库面试、多模型配置**于一体的 AI 面试训练平台。

系统基于大语言模型（LLM）、pgvector 向量检索与 Redis Stream 异步任务编排，为求职者提供「简历解析 → AI 出题 → 多轮追问 → 评分报告」的全链路练习能力，同时支持基于私有知识库的检索增强问答。

核心设计围绕两个问题展开：

- **LLM 输出不可控**：用结构化输出 + JSON Schema 校验 + 修复型重试，把自由文本收敛成可入库的数据结构；
- **AI 调用长耗时**：用 Redis Stream 消费组把简历分析、向量化、出题、评估等耗时任务异步化，并配套重试、补偿与执行代次防覆盖。

## 功能特性

### 简历管理

- 多格式解析：PDF / DOCX / DOC / TXT / Markdown，基于 Apache Tika 抽取正文。
- 异步分析：基于 Redis Stream 异步执行，前端可查看「待分析 / 分析中 / 已完成 / 失败」进度。
- 稳定性：失败自动重试（最多 3 次）+ 内容哈希去重 + 卡死任务定时补偿。
- 报告导出：AI 分析结果一键导出结构化 PDF 报告。

### 模拟面试（文字）

- **Skill 驱动出题**：11 个面试方向（Java 后端、阿里/字节/腾讯专项、前端、Python、算法、系统设计、测开、AI Agent 等），每个方向由 `SKILL.md` 定义考察范围、难度分布与参考知识库。
- **历史题目去重**：出题时排除历史会话已问过的题目。
- **智能追问**：每个主问题可生成配置数量的追问（默认 1 条），模拟多轮问答。
- **统一评估引擎**：文字与语音共用「分批评估 + 结构化输出 + 二次汇总 + 降级兜底」的评估流水线；报告支持异步生成与 PDF 导出。

### 面试安排

- 邀请解析：规则 + AI 双引擎，支持飞书 / 腾讯会议 / Zoom 邀请文本，自动提取公司、岗位、时间、会议链接。
- 日历管理：日 / 周 / 月视图 + 拖拽调整 + 列表视图。
- 状态流转与提醒：定时任务自动过期，支持手动标记与面试提醒。

### 知识库（RAG）

- 文档上传 → 解析 → 分块（默认 800 字符 / 最小 350）→ 异步向量化 → 写入 pgvector（1024 维、HNSW + COSINE）。
- 检索侧支持按知识库范围前置过滤、查询改写与双路召回分数融合。
- 问答采用 SSE 流式返回，支持会话管理、多知识库关联与来源标注。
- 运维能力：分类管理、下载、重新向量化、搜索与统计。

### 题库与题库面试

- 从已向量化文档异步生成主问题、参考答案、关键点、评分标准与追问。
- 题库维护：搜索、筛选、分页、增删改、草稿 / 启用 / 归档状态管理。
- 开始面试前按方向、难度、主问题数与追问数做容量校验；交卷后异步生成逐题评价与总分。

### 多模型与系统设置

- **OpenAI 兼容 Provider 抽象**：一套配置支持任意 OpenAI 兼容供应商，配置落库、运行时热切换（无需重启、无需改代码）。
- **默认模型切换**：设置页可切换默认对话模型与默认向量模型。
- **配置安全**：API Key 使用 AES-GCM 加密存储，运行时配置写入用户目录 `~/.interview-guide/`。

### 语音面试（可选，需要 DashScope）

WebSocket 承载音频流，接入实时 ASR/TTS 实现「边说边转写、分句流式合成」，并用「摘要 + 滑动窗口 + 字符硬预算」压缩多轮上下文以控制 token 成本与首字延迟。

> 语音能力依赖阿里云 DashScope 实时语音模型。未配置 `AI_BAILIAN_API_KEY` 时应用仍可正常启动，仅语音面试不可用（启动日志会给出 WARN）。

## 技术栈

### 后端

| 技术 | 版本 | 说明 |
| --- | --- | --- |
| Java | 25 | 开发语言（虚拟线程） |
| Spring Boot | 4.1.0 | 应用框架 |
| Spring AI | 2.0.0 | LLM 接入、结构化输出、Tool Calling |
| Spring AI Agent Utils | 0.10.0 | Skill 资源加载、Advisor 扩展 |
| PostgreSQL + pgvector | 16 | 关系数据库 + 向量存储 |
| Redis + Redisson | 7 / 4.0.0 | 会话缓存、Stream 异步任务、分布式锁与限流 |
| Apache Tika | 2.9.2 | 文档解析 |
| iText 8 | 8.0.5 | PDF 报告导出 |
| MapStruct | 1.6.3 | 对象映射 |
| SpringDoc OpenAPI | 3.0.2 | 接口文档 |
| AWS S3 SDK | 2.29.51 | S3 兼容对象存储（MinIO / RustFS） |
| DashScope SDK | 2.22.7 | 语音识别 / 合成（可选） |
| Gradle | 9.6.1 | 构建工具 |

### 前端

| 技术 | 版本 | 说明 |
| --- | --- | --- |
| React | 18.3 | UI 框架 |
| TypeScript | 5.6 | 开发语言 |
| Vite | 5.4 | 构建与开发服务器 |
| Tailwind CSS | 4.1 | 样式框架 |
| React Router | 7.11 | 路由 |
| Framer Motion | 12.23 | 动画 |
| Recharts | 3.6 | 图表 |
| React Virtuoso | 4.18 | 长列表虚拟滚动 |
| pnpm | 10.x | 包管理器 |

## 系统架构

![系统架构图](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/interview-guide-architecture-diagram.png)

### 模块与异步任务流

```
┌──────────────────────────── 前端 (React 18 + Vite, :5173) ────────────────────────────┐
│  简历 / 模拟面试 / 面试日历 / 知识库问答 / 题库面试 / 设置页                            │
└───────────────────────────────────┬──────────────────────────────────────────────────┘
                                    │ REST + SSE + WebSocket（Vite 代理 → :8080）
┌───────────────────────────────────▼──────────────────────────────────────────────────┐
│  Spring Boot 4.1 (:8080)                                                              │
│  ├── modules/interview            面试会话、出题、作答、评估、报告导出                  │
│  ├── modules/knowledgebase        知识库上传、向量化、RAG 问答、题库生成                │
│  ├── modules/resume               简历上传、解析、AI 分析                              │
│  ├── modules/interviewschedule    面试邀请解析与日历                                   │
│  ├── modules/voiceinterview       语音面试（WebSocket + ASR/TTS）                      │
│  ├── modules/llmprovider          Provider 配置管理与热切换                            │
│  └── common/ai                    ChatClient 注册表、结构化输出、Prompt 模板            │
│  └── common/async                 Redis Stream 生产者/消费者/重试/恢复模板              │
└───────┬─────────────────────┬───────────────────────┬────────────────────────────────┘
        │                     │                       │
   PostgreSQL+pgvector    Redis Stream           S3（MinIO / RustFS）
   业务数据 + 向量检索     异步任务与缓存          简历 / 知识库原文件
```

**异步任务流**（统一走 `AbstractStreamProducer` / `AbstractStreamConsumer`）：

| Stream | 用途 | 失败策略 |
| --- | --- | --- |
| `resume:analyze:stream` | 简历 AI 分析 | 重试 3 次 → FAILED，定时补偿 |
| `knowledgebase:vectorize:stream` | 文档分块向量化 | 重试 3 次 + 执行代次 fencing |
| `knowledgebase:question-gen:stream` | 题库题目生成 | 重试 3 次，保留草稿 |
| `interview:evaluate:stream` | 文字面试评估 | 重试 3 次，降级兜底 |
| `voice:evaluate:stream` | 语音面试评估 | 重试 3 次，降级兜底 |

## 效果展示

### 简历与面试

面试中心：

![面试中心](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-hub.png)

Skill 出题 + JD 解析：

![Skill 出题 + JD 解析](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-skill-jd-parse.png)

简历库：

![简历库](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-history.png)

简历上传分析：

![简历上传分析](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-upload-analysis.png)

简历分析详情：

![简历分析详情](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-analysis-detail.png)

面试记录：

![面试记录](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-history.png)

面试详情：

![面试详情](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-detail.png)

模拟面试：

![模拟面试](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-mock-interview.png)

面试安排：

![面试安排](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-schedule-list.png)

多模型切换 + 语音服务设置：

![管理聊天模型、向量模型和模块配置](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/llm-settings.png)

### 知识库

知识库管理：

![知识库管理](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-knowledge-base-management.png)

问答助手：

![问答助手](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-qa-assistant.png)

## 项目结构

```
mocksmith/
├── app/                                  # Spring Boot 后端
│   ├── src/main/java/interview/guide/
│   │   ├── common/                       # 限流、AI 调用、异步模板、配置、异常、统一响应
│   │   ├── infrastructure/               # 文件、导出、Redis、MapStruct 映射
│   │   └── modules/                      # 业务模块（各自 MVC 分层）
│   ├── src/main/resources/
│   │   ├── application.yml               # 主配置（含 Provider 预设）
│   │   ├── prompts/                      # StringTemplate(.st) Prompt 模板
│   │   ├── skills/                       # SKILL.md 出题知识（11 个方向）
│   │   └── db/migration/                 # Flyway 迁移脚本
│   └── src/test/resources/               # rag-eval / voice-context-eval 评测集与基线
├── frontend/                             # React + TypeScript 前端
│   └── src/{api,components,pages,hooks,types,utils}
├── docker/postgres/init.sql              # 数据库初始化
├── docker-compose.yml                    # 全栈编排（PG + Redis + MinIO + 前后端）
├── docker-compose.dev.yml                # 仅依赖服务
└── .env.example                          # 环境变量模板
```

## 快速开始

### 环境要求

| 依赖 | 版本 |
| --- | --- |
| JDK | 25（Gradle toolchain 自动拉取） |
| Node.js | 20+ |
| pnpm | 10.x |
| Docker | 用于 PostgreSQL / Redis / MinIO |

### 1. 配置环境变量

```bash
cp .env.example .env
```

最少需要填三项：

```dotenv
# Provider API Key 加密密钥（必填，部署后保持不变，改了会导致已存密钥无法解密）
APP_AI_CONFIG_ENCRYPTION_KEY=<随机长字符串>

# 对话模型
PROVIDER_DEEPSEEK_API_KEY=sk-xxx
PROVIDER_DEEPSEEK_MODEL=deepseek-flash

# 向量模型（DeepSeek 官方无 Embedding 接口，这里用硅基流动）
PROVIDER_SILICONFLOW_API_KEY=sk-xxx
PROVIDER_SILICONFLOW_EMBEDDING_MODEL=Qwen/Qwen3-Embedding-0.6B
```

`.env` 已被 `.gitignore` 忽略，不要把真实密钥提交到仓库。

### 2. 启动依赖服务

```bash
docker compose -f docker-compose.yml up -d postgres redis minio
```

- PostgreSQL：`localhost:5432`（库 `interview_guide`）
- Redis：`localhost:6379`（本机端口被占用时在 `.env` 里设 `REDIS_HOST_PORT=6380` 与 `REDIS_PORT=6380`）
- MinIO：`localhost:9000`（控制台 `:9001`，默认 `minioadmin/minioadmin`）

> 只用依赖服务时不要整份跑 `docker-compose.dev.yml`：它的 `rustfs` 需要额外拉镜像，且凭证是 `rustfsadmin`，与根 compose 的 MinIO 不是同一套。

### 3. 启动后端

```bash
./gradlew :app:bootRun
```

首次启动会自动执行 Flyway 迁移、把 `app.ai.providers` 里的 Provider 灌入数据库，并自动创建对象存储桶。

### 4. 启动前端

```bash
cd frontend && pnpm install && pnpm run dev
```

访问 <http://localhost:5173>（Vite 已把 `/api` 代理到 `:8080`）。

## 模型与向量配置

### Provider 机制

- Provider 列表**以数据库为准**：首次启动由 `app.ai.providers` 写入 `llm_provider_config`，之后只在设置页维护。
- 设置页支持新增 / 编辑 / 删除 Provider，切换默认对话模型与默认向量模型，切换后立即生效（Registry 清缓存重建，无需重启）。
- 修改 `.env` 只对**空数据库的首次启动**生效；已有数据库请在设置页改，或直接改数据库记录。

### 向量模型选择注意

应用在调用 Embedding 接口时会携带 `dimensions` 参数，因此要选支持该参数的模型：

| 供应商 / 模型 | 是否可用 | 说明 |
| --- | --- | --- |
| 硅基流动 `Qwen/Qwen3-Embedding-0.6B` | ✅ | 默认配置，1024 维（推荐，速度快成本低） |
| 硅基流动 `Qwen/Qwen3-Embedding-8B` | ✅ | 1024 维，效果更好、更慢更贵 |
| 硅基流动 `BAAI/bge-m3` | ❌ | 不识别 `dimensions`，返回 400（`code 20015`） |
| DashScope `text-embedding-v3` | ✅ | 1024 维 |

pgvector 表按 **1024 维**建，换向量模型时必须保证维度一致；换模型后已有知识库需要重新向量化（不同模型的向量空间不可混用）。

## 开发与测试

```bash
# 后端编译 / 测试
./gradlew :app:compileJava
./gradlew :app:test --no-daemon

# 前端
cd frontend && pnpm run build
```

仓库内还包含两个离线性评测资产（用于对比 Prompt / 模型变更前后的效果）：

- `app/src/test/resources/rag-eval/`：RAG 检索评测集（40 条）与基线报告
- `app/src/test/resources/voice-context-eval/`：语音上下文压缩对照与基线

## 常见问题

### 知识库上传后「向量化失败」

九成是 Embedding 调用失败，而不是文档解析失败：

1. 检查默认向量 Provider 的 API Key 是否已写入数据库（设置页可见 `sk-***` 掩码；`.env` 改了不会同步到已初始化的库）；
2. 检查向量模型是否支持 `dimensions` 参数（见上一节表格）；
3. 详细异常在应用日志里，数据库 `knowledge_bases.vector_error` 只有兜底文案。

### 启动报 `Port 8080 was already in use`

上一次的进程还在：

```powershell
Get-NetTCPConnection -LocalPort 8080 -State Listen | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
```

### 启动报 `检查存储桶失败: Connection refused` / `Forbidden`

- `Connection refused`：对象存储没起，执行步骤 2；
- `Forbidden`：凭证与容器不一致。根 compose 的 MinIO 是 `minioadmin/minioadmin`，`docker-compose.dev.yml` 的 RustFS 是 `rustfsadmin/rustfsadmin`。

### 日志刷 `XAUTOCLAIM` 报错

Redis 版本低于 6.2（Windows 原生 Redis 3.x 常见）。用容器里的 Redis 7，并确认应用连的是它（改 `.env` 的 `REDIS_HOST_PORT` / `REDIS_PORT`）。

### 设置页切换模型后不生效

Provider 配置缓存在 Registry 中，保存时会自动 reload；若是直接改了数据库，需要重启应用或调用 `POST /api/llm-provider/reload`。

### 语音面试连不上 / 没人说话

语音依赖 DashScope 实时语音（`AI_BAILIAN_API_KEY`）。未配置时语音面试不可用，但文字面试与知识库功能不受影响。

### Windows 下日志中文乱码

`app/src/main/resources/logback-spring.xml` 已固定 UTF-8，乱码一般是终端编码问题，可先执行 `chcp 65001`。

## 已知限制

- 语音面试端到端延迟偏高（服务端音频中转），无耳机时可能回声泄漏。
- 仅支持单用户（`userId` 固定），未做多租户与权限体系。
- 向量维度固定 1024，更换 Embedding 模型需重新向量化。
- 异步任务的补偿调度是单实例内的定时扫描，多实例部署需要额外加分布式协调。

## 许可证

本项目采用 [AGPL-3.0](LICENSE) 许可证，详见 `LICENSE` 文件。
