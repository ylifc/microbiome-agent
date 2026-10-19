# Microbiome 生信模式：功能、工作流程与迁移安装手册

版本：0.2.5；宿主协议：1；本轮验证宿主：DeepSeek Harness 0.1.0-rc.12。以下安装命令针对包含 `bundle/` 的 0.2.5 便携发布 ZIP。

## 1. 安装后得到什么

在 DeepSeek Harness 的新会话模式选择中增加“Microbiome 生信工作流”。保留 Python 工作流引擎、全部软件与数据库 Skills、工具描述与实现、环境探测、数据库契约、历史经验检索、跨样本适配器、冻结生产和结果报告功能。

所有模型调用通过当前 Harness 会话的模型服务完成，主规划、结构化编译及辅助模型使用同一路由。模式不要求填写 API key 或模型地址，不复制宿主凭据，也不启动独立的模型 HTTP 客户端。宿主本身须已完成其正常安装并能对话；本包不会替一台尚未配置模型的 Harness 创建账号或凭据。

软件、数据库及宿主 Skill 的实际清单见发布包 `bundle/catalog/skills.json`，资源目录审计见 `bundle/catalog/capabilities.json`。`local_runtime_status: NOT_PROBED` 表示尚未探测本机资源。Skill 数量是知识目录覆盖范围，不表示新机器已经安装相应软件或下载数据库。

安装包不包含用户原始测序数据、个人数据库二进制、旧机器 Conda 环境、项目结果、密钥或真实项目运行锁。保留旧的软件经验作为 UNVERIFIED 指导资料；历史标题中的“成功”不构成新机器的验证证据。软件和数据库体积、平台兼容性、授权条件不同，需要在新机器按当前项目批准的依赖计划复用或配置。

## 2. 功能范围

| 功能 | 实际行为及边界 |
| --- | --- |
| 问候与项目接入 | 介绍能力，明确原始数据、持久数据库、下载、结果项目和知识目录；保存项目路径后贯穿全部阶段。 |
| 本地输入解析 | 识别 FASTQ/FASTA、目录和样本清单，逐样本区分本地文件与明确远程编号；文件缺失不会被转换成 SRA 请求。 |
| 研究设计与取证 | 分析实验目标、分组、配对、样本数和交付物；检索研究资料，记录链接和实际读取范围，网络失败明确记录。 |
| 资源盘点 | 按冻结锁、验证注册表、Skill 映射、所有现有 Conda 环境、PATH/软件目录、包元数据的顺序查找能力。 |
| 软件环境 | 分开记录逻辑工具、软件家族、包名、执行文件、环境和 Skill；只在批准范围内创建或修复环境，禁止修改 base/root。 |
| 数据库契约 | 精确记录消费者使用的运行入口、版本、marker、来源、状态和指纹。数据库置于持久数据库根目录，不放入 Conda 环境。 |
| 资源学习 | Registry 记录位置和版本；Skill 记录功能和使用方式；Knowledge 记录已验证调用、失败、修复、安装与获取历史。 |
| 预检 | 执行真实计算命令，检查退出码、结果文件及声明的输出契约；测序输入保留样本与配对并减少读数，已声明的外部表格按全表合同验证。 |
| 冻结与生产 | 验收通过才冻结工具链、数据库和参数；全量阶段须有有效确认记录；生产仅重放锁，不重新规划、不下载或更新 READY 资源。 |
| 失败与恢复 | 记录失败类别、版本、命令及回归候选；资源漂移即停止；检查点支持用户明确恢复已授权批次。 |
| 结果交付 | 交付按样本及跨样本结果、执行日志、验收报告、最终 Markdown 报告与产物索引；结果只按实际验收状态解释。 |
| 宿主任务集成 | 后台运行返回任务编号，由 Harness 任务工具查询、等待和取消；状态工具读取绑定项目的路径、阶段与报告。 |

工具知识覆盖测序质量控制、宿主序列去除、扩增子处理、分类与丰度、组装、比对、分箱、基因预测与注释、功能分析、基因组质量和分类、比较分析、跨样本表格及统计等常用计算任务。原项目其他生信工具模块仍随代码分发，其专用 Python/R/CLI 依赖按任务单独确认，不在主引擎安装时强行安装全部可选依赖。源码兼容声明为 Python 3.11+，离线依赖按构建解释器锁定，请以发布 `manifest.json` 和安装器检查为准。资源审计警告以 `bundle/catalog/capabilities.json` 为准；说明文档不构成已验证资源，相关任务须先补齐运行契约。

## 3. 工作流程与确认点

1. 选择模式并提供分析目标及路径。宿主上的路径指实际执行计算的服务器路径，不是浏览器所在电脑的路径。
2. 确认输入来源，持久化样本清单；已确认本地清单会被后续所有阶段复用。原始数据获取和参考数据库获取分别审批。
3. Agent 检索研究设计，展示方法、参数依据、输入输出和预期交付；问题未明确时先补充需求。
4. 探测本地资源。展示可直接复用的资源、缺口及需要安装或下载的具体范围；READY 资源不重复下载。
5. 默认交互模式中，用户审阅方案并实际发送“确认执行”后执行已批准依赖准备及预检。新增范围以外的依赖必须生成新版本计划并重新确认。
6. 预检全部验收通过并完成冻结，展示预检结论、锁版本及全量资源预计需求。
7. 默认交互模式中，用户实际发送“确认全量执行”，才进入全量任务；出现错误时停止受影响分支并保留证据。
8. 返回最终报告、产物索引和真实完成状态。中断批次用“恢复批次 <batch-id>”恢复。

`microbiome_workflow` 消费真实用户消息，模型生成的“确认”和工具返回中的“确认”不能充当用户授权。已有项目委托覆盖相应主题与范围时，`microbiome_checkpoint` 可读取当前完整材料，绑定审阅及授权指纹完成确认并继续执行；它不能创建委托或伪造用户消息，材料或授权变化后须重新审阅。默认交互模式不因安装或升级而自动转为委托模式。重复请求优先返回原回执，进程重启后仍可复用；安装包自检使用固定的合成数据，不会替真实项目批准任务。

## 4. 新机器的前置条件

- Linux x86_64 是本次离线发布验证的平台。Windows 请在 WSL2 内安装并运行 Linux 版本的 Harness；macOS 与 ARM 的完整科学运行环境未在本次发布验证。
- 本次离线包使用 Python 3.13、Linux x86_64，需要可用的 `venv` 和 pip；安装器核对 Python 次版本、系统和架构。其他解释器或平台须构建匹配依赖并重新验收。
- 已可正常使用的 DeepSeek Harness。本轮针对 0.1.0-rc.12 的 Cordis preset、模型服务、subprocess 和 jobs 接口验证；其他版本须单独验证兼容性。
- 项目、数据库和下载目录有足够容量及权限。新分析的软件安装需要可用的 Conda/Mamba 等对应后端；这与主程序的独立 venv 是两层环境。

## 5. 下载、校验和安装

`microbiome-latest.zip` 解压后包含 `microbiome-0.2.5/bundle/` 和随交付提供的 `validation/`。`bundle/` 内包括代码、引擎 wheel、匹配的 Harness preset、锁定依赖 wheels、安装器和文档。先将 ZIP 的 SHA256 与交付报告中的值核对，再解压。以下命令在新目录执行：

```bash
sha256sum microbiome-latest.zip
unzip microbiome-latest.zip
cd microbiome-0.2.5/bundle
python3.13 scripts/install_harness.py --bundle . --dsh-home "$HOME/.dsh" --offline
```

Harness 安装器检查 manifest 的逐文件 SHA256，建立独立版本化 venv，执行 `pip check`、安装自检及实际导入身份验证，再切换模式目录。已有模式使用 Linux 原子目录交换；文件系统不支持时保留旧入口并报错。旧运行环境与回滚材料保留。离线包不提供离线模型、离线文献检索或尚未下载的科学数据库。

发布包根目录的 `install.sh`、`upgrade.sh`、`uninstall.sh` 服务于独立 CLI 安装，接受 `--prefix`，与上述 Harness 安装器分开。不要向这些脚本传入 Harness 的 `--dsh-home` 或 `--rollback`。如果同时需要 CLI，可在 `bundle/` 内运行 `MICROBIOME_INSTALL_PYTHON=python3.13 bash install.sh --prefix /新建的用户专属运行目录`；升级已有受管理 CLI 使用 `upgrade.sh --prefix /已有目录`。

`--dsh-home` 必须指向实际运行该 Harness 的 DSH_HOME。若宿主改过 preset 搜索目录，还需 `--preset-root /实际/preset/目录`；宿主的多个 preset 根目录可能存在同名模式遮蔽，安装后须在实际会话中确认选中了本包。用源码部署 Harness 的用户可加 `--harness-source /path/to/deepseek-harness` 检查版本；不同版本默认拒绝，`--allow-untested-harness` 仅绕过版本检查，不代表兼容验证通过。

安装器不修改宿主模型配置；只增加受管理的模式和专用运行环境。安装后重新加载模式列表，在新会话中选择“Microbiome 生信工作流”，调用 `microbiome_capabilities` 核对引擎版本、插件和契约指纹。正在使用旧 preset 的会话不会因 ZIP 更新自动升级。源码部署 Harness 须先按第 11 节构建发布包，不要把源码根目录当作已构建的 `bundle/`。

## 6. 安装后验收

从安装配置读取专用解释器，工作目录可在解压包之外：

```bash
MICROBIOME_PY=$(python3 -c 'import json,os; print(json.load(open(os.path.expanduser("~/.dsh/.agent-presets/microbiome/runtime.json")))["python"])')
"$MICROBIOME_PY" -m microbiome.harness.doctor
"$MICROBIOME_PY" -m microbiome.harness.smoke --directory "$HOME/microbiome-installation-check-01"
```

若使用其他 DSH_HOME 或 preset 根目录，请替换上面的 runtime.json 路径。合成验收目录必须不存在。测试真实运行两个样本复制、逐样本计数、跨样本汇总、小数据验收、冻结、全量重放和检查点恢复；预期 one=20、two=30。它证明核心计算链及适配器可运行，不证明尚未安装的生信 CLI、R 包或数据库可用，也不调用付费模型。

随后在 Harness 新会话中发送“介绍你的工作流程”，检查回复和项目状态。需要分开记录两类证据：离线回归及确定性适配器测试验证可重复的协议和执行行为；真实模型验收通过宿主既有路由发起请求，会消耗该账户额度，且取决于当时的网络与模型服务。上面的 `smoke` 不包含真实模型请求。

本轮真实宿主验收使用人工数字 TSV 和现有系统命令，逐阶段结果、模型用量、失败及恢复记录见交付目录 `validation/`。通过该任务只证明报告列出的链路；不证明所有复杂候选、其他软件或数据库都能顺利运行。原生宿主服务测试也不等于 Electron 桌面交互已验证。

## 7. 第一个分析会话

可发送如下内容，并将示例路径替换为新机器真实路径：

> 原始数据目录是 /srv/data/study01，持久数据库目录是 /srv/references，结果项目目录是 /srv/projects/study01，下载目录是 /srv/downloads/study01，知识目录是 /srv/projects/study01/knowledge。请先检查样本清单并为我的分析目标设计流程，展示可复用资源和缺口，等待我确认。

模式会调用 `microbiome_workflow` 返回后台任务编号；使用宿主 jobs 工具等待或查询，使用 `microbiome_project_status` 看项目阶段、路径及报告。请让宿主完整展示具体方案，不只显示“后台任务完成”。同一会话绑定一个项目；已有项目继续使用持久路径，更换项目请打开新会话。

软件缺失时先搜索全部现有环境。只有已批准依赖范围内的安装成功，且执行文件、版本、帮助与 smoke test 都验证通过后，才能登记 READY 并生成或更新对应 Skill 和知识记录。无法自动安装的资源会给出明确阻塞原因，不会伪造 READY。

## 8. 文件布局、迁移及维护

安装布局：

```text
DSH_HOME/
  .agent-presets/microbiome/  模式、插件及本机 runtime.json
  microbiome/runtimes/<release>/venv/  独立 Python 引擎
  microbiome/runtimes/<release>/docs/  随运行环境保留的说明文档
  microbiome/sessions/  会话到项目的绑定
  microbiome/preset-backups/  旧模式配置
用户项目目录/
  project_config.yaml  已确认路径
  project_state.yaml  工作阶段
  state/harness/  进程锁及请求回执
  pipeline/、config/、plans/  冻结锁、资源注册表及设计计划
  knowledge/、reports/  经验与报告
```

请迁移发布压缩包并在目标机器重新运行安装器，不要直接搬运 venv 或旧 runtime.json。安装后运行目录包含本机绝对路径；移动已安装的 DSH_HOME 后也须重新安装。数据库资源可以独立复制到新机器指定持久根目录，但旧 READY 注册表和冻结锁不得直接改路径后继续生产；需在维护阶段重新定位、校验版本和指纹、预检并形成该机器的新锁。

生产期间不要更新软件、数据库或该项目依赖的运行环境。升级前应结束或暂停任务。Harness 升级重复执行第 5 节的 `install_harness.py` 命令；相同发布复核并复用运行环境，不同发布创建独立目录。回滚前会校验旧 preset、配置、运行环境和身份，校验失败保留当前入口。在 `bundle/` 内分别执行回滚或卸载命令：

```bash
python3.13 scripts/install_harness.py --dsh-home "$HOME/.dsh" --rollback
python3.13 scripts/install_harness.py --dsh-home "$HOME/.dsh" --uninstall
```

卸载只移走本安装器管理的模式，保留项目、数据库、日志、回执与旧 venv。避免自动删除仍被冻结锁引用的解释器。旧环境的人工清理应先确认没有生产锁依赖。

## 9. 任务生命周期和排错

任务由 Harness 的 jobs 服务持有，调用工具的短时等待结束后任务可继续运行；浏览器断开是否影响会话取决于宿主生命周期。关闭/销毁 Agent、切换模式造成 Agent 销毁、退出 Harness 或机器关机，均不能保证后台计算继续。受管理 subprocess 取消会终止子进程树；恢复前先查项目状态和已有批次，不要新建重复批次。

| 现象 | 处理 |
| --- | --- |
| 模式未出现 | 检查实际 DSH_HOME、preset 搜索路径和同名遮蔽，重新加载或重启。 |
| HARNESS_UNAVAILABLE | 检查宿主当前会话模型路由及服务是否正常；无需向本模式填写密钥。 |
| NEEDS_PATHS | 提供原始数据、数据库和结果项目路径；已经绑定项目则使用已保存路径。 |
| PROJECT_BUSY | 同一项目已有活动操作，先查询/取消原任务；不要并发修改该项目。 |
| 资源 MISSING/INCOMPLETE/CORRUPT/DRIFT | 停止生产，进入明确维护阶段修复和重新验收。 |
| 离线 wheel 不兼容 | 核对 Linux 架构与 Python 次版本；换匹配离线包或用在线包重新解析依赖。 |
| 科学任务缺软件或数据库 | 核心安装自检不等于科学依赖就绪；按当前依赖计划处理。 |
| 网络检索失败 | 查看研究证据日志；补充可访问来源，不能把未读全文当作已读。 |
| Python 导入错误 | 查看版本运行目录的 doctor.json，执行专用 venv 的 pip check。 |

模型重试遵循宿主允许的错误类别并受模式上限限制；截断回答和不完整 JSON 作为失败处理。请求、响应和模型用量记录在会话扩展事件中，项目命令和产物留在项目日志；会话内的扩展用量事件不保证会进入宿主内置费用汇总视图。模型请求可能包含数据路径和分析摘要，请按项目数据权限管理宿主会话日志。

## 10. 发布验收与仍需现场验证的部分

发布验收分别记录 Python 冻结流程回归、传输授权边界测试、Node 模型桥接测试、安装版身份和 `doctor`、固定合成数据执行以及真实宿主模型调用。实际执行项目与结果见 ZIP 内 `validation/`，未执行的检查不能计入通过项。报告应保留失败案例，并明确使用的发布指纹、宿主版本、路由和任务范围。

真实生信数据、具体 CLI/R 软件、数据库完整性、集群调度、用户网络代理、真实付费模型额度和目标机器硬件必须在目标环境按任务验收。完整代码和说明的安装不代替科学分析有效性检查。

## 11. 重新构建

构建机需要项目 Python 依赖、setuptools 和 wheel。源码目录执行以下命令，将输出路径换为不存在的新目录：

```bash
python scripts/build_portable_release.py --output /新建的发布目录 --offline-deps
```

构建需要下载依赖；已有匹配 wheel 缓存时，可增加 `--wheel-cache /已有的wheelhouse --no-index` 仅使用本地缓存。完整源码位于 `bundle/source/`，后端 wheel 和 Harness preset 从同一源码快照产生。先完成安装版验收，再将审阅过的非敏感证据写入发布目录的 `validation/`，最后打包：

```bash
python scripts/package_release_zip.py --release /新建的发布目录 --output /输出目录/microbiome-latest.zip
```

打包器重新验证 bundle、ZIP CRC 和逐文件字节，一切通过后替换目标 ZIP；不包含发布目录之外的环境和用户数据。`validation/` 内容由交付者审阅，不应复制凭据或未脱敏宿主状态。离线 wheels 与构建机平台和 Python 次版本匹配。SHA256 用于传输完整性检查，不能替代可信来源或签名认证。
