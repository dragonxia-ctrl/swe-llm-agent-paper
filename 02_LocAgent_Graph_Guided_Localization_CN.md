# LocAgent：图引导的大语言模型智能体代码定位框架

**原文标题**：LocAgent: Graph-Guided LLM Agents for Code Localization  
**作者**：Zhaoling Chen, Xiangru Tang, Gangda Deng, Fang Wu, Jialong Wu, Zhiwei Jiang, Viktor Prasanna, Arman Cohan, Xingyao Wang  
**机构**：耶鲁大学、南加州大学、斯坦福大学、All Hands AI  

---

## 摘要（Abstract）

代码定位（Code Localization）——即在代码库中精确识别需要修改的位置——是软件维护中一项基础但极具挑战性的任务。现有方法在复杂代码库中难以高效导航并跨越多重依赖定位相关代码。本文提出了 **LOCAGENT**，一个通过基于图的中间表示来解决代码定位的智能体框架。LOCAGENT 将代码库解析为**有向异构图（Directed Heterogeneous Graph）**，以轻量级结构统一捕获代码层级（目录、文件、类、函数）与依赖关系（导入、调用、继承），使 LLM 智能体能够通过强大的多跳推理（Multi-hop Reasoning）高效检索和定位目标实体。实验表明，使用微调后的 **Qwen-2.5-Coder-Instruct-32B** 模型，LOCAGENT 即可达到与闭源 SOTA 模型（Claude-3.5）相当的精度，同时将成本降低约 **86%**，文件级定位准确率最高达 **92.7%**，并将下游 GitHub 问题解决率（Pass@10）提升 **12%**。此外，本文还构建了涵盖多种维护场景的新基准数据集 **LOC-BENCH**。

---

## 一、 引言（Introduction）

开发者在调试过程中有高达 **66%** 的时间花费在理解代码结构与定位缺陷上。传统的稠密检索方法需要维护庞大的向量索引，难以适应频繁迭代的代码库；而现有的 LLM 智能体（如 SWE-Agent、OpenHands）主要依赖目录遍历和文本搜索（`grep`），在面对 Issue 描述仅提及症状、而根因隐藏在跨目录深层调用链中的场景时，往往难以进行有效的跨文件多跳推理。

LOCAGENT 的核心贡献包括：
1. **异构代码图表示与稀疏分层索引**：将代码库转化为包含四种节点和四种边的有向异构图，并结合稀疏索引实现毫秒级建图与高效检索。
2. **面向 LLM 优化的统一检索工具集**：设计了 `SearchEntity`、`TraverseGraph` 和 `RetrieveEntity` 三个高阶工具，配合树状子图文本编码格式以最大化 LLM 的图推理能力。
3. **开源模型蒸馏与微调**：从 Claude-3.5 的成功探索轨迹中提炼高质量推理路径，对 Qwen2.5-Coder（7B 与 32B）进行 LoRA 微调，大幅降低部署成本。
4. **发布 LOC-BENCH 基准**：克服 SWE-Bench 数据泄露与单一缺陷偏见问题，涵盖 Bug 修复、功能请求、安全补丁与性能优化四大类别。

---

## 二、 LOCAGENT 框架设计（The LOCAGENT Framework）

### 2.1 基于图的代码表示（Graph-based Code Representation）
1. **代码图构建（Code Graph Construction）**：
   构建有向异构图 $\mathcal{G}(\mathcal{V}, \mathcal{E}, \mathcal{A}, \mathcal{R})$：
   * **节点类型 $\mathcal{A}$**：目录（`directory`）、文件（`file`）、类（`class`）、函数（`function`，作为最小检索粒度，存储完整代码内容）。
   * **边类型 $\mathcal{R}$**：包含（`contain`）、导入（`import`）、调用（`invoke`，包括函数调用与类实例化）、继承（`inherit`）。
   * 通过 AST 抽象语法树解析 Python 代码，将物理目录上相距甚远但存在调用或继承关系的模块在图拓扑中紧密连接。
2. **稀疏分层实体索引（Sparse Hierarchical Entity Indexing）**：
   为每个节点构建四层自上而下的查找索引：(1) 完整限定名 ID 索引（如 `src/utils.py:MathUtils.calculate_sum`）；(2) 实体短名称全局字典；(3) 实体 ID 的 BM25 倒排索引；(4) 代码块内容的 BM25 倒排索引。索引构建仅需数秒。

### 2.2 智能体引导的代码搜索（Agent-guided Code Search）
LOCAGENT 为智能体配备了三个只读统一工具：
* **`SearchEntity(keywords)`**：通过分层索引搜索匹配实体。为防止上下文膨胀，返回结果采用三档自适应细节格式：匹配数 $\le 3$ 时展示**完整代码（Full Code）**；单实体超过 300 行时展示**类/模块骨架（Preview）**；匹配数 $> 3$ 时仅折叠展示**实体 ID 列表（Fold）**。
* **`TraverseGraph(start_entities, direction, traverse_hops, entity_types, relation_types)`**：在异构图上执行类型感知的广度优先搜索（BFS），支持单步指定多跳（Multi-hop）与元路径（Meta-path）过滤。输出子图采用**扩展树状格式（Expanded Tree-Based Format）**，利用缩进表达跳数距离并显式标注正向/反向关系类型。
* **`RetrieveEntity(entity_ids)`**：获取指定实体 ID 的完整属性，包括文件路径、起止行号及完整源码。

**思维链规划与一致性重排**：智能体按“关键词提取 $\rightarrow$ 实体关联 $\rightarrow$ 故障逻辑流多跳追踪 $\rightarrow$ 目标实体定位”四步推理，并跨多轮采样利用倒数排名（Reciprocal Rank）聚合计算最终置信度得分。

### 2.3 开源模型微调（Open-source Model Fine-tuning）
收集 Claude-3.5 在 SWE-bench 训练集上的 433 条成功轨迹，结合微调后 Qwen2.5-32B 自生成的 335 条成功轨迹（共 768 条高质量轨迹），采用 LoRA 对 Qwen2.5-Coder-7B 和 32B 进行监督微调（SFT）。

---

## 三、 LOC-BENCH 基准与实验结果（LOC-BENCH & Experiments）

### 3.1 LOC-BENCH 数据集
从 2024 年 10 月后创建的 GitHub Issue 中收集 560 个样本，避免大模型预训练数据污染，包含 **242 个 Bug 报告**、**150 个功能请求**、**139 个性能优化**和 **29 个安全漏洞**。

### 3.2 核心实验结论
1. **SWE-bench Lite 定位表现**：
   * 搭配 Claude-3.5 时，LOCAGENT 文件级 Acc@1/Acc@5 达到 **77.74% / 94.16%**，函数级 Acc@5/Acc@10 达到 **73.36% / 77.37%**，全面超越 OpenHands、SWE-Agent、MoatlessTools 和 Agentless。
   * 微调后的 **Qwen2.5-32B(ft)** 文件级 Acc@5 达 **92.70%**，函数级 Acc@10 达 **77.01%**，几乎追平 Claude-3.5。
2. **多跳难度鲁棒性**：在需要 2 跳及 3 跳以上图遍历的高难度任务中，传统检索与 Agentless 性能急剧下滑，而 LOCAGENT 凭借图遍历工具保持了显著优势。
3. **成本与效率**：`Qwen2.5-7B(ft)` 单次定位成本仅 **$0.05**（6 轮交互），`Qwen2.5-32B(ft)` 仅 **$0.09**（9 轮交互），相比 Claude-3.5（$0.66）成本降低 **86%**。
4. **消融实验**：移除 `SearchEntity` 或 BM25 索引导致性能暴跌（函数级 Acc@10 从 71.53% 降至 53.28%）；限制 `TraverseGraph` 跳数为 1 或仅保留 `contain` 关系均显著削弱函数级定位精度；树状子图格式优于邻接行、DOT 和 JSON 格式。
5. **下游修复提升**：将 LOCAGENT 定位结果接入 Agentless 编辑模块后，SWE-bench Lite 上的 Pass@10 从 33.58% 提升至 **37.59%**（相对提升约 12%）。
