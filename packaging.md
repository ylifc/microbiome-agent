# 发行包安装与自检

发行包将 Microbiome 的完整运行源码、测试、22 份流程参考、141 个项目 Skill、文档和宿主模板放在一起。141 个 Skill 包括 107 个软件 Skill、33 个数据库 Skill 和 1 个宿主工作流 Skill。数量和文件内容由构建时实际扫描与 SHA256 清单核对。

科学软件、数据库、真实测序数据及用户账户不随包复制；安装这些资源仍需当前项目已批准的依赖计划。旧构建目录和 `github_upload` 快照不是当前源码，不重复打包。

## 目录用途

| 路径 | 内容 |
| --- | --- |
| `bundle/source/` | 当前运行源码、测试、构建脚本、环境描述和文档的完整发布快照 |
| `bundle/source/docs/` | 完整中文 Word 手册、Markdown 文档和全部 Skill 索引；相对源码链接在此保持有效 |
| `bundle/docs/README.md` | 指向完整文档的导航 |
| `bundle/wheels/` | 当前 Agent 的安装 wheel |
| `bundle/wheelhouse/` | 离线版的 Python 运行及源码构建依赖 |
| `bundle/skills/` | 可浏览的资源与宿主 Skill 原文镜像 |
| `bundle/workflows/` | 每个流程的 YAML 元数据与完整指南 |
| `bundle/integrations/templates/` | 宿主 Skill、Codex 与 VS Code 配置模板 |
| `bundle/catalog/` | 源码、依赖、Skills、流程和宿主模板的机器可读清单 |
| `bundle/examples/portable_demo/` | 安装后可执行的小文件统计演示 |
| `bundle/manifest.json` | 每个文件的 SHA256、构建环境和能力清单 |
| 发行目录中的压缩包与 `SHA256SUMS` | 方便复制和校验的完整安装包 |
| 发行目录中的 `validation/` | 本次实际安装与模拟运行的证据；独立于不可变 bundle |

源码中的文档相对路径以 `source/` 为项目根目录。不要单独移动 Word 手册后期待源码链接仍能定位；共享时保留发行目录结构。

## 安装

在 Linux 或 WSL 中，直接使用发行目录时进入 `bundle/`；复制压缩包时先解压，进入生成的 `microbiome-portable/`。两个目录的内容相同。运行：

```bash
bash install.sh
"$HOME/.local/share/microbiome-runtime/bin/microbiome" doctor
```

也可指定全新目录：

```bash
bash install.sh --prefix /absolute/new/microbiome-runtime
/absolute/new/microbiome-runtime/bin/microbiome doctor
```

离线版默认使用随包依赖且不访问包索引，要求匹配构建记录中的 Python 次版本、操作系统和架构。本次提供 Linux x86_64 / Python 3.13 离线依赖；其他环境可用 `source/` 重新构建。源码声明 Python 3.11+，不表示本次已验证所有操作系统和 Python 版本。

安装器创建独立 venv，拒绝覆盖非受管目录，核对入口映射后再执行自检，不继承系统第三方包、不写入模型配置。升级用 `bash upgrade.sh --prefix 原目录`；卸载用 `bash uninstall.sh --prefix 原目录`，会保留运行环境和分析数据。

## 简单任务与宿主接入

```bash
microbiome stats /data/reads/input.fasta --output /work/new-statistics
microbiome stats /data/reads/R1.fastq.gz /data/reads/R2.fastq.gz \
  --paired --output /work/new-paired-statistics
microbiome export --host codex --output /work/new-codex-adapter
```

没有把安装目录加入 PATH 时，使用上面的完整 `bin/microbiome` 路径。输出父目录须已存在，输出目录本身须为新目录且位于原始输入目录之外。

统计任务不用模型或科研项目，提供记录数、长度、GC 和配对核验；累计上限为 256 MiB 解压内容及 200 万条记录（双端两侧合计）。它不会批准其他分析、推断质量编码或生成分类结论。
宿主导出支持 Codex、VS Code 和 WorkBuddy；MCP 与已有模型端口的区别见[接入说明](portable_integration.md)。

## 模拟安装后的验证

在任意工作目录，以 Python 3.11+ 运行发行包自检脚本。`--prefix` 使用安装器记录定位实际 venv，不会扫描或更换模型：

```bash
python /absolute/bundle/scripts/verify_portable_release.py \
  --bundle /absolute/bundle \
  --prefix /absolute/installed/microbiome-runtime \
  --output /absolute/new/verification-results
```

脚本核对整个 bundle 的哈希，确认导入来自全新 venv，逐文件比较已安装代码与源码清单，并检查依赖一致性、五个公开启动命令、流程目录、宿主导出、项目初始化及轻量任务。`--prefix` 还会核对安装目录中的启动符号链接和受管记录；仅 Python 模块能导入不足以判定安装成功。
人工 FASTA 的预期值是 2 条序列、16 个碱基、8 个 G/C 和 14 个非歧义碱基；双端文件各含 2 条记录。坏 FASTQ、缺失输入和重复输出目录应失败，原始文件保持不变。

完整冻结生命周期另用合成数据测试输入确认、依赖范围、预检、冻结、生产批准与确定性重放。科研流程还需在真实任务上验证工具、数据库和质量指标，工程测试不等同于所有科学路线已验收。

## 重建与校验

```bash
python source/scripts/build_portable_release.py --verify /absolute/bundle
python source/scripts/build_portable_release.py \
  --output /absolute/new/release --offline-deps \
  --wheel-cache /absolute/bundle/wheelhouse --no-index
```

重建环境需要先安装 `build-requirements.in` 和 `requirements.in`，离线时使用 `wheelhouse` 与 `requirements.lock`。构建器不会覆盖非空发行目录，只有全部清单、镜像与 wheel 内容核验通过才发布成品。生成的包具有自己的构建时间、源码指纹与校验和。
