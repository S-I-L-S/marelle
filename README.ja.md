```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**あなたのターミナルのための、設定可能な AI エージェント。**
API キーは自分で持ち込んでください：Z.ai（Zhipu GLM）、Anthropic、OpenAI、OpenRouter、ローカルの Ollama、あるいは OpenAI 互換の任意のサーバー。まずは Linux と Termux から。

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · 日本語 · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **プレアルファ、バージョン 0.1.0-alpha.1。** 実装済み：ネイティブ TUI、マルチプロバイダーのチャット、権限を意識したエージェントループ、ローカルツール、再開できるローカルセッション。API や設定はまだ変わる可能性があります。これはプロトタイプであり、OpenCode や Hermes との機能対等ではありません。

*Marelle* はフランス語で跳びはね遊び（はねつき）の名前。マスの一つひとつが、一歩。

## インストール

**Android の Termux。** [リリースページ](https://github.com/S-I-L-S/marelle/releases)からビルド済みバイナリをダウンロードするか、スマホ上でビルドしてください。手順ガイド：[docs/TERMUX.md](docs/TERMUX.md)。

**Linux、および Rust が動くすべての場所。** Rust 1.85 以上が必要です：

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

あるいはリポジトリをクローンして `cargo build --release` を実行してください。バイナリは `target/release/marelle`、約 4 MiB です。

## 最初のチャット

キーは自分だけが読めるファイルに置き、その場所を Marelle に伝えてください。Marelle がキーを設定の中に保存することは決してありません。

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

`MODEL` はプロバイダーのリストにあるモデル名に置き換えてください。その他の使い方：

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

回答は stdout に出ます。思考過程、トークン数、注記、エラーは stderr に出るので、`marelle chat ... > answer.txt` とすれば回答だけが保存されます。

## エージェントのツール

Marelle は、OpenAI 互換および Anthropic の API を通じて、構造化されたツール定義を送ります。内蔵ツール一式：

| ツール | 目的 |
| --- | --- |
| `terminal` | 起動ディレクトリからシェルコマンドを実行する |
| `read_file`, `write_file`, `edit_file` | UTF-8 のテキストファイルを読み、変更する |
| `list_directory`, `search_text` | プロジェクトのファイルを一覧し、検索する |
| `fetch_url` | 公開 HTTP(S) ページをテキストとして取得する（`curl` が必要） |
| `list_skills`, `load_skill` | ローカルの `SKILL.md` 指示を見つけて読み込む |

`/tools` でツール一覧を、`/skills` で利用可能なスキルを確認できます。プロジェクトの指針は、`AGENTS.md`、`CLAUDE.md`、`SOUL.md`、`.cursorrules`、および選択された `.marelle/` ファイルから、許可されたパス内にある場合に読み込まれます。

ファイル系ツールは `permissions.allowed_paths` に制限され（既定：現在のプロジェクトディレクトリ）、`permissions.denied_paths` を尊重します。読み取りは既定で許可されています。ファイルの変更、シェルコマンド、ネットワーク取得は、実行前に尋ねます。シェル権限はホスト上のコマンドに適用されるものであり、ファイルシステムのサンドボックスではありません：モデルにコマンド実行を意図的に任せるつもりでなければ、`ask` のままにしてください。

エージェントは `agent.max_turns` 回のツール往復の後に停止します。ツールの応答は信頼できない内容として扱われます。リクエストは完全なモデル応答を使うため、ツールの引数が不完全なストリーム断片から組み立てられることはありません。

## セッション

会話は既定でローカルに保存されます。ツール呼び出しの履歴、モデルとセッションの設定を含みます。Unix ではセッションディレクトリはモード `0700`、JSON ファイルはモード `0600` です。ファイルは `$XDG_STATE_HOME/marelle/sessions.json` に、未設定なら `~/.local/state/marelle/sessions.json` に保存されます。API キーがこのファイルに書き込まれることは決してありません。永続化を無効にするには：

```sh
marelle config set agent.persist_sessions false
```

終了コード：`0` 成功、`1` プロバイダーまたはネットワークのエラー、`2` 使用法または設定のエラー。

## プロバイダー

| id | プロトコル | キー変数 | |
| --- | --- | --- | --- |
| `zai` | OpenAI 互換 | `ZAI_API_KEY` | Z.ai（Zhipu GLM）、国際版 |
| `zai_coding` | OpenAI 互換 | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai、Anthropic 互換エンドポイント |
| `zhipu` | OpenAI 互換 | `ZHIPUAI_API_KEY` | Zhipu BigModel、中国 |
| `zhipu_coding` | OpenAI 互換 | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan、中国 |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI 互換 | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI 互換 | `OPENROUTER_API_KEY` | 一つのキーの背後にある多数のモデル |
| `ollama` | OpenAI 互換 | なし | このマシン上の Ollama |

Z.ai と Zhipu のエンドポイントは第三者による文書から来ており、ベンダー自身のページからではありません。ご自身のプランと照らして確認し、異なる場合は任意の項目を上書きしてください。

プロバイダーのすべての項目は変更でき、新しいプロバイダーとは、名前とアドレスの組み合わせのことです：

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

プロバイダーの他の項目：`kind`（`openai-compatible` または `anthropic`）、`api_key_file`、`auth`（`bearer`、`x-api-key`、`api-key`、`none`）、`max_tokens_field`、`stream_usage`、`extra_body`。`api_key_env` が指す名前の変数が設定されていればそれが優先され、そうでなければファイルが読まれます。

## 設定

設定は層として届きます：組み込みの既定値、次に環境変数、次にフラグ。設定ファイルは既定値と環境の間に割り込みます。

- `MARELLE_CHAT__MODEL=zai/MODEL` は `chat.model` を設定します。二重の下線は点を表します。
- `marelle config show [prefix]` は設定と、それぞれがどの層から来たかを一覧します。
- `marelle config explain <key>` は、あるキーを設定したすべての層を、優先度の低い順に高い順まで示します。

| キー | 既定 | 意味 |
| --- | --- | --- |
| `chat.provider`, `chat.model` | なし | 誰と話すか。`chat.model` は `provider/model` でもかまいません。 |
| `chat.system` | なし | システムプロンプト |
| `chat.max_tokens`, `chat.temperature` | なし | 未設定ならプロバイダーに任せます |
| `chat.stream` | `true` | 任意のジャッジ応答のストリーミング。ツール使用のエージェントターンは現在、完全な応答を使います。ツール呼び出しが壊れないようにするためです |
| `chat.show_reasoning`, `chat.show_usage` | `false` | 思考過程とトークン数を stderr に表示 |
| `agent.max_turns` | `20` | ユーザー要求ひとつあたりのツール往復の上限 |
| `agent.persist_sessions` | `true` | ローカルの会話を起動間で保存する |
| `permissions.shell` | `ask` | シェルコマンドの実行：`allow`、`ask`、`deny` |
| `permissions.files_read` | `allow` | ファイルの読み取り：`allow`、`ask`、`deny` |
| `permissions.files_write` | `ask` | ファイルの変更：`allow`、`ask`、`deny` |
| `permissions.network` | `ask` | URL の取得：`allow`、`ask`、`deny` |
| `permissions.allowed_paths` | `["."]` | ファイルツールに許されたルート |
| `permissions.denied_paths` | なし | 明示的に拒否されたファイルパス |
| `network.connect_timeout_secs` | `15` | 接続に許される時間 |
| `network.idle_timeout_secs` | `300` | データ待ちに許される沈黙 |
| `network.proxy` | `system` | `system`、`none`、またはプロキシのアドレス |
| `network.ca_bundle` | なし | 信頼する認証局を追加するファイル |
| `network.allow_insecure_http` | `false` | キーが生の `http` で別ホストへ行くのを許可する |
| `update.check` | `true` | 対話セッションの開始時に、GitHub と GitLab に新しいバージョンがないか尋ねる |
| `update.github_repo` | `S-I-L-S/marelle` | チェックが最新リリースを尋ねる GitHub `owner/repo` |
| `update.gitlab_repo` | なし | 同じチェックが尋ねる GitLab `owner/repo`。空なら GitLab は無効 |
| `update.timeout_secs` | `3` | チェックが一つの情報源に割ける秒数 |

Marelle は信頼する認証局のリストを自身で持ち歩くため、Termux のようにシステムに無い場所でも動きます。`SSL_CERT_FILE` で追加できます。

新しいバージョンが見つかると、セッションはそれを一度だけ告げます — バージョン、あなたが持っているもの、そして入手先。まずリリースを尋ねます；コミットをタグでしか印していないリポジトリは、最新のタグを返します。何が失敗しても、静かに留まります。

## キーはあなたのまま

- 設定は、キーが眠る変数やファイルの名前を指すのであって、キーそのものを指すことは決してありません。`providers.x.api_key` のような設定は拒まれます。
- キーは `config show`、`config explain`、`providers` からは印刷されず、サーバーから返ってくるあらゆるエラーテキストからも取り除かれます。
- キーはプロバイダーのアドレスへ、ひとつのヘッダーでだけ送られ、許可しない限り生の `http` で別のマシンへ行くことは決してありません。
- リダイレクトは追われません。だからキーがよそへ運ばれることはありません。

## 言語

インターフェースは英語、ロシア語、フランス語、ウクライナ語で話します。`--lang <code>` か `MARELLE_LANG` で選びます。そうでなければ `LC_ALL`、`LC_MESSAGES`、`LANG` が決めます。すべてのメッセージはカタログの中に生きており、どれかの言語が一つ欠けばテストが落ちます。目標は、ユーラシアのすべての言語、そしてさらにその先です。

## ロードマップ

1. **話す。** OpenAI 互換と Anthropic のアダプター。SSE ストリーミングと完全応答の解読の両方に対応。ツール使用のエージェントターンは現在、完全な応答を使います。 *完了。*
2. **設定と外観。** TOML の設定ファイル。全体とプロジェクト別。`config set`、`get`、`unset` と、各キーの説明付き。[Hermes Agent](https://github.com/NousResearch/hermes-agent) の精神に沿うテーマ化されたインターフェース：バナー、情報パネル、枠付きの応答、スピナー、既定はフジの紫、あらゆる色は HEX で設定可能。 *完了。*
3. **多くのモデル。** モデルの別名。複数のモデルに同時に問い、応答を比べる。フォールバックの鎖。そしてそれらを溶け合わせるジャッジのモデル。 *完了。*
4. **行動する。** OpenAI 互換と Anthropic プロバイダーのための構造化されたツール呼び出し。ローカルのファイル・検索・ターミナルツール。URL の取得。ローカルのスキルと allow/ask/deny の権限。トランスクリプトはすべてのツール呼び出しとその結果を示します。 *完了。*
5. **覚える。** 名前をつけた会話は、起動の間、ローカルに残ります。長期の意味記憶はまだ先です。
6. **拡げる。** MCP サーバー、外部プラグイン、下位エージェント、そしてより豊かなスキルの支援。
7. **届ける。** ブラウザーの自動化、メッセージングのゲートウェイ、予定されたタスク。

設計の背後にある考えは [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) にあります。

## 既知の限界

- プロバイダーとモデルのセレクターは、設定済みのプロバイダーとモデルの別名を一覧します。Marelle はまだ遠方のモデルカタログを発見しません。
- セレクターはフォーカスのためのマウスクリックとホイールでのスクロールに対応します。トランスクリプトと入力エディタは引き続きキーボード駆動です。
- Ctrl-C は対話セッションを終えます。現時点では、現在の応答だけを取り消すことはしません。
- エージェントループ、権限を意識したローカルツール、持続するローカルセッションは実装されていますが、Marelle はなおプレアルファのプロトタイプです。現状の設計と限界は [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) を見てください。

## 開発

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

テストは決してネットワークに触れません：`crates/marelle-testkit` のモックサーバーと話します。GitHub Actions は Android のバイナリもビルドし、宣言された最小 Rust バージョンを検査します。GitLab ミラーのための GitLab CI パイプラインもあります。[docs/GITLAB.md](docs/GITLAB.md) を見てください。

## ライセンス

[AGPL-3.0-only](LICENSE)。第三者の通知は [NOTICE](NOTICE) にあります。リリースの書庫には、バイナリの中の Rust クレートのための `THIRD-PARTY-LICENSES.txt` が含まれます。
