# OrcaLoca：用于软件问题定位的大语言模型智能体框架

**原文标题**：OrcaLoca: An LLM Agent Framework for Software Issue Localization  
**作者**：Zhongming Yu, Hejia Zhang, Yujie Zhao, Hanxian Huang, Matrix Yao, Ke Ding, Jishen Zhao  
**机构**：加州大学圣地亚哥分校（UCSD）、英特尔公司（Intel）  
**发表会议**：ICML 2025  

---

## 摘要（Abstract）

大语言模型（LLM）智能体正在推动自主软件工程（ASE）的发展，然而“问题定位”——即在复杂代码库中精准导航至相关代码片段——仍是核心瓶颈。现有方法往往因缺乏智能体规划与精确代码搜索机制的有效融合而表现不佳。本文提出了 **ORCALOCA**，一个通过集成**基于优先级的动作调度（Priority-Based Scheduling）**、**带相关性评分的动作分解（Action Decomposition with Relevance Scoring）**以及**距离感知的上下文剪枝（Distance-Aware Context Pruning）**来提升软件问题定位精度的智能体框架。实验表明，ORCALOCA 在 SWE-bench Lite 上取得了 **65.33%** 的函数匹配率（Function Match Rate），刷新了开源框架的 SOTA 纪录，并将下游开源修复框架的最终解决率提升了 **6.33 个百分点**（达到 **41.00%**）。

---

## 一、 引言（Introduction）

SWE-bench 中的平均代码库包含约 3,010 个文件和 43.8 万行代码，而用户提交的 Issue 往往是模糊的自然语言描述。现有的 LLM 定位智能体面临三大挑战：
1. **动作规划低效与死板**：完全依赖 LLM 自由决策容易产生不稳定和重复的探索；而强行预设图遍历路径又会限制搜索灵活性。
2. **上下文简洁性与搜索空间完整性的矛盾**：仅使用代码骨架会丢失实现细节；引入全部代码又会导致严重的噪声与冗余。
3. **探索过程中的上下文污染**：在大型仓库中，函数重写（Overrides）和类继承等歧义会导致无关信息不断累积，误导 LLM 的最终判断。

---

## 二、 方法论（Methodology）

ORCALOCA 构建了一个包含 containment（包含）与 reference（引用/调用）关系的异构 **CodeGraph**，为每个代码实体分配唯一标识符 UID（格式为 `file_path(::cls)(::method)`），并围绕以下三大核心机制运行：

### 2.1 基于优先级的 LLM 引导动作调度（Priority-Based Scheduling）
* **动作调度队列（ASQ）**：使用堆（Heap）数据结构维护 LLM 生成的候选搜索动作。默认动作优先级为 1，但具有强上下文关联或由动作分解产生的子动作会被赋予更高优先级。
* **紧急度计数器（Urgency Counter）**：为每个唯一动作维护计数器 $C_{a_k}$。当 LLM 在多轮推理中反复提出同一个动作时，计数器累加并动态提升其在 ASQ 中的优先级，确保模型最关注的线索优先执行。
* **冗余动作消除**：在动作入队前，预取其 CodeGraph UID 并存入**动作搜索数据库（Action Search Database）**，自动拦截不同 API 形式（如 `search_class(C)` 与 `search_class_in_file(C, f)`）指向同一实体的重复查询。

### 2.2 带相关性评分的动作分解与消歧（Action Decomposition with Relevance Scoring）
* **文件与类动作分解**：当检索目标是一个庞大的类或文件（触发骨架模式）时，ORCALOCA 调用一个独立的**评分与排序子智能体（Code Scorer Sub-agent）**，根据问题描述对类内所有方法或文件内所有实体进行 0–100 分的相关性打分，仅保留 Top-$k$（类取 $k=3$，文件取 $k=2$）重组为高优先级细粒度搜索动作压入 ASQ，兼顾完备性与简洁性。
* **同名实体消歧机制**：构建倒排索引存储存在同名歧义的可调用实体。当检索遇到多匹配（如多个数据库后端均定义了 `SQLCompiler`）时，系统生成包含所有候选文件路径的消歧提示，并将拆分后的精确搜索动作重新排入队列。

### 2.3 距离感知的搜索上下文剪枝（Distance-Aware Searched Context Pruning）
上下文管理器（CM）在每一步探索中动态过滤历史搜索结果（SR），显式剔除骨架信息与消歧消息，仅保留具有有效 UID 的实体，并计算每个已检索节点 $v_{SR}$ 与当前候选缺陷位置集合 $PB$ 在 CodeGraph 上的平均双向最短路径距离：
$$d(SR, PB) = \frac{1}{|PB|} \sum_{v \in PB} \min(d(v_{SR}, v), d(v, v_{SR}))$$
CM 仅保留距离最近的 Top-12 个代码片段进入 LLM 上下文窗口，有效隔离无关调用分支的噪声。

### 2.4 辅助复现智能体与工程优化（讨论部分）
* **复现智能体（Reproducer Agent）**：利用 VizTracer 在 Docker 沙箱中执行生成的复现脚本，提取运行时调用栈与异常追踪，通过启发式规则过滤 Top-5 关键帧作为冷启动初始动作（约 38% 的 Issue 可成功复现）。
* **早停收敛与批量执行**：利用 BERT 嵌入计算相邻观测值的余弦相似度，连续 15 步超过 0.97 则触发早停；支持批量动作执行（`OrcaLoca-batch=2`），使单实例平均定位成本从 $0.87 降低 34% 至 **$0.58**。

---

## 三、 实验结果（Evaluation Results）

在 SWE-bench Lite（基于 Claude-3.5-Sonnet）及 SWE-bench Common（93 实例交集）上的评估结果如下：

1. **榜单 SOTA 表现**：
   * ORCALOCA 取得了 **65.33%**（196/300）的函数匹配率和 **83.33%**（250/300）的文件匹配率，在所有开源方案中排名第一。
   * 将定位结果接入 Agentless-1.5 的修复流程后，问题解决率达到 **41.00%**（123/300），较原版 Agentless-1.5（34.67%）提升 **6.33 个百分点**，函数匹配精确度（FMP）提升 **4.62 个百分点**。
2. **独特性分析**：与 AutoCodeRover、Agentless 和 OpenHands 相比，ORCALOCA 独立定位出了 6 个其他工具均未命中的 Issue，并独立解决了 8 个独有 Issue。
3. **消融实验（SWE-bench Common）**：完整 ORCALOCA 函数匹配率为 **76.34%**；移除优先级调度降至 **73.12%**，移除文件/类分解降至 **72.04%**，移除消歧分解降至 **70.97%**，移除距离感知上下文剪枝降至 **72.04%**，验证了各组件的有效性。
