```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**आपके टर्मिनल के लिए एक कॉन्फ़िगर किया जा सकने वाला AI एजेंट।**
अपनी API keys साथ लाइए: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, एक लोकल Ollama, या कोई भी OpenAI-compatible सर्वर। Linux और Termux सबसे पहले।

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · हिन्दी · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, वर्शन 0.1.0-alpha.1।** नेटिव TUI, multi-provider chat, permission-aware agent loop, लोकल tools और दोबारा जारी रखे जा सकने वाले लोकल sessions लागू हैं। API और सेटिंग्स अभी भी बदल सकती हैं। यह एक prototype है, OpenCode या Hermes के साथ feature parity नहीं।

*Marelle* फ़्रेंच में hopscotch यानी ठप्पा (बच्चों का खेल) कहने का शब्द है। हर वर्ग एक कदम है।

## इंस्टॉल

**Android पर Termux।** [Releases](https://github.com/S-I-L-S/marelle/releases) से पहले से बना (prebuilt) binary डाउनलोड करें, या फ़ोन पर ही build करें। कदम-दर-कदम: [docs/TERMUX.md](docs/TERMUX.md)।

**Linux, और हर वह जगह जहाँ Rust चलता है।** Rust 1.85 या नया:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

या repository को clone करें और `cargo build --release` चलाएँ। Binary `target/release/marelle` होता है, लगभग 4 MiB।

## पहली चैट

Key को ऐसी फ़ाइल में रखें जिसे केवल आप ही पढ़ सकें, और Marelle को बता दें कि वह कहाँ है। Marelle कभी भी key अपनी सेटिंग्स में सेव नहीं करता।

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

`MODEL` को अपने provider की list में से किसी model के नाम से बदलें। इसे इस्तेमाल करने के और तरीके:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

उत्तर stdout पर जाता है। Reasoning, token counts, नोट्स और त्रुटियाँ stderr पर जाती हैं, इसलिए `marelle chat ... > answer.txt` केवल उत्तर सेव करता है।

## एजेंट tools

Marelle OpenAI-compatible और Anthropic APIs के ज़रिए structured tool definitions भेजता है। Built-in tool set यह है:

| Tool | उद्देश्य |
| --- | --- |
| `terminal` | Launch directory से shell कमांड चलाना |
| `read_file`, `write_file`, `edit_file` | UTF-8 text फ़ाइलें पढ़ना और बदलना |
| `list_directory`, `search_text` | प्रोजेक्ट फ़ाइलें देखना और खोजना |
| `fetch_url` | कोई सार्वजनिक HTTP(S) पेज text के रूप में लाना (`curl` ज़रूरी) |
| `list_skills`, `load_skill` | लोकल `SKILL.md` निर्देश खोजना और लोड करना |

Tool list देखने के लिए `/tools` और उपलब्ध skills की list के लिए `/skills` इस्तेमाल करें। प्रोजेक्ट संबंधी निर्देश `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` और चुनी हुई `.marelle/` फ़ाइलों से लोड होते हैं, बशर्ते वे allowed paths के भीतर हों।

File tools `permissions.allowed_paths` तक सीमित हैं (डिफ़ॉल्ट: मौजूदा प्रोजेक्ट directory) और `permissions.denied_paths` का सम्मान करते हैं। पढ़ना डिफ़ॉल्ट रूप से अनुमत है; फ़ाइलों में बदलाव, shell कमांड और network fetch चलाने से पहले पूछते हैं। Shell permission host कमांड पर लागू होती है और यह filesystem sandbox नहीं है: इसे `ask` पर ही रखें, जब तक कि आप जानबूझकर model को कमांड चलाने का भरोसा न दें।

एजेंट `agent.max_turns` tool round-trips के बाद रुक जाता है। Tool responses अविश्वसनीय सामग्री माने जाते हैं। Requests पूर्ण model responses इस्तेमाल करती हैं, ताकि tool arguments अधूरे stream fragments से न जोड़े जाएँ।

## सेशन

बातचीत डिफ़ॉल्ट रूप से लोकल रूप से सेव होती है, जिसमें tool-call history और model/session सेटिंग्स शामिल हैं। Unix पर session directory का mode `0700` होता है और JSON फ़ाइल का mode `0600`। फ़ाइल `$XDG_STATE_HOME/marelle/sessions.json` पर सेव होती है, या `~/.local/state/marelle/sessions.json` जब `XDG_STATE_HOME` सेट न हो। API keys कभी भी इस फ़ाइल में नहीं लिखी जातीं। Persistence बंद करने के लिए:

```sh
marelle config set agent.persist_sessions false
```

Exit codes: `0` सफलता, `1` provider या network त्रुटि, `2` usage या configuration त्रुटि।

## Providers

| id | प्रोटोकॉल | Key वेरिएबल | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM), अंतरराष्ट्रीय |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-compatible endpoint |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel, चीन |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, चीन |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | एक ही key के पीछे कई models |
| `ollama` | OpenAI-compatible | none | इसी मशीन पर Ollama |

Z.ai और Zhipu endpoints तीसरे पक्ष के दस्तावेज़ों से लिए गए हैं, vendor के अपने पेजों से नहीं। इन्हें अपने plan से मिलाकर जाँचें, और अगर कोई field अलग हो तो उसे override कर दें।

Provider का हर field बदला जा सकता है, और नया provider तो बस एक नाम और एक address है:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

दूसरे provider fields: `kind` (`openai-compatible` या `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` और `extra_body`। अगर `api_key_env` द्वारा नामित वेरिएबल सेट है, तो वही चलता है; अन्यथा फ़ाइल पढ़ी जाती है।

## सेटिंग्स

सेटिंग्स परतों में आती हैं: built-in defaults, फिर environment variables, फिर flags। Setting files अगले चरण में defaults और environment के बीच अपनी जगह लेंगी।

- `MARELLE_CHAT__MODEL=zai/MODEL` से `chat.model` सेट होता है। दोहरा underscore एक dot का प्रतिनिधित्व करता है।
- `marelle config show [prefix]` सेटिंग्स और हर एक किस परत से आई, यह दिखाता है।
- `marelle config explain <key>` हर परत दिखाता है जिसने कोई key सेट की, सबसे निचली से सबसे ऊँची priority तक।

| Key | डिफ़ॉल्ट | अर्थ |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | किससे बात करनी है। `chat.model` `provider/model` रूप में हो सकता है। |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | सेट न होने पर provider पर छोड़ दिया जाता है |
| `chat.stream` | `true` | वैकल्पिक judge response को stream करता है; tool-सक्षम agent turns फिलहाल पूर्ण responses इस्तेमाल करते हैं ताकि tool calls अक्षुण्ण रहें |
| `chat.show_reasoning`, `chat.show_usage` | `false` | stderr पर reasoning और token counts |
| `agent.max_turns` | `20` | प्रति user request अधिकतम tool round-trips |
| `agent.persist_sessions` | `true` | रनों के बीच लोकल बातचीत सेव रखना |
| `permissions.shell` | `ask` | Shell कमांड चलाना: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | फ़ाइलें पढ़ना: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | फ़ाइलों में बदलाव: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL लाना: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | file tools के लिए अनुमत roots |
| `permissions.denied_paths` | none | स्पष्ट रूप से ब्लॉक किए गए file paths |
| `network.connect_timeout_secs` | `15` | कनेक्ट होने के लिए दिया गया समय |
| `network.idle_timeout_secs` | `300` | data की प्रतीक्षा में अनुमत खामोशी |
| `network.proxy` | `system` | `system`, `none`, या कोई proxy URL |
| `network.ca_bundle` | none | भरोसेमंद certificate authorities की एक अतिरिक्त फ़ाइल |
| `network.allow_insecure_http` | `false` | सादे `http` पर non-local host को key भेजने की अनुमति |
| `update.check` | `true` | interactive session शुरू होने पर GitHub और GitLab से नया वर्शन पूछना |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` जिससे check नया release पूछता है |
| `update.gitlab_repo` | none | GitLab `owner/repo` जिससे check यह भी पूछता है। खाली होने पर GitLab बंद |
| `update.timeout_secs` | `3` | check एक स्रोत पर कितने सेकंड बिता सकता है |

Marelle भरोसेमंद certificate authorities की अपनी list साथ रखता है, इसलिए वहाँ भी काम करता है जहाँ system में ये नहीं होतीं, जैसे Termux में। `SSL_CERT_FILE` और जोड़ता है।

जब कोई नया वर्शन मिलता है, तो session इसका एक बार ज़िक्र करता है — वर्शन, आपके पास कौन-सा है, और वह कहाँ से मिलेगा। पहले release पूछा जाता है; जो repository केवल अपने commits पर tags लगाती है वह अपने नवीनतम tag पर आ जाती है। जो कुछ भी गड़बड़ हो, चुपचाप रहता है।

## आपकी keys आपकी ही रहती हैं

- सेटिंग्स उस वेरिएबल या फ़ाइल का नाम लेती हैं जिसमें key होती है, कभी key खुद नहीं। `providers.x.api_key` जैसी सेटिंग अस्वीकार कर दी जाती है।
- Key `config show`, `config explain` या `providers` से प्रिंट नहीं होती, और server से लौटने वाले किसी भी error टेक्स्ट से हटा दी जाती है।
- यह केवल provider के address पर, एक header में भेजी जाती है, और सादे `http` पर किसी दूसरी मशीन को कभी नहीं जाती, जब तक आप अनुमति न दें।
- Redirects का पीछा नहीं किया जाता, इसलिए key कहीं और forward नहीं हो सकती।

## भाषाएँ

Interface English, Russian, French और Ukrainian बोलता है। `--lang <code>` या `MARELLE_LANG` से चुनें; अन्यथा `LC_ALL`, `LC_MESSAGES` और `LANG` तय करते हैं। हर संदेश एक catalog में रहता है, और अगर कोई भाषा कोई संदेश छोड़ दे तो एक test फेल हो जाता है। लक्ष्य है यूरेशिया की हर भाषा, और फिर आगे और भी।

## रोडमैप

1. **बात करना।** OpenAI-compatible और Anthropic adapters, SSE streaming और complete-response decoding दोनों के साथ। Tool-सक्षम agent turns फिलहाल पूर्ण responses इस्तेमाल करते हैं। *पूर्ण।*
2. **सेटिंग्स और रूप।** TOML setting फ़ाइलें, global और प्रति प्रोजेक्ट, `config set`, `get` और `unset` के साथ और हर key के लिए एक विवरण। [Hermes Agent](https://github.com/NousResearch/hermes-agent) की भावना में एक themed interface: banner, सूचना panel, फ़्रेम किए हुए उत्तर, spinner, डिफ़ॉल्ट रूप से wisteria बैंगनी में, कोई भी रंग HEX के रूप में सेट किया जा सकता है। *पूर्ण।*
3. **कई models।** Model aliases, एक साथ कई models से पूछना और उत्तरों की तुलना, fallback chains, और एक judge model जो उन्हें मिला देता है। *पूर्ण।*
4. **अमल।** OpenAI-compatible और Anthropic providers के लिए structured tool calling, लोकल file/search/terminal tools, URL fetching, लोकल skills और allow/ask/deny अनुमतियाँ, साथ में transcript जो हर tool call और उसका परिणाम दिखाता है। *पूर्ण।*
5. **याद रखना।** नाम वाली बातचीत रनों के बीच लोकल रूप से बनी रहती है। दीर्घकालिक semantic memory अभी आगे की बात है।
6. **बढ़ाना।** MCP servers, external plugins, subagents और समृद्ध skills समर्थन।
7. **पहुँच।** Browser automation, messaging gateways और शेड्यूल किए गए काम।

डिज़ाइन के पीछे की सोच [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) में है।

## ज्ञात सीमाएँ

- Provider और model selectors सेट किए गए providers और model aliases दिखाते हैं। Marelle अभी remote model catalogs खोज नहीं सकता।
- Selectors माउस के click-to-focus और wheel scrolling को समर्थन देते हैं; transcript और input editor keyboard-संचालित ही रहते हैं।
- Ctrl-C interactive session से बाहर हो जाता है। यह फिलहाल केवल मौजूदा response रद्द नहीं कर सकता।
- Agent loop, permission-aware लोकल tools और स्थायी लोकल sessions लागू हैं, पर Marelle अब भी एक pre-alpha prototype है; मौजूदा डिज़ाइन और सीमाओं के लिए [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) देखें।

## डेवलपमेंट

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Tests network को कभी छूते नहीं: वे `crates/marelle-testkit` में एक mock server से बात करते हैं। GitHub Actions Android binaries भी बनाता है और घोषित न्यूनतम Rust वर्शन जाँचता है। GitLab mirror के लिए एक GitLab CI pipeline उपलब्ध है; [docs/GITLAB.md](docs/GITLAB.md) देखें।

## लाइसेंस

[AGPL-3.0-only](LICENSE)। Third-party सूचनाएँ [NOTICE](NOTICE) में हैं। Release archives में binary के भीतर के Rust crates के लिए `THIRD-PARTY-LICENSES.txt` शामिल है।
