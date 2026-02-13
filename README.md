# Opsevo

<div align="center">

![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)
![Node](https://img.shields.io/badge/node-%3E%3D18.0.0-green.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)
![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)
![Vue](https://img.shields.io/badge/Vue-3.4-42b883.svg)

**智能网络运维平台 - 基于 AI 的 RouterOS 自动化运维系统**

[功能特性](#核心功能) • [快速开始](#快速开始) • [架构设计](#系统架构) • [API 文档](#api-端点) • [部署指南](#docker-部署)

</div>

---

## 项目简介

Opsevo 是一个智能网络运维管理平台，专为 MikroTik RouterOS 设备设计。通过集成先进的 AI 技术（支持 OpenAI、DeepSeek、Gemini、通义千问、智谱等多种 LLM），实现网络设备的智能监控、告警分析、故障自愈和自然语言交互。

### 核心价值

- 🤖 **AI 驱动运维** - 自然语言交互，智能告警分析，自动故障诊断
- 🔧 **自动化修复** - 基于 ReAct 循环的智能修复建议和自动执行
- 📊 **实时监控** - 流量分析、设备状态、告警管理一体化
- 🛡️ **安全可靠** - 多层安全机制，支持人工审批和安全模式

---

## 核心功能

### 🤖 AI 智能运维

#### 统一 AI Agent

基于大语言模型的智能运维助手：

- 支持多种 AI 服务商（OpenAI、DeepSeek、Gemini、通义千问、智谱）
- 三种工作模式：标准对话、知识增强（RAG）、ReAct Agent（多步推理）
- 流式响应、多轮对话、会话管理、消息收藏转知识

#### RAG 知识库

检索增强生成系统：

- LanceDB 向量存储，支持手动添加和对话提取知识
- 智能检索器（意图分析）、可信度计算（多维度评估）
- 知识格式化输出、引用追踪、有界缓存

#### ReAct 推理引擎

Thought → Action → Observation 循环：

- 自动选择执行工具（设备查询、知识搜索、告警分析等）
- 80+ RouterOS API 路径参考，失败重试和错误恢复
- 智能知识应用、输出验证、使用追踪
- **语义化循环检测** - 规范化参数比较，防止重复调用

#### Reranker 重排序

检索结果二次排序：

- BGE-Reranker 兼容 API，MMR 多样性选择
- 复合评分机制，阈值降级重试

#### Skill 技能系统

可扩展的技能管理：

- 内置技能和自定义技能支持
- 技能链管理、语义匹配、参数调优
- 技能感知的知识检索和工具选择

#### 并行执行系统 (NEW)

高效的多工具并发执行：

- **自适应模式选择** - SEQUENTIAL/PARALLEL/PLANNED 三种执行模式
- **依赖分析器** - 自动识别工具间的数据和资源依赖
- **熔断器保护** - 防止对失败工具的重复调用，支持自动恢复
- **可取消超时机制** - 防止内存泄漏，确保资源正确清理
- **完整回退链** - PLANNED → PARALLEL → SEQUENTIAL 优雅降级
- **并发安全** - 请求级别隔离，避免共享状态竞态条件

### 🛠️ AI-Ops 智能运维平台

#### 监控与告警

- 统一运维仪表盘（系统信息、资源监控、流量图表、告警概览）
- 智能告警系统（多级别告警、自定义规则、噪声过滤）
- Syslog 集成（自动转换为告警事件、修复方案生成）
- 多渠道通知（Web 推送、Webhook、邮件、企业微信、钉钉）

#### 自动化运维

- 定时巡检任务（Cron 调度）
- 配置快照管理（差异对比、一键恢复）
- 健康报告生成（Markdown/PDF 导出）
- 故障自愈引擎（PPPoE 断线重连、接口重启等）
- **脚本执行超时保护** - Promise.race 实现，防止系统阻塞
- **安全模式自动恢复** - 定时检查恢复条件，最大持续时间限制

#### Critic/Reflector 自愈迭代

智能故障修复闭环：

- CriticService：多维度评估修复效果
- ReflectorService：深度分析与学习
- IterationLoop：Execute → Evaluate → Reflect → Decide 循环
- 六种行动决策（重试/修改/替代/升级/回滚/完成）
- **Promise 回调正确性** - 真正的 resolve/reject 回调，超时通知

#### 告警处理流水线

完整的告警处理链：

- AlertPreprocessor：告警预处理和标准化
- NoiseFilter：噪声告警过滤
- RootCauseAnalyzer：根因分析
- RemediationAdvisor：修复方案建议
- DecisionEngine：智能决策引擎
- **pending 状态自动清理** - 防止内存泄漏

### 📋 RouterOS 管理

#### 网络配置

- 接口管理（物理接口、L2TP/PPPoE Client、VETH）
- IP 地址、路由、IP Pool、DHCP Client/Server
- IPv6 完整支持（地址、DHCPv6、ND、邻居表、路由）
- ARP 表管理

#### 安全管理

- 防火墙规则（Filter、NAT、Mangle、Address List）
- IPv6 防火墙
- 已知问题管理、维护窗口配置

#### 系统管理

- 容器管理（Docker 启停、环境变量、挂载点）
- 计划任务、脚本管理、电源管理
- Socksify 代理配置
- 审计日志查看

---

## 系统架构

```text
┌─────────────────────────────────────────────────────────────────────┐
│                     Frontend (Vue 3 + Element Plus)                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐ │
│  │Dashboard │ │ AI Chat  │ │ AI-Ops   │ │ Network  │ │  System   │ │
│  │  View    │ │  View    │ │  Views   │ │  Views   │ │   Views   │ │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              │ REST API / SSE
┌─────────────────────────────────────────────────────────────────────┐
│                    Backend (Node.js + Express + TypeScript)          │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                        AI-Ops Services                           ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │   Decision   │ │ Remediation  │ │    Fault     │            ││
│  │  │    Engine    │ │   Advisor    │ │   Healer     │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │   Critic/    │ │   Alert      │ │   Syslog     │            ││
│  │  │  Reflector   │ │  Pipeline    │ │  Receiver    │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │  Parallel    │ │  Circuit     │ │  Dependency  │            ││
│  │  │  Executor    │ │  Breaker     │ │  Analyzer    │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                         RAG Services                             ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │    RAG       │ │   ReAct      │ │  Knowledge   │            ││
│  │  │   Engine     │ │  Controller  │ │    Base      │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │  Embedding   │ │  Reranker    │ │   Intent     │            ││
│  │  │   Service    │ │   Service    │ │  Analyzer    │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                        Skill System                              ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            ││
│  │  │    Skill     │ │    Skill     │ │    Skill     │            ││
│  │  │   Manager    │ │   Matcher    │ │   Registry   │            ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘            ││
│  └─────────────────────────────────────────────────────────────────┘│
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│  │RouterOS  │ │  Alert   │ │ Metrics  │ │Scheduler │              │
│  │ Client   │ │ Engine   │ │Collector │ │ Service  │              │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────────┐
│                        External Services                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│  │RouterOS  │ │   LLM    │ │Telegram/ │ │ Syslog   │              │
│  │ Devices  │ │Providers │ │ WeChat   │ │ Sources  │              │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

### 技术栈

| 层级 | 技术 |
| ---- | ---- |
| **前端** | Vue 3.4, TypeScript 5.2, Element Plus 2.4, ECharts 6, Vite 5 |
| **后端** | Node.js 18+, Express 4, TypeScript 5.3, node-routeros |
| **数据库** | LanceDB (向量数据库), JSON 文件存储 |
| **AI/LLM** | OpenAI, DeepSeek, Gemini, 通义千问, 智谱 |
| **测试** | Jest, Vitest, fast-check (属性测试) |
| **部署** | Docker, Docker Compose, Nginx |

---

## 快速开始

### 环境要求

- Node.js >= 18.0.0
- RouterOS 设备（需开启 API 服务）

### 开发环境

```bash
# 克隆项目
git clone https://github.com/btnalit/opsevo.git
cd opsevo

# 安装依赖
cd backend && npm install
cd ../frontend && npm install

# 启动服务
cd backend && npm run dev    # 端口 3099
cd frontend && npm run dev   # 端口 5173

# 或使用根目录并发启动
npm run dev
```

### Docker 部署

```bash
# 使用预构建镜像
docker pull ghcr.io/btnalit/opsevo:latest
docker run -d --name opsevo -p 8080:3099 \
  -v opsevo-data:/app/backend/data \
  -v opsevo-logs:/app/backend/logs \
  ghcr.io/btnalit/opsevo:latest

# 或使用 Docker Compose
docker-compose -f docker-compose.simple.yml up -d

# 完整部署（含 Nginx）
docker-compose up -d
```

### 环境变量

| 变量名 | 默认值 | 说明 |
| ------ | ------ | ---- |
| PORT | 8080 | 外部访问端口 |
| NODE_ENV | production | 运行环境 |
| LOG_LEVEL | info | 日志级别 |
| SYSLOG_PORT | 514 | Syslog UDP 端口 |
| SKILL_SYSTEM_ENABLED | true | 是否启用 Skill 系统 |
| RERANK_API_KEY | - | Reranker API 密钥 |
| RERANK_BASE_URL | - | Reranker API 地址 |
| RERANK_MODEL_NAME | bge-reranker-v2-m3 | Reranker 模型名称 |

---

## RouterOS 配置

```routeros
# 启用 API 服务
/ip service set api disabled=no port=8728

# 创建 API 用户
/user add name=api password=yourpassword group=full

# Syslog 配置（可选，用于告警集成）
/system logging action add name=remote target=remote remote=192.168.1.100 remote-port=514
/system logging add topics=!debug action=remote
```

---

## 项目结构

```text
opsevo/
├── backend/                      # 后端 API 服务
│   ├── src/
│   │   ├── controllers/          # 控制器层
│   │   ├── routes/               # 路由定义
│   │   ├── services/
│   │   │   ├── ai/               # AI Agent 服务
│   │   │   │   ├── adapters/     # LLM 适配器
│   │   │   │   └── ...
│   │   │   ├── ai-ops/           # AI-Ops 智能运维
│   │   │   │   ├── rag/          # RAG 知识库子系统
│   │   │   │   └── skill/        # Skill 技能系统
│   │   │   └── core/             # 核心服务
│   │   ├── types/                # 类型定义
│   │   └── utils/                # 工具函数
│   ├── data/                     # 数据存储
│   │   └── ai-ops/               # AI-Ops 数据
│   │       ├── alerts/           # 告警数据
│   │       ├── analysis/         # 分析结果
│   │       ├── audit/            # 审计日志
│   │       ├── decisions/        # 决策历史
│   │       ├── rag/              # RAG 知识库
│   │       ├── remediations/     # 修复计划
│   │       ├── skills/           # 技能配置
│   │       └── snapshots/        # 配置快照
│   └── logs/                     # 日志文件
├── frontend/                     # 前端 Vue 应用
│   └── src/
│       ├── api/                  # API 请求封装
│       ├── components/           # 公共组件
│       ├── views/                # 页面组件 (40+ 视图)
│       ├── stores/               # Pinia 状态
│       └── utils/                # 工具函数
├── Dockerfile
├── docker-compose.yml
└── nginx.conf
```

---

## API 端点

后端服务运行在端口 `3099`：

| 模块 | 端点前缀 | 说明 |
| ---- | -------- | ---- |
| 系统 | `/api/health`, `/api/dashboard` | 健康检查、资源信息 |
| 连接 | `/api/connection` | RouterOS 连接管理 |
| 接口 | `/api/interfaces` | 网络接口管理 |
| IP | `/api/ip` | 地址、路由、DHCP、防火墙 |
| IPv6 | `/api/ipv6` | IPv6 完整管理 |
| 容器 | `/api/container` | Docker 容器管理 |
| 系统 | `/api/system` | 任务、脚本、电源 |
| AI Agent | `/api/unified-agent` | 会话、对话、收藏 |
| RAG | `/api/rag` | 知识库管理 |
| AI-Ops | `/api/ai-ops` | 监控、告警、快照、报告 |
| Skill | `/api/skills` | 技能管理 |
| 模板 | `/api/prompt-templates` | Prompt 模板管理 |

---

## 通知渠道配置

### 企业微信

```json
{
  "msgtype": "markdown",
  "markdown": {
    "content": "## 🚨 Opsevo 告警\n**{{title}}**\n{{body}}\n> 级别: {{severity}} | 时间: {{timestamp}}"
  }
}
```

### 钉钉

```json
{
  "msgtype": "markdown",
  "markdown": {
    "title": "{{title}}",
    "text": "## {{title}}\n{{body}}\n- 级别: {{severity}}\n- 时间: {{timestamp}}"
  }
}
```

---

## 测试

```bash
# 运行所有测试
npm test

# 后端测试
cd backend && npm test

# 前端测试
cd frontend && npm test

# 监听模式
cd backend && npm run test:watch
cd frontend && npm run test:watch
```

---

## 可靠性特性

基于 `bugfix-ai-ops-reliability` 规范实现的可靠性增强：

| 特性 | 说明 |
| ---- | ---- |
| **Promise 回调正确性** | IterationLoop 队列使用真正的 resolve/reject 回调 |
| **脚本执行超时保护** | FaultHealer 使用 Promise.race 实现 30 秒超时 |
| **安全模式自动恢复** | 5 分钟检查间隔，1 小时最大持续时间 |
| **RAGEngine 生命周期** | shutdown() 方法正确释放定时器资源 |
| **通知渠道回退** | 默认决策时回退到所有已启用渠道 |
| **pending 状态清理** | 1 小时后自动清理过期的修复计划 |
| **语义化循环检测** | 规范化参数比较，60 秒时间窗口检测 |
| **并行执行系统** | 自适应模式选择，熔断器保护，完整回退链 |
| **可取消超时机制** | try-finally 确保资源清理，防止内存泄漏 |
| **并发安全隔离** | 请求级别执行上下文，避免共享状态竞态 |

---

## 开发规范

- 使用 TypeScript 严格模式
- 遵循 ESLint + Prettier 规则
- 编写单元测试和属性测试
- 更新相关文档

---

## 许可证

MIT License - 允许自由使用、修改和分发，需保留版权声明。

---

<div align="center">

**⭐ 如果这个项目对你有帮助，请给一个 Star！**

</div>
