## 安装、宿主适配与迁移

本章面向 Microbiome **0.2.5（r10）公开发行包**。安装目标分为两条独立路线：使用 DeepSeek Harness 原生模式，或者安装通用 CLI 后接入 Codex、VS Code、WorkBuddy。两条路线使用相同发布源码与后端 wheel，但安装位置、启动入口和升级命令不同。用户可以同时安装两者；安装其中一条不会自动激活另一条。

本章命令已经对照该发行包的安装器、参数解析代码及安装后的 `--help` 核实。示例中的 `/srv/...` 是应替换的执行机器路径；命令不会因为文档展示而自动执行。安装、验证与升级均应由拥有相应目录权限的普通用户运行。软件包声明支持 Python 3.11+，本次公开的离线依赖包则限定为 **Linux、x86_64、Python 3.13 次版本**。这些声明不能互换。

### 先认识四层组件

**宿主层**负责聊天、模型、账号、会话和工具调度。DeepSeek Harness、Codex、VS Code 与 WorkBuddy 都属于这一层。宿主须先能够正常使用；Microbiome 安装器不会替用户创建模型账号、购买额度或者修复宿主登录。

**Microbiome 引擎层**包含输入合同、规划、状态机、资源注册、预检、冻结执行、诊断、恢复和报告。主程序使用专用 Python 虚拟环境，与用户既有 Conda 环境分开。普通 CLI、MCP 和 doctor 不需要模型密钥，也不会自动连接模型。

**科学软件层**是具体分析需要的外部可执行程序、R 包和相关运行环境。软件知识 Skill 的存在，仅代表包中具有使用指导。是否已经安装、版本是否合适、实际执行文件在哪里，必须在执行机器上探测。安装引擎不会一次性安装目录中列出的全部软件。

**数据资源层**包括原始文件、样本清单、数据库、索引、项目结果和审计证据。原始输入和持久数据库分别管理；数据库不放进 Python venv 或 Conda 环境。已有 READY 资源应被复用，其路径和版本不能在生产中自动替换。〔I01、I02〕

安装路径与各层关系参见第一篇图 2“安装与适配决策”；安装后的完整运行阶段参见图 3。

### 平台、目录和权限准备

#### 已验证范围与未验证范围

| 项目 | 本发行包的事实 | 安装时如何理解 |
| --- | --- | --- |
| 后端版本 | 0.2.5 | 版本号相同仍应比较发布清单和插件指纹 |
| 离线构建解释器 | Python 3.13.13 | 安装器核对 3.13 次版本，不要求完全相同补丁号 |
| 离线平台 | Linux、x86_64 | 不可把该 wheelhouse 当作 macOS、ARM 或 Windows 原生依赖包 |
| 源码 Python 声明 | 3.11+ | 表示源码包声明；依赖解析和目标平台仍要重新验收 |
| Harness | 0.1.0-rc.12 | 验收覆盖原生宿主服务、模型桥接、工具与状态链路 |
| Harness 桌面 UI | 本轮未验收 | 不能把服务测试写成所有 Electron 界面操作均已通过 |
| Codex、VS Code | 有 Skill 导出和 MCP 配置模板 | 导出与协议检查不等于各客户端版本的完整现场分析实测 |
| WorkBuddy | 有本地 Skill 和统一 CLI | 导出器没有生成 WorkBuddy MCP 配置文件 |

Windows 用户可在合适的 Linux/WSL2 执行环境中部署，但本发行验收没有覆盖所有 WSL2 发行版和宿主组合。关键是 Python、Microbiome、数据和实际命令运行在同一套可见的 Linux 路径环境里。macOS 或 ARM 用户应重新构建匹配依赖并测试；Harness 安装器本身明确要求 Linux。〔I03、I04〕

建议把运行时、原始数据、项目、数据库和下载暂存分开。例如：

```text
/srv/microbiome/
  releases/                 下载与解压后的发行包
  runtimes/                 独立 CLI 安装根目录
  raw/environment-study/    用户原始数据，只读使用
  projects/environment-study/  项目状态、计划和证据
  results/environment-study/   分析输出
  databases/                持久数据库和索引
  downloads/                经批准的下载与暂存
```

这些目录只是组织建议，不是必须逐字使用的硬编码布局。用户主目录中的普通目录也可以使用。原始数据目录应只包含用户希望分析或声明的数据，不能把整个主目录当作原始输入。项目目录应由当前用户写入；数据库目录根据组织管理方式设置读取或维护权限。不要通过把全部目录改成任何人可写来解决路径错误。

首次安装前可以在执行机器上检查基础条件：

```bash
uname -s
uname -m
python3.13 --version
python3.13 -m venv --help
command -v bwrap
df -h "$HOME"
```

`command -v bwrap` 只能证明找到了程序，不能证明内核允许其创建所需命名空间。引擎的原生命令、软件探针及批准后的资源维护使用 bubblewrap 建立文件边界；缺失 `bwrap` 或用户命名空间不可用时，会明确返回 `FILE_POLICY_UNAVAILABLE`。此时规划和部分只读维护仍可用，但执行不能继续。不要把这个错误解释成数据损坏，也不要去掉执行边界作为安装修复。组织管理员应按其系统标准配置 bubblewrap 和命名空间，再运行合成验收。〔I05〕

内存与磁盘没有一个对全部分析通用的固定最低值。主引擎、科学软件、数据库、原始数据和中间结果分别消耗空间；实际任务应在依赖和资源预检时评估。公开 ZIP 大约 93 MB，不代表安装后或科学分析只需要这一容量。离线安装只解决 Python 依赖来源，不提供离线模型、离线检索或尚未取得的科学数据库。

#### 带空格路径如何写

Shell 中每一个带空格的路径都要加引号，例如 `"/opt/DeepSeek Harness/resources/app"`。变量被用作路径时写成 `"$MICROBIOME_PREFIX"`，不能省略引号。JSON、TOML、YAML 又有自己的字符串规则；不要把 Shell 的引号写进 JSON 路径值内部。路径文字中的 `>` 在 Shell 中是重定向符，在自然语言里又可能只是分隔符，因此推荐用“原始数据目录：……；项目目录：……”明确标明角色。

安装器可以接受带空格的普通路径，但路径必须属于正确机器、存在或可按规则创建，并符合所有权检查。尽量使用真实目录路径，避免在受控写入根中依赖符号链接；执行文件策略会拒绝部分符号链接穿越。引号正确只能解决参数分割，不能解决目录不存在、挂载缺失或权限不足。

### 下载、校验和解压公开发行包

项目公开仓库为 [ylifc/microbiome-agent](https://github.com/ylifc/microbiome-agent)，本章固定版本对应 [v0.2.5 发布页](https://github.com/ylifc/microbiome-agent/releases/tag/v0.2.5)。建议从 Release 下载已经构建的 `microbiome-latest.zip`，不要把 GitHub 自动生成的“Source code ZIP”当作含完整离线 wheelhouse 的安装包。

2026-10-19 初始公开包的历史验收基线如下（不用于校验更新包）：

```text
文件：microbiome-latest.zip
字节数：92992994
ZIP 条目数：1433
SHA256：ca49adc748334ec99bb3bceb59f9ffefb34ae99303aadeae77d939a15c75c1ac
```

可在一个新的下载目录执行：

```bash
mkdir -p "$HOME/Downloads/microbiome-0.2.5-download"
cd "$HOME/Downloads/microbiome-0.2.5-download"
curl -fL -o microbiome-latest.zip \
  https://github.com/ylifc/microbiome-agent/releases/download/v0.2.5/microbiome-latest.zip
curl -fL -o microbiome-latest.zip.sha256 \
  https://github.com/ylifc/microbiome-agent/releases/download/v0.2.5/microbiome-latest.zip.sha256
sha256sum -c microbiome-latest.zip.sha256
unzip microbiome-latest.zip
cd microbiome-0.2.5/bundle
```

如果下载目录已有同名包，先确认它是否需要保留，再换一个新的下载目录；不要把多次不同发布解压到同一个旧目录。校验输出应为 `microbiome-latest.zip: OK`。如果校验失败，先检查下载是否完整、校验文件是否属于同一次发布，保留失败文件供诊断并重新下载。SHA256 可检测传输和文件差异，不能单独替代可信来源认证。

本次文档更新在 ZIP 的 bundle 外补入完整手册，引擎 bundle 和原始验收证据不变。当前 ZIP 身份以同一 Release 的 `.sha256` 文件及仓库 `release-manifest.json` 为准。本章旧摘要仅用于追溯初始公开包，不能校验补入手册后的更新包。

解压后应看到以下主要内容：

```text
microbiome-0.2.5/
  bundle/
    manifest.json              发布文件清单与摘要
    README.md、RELEASE.md       安装入口与平台说明
    install.sh                 通用 CLI 安装
    upgrade.sh                 通用 CLI 升级
    uninstall.sh               停用通用 CLI 入口
    scripts/                   安装器与验证器
    wheels/                    Microbiome 后端 wheel
    wheelhouse/                本平台的离线 Python 依赖
    requirements.lock          依赖约束
    harness/                   匹配的原生模式和插件
    source/                    888 个文件的源码快照
    skills/、workflows/         Skill 与工作流资料
    catalog/                   能力、源码和资源目录清单
    examples/                  合成数据演示
  validation/                  选定的验收记录与公开补充说明
```

`manifest.json` 管理 bundle 的文件完整性。不要在 bundle 内直接改配置、生成测试结果或保存临时缓存后再声称仍是原始发行包。安装器会逐文件验证清单；验证输出应写到 bundle 之外的新目录。项目和数据库也不应建在解压包内部。〔I06〕

### 路线 A：安装通用 CLI

#### 标准离线安装

下面命令在解压后的 `bundle/` 中执行。`--prefix` 指定的目录必须尚不存在，或者已经是本安装器管理的目录。父目录可以存在；不要事先手工创建这个专用 prefix 的空目录，因为现有但没有管理标记的目录也会被拒绝。

```bash
MICROBIOME_PREFIX="$HOME/.local/share/microbiome-runtime"
MICROBIOME_INSTALL_PYTHON=python3.13 \
  bash install.sh --prefix "$MICROBIOME_PREFIX"
"$MICROBIOME_PREFIX/bin/microbiome" --version
"$MICROBIOME_PREFIX/bin/microbiome" doctor
```

`MICROBIOME_INSTALL_PYTHON` 决定安装脚本使用哪个 Python 解释器。发行包中的 `install.sh` 默认加上 `--offline`，因此不会因为机器上刚好有其他 Python 就自动重新解析一套在线依赖。如果 `python3` 实际是 3.11 或 3.12，应明确使用 3.13；不要靠改 wheel 文件名规避平台检查。

安装器依次完成完整性校验、解释器和平台核对、专用 venv 创建、受约束的 wheel 安装、入口映射核对、doctor 和 `pip check`，最后生成受管理的启动链接。安装器不会修改 shell 配置文件。正常完成后，会返回实际安装目录、运行时目录、PATH 提示和后续导出命令。〔I07〕

受管理布局类似：

```text
~/.local/share/microbiome-runtime/
  .microbiome-portable.json    管理标记、活动运行时与历史运行时
  bin/
    microbiome                当前 CLI
    biomni                    兼容命令别名
    microbiome-run            同一 CLI 的别名
    microbiome-service        JSON 工作流服务 CLI
    microbiome-mcp            MCP 入口
  runtimes/<随机标识>/
    bin/python                此次安装的专用解释器
    bin/...                   实际控制台入口
    installation-check.json  安装自检结果
```

`microbiome`、`biomni`、`microbiome-run` 在此版本指向相同的新 CLI。直接输入 `microbiome` 显示帮助，不会自动启动旧版聊天 Agent。兼容入口 `microbiome-legacy-run` 等仍存在于发行包的 Python 入口定义中，但不属于本章推荐安装后的日常启动路径。

为了避免 PATH 冲突，首次验收始终使用完整路径。确认正常后，可在当前终端临时加入：

```bash
export PATH="$HOME/.local/share/microbiome-runtime/bin:$PATH"
microbiome --version
microbiome doctor
microbiome call catalog.list
```

是否把 PATH 配置持久写入 shell 启动文件由用户自行决定；安装器没有代做这一步。若 `microbiome --version` 与完整路径输出不同，说明命中了其他安装入口，应先检查 `command -v microbiome`，再调整 PATH，而不是反复安装。

#### 安装状态的正确解读

`doctor` 中 `ok: true` 表示包和基础接口检查通过；`model_owner: host` 表示模型由宿主负责；`model_connection: NOT_REQUESTED` 表示没有发起模型请求；`scientific_runtime: NOT_PROBED` 表示还没有验证具体科学软件和数据库。`file_execution_policy` 中可能显示已发现 bwrap，但 `kernel_probe: NOT_REQUESTED` 仍表示这次 doctor 没有运行内核命名空间探针。不要把这些“未检查”字段误读成“已可运行全部分析”。〔I08〕

如果看到 `already_installed`，说明该 prefix 已启用受管理运行时，普通安装请求没有替换它。需要升级时使用 `upgrade.sh`。如果提示非受管理目录，选一个新的 prefix；不要在已有 Conda 环境中伪造 `.microbiome-portable.json`。

#### 在线安装与源码开发安装

离线发行包适合复现当前构建。在线安装适合需要重新解析依赖的目标机器，仍须满足平台和执行边界要求。在 `bundle/` 中直接调用 Python 安装器且不传 `--offline`，可走在线依赖路径：

```bash
python3.13 scripts/install_portable.py \
  --bundle . \
  --prefix "$HOME/.local/share/microbiome-online-runtime"
```

这会联系当前 pip 配置所指定的软件来源；具体结果取决于包索引、代理及可用 wheel。它不等于“任意平台保证兼容”。已发布的 `requirements.lock` 仍会作为约束使用。

若要修改和调试源码，应从 `bundle/source/` 开始，并使用另一套新 venv。开发环境示例如下：

```bash
cd /实际解压目录/microbiome-0.2.5/bundle/source
python3.13 -m venv "$HOME/.local/share/microbiome-dev-0.2.5"
"$HOME/.local/share/microbiome-dev-0.2.5/bin/python" -m pip install -e .
"$HOME/.local/share/microbiome-dev-0.2.5/bin/python" -m microbiome.cli doctor
```

`pip install -e .` 是开发安装，源码修改会影响后续导入；它不生成可追溯的新发布 ZIP，也不自动更新已经安装的 Harness preset。完成修复后，应重新构建、安装和验收再部署。不要把修改过的开发目录当作冻结生产运行时。开发安装使用普通 Python venv，与受管理 CLI prefix 是不同布局，不能对这个开发 venv 直接调用 `upgrade.sh --prefix`。

### 路线 B：安装 DeepSeek Harness 原生模式

#### 安装之前明确宿主位置

DeepSeek Harness 应先正常启动并能使用其现有模型服务。本模式沿用宿主的模型路由与凭据，不要求用户在 Microbiome 中再次填写模型名、API Key 或模型地址。公开包不包含模型权重，也不会把其他机器的宿主账号迁移过来。

`DSH_HOME` 是 Harness 的状态目录。默认安装器使用环境变量 `DSH_HOME`，未设置时采用 `~/.dsh`。`--preset-root` 是模式搜索根目录，默认是 `DSH_HOME/.agent-presets`；这与程序安装目录不是一回事。`/opt/DeepSeek Harness/resources/app` 是本次验证机器上的应用目录示例，不是所有机器的默认安装路径。

如果用户改过宿主启动环境、状态根或 preset 搜索位置，应使用实际值。多个搜索根存在同名模式时可能发生遮蔽；仅看到磁盘上有 `microbiome` 文件夹不能证明当前会话加载了它。

#### 正式安装命令

在 `bundle/` 中执行：

```bash
python3.13 scripts/install_harness.py \
  --bundle . \
  --dsh-home "$HOME/.dsh" \
  --offline
```

如果已经知道 Harness 应用或源码目录，可以要求安装器读取其中的 `package.json` 检查版本：

```bash
python3.13 scripts/install_harness.py \
  --bundle . \
  --dsh-home "$HOME/.dsh" \
  --harness-source "/opt/DeepSeek Harness/resources/app" \
  --offline
```

`--harness-source` 名称含有 source，但实现读取的是该目录下的 `package.json`；应给出确实含该文件的实际应用或源码目录。没有传此参数时，安装器说明宿主服务会在模式加载时检查，不能将其解释为已经验证了机器上任意 Harness 版本。

只有使用非默认模式根目录时才加：

```bash
python3.13 scripts/install_harness.py \
  --bundle . \
  --dsh-home "/实际/DSH状态目录" \
  --preset-root "/实际/preset搜索目录" \
  --offline
```

`--allow-untested-harness` 是允许继续尝试未列入版本清单的宿主版本的显式选项。它只绕过安装器的版本限制，不补齐缺失服务，不改变协议，也不证明兼容。一般用户应使用已验证版本；适配开发者才在独立环境中使用此选项并补做验收。〔I04〕

#### 安装器具体做什么

Harness 安装器先验证 bundle，再根据“版本号 + manifest 摘要前缀”选择专用运行时目录。它创建 venv，安装后端和依赖，运行 doctor 与 `pip check`，保存发布清单和文档，比较实际导入的关键后端文件身份，然后在暂存目录准备 preset、插件、`runtime.json` 和管理标记。

存在旧受管理模式时，安装器使用 Linux 原子目录交换切换活动入口并保留旧模式备份。检查失败时保留旧入口；若文件系统不支持所需交换操作，应解决部署文件系统条件，不能直接删旧目录强行安装。非受管理的同名模式会被拒绝覆盖。

```text
DSH_HOME/
  .agent-presets/microbiome/
    runtime.json               本机 Python、运行时与状态目录
    .microbiome-managed.json    发布标记与身份
    ...                        preset 与插件文件
  microbiome/
    runtimes/<版本-清单摘要>/
      venv/                    专用 Python 环境
      docs/                    该运行时对应的说明
      installed.json           安装记录
      deployment-identity.json 实际导入身份检查
      release-manifest.json    发布清单
    preset-backups/            旧模式配置
    sessions/                  会话和项目绑定
```

安装器生成的 `runtime.json` 是本机部署记录，不是用户应随意修改的全局模型配置。模型请求次数、输出字节上限等技术字段也不能代替项目科学参数或授权。不要将别台机器的 `runtime.json` 直接覆盖进来。

#### 必须在新会话里验证

安装完成后，重新加载模式列表，并打开**新会话**选择“Microbiome 生信工作流”。旧会话可能继续持有旧插件对象和旧路径；ZIP 已更新、安装器已运行，都不意味着它已切换。建议先发送：

> 请介绍当前 Microbiome 模式的能力，并调用 microbiome_capabilities 核对引擎版本、协议、插件和合同身份。现在只检查安装状态，不启动分析。

能力回执应能报告当前 `engine_version`、协议版本、后端位置、插件摘要、维护与数据库合同摘要及可用功能。核对重点是实际加载的插件与后端一致，而不只是显示“0.2.5”。不一致时应停止创建新的分析，检查模式遮蔽、旧会话和安装身份。〔I09〕

本版本从宿主实际模型元数据以及同次请求的 prepared context 获得容量信息，宿主原有请求配置保持不变。缺少上下文容量时，应给出 `MODEL_CONTEXT_METADATA_MISSING` 等明确诊断；不得靠模型名称猜容量。`MODEL_CONTEXT_LIMIT` 表示完整请求预算不适配，需要查看各部分用量、去重与有效策略；不能把任意扩大数字视为解决。宿主模型认证、网络失败和额度问题仍由宿主侧处理。

### 分层安装验收

安装成功和完整任务成功是两种不同证据。建议按以下顺序记录每层结果，避免把上一层通过扩大解释成下一层也通过。

分层验收与后续冻结执行的关系参见第一篇图 3“完整生命周期”和图 8“结果验收与报告”。

#### CLI 的安装版验证

在 `bundle/` 中执行以下命令，输出目录须为 bundle 之外尚不存在的新目录：

```bash
python3.13 scripts/verify_portable_release.py \
  --bundle . \
  --prefix "$HOME/.local/share/microbiome-runtime" \
  --output "$HOME/microbiome-install-check-01"
```

验证器检查安装身份、隔离 venv、启动入口、源码摘要、依赖一致性、doctor、工作流目录、三种宿主导出及固定合成演示。绘图检查中产生的图片仍标为预览，必须实际查看才可完成视觉验收；图片能生成不等于科学结论成立。最终结果写入新目录的 `verification.json`，各子检查有独立标准输出和错误日志。此验证不会调用模型，也不会下载科学数据库。〔I10〕

如果不是受管理 CLI prefix，而是另一套已安装 venv，可使用 `--python /实际/venv/bin/python` 代替 `--prefix`。两个参数互斥。必须传入 venv 内的解释器路径，不能先把解释器符号链接解析成系统 Python 再交给验证器，否则会检查错环境。

#### Harness 的 doctor 与固定执行验收

默认安装路径可从当前 preset 的 `runtime.json` 读取解释器：

```bash
MICROBIOME_PY=$(python3 -c 'import json,os; print(json.load(open(os.path.expanduser("~/.dsh/.agent-presets/microbiome/runtime.json")))["python"])')
"$MICROBIOME_PY" -m microbiome.harness.doctor
"$MICROBIOME_PY" -m microbiome.harness.smoke \
  --directory "$HOME/microbiome-harness-smoke-01"
```

自定义 DSH_HOME 或 preset 根时，把读取路径替换为当前活动模式的 `runtime.json`。不要从猜测的旧运行时目录读解释器。

`smoke` 是显式请求的固定人工数据验收，会在全新目录写入合成输入和输出，真实运行复制、计数、预检、冻结重放和恢复检查，预期样本计数为 one=20、two=30。它不使用真实样本、不验证外部生信软件或数据库，也不调用付费模型。它内部对固定测试的阶段推进不能被复制为真实项目的“自动批准技巧”。〔I11〕

最后再在实际新宿主会话中运行一个范围明确的人工小任务，检查工具、状态事件、用户确认、回执和最终报告。实际模型请求会使用宿主账户额度；没有价格资料时费用应标为未知。本次发布的真实 Harness 验收采用人工数字 TSV 和系统复制命令，证明报告列出的链路；它不能证明所有科学软件和所有工作流都已经验收。

### 通用宿主的 Skill、MCP 与 CLI 适配

#### 先区分说明能力与执行能力

Skill 告诉宿主何时使用工具、怎样读取项目、怎样审阅输出；MCP 或 CLI 才是与后端实际交互的入口。只导入 Skill 而没有可运行的后端，宿主只能读到指导文字。只添加 MCP 而没有流程指导，宿主可能缺少正确的阶段与资源解释。建议成套导出两个正式 Skill：`microbiome-workflow` 和 `microbiome-visualization`。

导出命令只写到一个**新目录**，不自动修改宿主配置。它拷贝 Skill 目录及全部附属资料，校验字节并返回指纹。输出位置已经存在时会失败，防止混入上一版文件；升级时换一个新导出目录，再审阅替换。〔I12〕

```bash
microbiome export --host codex \
  --output "$HOME/microbiome-adapters/codex-0.2.5"
microbiome export --host vscode \
  --output "$HOME/microbiome-adapters/vscode-0.2.5"
microbiome export --host workbuddy \
  --output "$HOME/microbiome-adapters/workbuddy-0.2.5"
```

三条命令是不同宿主的选择示例，不要求用户全部执行。每个导出目录都含两个 Skill 文件夹。Codex 另外生成 `config.toml`；VS Code 另外生成 `mcp.json`；WorkBuddy 不生成连接器配置。实际导出的 Python 路径来自运行该命令的解释器，升级运行时之后应重新导出，而不是假设旧配置会自动改变。

#### Codex 接入

将导出的两个 Skill 目录放到当前 Codex 支持的用户或项目 Skill 目录中，例如项目的 `.agents/skills/` 或用户 `~/.agents/skills/`，保留整个文件夹，不要只复制 `SKILL.md`。把导出的 `config.toml` 中 MCP 表合并到实际配置文件，不要覆盖已有模型、其他 MCP 或权限配置。Skill 搜索路径及配置层级以安装的客户端版本为准。[Codex 官方 Skill 文档](https://developers.openai.com/codex/skills/)

本发行模板的 stdio 配置形态为：

```toml
[mcp_servers.microbiome]
command = "/实际运行时/bin/python"
args = ["-m", "microbiome.integrations.mcp_server"]
startup_timeout_sec = 30
tool_timeout_sec = 120
```

正常导出已替换 `command` 为实际解释器，用户不应保留 `/实际运行时/` 这样的占位符。MCP 由宿主按需启动时，标准输入输出承担协议通信；手工在终端运行后没有欢迎界面，并不代表进程卡住。Codex 支持 stdio 和 Streamable HTTP 配置，具体启用和重载以官方文档为准。[Codex 官方 MCP 文档](https://developers.openai.com/codex/mcp/)

重载配置或新建会话后，先让 Agent 调用能力和目录工具，核对版本，再读取一个已初始化项目的状态。不要第一次接入就提出大量下载和复杂执行。宿主能够读本地文件，不代表工作流中的每个写入动作都获得授权；项目阶段和文件所有权仍由后端检查。

#### VS Code 接入

把导出的 `mcp.json` 合并到工作区 `.vscode/mcp.json`。其 stdio 结构为：

```json
{
  "servers": {
    "microbiome": {
      "type": "stdio",
      "command": "/实际运行时/bin/python",
      "args": ["-m", "microbiome.integrations.mcp_server"]
    }
  }
}
```

需要保留已有 `servers` 条目，只新增或有意更新 `microbiome`。两个 Skill 目录放到当前版本支持的项目 Skill 位置；本项目接入文档使用 `.github/skills/` 示例。远程 SSH/WSL 工作区中，MCP 配置、Python 路径和数据路径必须对应远端执行环境，不能把本机解释器路径填给远端项目。VS Code 的服务器管理、工作区和远程配置行为见 [官方 MCP 配置说明](https://code.visualstudio.com/docs/agent-customization/mcp-servers)。

首次启动可能需要按宿主界面确认信任该 MCP 服务；这属于宿主连接层。连接成功之后，再检查 Microbiome 能力和项目状态。宿主显示已连接仅证明协议建立，不表示软件、数据库和分析计划已完成审批。

#### WorkBuddy 接入

本版本的 `--host workbuddy` 只导出两个本地 Skill，并返回明确说明：MCP 连接器配置需要针对宿主版本核实。可以按 WorkBuddy 当前提供的本地导入流程分别导入完整 Skill 文件夹，并让 `microbiome-service` 在实际执行机器的 PATH 中可用。[WorkBuddy 官方技能说明](https://www.workbuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Skills-Market)

若当前 WorkBuddy 支持连接 stdio MCP，可使用已核实的 Python 启动命令配置连接器，但本手册不虚构特定配置文件名或菜单名称。只有完成当前客户端实际导入、工具发现、路径可达和状态读取后，才能记录“此版本接入成功”。不能因为导出器接受 `workbuddy` 就声称全部 MCP 和后台任务交互已经实测。

没有可用 MCP 连接器时，Skill 可以通过宿主允许的 CLI 能力调用统一 JSON 操作。例如 `microbiome-service call catalog.list`；实际项目操作仍需真实路径、真实用户来源和阶段许可。

#### 原生 Harness 与通用 MCP 的工具名并不完全相同

Harness 原生模式提供以真实用户消息驱动的 `microbiome_workflow` 和原生检查点接口，结合宿主 jobs 服务推进阶段。通用 MCP 则暴露输入解析、需求提案、计划验证、依赖预览、审阅、作业提交等拆分接口。两者共用后端规则，但不是把所有工具名称机械复制一遍。

因此，在 Codex 或 VS Code 中没有找到 `microbiome_workflow`，不能直接认定安装坏了；应查看该客户端实际发现的 `microbiome_capabilities`、`microbiome_catalog`、`microbiome_project_status`、`microbiome_plan` 和作业工具。宿主应按照各自接口契约编排，不能捏造不存在的操作名称。〔I13〕

默认交互模式下，通用 MCP 提供完整审阅材料，用户在自己的交互终端通过 `decide` 记录决定；该操作会校验材料指纹和实际终端交互。明确的项目委托可以启用范围受限的宿主决定通道，仍须保存真实用户请求和授权来源。安装适配不自动创建委托，不把终端操作伪装成用户批准。具体决策与阶段使用见后续使用篇。

### MCP 的两种传输方式

**stdio** 是默认方式：宿主启动专用 Python 子进程并通过标准输入输出通信。它无需选择 TCP 端口，适合同一执行环境中的宿主。独立运行命令为：

```bash
microbiome mcp
```

**HTTP** 需要先启动服务，再导出指向它的配置：

```bash
microbiome mcp --transport http --port 8765
```

上述进程需保持运行。另一个终端中可执行：

```bash
microbiome export --host codex \
  --mcp-url http://127.0.0.1:8765/mcp \
  --output "$HOME/microbiome-adapters/codex-http-0.2.5"
```

HTTP 服务绑定 `127.0.0.1`，默认端口 8765；端口范围必须在 1–65535。导出器仅接受带显式端口、路径为 `/mcp`、无账号密码、查询和片段的回环 HTTP URL。导出配置不会帮用户启动服务。〔I13〕

`127.0.0.1` 总是指发起连接的环境自身。在远程桌面、WSL 或容器场景中，浏览器所在电脑的回环地址与计算服务器不是同一个对象。当前发行版没有内置公网认证部署、SSH 隧道安装或远程调度器。应使用宿主已经支持并经过组织配置的远程工作区；不要把这个本机服务随意映射成公网工具服务。

### 可选：连接已有的兼容模型端口

这一分支只用于用户已经拥有并能够访问的 OpenAI 兼容服务。它与 MCP 服务用途不同：MCP 的 `/mcp` 暴露工作流工具，模型端口提供 `/v1/models` 和 `/v1/chat/completions`。把 8765 的 MCP 地址填给模型端点不会让它变成推理服务。

```bash
microbiome endpoint --endpoint http://127.0.0.1:8000 --check
```

`--check` 会读取模型列表，并对最多三个候选发出短推理请求以验证聊天能力。因此它确实调用模型，可能使用服务额度；它不是纯静态配置检查。程序只使用显式地址，不扫描端口、不启动模型、不下载权重、不读取其他宿主的私有凭据。需要客户端认证但当前入口不具备认证时，应由已有授权网关或宿主接入解决。〔I14〕

确认可用之后，可针对已有项目运行：

```bash
microbiome endpoint \
  --endpoint http://127.0.0.1:8000 \
  --project /srv/microbiome/projects/environment-study
```

非交互调用还需要 `--prompt`；交互终端可继续对话。此入口会建立实际对话，并可能按用户请求推进项目；它不用于安装阶段自动试探真实数据。关闭前台对话与取消一个已提交的持久作业是不同动作，恢复时先查项目和 job ID。

### 升级、回退和卸载

#### 升级之前保存什么

先结束或按项目支持的方式暂停当前执行，再保存当前发行 ZIP、摘要、安装标记、活动运行时身份、项目状态和锁。运行中切换代码可能导致长进程与新工具请求使用不同版本；用户看到的版本号也可能混合，因此不能把升级当作运行中修复命令。

升级不会自动迁移所有旧计划或重写冻结锁。旧锁必须保持其原有含义；需要改变工具入口、资源路径或执行规则时，走独立维护、重新验证和新版本冻结。保留旧运行时不仅为回滚，也可能是旧锁继续复现所需。

#### 通用 CLI 升级

从新发行包的 `bundle/` 中执行：

```bash
MICROBIOME_INSTALL_PYTHON=python3.13 \
  bash upgrade.sh --prefix "$HOME/.local/share/microbiome-runtime"
"$HOME/.local/share/microbiome-runtime/bin/microbiome" --version
"$HOME/.local/share/microbiome-runtime/bin/microbiome" doctor
```

升级会新建运行时，通过验证后逐个切换受管理启动链接，并保留旧运行时。**0.2.5 的 CLI 多入口切换不是整体事务原子操作**；极端中断后可能需要核对五个入口是否指向同一运行时，再重新执行受管理升级或重新部署。不要宣传它拥有 Harness preset 的原子目录交换保证。

此 CLI 安装器没有 `--rollback`。需要回退时，可以从保留的可信旧 bundle 部署到一个新的独立 prefix，完成验证后把宿主配置和 PATH 有意切换过去；或者由维护人员根据受管理记录执行审阅过的修复。不要给 `upgrade.sh` 添加不存在的 `--rollback` 参数，也不要未经核对直接重写链接。〔I07〕

#### Harness 升级与回滚

Harness 升级就是使用新 bundle 再运行 `install_harness.py --bundle . --offline`，配合原来的 `--dsh-home` 与必要的 `--preset-root`。相同发布会复核并复用已有运行时，不同发布创建独立版本目录。之后必须新开会话，再做能力握手。

回滚命令为：

```bash
python3.13 scripts/install_harness.py \
  --dsh-home "$HOME/.dsh" --rollback
```

自定义 preset 根必须同时传回原值。回滚选择最近可用的旧模式备份，并验证其配置、运行环境和身份；没有备份或校验失败时，不会凭空制造一个旧环境。回滚后仍须使用新会话确认实际加载版本。

#### 停用入口与删除数据不同

CLI 停用：

```bash
bash uninstall.sh --prefix "$HOME/.local/share/microbiome-runtime"
```

Harness 模式停用：

```bash
python3.13 scripts/install_harness.py \
  --dsh-home "$HOME/.dsh" --uninstall
```

两者保留项目、数据库、运行环境和历史证据。它们不会卸载整个 DeepSeek Harness，不会退出所有宿主进程，也不会递归删除用户数据。是否清理旧运行时应另行检查：有没有活动进程、冻结锁、宿主配置或恢复记录仍引用它。不能把目录名“旧”作为可删除的充分依据。

### 备份、迁移与远程部署边界

备份至少应覆盖：原始输入或可验证的原始数据副本、`project_config.yaml`、`project_state.yaml`、计划和需求记录、批准和委托记录、资源注册、冻结锁、执行检查点、日志、结果及其索引。外部数据库应另有版本和指纹记录。发行 ZIP、SHA256、依赖清单和当前运行时身份也应一起保存。宿主账号凭据与分析交付资料分别管理，不混入公开报告。

备份应在项目没有活动写入时进行，或者使用能保证一致性的文件系统快照。只复制最终图片和报告无法恢复审计链；只复制项目状态而遗漏真实输出，也不能证明运行已经完成。

迁移到新机器时，推荐顺序是：复制可信发行包；在新机器重新运行安装器；恢复原始输入、项目及持久资源；确认路径和权限；在维护阶段重新定位和验证资源；运行安装与项目预检；生成适用于新机器的运行记录。不要直接复制旧 venv 后就开始生产，Python 启动脚本、环境前缀、数据库入口及 `runtime.json` 中存在本机绝对路径。

旧 READY 注册表和冻结锁不能通过批量字符串替换就变成新机器的有效锁。路径变化不仅是文本变化，还影响资源身份、marker、版本检查、消费者入口和文件指纹。原机器的锁应保留为历史证据，新机器形成新的验证记录。

远端部署的所有路径都指实际执行机器。用户在浏览器上传的小附件不等于服务器自动获得了指定目录；在 Windows 文件管理器看到的盘符也不等于 Linux 下可访问。路径错误应区分不存在、未挂载、无权限和声明角色不正确，而不是一律让用户重新复制数据。

容器部署可以作为管理员的工程方案，但当前公开包没有提供已经验收的通用容器镜像或集群调度配方。必须单独验证用户命名空间、bubblewrap、挂载权限、持久目录、进程取消、资源探针和冻结路径语义。这里不提供通过扩大容器权限绕过现有执行约束的操作。

### 常见安装与适配故障的处理顺序

| 现象 | 最先检查 | 应保存的证据 |
| --- | --- | --- |
| 离线 Python/平台不匹配 | 解释器次版本、系统与架构 | manifest 的构建字段、实际版本输出 |
| prefix 被拒绝 | 是否为已有非受管理目录或错误路径 | 错误信息、所选 prefix |
| `microbiome` 版本不对 | PATH 是否命中旧入口 | 完整路径和 `command -v` 输出 |
| Harness 模式未出现 | 实际 DSH_HOME、preset 根、同名遮蔽 | 安装回执与新会话选择情况 |
| 新模式仍是旧代码 | 是否一直使用旧会话 | capabilities 和插件身份 |
| MCP 连接不上 | 解释器、命令、stdio/HTTP 类型、执行机器 | 宿主 MCP 日志，导出配置中非敏感字段 |
| HTTP 地址能打开但工具不可用 | 是否访问 `/mcp`，是否为正确机器回环地址 | 服务启动命令、端口、宿主连接日志 |
| `FILE_POLICY_UNAVAILABLE` | bwrap 与内核命名空间 | 原始 bwrap 错误、操作系统信息 |
| 模型容量未知 | 宿主元数据接口是否提供容量 | 结构化容量诊断，不填写猜测数值 |
| doctor 通过但分析缺软件 | 科学资源尚未探测或未 READY | 资源发现与依赖预览回执 |
| 项目忙 | 是否已有活动操作或持久作业 | 原 job ID、项目状态，避免重复提交 |
| 升级后配置失效 | 导出配置是否仍指旧解释器 | 新旧 runtime 身份与实际导出文件 |

报告问题时优先提供版本、发布摘要、操作系统、Python 次版本、宿主版本、失败阶段、结构化错误码、对应日志路径和已经执行的操作。不要提交私有凭据、完整宿主账号配置或无关原始数据。文件名只是线索：例如存在 `compiler_last_failure.json` 不必然表示当前仍失败，应读取其中真实状态和错误字段。

### 本章命令核验与技术来源

本章编写时只运行只读帮助检查，没有安装、升级、联网调用模型或启动真实科学分析。下列八类命令的 `--help` 均返回 0；行为说明另由固定发行源码核对。

| 命令族 | 核验的关键参数 | 结果 |
| --- | --- | --- |
| `scripts/install_portable.py --help` | bundle、prefix、python、offline、upgrade、uninstall | 通过；没有 rollback |
| `scripts/install_harness.py --help` | bundle、dsh-home、preset-root、harness-source、offline、rollback、uninstall | 通过 |
| `microbiome export --help` | host、output、mcp-url | 通过 |
| `microbiome mcp --help` | transport、port | 通过 |
| `microbiome endpoint --help` | endpoint、check、project、prompt | 通过 |
| `scripts/verify_portable_release.py --help` | bundle、python/prefix 互斥、output | 通过 |
| `scripts/build_portable_release.py --help` | output、offline-deps、wheel-cache、no-index、verify | 通过 |
| `python -m microbiome.harness.smoke --help` | directory | 通过 |

以下路径相对于本次固定源码快照 `bundle/source/`，安装器同时在 `bundle/scripts/` 分发。行号对应 0.2.5（r10），后续版本可能变化。

| 引用 | 文件与定位 | 支持的事实 |
| --- | --- | --- |
| I01 | `AGENTS.md:1` | 本地输入、持久资源、阶段和冻结生产规则 |
| I02 | `pyproject.toml:7`、`pyproject.toml:45` | 包版本机制、Python 声明与控制台入口 |
| I03 | `docs/DeepSeek_Harness_Microbiome_Guide.md:44`；`bundle/manifest.json` | 平台、宿主版本和离线构建身份 |
| I04 | `scripts/install_harness.py:174` | Harness 参数、Linux 限制、版本和离线检查 |
| I05 | `microbiome/pipeline/process_file_policy.py:55`、`:65` | bwrap 要求、文件和网络边界，无非沙箱回退 |
| I06 | `scripts/install_portable.py:30` | 清单、路径、wheel 与依赖校验 |
| I07 | `scripts/install_portable.py:73`、`:100`、`:151`、`:187` | 所有权检查、安装/升级/卸载、逐入口切换 |
| I08 | `microbiome/cli.py:9`、`:40` | doctor 的实际范围、新 CLI 和别名行为 |
| I09 | `microbiome/integrations/capabilities.py:13`、`microbiome/harness/plugin.mjs:382` | 能力身份与宿主模型元数据 |
| I10 | `scripts/verify_portable_release.py:121` | 安装版验证器、全新输出目录与合成范围 |
| I11 | `microbiome/harness/smoke.py:10`、`:65` | 固定人工数据、真实执行检查和不覆盖范围 |
| I12 | `microbiome/integrations/cli.py:29`、`:60`、`microbiome/integrations/host_skills.py:35` | 导出条件、宿主差异、完整 Skill 指纹 |
| I13 | `microbiome/integrations/mcp_server.py:10`、`:230` | 通用工具集合、回环绑定和传输参数 |
| I14 | `microbiome/integrations/endpoint_cli.py:78`、`docs/portable_integration.md:59` | 显式模型端口及实际请求边界 |
| I15 | `scripts/install_harness.py:92`、`:195`、`:294` | 旧 preset 校验、回滚与原子切换 |

公开资产身份另外来自本机发布回执 `packaged/github-publication-20261019/github-publication-receipt.json` 和 `release-manifest.json`。宿主官方页面用于解释连接位置和产品操作，Microbiome 的参数、默认值和支持范围以该版本实际代码为准；官方页面可能随宿主版本更新。
