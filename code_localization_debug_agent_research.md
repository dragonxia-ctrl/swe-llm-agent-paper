# 代码定位与 Debug Agent：顶刊论文研究建议

> 研究目标：围绕代码定位（Code Localization）与自动化 Debug Agent（Automated Debugging Agent）开展研究，形成具有顶级软件工程期刊发表潜力的论文。

## 一、研究方向定位

建议将研究方向定义为：

**Repository-Level Autonomous Debugging Agent（仓库级自主代码调试 Agent）**

研究重点不应只是让 Agent 更准确地找到 Bug，而应研究如何构建具备代码库级理解、故障证据推理、根因定位、修复规划和验证能力的自主软件调试 Agent。

完整 Debug Agent 可以分为六个能力层次：

| 层次 | 核心能力 | 研究问题 |
|---|---|---|
| 1. Issue Understanding | Issue / Bug 理解 | 如何理解自然语言故障描述？ |
| 2. Code Localization | 代码定位 | 如何定位真正相关的文件、类、函数和代码行？ |
| 3. Root Cause Analysis | 根因分析 | 如何从错误现象推导出根本原因？ |
| 4. Fix Planning | 修复规划 | 如何生成可执行、低风险的修复方案？ |
| 5. Patch Generation | 补丁生成 | 如何生成正确且最小化的代码补丁？ |
| 6. Validation | 自动验证 | 如何通过测试和程序分析验证修复有效性？ |

**关键判断：Code Localization 应当是整个系统的核心研究对象，而不是 Debug Agent 中一个普通的检索模块。**

如果 Agent 找错了文件或函数，后续根因分析、补丁生成和测试验证都可能建立在错误基础上。

## 二、相关论文与文献入口

文献需要区分正式期刊论文、会议论文和预印本。预印本不能等同于已发表的顶刊论文；投稿和正式发表状态应以出版商页面为准。

### 2.1 值得重点核验的期刊论文

以下列出上一轮调研中提到的论文及官方检索入口。正式引用前，应进一步核对作者、年份、卷期、DOI、发表状态和全文可用性。

1. **Fault Localization from the Semantic Code Search Perspective**
   - 上一轮标注：ACM TOSEM，2026；DOI：`10.1145/3757915`
   - 关注点：语义代码检索与故障定位、候选代码排序。
   - [ACM DOI 页面](https://doi.org/10.1145/3757915)

2. **BLAZE: Cross-Language and Cross-Project Bug Localization via Dynamic Chunking and Hard Example Learning**
   - 上一轮标注：IEEE TSE，2024。
   - 关注点：跨语言、跨项目 Bug Localization，动态代码切分与困难样本学习。
   - [IEEE Xplore 检索入口](https://ieeexplore.ieee.org/search/searchresult.jsp?queryText=BLAZE%20Cross-Language%20Cross-Project%20Bug%20Localization)

3. **Integrating Various Software Artifacts for Better LLM-based Bug Localization and Program Repair**
   - 上一轮标注：ACM TOSEM，2024；相关工作 DEVLoRe。
   - 关注点：整合 Issue、源代码、测试用例、提交历史等软件工件。
   - [相关文献与资源索引](https://github.com/ZhonghaoJiang/Awesome-Issue-Solving)

4. **BugCerberus: Bridging Bug Localization and Issue Fixing: A Hierarchical Localization Framework Leveraging Large Language Models**
   - 上一轮标注：IEEE TSE，2025。
   - 关注点：分层式代码定位，以及定位与修复之间的衔接。
   - [IEEE Xplore 检索入口](https://ieeexplore.ieee.org/search/searchresult.jsp?queryText=BugCerberus%20Hierarchical%20Localization)

5. **Measure Twice, Locate Once: Mitigating Hallucinations in LLM-based Agents for Repository-Scale Fault Localization**
   - 上一轮标注：TOSEM，2026。
   - 关注点：仓库级 Fault Localization Agent 的幻觉、定位证据、候选验证和可信度。
   - [ACM 官方检索入口](https://dl.acm.org/action/doSearch?AllField=Measure+Twice+Locate+Once)

6. **GraMuS: Boosting statement-level fault localization via graph representation and multimodal information**
   - 上一轮标注：Journal of Systems and Software，2026。
   - 关注点：图表示、多模态信息与语句级故障定位。
   - [ScienceDirect 检索入口](https://www.sciencedirect.com/search?qs=GraMuS%20Boosting%20statement-level%20fault%20localization)

### 2.2 会议论文、预印本与研究资源

这些工作可用于追踪新思路，但需核验其正式发表状态。

| 论文 / 资源 | 上一轮提及的来源或入口 | 关注点 |
|---|---|---|
| GraphLocator: Graph-guided Causal Reasoning for Issue Localization | [研究索引](https://seer-radar.com/research-watch.html) | 图引导因果推理与 Issue 定位 |
| Towards AI-Powered Localization of Software Bugs: From Semantic Understanding to Agentic Cognition | FSE Companion 2026（需核验） | 从语义理解走向 Agentic Localization |
| Retrieval-Oriented Code Representations in Agentic Bug Localization | [arXiv:2607.11046](https://arxiv.org/abs/2607.11046) | 代码表示、检索效率和定位精度 |
| How Should Agents See Code? Rethinking Code Representations for Agentic Software Engineering | [arXiv:2611.07286](https://arxiv.org/abs/2611.07286) | AST、调用图、模块依赖与混合代码表示 |
| Agentless: Demystifying LLM-based Software Engineering Agents | [arXiv:2407.01489](https://arxiv.org/abs/2407.01489) | 定位、修复、验证三阶段设计 |
| Agentic Software Issue Resolution 文献索引 | [Awesome Issue Solving](https://github.com/ZhonghaoJiang/Awesome-Issue-Solving) | Issue 解决相关论文与代码 |

综述入口：

- [Large Language Models for Software Fault Localization: A Survey（2025，上一轮提供的入口）](https://doi.org/10.1109/DSA66321.2025.00024)
- [Survey on Learning-Based Dynamic Fault Localization（2026，上一轮提供的入口）](https://ink.library.smu.edu.sg/sis_research/11097/)

> **文献核验提醒：** 上述列表来自初步检索与上一轮整理，不应直接视为已完成书目核验的最终参考文献表。尤其是 2026 年论文、会议论文和预印本，应逐项核实出版状态、作者信息、DOI、论文内容及 PDF 是否可公开下载。引用前请以出版商、会议官网或 arXiv 原始页面为准。

## 三、研究演进与关键科学问题

可以将该领域的演进概括为：

1. **传统 Fault Localization**：SBFL、MBFL、IR-based Bug Localization，通过测试覆盖率、执行频谱和文本匹配定位错误。
2. **LLM-based Code Localization**：Embedding、RAG、Code Chunking、LLM Ranking，通过语义理解和检索提升候选代码定位能力。
3. **Repository-level Agentic Localization**：Multi-Agent、Graph-RAG、Tool Calling、Repository Understanding，让 Agent 主动探索大型代码仓库。
4. **Causal and Evidence-Grounded Debugging**：Causal Graph、Dynamic Slicing、Evidence Verification，从错误症状推导根因并验证推理。
5. **Autonomous Debugging**：Diagnosis、Fix Planning、Repair、Validation，形成可验证、可追溯的自主调试闭环。

### 3.1 五个值得研究的方向

| 方向 | 研究问题 | 潜在价值 |
|---|---|---|
| A. Graph-based Code Localization | 如何利用代码依赖图提高定位精度？ | 高 |
| B. Causal Fault Localization | 如何从症状定位真正根因？ | 高 |
| C. Evidence-Grounded Agent | 如何减少 Agent 幻觉和错误定位？ | 高 |
| D. Adaptive Repository Exploration | 如何动态决定下一步探索哪些代码？ | 高 |
| E. Self-Reflective Debugging | 如何根据测试反馈修正诊断结论？ | 较高 |

建议重点研究 A、B、C 的结合。

### 3.2 Graph-based Repository Understanding

传统 RAG 通常将代码切分成 Chunk，再通过 Embedding 检索相关代码。但软件代码具有明确的结构关系：

- Function Call Relationship
- Class Inheritance
- Variable Dependency
- Data Flow
- Control Flow
- Module Dependency
- API Dependency

因此，可以将 Repository 建模为一个异构图（Heterogeneous Code Graph）。

示意：

```text
Repository
    |
    +-- Module A
    |     |
    |     +-- Class A
    |     |     |
    |     |     +-- Function A1
    |     |     +-- Function A2
    |     |
    |     +-- Function A3
    |
    +-- Module B
          |
          +-- Class B
                |
                +-- Function B1
```

故障定位过程可以沿着结构关系展开：

```text
Issue
  |
  v
Symptom Function
  |
  v
Call Graph
  |
  v
Data Dependency
  |
  v
Suspicious Function
  |
  v
Root Cause Candidate
```

值得研究的问题：

1. 如何融合 AST、Call Graph、Data Flow Graph 和文本语义？
2. 如何在大型 Repository 中高效进行图检索？
3. 如何根据 Bug 类型动态选择需要探索的图关系？
4. 如何利用图结构识别传统 RAG 难以找到的跨文件依赖？
5. 如何降低 Graph Context 的 Token 消耗？

注意：Graph-RAG 本身已不是足够的新颖的论文贡献。需要证明提出的图表示、检索算法或推理机制解决了现有方法不能解决的问题。

### 3.3 Causal Fault Localization（因果故障定位）

这是建议优先投入的核心方向。

例如：

```python
def calculate_price(order):
    subtotal = calculate_subtotal(order)
    discount = calculate_discount(order)
    tax = calculate_tax(subtotal)
    return subtotal - discount + tax
```

Issue：

```text
The final price is incorrect
when a discount is applied.
```

简单语义搜索可能定位到 `calculate_price()`、`calculate_discount()` 和 `calculate_tax()`。但真正的问题可能是税额应根据折扣后的金额计算。

因此，需要解决的不是代码相关性，而是从 Observable Symptom 推导 Causal Root Cause。

可以设计以下推理过程：

```text
Observed Failure
      |
      v
Symptom Identification
      |
      v
Hypothesis Generation
      |
      v
Dependency Analysis
      |
      v
Causal Hypothesis
      |
      v
Counterfactual Verification
      |
      v
Root Cause Localization
```

值得重点研究 Counterfactual Verification（反事实验证）：

- 如果某个变量的值发生变化，错误是否仍然出现？
- 如果屏蔽某条调用路径，故障是否消失？
- 如果恢复某次历史代码变更，故障是否消失？
- 如果修改某个可疑条件分支，测试结果是否发生预期变化？

程序中的因果关系不能仅靠 LLM 自己判断，应结合动态执行、干预实验和测试结果，避免把相关性误认为因果性。

### 3.4 Evidence-Grounded Debug Agent

建议让 Agent 的每一个定位结论都必须有 Evidence 支撑。

| 定位结论 | 应提供的证据 |
|---|---|
| 文件可疑 | Issue 语义、文件职责、历史修改 |
| 函数可疑 | 调用链、输入输出、异常传播 |
| 代码行可疑 | AST、变量定义使用关系、执行轨迹 |
| 根因成立 | 失败测试、反事实实验、因果依赖 |
| 修复正确 | 原始失败测试通过、回归测试通过 |

可以设计 Evidence Record：

```python
class EvidenceRecord:
    source_file: str
    symbol: str
    line_range: tuple[int, int]

    evidence_type: str
    observation: str

    confidence: float
    causal_relation: str
    validation_status: str
```

真正的创新不只是给 Agent 增加 Evidence 字段，而是设计 **Evidence-Driven Hypothesis Generation and Verification Mechanism**：当证据不足时，Agent 主动寻找新证据，必要时拒绝给出确定性结论。

### 3.5 Adaptive Repository Exploration

传统 Agent 可能使用固定流程：

```text
Search → Read File → Search → Read File → Fix
```

但不同 Bug 需要不同探索策略：

- Null Pointer：追踪变量定义和数据流。
- Race Condition：分析线程、共享状态和同步关系。
- API Compatibility：分析接口调用和版本变更。
- Regression Bug：分析 Git 历史和代码变更。
- Configuration Bug：追踪配置文件和环境变量。

可以研究根据当前证据动态决定下一步工具调用的策略模型：

- Next-Best-Action Prediction
- Uncertainty-Aware Exploration
- Information Gain
- Exploration Cost Optimization
- Adaptive Tool Selection

### 3.6 Self-Reflective Debugging

研究 Agent 如何利用执行反馈持续修正定位结果：

```text
Initial Localization
        |
        v
Generate Hypothesis
        |
        v
Generate Reproduction Test
        |
        v
Execute Test
        |
        v
Analyze Feedback
        |
        +------ Failure remains
        |             |
        |             v
        |       Revise Hypothesis
        |             |
        +-------------+
        |
        v
Validate Root Cause
```

关键是区分“反复尝试不同代码”和“根据新的执行证据更新根因假设”。后者才是更有研究意义的 Self-Reflective Debugging。

## 四、建议的论文选题与系统架构

### 4.1 首选论文题目

**Evidence-Grounded Causal Debugging Agent for Repository-Level Fault Localization**

中文：面向仓库级故障定位的证据驱动因果调试智能体。

核心研究假设：

> 将 Repository Graph、Causal Reasoning 和 Evidence Verification 结合，可以比单纯依赖 LLM 语义检索或传统多 Agent 协作，更准确、更高效地定位真实根因。

拟议核心贡献：

1. 一种面向 Repository 的多层次代码依赖表示。
2. 一种基于证据约束的因果假设生成和动态扩展算法。
3. 一种基于反事实验证的 Root Cause Confirmation 机制。
4. 一个支持跨项目、跨语言评估的 Debug Agent 原型。
5. 一套细粒度的定位、因果解释、成本和泛化评估体系。

不要只将贡献写成“提出一个多 Agent 框架”。应强调算法贡献，例如：

> We propose an evidence-grounded causal localization mechanism that iteratively discovers and verifies root-cause hypotheses through repository dependency analysis and execution-based interventions.

### 4.2 建议的六层系统架构

| 层次 | 主要组件 | 可选技术 |
|---|---|---|
| L1 Repository Intelligence | Repo Analyzer | AST、Tree-sitter、LSIF、Code Graph |
| L2 Context Retrieval | Context Retriever | Hybrid Search、Embedding、Graph-RAG |
| L3 Causal Reasoning | Causal Reasoner | Dependency Analysis、Slicing、Causal Hypothesis |
| L4 Evidence Management | Evidence Manager | Evidence Graph、Confidence、Provenance |
| L5 Debug Agent | Planner、Localizer、Verifier | Google ADK、Tool Calling |
| L6 Validation | Test Agent | pytest、Coverage、Mutation Testing、Docker |

工程工具的角色：

- Google ADK：Agent 编排实现。
- Tree-sitter：语法结构分析。
- NetworkX 或图数据库：代码依赖图。
- Embedding / Reranker：候选代码检索。
- pytest：执行验证。
- Docker：实验环境隔离。

这些工具本身不是论文创新。论文应重点放在定位算法、因果推理和证据验证机制上。

## 五、实验设计

顶刊论文需要证明提出的方法在科学问题上确实有效，不能只展示 Agent 能运行若干 SWE-bench 案例。

### 5.1 建议使用的数据集

| 数据集 | 主要用途 | 优先级 |
|---|---|---|
| SWE-bench Verified | 真实仓库级 Issue 定位和修复 | 必须 |
| Defects4J | Java 传统故障定位 | 必须 |
| BugsInPy | Python 真实 Bug | 高 |
| QuixBugs | 算法级错误定位和修复 | 中 |
| ManyBugs | C 程序 Bug | 中 |
| SWE-bench Pro | 更复杂的仓库级任务 | 高 |

建议同时使用传统 Bug Benchmark 和现代 Repository-level Benchmark。SWE-bench 的最终修复成功率不能充分说明定位机制是否正确；Defects4J 等数据集更适合细粒度真实错误位置评估。

### 5.2 评价指标

**A. 定位精度**

| Metric | 含义 |
|---|---|
| Top-1 Accuracy | 第一候选是否命中真实 Bug |
| Recall@5 | 前五个候选是否包含真实 Bug |
| Recall@10 | 前十个候选是否包含真实 Bug |
| MRR | 首次命中位置的倒数排名 |
| MAP | 多个正确定位结果的平均精度 |

**B. 根因推理能力**

- Root Cause Accuracy
- Causal Relation Accuracy
- Evidence Coverage
- Counterfactual Verification Accuracy
- False Root Cause Rate

**C. Agent 效率**

- Average Token Consumption
- Average API Cost
- Number of Tool Calls
- Repository Exploration Time
- Number of Files Inspected

**D. 最终调试效果**

- Patch Success Rate
- Regression Test Pass Rate
- Test Generation Success Rate
- Patch Correctness
- Generalization Across Projects

### 5.3 Ablation Study

建议设计以下消融实验：

| 实验 | Graph | Causal Reasoning | Evidence Verification | Adaptive Exploration |
|---|---|---|---|---|
| Baseline | - | - | - | - |
| Exp 1 | ✓ | - | - | - |
| Exp 2 | ✓ | ✓ | - | - |
| Exp 3 | ✓ | ✓ | ✓ | - |
| Full Model | ✓ | ✓ | ✓ | ✓ |

还应比较：

1. 传统 SBFL。
2. 纯 LLM Localization。
3. Agentless。
4. SWE-agent。
5. OrcaLoca。
6. GraphLocator。
7. 最新可复现的 Repository-level Localization Agent。

必须控制模型、提示词预算、工具权限和上下文长度，避免不公平比较。

## 六、目标期刊

### 6.1 首选期刊

1. **IEEE Transactions on Software Engineering（TSE）**
   - 软件工程领域顶级期刊。
   - 适合具有明确算法创新、充分实验和理论分析的工作。
   - [期刊官网](https://www.computer.org/csdl/journal/ts)

2. **ACM Transactions on Software Engineering and Methodology（TOSEM）**
   - 适合 Fault Localization、Program Analysis、Automated Repair 和 Agent 方法。
   - [期刊官网](https://dl.acm.org/journal/tosem)

3. **Empirical Software Engineering（EMSE）**
   - 适合强调大规模实验、实证研究、方法比较和可复现性的论文。
   - [期刊官网](https://link.springer.com/journal/10664)

4. **Journal of Systems and Software（JSS）**
   - 适合自动化调试系统、程序分析方法和实证研究。
   - [期刊官网](https://www.sciencedirect.com/journal/journal-of-systems-and-software)

5. **Information and Software Technology（IST）**
   - 适合软件工程方法、自动化工具和工程实践相结合的研究。
   - [期刊官网](https://www.sciencedirect.com/journal/information-and-software-technology)

建议：
- 如果提出新的因果定位算法，并有扎实的程序分析和验证理论：优先 TSE / TOSEM。
- 如果主要贡献是大规模实证研究和系统性方法评估：考虑 EMSE。
- 如果主要贡献是完整工程系统和实用调试方法：考虑 JSS / IST。

## 七、9—12 个月研究计划

1. **第 1 阶段：文献调研（1—2 个月）**
   - 阅读 30—50 篇相关论文。
   - 重点研究 TSE、TOSEM、ICSE、FSE、ASE。
   - 建立 Bug Localization、Causal Debugging、Agentic SWE 三个文献分类。
   - 明确尚未解决的研究问题。

2. **第 2 阶段：Baseline 和数据集（1—2 个月）**
   - 复现 Agentless。
   - 复现 GraphLocator 或相关图定位基线。
   - 构建 Defects4J、BugsInPy、SWE-bench 实验环境。
   - 建立文件、函数、语句三级定位评价体系。

3. **第 3 阶段：核心算法（2—3 个月）**
   - 构建 Repository Graph。
   - 设计 Evidence Graph。
   - 实现 Causal Hypothesis Generation。
   - 实现 Counterfactual Verification。
   - 实现 Adaptive Exploration。

4. **第 4 阶段：系统实验（2—3 个月）**
   - 与最新方法进行对比。
   - 开展消融实验。
   - 开展跨语言、跨项目实验。
   - 开展成本与效率分析。
   - 进行错误案例分析和统计显著性检验。

5. **第 5 阶段：论文撰写（1—2 个月）**
   - 确定核心研究贡献。
   - 整理算法、实验和理论分析。
   - 完成论文及开源代码。
   - 按目标期刊格式准备投稿。

这是紧凑计划。如果需要构建新基准、开展人工标注或进行多语言验证，时间还需要增加。

## 八、最终研究建议

不要把论文做成一个更复杂的 Debug Agent，而应该把论文做成一个能够解决现有 Debug Agent 核心缺陷的新方法。

现有 Agent 的典型问题包括：

- 能找到相关文件，但不能可靠区分相关代码和根因代码。
- 能给出根因解释，但缺少足够的执行证据。
- 能反复探索代码库，但不能合理控制探索成本。

建议形成以下研究主线：

```text
Repository-Level Code Understanding
(AST + Dependency Graph + Semantic Retrieval)
                    |
                    v
Evidence-Grounded Causal Reasoning
(Hypothesis Generation + Causal Graph + Evidence)
                    |
                    v
Counterfactual Root Cause Verification
(Dynamic Slicing + Execution + Intervention)
                    |
                    v
Reliable Autonomous Debugging
(Root Cause + Fix Planning + Validated Patch)
```

## 九、建议优先确定的研究边界

初步建议：

- **核心方向：** A + B + C，即 Graph-based Code Localization + Causal Fault Localization + Evidence-Grounded Debugging。
- **研究范围：** Repository 级定位和根因分析。
- **实验语言：** Python + Java。
- **目标期刊：** TSE / TOSEM。

下一步应先完成竞争性文献核验，尤其重点阅读和比较 GraphLocator、Measure Twice, Locate Once、Retrieval-Oriented Code Representations，以及 Agentless，明确已有工作与拟议方法之间的差异，再冻结论文创新点。

---

## 附：重要资源链接

- [Awesome Issue Solving：Issue 解决相关论文与代码索引](https://github.com/ZhonghaoJiang/Awesome-Issue-Solving)
- [Agentless 论文（arXiv）](https://arxiv.org/abs/2407.01489)
- [Defects4J](https://github.com/rjust/defects4j)
- [SWE-bench](https://www.swebench.com/)
- [BugsInPy](https://github.com/soarsmu/BugsInPy)
- [IEEE TSE](https://www.computer.org/csdl/journal/ts)
- [ACM TOSEM](https://dl.acm.org/journal/tosem)
