# SWERANK+：面向软件问题定位的多语言、多轮次代码排序框架

**原文标题**：SWERANK+: Multilingual, Multi-Turn Code Ranking for Software Issue Localization  
**作者**：Revanth Gangi Reddy, Ye Liu, Wenting Zhao, JaeHyeok Doo, Tarun Suresh, Daniel Lee, Caiming Xiong, Yingbo Zhou, Semih Yavuz, Shafiq Joty  
**机构**：伊利诺伊大学厄巴纳-香槟分校（UIUC）、Salesforce AI Research、韩国科学技术院（KAIST AI）  

---

## 摘要（Abstract）

大规模多语言代码库的维护高度依赖精准的问题定位，即把自然语言错误描述映射到需要修改的相关函数。然而，现有的代码排序方法大多局限于 Python 仓库，且仅执行单次检索（Single-pass Search）。本文提出了 **SWERANK+**，一个将跨语言代码排序工具 **SWERANKMULTI** 与迭代式智能体搜索框架 **SWERANKAGENT** 相结合的新架构，用于对代码仓库进行多轮迭代推理。SWERANKMULTI 包含代码嵌入检索器和列表式 LLM 重排器，基于涵盖 10 种主流编程语言的大规模数据集 **SWELOCMULTI** 训练而成；SWERANKAGENT 则突破了单次检索的局限，通过维护记忆缓冲区（Memory Buffer）在多轮交互中逐步累积和精炼定位候选集。在多语言与 Python 基准测试上的实验表明，SWERANKMULTI 刷新了多语言代码排序的 SOTA 纪录，而 SWERANKAGENT 在此基础上进一步提升了复杂问题的定位精度。

---

## 一、 引言（Introduction）

此前提出的 SWERANK 虽然高效精准，但存在两大局限性：
1. **单语言局限**：真实企业的软件系统往往由多种编程语言（如前端 JS/TS、后端 Go/Java/Rust、底层 C/C++ 与 Python 混合）交织而成，而现有定位工具几乎全针对 Python 设计。
2. **单次传递（Single-pass）的瓶颈**：当 Bug 报告仅描述高层表象（例如报错堆栈停留在底层的正则表达式编译函数，而真正的缺陷在于上游解析逗号分隔配置的函数）时，单次查询无法跨越“语义鸿沟”追踪到上游根因，也难以覆盖分散在多个低耦合函数中的修改点。

为此，SWERANK+ 提出了双重升级：**SWERANKMULTI**（解决多语言问题）与 **SWERANKAGENT**（解决复杂多跳与跨语义鸿沟问题）。

---

## 二、 SWERANKMULTI：跨语言代码排序（SWERANKMULTI）

### 2.1 SWELOCMULTI 多语言数据集
扩展了 SWELOC 的采集与过滤流水线，覆盖 **JavaScript、Java、TypeScript、Ruby、Rust、Go、PHP、C、C++ 和 Python** 共 10 种语言。筛选 Star 数 $> 1000$、近半年有提交且目标语言占比 $\ge 40\%$ 的仓库，提取关联 Issue 且包含测试修改的 PR，最终构建了包含 **4,060 个仓库、56,279 个 PR 和 155,663 个训练实例**的 SWELOCMULTI 数据集（其中 Python 49,973 条，非 Python 超过 10.5 万条）。同样采用 Top-40 一致性过滤与难负例挖掘。

### 2.2 跨语言检索器与重排器
* **SWERANKEMBEDMULTI 检索器**：基于多语言能力强大的 `Qwen3-Embedding`（提供 **0.6B Small** 和 **8B Large** 两个版本）初始化，在 SWELOCMULTI 上使用 InfoNCE 对比损失微调，将不同语言的源码与自然语言 Issue 映射到统一的共享向量空间。
* **SWERANKLLMMULTI 重排器**：基于 `CodeRankLLM (7B)` 和 `Qwen-2.5-32B-Instruct` 初始化，沿用弱监督首 Token 预测目标在 SWELOCMULTI 上微调，实现跨语言候选函数的列表式精准重排。

---

## 三、 SWERANKAGENT：迭代式多轮智能体搜索（SWERANKAGENT）

SWERANKAGENT 采用 ReAct 风格的轻量级循环架构，将 `SWERANKMULTI` 作为核心检索工具封装，通过四个步骤在多轮对话中逐步逼近根因：

1. **搜索（Search）**：智能体根据当前查询（首轮为原始 Issue 描述，后续轮次为重构后的假设查询）调用 `SWERANKMULTI` 工具，检索并重排得到当前最相关的 Top-$k$（如 Top-10）函数。
2. **推理（Reasoning）**：分析本轮新检索到的函数源码，解释其与问题的关联性，评估现有证据是否已覆盖根本原因，或是否需要继续向上游/下游追踪。
3. **查询重构（Reformulation）**：基于已获取的代码结构线索，改写下一轮的搜索查询（例如从底层的报错函数名转向寻找上游的配置解析器或转换器函数），避免重复检索已看过的函数。
4. **聚合与全局重排（Aggregate）**：维护一个跨轮次的**内部记忆缓冲区（Memory Buffer）**，持续收集并去重各轮发现的高相关候选函数。当达到最大迭代轮次或智能体调用 `finish` 工具终止时，利用 `SWERANKLLMMULTI` 对记忆缓冲区中的所有候选函数执行一次**全局重排**，输出最终的 Top-10 定位列表。

---

## 四、 实验结果与深度分析（Experiments & Analysis）

在三大跨语言基准（**SWE-PolyBench**、**SWE-Bench-Multilingual**、**Multi-SWE-Bench**）以及三大 Python 基准（**SWE-Bench-Lite**、**LocBench**、**SWE-Bench-Verified**）上进行了全面评估：

1. **SWERANKMULTI 的跨语言与单语言双赢（RQ1）**：
   * `SWERANKEMBEDMULTI-Large (8B)` 配合 `SWERANKLLMMULTI-Large (32B)` 在所有基准上刷新 SOTA：在 SWE-Bench-Multilingual 上函数级 Acc@10 达到 **71.37%**（超越 GPT-4.1 重排的 70.94%）；在 SWE-PolyBench 上达到 **63.21%**。
   * 有趣的发现：得益于多语言训练带来的更强泛化能力，`SWERANKEMBEDMULTI` 在纯 Python 基准（如 SWE-Bench-Lite 达 **89.78%** Acc@10，Verified 达 **81.18%**）上的表现甚至超过了仅在 Python 数据上微调的模型。
2. **SWERANKAGENT 的多轮搜索增益（RQ2）**：
   * 对比单轮检索（Single-Query）以及直接用 GPT-5 一次性生成 5 个改写查询的“单轮多查询（Reformulate）”基线（该基线因盲目改写引入噪声反而导致精度下降），**SWERANKAGENT** 在所有数据集上均实现稳定提升：SWE-Bench-Multilingual 提升 **+6.41%**（从 66.67% 升至 **73.08%**），SWE-PolyBench 提升 **+3.09%**（升至 **62.63%**），SWE-Bench-Lite 提升 **+2.55%**（升至 **88.32%**）。
3. **复杂问题下的显著优势**：
   * 按词汇重叠度（ROUGE-1）与语义相似度分层分析显示，在重叠度最低（语义鸿沟最大）的困难样本桶中，`SWERANKAGENT` 的提升幅度最为显著；同时，在需要修改多个目标函数（$N \ge 2$）的复杂问题上，多轮聚合机制展现出了远超单次排序的覆盖能力。

---

## 五、 总结（Conclusion）

SWERANK+ 通过构建大规模多语言数据集 SWELOCMULTI 与训练跨语言排序器 SWERANKMULTI，打破了代码定位工具的 Python 语言壁垒；同时通过轻量级的 SWERANKAGENT 架构，将高效的代码排序器与多轮假设推理相结合，在不引入重型代码分析工具复杂度的前提下，攻克了低语义重叠与多函数耦合的复杂定位难题。
