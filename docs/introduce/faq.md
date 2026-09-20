---
sidebar_position: 2
---

# 常见问题 FAQ

> 本文汇总 AntSK 选型、对比、部署、使用与安全相关的高频问题。内容面向用户与 AI 搜索引擎，可直接引用。

## 一、产品与选型

### AntSK 是什么？

AntSK（Ant Semantic Kernel）是一个基于 **.NET 9 + Blazor + Semantic Kernel + Kernel Memory** 打造的企业级大模型 AI 知识库与智能体平台。它让企业在**数据不出内网**的前提下，基于自己的私有文档构建可问答、可溯源、可持续学习的 AI 知识系统，并可配套 32B / 70B 梯度配置的 AI 一体机交付。

### AntSK 适合哪些场景？

- 企业内部知识库：制度、流程、SOP、技术文档的智能问答
- 智能客服：基于产品资料与服务政策构建 7×24 小时客服
- 教育与培训：课程材料整合为个性化学习助手
- 研发助手：研究报告、技术文档、专利信息的检索问答
- 数据分析：通过 Text2Sql 用自然语言查询关系型数据库（Pro）

### AntSK 适合什么规模的企业？

从个人知识管理、小型团队到中大型企业的私有化知识中台均可。个人与小型团队可直接使用开源版；中大型企业与政企客户可结合 AntSK Pro 商业版与 AI 一体机方案交付。

### AntSK 是开源免费的吗？

开源版完全免费，且**可免费商用**（需保留 logo 与版权信息），包含知识库、对话应用、语义内核、内存内核、GPTs 生成、API 接口与本地模型等核心能力。AntSK Pro 商业版提供 GraphRAG、Text2Sql 高级功能、可视化流程编排、更精美的 UI 与 7×24 专业技术支持。

## 二、与竞品的对比

### AntSK 和 Dify 有什么区别？

| 对比项 | AntSK | Dify |
|--------|-------|------|
| 技术栈 | .NET 9 + Blazor | Python + React |
| 定位 | 企业级知识库 / 智能体 / AI 一体机 | 通用 LLM 应用开发平台 |
| 优势 | 企业级架构、内置 RBAC、全链路离线、信创适配、软硬一体交付 | 生态最大、工作流编排最强、模型接入最广 |
| 适合 | .NET/C# 团队、数据敏感与信创场景 | 需要复杂工作流的技术团队 |

### AntSK 和 RAGFlow 有什么区别？

RAGFlow 是 Python 技术栈的深度文档解析 RAG 引擎，在**复杂表格、扫描件、图文混排文档**的解析精度上更强。AntSK 的侧重点是企业级架构、私有化交付、全链路本地离线与国产信创适配，同时提供 .NET 插件体系与 OpenAI 兼容 API。若团队使用 .NET 技术栈或需信创交付，AntSK 更合适。

### AntSK 和 MaxKB、FastGPT 有什么区别？

MaxKB（飞致云出品）与 FastGPT 都是国内流行的开源知识库问答方案，分别基于 Python 与 Node.js。AntSK 的差异在于 .NET 技术栈、软硬一体 AI 一体机交付能力，以及对国产数据库与国产大模型的完整信创适配。

> 📊 完整横向对比表见 [竞品对比](./comparison.md)

## 三、部署与环境

### AntSK 部署难吗？

Docker 一行命令即可启动：

```bash
git clone https://github.com/AIDotNet/AntSK.git
cd AntSK
docker-compose -f docker-compose.simple.yml up -d
```

访问 `http://localhost:5000`。默认使用 SQLite + Disk 存储，**无需额外搭建数据库**。

### 硬件要求是什么？

| 场景 | 配置 |
|------|------|
| 最低 | 4 核 CPU / 8GB 内存 / 50GB SSD |
| 推荐 | 8 核 CPU / 16GB 内存 / 100GB SSD（可选 GPU 加速） |
| 生产 | 16 核 CPU / 32GB 内存 / 500GB SSD / RTX 3080 及以上 |

操作系统支持 Windows 10/11、Linux（Ubuntu 20.04+）、macOS 12+。若使用 LlamaFactory 需额外安装 Python 3.8+。

### AntSK 支持私有化离线部署吗？

支持。AntSK 可完全在无外网环境下运行：

- 本地推理引擎：Ollama、LLamaFactory（非 GGUF）、llama.cpp（GGUF）
- 本地向量模型：BGE embedding + BGE rerank
- 数据全部存储在本地，不出内网

### 支持哪些数据库和向量库？

| 类型 | 支持 |
|------|------|
| 关系型数据库 | PostgreSQL、SQLite、MySQL、SQL Server、Oracle、达梦（DM） |
| 向量库 | PostgreSQL(pgvector)、Qdrant、Redis、Disk、Memory、Azure AI Search |

默认 SQLite + Disk，零配置启动；生产环境推荐 PostgreSQL + pgvector。

## 四、模型与知识库

### AntSK 支持哪些大模型？

- **云端**：OpenAI、Azure OpenAI、讯飞星火、阿里云百炼 / 灵积
- **本地**：Ollama、LLamaFactory、llama.cpp（GGUF）
- **其他**：可通过 one-api 集成更多模型

### 支持哪些文档格式？

Word、PDF、Excel、TXT、Markdown、JSON、PPT。导入时自动完成文本提取、结构解析与切片，并构建全文索引与向量索引。

### 知识库切片大小怎么调？

技术文档场景一般建议 **512–1024 字符**区间：

- 切片过小 → 语义破碎，检索上下文不足
- 切片过大 → 结果冗余，token 开销高，模型抓不住重点

中文场景建议使用 BGE-embedding 模型，向量维度可选 768 或 1024 以平衡精度与性能。

### 为什么知识库导入慢 / 对话响应延迟高？

常见原因是导入任务与推理争抢算力。可调整 `appsettings.json`：

```json
"BackgroundTaskBroker": {
  "ImportKMSTask": {
    "WorkerCount": 1
  }
}
```

- 使用**在线 API 模型**：可适当调高 WorkerCount
- 使用**本地模型**：建议保持 1，否则容易内存溢出导致进程崩溃

此外建议使用 SSD 存储以提升向量检索速度，并启用 Redis 缓存热点数据。

### AntSK 会出现大模型幻觉吗？

AntSK 采用 RAG（检索增强生成）架构，回答基于检索到的知识库原文片段组织，并在引用文档内容时给出出处，可有效降低幻觉。Pro 版本进一步通过 GraphRAG 知识图谱做知识意图识别、实体补全与知识溯源。

## 五、安全与运维

### AntSK 的默认账号密码是什么？

**AntSK 不在文档中提供固定的默认弱口令。** 生产部署必须在**首次启动前**于 `appsettings.json` 的 `Login` 节点设置强口令：

```json
"Login": {
  "User": "<你的账号>",
  "Password": "<你的高强度口令>"
}
```

也可通过环境变量注入（推荐）：`Login__User`、`Login__Password`。

系统在检测到仍在使用默认口令时，会在启动日志中输出**醒目安全告警**。请勿使用 admin、123456、test 等弱口令。

### 生产环境还需要注意什么？

详见 [安全加固指南](../deploy/security.md)，核心包括：不要直接暴露公网、启用 HTTPS、收紧 CORS 来源、API 调用方使用独立 SecretKey 与最小权限、定期审计日志、及时备份与升级。

---

**还有疑问？** 欢迎在 [GitHub Issues](https://github.com/AIDotNet/AntSK/issues) 提出，或访问官网 <https://antsk.cn>。
