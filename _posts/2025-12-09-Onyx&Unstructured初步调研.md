---
title:      Onyx&Unstructured-RAG调研1（无技术细节版）
date:       2025-12-09
categories:
  - Tech
tags:
  - Agent
  - RAG
---

正好要做市场竞品调研，个么正好，本来也需要学习下市面上新的一些技术方案，就简单先看一个两个RAG方案的供应商：Onyx 和 Unstructured。

# 一、调研摘要

| 维度 | **Onyx** | **Unstructured** |
| --- | --- | --- |
| 成立时间 | 2023年 | 2022年 |
| 创始人 | 2位联合创始人（Chris Weaver, Yuhong Sun） | Brian Raymond（CEO） |
| 融资情况 | $10M Seed轮（Khosla Ventures + First Round + YC） | 未公开具体金额，注重快速商业化 |
| 团队规模 | 10-25人（估计） | 11-50人 |
| 客户数量 | "dozens of enterprise customers" | 65,000+组织（80%为《财富》1000强企业） |
| 客户画像 | 科技/信息密集型企业，具备 IT/平台团队，接受开源+自部署 | 企业AI团队、数据工程师（处理非结构化数据） |
| 产品形态 | 开源（Community）+ Cloud（Business $16/u/m）+ Enterprise | 开源库 + Serverless API + 企业级ETL平台 |
| 交付模式 | Cloud 为主，支持 Self-host（含 Air-gapped）与混合部署 | API优先 + 可视化UI + VPC部署 |
| 关键能力 | 40+ **数据源连接器**、RAG（带**来源的生成**与**权限过滤**） | 非结构化数据→LLM-ready JSON，OCR/表格/图片增强 |

# 二、Onyx

## 1.客户群体

基于公开客户推断：

**客户职能**（基于连接器和用例权重）：

*   工程团队：Code Repo + Jira（~40%）
    
*   销售/客服：Salesforce + Zendesk + Ramp案例（~30%）
    
*   全员/运营：Slack + Drive + "Company Wide"（~30%）
    

**垂类**（基于公开客户）：

*   科技SaaS：Netflix、Bitwarden（多数）
    
*   金融科技：Ramp（核心案例）
    
*   安全/防务：Thales（企业级验证）
    

## 2.核心功能架构

Data Source（40+ Connectors） -> RAG（Hybrid + LangGraph） -> Agent（Web / Slack / API）

### 2.1 数据源连接器（40+）

| 分类 | 代表工具 | 场景 |
| --- | --- | --- |
| Popular | Slack, Confluence, Notion, GitHub | 日常协作 |
| Knowledge Base | GitBook, Document360 | 文档查询 |
| Cloud Storage | Google Drive, S3, OCI | 文件检索 |
| Ticketing | Jira, Zendesk, Linear | 工单历史 |
| Messaging | Slack, Teams | 对话智慧 |
| Sales | Salesforce, HubSpot | 销售赋能 |
| Code Repo | GitHub, GitLab, Bitbucket | 工程知识 |

### 2.2 RAG 技术栈

\- 数据采集 → 权限继承的统一索引

\- Hybrid Search（BM25 + 向量 + Rerank）

\- Agent Search（LangGraph 多步拆解）

\- 生成+引用（**带来源生成与权限过滤**）

\- 差异化：企业级权限过滤 + 多源平衡 + 观测闭环

### 2.3 Agent 平台

\- Agent = 指令 + 知识源 + Actions（MCP/OpenAPI）

\- Actions：Jira 创建工单、CRM 更新、OAuth 集成

\- Agent Search：复杂问题 → 子任务 → 多源检索 → 聚合验证

\- 子 Agent：按部门定制（Support Agent、Sales Agent 等）

## 3.交付三种模式

\- Cloud：注册 → 连接数据源 → 使用（Business / Enterprise）

\- Self-host：Kubernetes 部署，支持 Air-gapped

\- 混合：客户 LLM + Onyx 检索/Agent

---

# 三、Unstructured

## 1.客户群体

1.  **客户职能分布**
    
    *   数据工程/AI平台团队：90%（数据清洗、RAG预处理）
        
    *   业务分析：5%（合规、财务报表）
        
    *   研发团队：5%（模型训练数据准备）
        
2.  **行业分布**
    
    *   金融/保险：30%（贷款单、理赔单、监管文件）
        
    *   医疗：20%（病历、影像报告）
        
    *   企业服务：20%（合同、HR文件）
        
    *   政府/合规：15%（审计文件）
        
    *   其他：15%
        

## 2.核心功能架构

### 2.1 文件处理流程

**Input**

**Partition**

把原始文件**智能拆分成语义元素**（Title、Paragraph、Table、Image等）

**Cleaning**

**Enrichment**（VLM）**商业版才有！**

用视觉模型增强：

1.  图片描述：VLM生成"这张图是销售漏斗图"
    
2.  生成式OCR：VLM修正传统OCR错误
    
3.  表格→HTML：VLM识别表格结构，输出可读HTML
    

**Chunking**

**Embedding**

**Staging（json、csv等）**

**preservation（向量数据库、s3等）**

**支持格式（40+）**

| **分类** | **代表格式** | **典型场景** |
| --- | --- | --- |
| 办公文档 | DOCX/PPTX/XLSX | 企业内部文件 |
| PDF/扫描件 | PDF/扫描OCR+布局解析 | 文档数字化 |
| 图片/多模态 | JPG/PNG/HEIC | 图片描述提取 |
| 邮件/网页 | EML/HTML/MSG | 通信记录解析 |
| 表格密集 | XLS/TSV/CSV→HTML | 结构化表格转换 |
| 特殊格式 | EPUB/RTF/ODT | 遗留文档处理 |

## 3.交付模式

1.  **Serverless API**
    
    *   按页付费：$0.0003-0.003/页
        
    *   响应速度：10ms
        
2.  **Business SaaS**
    
    *   定价：$X/用户/月
        
    *   功能：多用户协作 + 工作区管理
        
3.  **Dedicated VPC**
    
    *   适用场景：金融/医疗级安全需求
        
    *   特点：私有云隔离
        
4.  **In-VPC自部署**
    
    *   模式：客户云上自主运行
        
    *   优势：零供应商锁定

# 四、小小震撼

这两家都是技术驱动型的公司，而且都是"以点打面"的典范。Unstructured基本只做数据的解析；Onyx虽然Agent全流程都有，但Agent的workflow做得相当毛坯，但是在数据来源、RAG的召回方面做了很多的优化（有点像Cognition的风格）。两家体量都不大，但都在某个环节上做到了极致（好吧，说极致还太早，至少行业领先是肯定的）。这也一定是他们能快速崛起的最重要的原因之一。
