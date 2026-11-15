```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**আপনার টার্মিনালের জন্য একটি কনফিগারযোগ্য AI এজেন্ট।**
নিজের API keys নিয়ে আসুন: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, একটি লোকাল Ollama, অথবা যেকোনো OpenAI-compatible server। Linux এবং Termux প্রথমে।

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · বাংলা · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, সংস্করণ 0.1.0-alpha.1।** নেটিভ TUI, multi-provider chat, permission-aware agent loop, লোকাল tools এবং পুনরায় চালু করা যায় এমন লোকাল sessions বাস্তবায়িত হয়েছে। API এবং সেটিংস এখনও বদলাতে পারে। এটি একটি prototype, OpenCode বা Hermes-এর সঙ্গে feature parity নয়।

*Marelle* ফরাসি ভাষায় hopscotch অর্থাৎ বাচ্চাদের লাফানোর খেলা বোঝায়। প্রতিটি ঘর একটি পদক্ষেপ।

## ইনস্টল

**Android-এ Termux।** [Releases](https://github.com/S-I-L-S/marelle/releases) থেকে প্রি-বিল্ট binary ডাউনলোড করুন, অথবা ফোনেই build করুন। ধাপে ধাপে: [docs/TERMUX.md](docs/TERMUX.md)।

**Linux, এবং Rust যেখানেই চলে সেই সব জায়গা।** Rust 1.85 বা আরও নতুন:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

অথবা repository টি clone করে `cargo build --release` চালান। Binary টি হল `target/release/marelle`, প্রায় 4 MiB।

## প্রথম চ্যাট

Key টি এমন একটি ফাইলে রাখুন যা শুধু আপনি পড়তে পারেন, এবং Marelle-কে বলে দিন সেটি কোথায় আছে। Marelle কখনও তার সেটিংসে key সংরক্ষণ করে না।

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

`MODEL` এর জায়গায় আপনার provider-এর list থেকে একটি model-এর নাম বসান। এটি ব্যবহারের অন্য উপায়গুলো:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

উত্তর যায় stdout-এ। Reasoning, token counts, নোট এবং ত্রুটি যায় stderr-এ, তাই `marelle chat ... > answer.txt` শুধু উত্তরটিই সংরক্ষণ করে।

## এজেন্ট tools

Marelle OpenAI-compatible এবং Anthropic APIs-এর মাধ্যমে structured tool definitions পাঠায়। Built-in tool set এটি:

| Tool | উদ্দেশ্য |
| --- | --- |
| `terminal` | Launch directory থেকে shell কমান্ড চালানো |
| `read_file`, `write_file`, `edit_file` | UTF-8 text ফাইল পড়া এবং বদলানো |
| `list_directory`, `search_text` | প্রজেক্ট ফাইল দেখা এবং অনুসন্ধান |
| `fetch_url` | একটি পাবলিক HTTP(S) পেজ text হিসেবে আনা (`curl` প্রয়োজন) |
| `list_skills`, `load_skill` | লোকাল `SKILL.md` নির্দেশনা খুঁজে বের করা এবং লোড করা |

Tool list দেখতে `/tools` এবং available skills-এর তালিকা দেখতে `/skills` ব্যবহার করুন। প্রজেক্ট সংক্রান্ত নির্দেশনা `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` এবং নির্বাচিত `.marelle/` ফাইল থেকে লোড হয়, যদি সেগুলো allowed paths-এর ভেতরে থাকে।

File tools `permissions.allowed_paths`-এর মধ্যে সীমাবদ্ধ (ডিফল্ট: বর্তমান প্রজেক্ট directory) এবং `permissions.denied_paths` মানে। পড়া ডিফল্টভাবে অনুমোদিত; ফাইল পরিবর্তন, shell কমান্ড এবং network fetch চালানোর আগে জিজ্ঞেস করে। Shell permission host কমান্ডের ওপর প্রযোজ্য এবং এটি filesystem sandbox নয়: `ask`-এই রাখুন, যদি না আপনি ইচ্ছা করে model-কে কমান্ড চালানোর আস্থা দেন।

এজেন্ট `agent.max_turns` tool round-trips-এর পর থেমে যায়। Tool responses অবিশ্বস্ত বিষয়বস্তু হিসেবে গণ্য হয়। Requests সম্পূর্ণ model responses ব্যবহার করে, যাতে tool arguments অসম্পূর্ণ stream fragments থেকে না জোটে।

## সেশন

কথোপকথন ডিফল্টভাবে লোকাল ভাবে সংরক্ষিত হয়, tool-call history এবং model/session সেটিংসসহ। Unix-এ session directory-র mode `0700` এবং JSON ফাইলের mode `0600`। ফাইলটি `$XDG_STATE_HOME/marelle/sessions.json`-এ সংরক্ষিত হয়, অথবা `~/.local/state/marelle/sessions.json` যদি `XDG_STATE_HOME` সেট করা না থাকে। API keys কখনও এই ফাইলে লেখা হয় না। persistence বন্ধ করতে:

```sh
marelle config set agent.persist_sessions false
```

Exit codes: `0` সফলতা, `1` provider বা network ত্রুটি, `2` usage বা configuration ত্রুটি।

## Providers

| id | প্রোটোকল | Key ভেরিয়েবল | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM), আন্তর্জাতিক |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-compatible endpoint |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel, চীন |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, চীন |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | এক key-এর পেছনে অনেক model |
| `ollama` | OpenAI-compatible | none | এই মেশিনেই Ollama |

Z.ai এবং Zhipu endpoints তৃতীয় পক্ষের ডকুমেন্টেশন থেকে নেওয়া, vendor-এর নিজস্ব পেজ থেকে নয়। এগুলো আপনার plan-এর সঙ্গে মিলিয়ে দেখুন, এবং কোনো field আলাদা হলে সেটি override করুন।

Provider-এর প্রতিটি field বদলানো যায়, আর নতুন provider মানে শুধু একটি নাম আর একটি address:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

অন্য provider fields: `kind` (`openai-compatible` বা `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` এবং `extra_body`। যদি `api_key_env` দ্বারা নির্দিষ্ট variable সেট করা থাকে, তবে সেটিই প্রাধান্য পায়; নইলে ফাইল পড়া হয়।

## সেটিংস

সেটিংস স্তরে স্তরে আসে: built-in ডিফল্ট, তারপর environment variables, তারপর flags। Setting files পরের ধাপে ডিফল্ট এবং environment-এর মাঝে জায়গা নেবে।

- `MARELLE_CHAT__MODEL=zai/MODEL` দিয়ে `chat.model` সেট হয়। দ্বিগুণ underscore একটি dot-কে বোঝায়।
- `marelle config show [prefix]` সেটিংস এবং প্রতিটি কোন স্তর থেকে এসেছে তা তালিকাভুক্ত করে।
- `marelle config explain <key>` একটি key কোন কোন স্তর থেকে সেট হয়েছে তা দেখায়, সবচেয়ে নিচু থেকে সবচেয়ে উঁচু priority পর্যন্ত।

| Key | ডিফল্ট | অর্থ |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | কার সঙ্গে কথা বলা হবে। `chat.model` `provider/model` আকারে হতে পারে। |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | সেট না থাকলে provider-এর ওপর ছেড়ে দেওয়া হয় |
| `chat.stream` | `true` | ঐচ্ছিক judge response stream করে; tool-সক্ষম agent turns বর্তমানে সম্পূর্ণ responses ব্যবহার করে যাতে tool calls অক্ষত থাকে |
| `chat.show_reasoning`, `chat.show_usage` | `false` | stderr-এ reasoning এবং token counts |
| `agent.max_turns` | `20` | প্রতি user request-এ সর্বোচ্চ tool round-trips |
| `agent.persist_sessions` | `true` | রানের মাঝে লোকাল কথোপকথন সংরক্ষণ |
| `permissions.shell` | `ask` | Shell কমান্ড চালানো: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | ফাইল পড়া: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | ফাইল পরিবর্তন: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL আনা: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | file tools-এর জন্য অনুমোদিত roots |
| `permissions.denied_paths` | none | স্পষ্টভাবে ব্লক করা file paths |
| `network.connect_timeout_secs` | `15` | সংযোগ স্থাপনের জন্য দেওয়া সময় |
| `network.idle_timeout_secs` | `300` | data-র অপেক্ষায় অনুমোদিত নীরবতা |
| `network.proxy` | `system` | `system`, `none`, অথবা একটি proxy URL |
| `network.ca_bundle` | none | বিশ্বস্ত certificate authorities-এর একটি অতিরিক্ত ফাইল |
| `network.allow_insecure_http` | `false` | সাধারণ `http`-এ non-local host-এ key পাঠানোর অনুমতি |
| `update.check` | `true` | interactive session শুরু হলে GitHub এবং GitLab-কে নতুন সংস্করণ জিজ্ঞেস করা |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` যেখান থেকে check নতুন release জিজ্ঞেস করে |
| `update.gitlab_repo` | none | GitLab `owner/repo` যেখান থেকে check এটাও জিজ্ঞেস করে। খালি থাকলে GitLab বন্ধ |
| `update.timeout_secs` | `3` | check একটি source-এ কত সেকেন্ড ব্যয় করতে পারে |

Marelle বিশ্বস্ত certificate authorities-এর নিজস্ব তালিকা সঙ্গে রাখে, তাই system-এ সেগুলো না থাকলেও চলে, যেমন Termux-এ। `SSL_CERT_FILE` আরও যোগ করে।

নতুন সংস্করণ পাওয়া গেলে session একবার সেটার কথা বলে — সংস্করণ, আপনার কাছে কোনটি আছে, এবং সেটি কোথায় পাওয়া যাবে। প্রথমে release জিজ্ঞেস করা হয়; যে repository শুধু তার commit-গুলোতে tag দেয়, সে তার সর্বশেষ tag-এ চলে যায়। কিছু ভুল হলে সেটি নীরব থাকে।

## আপনার keys আপনারই থাকে

- সেটিংস শুধু সেই variable বা ফাইলের নাম নেয় যেখানে key থাকে, কখনও key নিজে নয়। `providers.x.api_key` জাতীয় সেটিং প্রত্যাখ্যাত হয়।
- Key `config show`, `config explain` বা `providers` দিয়ে প্রিন্ট হয় না, এবং server থেকে ফেরা যেকোনো error টেক্সট থেকে সরিয়ে দেওয়া হয়।
- এটি শুধু provider-এর address-এ, একটি header-এ পাঠানো হয়, এবং সাধারণ `http`-এ অন্য কোনো মেশিনে কখনও যায় না, যদি না আপনি অনুমতি দেন।
- Redirect অনুসরণ করা হয় না, তাই key অন্য কোথাও forward হতে পারে না।

## ভাষা

Interface English, Russian, French এবং Ukrainian বলে। `--lang <code>` বা `MARELLE_LANG` দিয়ে বেছে নিন; নইলে `LC_ALL`, `LC_MESSAGES` এবং `LANG` ঠিক করে। প্রতিটি বার্তা একটি catalog-এ থাকে, আর কোনো ভাষা কোনো বার্তা মিস করলে একটি test fail করে। লক্ষ্য হল ইউরেশিয়ার প্রতিটি ভাষা, তারপর আরও।

## রোডম্যাপ

1. **কথা বলা।** OpenAI-compatible এবং Anthropic adapters, SSE streaming এবং complete-response decoding দুটোই সহ। Tool-সক্ষম agent turns বর্তমানে সম্পূর্ণ responses ব্যবহার করে। *সম্পন্ন।*
2. **সেটিংস এবং চেহারা।** TOML setting ফাইল, global এবং প্রতি প্রজেক্টে, `config set`, `get` এবং `unset` সহ এবং প্রতিটি key-এর জন্য একটি বিবরণ। [Hermes Agent](https://github.com/NousResearch/hermes-agent)-এর স্পিরিটে একটি themed interface: banner, তথ্য panel, ফ্রেম করা উত্তর, spinner, ডিফল্টে wisteria বেগুনি, যেকোনো রং HEX হিসেবে সেট করা যায়। *সম্পন্ন।*
3. **অনেক model।** Model aliases, একসঙ্গে কয়েকটি model-কে জিজ্ঞেস করে উত্তরের তুলনা করা, fallback chains, এবং একটি judge model যা সেগুলো মিলিয়ে দেয়। *সম্পন্ন।*
4. **কাজ করা।** OpenAI-compatible এবং Anthropic providers-এর জন্য structured tool calling, লোকাল file/search/terminal tools, URL fetching, লোকাল skills এবং allow/ask/deny permissions, সঙ্গে transcript যা প্রতিটি tool call এবং তার ফলাফল দেখায়। *সম্পন্ন।*
5. **মনে রাখা।** নাম দেওয়া কথোপকথন রানের মাঝে লোকাল ভাবে থেকে যায়। দীর্ঘমেয়াদি semantic memory এখনও সামনের পথে।
6. **বিস্তার।** MCP servers, external plugins, subagents এবং সমৃদ্ধ skills সমর্থন।
7. **পৌঁছানো।** Browser automation, messaging gateways এবং নির্ধারিত কাজ।

ডিজাইনের পেছনের ভাবনা [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-এ আছে।

## পরিচিত সীমাবদ্ধতা

- Provider এবং model selectors কনফিগার করা providers এবং model aliases দেখায়। Marelle এখনও remote model catalogs আবিষ্কার করে না।
- Selectors মাউসের click-to-focus এবং wheel scrolling সমর্থন করে; transcript এবং input editor keyboard-নির্ভরই থাকে।
- Ctrl-C interactive session থেকে বেরিয়ে যায়। এটি এখনও শুধু চলমান response বাতিল করতে পারে না।
- Agent loop, permission-aware লোকাল tools এবং স্থায়ী লোকাল sessions বাস্তবায়িত, তবু Marelle একটি pre-alpha prototype-ই রয়ে গেছে; বর্তমান ডিজাইন এবং সীমানার জন্য [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) দেখুন।

## ডেভেলপমেন্ট

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Test কখনও network স্পর্শ করে না: তারা `crates/marelle-testkit`-এ একটি mock server-এর সঙ্গে কথা বলে। GitHub Actions Android binaries-ও তৈরি করে এবং ঘোষিত সর্বনিম্ন Rust সংস্করণ যাচাই করে। GitLab mirror-এর জন্য একটি GitLab CI pipeline দেওয়া আছে; [docs/GITLAB.md](docs/GITLAB.md) দেখুন।

## লাইসেন্স

[AGPL-3.0-only](LICENSE)। Third-party নোটিশ [NOTICE](NOTICE)-এ আছে। Release archive-গুলোতে binary-র ভেতরের Rust crates-এর জন্য `THIRD-PARTY-LICENSES.txt` থাকে।
