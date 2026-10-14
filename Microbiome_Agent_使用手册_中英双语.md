# Microbiome Agent 使用总览与操作手册

**版本说明：** 根据 Microbiome 运行源码、流程目录与可视化技能整理，2026 年 10 月 14 日。面向研究者和计算生物学开发者。本文描述的是通用软件能力和接口；每个项目仍须检查自身版本、输入、软件、数据库、运行证据和科学验收状态。

## 总体使用思维导图

![Microbiome Agent 总体使用思维导图](mindmap.png)

建议按“先明确问题与样本单位，再解析输入，再确定方法与资源，先小试验收，再冻结生产，最后审计结果和图”的顺序使用。模型负责理解与提出候选设计，Microbiome 服务负责目录、状态、约束和执行接口；实际科学结论要由使用者结合证据判断。

## 1. Microbiome Agent 是什么

Microbiome 是供 Codex、VS Code、WorkBuddy 等宿主调用的微生物生物信息工具和工作流引擎。宿主模型承接自然语言对话；Microbiome 提供版本化 JSON 操作、流程目录、资源登记、项目状态、持久任务和可审计的执行生命周期。需要时，用户也可以显式连接已有的本地或网关模型服务。

普通安装不启动另一个模型，不读取宿主私有账户凭据，也不要求填写宿主 API 密钥。MCP 是模型宿主与工具之间的协议，MCP 服务本身不是模型服务。默认 HTTP MCP 只监听 `127.0.0.1`，不等于公网 API、远程调度器或 SSH 隧道。

### 能力证据要分层理解

| 证据 | 能说明 | 不能说明 |
| --- | --- | --- |
| `CURATED_REFERENCE` 方法指南 | 有适用范围、输入、步骤、限制和来源说明 | 不能证明工具已安装或流程可在本机运行 |
| 方案通过结构校验 | 方案字段和接口契约可接受 | 不能证明命令已经成功或结论正确 |
| 软件/数据库状态 `READY` | 已登记资源通过对应检查 | 不授权升级、搬迁或重建资源 |
| 项目 pretest 通过 | 指定输入和方案通过已列明的预检 | 不自动外推到新输入或全部生物学结论 |
| 冻结锁与生产记录 | 本次生产使用已审阅的参数、工具和数据库指纹 | 不能把变更的分析设计伪装成原流程重放 |

当前目录中存在 22 条 `CURATED_REFERENCE` 工作流参考；目录条目本身不等于本机已验证的可执行管线。发布包的 `catalog/` 记录打包时的实际 Skill、流程及哈希清单。

## 2. Agent 能覆盖哪些微生物分析

当前方法目录覆盖扩增子、shotgun 分类与功能、MAG、contig 基因目录、细菌/古菌系统发育、真菌 MAG 和 contig、单菌 WGS、混合与长读长组装、细菌/真菌/病毒宏转录组、DNA/RNA 病毒、病毒系统发育、微生物 RNA-seq、功能注释、宿主去除、移动元件和耐药相关注释、基础序列统计等。每一条路线都需要根据真实输入和研究目标选择；同一名称不保证所有样本都适用。

### 22 条流程参考索引

| 流程 ID | 适用问题 |
| --- | --- |
| `sequence-statistics` | 核酸 FASTA/FASTQ 的格式、配对和基础统计 |
| `amplicon` | 16S、18S、ITS 扩增子特征表和群落分析 |
| `shotgun-taxonomy-function` | 不组装 shotgun 的物种组成与功能谱 |
| `assembly-mag` | 宏基因组细菌与古菌 MAG 重建 |
| `metagenome-contigs` | 不分箱的 contig 与基因目录分析 |
| `microbial-functional-annotation` | 微生物基因和蛋白功能注释 |
| `bacterial-archaeal-phylogeny` | 细菌与古菌系统发育及系统基因组学 |
| `fungal-mag` | 真菌 MAG 和真核基因组质量评价 |
| `fungal-metagenome-contigs` | 宏基因组真菌 contig 识别和功能分析 |
| `short-read-isolate-wgs` | 单菌短读长全基因组分析 |
| `long-hybrid-isolate` | 单菌长读长或混合测序组装 |
| `comparative-outbreak-genomics` | 分离株比较基因组与群体关系 |
| `microbial-rna` | 单菌参考引导或无参考转录组 |
| `metatranscriptome-bacterial-mag` | 宏转录组与细菌/古菌 MAG 表达证据 |
| `metatranscriptome-eukaryotic-genomes` | 宏转录组与真核微生物基因组表达 |
| `metatranscriptome-virome` | 宏转录组病毒发现与表达证据 |
| `dna-virus` | DNA 病毒组装、共识和质量评价 |
| `rna-virus` | RNA 病毒组装与参考共识 |
| `virome` | 宏基因组病毒 contig 识别和质量评价 |
| `viral-phylogeny` | 病毒同源序列比较和系统发育 |
| `host-associated-depletion` | 宿主相关样本中的宿主识别与去除 |
| `mobile-amr` | 移动元件及耐药相关序列计算注释 |

以上均是方法目录项。只有具体项目经过输入解析、软件和数据库检查、方案审阅、预检与必要验收后，才可称为该项目可执行或已完成。

## 3. 从安装到连接宿主

运行端面向 Linux 或 WSL2，Python 源码包要求 Python 3.11 及以上。推荐安装在独立受管目录中：

```bash
python3 -m venv /work/microbiome-python
/work/microbiome-python/bin/python -m pip install /path/to/Microbiome/source
/work/microbiome-python/bin/microbiome doctor
/work/microbiome-python/bin/microbiome-service call catalog.list
```

离线发行包的 Python 次版本、系统和 CPU 架构须与构建记录匹配。`doctor` 是安装与接口自检，不代表每个科学软件和数据库都已就绪。

导出宿主配置会创建一个新目录供检查，不会擅自覆盖宿主配置：

```bash
/work/microbiome-python/bin/microbiome export --host codex --output /work/new-codex-adapter
/work/microbiome-python/bin/microbiome export --host vscode --output /work/new-vscode-adapter
/work/microbiome-python/bin/microbiome export --host workbuddy --output /work/new-workbuddy-skill
```

随后将生成的 MCP 配置合并到宿主现有配置，并把 `microbiome-workflow` Skill 安装到宿主支持的 Skill 目录。安装输出中的 `microbiome-visualization` 技能包提供图种选择、R/Python 配方和质量检查；它是可独立安装的指导资源，不会因为复制到仓库就自动进入所有宿主。

MCP 使用标准输入输出，或显式启动本机 HTTP 服务：

```bash
microbiome mcp
microbiome mcp --transport http --port 8765
```

HTTP MCP 地址为 `http://127.0.0.1:8765/mcp`。宿主配置应指向执行机器中的同一 Microbiome 运行时。MCP 服务与模型服务端口不同。

如果用户已经有 OpenAI 兼容的本地模型服务，可明确指定它：

```bash
microbiome endpoint --endpoint http://127.0.0.1:8000 --check
microbiome endpoint --endpoint http://127.0.0.1:8000 --project /work/study
```

该操作探测用户显式给定的模型地址；不会扫描本机端口、下载模型或读取其他应用的密钥。连接地址需能访问 OpenAI 兼容的模型列表和对话接口。

## 4. 建立项目并确认样本

为每个项目明确原始数据目录、数据库根目录、结果目录和项目根目录。数据库放在 Conda 环境之外，结果目录不能覆盖输入。

```bash
microbiome-service init \
  --project /work/study \
  --raw-data /data/study/reads \
  --database /data/microbiome-databases \
  --output /work/study-results

microbiome-service call project.inspect \
  --json '{"project":"/work/study"}'
```

初始化要求项目尚不存在；已有项目用 `project.inspect` 查看，不要重复初始化来“恢复”状态。输入默认本地优先：显式给出的本地文件、目录和清单是数据来源。文件缺失是路径或数据问题，不会自动变成 SRA 下载请求。

```bash
microbiome-service call input.resolve --json \
  '{"project":"/work/study","declaration":"/data/study/sample_manifest.tsv"}'
```

检查解析出的样本 ID、配对文件、样本数量、批次、分组、协变量和独立生物学单位。池化样本、技术重复和动物/患者个体需要如实记录。研究单位和组别未确认时，要求系统提问或只做描述性汇总，不能把技术重复误当独立重复。

## 5. 研究设计、讨论与方案审阅

大型分析开始前应先讲清研究问题、样本单位、数据类型、比较设计、预期图表和资源成本。对于会改变结论的选择，应让模型列出方案、备选、取舍、纳入规则、限制与小试标准，再由研究者确认。不要在昂贵生产已经启动后临时改变筛选、软件、数据库或统计参数。

服务接口可按以下顺序调用：

```bash
microbiome-service call requirements.propose --json '{"project":"/work/study","topic":"analysis_goal","value":{"question":"..."}}'
microbiome-service call planning.context --json '{"project":"/work/study","query":"...","workflow_ids":["assembly-mag"]}'
microbiome-service call plan.validate --file /work/plan.json
microbiome-service call plan.save --file /work/plan.json
microbiome-service call decision.inspect --json '{"project":"/work/study","topic":"analysis_plan"}'
```

上例中的文字和方案对象应替换成完整、真实的输入。`plan.save` 需要已确认的需求和输入 manifest，并要求保存本项目的流程参考证据。规划上下文会提供选中流程的正文、证据和本机资源信息。提交计划时，要为每条被采用或调整的流程写出理由与关联步骤；不适用的流程应说明理由。

一个可执行计划至少要有唯一步骤 ID、科学目的、软件/可执行文件、版本与环境、显式 argv 参数、输入输出路径、数据库、资源预算、依赖关系、成功条件和失败处理。模型生成的计划必须先检查文件身份、命令和数据库版本，不能只因为通过 JSON 格式校验就开跑。

## 6. 软件、数据库和审批

软件发现首先查看冻结锁、已验证资源登记、Skill 映射和本机已有环境，再讨论批准范围内的安装。不要看到 PATH 无命令就假设未安装，也不要把一个包名直接当成命令名。已登记 `READY` 的资源应复用；生产期禁止偷偷下载、升级或搬移数据库。

新增依赖先预览，再在当前计划版本上复核并批准：

```bash
microbiome-service call software.inspect --json '{"project":"/work/study","logical_tool":"iqtree3"}'
microbiome-service call dependencies.preview --file /work/plan.json
microbiome-service call dependencies.review --json '{"project":"/work/study"}'
```

每项决策都应查看完整的 `decision.inspect` 内容、指纹和该项目的授权方式。交互审批由操作者在终端按确切指纹确认；若项目使用用户明确配置的 host-delegated 模式，也必须保留真实审核材料和记录。已有有效审批可以复用，不要求对未变化事项反复确认。host-delegated 的记录是授权渠道，不等于系统认证了操作者身份。

## 7. 小试、冻结和正式运行

先用能回答关键风险的预检。纯工具行为探针、少量诊断子集、完整派生试跑和真实数据科学验收是不同层级，报告中不能混为一谈。预检要保留实际命令、日志、错误、输出数量、样本身份、资源测量和接受标准。小试通过后才生成冻结计划与资源锁，再明确批准全量生产。

服务的长任务通过稳定 `request_id` 提交，重复请求不会产生重复运行：

```bash
microbiome-service call job.start --json @job-request.json
microbiome-service call job.status --json '{"project":"/work/study","job_id":"<returned-job-id>"}'
```

真实 job 请求对象要引用已保存的 input manifest、计划/验证 ID、完整样本范围、批准的资源限制与唯一输出目录。已确认的完整派生预检可以显式禁止 fraction；普通 fraction 预检与完整派生预检应按当前接口契约设置。提交后按 `job.status` 返回的日志和状态检查，不要重新发起另一个逻辑任务来“看是否更快”。

正式生产按冻结锁和已确认参数执行。失败时先查看真实任务、子进程、日志、checkpoint、report 和 ledger。只有公开接口声明支持并且满足恢复契约时才能恢复；生产重放使用原 `run_id` 并经过状态门控。不要编辑引擎状态文件来伪造成功。

性能记录至少区分墙钟时间、累计 CPU 时间、CPU 利用率、峰值 RSS、I/O 和调度并发。CPU 核心繁忙不等于墙钟加速；64 线程不能自动说明比 32 线程快一倍。用相同输入、相同科学工具和等价工作负载比较，保留测量误差和算法差异。

## 8. 可视化技能和科学图表

`microbiome-visualization` 技能包含群落、系统发育、基因组/MAG、基因丰度、KEGG/NR/COG/arCOG、contig 邻域和 WGCNA 图的图种目录，附数据契约、R 或其他软件路线、解释限制和验收要点。包括 R/base、ggplot2/phyloseq/vegan、ape、iTOL、FigTree、Cytoscape、IGV、Python 大型树及功能图适配路线。文档记录的包路线需要在当前环境再次检查，不能据此声称这些 R 包都已经安装。

覆盖范围包括组成与多样性、稀释曲线、排序分析、差异结果、生态网络、纵向数据、MAG 质量与谱系、ANI/AAI、基因组地图、系统发育树和树注释、contig 基因箭头图、功能热图、基因到 KO/pathway/module 的贡献图、模块步骤完整度和多层功能关联图。大树可用 overview 与分页细节，保持所有叶身份和分支；不同支持度指标须分开显示。

绘图流程是：确认问题和输入表 → 校验主键与单位 → 给出小型布局草图 → 确认关键科学选择 → 选本地可用绘图路线 → 生成可追溯代码与数据表 → 渲染并查看真实图像 → 记录验收结果。图片文件存在或脚本退出码为 0，都不能代替图像内容检查。

## 9. 基因功能、TPM、contig 关系和 WGCNA

新扩展的功能组学指导把基因身份分为 gene family 与 contig locus，记录注释数据库和版本、命中与置信规则、TPM/计数/覆盖单位、样本 ID 及多重映射政策。KEGG、NR、COG 和 arCOG 的证据类型、ID 空间和层级不同；NR 描述不能直接当 KEGG Orthology，COG/arCOG ID 不能直接互换。

基因到 KO、pathway 或 KEGG module 是有版本的多对多映射。多数据库注释要保留逐库证据，避免每个命中都重复计数。要明确使用完整 membership 还是 fractional allocation，并保持 unknown 与零值分开。DNA 来源 TPM 描述相对基因丰度，不是 RNA 表达或代谢通量。代表基因家族的丰度不能复制给每个 contig 位点后再加总。

功能模块展示时至少分开：基因丰度、成员覆盖比例、模块规则完整度、样本差异和注释置信度。模块逻辑必须有特定数据库版本中 AND/OR/optional 步骤的解释。contig 上的物理相邻、样本间丰度相关和数据库知识关联是不同证据，不能据此推断操纵子、因果关系或代谢活性。

WGCNA 是加权共丰度/共表达网络分析，不只是一个绘图选项。输入矩阵必须标明样本和基因方向、TPM/计数转换、独立样本单位和元数据；完整核对矩阵与元数据 ID，避免把 `001` 和 `1` 合并。过滤规则需对组别盲法并记录，不能先根据差异结果挑选基因再拟网。需要审阅 signed/unsigned 网络、相关方法、软阈值扫描、模块切分、eigengene、基因 kME、性状关联多重检验和稳定性分析。

WGCNA 的样本量限制依研究结构评估。当前技能建议小样本网络默认仅作描述性探索，不当作已验证调控机制；例如 12 个独立笼混样难以支撑稳健网络推断。当前记录中的 R 4.4.1 环境缺少 `WGCNA` 包，故使用前要核对运行环境和依赖审批；技能文件中的代码路线并不表示本机已经运行过真实网络。

## 10. 常见接口和状态查询

| 目的 | 服务操作 |
| --- | --- |
| 目录列表/搜索/查看 | `catalog.list`、`catalog.search`、`catalog.show` |
| 项目查看 | `project.inspect` |
| 软件只读检查 | `software.inspect` |
| 输入解析 | `input.resolve` |
| 研究目标 | `requirements.propose` |
| 流程完整上下文 | `planning.context` |
| 计划校验与保存 | `plan.validate`、`plan.save` |
| 依赖预览与复核 | `dependencies.preview`、`dependencies.review` |
| 审批材料 | `approval.inspect`、`decision.inspect` |
| 持久任务 | `job.start`、`job.status`、`job.cancel` |

JSON CLI 使用：

```bash
microbiome-service call catalog.search --json '{"query":"MAG quality assessment"}'
microbiome-service call project.inspect --json '{"project":"/work/study"}'
microbiome-service call decision.inspect --json '{"project":"/work/study","topic":"analysis_plan"}'
```

响应含 `schema_version`、`ok` 和 `result` 或 `error`；调用有返回不代表工作流成功。检查 `ok`、决策指纹和任务真实状态。

## 11. 使用边界和故障定位

- 缺输入：检查绝对路径、权限、manifest 和执行机器。不要自动切换到云端或 SRA。
- 找不到软件：查看 `software.inspect`、注册表和已有 Conda 环境；只有经审阅且在当前依赖范围内才进入 bootstrap。
- 数据库状态异常：停在资源核验阶段，先检查来源、版本、指纹和索引完整性。
- 预检失败：根据真实日志和输出诊断原因；保留失败产物，按新 request_id 或公开恢复契约处理。
- 输出图难读：记录实际图像问题，调整独立渲染布局；不要改动运行中的科学分析或支持度。
- 结果文件缺失：核对 job 状态、步骤输出、报告与依赖闭合，不以进程退出码推断整条流程通过。
- 样本重复或池化：依据生物学实验单位和采样设计处理，不能靠绘图或统计软件增加独立重复数。

安装、流程参考、真实预检、冻结生产和图像验收必须分别报告。真实数据的筛选、统计、分类、系统发育或代谢解释应引用对应输入、参数、数据库版本、样本身份和局限。

## 12. English overview and quick use

Microbiome is a host-callable microbial bioinformatics workflow service. Codex, VS Code, or WorkBuddy supplies the conversation model; Microbiome supplies versioned JSON/MCP tools, workflow references, project state, resource inspection, durable jobs, and a controlled pretest-to-production lifecycle. An OpenAI-compatible model endpoint can be connected only when the user explicitly provides it.

The current catalog contains 22 curated workflow references across amplicons, shotgun profiles, bacterial/archaeal/fungal MAGs and contigs, isolate genomes, RNA-seq and metatranscriptomes, viruses, phylogeny, functional annotation, host depletion, mobile elements/AMR, and sequence statistics. A curated reference does not mean that its software is installed or that the complete route has been validated on the current data.

```bash
bash install.sh --prefix /work/microbiome-runtime
/work/microbiome-runtime/bin/microbiome doctor
/work/microbiome-runtime/bin/microbiome export --host codex --output /work/new-codex-adapter
/work/microbiome-runtime/bin/microbiome-service init --project /work/study --raw-data /data/reads --database /data/db --output /work/results
/work/microbiome-runtime/bin/microbiome-service call project.inspect --json '{"project":"/work/study"}'
```

Resolve and review local inputs first. State the biological unit, groups, design, intended outputs, alternatives, limitations, and a measurable pilot. Inspect the complete plan and dependency decisions, validate a representative pretest, review the real output and resource evidence, freeze the accepted contract, then explicitly start production. Track long jobs with `job.start` and `job.status`; use only documented recovery paths.

The separate `microbiome-visualization` skill documents R/Python and GUI routes for community ecology, phylogeny, MAG/contig/gene views, KEGG/NR/COG/arCOG, TPM, and WGCNA. It preserves exact IDs and units and distinguishes DNA abundance from RNA expression, module membership from activity, physical contig adjacency from statistical association, and curated examples from installed packages. WGCNA needs adequate independent samples; in the current 12-cage example it should be described as exploratory. The current R 4.4.1 environment lacks the WGCNA package, so no real network fit is claimed.

## 13. English scientific and operational boundaries

Local input paths take precedence. A missing local file is an input problem, not permission to download from SRA. Sample data acquisition and reference database acquisition are separate. Reuse verified `READY` software and databases; never install, download, relocate, or silently change scientific parameters during frozen production. Keep raw data, databases, credentials, user-home state, and private model configuration out of public software releases.

The quality of evidence is layered: a reference is not a validated plan; schema validation is not execution; a successful command is not full workflow acceptance; and an attractive figure is not scientific verification. Record input hashes, software/database versions, exact parameters, sample identities, logs, execution status, uncertainty, and visual review. Distinguish descriptive, diagnostic, pilot, and production results.

## References within the repository

- [Current software source and capability catalog](../../README.md)
- [Microbiome workflow skill](../../microbiome/integrations/templates/skills/microbiome-workflow/SKILL.md)
- [Visualization skill and figure atlases](skills/microbiome-visualization/SKILL.md)
- [Workflow catalog summary](../workflow_catalog.md)
- [Portable release and verification guide](../packaging.md)
