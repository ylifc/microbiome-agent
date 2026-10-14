# Pipeline 参考目录与规划契约

Microbiome 将流程方法、软件使用知识、服务器资源事实和生产合同分开保存。
`microbiome/workflows/<id>/workflow.yaml` 与 `GUIDE.md` 是可分发的方法参考，状态为
`CURATED_REFERENCE`；它们不表示该流程已在服务器跑通，也不授权安装、下载或执行。
通用入口见 [跨宿主安装指南](portable_integration.md)。

## 流程目录

目录覆盖细菌/古菌 MAG、宏基因组 contig、RNA 与细菌 MAG 联合、单菌 WGS、单菌 RNA、
DNA 病毒、RNA 病毒、宏基因组病毒、宏转录组病毒、病毒系统发育、细菌/古菌系统发育、
真菌 contig、真菌 MAG、RNA 与真核微生物基因组联合、宿主相关微生物去宿主，
并保留长读长/混合组装、比较基因组、shotgun profiling、功能注释、扩增子及移动元件路线。
当前包含以下 22 份版本化参考：

新增轻量入口可直接做本地核酸序列检查与统计，操作边界见下文。

| 参考流程 | ID |
| --- | --- |
| [本地核酸 FASTA/FASTQ 检查与基础统计](../microbiome/workflows/sequence-statistics/GUIDE.md) | `sequence-statistics` |
| [宏基因组细菌与古菌 MAG 重建](../microbiome/workflows/assembly-mag/GUIDE.md) | `assembly-mag` |
| [宏基因组 contig 与基因目录分析（不分箱）](../microbiome/workflows/metagenome-contigs/GUIDE.md) | `metagenome-contigs` |
| [宏转录组联合细菌与古菌 MAG 表达分析](../microbiome/workflows/metatranscriptome-bacterial-mag/GUIDE.md) | `metatranscriptome-bacterial-mag` |
| [单菌短读长 WGS 组装与分析](../microbiome/workflows/short-read-isolate-wgs/GUIDE.md) | `short-read-isolate-wgs` |
| [单菌转录组：参考引导与无参考组装](../microbiome/workflows/microbial-rna/GUIDE.md) | `microbial-rna` |
| [DNA 病毒序列组装、共识与质量评价](../microbiome/workflows/dna-virus/GUIDE.md) | `dna-virus` |
| [RNA 病毒序列组装与参考共识](../microbiome/workflows/rna-virus/GUIDE.md) | `rna-virus` |
| [宏基因组病毒 contig 识别与质量评价](../microbiome/workflows/virome/GUIDE.md) | `virome` |
| [宏转录组病毒发现与表达证据分析](../microbiome/workflows/metatranscriptome-virome/GUIDE.md) | `metatranscriptome-virome` |
| [病毒同源序列比较与系统发育](../microbiome/workflows/viral-phylogeny/GUIDE.md) | `viral-phylogeny` |
| [细菌与古菌系统发育及系统基因组学](../microbiome/workflows/bacterial-archaeal-phylogeny/GUIDE.md) | `bacterial-archaeal-phylogeny` |
| [宏基因组真菌 contig 识别与功能分析](../microbiome/workflows/fungal-metagenome-contigs/GUIDE.md) | `fungal-metagenome-contigs` |
| [宏基因组真菌 MAG 重建与真核质量评价](../microbiome/workflows/fungal-mag/GUIDE.md) | `fungal-mag` |
| [宏转录组联合真核微生物基因组表达分析](../microbiome/workflows/metatranscriptome-eukaryotic-genomes/GUIDE.md) | `metatranscriptome-eukaryotic-genomes` |
| [宿主相关与共生菌数据的宿主识别和去除](../microbiome/workflows/host-associated-depletion/GUIDE.md) | `host-associated-depletion` |
| [长读长与混合测序单菌基因组组装](../microbiome/workflows/long-hybrid-isolate/GUIDE.md) | `long-hybrid-isolate` |
| [分离株比较基因组与群体系统关系](../microbiome/workflows/comparative-outbreak-genomics/GUIDE.md) | `comparative-outbreak-genomics` |
| [非组装型 shotgun 物种组成与功能谱](../microbiome/workflows/shotgun-taxonomy-function/GUIDE.md) | `shotgun-taxonomy-function` |
| [微生物基因与蛋白功能注释](../microbiome/workflows/microbial-functional-annotation/GUIDE.md) | `microbial-functional-annotation` |
| [16S、18S 与 ITS 扩增子分析](../microbiome/workflows/amplicon/GUIDE.md) | `amplicon` |
| [移动元件与耐药相关序列的计算注释](../microbiome/workflows/mobile-amr/GUIDE.md) | `mobile-amr` |

实际 ID、版本和适用条件以运行时目录为准：

```bash
microbiome-service call catalog.list
microbiome-service call catalog.search --json '{"query":"宏转录组联合细菌MAG表达分析"}'
microbiome-service call catalog.show --json '{"workflow_id":"assembly-mag"}'
```

上面的命令格式与入口由 `microbiome.integrations.cli` 提供；完整选项见安装指南及 `--help`。
目录也可直接通过 Python 读取：

```python
from microbiome.workflows import WorkflowCatalog

catalog = WorkflowCatalog()
selection = catalog.selection_context("宏基因组真菌分箱")
reference = catalog.get("assembly-mag")
```

“单菌宏转录组”需要先澄清：纯培养单菌的 RNA-seq 属于单菌转录组；混合群落 RNA 属于
宏转录组。参考指南会要求确认这一点。RNA 病毒与 DNA 病毒、真核与原核基因组使用不同的
输入假设、组装/注释和质量评估规则。预检证明命令、接口和输出合同可运行，不自动证明
物种鉴定、差异表达或进化结论正确；正式生物学判断仍需与任务匹配的质量证据。

## 小任务直接执行

基础序列检查无需先建立科学项目或调用模型：

```bash
microbiome stats /absolute/raw/input.fastq.gz --output /absolute/results/new-stats
microbiome stats /absolute/raw/R1.fastq.gz /absolute/raw/R2.fastq.gz --paired --output /absolute/results/new-paired-stats
```

输出目录须不存在、父目录已存在，且在原始输入目录之外。支持核酸 FASTA/FASTQ 及 gzip；
核验记录格式、核酸字符、FASTQ 质量长度/字符和条件配对关系，输出记录数、长度与 GC，
保存 `stats.json`、`execution.json`、`execution.log.jsonl` 及输入/结果/源码指纹。
累计解压上限 256 MiB、总记录上限 200 万（双端两侧累计）；损坏、超限或配对错误返回失败。
GC 分母为 A/C/G/T/U，歧义另列。此操作不推断 Phred 编码、覆盖度、物种或差异表达。
服务操作为 `sequence.stats`，MCP 工具为 `microbiome_sequence_stats`。

这是已实现的有界计算操作；目录中 `executable: false` 表示参考文档本身不是可重放科学 DAG。
复杂分析仍需编译、资源核查、审批、小试及冻结流程；小统计不会改变项目科研批准状态。

## 本轮方法审查

已复核全部 21 份科学参考的入口、阶段与方法边界，补充第 22 份基础统计参考。
RNA 统计必须通过共同 feature 矩阵和样本设计进入 cohort；无参考表达比较先固定共同
转录本集合，当前编译器不支持 cohort 输出反向供 sample 模板消费。contig-only 入口可
直接分类/注释，缺少匹配 reads 时覆盖度与丰度标为未测定。备选工具、条件 CheckV/VADR
及其专用模型单列条件；CoverM 内部 mapper 仍需真实资源绑定。

审查与合成数据工程测试不等于 21 类科学方法均有真实数据端到端验收。数据库适配、
统计模型可识别性、真菌 MAG 质量、病毒模型覆盖、未知工具与多组学科学解释仍需逐项目小试。

## 模型实际收到什么

1. 输入解析先完成，后续复用持久 manifest，不重新猜测本地/远程来源。
2. `PlanningContext.collect()` 从当前目标或显式 `plan.workflow_ids` 选择参考；交给模型时保存全文快照。
3. `workflow_for_model()` 将选中的完整 YAML 元数据和 Markdown 正文交给初始规划器。
   软件历史仍使用独立、有边界的资源证据摘要，不会挤掉已选方法的正文。
4. 初次方案编译、资源/文献修订及编译错误修复都会再次收到对应完整文档。
   新增 workflow ID 会触发新一轮读取与编译；超过预算明确失败，不截断正文后冒充完整读取。
5. 编译器必须说明每份参考的采用情况；应用校验覆盖范围、理由和实际步骤绑定。
6. 向用户展示的方案包含这些决定。参考版本、内容指纹和证据快照随计划持久化，冻结时
   复制到 lock；生产阶段不会读取新版目录或调用规划器。

规划器还可以调用只读 `list_workflow_references`、`search_workflow_references`、
`read_workflow_reference` 和 `inspect_software_request`，补充方法或服务器软件事实。
非空选择不是自动科学背书：用户目标和输入适用性始终决定是否采用某条路线。

外部 Codex、VS Code、WorkBuddy 由宿主模型负责设计。它们通过 `planning.context`
取得同一套全文与证据回执，再将回执作为 `plan.workflow_reference_evidence` 随方案提交。
`plan.save` 检查回执属于当前项目、输入清单与参考版本一致；缺少对应回执时要求先读取。
这证明接口已提供文档，并要求方案给出可审查的采用说明，不能证明模型内部的推理质量。
外部接口不会启动另一套模型或要求宿主的模型账户凭据。

## 候选计划中的方法决定

例如，已经实际读取 `assembly-mag` 后，一个方案可以声明：

```yaml
workflow_ids:
  - assembly-mag
workflow_decisions:
  - workflow_id: assembly-mag
    disposition: adapt
    reason: 使用已提供的组装结果，从DNA覆盖度计算开始；不重复组装。
    step_ids: [coverage, binning, quality]
```

`step_ids` 必须存在于本次 `plan.steps`，这里的名称仅作格式示例。`use` 和 `adapt` 需要
实际步骤；`not_applicable` 需要具体理由且 `step_ids: []`。遗漏参考、重复 ID、虚构步骤和
编译后改变的参考指纹都会阻止提交。应用记录 `workflow_references` 和
`workflow_reference_evidence`，模型不应自行伪造这些来源字段。

## Skill 数量与完整清单

正式包共有 141 个 Skill：107 个软件资源、33 个数据库资源、1 个宿主薄 Skill。
逐项路径见[完整 Skill 清单](skill_inventory.md)。22 份流程参考不计入 Skill 数量；
资源 Skill 存在不代表服务器已经安装或 READY。

## 新软件与环境

指南中的软件列表是候选方法信息，只有本次选定的步骤才形成依赖请求。解析顺序保持为：
冻结 lock → 已验证 Registry → Skill 映射 → 所有已有 Conda 环境 → PATH/项目软件 →
Conda 元数据 → 已批准安装。已有环境名称不必与规划建议一致。

新工具须区分 `tool`、`software_family`、`environment.executable`、`package_candidates`、
`required_packages`、`channels` 与环境名/前缀。缺失 Skill 触发发现；本地与官方元数据都
无法证明包映射时，返回 `NEEDS_PACKAGE_MAPPING`，不能根据一个工具名字猜包并执行安装。
只读软件检查报告会区分 Skill 文档依据与本地 Conda 文件归属依据。

批准当前依赖计划后，引擎可创建独立环境并安装计划范围内的依赖。READY 注册要求实际
可执行文件、版本和帮助检查通过，且记录安装经验。生产使用验证过的准确前缀和运行方式，
不会依赖用户当前 shell 激活了哪个环境，也不会自动替换 READY 软件或在 base/root 安装。
新增依赖或修复需要回到允许变更的阶段，修改方案版本并重新审批、预检。

数据库仍在指定的持久根目录中，样本获取与参考数据库获取分别管理。指南中的数据库 ID
不等于合法的已就绪路径、许可或可下载发行版。

## 扩充和维护

新增目录包含一个 `workflow.yaml` 和一个 `GUIDE.md`。YAML 声明 ID、语义版本、别名与
选择条件、输入、必须澄清的问题、阶段、产物、资源、验收、局限和一手来源。
正文解释路线选择与结果解释。引用现有资源 Skill ID；未知资源标记发现需求。
不要放本机绝对路径、安装密钥、自动批准语句或一套适用于所有样本的固定参数。

目录是唯一的流程参考正文来源，宿主 Skill 通过接口按需读取。旧 `know_how` 中的 wrapper
示例保留供兼容使用，不能覆盖当前方法参考或 Registry/lock。方法说明、可编译候选计划、
测试通过以及具体项目的小试通过是不同层级的证据。

验证入口：

```bash
pytest -q tests/test_workflow_catalog.py tests/test_workflow_planning_references.py
pytest -q tests/test_workflow_dependency_installation.py tests/test_portable_integration.py
pytest -q tests/test_frozen_pipeline.py
```

工程测试采用合成数据和隔离安装替身；不把这些结果写成所有真实生信软件、数据库和科学
路线已完成端到端验收。
