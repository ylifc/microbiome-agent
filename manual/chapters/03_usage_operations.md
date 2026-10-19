## 日常使用：先确定入口、项目和事实

本章以公开发行版 **Microbiome 0.2.5，r10 产品代码** 为基准。所有目录都是运行后端的计算机上的目录；浏览器所在电脑、聊天附件所在位置和计算服务器目录可能是三个不同位置。首次使用时，先确认实际后端版本，再提供项目路径和任务目标。以下 `/srv/microbiome-demo/…` 均为示例占位路径，必须替换为你实际拥有权限的目录；不表示安装后自动存在。

这个 agent 把自然语言需求转换成可审阅方案，再由工作流引擎执行经过验证的合同。你与模型的聊天是需求来源，持久输入清单、需求记录、已批准方案、资源注册表及冻结锁是阶段推进的事实来源。模型说“已经完成”不等于任务完成；应看到对应阶段的持久回执、真实命令记录和验证报告。

### 三种操作入口的关系

| 入口 | 谁组织规划 | 你实际使用的界面 | 任务查询方式 | 适用场景 |
| --- | --- | --- | --- | --- |
| DeepSeek Harness 原生 Microbiome 预设 | 原生工作流调用宿主当前模型 | 正常聊天；agent 调用 `microbiome_workflow` 等工具 | 宿主 `job_output`、`job_list`、`job_kill` | 已安装 Harness 的日常分析、会话式审阅和恢复 |
| MCP 适配 | 接入 MCP 的宿主 agent 组织步骤和方案 | 宿主的 MCP 工具，例如 `microbiome_plan`、`microbiome_job_start` | `microbiome_job_status`、`microbiome_job_cancel` | Codex 等外部宿主通过正式服务接口访问引擎 |
| JSON CLI | 操作者或调用程序组织请求 | `microbiome-service call 操作名 --file 参数文件` | `job.status`、`job.cancel` 等版本化操作 | 运维核查、自动化集成、可复核的结构化调用 |

三者共享重要的后端合同，但**函数名、参数和作业系统并不相同**。原生 `microbiome_workflow` 没有 `mode`、`prompt`、`approved` 参数。MCP 工具也没有一个与原生工作流完全同形的总入口；它提供解析输入、提议需求、保存计划、审阅决策和启动作业等细分工具。不要把另一个入口的示例参数直接复制到当前工具。

JSON CLI 的 `call` 不会自动替你完成整段会话。它执行指定操作、返回结构化结果。把一次 `plan.validate` 成功当作分析完成，或把 `job.start` 返回编号当作生产成功，都是使用错误。

### 每次新会话的推荐开场

对原生 Harness，可以直接发送：

> 请先检查当前实际加载的 Microbiome 后端、插件和契约是否兼容，然后读取本会话已有项目绑定。已有路径和目标请继续沿用；只列出仍缺少且确实影响下一步的事实。

agent 应先调用无参数 `microbiome_capabilities`。重点看 `engine_version`、协议版本、契约摘要、`features`、`compatibility.compatible` 和不匹配字段。安装目录里存在某个新文件，不能证明当前会话已经加载新插件。升级后应新建会话并选择对应预设；若能力检查报告组件不一致，应先修复部署，继续猜参数通常只会增加无效重试。

项目已绑定后，可发送：

> 请读取项目状态和维护摘要，说明当前阶段、输入是否已确认、当前阻塞、已经保留的成功事实，以及下一步是系统操作还是需要我提供新信息。

这应产生状态查询或维护只读结果，而不是重新启动整个分析。原生状态工具无参数，读取的是本会话绑定项目；MCP 的同名状态工具需要明确 `project`。

### 新项目要交代什么

最少需要明确原始数据目录、持久数据库目录、结果项目目录和分析目标。建议一次提供以下信息：

| 信息 | 为什么需要 | 一个合适的表述 |
| --- | --- | --- |
| 原始输入 | 确定输入来源和只读范围 | “输入是 `/srv/project/raw` 中我指定的文件，以清单为准。” |
| 持久数据库根 | 发现、验证和复用已有资源 | “已有数据库放在 `/srv/reference`，先检查并复用已验证资源。” |
| 项目/结果目录 | 保存状态、审阅材料、运行结果 | “新建项目 `/srv/project/results/demo-01`。” |
| 必需交付物 | 防止方案偏离目标 | “交付清洗后的表格、汇总统计、可追溯报告和命令记录。” |
| 样本/分组/设计 | 决定统计解释是否合法 | “样本对应关系见清单；分组表不完整时先指出缺项。” |
| 已知资源约束 | 防止超出机器能力或依赖范围 | “优先使用已有环境；新增安装和大规模下载先列出范围。” |
| 允许的自动化程度 | 明确哪些决策由用户亲自作出 | “先展示方案，再由我确认预检；全量阶段另外确认。” |

下载目录和知识目录为可选项。未单独提供时，项目初始化有默认规则，但应以 `effective_paths` 回执为准。不要假定“没有明确下载目录”就绝不会写文件；默认目录也必须被正确解释和记录。

原生新项目工具参数如下。这是接口结构说明，不是需要普通用户亲自填写的配置：

```json
{
  "project": "/srv/microbiome-demo/project",
  "data_dir": "/srv/microbiome-demo/raw",
  "db_dir": "/srv/microbiome-demo/database",
  "download_dir": "/srv/microbiome-demo/download",
  "knowledge_dir": "/srv/microbiome-demo/knowledge"
}
```

其中 `download_dir`、`knowledge_dir` 可以省略。已有项目通常只需提供 `project`。同一原生会话一旦绑定项目，不能把它偷偷换成另一项目；另一个项目使用新会话。项目根与结果根在底层可以不同，所有查看和排错应从持久配置读取，不应依赖当前 shell 工作目录。

### 怎样提出容易成功且不丢目标的请求

优秀请求把目标、输入和约束写清楚，但不要求用户替模型填写内部计划 JSON。例如：

> 原始数据：`/srv/example/raw`。数据库：`/srv/example/database`。结果项目：`/srv/example/results/run-01`。请先核实本地输入清单与已有资源，再根据数据类型提出分析方案。必需交付物是每个输入的质量统计、汇总表、方法说明和真实执行报告。请保留完整输入集合；遇到预算或格式问题先修复技术问题，不要自动减少交付物。先给我可审阅方案。

如果只是维护，应直接描述维护目标：

> 请检查这个项目目前卡在哪里，读取完整的当前诊断，保留已经验证成功的输入和预检事实。先给出可执行的修复动作；不要把历史错误当成当前阻塞。

如果要查看文件，应明确“列目录”或“读取文本”：

> 请列出已绑定数据库根第一层目录，并读取当前数据库注册表中与本项目相关的条目。说明哪些只是发现的目录，哪些已经验证，哪些是可复用的 READY 资源。

这里 agent 有正式的受限读取接口，不应让你代读它已经能够访问的绑定目录；但它也不是任意文件浏览器。超出绑定范围、需要凭据或文件系统权限不足时，应报告具体缺口。

## 六个用户阶段与真实生命周期

用户看到的是六个主要阶段；后端记录更细的生命周期状态。细分状态是为了区分“计划已生成”“依赖已批准”“命令已成功”“冻结已完成”“生产已获准”等不同事实，而不是要求用户记住全部枚举。

### 阶段一：路径与输入确认

入口通常经过 `NEW → PATH_CONFIRMATION → DATA_DISCOVERY → DATA_CHARACTERIZATION`。显式传入并确认的路径可能使初始化直接进入发现阶段；并不需要人为再走一遍形式上的提问。

该阶段完成的证据包括：有效路径绑定、明确输入来源、样本/文件映射、文件存在性与格式检查、输入清单及对应确认记录。仅仅“找到若干 FASTQ”还不表示已确认它们都是本次样本；样本身份、配对、清单过滤及分组关系仍需一致。

本地优先规则贯穿整个项目：

1. 明确本地文件、目录或清单优先。
2. `sample_list.txt` 这类清单首先是输入声明，不因注册表缺项就变成下载请求。
3. 本地文件缺失分别判断不存在、无权限或挂载不可达；不能改口说需要远程获取。
4. 混合清单按每个样本分类，只处理明确声明的远程记录。
5. 已确认清单成为后续事实来源，后续聊天中出现的模糊路径不能覆盖它。
6. 原始数据获取与参考资源获取属于不同合同，不能互相代替。

工作区、数据库、输出、下载和知识目录的角色不能混淆。写“工作区 > /srv/results”并不等于授权把结果目录当作原始输入。相对路径必须按照明确的服务器工作区或已绑定根解析；agent 应复用回执中的规范路径，不反复自行手写拼接。

对于目录内存在多个合理候选的情况，系统应列出具体候选和差异；对于在允许范围内唯一且可以核实的解释，可以通过 `repair_input` 保存修正，不需要你学习内部 JSON。修正仍应保留原始声明和修订历史，并展示 `sample_change` 中新增、删除、变化的样本。

### 阶段二：需求、目标、资源和证据

常见状态为 `GOAL_DEFINITION → REQUIREMENTS_CONFIRMED → RESOURCE_DISCOVERY → EVIDENCE_REVIEW → PLAN_DRAFTING`。

目标需要从笼统动词转化成可验收交付物，例如“有一个汇总表”还应说明表的对象、列含义、样本覆盖和是否需要图形。必需交付物应被持久保存；技术修复时不能为了通过预算而删掉你已确认的目标。

资源发现应先检查已有能力。没有对应 Skill 并不代表软件不存在；已有工具也不一定装在模型预期的环境名中。后端区分逻辑工具名、软件家族、包名、可执行文件、环境名和 Skill ID。资源检查可记录版本、帮助输出、真实入口和指纹；不能仅凭目录名推断可用。

文献或指南证据也有状态。`未检索`、`检索失败`、`无结果`不同；无结果不是证明某种方法没有依据。审阅时应查看缺口影响哪些需求、是否阻塞和建议下一动作。人工文件复制这类软件验证可能得到非阻塞文献缺口；不能因此把检索状态伪装成成功。

### 阶段三：具体方案与依赖范围审阅

方案进入 `PLAN_CONFIRMATION` 后，应展示真实输入、步骤、参数、输入输出绑定、资源、验收断言和缺口。r10 新原生草稿使用 `structured_native_review_v1` 从结构合同生成正文；原模型文稿另存只读审计文件，不作为当前执行或授权依据。

审阅时按下面顺序核对，比单看“方案看起来专业”更可靠：

| 核对项 | 应能回答的问题 |
| --- | --- |
| 输入集合 | 样本数量、文件映射与已确认清单一致吗？有没有未经同意添加或删掉样本？ |
| 目标覆盖 | 每一项必需交付物由哪个步骤或引擎报告实现？ |
| 命令与参数 | 关键选择明确了吗？是否仍夹杂待回答问题或互相矛盾的备选项？ |
| 软件 | 逻辑能力对应哪一个真实可执行文件和版本？ |
| 数据资源 | 资源身份、版本、consumer 入口和指纹是否明确？ |
| 依赖变更 | 哪些只是复用，哪些要安装、获取、构建或修复？ |
| 预检策略 | 抽样对象是什么，是否保留必要样本与配对，是否适用于当前输入类型？ |
| 验收 | 哪些是实际执行的 `success_criteria`，哪些只是方法描述或期望？ |
| 权限与成本 | 本次批准覆盖什么，不覆盖什么；未知费用是否明确为未知？ |

依赖审阅通常进入 `DEPENDENCY_REVIEW`、`INSTALL_APPROVAL`。当前依赖计划是边界：发现额外依赖并不自动扩大已批准范围。需要新增依赖时应生成新版本并按现有授权机制审阅，而不是临时在生产命令里安装。

原生默认交互模式下，按系统展示的当前确认点给出真实回复。引导模式可能先要求“确认方案”，再选择预检比例，最后“确认执行”。“确认执行”只启动当前展示范围内的资源准备和预检；它不等于未来所有生产都获准。“暂停”“请解释一下”“修改某个参数”不应被当作确认。

### 阶段四：依赖准备与预检

进入 `BOOTSTRAP` 后，只准备当前已批准依赖范围。软件创建/修复允许的阶段为 `BOOTSTRAP` 或 `PRETEST_REPAIR`；健康环境和 READY 资源应复用。数据库与索引放在持久数据库根，不放进 Conda 环境目录。

预检过程常见 `PRETEST_PREPARATION → PRETEST_RUNNING`。对于读段型任务，引导策略通常是 5%–10% 数据预检；跨样本分析应保留全部必要样本，减少每个样本读数并保持配对。这个比例不是对所有数据类型机械适用的通用规则：**外部队列表格使用 `complete_external_cohort`，保留完整表格和全部记录组**。人工 TSV 示例正是这种模式。

失败时可进入 `PRETEST_REPAIR`。应先找出失败点属于命令、资源、输入、后置校验还是管理收尾，再确定修复范围。返回码为零并不证明全部输出合同成立；反过来，后置断言失败也不能抹去已经执行成功的命令事实。

系统支持的输出断言由编译器检查。不要在自然语言里要求一个名为 `sha256` 的断言，就假定当前引擎支持该类型。真正的哈希比对若未配置为受支持合同，需要独立核查并明确标注，不能冒充引擎自动验证。

### 阶段五：冻结、就绪和全量执行

预检成功后，后端会经过 `PRETEST_PASSED → PRETEST_FINALIZING → FROZEN → PIPELINE_LOCK_VALIDATING → PRODUCTION_READY → AWAITING_PRODUCTION_APPROVAL`。

旧项目可能保留兼容状态 `VALIDATED`。它与新状态名不同，但不能仅因名称旧就改写历史。

冻结保存的是可重放合同：流程、工具链、数据库入口与版本、参数、输入指纹、验证结果等。生产入口要再次核对运行条件和冻结记录。一个资源可以有多个合法 consumer 入口；预检、预览、冻结和执行必须使用同一套确定性规则，不能在最终命令中改用一个“看起来差不多”的通用目录。

全量审阅应针对当前冻结快照。真实确认后进入 `PRODUCTION_APPROVED → PRODUCTION_PREFLIGHT → PRODUCTION_RUNNING`，随后才可能 `COMPLETED`。预检成功、ready=true、批准成功和生产完成是四种不同事实。

冻结生产不调用规划器重新选择方法，不下载、重建或替换 READY 资源，不改变科学参数。发现资源漂移、锁不一致或前置条件失效，应停止并走重新验证路径。不能为了让旧锁通过而直接改文件里的指纹。

### 阶段六：结果验收和交付

最终交付应包含结果位置、实际完成的样本/步骤、失败或跳过项、验证范围、软件和数据库版本、执行日志及报告。结果解释必须来自真实输出，而不是从方案预测。

交付时推荐要求：

> 请按实际运行记录总结：最终状态、成功和失败步骤、每项交付物路径、执行过的验收检查、没有覆盖的检查、资源版本、报告与导出校验值。模型推断请与已验证事实分开。

若需要重新运行，先说明是“恢复原批次”还是“新建批次/新任务”。原生引导支持用户发出“恢复批次 `<batch-id>`”；编号必须来自实际回执。不要用重新提交同一句宽泛任务来代替恢复。

### 生命周期中的分支

| 状态/分支 | 实际意义 | 正确动作 |
| --- | --- | --- |
| `PAUSED` | 在生产确认附近明确暂停 | 查看当前待审阅材料；按实际意图恢复或取消 |
| `PLAN_MODIFICATION` | 需要修改方案 | 返回目标/方案流程，创建新版本并重新验证受影响部分 |
| `PRETEST_FINALIZATION_FAILED` | 预检之后的冻结/元数据收尾失败 | 查看失败类别；若 `requires_pretest=false`，只恢复收尾，不自动重跑预检 |
| `PRODUCTION_FAILED` | 生产实际失败 | 保留命令和输出事实，进入重新验证处理 |
| `REVALIDATION_REQUIRED` | 条件变化或失败导致旧验证不足 | 按受影响合同重做必要验证 |
| `CANCELLED` | 项目流程已取消 | 保留历史；若以后开展新目标，使用正式流程进入新目标阶段 |

## 原生 Harness 工具手册

原生预设暴露七个 Microbiome 工具，加上三个宿主作业工具。它不提供通用 shell 和任意服务器文件读取。下面列出的参数是 r10 插件实际契约；字段以当前能力回执为最终依据。

| 工具 | 参数 | 返回和注意事项 |
| --- | --- | --- |
| `microbiome_capabilities` | 无参数 `{}` | 实际后端/插件/契约能力及兼容性；不启动分析 |
| `microbiome_workflow` | 可选字符串 `project`、`data_dir`、`db_dir`、`download_dir`、`knowledge_dir` | 消费最近一条真实用户消息，通常返回宿主 `job_id`；不能传入代写确认 |
| `microbiome_project_status` | 无参数 `{}` | 当前会话绑定项目的状态、路径、报告和恢复指引 |
| `microbiome_maintenance` | `operation` 与按操作定义的 `arguments` | 九项维护操作；只读通常 inline，写入通常后台 |
| `microbiome_resume_planning` | `recovery_token`：当前回执中的 64 个十六进制字符组成的字符串 | 仅恢复规划到待审阅状态；不重放确认、不启动预检/生产 |
| `microbiome_database` | `operation` 与操作特定 `arguments` | 四项本地资源操作；不是下载工具 |
| `microbiome_checkpoint` | `action`、`checkpoint` 与该动作要求的指纹/理由 | 在真实已有受限宿主委托下审阅、确认、执行检查点 |
| 宿主 `job_output` | 由当前 Harness 的工具 schema 决定 | 查询已有后台任务输出；使用真实返回的作业编号 |
| 宿主 `job_list` | 由当前 Harness schema 决定 | 列出宿主作业，避免会话中断后重复提交 |
| 宿主 `job_kill` | 由当前 Harness schema 决定 | 取消所选宿主作业；不是全机进程清理工具 |

本说明书不猜测宿主 `job_*` 的可选字段，因为这些由宿主插件提供，可能随版本变化。agent 应读取当前宿主 schema。不要把 MCP 的 `microbiome_job_status(project, job_id)` 当作原生调用方式。

### 后台任务与 inline 返回

看到 `status=started` 和 `job_id` 时，保存编号并查询同一作业。不要连续再次调用工作流来“催一下”，否则可能引入新的用户事件或触发错误阶段。

看到 `delivery=inline`、`status=completed` 的短只读回执时，结果已经返回，不应再寻找不存在的后台作业。这包括常见维护摘要、目录/文件读取和检查点 `inspect`。

断线后优先查询已知作业。取消也应绑定具体作业，说明它只停止执行，不等于删除结果、释放所有存储或撤销历史批准。项目保留状态是恢复与审计所需材料。

### 检查点工具的三个动作

`checkpoint` 仅为 `pretest` 或 `production`。动作字段必须分别满足：

| `action` | 必填 | 不应出现 | 含义 |
| --- | --- | --- | --- |
| `inspect` | `action`、`checkpoint` | 审阅指纹、授权指纹、理由 | 返回当前完整审阅材料和指纹 |
| `confirm` | 前述字段，加 `review_fingerprint`、`authorization_fingerprint`、`reason` | 任意自创批准开关 | 对刚审阅材料记录已有委托范围内的决定 |
| `execute` | `action`、`checkpoint`、`review_fingerprint` | `authorization_fingerprint`、`reason` | 消费已确认决定，执行对应阶段 |

指纹必须来自当前真实回执，不能使用文档中的示意字符串。`confirm` 成功后按 `next_action` 执行；不要在同一预检确认后又无条件 `inspect` 一遍，使阶段发生变化后被守卫拒绝。材料变化时，应重新读取并重新审阅，而不是复用过期指纹。

这里的委托是宿主根据真实用户请求记录的、限定项目与范围的授权。检查点工具不能创建、扩大或伪造委托。没有有效委托时，使用默认真实用户确认流程。`reason` 应解释对当前材料的核对，不能只写“用户已经批准”来替代事实。

## 维护工具：九项操作与可操作示例

旧会话提到“八项维护操作”，不适用于最终 r10。当前共有 **九项**，新增 `planning_policy`。维护参数由同一机器契约提供给原生插件和后端，不能给 `inspect` 塞入 `path/file/scope/include`，期待它变成通用读文件接口。

所有维护操作还可有公共参数 `request_id` 和 `expected_revision`。前者为稳定请求标识，重试同一操作时保持不变；后者用于维护注册表的乐观并发校验。不要把 `expected_revision` 与输入修订号、策略修订号混为一谈。

### `inspect`：优先读取当前事实

| 字段 | 范围/默认 |
| --- | --- |
| `section` | `summary`、`input`、`artifacts`、`pending`、`permissions`；默认 `summary` |
| `limit` | 1–100，默认 20 |
| `cursor` | 上次返回的分页游标，可省略 |

原生调用示例：

```json
{"operation":"inspect","arguments":{"section":"summary"}}
```

先看当前阻塞、保留事实、有效路径和下一动作。需要操作范围与参数时读取 `section=permissions`；需要已登记产物时读 `artifacts`；需要恢复中断维护请求时读 `pending`。报告目录为空不代表整个输出目录为空。

### `list_directory`：列绑定根中的一个目录

必填 `root`，取值 `raw_data`、`database`、`output`、`download`、`knowledge`、`project`。`path` 是该根内的相对目录，默认 `.`；`limit` 1–100，默认 50；`cursor` 用于续页。

```json
{"operation":"list_directory","arguments":{"root":"database","path":".","limit":50}}
```

返回目录元数据、快照和分页信息。它可以列出未登记的用户文件；读取/发现不需要把文件冒充成 agent 自有产物。相反，列到了某文件也不赋予修改或删除它的权限。绝对路径和 `..` 路径穿越被拒绝。

### `read_artifact`：分页读取受限文本

必填 `root` 和 `path`。`root` 同上；`path` 是绑定根内相对文件路径。可选 `offset` 为脱敏文本中的字符位置，默认 0；`limit` 为最多返回 Unicode 字符数，1–65536，默认 4096；`snapshot` 为预期源文件 SHA-256；`cursor` 携带续页状态。

```json
{"operation":"read_artifact","arguments":{"root":"project","path":"config/database_registry.yaml","limit":4096}}
```

续页时仍应保留 `root`、`path`，并使用返回的 `next_cursor`。仅发送 `cursor` 会因必填字段缺失失败；这是最终实机验收中出现过的一次可恢复错误。使用游标时不再同时手算 `offset`。遇到 `SNAPSHOT_CHANGED`，重新读取当前文件，不把两个快照的片段拼成一个“完整原文”。

该接口读取经过凭据脱敏的文本，拒绝链接、秘密存储、二进制和超过限制的文件。因此它适合配置、注册事实、日志片段和报告，不适合把大型原始测序文件逐页搬进模型上下文。`limit` 是返回字符数，不是磁盘字节上限。

### `read_diagnostic`：区分当前与历史诊断

必填 `diagnostic_id`，当前支持 `input_resolution`、`compiler_failure`。`offset`、`limit`、`snapshot`、`cursor` 与文本分页规则类似，默认 limit 4096，最大 65536。

```json
{"operation":"read_diagnostic","arguments":{"diagnostic_id":"compiler_failure","limit":8192}}
```

先通过摘要了解有没有相应诊断，再按标识读取。诊断记录可能已解决或只是历史；判断应看当前状态、错误字段、是否仍阻塞及对应输入/候选指纹，而不是文件名。`compiler_last_failure.json` 存在并不自动证明发生过未解决的编译失败。

### `write_draft`：保存管理草稿

必填 `path`、`content`；可选 `role` 为 `draft` 或 `scratch`，默认 `draft`。

```json
{"operation":"write_draft","arguments":{"path":"next-actions.md","content":"记录当前阻塞、保留事实和下一动作。","role":"draft"}}
```

真实写入根来自权限快照：通常在输出根 `.agent-workspace/drafts` 或 `.agent-workspace/scratch`。这是管理文本，不是直接覆盖实验计划、冻结锁、批准或结果的入口。已有非自有文件不能因为同名就覆盖；已登记文件发生外部变化也需要核对。

草稿在冻结阶段仍可写，用来保存说明；scratch 在冻结或执行忙碌时受限。两者权限不同，不要为了绕过限制把科学输出随意标记成 draft。

### `repair_input`：修正输入声明而非删除项目

`declaration` 与 `selection` 二选一，可选 `expected_input_revision`。`declaration` 是真实输入说明文本；`selection` 可选 `files`、`directories`、`manifest`、`samples` 中一种来源，可加 `layout=single_end/paired_end`。

结构化来源含义如下：

| 来源 | 内容 | 注意 |
| --- | --- | --- |
| `files` | 明确文件路径列表 | 不用它冒充任意矩阵或输出目录 |
| `directories` | 允许发现的目录列表 | 仍需验证样本身份与布局 |
| `manifest` | 一个本地清单路径 | 与其他来源字段互斥 |
| `samples` | 每行含 `sample_id`，并提供 r1/r2、fasta 或 accession | 路径/远程标识不可混填；R1/R2不能指向同一文件 |

适合初学者的请求是：

> 原清单里路径的根目录写错了。请在已绑定原始数据根内核实唯一对应文件，保持样本 ID、样本集合和配对不变，通过输入修正接口保存，再报告实际验证结果。

`APPLIED` 只代表声明已保存，不能直接宣布输入修好。继续检查 `validation_status`、`resume_ready`、`sample_change`、`planning_recovery` 和 `next_action`。若状态是 `PENDING_CHANGE`，按回执处理待变更；受保护阶段不能靠输入修正偷换当前冻结合同。

### `cleanup`：清理登记的自有产物

使用非空 `artifact_ids` 或 `paths`，二选一，每次最多 100 项。优先使用 `inspect` 返回的真实 artifact ID。`action` 为 `preview`、`quarantine`、`restore`、`purge`，默认 `preview`。

```json
{"operation":"cleanup","arguments":{"artifact_ids":["从inspect取得的真实artifact_id"],"action":"preview"}}
```

操作顺序通常是预览清单、核对对象，再隔离；需要时恢复。永久清理 `purge` 只允许已登记 scratch。草稿可以隔离/恢复，导出文件不能走清理接口。即使是自有 scratch，仍被有效计划或检查点引用时也会返回 `ARTIFACT_IN_USE`。清理不会“认领”未知用户文件，也不接受模型自报所有权作为授权。

正在运行的任务与活跃子进程会阻止不安全清理。隔离要求同文件系统；跨设备失败时应保留原文件，不能暗中改成复制后删除。原始输入、健康环境、READY 资源、冻结锁、批准、注册表和科学结果不在通用清理范围。

### `export`：导出真实可见会话与项目回执

可选 `destination` 为用户明确指定的绝对 `.json` 文件名或已有目录。省略时使用后端默认导出位置。已有文件不会被覆盖。

```json
{"operation":"export","arguments":{}}
```

返回实际路径、覆盖范围、快照标识和校验值。原生宿主能提供可见正文、工具调用/结果及对应标识；MCP 若未提供宿主事件，导出应明确只有项目侧记录，不能称为完整聊天历史。

导出调用发生时，这次调用自己的工具结果及之后的最终回复尚未产生，可能出现 `pending` 或不覆盖最终答复。审计需要最终可见快照时，应在对话完成后再导出并检查 pending。推理内容、原始宿主数据库和全部原始事件不自动包含在内；人工摘要不能替代这些边界说明。

### `planning_policy`：有限技术预算调整

必填 `action` 为 `inspect`、`estimate`、`apply`、`rollback`。`apply` 需要 `expected_policy_revision` 和 `changes`；`rollback` 需要当前预期修订号和 `rollback_revision`。修改只支持三项呈现预算：

| 字段 | r10 默认 | 契约范围 | 含义 |
| --- | --- | --- | --- |
| `evidence_bytes` | 65536，即 64 KiB | 1024–262144 | 证据摘要片段预算 |
| `guidance_bytes` | 131072，即 128 KiB | 1024–262144 | 指南材料预算 |
| `history_bytes` | 16384，即 16 KiB | 1024–65536 | 模型可见历史预算 |

项目管理员可以设定更小的有效上限；以 `adjustable_fields` 和 `limits` 回执为准。接口不能修改模型、费用上限、科学参数、输入、资源、批准或锁。

推荐先查看再测量：

```json
{"operation":"planning_policy","arguments":{"action":"inspect"}}
```

```json
{"operation":"planning_policy","arguments":{"action":"estimate"}}
```

`estimate` 展示已记录请求的测量，不保证未来任何请求都能放下。检查 `measurement_available`、`measurement_matches_policy`、测量来源以及最终宿主请求。没有测量时应明确“尚无记录”，不能伪造估算为成功证明。

只有在允许阶段、没有冻结锁且理由成立时才应用修改。示意结构如下，`expected_policy_revision` 必须替换为刚读取的真实值：

```json
{"operation":"planning_policy","arguments":{"action":"apply","expected_policy_revision":0,"changes":{"evidence_bytes":98304}}}
```

这里的 `0` 仅示意参数类型；不能盲目复制到已有项目。策略变化后调用维护 `operation=inspect`、`arguments={"section":"summary"}`，检查 `planning_recovery.available=true` 并从 `planning_recovery.recovery_token` 读取当前 token，再恢复失败的构建环节。这里不是 `planning_policy` 的 `action=inspect`；后者不会返回恢复 token。切换策略、改时间戳或换 token 不会补充格式修复额度。

完整请求还受模型输入、输出预留、协议余量和宿主实际容量约束。`MODEL_CONTEXT_METADATA_MISSING` 表示宿主容量缺失；增加 evidence 字节无法解决它。未知 tokenizer 的估算明确标注为 UTF-8 字节上界估计，不等于精确 token 数。

## 资源操作：发现、验证、绑定的区别

原生 `microbiome_database` 有四项操作。名称虽然叫 database，也用于已有参考/索引资源合同；本章只讲通用本地资源管理，不提供特定病原任务的操作方案。

| operation | 参数 | 做了什么 | 没有做什么 |
| --- | --- | --- | --- |
| `discover` | 可选 `path`、`limit` 1–100、`cursor` | 列本地候选及可追溯身份 | 不根据文件名注册 READY |
| `inspect` | 可选 `resource_id`、`limit`、`cursor` | 查看已验证注册事实和可复用引用 | 不修复或重建资源 |
| `validate` | 必填 `candidate_id`、`resource_id`、`consumer`；可选 `manifest`、`binding_id` | 读取现有清单或已保存方案合同，核验全部声明组件并保存回执 | 不下载，不修改资源，不直接注册 READY |
| `bind` | 必填 `validation_id`、`registry_revision` | 在当前批准范围和允许阶段内登记已验证本地资源 | 不接受 `approved=true` 等模型提供开关 |

候选 ID 来自 discover，验证 ID 和注册表修订摘要来自 validate。不要手工编造 ID 或只验证一个 marker 就宣称资源完整。资源名称、版本、入口类型、所需组件和 consumer 合同必须能被证据支持。

`bind` 只在 `BOOTSTRAP` 或 `PRETEST_REPAIR`，并且当前依赖范围已批准时允许。方案尚未批准时返回 `RESOURCE_PHASE_DENIED` 是正常阶段保护，不是让用户去修改注册表的提示。已有批准仍有效且范围匹配时应复用，不需要为了每次查看再索要一轮确认。

READY 资源可通过 inspect 返回的 `reference` 放入方案中的资源引用，保留 `resource_id`、`fingerprint`、`consumer`。不要在短引用上叠加路径或参数覆盖，让执行解释改变。资源版本可为有来源的发布标签、日期等，不必强行写成语义版本；`UNKNOWN` 本身不能证明可注册。

## 权限：可以自动做什么，为什么某些动作仍会拒绝

权限由动作、阶段、绑定根、文件所有权、引用关系和批准范围共同决定。操作系统给予进程广泛权限，不意味着工作流会开放任意注册表修改。反过来，工具没有 shell 也不意味着不能读必要证据：目录、文本和诊断有独立正式入口。

### 按阶段理解自动操作

| 场景 | 通常允许 | 受限制的动作 |
| --- | --- | --- |
| 发现和规划 | 读取绑定证据、写草稿、保存检查点、有限技术预算调整 | 不能把发现候选等同批准；不能直接改科学结果 |
| 依赖准备 | 在已批准范围内发现、复用或创建/修复依赖 | 新依赖范围需新计划；不能破坏健康用户环境 |
| 预检修复 | 保存失败事实，修复获准依赖，处理自有失败暂存 | 不能删除审计记录让失败消失；科学变更仍需相应确认 |
| 冻结和生产 | 读取冻结资源、写本批次结果与日志、写管理草稿、导出 | 不能重新规划、改变锁解释、下载或替换 READY 资源 |
| 长任务运行中 | 状态、只读诊断、必要草稿和导出 | 清理、输入修正、scratch 写入或策略写入可能因锁与阶段拒绝 |

权限快照中 `planning_policy` 允许查询不代表 apply 一定允许；策略服务另查允许阶段和冻结锁。`repair_input` 可用也不等于当前受保护输入会立即被替换，必须看其专用回执。

### 写权限与删权限分开

读取现有用户文件无需 ownership；更新/清理自有产物必须有持久注册和未变化的指纹。路径看起来在 output 下面，不代表它都归 agent 所有。科学运行结果由流水线生命周期服务管理，维护草稿由 `.agent-workspace` 管理，两者不能混用。

后端使用安全路径访问、链接检查、文件类型和指纹核对，避免符号链接、硬链接或外部替换把有限授权扩展到其他文件。遇到 `UNSAFE_PATH` 不应通过关闭全部保护来解决；应核实真实路径、挂载和文件类型，选择符合当前合同的存储位置。

### 用户确认与宿主委托

默认交互确认适合首次任务和需要判断方法的项目。宿主委托适合用户已经明确授权限定目标、依赖范围和阶段的自动化。两者都必须对应真实材料和真实来源。

MCP 的委托配置是宿主声明，保存真实用户请求、来源、范围和允许 topic；它不是独立的人类身份认证。原生检查点使用既有委托，不提供配置授权接口。普通用户不需要把授权写成 JSON，也不应把“已经得到批准”加入草稿来推进引擎。

需要改变方法、样本集合、新增未批准依赖或费用范围时，应给用户具体可审阅差异。格式规范化、读取诊断、保存管理草稿、在范围内修复可核实路径不应反复打断用户。

## MCP 与 JSON CLI 的具体使用

### 查看能力和项目

安装后的 JSON CLI 可执行以下只读操作。示例路径需要替换，命令本身已按当前 argparse 与 dispatch 名称核对：

```bash
microbiome-service call runtime.capabilities --json '{}'
microbiome-service call project.inspect --json '{"project":"/srv/microbiome-demo/project"}'
microbiome-service call project.recovery --json '{"project":"/srv/microbiome-demo/project"}'
microbiome-service call catalog.list --json '{}'
```

参数较长时保存为 UTF-8 JSON 文件，再执行：

```bash
microbiome-service call project.inspect --file /srv/microbiome-demo/inspect-request.json
```

该文件内容示例为 `{"project":"/srv/microbiome-demo/project"}`。`--file -` 读取标准输入。CLI 成功退出码为 0，失败为 2；仍应检查返回 JSON 的 `ok`、`error` 和 `result`，不要只抓最后一行文本。

### 初始化一个真正的新项目

```bash
microbiome-service init \
  --project /srv/microbiome-demo/project \
  --raw-data /srv/microbiome-demo/raw \
  --database /srv/microbiome-demo/database \
  --output /srv/microbiome-demo/output
```

原始数据目录应已存在，四个路径应符合分离规则。已有项目不要重新 init；使用 project.inspect。原生 Harness 新项目流程会自行初始化，不需要先手工执行这一命令。

### MCP 工具分组

当前 MCP 共 22 个工具，完整机器签名另见本说明书附带 `reference/interface-signatures.json`。下表概括其职责，避免与原生七工具混淆。

| 分组 | 工具 |
| --- | --- |
| 能力与项目 | `microbiome_capabilities`、`microbiome_project_status`、`microbiome_maintenance`、`microbiome_planning_recovery` |
| 指南与资源 | `microbiome_catalog`、`microbiome_software_inspect`、`microbiome_database` |
| 输入 | `microbiome_resolve_input`、`microbiome_resolve_derived_input`、`microbiome_resolve_external_cohort_input` |
| 需求和方案 | `microbiome_propose_requirement`、`microbiome_planning_context`、`microbiome_plan`、`microbiome_dependencies` |
| 审阅与委托 | `microbiome_decision_review`、`microbiome_approval_mode`、`microbiome_record_decision` |
| 后台作业 | `microbiome_job_start`、`microbiome_job_status`、`microbiome_job_cancel` |
| 独立轻量功能 | `microbiome_sequence_stats`、`microbiome_visualization` |

MCP `microbiome_maintenance` 需要 `project`、`operation`、`user_message`、`message_id`，另可有 `arguments`、`session_id`、`host_events`。其中用户消息与 ID 应由宿主转交真实来源，不是模型编写批准文本的地方。`microbiome_database` 也要求类似的真实消息来源。原生插件则从会话提取这些字段，模型不直接传它们。

### 结构化规划的一般顺序

外部宿主通常先读取能力/项目，解析输入形成待审阅提案，确认输入与 dataset/target/analysis_goal，再读取 planning.context，生成候选、validate/save，审阅方案和依赖，最后提交获准作业。`plan.validate` 只校验候选；`plan.save` 保存供审阅；`dependencies.preview` 预览资源缺口；`dependencies.review` 使用已确认的保存方案。

支持的决策 topic 为 `input`、`dataset`、`target`、`analysis_goal`、`analysis_plan`、`dependencies`、`production`。先 `decision.inspect` 得到当前材料与 `decision_fingerprint`，再由实际操作者交互决定或使用有效受限委托。

操作者 CLI 的形式是：

```bash
microbiome-service decide \
  --project /srv/microbiome-demo/project \
  --topic analysis_plan \
  --action CONFIRM \
  --expected '<decision.inspect返回的完整decision_fingerprint>'
```

占位符须替换为实际指纹。该命令要求真实交互终端，展示材料并要求操作者输入包含短指纹的挑战文本。它不接受管道输入，也不是 MCP 批准后门。文档不提供自动代输确认的脚本。

### 后台作业管理

MCP 的 `microbiome_job_start` 接受 `project`、`operation`、稳定的 `request_id` 和可选 `parameters`。作业操作包括 `bootstrap`、`resolve_databases`、`pretest`、`finalize`、`production`，仍受各自批准与阶段保护。

JSON CLI 对应查询与取消：

```bash
microbiome-service call job.status --json '{"project":"/srv/microbiome-demo/project","job_id":"实际作业编号"}'
microbiome-service call job.cancel --json '{"project":"/srv/microbiome-demo/project","job_id":"实际作业编号"}'
```

不要把示意编号当作实际请求执行。提交结果不确定时，保持 `request_id` 不变并先查询。死去的 worker 应报告 interrupted；取消只针对已验证的本地进程组，不清理项目产物。`job.recover` 是 JSON 服务操作之一，但不是本版单独暴露的 MCP 工具；不要由名称相似性推断 MCP 上存在 `microbiome_job_recover`。

## 可复核的完整案例：46 字节人工表格复制

这个案例用于理解阶段、审阅、冻结、真实执行和报告，不涉及真实生物样本。最终 r10 实机验收已经运行过同类型任务；你在自己机器上重做会产生新的计划、指纹、作业号和回执，不能复制历史批准记录冒充当前验证。

### 输入与目标

人工文件的精确内容为以下三行，字段之间是真实 Tab，最后有换行：

```text
record_id	batch_a	batch_b
row_a	1	2
row_b	3	4
```

总计 46 字节。历史输入 SHA-256 为 `a1d586e36446e12c8024119db47646b958e9c345224761c0549aad19fe5d5f24`。两个数字列只是人工记录组，不是生物样本。

如需自行建立教学文件，可在自己的新目录运行下面的普通文件准备命令。它只创建目录和人工文本，使用独占写入，不覆盖已有文件：

```bash
python - <<'PY'
from pathlib import Path
base = Path.home() / "microbiome-manual-demo"
for name in ("raw", "database"):
    (base / name).mkdir(parents=True, exist_ok=True)
target = base / "raw" / "artificial-records.tsv"
with target.open("xb") as handle:
    handle.write(b"record_id\tbatch_a\tbatch_b\nrow_a\t1\t2\nrow_b\t3\t4\n")
print(target)
print(target.stat().st_size)
PY
```

随后把打印的绝对路径提供给 agent。若文件已存在，命令会报错；核对原文件即可，不要为重复演示而覆盖数据。

可以发送以下请求，替换其中路径：

> 请在新项目 `/我的目录/microbiome-manual-demo/project` 做一次工作流软件验收。原始目录为 `/我的目录/microbiome-manual-demo/raw`，数据库根为 `/我的目录/microbiome-manual-demo/database`。输入只有 `artificial-records.tsv`，它是 46 字节人工数字表，没有生物学含义。`record_id` 为行标识，`batch_a`、`batch_b` 是两个人工记录组。目标是用现有 `/usr/bin/cp` 的一个步骤把完整表原样复制到本次运行输出，保留所有行列。由引擎生成预检、冻结、生产和验证报告；不增加自定义科学程序。请先展示方案及实际支持的验收检查，再由我确认预检和全量阶段。最终区分引擎检查与独立字节比对。

这个自然语言请求适合说明目标，但原生工作流不应把任意 TSV 自动当成测序样本清单。最终实机验收采用了正式 `external_cohort_tables` 输入合同，并在运行前由真实宿主范围声明建立已确认需求。初学者若在原生入口遇到表格类型不支持，应让宿主通过 MCP/JSON 的 `input.resolve_external_cohort` 正式提议该输入，再由操作者确认；不能靠改扩展名伪装 FASTQ，或把表格内容写进普通样本路径清单。

### 外部表格输入怎样表达

通过 MCP 的 `microbiome_resolve_external_cohort_input` 或 JSON CLI 的 `input.resolve_external_cohort` 提交以下类型的数据。它只是输入提案，不是批准：

```json
{
  "project":"/srv/microbiome-demo/project",
  "local_roots":["/srv/microbiome-demo/raw"],
  "samples":[
    {"sample_id":"batch_a","metadata":{"synthetic":true,"biological_meaning":false}},
    {"sample_id":"batch_b","metadata":{"synthetic":true,"biological_meaning":false}}
  ],
  "cohort_artifacts":{
    "records":{
      "path":"/srv/microbiome-demo/raw/artificial-records.tsv",
      "format":"tsv",
      "schema":{
        "id_column":"record_id",
        "sample_columns":{"batch_a":"batch_a","batch_b":"batch_b"},
        "integer":true
      }
    }
  },
  "provenance":{"synthetic":true,"biological_meaning":false,"purpose":"workflow software acceptance"}
}
```

保存为参数文件后可使用 `microbiome-service call input.resolve_external_cohort --file <实际参数文件>`。随后读取返回的 input 决策材料，按真实交互确认。数据表需要至少一份明确映射全部声明记录组的矩阵；列名、行标识、非负数/整数约束和完整表格语义均要一致。

### 预期审阅内容

这个案例只需要一个复制步骤，复用现有 `/usr/bin/cp`，没有新增安装包和数据库。输入是完整外部表格，预检模式为 `complete_external_cohort`，不能抽掉其中一个记录组或仅复制 5% 的行而声称完成相同目标。

审阅材料应清楚列出真实命令入口、完整输入绑定、独立运行输出、支持的验收断言及引擎报告。输出存在、非空、TSV 可解析、表头及 ID 唯一性等检查与逐字节相同是不同性质的检查。若计划没有原生哈希等同性断言，必须另行核对并在报告中明示。

### 最终 r10 的真实历史结果

以下是已经完成的历史实机记录，不是用户重做后必然得到的数字：

| 项目 | r10 历史事实 |
| --- | --- |
| 宿主 | DeepSeek Harness 0.1.0-rc.12 原生服务和 agent-loop；没有操作 Electron 界面 |
| 模型路由 | `deepseek-official / deepseek-v4-flash`，真实模型 |
| 完成时间 | 2026-10-19 13:13:43–13:17:11，Asia/Shanghai |
| 最终生命周期 | `COMPLETED` |
| 内部规划模型调用 | 4 次，格式修复 0 次 |
| 外层助手步骤/工具调用 | 22 步、29 次工具调用 |
| 真实执行 | 1 次预检复制、1 次生产复制，总计 2 条命令，均 `SUCCEEDED`、exit=0 |
| 就绪检查 | ready=true，17 个区域通过 |
| 输出 | 两份复制输出与 46 字节原始输入一致，独立验收脚本核查 |
| 独立审计 | 49 项通过，结论为 `PASSED_WITH_FINDINGS` |
| 最终可见会话导出 | 覆盖 22 条可见助手消息，pending 为空，摘要一致 |

该运行有一次分页调用遗漏 root/path，后来按 schema 补齐并成功。它不是“整个过程中没有出现任何报错”。助手最后还把历史诊断文件名误读成编译失败、把宿主大整数显示精度问题误读成输入 mtime 漂移；原始持久记录否定了这两项推断。验收报告采用原始事实。

历史成功只验证了所覆盖的软件阶段，不证明真实序列抽样、多工具科学分析、参考获取、索引构建、任何新宿主版本或所有异常组合都已通过。没有宿主计价依据，费用为未知。未在实机额外发起同一请求的重复生产重放，因此幂等代码回归和单次真实执行证据应分别表述。

## 诊断、恢复和避免无效循环

### 先把四类成功/失败分开

1. **模型/规划层**：消息构建、上下文预算、候选格式、资源收敛。
2. **工具/传输层**：请求参数、宿主任务、超时、连接、取消。
3. **实际命令层**：可执行文件、真实参数、返回码、stdout/stderr、输出。
4. **验证/管理层**：后置断言、冻结写入、指纹、就绪、报告收尾。

“复制成功但后置断言失败”不是“命令没执行”；“预检通过但冻结收尾失败”不是“需要重新跑一遍预检”；“只读诊断参数错”也不等于“格式修复额度减一”。r10 已专门保留这些边界。

每次错误反馈最好回答四件事：失败在哪一层；自动做过什么；哪些事实仍有效；下一步由系统还是用户完成。如果要用户处理，给具体路径、缺少的元数据或待判断候选，避免一句“提高权限”“缩小范围”结束。

### 规划恢复 token 的含义

`recovery_token` 是当前有效事实的摘要，不是密码，也不是执行批准。它绑定任务、输入、资源注册事实、有效策略、消息构建器与相关合同代码、候选和检查点等。新证据可能产生新 token；只改日志时间戳不会合理地创造新机会。

推荐流程：

1. `maintenance inspect` 查看当前阻塞和 `planning_recovery`。
2. 只读取诊断要求的具体材料，避免把所有目录反复扫一遍。
3. 通过正式接口修正唯一可核实输入或允许范围内的策略。
4. 再次 inspect，确认 `available=true`，取当前 token。
5. 原生调用 `microbiome_resume_planning`，等待其返回的真实 job_id。
6. 对恢复后的方案重新审阅；恢复本身不授予预检或生产权限。

常见恢复状态：

| reason | 含义 | 应对 |
| --- | --- | --- |
| `PLANNING_RECOVERY_AVAILABLE` | 当前事实允许一次有界恢复 | 使用当前 token |
| `UNCHANGED_ATTEMPT_RECORDED` | 同条件已记录过恢复 | 不重复撞墙；先修正真实输入/配置/合同缺陷 |
| `PROTECTED_PHASE` | 已进入保护阶段 | 使用对应检查点、批次或收尾恢复，不能回到随意规划 |
| `NO_USER_TASK` | 没有真实任务来源 | 提供真实任务需求 |
| `NEEDS_USER_ANSWER` | 尚有影响方案的待答问题 | 回答具体问题，而非反复让系统“继续” |
| `PLAN_ALREADY_READY` | 已有可审阅方案 | 读取并审阅，不再恢复规划 |

### 调用次数、格式修复和收敛次数

r10 默认总模型调用上限为 24，当前编译合同格式修复上限为 3，资源收敛轮次每合同为 3；技术策略调整另有默认 3 次计数边界。以当前有效策略与规划账本为准，不能把这些数相加当作“最多可以重试多少次”。

模型调用在发出前持久预留，崩溃或不确定传输响应不应额外赠送一次调用。格式修复轮次绑定实际安装的编译/输入合同代码；技术策略、宿主 profile 或 token 改变不会重置它。真正代码升级可以出现新合同轮次，但旧调用和累计计数保留；A→B→A 切回旧合同也不会再次重置 A。

一次只读 `read_artifact` 参数错、阶段拒绝或 worker 边界异常，不应被统一记为模型格式修复。读取 `planning_ledger` 中的真实计数和具体 blocker，区分“错误可能可恢复”与“已经实际消耗一次修复”。

### 成功编译后的检查点

候选编译成功后立即保存候选、阶段、合同指纹、证据与预算策略。之后如果证据展示超预算或保存环节出错，应优先复用最新已验证候选，从失败环节继续，不再生成一整份不同方案。

修复前后核对目标、输入、资源身份和核心参数。若只是格式规范化，用户可见科学含义应不变；若必须改变方法，显示差异并重新审阅。不能用重新建项目来清空历史，再把它称为原项目恢复。

### 冻结收尾失败的恢复

`PRETEST_FINALIZATION_FAILED` 需要看失败分类和 `requires_pretest`。若属于管理元数据问题且已有真实 PASSED 预检，引擎可验证原授权、输入、目标、资源与预检指纹后只做 Finalize。这个路径不能重新规划、改写旧锁或悄悄重新执行预检。

历史 r3→r5 恢复曾验证这一点：同一项目最终完成，内部模型调用累计 12→12，不增加调用；原有 10 个关键文件字节和 mtime 不变，仍只存在一次预检复制和一次生产复制。这个案例用于说明允许的恢复边界，不应把其具体指纹和批准复制到新项目。

### 幂等请求与并发

重试同一维护或作业请求时保持稳定 `request_id`。参数变了却复用同一 ID，会返回冲突；应先理解变更并生成新逻辑请求。幂等性保护的是同一个操作身份，不是保证任意新消息都不会产生新任务。

维护、CLI、MCP 和 worker 使用项目协调锁。`PROJECT_BUSY` 表明另一个有效操作占用项目，先查作业状态；不要删除锁文件来抢占执行。中断维护请求有 PREPARED 日志时，按提示重试同一请求完成恢复，再提交其他变更。

### 高频排错表

| 现象/代码 | 首先检查 | 正确修复方向 | 应避免 |
| --- | --- | --- | --- |
| `CAPABILITY_MISMATCH` | 实际加载插件/后端及契约摘要 | 同包更新、核验导入路径、新会话重载 | 绕到另一个工具继续猜参数 |
| `NEEDS_PATHS` | 缺少 data_dir/db_dir，宿主工作区 | 提供服务器真实路径 | 把 cwd 自动当原始输入 |
| `NEEDS_PROJECT` | 当前会话是否绑定项目 | 使用正确项目或新建会话 | 任意填写别的项目路径 |
| `UNSUPPORTED_MAINTENANCE_ARGUMENT` / `INVALID_ARGUMENT` | operation schema、字段错误、suggested_call | 改用正式 list/read 操作 | 给 inspect 继续添加 path/file |
| `SNAPSHOT_CHANGED` | 文件是否在分页时变化 | 重新读取同一新快照 | 混接旧页和新页 |
| `DIAGNOSTIC_NOT_FOUND` | 摘要是否声明有此诊断 | 查看现有诊断 ID 和状态 | 把无诊断等同隐藏失败 |
| 本地输入缺失 | 实际文件、权限、挂载和角色 | 修正可核实路径/挂载，保留目标 | 自动下载替代缺失本地文件 |
| `INPUT_REVISION_CONFLICT` | 当前输入声明 revision | 重新读当前输入和变更差异 | 用过期修订号覆盖 |
| `MODEL_CONTEXT_METADATA_MISSING` | host_model_profile 与实际宿主模型容量 | 修复宿主元数据传递 | 仅扩大 evidence_bytes |
| `MODEL_CONTEXT_LIMIT` | 最终请求、输出预留、协议余量与分项 | 去重、合理分配允许预算 | 删除必需输入或交付目标 |
| 证据片段超限 | 证据/指南/历史分项与重复项 | 复用候选，有限策略调整 | 反复重新规划整份方案 |
| `PLANNING_POLICY_REVISION_CONFLICT` | 当前 policy_revision | 重新 inspect 后再调整 | 把维护 revision 当策略 revision |
| `PLANNING_POLICY_PHASE_DENIED` | 生命周期及冻结锁 | 在合法规划阶段处理 | 生产中修改配置逃过守卫 |
| `PLANNING_FORMAT_REPAIR_LIMIT` | 当前编译合同、保留候选、账本 | 修复真实代码/合同缺陷并核查升级 | 换 token 或反复改预算重置额度 |
| `PLANNING_CONVERGENCE_LIMIT` | 资源事实是否稳定、候选是否被重复改动 | 保持无关字段，查具体差异 | 缩小用户范围凑成功 |
| `CHECKPOINT_PLAN_NOT_READY` | 草稿是否真正生成 | 查看规划恢复与当前候选 | 直接称草稿指纹损坏 |
| 过期审阅指纹 | 当前方案/需求/授权是否变化 | 重读当前材料并审阅 | 复用历史 confirm 参数 |
| `RESOURCE_PHASE_DENIED` | 是否 BOOTSTRAP/PRETEST_REPAIR，依赖已批准否 | 走正确依赖审阅阶段 | 手工写 READY 注册记录 |
| `PROJECT_BUSY` | 活跃作业和子进程证据 | 查询/取消指定作业或等待释放 | 删除锁、kill全部进程 |
| `REQUEST_ID_CONFLICT` | 同 ID 是否承载不同参数 | 区分重试与新操作 | 用变更参数硬续旧请求 |
| `MAINTENANCE_RECOVERY_REQUIRED` | pending 中断请求 | 按原身份恢复 | 删除 PREPARED 审计文件 |
| `OWNERSHIP_REQUIRED` | 文件是否真实登记且归当前项目 | 使用正式已登记 artifact | 把未知文件认领为 scratch |
| `ARTIFACT_IN_USE` | 有效计划/检查点引用 | 保留引用所需文件 | 先删再让状态“自动修复” |
| `PURGE_SCOPE_DENIED` | role 是否 scratch | 草稿隔离/恢复，受保护对象走专门流程 | 永久删审计、结果或锁 |
| `DESTINATION_EXISTS` | 导出目标已有文件 | 指定新的文件名/目录 | 假定 export 会覆盖旧文件 |
| exit=0 但验收失败 | 原命令记录和后置断言 | 保留成功执行事实，修复验收合同 | 重跑命令或宣称已通过 |
| 纳秒时间戳显示不同 | 原始持久整数与宿主显示 | 以无损原始记录比较 | 仅凭JS展示判输入漂移 |

## 项目文件：在哪里找依据

项目根保存配置与部分状态；结果根保存需求、计划、预检、冻结和生产材料。两根可能相同，也可能不同。以下路径均相对对应根；不存在的文件通常表示该阶段尚未生成，不应凭空补写。

| 相对位置 | 所属根 | 用途 | 运维注意 |
| --- | --- | --- | --- |
| `project_config.yaml` | 项目根 | 路径、项目配置和来源 | 从这里确认 output 等有效根 |
| `experiment_plan.yaml` | 项目根 | 保存的结构计划 | 不能手工改已批准计划后继续生产 |
| `config/tool_registry.yaml` | 项目根 | 工具真实入口、版本等 | 注册事实由正式服务维护 |
| `config/database_registry.yaml` | 项目根 | 数据库/索引注册事实 | READY 不可随意修改 |
| `config/tool_alias_registry.yaml` | 项目根 | 逻辑能力与入口映射 | 别名不是软件存在的唯一证据 |
| `state/conversation.json` | 项目根 | 持久对话状态、候选、输入和规划检查点 | 不删除它来解除次数限制 |
| `state/planning_runs/` | 项目根 | 按任务记录模型调用、修复与收敛账本 | 区分累计与当前合同轮次 |
| `state/planning_policy.json` | 项目根 | 技术策略变更与修订 | 通过维护接口修改 |
| `state/host_model_profile.json` | 项目根 | 宿主容量事实 | 缺失不是任意填容量的理由 |
| `state/last_host_budget.json` | 项目根 | 最终宿主请求预算测量 | 与策略指纹、测量来源一起读 |
| `state/input_commit.json` | 项目根 | 输入确认事务 | 未完成提交不能当确认成功 |
| `state/maintenance/registry.json` | 项目根 | 自有管理产物和完成请求 | 不是任意文件所有权声明 |
| `state/maintenance/requests/` | 项目根 | PREPARED/完成等维护回执 | 中断恢复必须保留 |
| `state/portable/jobs/` | 项目根 | 便携服务作业状态、进程证据 | 与宿主 job ID 不要混淆 |
| `requirements/requirements.yaml` | 结果根 | 已确认路径、输入、目标、交付物 | 确认事实的权威记录 |
| `requirements/input_manifest.yaml` | 结果根 | 已确认输入清单 | 后续不从模糊聊天重新推断 |
| `requirements/decision_log.jsonl` | 结果根 | 决策历史 | 历史只能追加/正式版本化 |
| `project_state.yaml` | 结果根 | 生命周期和状态指纹 | 直接改状态文本不能推进流程 |
| `plans/` | 结果根 | 草稿、依赖报告、证据、预算 | 区分当前材料和历史材料 |
| `plans/visible_plan_sources/` | 结果根 | 原模型草稿只读审计来源 | 不作为当前执行/授权依据 |
| `pre_test_run/` | 结果根 | 预检运行与验收材料 | PASSED事实应保留 |
| `pipeline/pipeline.lock.yaml` | 结果根 | 冻结流程 | 生产只读重放 |
| `pipeline/toolchain.lock.yaml` | 结果根 | 冻结工具链 | 检查真实入口/版本/指纹 |
| `pipeline/database.lock.yaml` | 结果根 | 冻结资源与consumer入口 | 不能用通用路径替代已验证入口 |
| `pipeline/validation_report.json` | 结果根 | 冻结所依据验证结果 | 不等于全量已经执行 |
| `pipeline/production_approval.yaml` | 结果根 | 当前生产批准 | 不允许模型伪造 |
| `runs/` | 结果根 | 生产批次记录和产物 | 按批次定位实际命令 |
| `reports/` | 结果根 | 人可读/机器报告 | 报告解释应与原始回执一致 |
| `.agent-workspace/` | 结果根 | 管理草稿、scratch、隔离区 | 仍按登记和引用关系清理 |
| `resource_skills/` | 结果根 | 项目学习的资源用法 | 指南不能代替冻结事实 |
| `knowledge/` 或配置位置 | 有效知识根 | 成功调用、失败/修复与来源 | 未验证历史不称为成功经验 |

读文件时若项目根和结果根不同，选择 `root=project` 与 `root=output` 会得到不同位置。不要看到 `project/reports` 不存在就断言报告丢失，先读 effective_paths。

## 日常运维与交付检查

### 开始前

核对能力兼容、路径角色、输入清单、目标与必需交付物。资源先发现复用，再谈安装。任务复杂或数据大时，先审阅真实依赖范围和预检方式。价格、网络和存储规模没有证据时记录为未知，不填虚构数字。

### 执行中

使用已有 job ID 查询，不重复提交。按报告定位具体失败步骤，保留 stdout/stderr、真实返回码和后置检查。不要清理正在引用的暂存；需要取消时操作具体任务并再次核查终态。

### 结束后

先核对 `COMPLETED` 与报告覆盖，再检查每项交付物实际存在且满足约定检查。最后导出实际可见会话和项目回执，记录文件路径、范围和哈希；需要包含最终答复时另取最终快照。归档时保留输入清单、需求、决策、三份锁、验证报告和批次记录。输入与持久资源仍按用户存储策略保留，不因“任务完成”自动删除。

### 交接给另一位操作者

最少交接项目根、有效结果根、当前状态、当前作业号或批次号、输入清单版本、计划/锁标识、最近阻塞与下一动作。新的操作者先 inspect，不需要复制整段聊天重新规划。迁移机器涉及路径、可执行文件、资源入口与指纹的重新验证，复制文件夹不等于冻结合同在新机器自动有效。

## 本章源码与证据索引

本章基准为公开 ZIP 内的 `microbiome-0.2.5/bundle/source/`。下列路径相对该 source 根；行号用于定位本次公开快照，后续版本可能变化。它们是本章接口与行为描述的主要依据，不要求用户直接修改这些文件。

| 主题 | 源码位置 |
| --- | --- |
| 原生工具允许清单与能力入口 | `microbiome/harness/plugin.mjs:604`、`:621` |
| 原生 workflow 参数、真实消息、项目绑定与后台任务 | `microbiome/harness/plugin.mjs:632` |
| 原生维护 inline/后台与导出快照 | `microbiome/harness/plugin.mjs:675` |
| 原生规划恢复 | `microbiome/harness/plugin.mjs:722` |
| 原生资源服务 | `microbiome/harness/plugin.mjs:755` |
| 原生检查点三个动作 | `microbiome/harness/plugin.mjs:782` |
| 原生项目状态 | `microbiome/harness/plugin.mjs:838` |
| 原生会话行为约定、六阶段与恢复用语 | `harness/agent.cordis.yml:5` |
| 九项维护的完整参数 | `microbiome/pipeline/maintenance_contract.json:1` |
| 四项资源操作完整参数 | `microbiome/pipeline/database_contract.json:1` |
| 生命周期与合法转换 | `microbiome/pipeline/state.py:16` |
| 实际权限、保护角色与写根 | `microbiome/pipeline/maintenance.py:186` |
| 清理所有权/引用/删除范围 | `microbiome/pipeline/maintenance.py:427` |
| 维护幂等、锁与中断恢复 | `microbiome/pipeline/maintenance.py:622` |
| 预算默认值、有效上限与可调整阶段 | `microbiome/pipeline/planning_policy.py:18` |
| 策略查询与测量边界 | `microbiome/pipeline/planning_policy.py:143` |
| 规划账本与合同绑定轮次 | `microbiome/agent/planning_runtime.py:15`、`:109`、`:160` |
| 恢复资格和证据token | `microbiome/agent/recovery.py:76` |
| 输入角色、结构化声明和本地优先 | `microbiome/input_resolution.py:1`、`:678` |
| 已确认输入及事务核验 | `microbiome/integrations/service.py:130` |
| 外部表格字段、完整表格模式 | `microbiome/pipeline/external_cohort.py:15`、`:139` |
| MCP工具完整定义 | `microbiome/integrations/mcp_server.py:10` |
| JSON CLI 参数及真实交互决策 | `microbiome/integrations/cli.py:13`、`:104` |
| JSON服务操作表 | `microbiome/integrations/service.py:435` |
| 项目根/结果根与各路径 | `microbiome/pipeline/project.py:64` |

实机证据位于公开包的 `validation/REPORT.md`、`validation/default-r10/report.json`、`validation/default-r10/conversation.json`、`validation/default-r10-independent-audit.json` 及 `validation/default-r10/execution-evidence/`。本章历史结果引用这些记录；原始失败试次保留，不以新说明书改写旧事实。
