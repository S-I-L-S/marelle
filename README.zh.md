```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**为你的终端准备的可配置 AI 智能体。**
自带 API 密钥：Z.ai（智谱 GLM）、Anthropic、OpenAI、OpenRouter、本地 Ollama，或任何兼容 OpenAI 的服务器。首选 Linux 和 Termux。

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · 简体中文 · [繁體中文](README.zh-Hant.md)

> **预 Alpha，版本 0.1.0-alpha.1。** 已实现：原生 TUI 界面、多供应商对话、权限感知的智能体循环、本地工具以及可恢复的本地会话。API 与设置仍可能变化。这是一个原型，并非与 OpenCode 或 Hermes 功能对等。

*Marelle* 是法语里跳房子游戏的名字。每一格都是一步。

## 安装

**Android 上的 Termux。** 从[发布页](https://github.com/S-I-L-S/marelle/releases)下载预编译的二进制文件，或在手机上自行构建。逐步指南：[docs/TERMUX.md](docs/TERMUX.md)。

**Linux，以及其他任何能运行 Rust 的地方。** 需要 Rust 1.85 或更新版本：

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

或者克隆本仓库并运行 `cargo build --release`。二进制文件为 `target/release/marelle`，约 4 MiB。

## 第一次对话

把密钥放在只有你能读取的文件里，并告诉 Marelle 它在哪。Marelle 绝不会把密钥存进它的设置中。

```sh
mkdir -p ~/.config/marelle
( umask 077; read -rsp "API key: " key; echo; printf '%s' "$key" > ~/.config/marelle/zai.key )
export MARELLE_PROVIDERS__ZAI__API_KEY_FILE=~/.config/marelle/zai.key

marelle providers                                    # what exists, and whether the key was found
marelle config set chat.provider zai
marelle config set chat.model glm-5.2
marelle                                                # open the interactive terminal UI
marelle chat -m zai/MODEL "Explain hopscotch in one sentence"
```

把 `MODEL` 换成你供应商列表中的模型名。其他用法：

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

回答输出到 stdout。推理过程、token 计数、备注和错误输出到 stderr，因此 `marelle chat ... > answer.txt` 只保存答案本身。

## 智能体工具

Marelle 通过兼容 OpenAI 与 Anthropic 的 API 发送结构化的工具定义。内置工具集：

| 工具 | 用途 |
| --- | --- |
| `terminal` | 从启动目录运行 shell 命令 |
| `read_file`, `write_file`, `edit_file` | 读取和修改 UTF-8 文本文件 |
| `list_directory`, `search_text` | 查看和检索项目文件 |
| `fetch_url` | 以文本形式抓取公开的 HTTP(S) 页面（需要 `curl`） |
| `list_skills`, `load_skill` | 发现并加载本地 `SKILL.md` 指令 |

用 `/tools` 查看工具列表，用 `/skills` 列出可用技能。项目指引会从 `AGENTS.md`、`CLAUDE.md`、`SOUL.md`、`.cursorrules` 及选定的 `.marelle/` 文件中加载，前提是它们位于允许的路径内。

文件类工具被限制在 `permissions.allowed_paths` 中（默认：当前项目目录），并遵守 `permissions.denied_paths`。读取默认允许；文件修改、shell 命令和网络抓取在执行前会先询问。shell 权限针对主机上的命令，并非文件系统沙箱：除非你有意让模型执行命令，否则保持为 `ask`。

智能体在经过 `agent.max_turns` 轮工具往返后停止。工具响应被视为不可信内容。请求使用完整的模型响应，因此工具参数不会由不完整的流片段拼凑而成。

## 会话

对话默认在本地保存，包括工具调用历史以及模型与会话设置。在 Unix 上，会话目录权限为 `0700`，JSON 文件权限为 `0600`。文件保存在 `$XDG_STATE_HOME/marelle/sessions.json`；若 `XDG_STATE_HOME` 未设置，则为 `~/.local/state/marelle/sessions.json`。API 密钥绝不会写入该文件。用以下命令关闭持久化：

```sh
marelle config set agent.persist_sessions false
```

退出码：`0` 成功，`1` 供应商或网络错误，`2` 用法或配置错误。

## 供应商

| id | 协议 | 密钥变量 | |
| --- | --- | --- | --- |
| `zai` | 兼容 OpenAI | `ZAI_API_KEY` | Z.ai（智谱 GLM），国际 |
| `zai_coding` | 兼容 OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai，兼容 Anthropic 的端点 |
| `zhipu` | 兼容 OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel，中国 |
| `zhipu_coding` | 兼容 OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan，中国 |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | 兼容 OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | 兼容 OpenAI | `OPENROUTER_API_KEY` | 一个密钥背后的多个模型 |
| `ollama` | 兼容 OpenAI | 无 | 本机上的 Ollama |

Z.ai 与 Zhipu 的端点信息来自第三方文档，并非供应商官方页面。请对照你的套餐核实，如有出入可覆盖任意字段。

供应商的每个字段都可修改，新供应商就是一个带地址的名字：

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

供应商的其他字段：`kind`（`openai-compatible` 或 `anthropic`）、`api_key_file`、`auth`（`bearer`、`x-api-key`、`api-key`、`none`）、`max_tokens_field`、`stream_usage` 和 `extra_body`。若 `api_key_env` 所指名的变量已设置，它优先；否则读取文件。

## 设置

设置分层而来：内置默认值，然后是环境变量，再是标志。设置文件介于默认值与环境之间。

- `MARELLE_CHAT__MODEL=zai/MODEL` 设置 `chat.model`。双下划线代表一个点。
- `marelle config show [prefix]` 列出各项设置及各自来自的层。
- `marelle config explain <key>` 显示设置过某个键的每一层，从最低优先级到最高。

| 键 | 默认 | 含义 |
| --- | --- | --- |
| `chat.provider`, `chat.model` | 无 | 与谁对话。`chat.model` 可以是 `provider/model`。 |
| `chat.system` | 无 | 系统提示词 |
| `chat.max_tokens`, `chat.temperature` | 无 | 未设置时交给供应商 |
| `chat.stream` | `true` | 流式输出可选的裁判应答；启用工具的智能体轮次目前使用完整响应，以保证工具调用完整无损 |
| `chat.show_reasoning`, `chat.show_usage` | `false` | 推理过程与 token 计数输出到 stderr |
| `agent.max_turns` | `20` | 每个用户请求的工具往返上限 |
| `agent.persist_sessions` | `true` | 在多次运行之间保存本地对话 |
| `permissions.shell` | `ask` | Shell 命令执行：`allow`、`ask`、`deny` |
| `permissions.files_read` | `allow` | 文件读取：`allow`、`ask`、`deny` |
| `permissions.files_write` | `ask` | 文件修改：`allow`、`ask`、`deny` |
| `permissions.network` | `ask` | 抓取 URL：`allow`、`ask`、`deny` |
| `permissions.allowed_paths` | `["."]` | 文件工具允许的根路径 |
| `permissions.denied_paths` | 无 | 明确禁止的文件路径 |
| `network.connect_timeout_secs` | `15` | 允许建立连接的时间 |
| `network.idle_timeout_secs` | `300` | 等待数据时允许的静默时间 |
| `network.proxy` | `system` | `system`、`none`，或代理地址 |
| `network.ca_bundle` | 无 | 额外的受信任证书机构文件 |
| `network.allow_insecure_http` | `false` | 允许密钥经纯 `http` 发往非本地主机 |
| `update.check` | `true` | 交互会话启动时向 GitHub 和 GitLab 询问更新版本 |
| `update.github_repo` | `S-I-L-S/marelle` | 检查向其询问最新发布的 GitHub `owner/repo` |
| `update.gitlab_repo` | 无 | 同一检查也会询问的 GitLab `owner/repo`。留空则禁用 GitLab |
| `update.timeout_secs` | `3` | 检查在每个来源上可用的秒数 |

Marelle 自带受信任证书机构列表，因此在系统没有此类列表的地方（如 Termux）也能工作。`SSL_CERT_FILE` 可追加更多。

发现更新版本时，会话会提及一次——版本号、你当前的版本，以及获取地址。优先询问正式发布；只用标签标记提交的仓库则退回到最新的标签。任何失败都保持沉默。

## 你的密钥仍是你的

- 设置只命名存放密钥的变量或文件，绝不命名密钥本身。像 `providers.x.api_key` 这样的设置会被拒绝。
- `config show`、`config explain` 和 `providers` 不会打印密钥，且密钥会从任何来自服务器的错误文本中剔除。
- 它只发送到供应商的地址，放在单一请求头中；除非你允许，绝不经纯 `http` 发往其他机器。
- 不跟随重定向，因此密钥不会被转发到别处。

## 语言

界面支持英语、俄语、法语和乌克兰语。用 `--lang <code>` 或 `MARELLE_LANG` 选择；否则由 `LC_ALL`、`LC_MESSAGES` 和 `LANG` 决定。每条消息都存于目录中，若有语言遗漏一条，测试即失败。目标是覆盖欧亚大陆的每种语言，继而更多。

## 路线图

1. **对话。** 兼容 OpenAI 与 Anthropic 的适配器，既支持 SSE 流式，也支持完整响应解码。启用工具的智能体轮次目前使用完整响应。*完成。*
2. **设置与外观。** TOML 设置文件，全局与按项目，配合 `config set`、`get`、`unset`，每个键都有描述。受 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 启发的主题化界面：横幅、信息面板、带框回答、加载指示，默认紫藤色，任何颜色均可按 HEX 设置。*完成。*
3. **多模型。** 模型别名、同时询问多个模型并比较回答、回退链，以及融合它们的裁判模型。*完成。*
4. **行动。** 为兼容 OpenAI 与 Anthropic 的供应商提供结构化工具调用、本地文件/搜索/终端工具、URL 抓取、本地技能以及 allow/ask/deny 权限；记录屏会显示每次工具调用及其结果。*完成。*
5. **记忆。** 命名对话在多次运行之间本地持久化。长期语义记忆尚在前方。
6. **扩展。** MCP 服务器、外部插件、子智能体以及更丰富的技能支持。
7. **触达。** 浏览器自动化、消息网关和定时任务。

设计背后的考量见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

## 已知限制

- 供应商与模型选择器列出已配置的供应商和模型别名。Marelle 尚未发现远程模型目录。
- 选择器支持鼠标点击聚焦和滚轮滚动；记录屏与输入编辑器仍由键盘驱动。
- Ctrl-C 会退出交互会话。目前尚不能只取消当前回答。
- 智能体循环、权限感知的本地工具和持久的本地会话均已实现，但 Marelle 仍是预 Alpha 原型；当前设计与边界见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

## 开发

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

测试从不触网：它们与 `crates/marelle-testkit` 中的模拟服务器对话。GitHub Actions 也会构建 Android 二进制并检查声明的最低 Rust 版本。GitLab 镜像提供 GitLab CI 流水线；参见 [docs/GITLAB.md](docs/GITLAB.md)。

## 许可证

[AGPL-3.0-only](LICENSE)。第三方声明见 [NOTICE](NOTICE)。发布归档包含面向二进制内 Rust 箱的 `THIRD-PARTY-LICENSES.txt`。
