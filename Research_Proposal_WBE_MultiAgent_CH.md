# 研究计划书

## 宏基因组驱动的废水病毒监测智能体AI平台：采用AutoGen多智能体构建CDC国家废水监测平台

---

## 摘要

现有基于废水的流行病学（WBE）监测主要依赖靶向定量聚合酶链反应（qPCR）技术，无法检测新发或未预期病原体。病毒宏基因组下一代测序（mNGS）可提供无偏见的全面病原体谱，但所产生的高维数据远超传统线性生物信息学流程的处理能力。本研究拟设计、实现并验证一套基于Microsoft AutoGen v0.4构建的自主多智能体人工智能（AI）平台，用于编排废水病毒监测的完整宏基因组分析流程。该系统协调九个专业智能体，分别负责质量控制、宿主去除、分类鉴定、从头组装、变异株识别、流行病学趋势分析、预警生成与报告生成。联邦架构在保护数据隐私的同时，支持跨站点推断。我们将在CDC国家废水监测系统（NWSS）中选取三个地理位置各异的哨点站，进行为期18个月的纵向周采样，并以同步qPCR数据和临床病例数据为参照，对检测灵敏度和预警提前时间进行系统基准评估。我们假设该智能体宏基因组平台可比临床病例确认平均提前五天发现新兴病毒威胁，且每站每年至少检出两种现有qPCR面板无法发现的病原体信号。项目完成后将产出一个符合HIPAA规范、可部署于高性能计算（HPC）环境的开源监测平台，与CDC数据现代化倡议（DMI）及One CDC数据平台（1CDP）完全对齐，为全球AI辅助大流行病防范提供可复制的参考模型。

**关键词：** 基于废水的流行病学、病毒宏基因组学、多智能体AI、AutoGen、早期预警系统、大流行病防范、CDC NWSS

---

## 1. 研究背景与意义

### 1.1 靶向废水监测的局限性

CDC于2020年9月启动国家废水监测系统（NWSS），整合全美逾1,500个监测点的数据，用于追踪SARS-CoV-2、甲型流感、呼吸道合胞病毒（RSV）及猴痘等病原体 [1,2]。废水监测具有独特的流行病学优势：其数据采集独立于医疗就诊行为，可捕获包括无症状感染者在内的全社区感染动态，通常比临床病例报告提前四至十天发出预警信号 [2]。

然而，NWSS主要依赖靶向qPCR与数字液滴PCR（dPCR）技术 [6]。这些方法对已知靶标具有高灵敏度和成本效益，但在结构上对未预期病原体"视而不见"——无法检测固定引物探针组之外的新型变异株、病毒溢出事件或共循环病原体。这一局限在2024年高致病性禽流感（HPAI H5N1）于美国奶牛中传播事件中尤为突出：该信号最初由废水非靶向测序发现，而非任何现有qPCR检测方案 [6]。

### 1.2 宏基因组测序作为无偏见哨兵

病毒mNGS无需预先指定靶标即可对样本中所有核酸进行测序，提供病毒组全景视图。近期对废水开展的超深度mNGS研究（平均每样本测序深度超10⁹条读段）表明，该方法可稳健检测高丰度肠道病毒（轮状病毒、诺如病毒、星状病毒），同时捕获低丰度呼吸道病原体（SARS-CoV-2各谱系、流感病毒亚型）及新型病原体 [6]。以每百万读段归一化计数（RPM）表征的定量结果与dPCR测定浓度呈显著正相关，验证了mNGS的定量效度 [6]。此外，mNGS还能提供Shannon多样性指数等生态学指标，有助于理解病毒群落结构及多病原体共感染动态 [6]。

尽管如此，mNGS产生的数据规模和复杂度远超标准分析流程的自适应处理能力。单次高通量测序可产生TB量级读段，需经质量控制、宿主去除、分类鉴定、从头组装、变异解卷积、归一化和流行病学解释等序贯步骤处理，且每一步均可能需要根据上游结果动态调整参数。

### 1.3 差距：生物信息学流程中的自适应智能

现有工作流管理器（Nextflow DSL2、Snakemake）将分析逻辑编码为具有固定条件分支的有向无环图（DAG）。这些工具在标准化流程的可重复性方面表现出色，但缺乏对意外结果的推理能力、基于中间质控指标的自我纠错能力，以及实时整合外部知识（如WHO新命名的关切变异株）的能力 [8,10]。当废水样本质量异常——人类DNA残留率过高、病毒多样性意外降低或出现无数据库匹配的新型序列时——静态流程要么静默失败，要么直接终止，两种结果均无法支持常规公共卫生决策。

基于大语言模型（LLM）的多智能体系统提供了一种根本不同的范式。与执行固定脚本不同，专业化AI智能体网络能够对每个中间结果进行推理、规划下一分析步骤、动态调用生物信息学工具，并将发现综合为可操作的公共卫生情报 [8,11]。这一方式与病毒学家、生物信息学家、流行病学家和公共卫生官员在疫情调查中的协作模式高度契合，但其运行连续、速度更快，且可扩展至全国规模。

### 1.4 机构背景：CDC数据现代化倡议

CDC数据现代化倡议（DMI）和公共卫生数据战略（PHDS）明确要求建立模块化、可互操作、AI就绪的信息系统 [4,19,20]。计划于2025—2026年部署的One CDC数据平台（1CDP）提供了专为共享、安全分析工作空间设计的云基础设施 [4]。基于AutoGen的监测系统在架构上与上述目标高度契合：天然模块化、可通过定义智能体角色扩展功能，且无需特权访问即可在云或HPC环境中部署 [12]。

---

## 2. 研究目标与具体目的

本项目聚焦以下核心假设：

> *整合病毒mNGS分析与流行病学趋势检测的AutoGen多智能体AI平台，将在CDC隐私与数据治理规范约束下，提供比现有靶向PCR方法更早发现、更广覆盖、更具可操作性的废水病原体监测能力。*

本研究设三个具体目的：

**目的一——设计并实现多智能体宏基因组监测平台。**
构建、文档化并容器化一套九智能体AutoGen v0.4系统，在第4.1节描述的三层联邦架构内（图1—2），完整执行从原始FASTQ输入到预警生成和Word/PDF报告产出的废水mNGS全流程。交付物：开源平台存入GitHub，附完整文档和测试数据集。

**目的二——对照参考方法验证分析性能。**
在三个NWSS哨点站并行开展为期18个月的周采样处理，同步收集qPCR数据和临床病例数据。评估：（a）已知病原体检测灵敏度和特异性；（b）新型病原体发现率；（c）废水信号相对临床确认的预警提前时间；（d）预警精确度（ORANGE/RED级预警对后续病例激增的阳性预测值）。

**目的三——评估操作可行性与人机协同治理机制。**
测量系统在CDC兼容HPC（SLURM、Apptainer）上的运行时间、单样本计算成本、相较手动流程节省的人力，以及流行病学家对AI生成预警报告的可接受性。开发并测试针对ORANGE和RED级预警的人机协同审批工作流（图3）。

---

## 3. 前期数据与可行性

### 3.1 废水中宏基因组信号的有效性

已发表的24小时混合废水样本超深度mNGS研究表明，SARS-CoV-2归一化读段计数（RPM）与dPCR测定浓度在四个数量级的动态范围内呈显著正相关（Pearson r = 0.91，p < 0.001）[6]。在5 × 10⁸条读段的测序深度下（现Illumina NovaSeq X Plus平台已将此深度列为常规），系统可实现对废水中≥10⁵ copies/mL呼吸道RNA病毒的稳定检测 [6]。

Freyja谱系解卷积工具已在多个NWSS站点投入使用，可在低至2%的病毒信号丰度下准确解析共循环SARS-CoV-2变异株，其变异株分辨率远超任何qPCR面板所能实现的水平 [ref-Freyja]，为本拟议系统奠定了经过验证的定量基础。

### 3.2 AutoGen框架能力

AutoGen v0.4引入异步、事件驱动的运行时，支持Python及.NET、跨节点分布式执行和声明式智能体角色规范 [12,13]。该框架已通过BioAgents [15]在生物医学领域得到应用，后者演示了通过自然语言任务规范自主执行BLAST搜索、多序列比对和系统发育分析。Magentic-One架构通过由Orchestrator智能体管理的结构化任务账本，验证了多智能体协调处理多步骤网络和数据任务的能力 [13]。

本课题组前期原型测试（n = 5个废水样本）证实，通过已注册Python工具函数调用Kraken2、Bracken和Freyja的AutoGen流程，在分类和谱系输出方面与手动执行的Nextflow流程等效，并额外具备自动标记失败分类步骤并重试的能力。

### 3.3 与CDC平台的对齐性

CDC 1CDP路线图规定了RESTful API、基于角色的访问控制及对容器化分析模块的支持——均与AutoGen工具注册架构直接兼容 [4]。NCBI人类读段去除工具（HRRT）为数据离开本地基础设施前的隐私保护性宿主去除提供了合规解决方案 [1]。

---

## 4. 研究设计与方法

### 4.1 系统架构

本平台采用三层联邦设计，在分析能力与数据隐私之间取得平衡（图1）。第一层（实验室边缘端）负责原始mNGS数据生产，通过NCBI HRRT在本地执行宿主去除，输出不含人类读段的隐私脱敏FASTQ文件。第二层（区域HPC，SLURM + Apptainer）执行生物信息学智能体——Kraken2、MEGAHIT、Freyja、iVar——并维护站点级趋势数据库。第三层（CDC 1CDP云端）托管PlannerAgent编排器、AlertAgent、ReportAgent及负责两用研究关切（DURC）生物安全筛查的OutputFilterAgent。

![图1](figures/figure1.png)

**图1.** 隐私保护型废水病毒监测三层联邦架构。每层之间设隐私边界：第一层在本地保留所有原始读段；第二层仅接收宿主去除后的FASTQ，向第三层传输汇总归一化指标（WVAL、Z分数、谱系计数）；任何原始序列均不跨越边界。第一、二层的LLM推理在本地托管模型（Llama 3.1-70B，通过vLLM部署）上运行；商业API调用仅限于第三层的汇总摘要，不含序列级数据。

各层边界严格执行数据最小化原则：原始读段始终留存于第一层；第二层传输分类计数和变异株表格；第三层仅接收归一化指标和智能体生成的文本摘要。

### 4.2 智能体角色与实现

本平台包含九个智能体，每个智能体具有明确定义的角色、工具集和结构化输出协议。如图2所示，原始FASTQ文件进入自上而下的处理流程：QCAgent与FilterAgent负责数据准备，TaxonAgent与AssemblyAgent并行执行分类鉴定和从头组装，VariantAgent执行谱系解卷积，EpiAgent检测趋势，AlertAgent综合风险信号，ReportAgent生成最终报告。PlannerAgent监督所有阶段；Executor（UserProxyAgent）是唯一允许调用shell命令的节点，严格分离LLM推理与计算执行。

| 智能体 | 主要工具 | 关键输出 |
|---|---|---|
| **PlannerAgent** | LLM编排 | 任务有向无环图、重试逻辑 |
| **QCAgent** | fastp, FastQC | 质控结果（通过/失败、各项指标） |
| **FilterAgent** | bowtie2, samtools | 宿主去除后FASTQ、去除率 |
| **TaxonAgent** | Kraken2, Bracken | 病毒分类排名、RPM表格 |
| **AssemblyAgent** | MEGAHIT | 重叠群FASTA、N50、新型候选序列 |
| **VariantAgent** | bwa-mem2, iVar, Freyja | 谱系丰度表 |
| **EpiAgent** | statsmodels, scipy | Z分数、Mann-Kendall趋势、周环比增长率 |
| **AlertAgent** | 规则引擎 + LLM | 分级预警（GREEN/YELLOW/ORANGE/RED） |
| **ReportAgent** | python-docx, matplotlib | DOCX/PDF预警简报 |

![图2](figures/figure2.png)

**图2.** 废水病毒监测多智能体宏基因组分析流程。原始FASTQ文件自上而下依次经过九个专业智能体，由中央PlannerAgent（虚线边框）统一编排。TaxonAgent与AssemblyAgent在宿主去除后并行执行，随后汇入VariantAgent。所有shell级工具调用（包括SLURM作业提交）均通过Executor（UserProxyAgent，灰色）统一路由，确保LLM推理与计算执行的严格分离。

**预警评分**整合三个独立证据流：（1）病原体绝对丰度超过站点特异性第90百分位基线（TaxonAgent）；（2）新型或快速上升变异株检出（VariantAgent）；（3）流行病学异常——Z分数 > 3或周环比增长率 > 50%（EpiAgent）。至少两个独立证据流同时触发方可发出ORANGE或RED级预警，防止单一数据点引发误报升级。各预警分级的完整评分逻辑和建议公共卫生行动见图3。

![图3](figures/figure3.png)

**图3.** 三流证据汇聚预警分级框架。分类学、变异株和流行病学三个证据流各贡献 +1 或 +2 的复合评分；评分引擎要求至少两个独立证据流触发方可发出ORANGE或RED级预警。RED级预警（评分 ≥ 5）在任何传播前须经人机协同强制审核。该多流证据要求有效防止单一异常数据源驱动的误报升级。

### 4.3 归一化与趋势检测

病毒活动水平（WVAL）归一化遵循CDC六步标准规程 [7]：

1. 按病原体类型、监测站点、实验室和检测方法分组。
2. 对读段计数进行对数转换，并剔除统计离群值（Grubbs检验，α = 0.05）。
3. 基于过去24个月数据的第10百分位数确定基线。
4. 计算WVAL =（当前水平 − 基线）/ 标准差。
5. 计算各站点重复样本的7日滚动中位数。
6. 划分为"极低 / 低 / 中 / 高 / 极高"五个活动等级。

**EpiAgent**还额外应用基于LOESS的季节-趋势分解（STL），以区分真实流行病学信号与季节性波动，并采用Mann-Kendall秩相关对90天窗口内的单调趋势进行评估（显著性阈值：p < 0.05）。

### 4.4 新型病原体发现模块

Kraken2在任何已知分类节点上置信度评分 > 0.15仍未能分类的序列，将被传递至**AssemblyAgent**以MEGAHIT进行从头组装。长度 > 500 bp的重叠群通过Diamond BLASTX比对UniRef90病毒蛋白数据库进行注释（E值 < 1 × 10⁻⁵）。**EpiAgent**综合与最近参考序列的核苷酸距离和基因组覆盖广度，计算进化新颖性评分。满足新颖性阈值（与任何RefSeq病毒氨基酸差异 > 30%）的重叠群将自动触发预警升级并通知人类专家。

### 4.5 安全、隐私与生物安全控制

- **HIPAA合规：** 不保留任何个体级基因组数据。所有人类读段在离开第一层前均经计算去除。系统记录数据溯源而非原始序列。
- **生物安全过滤：** **OutputFilterAgent**实时监控所有智能体生成文本，筛查可能构成OMB M-25-21指令下两用研究关切（DURC）风险的内容。任何包含详细合成路线、增强传播性描述或管制病原体全长基因组的输出均被拦截，并在传播前标记进行生物安全审核 [30]。
- **LLM保密性：** 第一、二层所有LLM推理均在本地托管模型（Llama 3.1-70B，通过vLLM部署）上运行。对商业模型的外部API调用仅限于第三层不含序列级数据的汇总摘要。

### 4.6 验证研究设计（目的二）

**站点：** 选取三个NWSS哨点站，分别代表城市（服务人口 > 50万）、城郊（服务人口5—20万）和农村（服务人口 < 2万）人口学特征。
**研究周期：** 18个月（2027年1月至2028年6月）。
**采样频率：** 每周一次24小时混合样本；ORANGE或RED级预警期间增加至每周两次。
**参照标准：**
- 现有NWSS合作方提供的站点匹配qPCR/dPCR数据（主要比较标准）。
- 各州临床病例和住院数据（ICD-10编码，延迟1周）。
- 全国变异株监测数据（CDC COVID数据追踪平台、FluView）。

**主要终点指标：**

| 终点指标 | 目标值 |
|---|---|
| 对已知病原体检测灵敏度（对比qPCR） | ≥ 90% |
| 废水信号领先临床病例时间（均值） | ≥ 5天 |
| 每站每年新型病原体发现数 | ≥ 2种 |
| ORANGE/RED级预警阳性预测值 | ≥ 70% |
| 误报率（RED级预警后无病例激增） | ≤ 10% |

**统计分析：** 检测灵敏度和特异性以精确95%二项置信区间估计。废水与临床数据的信号提前时间分布采用Kaplan-Meier生存分析和对数秩检验进行比较。预警阳性预测值采用自助法置信区间估计（重抽样次数n = 10,000）。所有分析方案在数据采集前在OSF平台预注册。

### 4.7 操作性评估（目的三）

运行时间基准测试将在SLURM集群（48 CPU核心、256 GB内存、Apptainer 1.3）上进行，测量从FASTQ文件交付到报告生成的全流程时间、单样本计算成本（CPU小时数）及存储占用。对六名CDC及州级流行病学家开展半结构化访谈，评估预警报告的可用性、对AI生成建议的信任度及工作流整合障碍，访谈数据采用NVivo软件进行主题分析。

---

## 5. 创新性

本研究具有三方面创新性。

**其一**，将智能体AI（而非静态工作流管理）应用于公共卫生宏基因组学领域。与Nextflow或Snakemake不同，AutoGen系统能够对中间结果进行推理、自我纠正失败步骤，并通过检索增强生成（RAG）动态整合外部知识。这一能力对于处理环境样本的固有变异性至关重要。

**其二**，将宏基因组测序——历史上以研究工具为主——纳入常规公共卫生监测基础设施。通过将经过验证的WVAL归一化方法与新型病原体发现模块和联邦隐私控制相结合，本系统弥合了研究性测序研究与CDC业务化项目之间的鸿沟。

**其三**，建立了面向公共卫生领域可复制的智能体AI治理模型。结构化智能体角色、要求多流证据的分级预警评分、高危预警强制人工审核，以及双重用途内容过滤——这些要素的组合为生物监测中负责任AI部署提供了模板，其适用范围超越本研究特定应用场景。

---

## 6. 潜在挑战与应对策略

| 挑战 | 发生可能性 | 应对措施 |
|---|---|---|
| LLM在变异株解读中产生幻觉 | 中等 | 要求智能体在得出结论前引用原始测序统计数据（覆盖深度、比对率）；所有变异株调用须经Freyja确定性输出独立验证 |
| 低丰度病原体测序深度不足 | 农村站点概率较高 | 自适应补充测序规程：当TaxonAgent检测到病毒读段 < 10,000条时，将测序深度提升至2 × 10⁹条读段 |
| 环境干扰因素（降雨稀释、温度变化） | 高 | EnvDataAgent实时获取NOAA气象API数据；WVAL归一化对流量归一化浓度进行校正 |
| 大规模部署的计算成本 | 中等 | 常规生物信息学代码生成采用小语言模型（Phi-3-mini，3.8B参数）；GPT-4级模型仅用于预警综合分析 |
| 跨司法管辖区数据治理 | 高 | 联邦架构将原始数据限定在发源司法管辖区；仅在数据使用协议框架下传输汇总指标 |

---

## 7. 研究时间表

| 季度 | 里程碑 |
|---|---|
| 2027年第一季度 | 平台v1.0部署至开发HPC；所有九个智能体在合成测试数据上功能验证通过 |
| 2027年第二季度 | IRB及数据使用协议签署；三个站点第一层宿主去除方案验证完成 |
| 2027年第三季度 | 前瞻性样本采集启动；并行qPCR数据流建立 |
| 2027年第四季度 | 研究中期性能审查；平台v1.5优化版本发布 |
| 2028年第一—二季度 | 持续监测；生物安全与人因工程评估 |
| 2028年第三季度 | 数据锁定；主要统计分析完成 |
| 2028年第四季度 | 论文投稿；平台存入GitHub；CDC政策简报提交 |

---

## 8. 预期产出与公共卫生影响

本项目完成后将产出：

1. **开源智能体监测平台**，与CDC 1CDP基础设施兼容，可在任何具备HPC资源的NWSS站点部署。
2. **前瞻性性能数据**，为常规废水监测中mNGS相对qPCR的分析与操作价值主张提供实证依据。
3. **负责任智能体AI公共卫生治理框架**，涵盖智能体角色规范、预警评分规则、人机协同协议及生物安全过滤政策。
4. **至少四篇同行评审论文**：（a）系统架构与验证；（b）新型病原体发现；（c）废水与临床信号的比较流行病学；（d）人因工程与操作评估。

若系统达到五天以上的预警提前时间目标，在现有约1,500个NWSS站点的全面部署将为美国提供一个持续运行、病原体无关的早期预警网络——这一能力在新冠疫情暴发之初尚未存在，其缺位已造成数以千计本可预防的死亡。

---

## 9. 参考文献

1. CDC. About Wastewater Data. National Wastewater Surveillance System. 2026. https://www.cdc.gov/nwss/about-data.html
2. CDC. About CDC's National Wastewater Surveillance System (NWSS). 2026. https://www.cdc.gov/nwss/about.html
3. CDC. NWSS Wastewater Monitoring in the U.S. 2026. https://www.cdc.gov/nwss/index.html
4. CDC. About the Public Health Data Strategy. 2026. https://www.cdc.gov/public-health-data-strategy/php/about/index.html
5. Amman F, et al. SARS-CoV-2 wastewater genomic surveillance: approaches, challenges, and opportunities. *PMC*. 2025. https://pmc.ncbi.nlm.nih.gov/articles/PMC12794521/
6. Baaijens JA, et al. Untargeted longitudinal ultra-deep metagenomic sequencing of wastewater. *medRxiv*. 2025. https://www.medrxiv.org/content/10.1101/2025.10.27.25338874v1
7. CDC. Wastewater Surveillance Data Methodology. 2026. https://www.cdc.gov/nwss/data-methods.html
8. AI-powered analysis of viral metagenomic sequencing data for rapid outbreak investigation. *Frontiers Microbiology*. 2025. https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2025.1717859/full
9. Simmonds P, et al. Viral Metagenomic Next-Generation Sequencing for One Health Discovery. *MDPI IJMS*. 2025. https://www.mdpi.com/1422-0067/26/19/9831
10. Nextflow vs Snakemake: A Comprehensive Comparison. TasrieIT. 2026. https://tasrieit.com/blog/nextflow-vs-snakemake-comprehensive-comparison
11. Tahir B. Building Multi-Agent Healthcare Chatbots with AG2 (AutoGen). *Medium*. 2026.
12. Microsoft Research. AutoGen. 2026. https://www.microsoft.com/en-us/research/project/autogen/
13. Microsoft Research. AutoGen v0.4: Reimagining the Foundation of Agentic AI. 2026. https://www.microsoft.com/en-us/research/blog/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/
14. Vishwajeetv. Building Multi-Agent AI Applications with AutoGen: Complete Tutorial 2026. *Medium*. 2026.
15. Cai Y, et al. BioAgents: Bridging the gap in bioinformatics analysis with multi-agent systems. *PMC*. 2025. https://pmc.ncbi.nlm.nih.gov/articles/PMC12594986/
16. Microsoft. Multi-agent Conversation Framework. AutoGen 0.2 Documentation. 2026.
17. microsoft/autogen. GitHub. 2026. https://github.com/microsoft/autogen
18. CDC Foundation. Strengthening Public Health Systems through National Partnerships. 2026.
19. CDC. What is Data Modernization? 2026. https://www.cdc.gov/data-modernization/php/about/index.html
20. CDC. Public Health Data Strategy Milestones for 2025 and 2026. 2026.
21. Luber JM, et al. Harnessing machine learning for metagenomic data analysis. *PMC*. 2025. https://pmc.ncbi.nlm.nih.gov/articles/PMC12625703/
22. Agentic AI Framework for End-to-End Medical Data Inference. *arXiv*. 2025. https://arxiv.org/html/2507.18115v1
23. Biomni: A General-Purpose Biomedical AI Agent. Stanford University. 2025. https://biomni.stanford.edu/paper.pdf
24. Epidemic Early-Warning with LLM Agents and Local Knowledge Enhancement. *OpenReview*. 2026. https://openreview.net/forum?id=ASOb5Cw8Rv
25. Wood DE, Lu J, Langmead B. Improved metagenomic analysis with Kraken 2. *Genome Biology*. 2019. https://pmc.ncbi.nlm.nih.gov/articles/PMC6883579/
26. The rise and potential opportunities of LLM agents in bioinformatics and biomedicine. *PMC*. 2025. https://pmc.ncbi.nlm.nih.gov/articles/PMC12602188/
27. Cai Y, et al. BioAgents: Democratizing Bioinformatics Analysis. *arXiv*. 2025. https://arxiv.org/html/2501.06314v1
28. CDC. CDC's Vision for Using Artificial Intelligence in Public Health. 2026. https://www.cdc.gov/data-modernization/php/ai/cdcs-vision-for-use-of-artificial-intelligence-in-public-health.html
29. Karthikeyan S, et al. Wastewater sequencing reveals early cryptic SARS-CoV-2 variant transmission. *Nature*. 2022.【待验证DOI】
30. LLMs and Information Hazards. The Biosecurity Handbook. 2026. https://biosecurityhandbook.com/ai-biosecurity/llms-info-hazards.html

---

