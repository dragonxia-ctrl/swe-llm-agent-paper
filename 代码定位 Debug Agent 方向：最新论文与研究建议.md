# 代码定位 / Debug Agent 方向：最新论文与研究建议

整理日期：2026-11-19。以下论文信息来自一次网络检索，多数只看了摘要和部分正文，没有通读；会议、期刊信息和下载链接请投稿前自行核对。

## 一、近期相关论文与下载地址

### 基线级工作（几乎必须复现或对比）

- **LocAgent**（ACL 2025）：图引导的代码定位 agent 框架。https://arxiv.org/pdf/2503.09089
- **OrcaLoca**（ICML 2025）：用基于优先级的调度、带相关性打分的动作分解和距离感知的上下文裁剪来提升 issue 定位精度。https://arxiv.org/pdf/2502.00350
- **CoSIL**（ASE 2025，据其 GitHub 仓库标注）：无需训练或预建索引，通过迭代式代码图搜索做函数级定位；论文报告用 Qwen2.5-Coder-32B 在 SWE-bench Lite / Verified 上的 Top-1 准确率分别为 43.3% 和 44.6%。https://arxiv.org/pdf/2503.22424
- **SweRank**：把定位建模为“检索 + 重排”，7B 模型的效果与基于 Claude-3.5 的 agent 相当或更好，成本低很多。https://arxiv.org/pdf/2505.07849
- **SweRank+**：扩展到多语言、多轮定位，其 SweRankAgent 的迭代推理稳定优于单次排序。https://arxiv.org/pdf/2512.20482
- **Agentless**：https://arxiv.org/abs/2407.01489
- **SWE-agent**：https://arxiv.org/abs/2405.15793

### 2026 年的新工作

- **GraphLocator**：用因果 issue 图（CIG）处理涉及多个函数、存在依赖的复杂 issue，并与 LocAgent、CoSIL、SweRank 对比；版本信息显示会议为 2026 年 7 月在蒙特利尔举行。https://arxiv.org/abs/2512.22469
- **SeeRepo / LLM Agents Can See Code Repositories**（2026.6）：把基于图的仓库结构上下文以视觉方式提供给多模态 agent，发现视觉工具在定位阶段最有效，token 成本约降低 25%。https://arxiv.org/abs/2606.14061
- **Retrieval-Oriented Code Representations in Agentic Bug Localization**（2026.7）：在 Long Code Arena 和 SWE-bench Verified 上比较五种代码表示，发现表示方式同时影响定位效果和成本。https://arxiv.org/abs/2607.11046
- **SemNav**：以候选为中心的 issue 定位框架，先用确定性检索得到候选集，再由 LLM 定位 agent 持续修正。https://arxiv.org/pdf/2609.31176
- **A²Agent**：针对仓库级代码定位 agent 的动作感知强化学习，工具基础设施沿用 LocAgent。https://arxiv.org/pdf/2608.29831
- **Cost-Effective Repository Exploration for Agentic Issue Localization**：发表于 AgenticDev '26 研讨会，研究文件级定位中的成本与模型分配问题。https://arxiv.org/pdf/2608.29675
- **IssueExec**：测试驱动的 issue 定位方法（文档显示 2026 年 4 月被接收，具体期刊或会议未核实）。https://arxiv.org/pdf/2607.17286
- **LIDL**：面向 LLM 集成软件的缺陷定位，覆盖源码、prompt 和配置文件的交互边界。https://arxiv.org/abs/2601.05539

## 二、研究建议

### 1. 先找到差异化切入点

静态代码图加 LLM agent 的路线已被 LocAgent、CoSIL、OrcaLoca、GraphLocator 占了不少位置，再做“又一个图搜索 agent”很难让审稿人信服。相对有空间的角度：

- **动态信息**：让 agent 使用调试器、运行时 trace、测试覆盖等执行反馈来做定位，而不只是读代码。上面多数工作偏静态，IssueExec 是少见的测试驱动例子。
- **训练而不是提示**：用强化学习或可验证奖励训练小模型做定位（A²Agent 已在做，需要找到比它更明确的新点，例如奖励设计、数据合成）。
- **成本与效率**：SweRank 已说明检索重排在成本上有优势，“何时用 agent、何时用排序”的自适应调度是值得做的问题。
- **非 Python、多语言**：SweRank+ 指出已有方法多只在 Python 上训练或评测。
- **非 issue 描述场景**：只有崩溃日志、失败测试、性能问题，或 LLM 集成软件缺陷（LIDL 方向）。
- **评测本身**：定位基准是否有数据泄漏或记忆问题，不同粒度指标与最终修复成功率是否一致。

### 2. 评测设计要扎实

- 至少覆盖 SWE-bench Lite / Verified 和 LocBench，最好再加一个多语言或更新的基准。
- 报告文件、模块、函数三个粒度的 Acc@k，同时报告成本和下游修复成功率。
- 用多个骨干模型（含开源模型）验证泛化，不要只依赖单一闭源模型。

### 3. 做细致的失败分析

先完整复现 LocAgent 和 CoSIL，逐例统计它们失败的原因（多处修改、缺少调用依赖、issue 描述含糊等），用分类出的失败模式来驱动方法设计，论文动机会更有说服力。

### 4. 目标刊会要现实

软件工程方向的“顶级”主要是会议（ICSE、FSE、ASE、ISSTA）加 TSE、TOSEM 等期刊；偏机器学习的可以投 ACL / EMNLP、ICML、NeurIPS、ICLR。截稿时间每年变化，请到官网确认，并据此倒推实验周期。

### 5. 结合自身条件

本科阶段做这种级别的工作，最现实的路径是：

- 找到有软件工程或 LLM agent 方向的导师或合作者；
- 尽早确定算力预算（API 费用是主要成本，“低成本方案”本身就是合适的切入点）；
- 先投较小的会议或研讨会积累经验，再冲击更高级别。

## 三、下一步

如果能补充手头的算力、已有的想法或导师方向，可以把上面几个切入点收敛成一个具体的研究问题和实验计划。
