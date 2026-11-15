```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**為你的終端機準備的可設定 AI 智慧代理。**
自備 API 金鑰：Z.ai（智譜 GLM）、Anthropic、OpenAI、OpenRouter、本機 Ollama，或任何相容 OpenAI 的伺服器。首選 Linux 與 Termux。

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · 繁體中文

> **預 Alpha，版本 0.1.0-alpha.1。** 已實作：原生 TUI 介面、多供應商對話、權限感知的智慧代理迴圈、本機工具以及可恢復的本機工作階段。API 與設定仍可能變動。這是一個原型，並非與 OpenCode 或 Hermes 功能對等。

*Marelle* 是法語裡跳格子遊戲的名字。每一格都是一步。

## 安裝

**Android 上的 Termux。** 從[發行頁面](https://github.com/S-I-L-S/marelle/releases)下載預先建置的二進位檔，或在手機上自行建置。逐步指南：[docs/TERMUX.md](docs/TERMUX.md)。

**Linux，以及其他任何能執行 Rust 的地方。** 需要 Rust 1.85 或更新版本：

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

或者複製本存放庫並執行 `cargo build --release`。二進位檔為 `target/release/marelle`，約 4 MiB。

## 第一次對話

把金鑰放在只有你能讀取的檔案裡，並告訴 Marelle 它在哪。Marelle 絕不會把金鑰存進它的設定中。

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

把 `MODEL` 換成你供應商清單中的模型名稱。其他用法：

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

回答輸出到 stdout。推理過程、token 計數、備註和錯誤輸出到 stderr，因此 `marelle chat ... > answer.txt` 只保存回答本身。

## 智慧代理工具

Marelle 透過相容 OpenAI 與 Anthropic 的 API 傳送結構化的工具定義。內建工具集：

| 工具 | 用途 |
| --- | --- |
| `terminal` | 從啟動目錄執行 shell 命令 |
| `read_file`, `write_file`, `edit_file` | 讀取和修改 UTF-8 文字檔案 |
| `list_directory`, `search_text` | 檢視和搜尋專案檔案 |
| `fetch_url` | 以文字形式擷取公開的 HTTP(S) 頁面（需要 `curl`） |
| `list_skills`, `load_skill` | 探索並載入本機 `SKILL.md` 指引 |

用 `/tools` 檢視工具清單，用 `/skills` 列出可用技能。專案指引會從 `AGENTS.md`、`CLAUDE.md`、`SOUL.md`、`.cursorrules` 及選定的 `.marelle/` 檔案載入，前提是它們位於允許的路徑內。

檔案類工具被限制在 `permissions.allowed_paths` 中（預設：目前專案目錄），並遵守 `permissions.denied_paths`。讀取預設允許；檔案修改、shell 命令和網路擷取在執行前會先詢問。shell 權限針對主機上的命令，並非檔案系統沙箱：除非你有意讓模型執行命令，否則保持為 `ask`。

智慧代理在經過 `agent.max_turns` 輪工具往返後停止。工具回應被視為不可信內容。請求使用完整的模型回應，因此工具引數不會由不完整的串流片段拼湊而成。

## 工作階段

對話預設在本機保存，包括工具呼叫歷史以及模型與工作階段設定。在 Unix 上，工作階段目錄權限為 `0700`，JSON 檔案權限為 `0600`。檔案保存在 `$XDG_STATE_HOME/marelle/sessions.json`；若 `XDG_STATE_HOME` 未設定，則為 `~/.local/state/marelle/sessions.json`。API 金鑰絕不會寫入該檔案。用以下命令關閉持久化：

```sh
marelle config set agent.persist_sessions false
```

結束碼：`0` 成功，`1` 供應商或網路錯誤，`2` 用法或設定錯誤。

## 供應商

| id | 協定 | 金鑰變數 | |
| --- | --- | --- | --- |
| `zai` | 相容 OpenAI | `ZAI_API_KEY` | Z.ai（智譜 GLM），國際 |
| `zai_coding` | 相容 OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai，相容 Anthropic 的端點 |
| `zhipu` | 相容 OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel，中國 |
| `zhipu_coding` | 相容 OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan，中國 |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | 相容 OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | 相容 OpenAI | `OPENROUTER_API_KEY` | 一把金鑰背後的多個模型 |
| `ollama` | 相容 OpenAI | 無 | 本機上的 Ollama |

Z.ai 與 Zhipu 的端點資訊來自第三方文件，並非供應商官方頁面。請對照你的方案核實，如有出入可覆寫任何欄位。

供應商的每個欄位都可修改，新供應商就是一個帶位址的名稱：

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

供應商的其他欄位：`kind`（`openai-compatible` 或 `anthropic`）、`api_key_file`、`auth`（`bearer`、`x-api-key`、`api-key`、`none`）、`max_tokens_field`、`stream_usage` 與 `extra_body`。若 `api_key_env` 所指名的變數已設定，它優先；否則讀取檔案。

## 設定

設定分層而來：內建預設值，然後是環境變數，再是旗標。設定檔位於預設值與環境之間。

- `MARELLE_CHAT__MODEL=zai/MODEL` 設定 `chat.model`。雙底線代表一個點。
- `marelle config show [prefix]` 列出各項設定及各自來自的層。
- `marelle config explain <key>` 顯示設定過某個鍵值的每一層，從最低優先權到最高。

| 鍵值 | 預設 | 意義 |
| --- | --- | --- |
| `chat.provider`, `chat.model` | 無 | 與誰對話。`chat.model` 可以是 `provider/model`。 |
| `chat.system` | 無 | 系統提示詞 |
| `chat.max_tokens`, `chat.temperature` | 無 | 未設定時交由供應商 |
| `chat.stream` | `true` | 串流輸出可選的裁判回應；啟用工具的代理輪次目前使用完整回應，以確保工具呼叫完整無損 |
| `chat.show_reasoning`, `chat.show_usage` | `false` | 推理過程與 token 計數輸出到 stderr |
| `agent.max_turns` | `20` | 每個使用者請求的工具往返上限 |
| `agent.persist_sessions` | `true` | 在多次執行之間保存本機對話 |
| `permissions.shell` | `ask` | Shell 命令執行：`allow`、`ask`、`deny` |
| `permissions.files_read` | `allow` | 檔案讀取：`allow`、`ask`、`deny` |
| `permissions.files_write` | `ask` | 檔案修改：`allow`、`ask`、`deny` |
| `permissions.network` | `ask` | 擷取 URL：`allow`、`ask`、`deny` |
| `permissions.allowed_paths` | `["."]` | 檔案工具允許的根路徑 |
| `permissions.denied_paths` | 無 | 明確禁止的檔案路徑 |
| `network.connect_timeout_secs` | `15` | 允許建立連線的時間 |
| `network.idle_timeout_secs` | `300` | 等待資料時允許的靜默時間 |
| `network.proxy` | `system` | `system`、`none`，或代理位址 |
| `network.ca_bundle` | 無 | 額外的受信任憑證機構檔案 |
| `network.allow_insecure_http` | `false` | 允許金鑰經純 `http` 傳往非本機主機 |
| `update.check` | `true` | 互動工作階段啟動時向 GitHub 和 GitLab 詢問更新版本 |
| `update.github_repo` | `S-I-L-S/marelle` | 檢查向其詢問最新發行的 GitHub `owner/repo` |
| `update.gitlab_repo` | 無 | 同一檢查也會詢問的 GitLab `owner/repo`。留空則停用 GitLab |
| `update.timeout_secs` | `3` | 檢查在每個來源上可用的秒數 |

Marelle 自帶受信任憑證機構清單，因此在系統沒有此類清單的地方（如 Termux）也能運作。`SSL_CERT_FILE` 可追加更多。

發現更新版本時，工作階段會提及一次——版本號、你目前的版本，以及取得位址。優先詢問正式發行；只用標籤標記提交的存放庫則退回到最新的標籤。任何失敗都保持沉默。

## 你的金鑰仍是你的

- 設定只命名存放金鑰的變數或檔案，絕不命名金鑰本身。像 `providers.x.api_key` 這樣的設定會被拒絕。
- `config show`、`config explain` 和 `providers` 不會列印金鑰，且金鑰會從任何來自伺服器的錯誤文字中剔除。
- 它只傳送到供應商的位址，放在單一請求標頭中；除非你允許，絕不經純 `http` 傳往其他機器。
- 不追隨重新導向，因此金鑰不會被轉送到別處。

## 語言

介面支援英語、俄語、法語和烏克蘭語。用 `--lang <code>` 或 `MARELLE_LANG` 選擇；否則由 `LC_ALL`、`LC_MESSAGES` 和 `LANG` 決定。每則訊息都存於目錄中，若有語言遺漏一條，測試即失敗。目標是涵蓋歐亞大陸的每種語言，繼而更多。

## 路線圖

1. **對話。** 相容 OpenAI 與 Anthropic 的介接器，同時支援 SSE 串流與完整回應解碼。啟用工具的代理輪次目前使用完整回應。 *完成。*
2. **設定與外觀。** TOML 設定檔案，全域與按專案，配合 `config set`、`get`、`unset`，每個鍵值都有描述。受 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 啟發的主題化介面：橫幅、資訊面板、帶框回答、載入指示，預設紫藤色，任何顏色均可按 HEX 設定。 *完成。*
3. **多模型。** 模型別名、同時詢問多個模型並比較回應、後備鏈，以及融合它們的裁判模型。 *完成。*
4. **行動。** 為相容 OpenAI 與 Anthropic 的供應商提供結構化工具呼叫、本機檔案／搜尋／終端機工具、URL 擷取、本機技能以及 allow/ask/deny 權限；記錄會顯示每次工具呼叫及其結果。 *完成。*
5. **記憶。** 具名對話在多次執行之間本機持久化。長期語義記憶尚在前方。
6. **擴充。** MCP 伺服器、外部外掛、子代理以及更豐富的技能支援。
7. **觸及。** 瀏覽器自動化、訊息閘道和排程任務。

設計背後的考量見 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

## 已知限制

- 供應商與模型選擇器列出已設定的供應商和模型別名。Marelle 尚未探索遠端模型目錄。
- 選擇器支援滑鼠點擊聚焦和滾輪捲動；記錄與輸入編輯器仍由鍵盤驅動。
- Ctrl-C 會離開互動工作階段。目前尚不能只取消當前回應。
- 代理迴圈、權限感知的本機工具和持久的本機工作階段均已實作，但 Marelle 仍是預 Alpha 原型；目前設計與邊界見 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

## 開發

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

測試從不觸網：它們與 `crates/marelle-testkit` 中的模擬伺服器對話。GitHub Actions 也會建置 Android 二進位檔並檢查宣告的最低 Rust 版本。GitLab 鏡像提供 GitLab CI 管線；參見 [docs/GITLAB.md](docs/GITLAB.md)。

## 授權條款

[AGPL-3.0-only](LICENSE)。第三方聲明見 [NOTICE](NOTICE)。發行歸檔包含面向二進位檔內 Rust 箱的 `THIRD-PARTY-LICENSES.txt`。
