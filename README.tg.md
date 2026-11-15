```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Агенти ИИ-и танзимшаванда барои терминали шумо.**
Калидҳои API-и худро биёред: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, Ollama-и маҳаллӣ ё ҳар сервере, ки бо OpenAI мувофиқ аст. Аввал Linux ва Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
Тоҷикӣ · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Пеш-алфа, версия 0.1.0-alpha.1.** Татбиқ шудааст: интерфейси миллӣ TUI, чат бо чанд таъминкунанда, даври агент, ки ба иҷозатҳо огоҳ аст, асбобҳои маҳаллӣ ва сессияҳои маҳаллии идомапазир. API-ҳо ва танзимотҳо ҳанӯз тағйирпазир. Ин прототип аст, на мувофиқати имкониятҳо бо OpenCode ё Hermes.

*Marelle* — номи фаронсавии бозии ҷаҳондани кӯдакон. Ҳар як мураббаъ — як қадам.

## Насбкунӣ

**Termux дар Android.** Файли бинарии омодаро аз саҳифаи [нашрҳо](https://github.com/S-I-L-S/marelle/releases) бегиред, ё дар телефон ҷамъбиёред. Роҳномаи қадам ба қадам: [docs/TERMUX.md](docs/TERMUX.md).

**Linux ва ҳар ҷое, ки фаъол аст Rust.** Rust 1.85 ё навтар лозим аст:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ё анборро клон карда, `cargo build --release`-ро иҷро кунед. Бинарии файл `target/release/marelle` аст, тақрибан 4 МиБ.

## Аввалин чат

Калидро дар файле нигоҳ доред, ки танҳо шумо мехонед, ва ба Marelle бигӯед, куҷост. Marelle калидро ҳеҷ гоҳ дар танзимоти худ намегузорад.

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

`MODEL`-ро бо номи модели аз рӯйхати таъминкунандаи худ иваз кунед. Усулҳои дигари истифода:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Ҷавоб ба меравад stdout. Мулӯҳазот, шумори токенҳо, ёддоштҳо ва хатоҳо ба меравад stderr; бинобар ин `marelle chat ... > answer.txt` танҳо ҷавобро нигоҳ медорад.

## Асбобҳои агент

Marelle таърифҳои сохторноки асбобҳоро тавассути API-ҳои мувофиқи OpenAI ва Anthropic мепфиристад. Мӯҳтавои ҷузвии асбобҳо:

| Асбоб | Мақсад |
| --- | --- |
| `terminal` | Иҷрои фармони қабаки аз феҳристи иҷро |
| `read_file`, `write_file`, `edit_file` | Хондан ва тағйири файлҳои матнӣ UTF-8 |
| `list_directory`, `search_text` | Муоина ва ҷустҷӯ дар файлҳои лоиҳа |
| `fetch_url` | Гирифтани саҳифаи ҷамъии HTTP(S) ҳамчун матн (лозим `curl`) |
| `list_skills`, `load_skill` | Ёфтан ва боркунии дастурҳои маҳаллӣ `SKILL.md` |

Фармони `/tools` рӯйхати асбобҳоро нишон медиҳад, `/skills` — малакаҳои дастрасро. Ҳидоятҳои лоиҳа аз `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` ва файлҳои интихобии `.marelle/` ҳамчун як ҷаъфарӣ бо китоби мунтахаби матнӣ `SOUL.md` ва дигар файлҳо бор мешаванд, агар дар роҳҳои иҷозатдодашуда бошанд.

Асбобҳои файлӣ маҳдуданд ба `permissions.allowed_paths` (бе интихоб: директории ҷории лоиҳа) ва `permissions.denied_paths`-ро риоя мекунанд. Хондан бе интихоб иҷозат аст; тағйири файлҳо, фармони қабакҳо ва боркуниҳои шабака аввал мепурсанд. Иҷозати қабак ба фармонҳои хост тааллуқ дорад ва як ҳуҷҷатҳои системати файлӣ нест: онро дар `ask` нигоҳ доред, агар ба модели иҷрои фармонҳо тамомо вақт ба таври намаён бовар накунед.

Агент баъди `agent.max_turns` гардҳои асбоб меистад. Ҷавобҳои асбобҳо ҳамчун мазмуни беитимод ҳисоб мешаванд. Дархостҳо ҷавобҳои пурраи моделро истифода мебаранд, бино бар ин аргументҳои асбобҳо аз порчаҳои номукамми ҷараён сохта намешаванд.

## Сессияҳо

Муколамаҳо ба таври пешфарз маҳаллӣ нигоҳ дошта мешаванд, аз ҷумла таърихи даъватҳои асбобҳо ва танзимоти модел ва сессия. Дар Unix директории сессияҳо режим `0700` дорад ва файли JSON `0600`. Файл дар `$XDG_STATE_HOME/marelle/sessions.json` нигоҳ дошта мешавад, ё дар `~/.local/state/marelle/sessions.json`, агар `XDG_STATE_HOME` насб нашуда бошад. Калидҳои API ҳеҷ гоҳ ба ин файл сабт намешаванд. Муҳофизатро ин тавр хомӯш кунед:

```sh
marelle config set agent.persist_sessions false
```

Кодҳои баромад: `0` муваффақият, `1` хатои таъминкунанда ё шабака, `2` хатои истифода ё танзим.

## Таъминкунандагон

| id | Протокол | Тағйирёбандаи калид | |
| --- | --- | --- | --- |
| `zai` | мувофиқи OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), байналмилалӣ |
| `zai_coding` | мувофиқи OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, нуқтаи охирини мувофиқи Anthropic |
| `zhipu` | мувофиқи OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Чин |
| `zhipu_coding` | мувофиқи OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Чин |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | мувофиқи OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | мувофиқи OpenAI | `OPENROUTER_API_KEY` | бисёр моделҳо пас аз як калид |
| `ollama` | мувофиқи OpenAI | нест | Ollama дар ин мошин |

Нуқтаҳои охирини Z.ai ва Zhipu аз ҳуҷҷатҳои шахси сеюм меоянд, на аз саҳифаҳои худи таъминкунанда. Онҳоро бо нақшаатон муқоиса кунед ва агар фарқ кунанд, ҳар ягон майдонро болон нависед.

Ҳар майдони таъминкунанда тағйирёбанда аст, ва таъминкунандаи нав танҳо ном бо суроға:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Майдонҳои дигари таъминкунанда: `kind` (`openai-compatible` ё `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ва `extra_body`. Агар тағйирёбандае, ки бо `api_key_env` гуфта мешавад, дода шуда бошад, он ғолиб меояд; вагарна файл хонда мешавад.

## Танзимотҳо

Танзимотҳо ба табақҳо меоянд: маъноҳои пешфарзи сохтӣ, пас тағйирёбандаҳои муҳит, пас парчамҳо. Файлҳои танзимот байни пешфарзҳо ва муҳит ҷой мегиранд.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model`-ро мегузорад. Дукаратонаи зери хат барои нуқта меистад.
- `marelle config show [prefix]` — танзимотҳо ва табақаро, ки аз куҷо омадаанд, менависад.
- `marelle config explain <key>` — ҳар табақро, ки калидро гузошт, аз амултарин ба олӣ нишон медиҳад.

| Калид | Пешфарз | Маъно |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ҳеҷ | Бо кӣ сӯҳбат. `chat.model` метавонад `provider/model` бошад. |
| `chat.system` | ҳеҷ | Диққати системӣ |
| `chat.max_tokens`, `chat.temperature` | ҳеҷ | Дода нашуда — таъминкунанда мемонад |
| `chat.stream` | `true` | Ҷараёни ҷавоби довари ихтиёрӣ; даврҳои агент бо асбобҳо имрӯз ҷавобҳои пурра истифода мебаранд, то даъватҳои асбобҳо сохта бимонанд |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Мулӯҳазот ва шумори токенҳо дар stderr |
| `agent.max_turns` | `20` | Максимуми даврҳои асбобӣ ба як дархости корбар |
| `agent.persist_sessions` | `true` | Нигоҳ доштани муколамаҳои маҳаллӣ миёни коргузориҳо |
| `permissions.shell` | `ask` | Иҷрои фармонҳои қабак: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Хондани файлҳо: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Тағйири файлҳо: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Боркунии URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Решаҳои иҷозатдодашуда барои асбобҳои файлӣ |
| `permissions.denied_paths` | ҳеҷ | Роҳҳои файлӣ, ки сарохирон баста шудаанд |
| `network.connect_timeout_secs` | `15` | Вақти иҷозатшуда барои пайвастан |
| `network.idle_timeout_secs` | `300` | Сукути иҷозатшуда дар мунтазири маълумот |
| `network.proxy` | `system` | `system`, `none`, ё суроғаи прокси |
| `network.ca_bundle` | ҳеҷ | Файли иловагии марказҳои сертификатии боварӣ |
| `network.allow_insecure_http` | `false` | Иҷозат додани калид дар `http`-и муқаррарӣ ба хостаи аҷнабӣ |
| `update.check` | `true` | Пурсидани GitHub ва GitLab дар бораи версияи навтар дар ибтиёи сессияи интерактивӣ |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, ки санҷиш аз он нашри навтарро мепурсад |
| `update.gitlab_repo` | ҳеҷ | GitLab `owner/repo` барои ҳамин санҷиш. Холӣ GitLab-ро хомӯш мекунад |
| `update.timeout_secs` | `3` | Чанд сония санҷиш метавонад ба як манбаъ сарф кунад |

Marelle рӯйхати худро аз марказҳои сертификатии боварӣ бо худ мебарад; бинобар ин ҳамон ҷое кор мекунад, ки система надорад, масалан дар Termux. `SSL_CERT_FILE` илова мекунад.

Агар версияи навтар ёфт шавад, сессия онро як бор зикр мекунад — версияро, онро, ки шумо доред, ва аз куҷо бигиред. Аввал нашр пурсида мешавад; анборе, ки коммитҳои худро танҳо бо тег менишонад, теги навтарро медиҳад. Ҳар чи ноком шавад, хомӯш мемонад.

## Калидҳои шумо шумо мешаванд

- Танзимотҳо тағйирёбанда ё файлро меноманд, ки калид дар он аст, ҳеҷ гоҳ худи калидро не. Танзимоте чун `providers.x.api_key` рад карда мешавад.
- Калидро `config show`, `config explain` ё `providers` чоп намекунанд, ва он аз ҳар матни хато, ки аз сервер меояд, бурида мешавад.
- Он танҳо ба суроғаи таъминкунанда, дар як сарлавҳа фиристода мешавад, ва ҳеҷ гоҳ бо `http`-и муқаррарӣ ба мошини дигар намебарад, агар иҷозат надиҳед.
- Ба тағйири масир пайравӣ намешавад; бино бар ин калид ҷои дигар фиристода намешавад.

## Забоҳон

Интерфейс бо забонҳои англисӣ, русӣ, фаронсавӣ ва укройнӣ сухан мегӯяд. Бо `--lang <code>` ё `MARELLE_LANG` интихоб кунед; вагарна `LC_ALL`, `LC_MESSAGES` ва `LANG` муайян мекунанд. Ҳар паём дар каталог ҳаст, ва санҷиш меафтад, агар ягон забон як паёмро тарк кунад. Ҳадаф — ҳар забони Ёврозиё ва баъд бештар.

## Нақша

1. **Сӯҳбат.** Мутобиқкунандагони мувофиқи OpenAI ва Anthropic, бо ҷараёни SSE ва хониши ҷавобҳои пурра. Даврҳои агент бо асбобҳо имрӯз ҷавобҳои пурра истифода мебаранд. *Тайёр.*
2. **Танзимот ва намуд.** Файлҳои танзимоти TOML, умумӣ ва барои лоиҳа, бо `config set`, `get` ва `unset` ва тавсифи ҳар калид. Интерфейси мавзуд [Hermes Agent](https://github.com/NousResearch/hermes-agent)-дар руҳ: баннери, панели маълумот, ҷавобҳои чорчӯбанд, спиннер, бе интихоб дар банафши глисиния, ҳар ранг ҳамчун HEX. *Тайёр.*
3. **Бисёр моделҳо.** Тахаллусҳои моделҳо, яквақт пурсидани чанд модел ва муқоисаи ҷавобҳо, занҷирҳои эҳтиётӣ, ва модели довар, ки онҳоро омехта мекунад. *Тайёр.*
4. **Амал кардан.** Даъватҳои сохтории асбобҳо барои таъминкунандагони мувофиқи OpenAI ва Anthropic, асбобҳои маҳаллии файлӣ, ҷустҷӯ ва терминал, боркунии URL, малакаҳои маҳаллӣ ва иҷозатҳои allow/ask/deny; матни ҷараён ҳар даъвати асбоб ва натиҷаашро нишон медиҳад. *Тайёр.*
5. **Ёд доштан.** Муколамаҳои номдор дар миёни коргузориҳо маҳаллӣ мемонанд. Ҳофизаи маъноии дармаддавт ҳанӯз пеш аст.
6. **Густариш.** Серверҳои MCP, васлҳои берунӣ, суб-агентҳо ва дастгирии бойтари малакаҳо.
7. **Расидан.** Автоматизатсияи браузер, дарвозаҳои паёмбарон ва вазифаҳои нақшабандӣ.

Мулӯҳиза дар ақиби тарҳ дар [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Маҳдудиятҳои маълум

- Интихобкунандаҳои таъминкунанда ва моделҳо таъминкунандагони танзимшуда ва тахаллусҳои моделҳоро менависанд. Marelle ҳанӯз каталогҳои дурдасти моделҳоро намеёбад.
- Интихобкунандаҳо клики мушро барои фокус ва чархи сайҷиро дастгирӣ мекунанд; матни ҷараён ва муҳаррири ворид аз клавиатура идора мешаванд.
- Ctrl-C аз сессияи интерактивӣ мебарояд. Ҳанӯз танҳо ҷавоби ҷорӣро қатъ намекунад.
- Даври агент, асбобҳои маҳаллии огоҳ аз иҷозатҳо ва сессияҳои маҳаллии идомашаванда татбиқ шудаанд, лекин Marelle прототипи пеш-алфа мемонад; барои тарҳи ҳолӣ ва ҳудудҳо бихонед [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Рушд

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Санҷишҳо ҳеҷ гоҳ ба шабака даст намезананд: онҳо бо сервери сохтагӣ дар `crates/marelle-testkit` сухан мегӯянд. GitHub Actions инчунин бинариҳои Android-ро месозад ва версияи ақаллии Rust-и маълумшударо месанҷад. Барои оинаи GitLab хати GitLab CI ҳаст; бинед [docs/GITLAB.md](docs/GITLAB.md).

## Лисенсия

[AGPL-3.0-only](LICENSE). Изҳороти шахси сеюм дар [NOTICE](NOTICE). Архиҳои нашр `THIRD-PARTY-LICENSES.txt`-ро барои қуттиҳои Rust дар дохили бинарии файл меоранд.
