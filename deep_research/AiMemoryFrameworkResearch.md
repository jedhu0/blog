

# **AI 记忆层：面向有状态 LLM 应用的主流框架对比分析**

---

### **执行摘要**

随着大型语言模型（LLM）从单纯的文本生成器向能够执行复杂任务的智能代理（Agent）演进，其固有的“无状态”特性已成为发展的核心瓶颈。为了克服这一局限，一个名为“AI 记忆”的新兴基础设施层应运而生，其目标是使应用程序和代理能够跨越单次会话的边界，实现回忆、个性化和自适应学习 1。本报告旨在对当前主流的 AI 记忆框架进行详尽的调研与对比分析，为技术领导者和 AI 工程师在选择关键技术栈时提供战略性参考。

本报告深入剖析了七个在市场上备受关注的记忆框架，它们代表了不同的架构范式：

1. **混合数据存储（Hybrid Datastores）**：以 **Mem0** 为代表，通过结合向量、图和键值存储，在性能、成本和功能之间寻求务实的平衡。  
2. **时序知识图谱（Temporal Knowledge Graphs）**：以 **Zep** 为代表，专注于构建能够追踪信息随时间演变的关系图谱，尤其适用于复杂的企业级应用。  
3. **操作系统抽象（OS Abstractions）**：以 **Letta (前身为 MemGPT)** 和 **MemOS (MemTensor)** 为代表，它们借鉴传统操作系统的概念，将 LLM 的上下文窗口视为“内存（RAM）”，将外部存储视为“硬盘（Disk）”，并由代理自身进行管理。  
4. **用户画像模型（User-Profile Models）**：以 **Memobase** 为代表，其核心理念是为“用户”而非“代理”构建记忆，专注于生成和维护结构化的、不断演进的用户画像。  
5. **集成式 SDK（Integrated SDKs）**：以 **LangMem** 和 **OpenAI 原生记忆功能**为代表，它们并非独立的平台，而是深度集成于现有生态系统（如 LangChain 和 ChatGPT）的功能模块，优先考虑无缝的开发体验。

核心分析发现，这些框架在架构理念、性能表现、开发者体验和商业模式上存在显著差异。性能基准测试（如 LOCOMO 和 LongMemEval）的结果充满了争议，这揭示了当前行业在记忆系统评估标准上的不成熟。因此，技术选型不应仅依赖于公开的性能数据，而更应关注框架的架构是否与具体的业务需求相匹配。例如，需要处理复杂时序关系的应用更适合 Zep，而追求快速原型开发和用户画像构建的场景则更倾向于 Memobase。

[本报告](https://gemini.google.com/share/dac97b9ce586)的战略性建议是，技术决策者应根据项目的核心需求——无论是快速上市、深度定制能力、企业级的时序推理，还是生态系统的无缝集成——来选择最合适的记忆框架。最终的选择将深刻影响 AI 应用的最终能力、运营成本和用户体验。

---

### **第一节：从无状态 LLM 到有状态代理的范式转变**

本章节旨在阐明 AI 记忆框架所要解决的根本性问题，为后续的深入分析提供必要的背景。大型语言模型虽然在自然语言处理方面取得了巨大成功，但其核心的“无状态”设计使其无法自然地在多次交互中保留信息，这构成了实现真正智能和个性化代理的主要障碍。

#### **1.1. 超越上下文窗口：LLM 的内在局限**

从根本上说，标准的大型语言模型是一个无状态的函数：它接收一段文本输入，并生成一段文本输出，每次交互都是一个独立的事件，模型本身不会保留之前交互的任何记忆 3。我们日常体验到的“对话记忆”实际上是一种假象，其实现方式是将整个对话历史作为新的输入，重新提交给模型处理 4。这种机制依赖于模型的“上下文窗口”（Context Window），即模型单次能够处理的文本长度（以 token 计）。

尽管技术不断进步，上下文窗口的容量从几千 token 扩展到数百万 token，但仅仅扩大窗口并不能从根本上解决记忆问题，反而带来了新的挑战：

* **性能与成本问题**：上下文窗口的长度与处理所需的计算资源和时间成指数级增长。对于长对话，这意味着更高的延迟和显著增加的 API 调用成本，使其在实际应用中不具备经济可行性 6。  
* **信息召回率下降**：研究表明，当上下文窗口变得极长时，模型在信息检索任务中的准确率会下降，即所谓的“大海捞针”（Needle-in-a-Haystack）问题。模型难以从海量无关信息中精准定位并利用关键细节 6。

这些局限性表明，业界正在从一种“暴力破解”式的方法（无限扩大上下文窗口）转向一种更精巧的架构性解决方案（智能化的记忆管理）。AI 记忆框架的出现，正是对“无限上下文”作为终极解决方案这一理念失效的直接回应。这一转变对于那些期望下一代 LLM 能自动解决所有记忆问题的技术决策者而言，是一个至关重要的认知。

#### **1.2. 从 RAG 到推理：外部知识的演进**

检索增强生成（Retrieval-Augmented Generation, RAG）是解决 LLM 知识局限性的早期关键技术。其核心思想是在生成回答前，从外部知识库（如文档集合）中检索相关信息，并将其注入到模型的上下文中。然而，传统的 RAG 是一种“无状态的变通方案” 9。它虽然引入了外部知识，但存在以下根本性缺陷：

* **知识是静态的**：RAG 通常从静态文档中读取信息，无法处理随时间变化或相互矛盾的知识。例如，它很难理解一个用户最初喜欢 A 品牌，后来因为一次糟糕的体验而转向 B 品牌这一动态过程 10。  
* **缺乏生命周期管理**：RAG 检索出的信息片段没有生命周期控制，无法实现知识的更新、修正或废弃 9。

从 RAG 到现代记忆框架的演进，标志着 AI 与知识交互方式的根本性转变——从一个“只读”模式演变为一个“读写”模式。传统 RAG 将外部知识视为一个静态的、仅供查询的图书馆。而 Mem0、Zep 和 Letta 等现代框架则赋予了代理“写入”或修改其记忆的能力。代理可以根据新的交互来更新用户偏好、将过时的事实标记为无效，或整合新的信息 11。这种“读写”能力是实现真正学习和适应的基础，其概念远比简单的信息检索更为强大和深刻。

#### **1.3. 定义现代 AI 记忆：一个认知框架**

现代 AI 记忆系统的设计理念深受人类认知科学的启发，旨在模拟人类复杂的记忆机制 13。一个完整的记忆框架通常包含以下核心组件：

* **短期记忆 (Short-Term Memory, STM)**：也称为工作记忆，用于追踪最近的交互信息，对维持即时对话的连贯性和执行多步推理至关重要 13。这在功能上类似于 LLM 的当前上下文窗口。  
* **长期记忆 (Long-Term Memory, LTM)**：这是 AI 记忆框架的核心。它负责跨会话、跨用户、跨应用地持久化存储知识，使代理能够积累经验并不断进化 19。

长期记忆本身又可以根据存储内容的性质，细分为不同的类型：

* **情景记忆 (Episodic Memory)**：负责记录具体的事件和体验，通常带有时间戳。例如，记住“上周二和用户讨论过项目 A 的预算问题” 14。  
* **语义记忆 (Semantic Memory)**：存储结构化的事实、概念和用户偏好。例如，“用户的名字是张三，他是一名素食主义者” 14。  
* **程序记忆 (Procedural Memory)**：存储技能、规则和习得的行为模式，使代理能够自动化地执行任务。例如，学习到“当用户询问订单状态时，应首先调用订单查询工具” 14。

这些不同类型的记忆协同工作，共同构成了一个能够让 AI 代理超越简单问答、实现持续学习和深度个性化的认知基础。

---

### **第二节：架构深度剖析：AI 记忆的竞争哲学**

本章节将对每个框架的核心设计进行详细的技术分解，阐明它们的工作原理以及彼此之间的本质区别。这些框架代表了解决 AI 记忆问题的不同哲学思想，从务实的工程实现到前沿的理论探索。

#### **2.1. Mem0：务实的混合数据存储**

* **核心哲学**：Mem0 的设计理念体现了高度的工程实用主义，旨在构建一个可扩展、面向生产的架构，通过动态提取、整合和检索对话中的关键信息，在性能、能力和成本之间取得最佳平衡 23。  
* **架构**：  
  * **两阶段流水线 (Two-Phase Pipeline)**：记忆处理分为两个阶段。首先是“提取（Extraction）”阶段，系统使用 LLM 从对话中识别并提取候选记忆。随后是异步的“更新（Update）”阶段，LLM 会根据新信息与现有记忆的语义关系，决定执行 ADD（新增）、UPDATE（更新）、DELETE（删除）或 NOOP（无操作）中的一种，以保证记忆库的连贯性和无冗余性 6。  
  * **混合数据存储 (Hybrid Datastore)**：这是 Mem0 架构的基石。它战略性地组合了多种数据库以优化不同类型信息的存储和检索效率：使用向量数据库（如 ChromaDB、Qdrant）进行语义相似度搜索；使用键值存储（Key-Value Store）快速访问结构化的事实和偏好；使用图数据库（如 Neo4j）来维护实体间的复杂关系 25。其集成了图数据库的版本被称为  
    Mem0g 11。  
  * **评分层 (Scoring Layer)**：在检索阶段，从各个数据存储中获取的候选记忆会经过一个评分层。该层级会综合考虑记忆的相关性、重要性和时近性（recency）进行打分，最终只将最有用、最相关的上下文提供给 LLM 27。  
* **关键特性**：支持多层级记忆（用户、会话、代理状态），提供对开发者友好的 API 和多平台 SDK，并始终将降低 token 消耗和响应延迟作为核心优化目标 27。

#### **2.2. Zep & Graphiti：企业级时序知识图谱**

* **核心哲学**：Zep 认为，记忆不仅是孤立事实的集合，更是一个不断演进的、具有强大时间维度的关系网络。这种理念使其特别适合需要深刻理解信息“如何”以及“何时”发生变化的企业级应用场景 30。  
* **架构**：  
  * **Graphiti 引擎**：Zep 的核心是其开源的 Graphiti 引擎。Graphiti 能够从非结构化（如对话）和结构化（如业务数据）的“事件（episodes）”中，自主地构建一个时序感知的知识图谱 30。  
  * **双时序数据模型 (Bi-Temporal Data Model)**：这是 Zep 的关键技术壁垒。图中的每一个事实（边）都记录了四个时间戳：created\_at（Zep 得知该事实的时间）、valid\_at（事实在现实世界中发生的时间）、invalid\_at（事实失效的时间）以及 expired\_at（Zep 得知事实失效的时间）。这种设计支持精确的“时间点（point-in-time）”查询和复杂的矛盾处理机制 32。  
  * **分层图结构 (Hierarchical Graph)**：Zep 的知识图谱由三个层级构成：**事件子图 (Episode Subgraph)** 存储原始输入数据；**语义实体子图 (Semantic Entity Subgraph)** 包含从事件中提取的实体（节点）和关系（边）；**社区子图 (Community Subgraph)** 则是由紧密连接的实体组成的聚类，代表了更高层次的抽象概念 34。  
* **关键特性**：自动化的实体与关系提取，结合了语义、关键词和图遍历的混合检索策略，以及在时序推理任务上的卓越性能 32。

#### **2.3. Letta (前 MemGPT)：LLM 即操作系统**

* **核心哲学**：Letta 的设计源于其创始团队在加州大学伯克利分校发表的著名研究论文《MemGPT: 迈向作为操作系统的 LLM》。其核心思想是将 LLM 有限的上下文窗口比作计算机的“物理内存（RAM）”，将外部存储比作“硬盘（Disk）”。而代理本身（由 LLM 驱动）则扮演“操作系统”的角色，通过在“内存”和“硬盘”之间分页调度信息来管理一个看似无限的“虚拟上下文” 12。  
* **架构**：  
  * **记忆层级 (Memory Hierarchy)**：Letta 将记忆明确划分为两类：**核心记忆 (Core Memory)**，这是存在于上下文窗口内的“内存”，用于存放代理的角色设定（Persona）和用户信息（Human）；以及**外部记忆 (External Memory)**，这是脱离上下文的“硬盘”，用于归档海量对话历史（Recall Memory）和进行向量检索（Archival Memory） 38。  
  * **通过工具调用实现自我编辑记忆 (Self-Editing Memory via Tool Calling)**：代理通过调用一系列预定义的、专门用于记忆管理的工具（函数）来主动读写其记忆库。例如，代理可以调用 core\_memory\_replace 工具来更新自己对用户的认知，或者调用搜索工具从归档记忆中查找信息 12。  
  * **持久化层 (Persistence Layer)**：所有代理的状态，包括其全部记忆，都由 Letta 服务器持久化到一个数据库中。官方推荐使用 PostgreSQL 以支持版本间的平滑迁移，而默认的 pip 安装则使用 SQLite 43。  
* **关键特性**：提供作为独立服务运行的有状态代理，配备用于可视化和调试的代理开发环境（Agent Development Environment, ADE），并原生支持多代理协作 44。

#### **2.4. MemOS (MemTensor)：统一的抽象层**

* **核心哲学**：MemOS 旨在将“记忆”提升为一种与 CPU、存储同等重要的、可被调度和管理的一等系统资源。其宏大愿景是创建一个统一的、可治理的框架，以整合并调度目前分散存在的各种记忆类型 9。  
* **架构**：  
  * **三层系统架构 (Three-Layer System)**：该架构自顶向下分为：**接口层 (Interface Layer)**，负责解析用户查询并转换为记忆 API 调用；**操作层 (Operation Layer)**，负责记忆的调度、生命周期管理和演化；**基础设施层 (Infrastructure Layer)**，负责底层存储和治理（如访问控制） 45。  
  * **“记忆立方体” (MemCube)**：这是 MemOS 的核心抽象。MemCube 是一个标准化的数据结构，用于封装任何类型的记忆内容，并附带丰富的元数据，如来源、版本、治理策略等。这使得对不同记忆进行统一的操作系统级调度成为可能 45。  
  * **统一的记忆类型 (Unified Memory Types)**：MemOS 明确地建模并统一管理三种核心记忆类型：**明文记忆 (Plaintext Memory)**，如外部文档和知识库；**激活记忆 (Activation Memory)**，如推理过程中的 KV 缓存和注意力状态；以及**参数记忆 (Parametric Memory)**，即固化在模型权重（如 LoRA 适配器）中的知识 9。  
* **关键特性**：具备预测性的、意图感知的调度机制，能够预加载相关记忆以降低延迟；采用树状层级结构并支持图风格的交叉链接；其最终愿景是实现跨模型、跨平台的记忆共享 45。

#### **2.5. Memobase：以用户为中心的画像模型**

* **核心哲学**：Memobase 的理念独树一帜，它明确提出“为用户记忆，而非为代理记忆”（Memory for User, not Agent）。其核心目标是构建和维护一个结构化的、不断演进的用户画像，而不是简单地存储对话流水账 55。  
* **架构**：  
  * **基于画像的记忆 (Profile-Based Memory)**：系统从交互中提取有意义的用户洞察，并将其填充到一个结构化的用户画像中，从而避免了原始对话记录带来的“数据膨胀”问题。开发者可以自定义和控制画像的模式（schema） 55。  
  * **异步批处理 (Asynchronous Processing)**：Memobase 为每个用户设置了一个缓冲区（buffer zone），用于暂存新产生的数据（称为“blobs”）。系统会异步地、分批次地处理这些数据来更新用户画像。处理的触发条件可以是缓冲区满、长时间无活动，或者通过 flush() API 手动触发。这种设计确保了与用户直接交互的“热路径”始终保持高速响应 55。  
  * **非嵌入式方法 (Non-Embedding Approach)**：Memobase 的一个显著特点是，其核心的用户画像构建过程不依赖于向量嵌入或图数据库，这旨在实现更低的延迟和更简单的基础设施架构 57。  
* **关键特性**：支持时间感知的记忆（在画像中存储具体日期），提供可控的画像模式，并提供简洁的 API，可将完整的用户画像作为单个字符串注入到 LLM 的提示中 55。

#### **2.6. LangMem：可扩展的 LangGraph 原生 SDK**

* **核心哲学**：LangMem 是 LangChain 团队推出的一个 SDK，旨在提供一套与存储后端无关、但与 LangGraph 生态系统深度集成的记忆工具。它不提供一个大而全的平台，而是为开发者提供构建特定应用记忆系统的基础“原语”（primitives） 60。  
* **架构**：  
  * **核心原语 (Core Primitives)**：提供一个无状态的核心 API，用于执行记忆的转换操作（如新增、更新、删除），可以与任何存储系统配合使用 63。  
  * **记忆工具 (Memory Tools)**：提供 create\_manage\_memory\_tool 和 create\_search\_memory\_tool 等工具，允许代理在对话的“热路径”中通过工具调用的方式主动管理自己的记忆 60。  
  * **后台管理器 (Background Manager)**：包含一个后台进程，可以在对话流之外自动提取、整合和更新代理的知识库 60。  
* **关键特性**：将记忆分为语义、情景和程序三种类型 22。原生利用 LangGraph 的  
  BaseStore 接口进行持久化，测试时可使用 InMemoryStore，生产环境则推荐使用 AsyncPostgresStore 等数据库支持的存储 60。

#### **2.7. OpenAI 原生记忆：集成化的平台解决方案**

* **核心哲学**：作为 ChatGPT 生态系统的一部分，OpenAI 的原生记忆功能被设计为一个对用户友好、高度集成的功能，其目标是简化使用，并将底层的复杂性完全抽象掉 64。  
* **架构**：  
  * **双重机制系统 (Dual-Mechanism System)**：其记忆功能通过两种方式实现：**已保存的记忆 (Saved Memories)**，即用户明确要求或模型自动认为重要的、需要长期记住的事实；以及**聊天历史参考 (Chat History Reference)**，即模型可以从用户的所有历史对话中隐式地获取上下文 66。  
  * **类 RAG 检索 (RAG-like Retrieval)**：聊天历史参考功能极有可能通过一个类似 RAG 的系统工作。当用户提出问题时，系统会在后台对索引过的聊天历史数据库进行语义搜索，并将最相关的对话片段注入到当前上下文中，以提供更具个性化的回答 67。  
  * **黑盒 (Black Box)**：OpenAI 并未公开其记忆功能的精确底层架构，例如使用了何种向量数据库、具体的摘要或索引策略等。对于开发者而言，它是一个功能明确但实现细节不透明的“黑盒” 5。  
* **关键特性**：记忆由用户控制（用户可以随时查看、删除或关闭记忆功能），模型能够自动更新记忆，并提供“临时聊天”（Temporary Chat）模式以进行无记忆的对话 64。

---

### **第三节：开发者体验与生态系统分析**

本节评估使用各个框架的实际体验，重点关注集成难度、可用工具、文档质量和社区支持。一个架构再优越的框架，如果开发者体验不佳，也难以在市场中获得成功。

#### **3.1. 集成便利性：快速入门对比**

* **Mem0**：以其极简的上手体验而备受赞誉。开发者只需通过 pip install mem0ai 安装，实例化 Memory 类，然后便可直接调用 .add() 和 .search() 等核心方法。整个过程代码量极少，非常直观 27。  
* **Letta (MemGPT)**：设置过程相对复杂。官方推荐使用 Docker 运行 Letta 服务器，该服务器负责代理状态的持久化管理。开发者通过一个客户端 SDK (letta-client) 连接到服务器的 REST API 进行交互。这种模式比简单的库导入要复杂，但它能够构建出更健壮、真正有状态的代理服务 68。  
* **Zep**：与 Letta 类似，Zep 也作为一个独立服务运行（提供 Zep Cloud 或自托管选项）。开发者通过相应语言的 SDK（如 Python 的 zep-cloud）连接到服务端点。设置过程主要围绕获取 API 密钥并配置客户端 30。  
* **Memobase**：同样追求简洁性。通过 pip install memobase 安装后，连接客户端，然后使用 u.insert() 和 u.flush() 等方法。其独特的异步批处理机制是开发者需要理解的一个关键点 55。  
* **LangMem**：对于已经熟悉 LangChain 生态的开发者来说，其集成体验最为原生。开发者只需在 LangGraph 代码中初始化一个存储后端（InMemoryStore 或持久化存储），然后将 LangMem 提供的记忆工具传递给代理构造函数即可 60。

#### **3.2. 工具与可观测性：超越 API 的价值**

* **Letta 的代理开发环境 (ADE)**：这是 Letta 的一个显著差异化优势。ADE 是一个图形化用户界面，允许开发者创建、部署、交互和观察代理。它能清晰地展示代理的记忆、上下文窗口和决策过程，这对于调试复杂的代理行为至关重要 43。一个正在努力理解其代理为何行为异常的开发者，会发现 ADE 提供的这种透明度能够极大地缩短调试周期，从而构成强大的竞争壁垒。  
* **Mem0 的可观测性与仪表盘**：Mem0 提供了内置的可观测性和追踪功能，开发者可以追踪每个记忆的生命周期（TTL）、大小和访问情况。其云平台还提供了一个仪表盘，用于搜索和管理所有记忆 73。  
* **Memobase 的 Inspector 和 Playground**：Memobase 提供了一个开源的 Web UI (Inspector)，其中包含用户表、用量图表和测试场。Playground 则是一个无需任何设置即可实时体验和可视化记忆演变过程的环境 71。  
* **Zep, MemOS, LangMem**：这些框架更偏向于后端，其专用的 UI 或可观测性工具尚不成熟或未公开展示。它们通常依赖于与 LangSmith 等第三方工具的集成来实现可观测性 61。

#### **3.3. 文档、社区健康度与支持**

* **Mem0**：在 GitHub 上非常活跃（36.7k 星标，3.7k 复刻），拥有大量的 issue 和 discussion，这表明其社区充满活力，但同时也可能意味着项目迭代速度快、潜在 bug 较多。社区支持渠道包括 Discord 和创始人联系方式 29。  
* **Letta (MemGPT)**：拥有强大的社区基础（17.3k 星标，1.8k 复刻）和活跃的 Discord。项目从 MemGPT 更名为 Letta，标志着其正从一个研究项目向更成熟、商业友好的产品转型 43。  
* **Zep**：社区规模较小但非常专注。主仓库拥有 3.4k 星标。社区沟通渠道包括 Discord 和一个高质量的技术博客。其战略性地弃用一体化的“社区版”，转而聚焦于 Zep Cloud 和开源核心 Graphiti，是其发展策略中的一个关键举措 30。  
* **MemOS (MemTensor)**：一个较新的项目，但关注度正在迅速增长（1.4k 星标）。拥有 Discord 和微信群，显示出其与中国研究背景的紧密联系 48。  
* **Memobase**：同样是新晋项目，但已获得良好关注（1.5k 星标）。在 Discord 和 Twitter 上很活跃，并围绕其用户画像的细分市场清晰地构建社区 71。  
* **LangMem**：作为 LangChain 生态的一部分，它受益于更广泛的社区，但其自身的专属关注度较低（数百星标）。GitHub issue 表明开发者在使用过程中遇到了一些阻力和 bug 78。

#### **3.4. 框架互操作性**

几乎所有框架都强调了它们与主流代理框架（如 LangChain、LangGraph 和 CrewAI）的集成能力，这表明互操作性是市场的一个基本要求 73。值得注意的是，Letta 和 Mem0 还特别强调了对模型上下文协议（Model Context Protocol, MCP）的支持。MCP 是一个新兴的、用于标准化工具调用的协议，这表明这些框架在设计上具有前瞻性，致力于与未来的工具生态系统保持兼容 82。

一个重要的市场趋势是，开源正在成为建立信任和分发渠道的核心战略。对于处理潜在敏感用户数据（记忆）的系统而言，开源能够建立开发者信任，因为代码是可审查的 85。此外，它也是一种强大的市场推广策略。像 Mem0 和 Letta 这样的项目通过 GitHub 获得了巨大的关注度和社区反馈，然后将这些流量引导至其商业云服务 29。Zep 弃用其一体化的“社区版”，转而采用更精细的“Graphiti”核心与云优先的模式，正是对这一战略的精炼和调整 30。

---

### **第四节：性能基准测试：一个充满争议的领域**

本节将审慎地审视各框架的性能声明，并特别关注 Mem0 与 Zep 之间的公开争议，旨在为用户提供一个中立的分析视角。

#### **4.1. 理解基准测试**

在 AI 记忆领域，两个基准测试被频繁引用：

* **LOCOMO**：这是一个包含极长对话（平均 9k-26k tokens）的数据集，旨在通过问答、摘要和对话生成等任务来评估模型的长期记忆能力。问题类型包括单跳、多跳、时序和开放域推理 86。  
* **LongMemEval**：这是一个更具挑战性的基准测试，具有更长的上下文（平均 115k tokens），并专注于评估五种核心记忆能力：信息提取、跨会话推理、知识更新、时序推理和拒答（即识别问题无法回答的能力） 88。

#### **4.2. 公开的声明与反驳**

* **Mem0 的初始声明**：在其研究论文中，Mem0 宣称在 LOCOMO 基准测试上取得了业界领先（SOTA）的性能，相较于 OpenAI 的原生记忆功能，其准确率有 26% 的相对提升，并优于 LangMem 和 Zep 11。  
* **Zep 的反驳**：Zep 随后发表了一篇题为《谎言、该死的谎言和统计数据：Mem0 真的是代理记忆的 SOTA 吗？》的博客文章 92。他们指出，Mem0 对 Zep 的评估存在严重的方法论缺陷，包括错误的用户模型实现、不恰当的时间戳处理方式以及串行而非并行的搜索执行，这些都人为地拉低了 Zep 的性能得分 92。Zep 声称，在正确实现的情况下，其框架在同一基准测试上的表现远超 Mem0。  
* **Mem0 的再反驳**：Mem0 的首席技术官在 GitHub issue 中对此作出回应，辩称他们的实现遵循了 Zep 当时可用的文档和示例代码，并指出 Zep 的反驳文章本身也存在计算错误（例如，包含了基准测试中一个本应被排除的、无法使用的类别）。在双方各自修正后，性能差距似乎有所缩小，但争议依然存在 94。  
* **MemOS 的声明**：MemTensor 团队的 MemOS 论文同样使用了 LOCOMO 基准测试，并声称其性能在所有类别中都持续领先于包括 Mem0、Zep、LangMem 和 OpenAI 在内的所有基线，尤其在多跳和时序推理等挑战性任务上优势明显 47。

这场公开的争议是本报告中最重要的发现之一。它清晰地表明，AI 记忆领域的基准测试仍处于不成熟和不可靠的阶段。双方都指责对方的方法存在缺陷，而 LOCOMO 基准测试本身也因对话长度不足、缺乏知识更新测试和数据质量问题而受到批评 92。这意味着技术领导者不能盲目相信任何一方论文中的标题数字。决策必须基于对架构原理的理解，并在可能的情况下，进行针对自身应用场景的独立测试。

#### **4.3. 关键指标：准确率、延迟与 Token 效率**

尽管具体的数值存在争议，但从各方的报告中可以清晰地看到一个经典的工程权衡：

* **准确率 (LLM-as-a-Judge Score)**：Mem0 声称在 LOCOMO 上得分 66.9%，而 OpenAI 为 52.9% 11。Zep 在修正后声称得分为 75.14% 92。MemOS 则声称全面超越所有对手 48。这些相互矛盾的数据进一步印证了基准测试的不可靠性。  
* **延迟 (Latency)**：低延迟是 Mem0 和 Zep 的共同卖点。Mem0 报告其端到端 p95 延迟为 1.44 秒，相比于将全部上下文提供给模型的暴力方法（17.12 秒），降低了 91% 11。Zep 同样声称延迟降低了 90% 34。  
* **Token 效率 (Token Efficiency)**：Mem0 声称节省了 90% 的 token 消耗 11。一篇分析文章指出，在一次测试中，Zep 因缓存完整的摘要而消耗了超过 60 万个 token，而 Mem0 仅消耗了 7 千个，这揭示了两者在架构上对成本的巨大影响 96。

数据揭示了一个一致的模式：简单的向量/键值存储（如基础版 Mem0）速度最快、成本最低，但可能难以处理复杂的关系查询。而基于图的系统（如 Zep、Mem0g、MemOS）在处理时序和多跳推理任务时表现更优，但相应地带来了更高的延迟和 token/计算成本 11。将全部上下文输入模型的方式虽然在短对话中准确率最高，但在规模化应用中，其高昂的延迟和成本是无法接受的 86。这是一个关键的权衡，需要根据业务需求来决策。一个简单的个性化聊天机器人可能更看重 Mem0 基础版的速度和低成本；而一个需要分析时序事件链的企业级欺诈检测代理，则会认为 Zep 时序图谱带来的额外成本是值得的。

---

### **第五节：部署与商业模式**

本节分析各个框架的商业化策略，包括部署方式和定价模型，这直接关系到项目的总拥有成本和可扩展性。

#### **5.1. 云服务 vs. 自托管：便利性与控制权的权衡**

* **云优先，核心开源 (Cloud-First with Open Source Core)**：这是当前市场的主流模式，被 Mem0、Zep、Letta 和 Memobase 等主要厂商采用。它们提供一个托管的、可扩展的云服务，同时将其核心技术以开源形式发布。云版本通常包含额外的增值功能，如高级分析、监控和企业级安全 30。  
* **完全自托管 (Fully Self-Hosted)**：以 LangMem 和 MemOS 为代表。它们主要是以 SDK 或框架的形式提供，设计初衷就是由开发者在自己的环境中运行。这种模式提供了最大程度的控制权和数据隐私，但需要开发者承担更多的运维开销 48。

#### **5.2. 定价模型对比分析**

* **Mem0**：提供一个免费的“业余爱好”套餐（1万条记忆，1000次检索/月），一个每月 19 美元的“入门”套餐，以及一个每月 249 美元的“专业”套餐，后者包含图记忆等高级功能。此外还提供企业方案 99。有评论指出，从入门版到专业版的跨度较大 100。  
* **Letta**：提供免费套餐（10个活跃代理，500次标准请求/月），每月 20 美元的“专业”套餐，以及每月 750 美元的“规模化”套餐。一个关键特性是，用户可以使用自己的 LLM API 密钥，从而避免计入其请求配额 97。  
* **Zep**：采用计量付费模式。每月提供免费额度（2500条消息，2.5MB 图数据），超出部分按量计费（每千条消息 1.25 美元，每 MB 数据 2.50 美元）。同时提供企业版和“自带云”（BYOC）部署选项 102。  
* **其他框架**：Memobase 和 MemOS 目前主要以开源为主，并计划推出云服务 54。LangMem 是一个免费的开源 SDK。OpenAI 的记忆功能则包含在 ChatGPT 的订阅服务（如 Plus、Pro）中 64。

这些定价结构清晰地展示了典型的产品导向增长（PLG）战略。厂商通过慷慨的免费套餐和开源版本吸引开发者，然后通过基于用量的定价和高价值的企业功能（如单点登录、服务等级协议、专属支持）实现商业变现。这种模式降低了开发者的入门门槛，同时为项目创造了清晰的收入路径，是项目商业上是否可行和能否长期维持的一个重要标志。

#### **5.3. 开源许可与社区贡献**

* 大多数主流开源框架（Mem0、Letta、Zep 的 Graphiti、MemOS、Memobase）都采用了宽松的 Apache 2.0 许可证，这极大地鼓励了商业采纳和二次开发 29。  
* 开源项目的健康度（如 GitHub 星标数、复刻数、贡献者活跃度）是衡量项目发展势头和社区支持可用性的一个有力指标 30。

值得注意的是，Zep 提供的“自带云”（BYOC）部署选项 102 是对企业数据隐私担忧的直接回应。对于金融、医疗等受严格监管的行业，将敏感的用户对话数据发送到第三方 SaaS 平台往往是不可接受的。BYOC 模式允许企业在自己的云环境（AWS、GCP、Azure）中部署 Zep 的托管软件，确保数据永远不会离开其安全边界。这是赢得大型企业客户的关键功能。

---

### **第六节：战略选型框架**

本节综合前述所有分析，为技术决策者提供一个清晰、可操作的选择指南。

#### **6.1. 全面比较表**

为了提供一个密集的、一目了然的概览，下表总结了所有被分析框架在关键决策维度上的表现。这张表格是本报告中最具价值的产出之一，它将数万字的分析浓缩成一个可直接比较的格式，使技术领导者能够快速评估和权衡。通过此表，决策者可以清晰地看到复杂的权衡关系，例如，MemOS 拥有高度先进的统一架构，但其社区规模和工具链成熟度低于 Letta；或者 Memobase 在用户画像这一细分领域极具价值，但其通用记忆能力不如 Mem0。

| 框架 | 核心架构 | 关键差异点 | 支持的记忆类型 | 开发者工具 | 性能声明 (附注) | 部署模式 | 定价模型 | 理想用例 |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **Mem0** | 混合数据存储 (向量+图+KV) | 两阶段流水线 (提取/更新)，在性能、成本和功能间取得平衡 | 情景、语义、用户/会话/代理状态 29 | API/SDKs (Python, JS), 可观测性仪表盘, CLI 73 | 在 LOCOMO 上表现均衡，延迟低，token 效率高 11 | 云服务 & 自托管 (开源) | 免费/分级订阅 ($19-$249+/月) 99 | 需要平衡性能与成本的通用生产级代理，如智能客服、AI 助手。 |
| **Zep** | 时序知识图谱 (Graphiti 引擎) | 双时序数据模型，强大的时序和因果关系推理能力 | 情景、语义、实体、社区 34 | API/SDKs (Python, JS, Go), LangChain/LlamaIndex 集成 30 | 在 LongMemEval 上表现出色，尤其擅长时序推理 95 | 云服务 & 自托管 (Graphiti) & BYOC | 免费额度 \+ 计量付费 102 | 复杂的企业系统，如客户关系管理、金融风控、需要追踪状态随时间变化的场景。 |
| **Letta** | LLM 即操作系统 (MemGPT) | 代理通过工具调用自我编辑记忆，提供虚拟无限上下文 | 核心记忆 (Persona/Human), 外部记忆 (Archival/Recall) 41 | 代理开发环境 (ADE), API/SDKs (Python, JS), CLI 43 | 在 LOCOMO 上表现一般，更注重架构的灵活性 86 | 云服务 & 自托管 (开源) | 免费/分级订阅 ($20-$750+/月) 97 | 需要深度定制和高透明度的代理，研究项目，多代理系统，复杂代理行为调试。 |
| **MemOS** | 统一的操作系统抽象 | MemCube 统一封装明文、激活和参数记忆，具备系统级调度能力 | 明文、激活、参数 48 | API/SDKs, 命令行工具 48 | 在 LOCOMO 上声称全面领先，尤其在多跳和时序任务上 48 | 自托管 (开源) | 免费 (开源)，计划推出云服务 | 前沿研究，探索未来 AI 架构，不适合立即投入生产。 |
| **Memobase** | 用户画像模型 | 专注为“用户”而非“代理”构建记忆，非嵌入式架构，异步批处理 | 用户画像、事件记忆 59 | API/SDKs (Python, Go, JS), Playground, Inspector UI 71 | 在真实聊天数据上生成高质量用户画像 55 | 云服务 & 自托管 (开源) | 免费 (开源)，提供云服务 | 用户中心型应用，如 AI 伴侣、个性化教育、角色扮演游戏。 |
| **LangMem** | 集成式 SDK | 深度集成 LangGraph，提供与存储后端解耦的记忆操作原语 | 情景、语义、程序 22 | Python SDK, 作为 LangGraph 工具集的一部分 61 | 在 LOCOMO 上表现中等，但延迟较高 104 | 自托管 (开源库) | 免费 (开源) | 已深度使用 LangChain/LangGraph 生态系统的项目，需要灵活构建自定义记忆逻辑。 |
| **OpenAI** | 集成平台功能 (黑盒) | 无缝集成于 ChatGPT 生态，对用户和开发者完全抽象底层实现 | 已保存的记忆 (Saved Memories), 聊天历史参考 (Chat History) 66 | 通过 ChatGPT 界面或 API 隐式使用 | 在 LOCOMO 上表现为基线水平，简单任务尚可，复杂推理较弱 86 | 集成于 OpenAI 产品中 | 包含在 ChatGPT 订阅中 | 简单的、轻量级的个性化需求，且应用场景完全在 OpenAI 生态内。 |

#### **6.2. 基于用例的决策指南**

以下是基于常见项目原型，为技术决策者提供的叙述性建议：

* **场景一：快速原型开发与用户中心型个性化**  
  * **推荐框架：Memobase**  
  * **理由**：Memobase 对用户画像的极致专注、简洁的 API 和非嵌入式架构，使其成为快速构建 AI 伴侣、个性化导师或角色扮演游戏等应用的最佳选择。在这些场景中，深刻理解“用户是谁”比单纯记录“对话内容”更为重要。其异步处理机制也确保了前端交互的流畅性 55。  
* **场景二：寻求速度与准确率平衡的生产级 SaaS 应用**  
  * **推荐框架：Mem0**  
  * **理由**：Mem0 务实的混合架构、在通用基准测试上的均衡表现以及成熟的云服务，使其成为广泛应用的可靠选择。对于大多数需要可靠记忆功能但又不希望引入时序图谱等复杂性的标准应用（如智能客服、通用 AI 助手），Mem0 提供了一个高性价比的解决方案 73。  
* **场景三：具有复杂时序推理需求的企业级系统**  
  * **推荐框架：Zep**  
  * **理由**：Zep 的时序知识图谱是为需要追踪状态随时间变化的关键任务而生。例如，在客户支持场景中追踪一个问题的完整演变历史，或在金融领域分析一系列交易的因果关系。其对 SOC 2、HIPAA 等企业级合规性的支持，以及 BYOC 部署选项，使其成为受监管行业的有力竞争者 10。  
* **场景四：需要深度定制和前沿代理行为研究**  
  * **推荐框架：Letta (MemGPT)**  
  * **理由**：其“LLM 即操作系统”的架构提供了极高的“白盒”透明度，为研究人员和希望构建新颖代理架构的开发者提供了最大程度的控制。其独特的 ADE 工具为调试复杂的代理行为提供了无与伦g比的可视性，是学术研究和 R\&D 团队的理想选择 44。  
* **场景五：深度集成于 LangChain 生态系统**  
  * **推荐框架：LangMem**  
  * **理由**：对于已经大量投入 LangChain 和 LangGraph 生态的团队而言，LangMem 提供了最无缝、最原生的集成路径。它将记忆视为 LangChain 工具箱中的又一个工具，允许开发者用熟悉的方式灵活地构建自定义记忆逻辑 60。  
* **场景六：探索尖端架构的前沿研究**  
  * **推荐框架：MemOS (MemTensor)**  
  * **理由**：MemOS 将明文、激活和参数记忆统一到“记忆立方体”这一抽象中的宏大愿景，代表了 AI 架构的研究前沿。它最适合那些探索 AI 未来架构的学术或 R\&D 团队，而非寻求即时生产部署的工程团队 45。  
* **场景七：追求 OpenAI 生态内的极简实现**  
  * **推荐框架：OpenAI 原生记忆**  
  * **理由**：对于直接构建在 ChatGPT 之上或简单的 API 封装应用，如果首要目标是易用性，且不需要深度控制，那么 OpenAI 的原生记忆功能是阻力最小的选择。它免去了集成第三方服务的复杂性 64。

#### **6.3. 最终建议**

选择 AI 记忆框架是一项关键的架构决策。任何单一框架都无法在所有维度上取得胜利。最终的决定应基于对项目核心需求的清晰认知。我们建议技术领导者在决策前回答以下问题：

1. **核心需求**：我们的应用最需要的是什么？是复杂的时序推理、深度的用户画像，还是快速的语义检索？  
2. **团队专长**：我们的团队更适应一个开箱即用的云服务，还是一个需要深度配置和运维的开源框架？  
3. **上市时间**：我们是追求快速推向市场，还是有时间和资源进行深度研发和定制？  
4. **预算**：我们对 token 消耗、API 调用和托管服务的成本有多敏感？

将这些问题的答案与本报告中各框架的特性进行匹配，将引导您走向最符合自身战略目标的正确选择。

---

### **第七节：AI 记忆的未来**

AI 记忆层正在从一系列学术性的“技巧”迅速演变为一个稳定、商业化的基础设施类别 46。展望未来，几个关键趋势将塑造该领域的下一阶段发展。

#### **7.1. 新兴趋势**

* **多代理记忆 (Multi-Agent Memory)**：随着多代理系统的兴起，如何让多个代理共享记忆或维持独立的记忆空间成为一个核心挑战。Letta 提供了跨代理通信的工具 106，Zep 提出了“群组图谱”（Group Graphs）的概念 107，但这仍然是一个复杂且有待深入探索的新兴领域 108。  
* **通过 MCP 实现标准化 (Standardization via MCP)**：Letta 和 Mem0 等框架对模型上下文协议（MCP）的采纳，预示着一个工具和记忆系统可以互操作的未来。标准化将降低集成成本，促进生态繁荣 82。  
* **端侧与联邦记忆 (On-Device and Federated Memory)**：出于对数据隐私和低延迟的追求，未来的研究将更多地转向在用户设备上直接运行的记忆方案，以及能够在不暴露原始数据的情况下整合经验的联邦学习记忆系统 46。  
* **超越文本 (Beyond Text)**：当前记忆框架主要处理文本数据。随着多模态大模型的发展，如何有效存储、索引和检索图像、音频等多模态记忆，将成为下一个重要的研究方向 76。

#### **7.2. 结语**

AI 记忆层的出现，标志着智能系统开发理念的一次深刻变革。它不再是可有可无的附加功能，而是构建能够真正学习、适应和与用户建立长期关系的 AI 应用的核心基石。对于任何旨在构建超越简单问答的、具有持续价值的 AI 产品的团队而言，理解并审慎选择一个合适的记忆框架，将是通往成功的关键一步。

---

### **附录：澄清“MemOS”的混淆**

为了避免用户在调研过程中产生困惑，本附录旨在明确区分市场上几个名称相似但实质完全不同的“MemOS”项目。

* **MemTensor/MemOS**：这是本报告中“MemOS”分析的主要对象。它是一个由上海交通大学等机构的研究人员发起的、研究驱动的开源项目。其目标是构建一个真正的“AI 记忆操作系统”，核心概念包括三层架构和统一的“记忆立方体”（MemCube）抽象 45。  
* **BAI-LAB/MemoryOS**：这是一个来自北京邮电大学的独立研究项目。它同样借鉴了操作系统的理念，设计了一个分层存储架构（短期、中期、长期记忆），但在实现上与 MemTensor 的项目是不同的 111。  
* **usememos/memos**：这是一个非常受欢迎（GitHub 42.7k 星标）的开源、自托管的**笔记和知识管理平台**。尽管名称相似，但它**不是**一个 AI 代理记忆框架，其功能与本报告讨论的其他框架完全不同。为避免混淆，不应将其纳入 AI 记忆框架的比较范畴 113。

#### **Works cited**

1. AI Memory Layer: Top Platforms and Approaches, accessed July 13, 2025, [https://arize.com/ai-memory/](https://arize.com/ai-memory/)  
2. arize.com, accessed July 13, 2025, [https://arize.com/ai-memory/\#:\~:text=AI%20memory%20is%20the%20emerging,time%20rather%20than%20just%20react.](https://arize.com/ai-memory/#:~:text=AI%20memory%20is%20the%20emerging,time%20rather%20than%20just%20react.)  
3. LLM Memory: Integration of Cognitive Architectures with AI \- Cognee, accessed July 13, 2025, [https://www.cognee.ai/blog/fundamentals/llm-memory-cognitive-architectures-with-ai](https://www.cognee.ai/blog/fundamentals/llm-memory-cognitive-architectures-with-ai)  
4. Adding memory to LLMs with Letta \- Terse Systems, accessed July 13, 2025, [https://tersesystems.com/blog/2025/02/14/adding-memory-to-llms-with-letta/](https://tersesystems.com/blog/2025/02/14/adding-memory-to-llms-with-letta/)  
5. What's Letta ai? A complete guide | by Aaryan Kansari \- Medium, accessed July 13, 2025, [https://medium.com/@pbzbhzxk/whats-letta-ai-a-complete-guide-230d572a6fd2](https://medium.com/@pbzbhzxk/whats-letta-ai-a-complete-guide-230d572a6fd2)  
6. How Mem0 Lets LLMs Remember Everything Without Slowing Down \- Apidog, accessed July 12, 2025, [https://apidog.com/blog/mem0-memory-llm-agents/](https://apidog.com/blog/mem0-memory-llm-agents/)  
7. Episodic memory in ai agents poses risks that should be studied and mitigated \- arXiv, accessed July 12, 2025, [https://arxiv.org/html/2501.11739v1?ref=community.heartcount.io](https://arxiv.org/html/2501.11739v1?ref=community.heartcount.io)  
8. LLM4LLM: Longer-Lasting Memory for LLMs | UC Berkeley School of Information, accessed July 13, 2025, [https://www.ischool.berkeley.edu/projects/2024/llm4llm-longer-lasting-memory-llms](https://www.ischool.berkeley.edu/projects/2024/llm4llm-longer-lasting-memory-llms)  
9. MemOS: A Memory OS for AI System : r/LocalLLaMA \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/LocalLLaMA/comments/1lv9m3j/memos\_a\_memory\_os\_for\_ai\_system/](https://www.reddit.com/r/LocalLLaMA/comments/1lv9m3j/memos_a_memory_os_for_ai_system/)  
10. Stop Using RAG for Agent Memory \- Zep, accessed July 12, 2025, [https://blog.getzep.com/stop-using-rag-for-agent-memory/](https://blog.getzep.com/stop-using-rag-for-agent-memory/)  
11. Scalable Long-Term Memory for Production AI Agents \- Mem0, accessed July 13, 2025, [https://mem0.ai/research](https://mem0.ai/research)  
12. An interview with a researcher behind MemGPT, the system enabling AI companions to have infinite memory | by Chase Roberts | Vertex Ventures US | Medium, accessed July 12, 2025, [https://medium.com/vvus/an-interview-with-a-researcher-behind-memgpt-the-system-enabling-ai-companions-to-have-infinite-e23ffaa1df8a](https://medium.com/vvus/an-interview-with-a-researcher-behind-memgpt-the-system-enabling-ai-companions-to-have-infinite-e23ffaa1df8a)  
13. What is AI Memory? \- TechSee, accessed July 13, 2025, [https://techsee.com/glossary/ai-memory/](https://techsee.com/glossary/ai-memory/)  
14. What Is AI Agent Memory? | IBM, accessed July 12, 2025, [https://www.ibm.com/think/topics/ai-agent-memory](https://www.ibm.com/think/topics/ai-agent-memory)  
15. The Rise of AI Memory: How Prompt Engineering Can Influence Learning \- Arsturn, accessed July 12, 2025, [https://www.arsturn.com/blog/the-rise-of-ai-memory-how-prompt-engineering-can-influence-learning](https://www.arsturn.com/blog/the-rise-of-ai-memory-how-prompt-engineering-can-influence-learning)  
16. How AI Agents Store, Forget, and Retrieve: A Fresh Look at Memory Operations for the Next-Gen LLMs \- UBOS.tech, accessed July 12, 2025, [https://ubos.tech/news/how-ai-agents-store-forget-and-retrieve-a-fresh-look-at-memory-operations-for-the-next-gen-llms/](https://ubos.tech/news/how-ai-agents-store-forget-and-retrieve-a-fresh-look-at-memory-operations-for-the-next-gen-llms/)  
17. \[2504.15965\] From Human Memory to AI Memory: A Survey on Memory Mechanisms in the Era of LLMs \- arXiv, accessed July 13, 2025, [https://arxiv.org/abs/2504.15965](https://arxiv.org/abs/2504.15965)  
18. AI Memory: How Smart Assistants Learn and Retain Data \- Tanka, accessed July 13, 2025, [https://www.tanka.ai/blog/posts/ai-memory](https://www.tanka.ai/blog/posts/ai-memory)  
19. LLM agents: The ultimate guide 2025 | SuperAnnotate, accessed July 12, 2025, [https://www.superannotate.com/blog/llm-agents](https://www.superannotate.com/blog/llm-agents)  
20. The Importance of AI System Memory \- DZone, accessed July 12, 2025, [https://dzone.com/articles/importance-of-ai-system-memory](https://dzone.com/articles/importance-of-ai-system-memory)  
21. \[2501.13121\] Episodic Memories Generation and Evaluation Benchmark for Large Language Models \- arXiv, accessed July 12, 2025, [https://arxiv.org/abs/2501.13121](https://arxiv.org/abs/2501.13121)  
22. Long-term Memory in LLM Applications, accessed July 13, 2025, [https://langchain-ai.github.io/langmem/concepts/conceptual\_guide/](https://langchain-ai.github.io/langmem/concepts/conceptual_guide/)  
23. Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory \- arXiv, accessed July 13, 2025, [https://arxiv.org/html/2504.19413v1](https://arxiv.org/html/2504.19413v1)  
24. Mem0: Building Production-Ready AI Agents with \- arXiv, accessed July 13, 2025, [https://arxiv.org/pdf/2504.19413](https://arxiv.org/pdf/2504.19413)  
25. Mem0 launches: Open Source Memory Layer for AI Apps \- Fondo, accessed July 12, 2025, [https://www.tryfondo.com/blog/mem0-launches](https://www.tryfondo.com/blog/mem0-launches)  
26. Mem0: The Memory layer for your AI apps | Y Combinator, accessed July 12, 2025, [https://www.ycombinator.com/companies/mem0](https://www.ycombinator.com/companies/mem0)  
27. markmbain/mem0ai-mem0: The memory layer for Personalized AI \- GitHub, accessed July 13, 2025, [https://github.com/markmbain/mem0ai-mem0](https://github.com/markmbain/mem0ai-mem0)  
28. AI Memory Management System: Introduction to mem0 | by PI | Neural Engineer | Medium, accessed July 13, 2025, [https://medium.com/neural-engineer/ai-memory-management-system-introduction-to-mem0-af3c94b32951](https://medium.com/neural-engineer/ai-memory-management-system-introduction-to-mem0-af3c94b32951)  
29. GitHub \- mem0ai/mem0: Memory for AI Agents; Announcing OpenMemory MCP, accessed July 13, 2025, [https://github.com/mem0ai/mem0](https://github.com/mem0ai/mem0)  
30. getzep/zep: Zep | Examples, Integrations, & More \- GitHub, accessed July 13, 2025, [https://github.com/getzep/zep](https://github.com/getzep/zep)  
31. Zep: Context Engineering Platform for AI Agents, accessed July 12, 2025, [https://www.getzep.com/](https://www.getzep.com/)  
32. getzep/graphiti: Build Real-Time Knowledge Graphs for AI ... \- GitHub, accessed July 12, 2025, [https://github.com/getzep/graphiti](https://github.com/getzep/graphiti)  
33. Beyond Static Graphs: Engineering Evolving Relationships \- Zep, accessed July 12, 2025, [https://blog.getzep.com/beyond-static-knowledge-graphs/](https://blog.getzep.com/beyond-static-knowledge-graphs/)  
34. ZEP:ATEMPORAL KNOWLEDGE GRAPH ARCHITECTURE FOR AGENT MEMORY, accessed July 12, 2025, [https://blog.getzep.com/content/files/2025/01/ZEP\_\_USING\_KNOWLEDGE\_GRAPHS\_TO\_POWER\_LLM\_AGENT\_MEMORY\_2025011700.pdf](https://blog.getzep.com/content/files/2025/01/ZEP__USING_KNOWLEDGE_GRAPHS_TO_POWER_LLM_AGENT_MEMORY_2025011700.pdf)  
35. Zep: A Temporal Knowledge Graph Architecture for Agent Memory \- ResearchGate, accessed July 13, 2025, [https://www.researchgate.net/publication/388402077\_Zep\_A\_Temporal\_Knowledge\_Graph\_Architecture\_for\_Agent\_Memory](https://www.researchgate.net/publication/388402077_Zep_A_Temporal_Knowledge_Graph_Architecture_for_Agent_Memory)  
36. zep:atemporal knowledge graph architecture for agent memory \- arXiv, accessed July 13, 2025, [https://arxiv.org/pdf/2501.13956](https://arxiv.org/pdf/2501.13956)  
37. \[Literature Review\] Zep: A Temporal Knowledge Graph Architecture for Agent Memory, accessed July 13, 2025, [https://www.themoonlight.io/en/review/zep-a-temporal-knowledge-graph-architecture-for-agent-memory](https://www.themoonlight.io/en/review/zep-a-temporal-knowledge-graph-architecture-for-agent-memory)  
38. Letta: Home, accessed July 13, 2025, [https://docs.letta.com/](https://docs.letta.com/)  
39. LLMs as Operating Systems: Agent Memory \- DeepLearning.AI, accessed July 13, 2025, [https://www.deeplearning.ai/short-courses/llms-as-operating-systems-agent-memory/](https://www.deeplearning.ai/short-courses/llms-as-operating-systems-agent-memory/)  
40. MemGPT \- Letta, accessed July 13, 2025, [https://docs.letta.com/concepts/memgpt](https://docs.letta.com/concepts/memgpt)  
41. Agent Memory | Letta, accessed July 13, 2025, [https://docs.letta.com/guides/agents/memory](https://docs.letta.com/guides/agents/memory)  
42. letta/examples/docs/example.py at main · letta-ai/letta \- GitHub, accessed July 12, 2025, [https://github.com/letta-ai/letta/blob/main/examples/docs/example.py](https://github.com/letta-ai/letta/blob/main/examples/docs/example.py)  
43. Letta (formerly MemGPT) is the stateful agents framework with memory, reasoning, and context management. \- GitHub, accessed July 13, 2025, [https://github.com/letta-ai/letta](https://github.com/letta-ai/letta)  
44. Letta Overview, accessed July 13, 2025, [https://docs.letta.com/overview](https://docs.letta.com/overview)  
45. Paper page \- MemOS: A Memory OS for AI System \- Hugging Face, accessed July 13, 2025, [https://huggingface.co/papers/2507.03724](https://huggingface.co/papers/2507.03724)  
46. Chinese researchers unveil MemOS, the first 'memory operating system' that gives AI human-like recall : r/singularity \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/singularity/comments/1lvg6ea/chinese\_researchers\_unveil\_memos\_the\_first\_memory/](https://www.reddit.com/r/singularity/comments/1lvg6ea/chinese_researchers_unveil_memos_the_first_memory/)  
47. \\titlefontMemOS: A Memory OS for AI System \- arXiv, accessed July 13, 2025, [https://arxiv.org/html/2507.03724v1](https://arxiv.org/html/2507.03724v1)  
48. MemTensor/MemOS: MemOS (Preview) | Intelligence Begins with Memory \- GitHub, accessed July 13, 2025, [https://github.com/MemTensor/MemOS](https://github.com/MemTensor/MemOS)  
49. MemOS: An Operating System for Memory-Augmented Generation (MAG) in Large Language Models (Short Version) \- arXiv, accessed July 12, 2025, [https://arxiv.org/html/2505.22101v1](https://arxiv.org/html/2505.22101v1)  
50. (PDF) MemOS: A Memory OS for AI System \- ResearchGate, accessed July 13, 2025, [https://www.researchgate.net/publication/393476865\_MemOS\_A\_Memory\_OS\_for\_AI\_System](https://www.researchgate.net/publication/393476865_MemOS_A_Memory_OS_for_AI_System)  
51. MemOS: A Memory-Centric Operating System for Evolving and Adaptive Large Language Models \- MarkTechPost, accessed July 13, 2025, [https://www.marktechpost.com/2025/06/14/memos-a-memory-centric-operating-system-for-evolving-and-adaptive-large-language-models/](https://www.marktechpost.com/2025/06/14/memos-a-memory-centric-operating-system-for-evolving-and-adaptive-large-language-models/)  
52. Chinese researchers unveil MemOS, the first 'memory operating system' that gives AI human-like recall \- Blog \- iStart Valley, accessed July 12, 2025, [https://www.istartvalley.org/blog/chinese-researchers-unveil-memos-the-first-memory-operating-system-that-gives-ai-human-like-recall](https://www.istartvalley.org/blog/chinese-researchers-unveil-memos-the-first-memory-operating-system-that-gives-ai-human-like-recall)  
53. (PDF) MemOS: An Operating System for Memory-Augmented Generation (MAG) in Large Language Models \- ResearchGate, accessed July 13, 2025, [https://www.researchgate.net/publication/392167750\_MemOS\_An\_Operating\_System\_for\_Memory-Augmented\_Generation\_MAG\_in\_Large\_Language\_Models](https://www.researchgate.net/publication/392167750_MemOS_An_Operating_System_for_Memory-Augmented_Generation_MAG_in_Large_Language_Models)  
54. INTELLIGENCE BEGINS WITH MEMORY, accessed July 13, 2025, [https://memos.openmem.net/](https://memos.openmem.net/)  
55. readme.md \- memodb-io/memobase \- GitHub, accessed July 13, 2025, [https://github.com/memodb-io/memobase/blob/main/readme.md](https://github.com/memodb-io/memobase/blob/main/readme.md)  
56. Memobase, accessed July 13, 2025, [https://www.memobase.cn/](https://www.memobase.cn/)  
57. Exploring global user modeling as a missing memory layer in toC AI Apps \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/LLMDevs/comments/1lr7orp/exploring\_global\_user\_modeling\_as\_a\_missing/](https://www.reddit.com/r/LLMDevs/comments/1lr7orp/exploring_global_user_modeling_as_a_missing/)  
58. Tips to Use Memobase, accessed July 13, 2025, [https://docs.memobase.io/practices/tips](https://docs.memobase.io/practices/tips)  
59. What is Memobase? \- Memobase, accessed July 13, 2025, [https://docs.memobase.io/introduction](https://docs.memobase.io/introduction)  
60. LangMem: Long-Term Memory for AI Agents | by Astropomeai | Medium, accessed July 13, 2025, [https://medium.com/@astropomeai/langmem-long-term-memory-for-ai-agents-366d7256ddce](https://medium.com/@astropomeai/langmem-long-term-memory-for-ai-agents-366d7256ddce)  
61. langchain-ai/langmem \- GitHub, accessed July 13, 2025, [https://github.com/langchain-ai/langmem](https://github.com/langchain-ai/langmem)  
62. LangMem, accessed July 13, 2025, [https://langchain-ai.github.io/langmem/](https://langchain-ai.github.io/langmem/)  
63. Understanding LangMem's Long-Term Memory: Overview and Usage | Mamezou Developer Portal \- 豆蔵デベロッパーサイト, accessed July 13, 2025, [https://developer.mamezou-tech.com/en/blogs/2025/02/26/langmem-intro/](https://developer.mamezou-tech.com/en/blogs/2025/02/26/langmem-intro/)  
64. Memory FAQ \- OpenAI Help Center, accessed July 13, 2025, [https://help.openai.com/en/articles/8590148-memory-faq](https://help.openai.com/en/articles/8590148-memory-faq)  
65. ChatGPT now has Memory, remembering everything you've said: Here's how to use it, accessed July 13, 2025, [https://etedge-insights.com/technology/artificial-intelligence/chatgpt-now-has-memory-remembering-everything-youve-said-heres-how-to-use-it/](https://etedge-insights.com/technology/artificial-intelligence/chatgpt-now-has-memory-remembering-everything-youve-said-heres-how-to-use-it/)  
66. What is Memory? | OpenAI Help Center, accessed July 13, 2025, [https://help.openai.com/en/articles/8983136-what-is-memory](https://help.openai.com/en/articles/8983136-what-is-memory)  
67. ELI5: How does ChatGPT's memory actually work behind the scenes? : r/OpenAI \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/OpenAI/comments/1jy3e6z/eli5\_how\_does\_chatgpts\_memory\_actually\_work/](https://www.reddit.com/r/OpenAI/comments/1jy3e6z/eli5_how_does_chatgpts_memory_actually_work/)  
68. Improving User Experiences with Memory Export \- Mem0, accessed July 12, 2025, [https://mem0.ai/blog/improving-user-experiences-with-memory-export/](https://mem0.ai/blog/improving-user-experiences-with-memory-export/)  
69. Developer quickstart | Letta, accessed July 12, 2025, [https://docs.letta.com/quickstart](https://docs.letta.com/quickstart)  
70. getzep/zep-go: Zep: Long-Term Memory for ‍AI Assistants (Go SDK) \- GitHub, accessed July 12, 2025, [https://github.com/getzep/zep-go](https://github.com/getzep/zep-go)  
71. memodb-io/memobase: Profile-Based Long-Term Memory for AI Applications. Memobase handles user profiles, memory events, and evolving context — perfect for chatbots, companions, tutors, customer service bots, and all chat-based agents. \- GitHub, accessed July 13, 2025, [https://github.com/memodb-io/memobase](https://github.com/memodb-io/memobase)  
72. Letta AI, accessed July 13, 2025, [https://www.letta.com/](https://www.letta.com/)  
73. Mem0 \- The Memory Layer for your AI Apps, accessed July 12, 2025, [https://mem0.ai/](https://mem0.ai/)  
74. LLM Observability Tools: 2025 Comparison \- lakeFS, accessed July 12, 2025, [https://lakefs.io/blog/llm-observability-tools/](https://lakefs.io/blog/llm-observability-tools/)  
75. Discussions \- mem0ai mem0 \- GitHub, accessed July 12, 2025, [https://github.com/mem0ai/mem0/discussions](https://github.com/mem0ai/mem0/discussions)  
76. Issues · mem0ai/mem0 \- GitHub, accessed July 12, 2025, [https://github.com/mem0ai/mem0/issues](https://github.com/mem0ai/mem0/issues)  
77. Letta \- GitHub, accessed July 12, 2025, [https://github.com/letta-ai](https://github.com/letta-ai)  
78. Discussions \- langchain-ai langmem \- GitHub, accessed July 13, 2025, [https://github.com/langchain-ai/langmem/discussions](https://github.com/langchain-ai/langmem/discussions)  
79. Issues · langchain-ai/langmem \- GitHub, accessed July 13, 2025, [https://github.com/langchain-ai/langmem/issues](https://github.com/langchain-ai/langmem/issues)  
80. SyntaxError: invalid syntax in action\_type \= typing.Literal\[\*actions\_permitted\] · Issue \#9 · langchain-ai/langmem \- GitHub, accessed July 13, 2025, [https://github.com/langchain-ai/langmem/issues/9](https://github.com/langchain-ai/langmem/issues/9)  
81. A Tour of Popular Open Source Frameworks for LLM-Powered Agents | by Loic Vanel Tabueu Tagne | data from the trenches | Medium, accessed July 12, 2025, [https://medium.com/data-from-the-trenches/a-tour-of-popular-open-source-frameworks-for-llm-powered-agents-1cbd9958d227](https://medium.com/data-from-the-trenches/a-tour-of-popular-open-source-frameworks-for-llm-powered-agents-1cbd9958d227)  
82. Mem0 Blog, accessed July 12, 2025, [https://mem0.ai/blog/](https://mem0.ai/blog/)  
83. mem0ai · GitHub Topics, accessed July 12, 2025, [https://github.com/topics/mem0ai](https://github.com/topics/mem0ai)  
84. I made a turnkey Letta search agent with Open WebUI frontend : r/Letta\_AI \- Reddit, accessed July 12, 2025, [https://www.reddit.com/r/Letta\_AI/comments/1jsxmzp/i\_made\_a\_turnkey\_letta\_search\_agent\_with\_open/](https://www.reddit.com/r/Letta_AI/comments/1jsxmzp/i_made_a_turnkey_letta_search_agent_with_open/)  
85. We forked Mem0 a month ago to create a persistent memory for LLMs. Today, we have 300 users, paying customers, and are the most popular fork. Here's what we've learned. : r/selfhosted \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/selfhosted/comments/1li7mvy/we\_forked\_mem0\_a\_month\_ago\_to\_create\_a\_persistent/](https://www.reddit.com/r/selfhosted/comments/1li7mvy/we_forked_mem0_a_month_ago_to_create_a_persistent/)  
86. Benchmarking AI Agent Memory Providers for Long-Term Memory : r/LocalLLaMA \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/LocalLLaMA/comments/1kavtwr/benchmarking\_ai\_agent\_memory\_providers\_for/](https://www.reddit.com/r/LocalLLaMA/comments/1kavtwr/benchmarking_ai_agent_memory_providers_for/)  
87. \[2402.17753\] Evaluating Very Long-Term Conversational Memory of LLM Agents \- arXiv, accessed July 13, 2025, [https://arxiv.org/abs/2402.17753](https://arxiv.org/abs/2402.17753)  
88. \[2410.10813\] LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory \- arXiv, accessed July 13, 2025, [https://arxiv.org/abs/2410.10813](https://arxiv.org/abs/2410.10813)  
89. LongMemEval: Benchmarking Chat Assist- ants on Long-Term Interactive Memory \- arXiv, accessed July 13, 2025, [https://arxiv.org/html/2410.10813](https://arxiv.org/html/2410.10813)  
90. LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory \- Di Wu, accessed July 13, 2025, [https://xiaowu0162.github.io/long-mem-eval/](https://xiaowu0162.github.io/long-mem-eval/)  
91. Paper page \- Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory \- Hugging Face, accessed July 13, 2025, [https://huggingface.co/papers/2504.19413](https://huggingface.co/papers/2504.19413)  
92. Lies, Damn Lies, & Statistics: Is Mem0 Really SOTA in Agent Memory? \- Zep, accessed July 13, 2025, [https://blog.getzep.com/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/](https://blog.getzep.com/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/)  
93. Lies, Damn Lies, & Statistics: Is Mem0 Really SOTA in Agent Memory? \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/LangChain/comments/1kg5qas/lies\_damn\_lies\_statistics\_is\_mem0\_really\_sota\_in/](https://www.reddit.com/r/LangChain/comments/1kg5qas/lies_damn_lies_statistics_is_mem0_really_sota_in/)  
94. Revisiting Zep's 84% LoCoMo Claim: Corrected Evaluation & 58.44% Accuracy \#5 \- GitHub, accessed July 13, 2025, [https://github.com/getzep/zep-papers/issues/5](https://github.com/getzep/zep-papers/issues/5)  
95. A Temporal Knowledge Graph Architecture for Agent Memory \- Zep, accessed July 13, 2025, [https://blog.getzep.com/zep-a-temporal-knowledge-graph-architecture-for-agent-memory/](https://blog.getzep.com/zep-a-temporal-knowledge-graph-architecture-for-agent-memory/)  
96. Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory \- Medium, accessed July 13, 2025, [https://medium.com/@EleventhHourEnthusiast/mem0-building-production-ready-ai-agents-with-scalable-long-term-memory-9c534cd39264](https://medium.com/@EleventhHourEnthusiast/mem0-building-production-ready-ai-agents-with-scalable-long-term-memory-9c534cd39264)  
97. Pricing \- Letta, accessed July 12, 2025, [https://www.letta.com/pricing](https://www.letta.com/pricing)  
98. Memobase: Scalable User Profile-Based Memory for GenAI Applications \- Sanssapien, accessed July 13, 2025, [https://www.sanssapien.com/tool/memobase](https://www.sanssapien.com/tool/memobase)  
99. The Memory Layer for your AI Apps \- Mem0, accessed July 12, 2025, [https://mem0.ai/pricing](https://mem0.ai/pricing)  
100. From Beta to Battle‑Tested: Picking Between Letta, Mem0 & Zep for AI Memory | by Calvin Ku | Asymptotic Spaghetti Integration | Medium, accessed July 13, 2025, [https://medium.com/asymptotic-spaghetti-integration/from-beta-to-battle-tested-picking-between-letta-mem0-zep-for-ai-memory-6850ca8703d1](https://medium.com/asymptotic-spaghetti-integration/from-beta-to-battle-tested-picking-between-letta-mem0-zep-for-ai-memory-6850ca8703d1)  
101. Plans & Pricing \- Letta, accessed July 12, 2025, [https://docs.letta.com/guides/cloud/plans](https://docs.letta.com/guides/cloud/plans)  
102. Pricing \- Zep, accessed July 12, 2025, [https://www.getzep.com/pricing/](https://www.getzep.com/pricing/)  
103. Zep: A Temporal Knowledge Graph Architecture for Agent Memory \- arXiv, accessed July 13, 2025, [https://arxiv.org/html/2501.13956v1](https://arxiv.org/html/2501.13956v1)  
104. Benchmarked OpenAI Memory vs LangMem vs MemGPT vs Mem0 for Long-Term Memory \- Here's How They Stacked Up, accessed July 13, 2025, [https://mem0.ai/blog/ai-agent-memory-benchmark/](https://mem0.ai/blog/ai-agent-memory-benchmark/)  
105. I Benchmarked OpenAI Memory vs LangMem vs Letta (MemGPT) vs Mem0 for Long-Term Memory: Here's How They Stacked Up : r/LangChain \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/LangChain/comments/1kash7b/i\_benchmarked\_openai\_memory\_vs\_langmem\_vs\_letta/](https://www.reddit.com/r/LangChain/comments/1kash7b/i_benchmarked_openai_memory_vs_langmem_vs_letta/)  
106. Building Multi-Agent Systems with Letta \- YouTube, accessed July 13, 2025, [https://www.youtube.com/watch?v=LX-qO5o8iRQ](https://www.youtube.com/watch?v=LX-qO5o8iRQ)  
107. FlockX Case Study \- Zep, accessed July 12, 2025, [https://www.getzep.com/customers/flockx/](https://www.getzep.com/customers/flockx/)  
108. Developments in AI Agents: Q1 2025 Landscape Analysis, accessed July 12, 2025, [https://www.ml-science.com/blog/2025/4/17/developments-in-ai-agents-q1-2025-landscape-analysis](https://www.ml-science.com/blog/2025/4/17/developments-in-ai-agents-q1-2025-landscape-analysis)  
109. Letta\_AI \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/Letta\_AI/top/?t=month](https://www.reddit.com/r/Letta_AI/top/?t=month)  
110. \[2507.03724\] MemOS: A Memory OS for AI System \- arXiv, accessed July 13, 2025, [https://arxiv.org/abs/2507.03724](https://arxiv.org/abs/2507.03724)  
111. BAI-LAB/MemoryOS \- GitHub, accessed July 12, 2025, [https://github.com/BAI-LAB/MemoryOS](https://github.com/BAI-LAB/MemoryOS)  
112. Memory OS of AI Agent \- arXiv, accessed July 12, 2025, [https://www.arxiv.org/pdf/2506.06326](https://www.arxiv.org/pdf/2506.06326)  
113. Memos \- GitHub, accessed July 12, 2025, [https://github.com/usememos](https://github.com/usememos)  
114. usememos/memos: A modern, open-source, self-hosted knowledge management and note-taking platform designed for privacy-conscious users and organizations. \- GitHub, accessed July 12, 2025, [https://github.com/usememos/memos](https://github.com/usememos/memos)  
115. Memos \- Open Source, Self-hosted, Your Notes, Your Way, accessed July 12, 2025, [https://www.usememos.com/](https://www.usememos.com/)  
116. Is Memos safe to use? : r/selfhosted \- Reddit, accessed July 13, 2025, [https://www.reddit.com/r/selfhosted/comments/1b8idx7/is\_memos\_safe\_to\_use/](https://www.reddit.com/r/selfhosted/comments/1b8idx7/is_memos_safe_to_use/)