```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**آپ کے ٹرمینل کے لیے ایک قابلِ ترتیب AI ایجنٹ۔**
اپنی API keys ساتھ لائیں: Z.ai (Zhipu GLM)، Anthropic، OpenAI، OpenRouter، ایک مقامی Ollama، یا کوئی بھی OpenAI-compatible server۔ Linux اور Termux سب سے پہلے۔

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · اردو · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha، ورژن 0.1.0-alpha.1۔** نیٹو TUI، multi-provider chat، permission-aware agent loop، مقامی tools اور دوبارہ شروع کیے جا سکنے والے مقامی sessions نافذ ہیں۔ APIs اور سیٹنگز ابھی بدل سکتی ہیں۔ یہ ایک prototype ہے، OpenCode یا Hermes کے ساتھ feature parity نہیں۔

*Marelle* فرانسیسی میں hopscotch یعنی زمین پر بنے خانوں میں اچھلنے والا بچوں کا کھیل کہلاتا ہے۔ ہر خانہ ایک قدم ہے۔

## انسٹال

**Android پر Termux۔** [Releases](https://github.com/S-I-L-S/marelle/releases) سے پہلے سے تیار شدہ binary ڈاؤن لوڈ کریں، یا فون پر ہی build کریں۔ قدم بہ قدم: [docs/TERMUX.md](docs/TERMUX.md)۔

**Linux، اور ہر وہ جگہ جہاں Rust چلتا ہے۔** Rust 1.85 یا نیا:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

یا repository کو clone کریں اور `cargo build --release` چلائیں۔ Binary `target/release/marelle` ہوتا ہے، تقریباً 4 MiB۔

## پہلی چیٹ

Key ایسی فائل میں رکھیں جو صرف آپ ہی پڑھ سکیں، اور Marelle کو بتا دیں کہ وہ کہاں ہے۔ Marelle کبھی بھی key اپنی سیٹنگز میں محفوظ نہیں کرتا۔

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

`MODEL` کو اپنے provider کی list سے کسی model کے نام سے بدلیں۔ اسے استعمال کرنے کے دوسرے طریقے:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

جواب stdout پر جاتا ہے۔ Reasoning، token counts، نوٹس اور غلطیاں stderr پر جاتی ہیں، اس لیے `marelle chat ... > answer.txt` صرف جواب محفوظ کرتا ہے۔

## ایجنٹ tools

Marelle OpenAI-compatible اور Anthropic APIs کے ذریعے structured tool definitions بھیجتا ہے۔ Built-in tool set یہ ہے:

| Tool | مقصد |
| --- | --- |
| `terminal` | Launch directory سے shell کمانڈ چلانا |
| `read_file`, `write_file`, `edit_file` | UTF-8 text فائلیں پڑھنا اور بدلنا |
| `list_directory`, `search_text` | پراجیکٹ فائلیں دیکھنا اور تلاش کرنا |
| `fetch_url` | کوئی عوامی HTTP(S) صفحہ متن کے طور پر لانا (`curl` درکار) |
| `list_skills`, `load_skill` | مقامی `SKILL.md` ہدایات تلاش کرنا اور لوڈ کرنا |

Tool list دیکھنے کے لیے `/tools` اور دستیاب skills کی فہرست کے لیے `/skills` استعمال کریں۔ پراجیکٹ سے متعلق رہنمائی `AGENTS.md`، `CLAUDE.md`، `SOUL.md`، `.cursorrules` اور منتخب `.marelle/` فائلوں سے لوڈ ہوتی ہے، بشرطیکہ وہ allowed paths کے اندر ہوں۔

File tools `permissions.allowed_paths` تک محدود ہیں (ڈیفالٹ: موجودہ پراجیکٹ directory) اور `permissions.denied_paths` کا احترام کرتے ہیں۔ پڑھنا ڈیفالٹ طور پر اجازت شدہ ہے؛ فائلوں میں تبدیلی، shell کمانڈز اور network fetch چلنے سے پہلے پوچھتے ہیں۔ Shell permission host کمانڈز پر لاگو ہوتی ہے اور یہ filesystem sandbox نہیں ہے: اسے `ask` پر ہی رکھیں، جب تک کہ آپ جان بوجھ کر model کو کمانڈ چلانے کا اعتماد نہ دیں۔

ایجنٹ `agent.max_turns` tool round-trips کے بعد رک جاتا ہے۔ Tool responses غیر قابلِ اعتماد مواد سمجھے جاتے ہیں۔ Requests مکمل model responses استعمال کرتی ہیں تاکہ tool arguments نامکمل stream fragments سے نہ جوڑے جائیں۔

## سیشنز

گفتگو ڈیفالٹ طور پر مقامی طور پر محفوظ ہوتی ہے، بشمول tool-call history اور model/session سیٹنگز۔ Unix پر session directory کا mode `0700` ہوتا ہے اور JSON فائل کا mode `0600`۔ فائل `$XDG_STATE_HOME/marelle/sessions.json` پر محفوظ ہوتی ہے، یا `~/.local/state/marelle/sessions.json` جب `XDG_STATE_HOME` سیٹ نہ ہو۔ API keys کبھی بھی اس فائل میں نہیں لکھی جاتیں۔ Persistence بند کرنے کے لیے:

```sh
marelle config set agent.persist_sessions false
```

Exit codes: `0` کامیابی، `1` provider یا network کی غلطی، `2` usage یا configuration کی غلطی۔

## Providers

| id | پروٹوکول | Key ویری ایبل | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM)، بین الاقوامی |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai، Anthropic-compatible endpoint |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel، چین |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan، چین |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | ایک ہی key کے پیچھے کئی models |
| `ollama` | OpenAI-compatible | none | اسی مشین پر Ollama |

Z.ai اور Zhipu endpoints تیسرے فریق کی دستاویزات سے لیے گئے ہیں، vendor کے اپنے صفحات سے نہیں۔ انہیں اپنے plan سے مل کر جانچیں، اور اگر کوئی field مختلف ہو تو اسے override کر دیں۔

Provider کا ہر field بدلا جا سکتا ہے، اور نیا provider تو محض ایک نام اور ایک address ہے:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

دوسرے provider fields: `kind` (`openai-compatible` یا `anthropic`)، `api_key_file`، `auth` (`bearer`، `x-api-key`، `api-key`، `none`)، `max_tokens_field`، `stream_usage` اور `extra_body`۔ اگر `api_key_env` کے نام والا variable سیٹ ہو تو وہی غالب آتا ہے؛ ورنہ فائل پڑھی جاتی ہے۔

## سیٹنگز

سیٹنگز پرتوں میں آتی ہیں: built-in ڈیفالٹس، پھر environment variables، پھر flags۔ Setting files اگلے مرحلے میں ڈیفالٹس اور environment کے درمیان اپنی جگہ لیں گے۔

- `MARELLE_CHAT__MODEL=zai/MODEL` سے `chat.model` سیٹ ہوتا ہے۔ ڈبل underscore ایک dot کی نمائندگی کرتا ہے۔
- `marelle config show [prefix]` سیٹنگز کی فہرست بناتا ہے اور بتاتا ہے کہ ہر ایک کس پرت سے آئی۔
- `marelle config explain <key>` ہر وہ پرت دکھاتا ہے جس نے کوئی key سیٹ کی، سب سے کم سے سب سے زیادہ priority تک۔

| Key | ڈیفالٹ | مطلب |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | کس سے بات کرنی ہے۔ `chat.model` `provider/model` شکل میں ہو سکتا ہے۔ |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | سیٹ نہ ہوں تو provider پر چھوڑ دیا جاتا ہے |
| `chat.stream` | `true` | اختیاری judge response کو stream کرتا ہے؛ tool-فعال agent turns فی الحال مکمل responses استعمال کرتے ہیں تاکہ tool calls محفوظ رہیں |
| `chat.show_reasoning`, `chat.show_usage` | `false` | stderr پر reasoning اور token counts |
| `agent.max_turns` | `20` | فی user request زیادہ سے زیادہ tool round-trips |
| `agent.persist_sessions` | `true` | رنز کے درمیان مقامی گفتگو محفوظ رکھنا |
| `permissions.shell` | `ask` | Shell کمانڈ چلانا: `allow`، `ask`، `deny` |
| `permissions.files_read` | `allow` | فائلیں پڑھنا: `allow`، `ask`، `deny` |
| `permissions.files_write` | `ask` | فائلوں میں تبدیلی: `allow`، `ask`، `deny` |
| `permissions.network` | `ask` | URL لانا: `allow`، `ask`، `deny` |
| `permissions.allowed_paths` | `["."]` | file tools کے لیے اجازت شدہ roots |
| `permissions.denied_paths` | none | واضح طور پر بلاک شدہ file paths |
| `network.connect_timeout_secs` | `15` | کنکٹ ہونے کے لیے دیا گیا وقت |
| `network.idle_timeout_secs` | `300` | data کا انتظار کرتے ہوئے اجازت شدہ خاموشی |
| `network.proxy` | `system` | `system`، `none`، یا کوئی proxy URL |
| `network.ca_bundle` | none | قابلِ اعتماد certificate authorities کی ایک اضافی فائل |
| `network.allow_insecure_http` | `false` | سادے `http` پر غیر مقامی host کو key بھیجنے کی اجازت |
| `update.check` | `true` | interactive session شروع ہونے پر GitHub اور GitLab سے نیا ورژن پوچھنا |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` جس سے check نیا release پوچھتا ہے |
| `update.gitlab_repo` | none | GitLab `owner/repo` جس سے check یہ بھی پوچھتا ہے۔ خالی ہو تو GitLab بند |
| `update.timeout_secs` | `3` | check ایک ذریعے پر کتنے سیکنڈ گزار سکتا ہے |

Marelle قابلِ اعتماد certificate authorities کی اپنی فہرست ساتھ رکھتا ہے، اس لیے وہاں بھی کام کرتا ہے جہاں system میں یہ نہ ہوں، جیسے Termux میں۔ `SSL_CERT_FILE` مزید شامل کرتا ہے۔

جب کوئی نیا ورژن ملے تو session اس کا ایک بار ذکر کرتا ہے — ورژن، آپ کے پاس کون سا ہے، اور وہ کہاں سے ملے گا۔ پہلے release پوچھا جاتا ہے؛ جو repository صرف اپنے commits پر tags لگاتی ہے وہ اپنے تازہ ترین tag پر آ جاتی ہے۔ جو کچھ غلط ہو، خاموش رہتا ہے۔

## آپ کی keys آپ کی ہی رہتی ہیں

- سیٹنگز صرف اسی variable یا فائل کا نام لیتی ہیں جس میں key ہو، کبھی خود key نہیں۔ `providers.x.api_key` جیسی سیٹنگ مسترد کر دی جاتی ہے۔
- Key `config show`، `config explain` یا `providers` سے پرنٹ نہیں ہوتی، اور server سے آنے والے کسی بھی error متن سے نکال دی جاتی ہے۔
- یہ صرف provider کے address پر، ایک header میں بھیجی جاتی ہے، اور سادے `http` پر کسی دوسری مشین کو کبھی نہیں جاتی، جب تک آپ اجازت نہ دیں۔
- Redirects کی پیروی نہیں کی جاتی، اس لیے key کہیں اور forward نہیں ہو سکتی۔

## زبانیں

Interface English، Russian، French اور Ukrainian بولتا ہے۔ `--lang <code>` یا `MARELLE_LANG` سے منتخب کریں؛ ورنہ `LC_ALL`، `LC_MESSAGES` اور `LANG` فیصلہ کرتے ہیں۔ ہر پیغام ایک catalog میں رہتا ہے، اور اگر کوئی زبان کوئی پیغام چھوڑ دے تو ایک test فیل ہو جاتا ہے۔ مقصد یوریشیا کی ہر زبان ہے، اور پھر مزید۔

## روڈ میپ

1. **بات کرنا۔** OpenAI-compatible اور Anthropic adapters، SSE streaming اور complete-response decoding دونوں کے ساتھ۔ Tool-فعال agent turns فی الحال مکمل responses استعمال کرتے ہیں۔ *مکمل۔*
2. **سیٹنگز اور شکل۔** TOML setting فائلیں، global اور فی پراجیکٹ، `config set`، `get` اور `unset` کے ساتھ اور ہر key کے لیے ایک وضاحت۔ [Hermes Agent](https://github.com/NousResearch/hermes-agent) کے اسلوب سے ملتا جلتا themed interface: banner، معلومات کا panel، فریم شدہ جوابات، spinner، ڈیفالٹ طور پر wisteria جامنی میں، کوئی بھی رنگ HEX کے طور پر سیٹ کیا جا سکتا ہے۔ *مکمل۔*
3. **کئی models۔** Model aliases، ایک ساتھ کئی models سے پوچھنا اور جوابات کا موازنہ، fallback chains، اور ایک judge model جو انہیں یکجا کر دے۔ *مکمل۔*
4. **عمل۔** OpenAI-compatible اور Anthropic providers کے لیے structured tool calling، مقامی file/search/terminal tools، URL fetching، مقامی skills اور allow/ask/deny permissions، ساتھ transcript جو ہر tool call اور اس کا نتیجہ دکھاتا ہے۔ *مکمل۔*
5. **یاد رکھنا۔** نام رکھی گئی گفتگو رنز کے درمیان مقامی طور پر برقرار رہتی ہے۔ طویل مدتی semantic memory ابھی آگے کی بات ہے۔
6. **توسیع۔** MCP servers، external plugins، subagents اور بہتر skills سپورٹ۔
7. **رسائی۔** Browser automation، messaging gateways اور شیڈول کیے گئے کام۔

ڈیزائن کے پیچھے کی سوچ [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) میں ہے۔

## معلوم حدود

- Provider اور model selectors سیٹ کیے گئے providers اور model aliases دکھاتے ہیں۔ Marelle ابھی remote model catalogs تلاش نہیں کر سکتا۔
- Selectors ماؤس کے click-to-focus اور wheel scrolling کی سپورٹ رکھتے ہیں؛ transcript اور input editor keyboard سے چلنے والے ہی رہتے ہیں۔
- Ctrl-C interactive session سے باہر نکال دیتا ہے۔ یہ فی الحال صرف موجودہ response منسوخ نہیں کر سکتا۔
- Agent loop، permission-aware مقامی tools اور دائمی مقامی sessions نافذ ہیں، مگر Marelle اب بھی ایک pre-alpha prototype ہے؛ موجودہ ڈیزائن اور حدود کے لیے [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) دیکھیں۔

## ڈیولپمنٹ

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

ٹیسٹس network کو کبھی چھوتے نہیں: وہ `crates/marelle-testkit` میں ایک mock server سے بات کرتے ہیں۔ GitHub Actions Android binaries بھی بناتا ہے اور اعلان کردہ کم سے کم Rust ورژن جانچتا ہے۔ GitLab mirror کے لیے ایک GitLab CI pipeline فراہم کیا گیا ہے؛ [docs/GITLAB.md](docs/GITLAB.md) دیکھیں۔

## لائسنس

[AGPL-3.0-only](LICENSE)۔ Third-party نوٹسز [NOTICE](NOTICE) میں ہیں۔ Release archives میں binary کے اندر کے Rust crates کے لیے `THIRD-PARTY-LICENSES.txt` شامل ہے۔
