# Qinglai Knowledge Base / 清徕智能知识库

Public notes on RAG, Agents, ML basics, evaluation, backend practice, and interview topics.  
公开查阅的学习笔记：检索增强、智能体、机器学习基础、评估观测、后端实践与面试补强。

仓库地址：https://github.com/kwang-zoed/qinglai-knowledge-base

## Reading Order / 推荐阅读顺序

1. ML basics → 2. RAG → 3. Agent → 4. Eval (LangSmith) → 5. Backend → 6. Interview  
先打基础，再看检索与智能体，然后是评估与工程实践，最后补面试。

## Catalog / 目录

### 01-RAG-检索增强

| Doc | Summary |
| --- | --- |
| [RAG优化策略-Optimization.md](docs/01-RAG-检索增强/RAG优化策略-Optimization.md) | RAG 优化策略与客服场景 SOTA 流水线（含图转文） |

### 02-Agent-智能体

| Doc | Summary |
| --- | --- |
| [Agent协同与Function-Calling.md](docs/02-Agent-智能体/Agent协同与Function-Calling.md) | 多 Agent 状态传递、Function Calling 原理与生产避坑 |
| [AI-Agent防注入-Prompt-Injection.md](docs/02-Agent-智能体/AI-Agent防注入-Prompt-Injection.md) | Prompt Injection 攻防与检测思路 |

### 03-ML-机器学习

| Doc | Summary |
| --- | --- |
| [机器学习基础-Machine-Learning.md](docs/03-ML-机器学习/机器学习基础-Machine-Learning.md) | 监督学习 / 无监督 / 强化学习分类与处理流程 |

### 04-Eval-评估观测

| Doc | Summary |
| --- | --- |
| [LangSmith评估流程-Evaluation.md](docs/04-Eval-评估观测/LangSmith评估流程-Evaluation.md) | Dataset / Experiment / Feedback 离线评估逻辑 |

### 05-Backend-后端实践

| Doc | Summary |
| --- | --- |
| [智能客服售后系统-FastAPI实践.md](docs/05-Backend-后端实践/智能客服售后系统-FastAPI实践.md) | 智能客服售后：JWT、Session、注册登录与接口数据流 |

### 06-Interview-面试

| Doc | Summary |
| --- | --- |
| [Python面试补强-GIL内存与分布式锁.md](docs/06-Interview-面试/Python面试补强-GIL内存与分布式锁.md) | GIL、内存管理、数据结构、FastAPI、分布式锁 |

### 99-Journal-学习记录

| Doc | Summary |
| --- | --- |
| [七月安排及记录-July-Notes.md](docs/99-Journal-学习记录/七月安排及记录-July-Notes.md) | 七月学习安排、微调笔记、Harness 七层骨架等流水账 |

## Naming / 命名约定

- Folder: `序号-English-中文`（如 `01-RAG-检索增强`）
- File: `中文标题-English-Keywords.md`
- Images in source notes were converted to text blocks marked `（原图转文字）`

## License note

Personal study notes. Feel free to read; please attribute if you republish large excerpts.  
个人学习笔记，欢迎查阅；大量转载请注明出处。
