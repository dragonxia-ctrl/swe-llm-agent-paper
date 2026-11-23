# SWERANK：基于代码排序的软件问题定位框架

**原文标题**：SWERANK: Software Issue Localization with Code Ranking  
**作者**：Revanth Gangi Reddy, Tarun Suresh, JaeHyeok Doo, Ye Liu, Xuan Phi Nguyen, Yingbo Zhou, Semih Yavuz, Caiming Xiong, Heng Ji, Shafiq Joty  
**机构**：伊利诺伊大学厄巴纳-香槟分校（UIUC）、Salesforce Research、韩国科学技术院（KAIST AI）  
**发表会议**：ICLR 2026  

---

## 摘要（Abstract）

软件问题定位旨在根据自然语言问题描述（如 Bug 报告、功能请求）识别需要修改的精确代码位置。尽管近期基于 LLM 的智能体方法展现出良好潜力，但由于复杂的多步推理以及对闭源大模型的依赖，它们往往伴随着高昂的延迟与推理成本。另一方面，传统的代码排序模型主要针对“查询到代码（Query-to-Code）”或“代码到代码（Code-to-Code）”检索进行优化，难以适应冗长且侧重描述系统故障表象的问题定位查询。为弥合这一差距，本文提出了 **SWERANK**，一个高效且精准的“检索-重排（Retrieve-and-Rerank）”软件问题定位框架，并构建了大规模对比学习数据集 **SWELOC**。实验表明，SWERANK 在 SWE-Bench-Lite 和 LocBench 上全面超越了现有代码排序模型以及使用 Claude-3.5 的高成本智能体系统，在大幅降低延迟与成本的同时刷新了 SOTA 纪录。

---

## 一、 引言（Introduction）

传统的代码检索查询通常简短（如 CodeSearchNet 平均仅 12 个 Token）且直接描述所需功能；而软件 Issue 描述则冗长（SWE-Bench 平均达 460 个 Token），且包含大量堆栈报错、异常现象而非直接的代码功能说明。与此同时，多轮智能体（如 LocAgent、OpenHands）平均需要 7–10 轮交互，单次查询成本高达 ~$0.66，且轨迹脆弱。

SWERANK 将问题定位重构为**单次传递的排序问题（Single-shot Ranking）**，包含两大核心组件与一个大规模数据集：
1. **SWELOC 数据集**：从 GitHub 真实仓库自动挖掘并经过严格质量过滤的大规模问题定位训练集。
2. **SWERANKEMBED**：双编码器（Bi-encoder）密集向量检索器，用于从大型代码库中快速召回候选函数。
3. **SWERANKLLM**：基于指令微调的列表式（Listwise）大语言模型重排器，对 Top-$K$ 候选函数进行精细重排。

---

## 二、 SWELOC 数据集构建（SWELOC: Issue Localization Data）

1. **相关 PR 筛选**：从 GitHub 上排名前 11,000 的 PyPI 软件包对应仓库中筛选 Python 代码占比 $\ge 80\%$ 的项目，剔除 SWE-Bench 和 LocBench 包含的仓库以防数据泄露，去重后得到 3,387 个高质量仓库。提取关联了 GitHub Issue 且包含测试文件修改的拉取请求（PR），获得 67,341 个 `(PR, codebase)` 对（平均查询长度 382.56 Token）。
2. **定位样本处理**：将 Issue 描述作为查询 $t_i$，PR 中修改的每个函数作为正样本 $c_i^+$，同仓库未修改的函数作为负样本池 $F_i$。
3. **一致性过滤与难负例挖掘（Consistency Filtering & Hard Negatives）**：
   * 使用预训练的 `CodeRankEmbed` 计算相似度，仅保留正样本函数在全库中排名前 $K=20$ 的实例（实验证明不过滤 $K=\text{None}$ 会因噪声导致性能下降）。
   * 从非修改函数中挖掘与查询语义最相似的前 $M=15$ 个函数作为**难负例（Hard Negatives）**。

---

## 三、 SWERANK 方法论（SWERANK Methodology）

### 3.1 SWERANKEMBED 检索器
采用双编码器架构，将 Issue 描述与代码函数映射到共享向量空间，使用 **InfoNCE 对比损失**进行训练（批内负样本结合挖掘的难负例）。推理时，代码库所有函数的向量可预先离线计算并缓存，线上仅需编码 Issue 文本并计算余弦相似度即可完成毫秒级检索。模型提供两种规模：
* **SWERANKEMBED-SMALL（137M）**：基于 `CodeRankEmbed` 初始化。
* **SWERANKEMBED-LARGE（7B）**：基于 `GTE-Qwen2-7B-Instruct` 初始化。

### 3.2 SWERANKLLM 列表式重排器
传统的列表式重排器需要完整的真实排序序列作为监督信号，但 SWELOC 仅知道哪个函数被修改（单正例），缺乏负例之间的相对排序。为此，SWERANK 提出了一种**弱监督列表式训练目标**：
* 将正样本与随机采样的 9 个难负例打乱顺序，分配编号 $1$ 到 $10$。
* 训练模型在第一个生成位置预测正样本的编号 $I_i^+$，即最小化负对数似然 $\mathcal{L}_{LM} = -\log P_\theta(I_i^+ | x)$，同时在训练中省略 EOS 结束符以保留模型在推理时输出完整排序列表的能力。
* 提供 **SWERANKLLM-SMALL（7B）**（基于 `CodeRankLLM`）和 **SWERANKLLM-LARGE（32B）**（基于 `Qwen-2.5-32B-Instruct`）两个版本。

---

## 四、 实验与分析（Experiments & Analysis）

1. **SWE-Bench-Lite 与 LocBench 全面领先**：
   * 在 SWE-Bench-Lite 上，`SWERANKEMBED-LARGE` 单独检索的函数级 Acc@10 达 **82.12%**，已超越使用 Claude-3.5 的 LocAgent（77.37%）。
   * 叠加 `SWERANKLLM-LARGE (32B)` 重排后，文件级 Acc@1 达 **83.21%**，函数级 Acc@5 / Acc@10 飙升至 **81.39% / 88.69%**，全面超越所有开源及闭源智能体基线（甚至在多个指标上优于 GPT-4.1 重排）。
   * 在 LocBench 上，函数级 Acc@10 / Acc@15 达到 **71.25% / 76.25%**（比 LocAgent+Claude-3.5 高出逾 15 个百分点）。
2. **极致的推理成本与延迟优势**：
   * 单实例推理成本：`SWERANKLLM (7B)` 仅需 **$0.011**，`SWERANKLLM (32B)` 仅需 **$0.015**，性价比（Acc@10 / Cost）高达 **79.0**，较 LocAgent 提升 **6 倍**以上。
   * 推理延迟：本地单张 A100 部署 `SWERANKLLM (7B)` 平均仅需 **12.5 秒**，比 LocAgent（85.3 秒）快约 **7 倍**，输入 Token 消耗减少约 3 倍。
3. **复杂场景与跨语言泛化**：
   * 在词汇重叠度极低（ROUGE-1 $< 0.1$）或语义相似度较低的困难样本上，SWERANK 依然显著优于多跳推理的 LocAgent。
   * 尽管仅在 Python 数据上训练，在涵盖 9 种非 Python 语言的 **SWE-Bench Multilingual** 上，`SWERANKEMBED-LARGE`（50.85% Acc@10）依然超越了闭源的 Gemini-Embedding（47.44%）。
4. **下游修复提升**：接入 SWE-Fixer 修复流程后，SWE-Bench-Lite 的 Repair Pass@1 从 21.0%（原生）和 22.6%（LocAgent）提升至 **24.5%**（逼近 Oracle 上限 25.9%）。
