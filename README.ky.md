```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Терминалыңыз үчүн конфигурациялануучу ЖИ агент.**
Өзүңүздүн API ачкычтарыңызды алып келиңиз: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, жергиликтүү Ollama же OpenAI менен үйлөшкөн кайсы бир сервер. Алгач Linux жана Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · Кыргызча
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Алдын-алфа, нуска 0.1.0-alpha.1.** Жүзөгө ашырылды: өзөк TUI интерфейси, бир нече жеткирүүчү менен чат, укуктарды сезген агент цикли, жергиликтүү куралдар жана кайра улантууга болгон жергиликтүү сессиялар. API-лер жана параметрлер али өзгөрүшү мүмкүн. Бул прототип, OpenCode же Hermes менен мүмкүнчүлүктөрү тең эмес.

*Marelle* — балдардын секирилүү оюнунун франсуз аталышы. Ар бир квадрат — бир кадам.

## Орнотуу

**Андроидтеги Termux.** Даяр бинар файлды [чыгарылыштар](https://github.com/S-I-L-S/marelle/releases) барагынан жүктөп алыңыз, же телефонда топтогула. Кадам-кадам менен нускамо: [docs/TERMUX.md](docs/TERMUX.md).

**Linux жана Rust иштеген ар жерде.** Rust 1.85 же жаңысы керек:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Же болбосо репозиторийди клондоңуз да, `cargo build --release` күйгүзүңүз. Бинар файл — `target/release/marelle`, болжол менен 4 МиБ.

## Алгачкы чат

Ачкычты сиз гана окуган файлда сактагыла да, Marelleге кайда турарын айтыңыз. Marelle ачкычты параметрлеринде эч качан сактабайт.

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

`MODEL` ордуна жеткирүүчүңүздүн тизмесинен модель аталышын коюңуз. Пайдалануунун башка жолдору:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Жооб stdout'ка барады. Ой жүгүртүүлөр, токен сандары, эскертүүлөр жана каталар stderr'ге барады; ошентип `marelle chat ... > answer.txt` жообун гана сактайт.

## Агенттин куралдары

Marelle OpenAI менен үйлөшкөн жана Anthropic API аркылуу түзүлүштүү курал аныктамаларын жиберет. Курамына киргизилген курал топтому:

| Курал | Максаты |
| --- | --- |
| `terminal` | Жүгүртүү папкасынан кабык буйругун аткаруу |
| `read_file`, `write_file`, `edit_file` | UTF-8 тексттик файлдарды окуу жана өзгөртүү |
| `list_directory`, `search_text` | Проекттин файлдарын көрүп чыгуу жана издөө |
| `fetch_url` | Жария HTTP(S) барагын текст катары алуу (`curl` керек) |
| `list_skills`, `load_skill` | Жергиликтүү `SKILL.md` көрсөтмөлөрүн таап, жүктөө |

`/tools` буйругу курал тизмесин, `/skills` — бар көндүмдөрдү көрсөтөт. Проекттөө көрсөтмөлөрү `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` жана тандалган `.marelle/` файлдарынан, уруксат берилген жолдордо болсо, жүктөлөт.

Файлдык куралдар `permissions.allowed_paths` менен чектелет (адаттагы: проекеттин азыркы папкасы) жана `permissions.denied_paths`-ти эске алат. Окуу адаттагыдай уруксат берилген; файлды өзгөртүү, кабык буйруктары жана тармактан түшүрүү алды менен сурашат. Кабык укугу чөйрөктөгү буйруктарга таандык жана файлдык система чөйрөсү эмес: модельге буйруктарды аткарууну атайын ишенбесеңиз `ask`-та кармаңыз.

Агент `agent.max_turns` курал айланма-тургандан кийин токтойт. Курал жооптору ишенбеген мазмун саналат. Суроо-талаптар модельдин толук жоопторун колдонот; ошентип курал аргументтери агымдын жарык чүкөлөрүнөн жыйналбайт.

## Сессиялар

Сүйлөшүүлөр адаттагыдай жергиликтүү сакталат; курал чакырыктарынын тарыхы менен, модель жана сессия параметрлери менен берге. Unix-те сессия папкасынын тартиби `0700`, JSON файлдики `0600`. Файл `$XDG_STATE_HOME/marelle/sessions.json` жайында, же `XDG_STATE_HOME` койулбаса `~/.local/state/marelle/sessions.json` жайында сакталат. API ачкычтары бул файлга эч качан жазылат эмес. Сактоону буну менен өчүрүңүз:

```sh
marelle config set agent.persist_sessions false
```

Чыгуу коддору: `0` ийгилик, `1` жеткирүүчү же тармак катасы, `2` колдонуу же конфигурация катасы.

## Жеткирүүчүлөр

| id | Протокол | Ачкыч өзгөрмөсү | |
| --- | --- | --- | --- |
| `zai` | OpenAI менен үйлөшкөн | `ZAI_API_KEY` | Z.ai (Zhipu GLM), эл аралык |
| `zai_coding` | OpenAI менен үйлөшкөн | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic менен үйлөшкөн акыркы чекит |
| `zhipu` | OpenAI менен үйлөшкөн | `ZHIPUAI_API_KEY` | Zhipu BigModel, Кытай |
| `zhipu_coding` | OpenAI менен үйлөшкөн | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Кытай |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI менен үйлөшкөн | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI менен үйлөшкөн | `OPENROUTER_API_KEY` | бир ачкыч артында көп модель |
| `ollama` | OpenAI менен үйлөшкөн | жок | Ошол машинанын Ollamasi |

Z.ai жана Zhipu акыркы чекиттери үчүнчү тарап документациясынан келет, жеткирүүчүнүн өз барагынан эмес. Аларды өз тарифиңиз менен салыштырып көрүңүз, айырмаланса кайсы бир талааны үстүнөн жазыңыз.

Жеткирүүчүнүн ар бир талаасы өзгөртүлөт; жаңы жеткирүүчү — жөн эле аталышы бар дарек:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Жеткирүүчүнүн калган талаалары: `kind` (`openai-compatible` же `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` жана `extra_body`. `api_key_env` атаган өзгөрмө коюлган болсо, ал жеңишке жетет; болбосо файл окулат.

## Параметрлер

Параметрлер катмарлар менен келет: курамына киргизилген адаттагы маанилер, андан кийин чөйрө өзгөрмөлөрү, андан кийин туунуктар. Баптоо файлдары адаттагылар менен чөйрөнүн ортосуна турат.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model` койот. Кош алты сызык чекиттин орнун турат.
- `marelle config show [prefix]` — параметрлерди жана ар биринин кайсы катмардан келгенин тизмектейт.
- `marelle config explain <key>` — ачкычты койгон ар катмарды, эң төмөндөн эң жогорку мурдуздугуна чейин көрсөтөт.

| Ачкыч | Адаттагы | Мааниси |
| --- | --- | --- |
| `chat.provider`, `chat.model` | жок | Ким менен сүйлөшүү. `chat.model` `provider/model` болушу мүмкүн. |
| `chat.system` | жок | Системалык үндөтүү |
| `chat.max_tokens`, `chat.temperature` | жок | Койулган эмес — жеткирүүчүгө калтырылат |
| `chat.stream` | `true` | Бийче маделинин ыктыярдуу жообунун агымы; куралдуу агент кадамдары азыр толук жоопторду колдонот, курал чакырыктары бүтүн калсын дегенде |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Ой жүгүртүүлөр жана токен сандары stderr'де |
| `agent.max_turns` | `20` | Пайдалануучунун бир суроосуна курал айланма-турганын эң көбү |
| `agent.persist_sessions` | `true` | Жергиликтүү сүйлөшүүлөрдү жүгүртүүлөр аралыгында сактоо |
| `permissions.shell` | `ask` | Кабык буйруктарын аткаруу: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Файл окуу: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Файл өзгөртүү: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL түшүрүү: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Файлдык куралдарга уруксаттуу тамырлар |
| `permissions.denied_paths` | жок | Ачык түрдө тосулган файл жолдору |
| `network.connect_timeout_secs` | `15` | Байланышууга берилген убакыт |
| `network.idle_timeout_secs` | `300` | Маалымат күтүүдө уруксаттуу тынчык |
| `network.proxy` | `system` | `system`, `none`, же прокси дареги |
| `network.ca_bundle` | жок | Ишенимдүү күбөлүк берүү мекемелеринин кошумча файлы |
| `network.allow_insecure_http` | `false` | Ачкычтын жынаа `http` аркылуу бөтөн хостко барууга уруксат |
| `update.check` | `true` | Интерактивдүү сессия башталганда GitHub жана GitLab'дан жаңыраак нуска сураш |
| `update.github_repo` | `S-I-L-S/marelle` | Текшерүү эң жаңы чыгарылышты сураган GitHub `owner/repo` |
| `update.gitlab_repo` | жок | Ошол эле текшерүү үчүн GitLab `owner/repo`. Бош болсо GitLab өчүрүлөт |
| `update.timeout_secs` | `3` | Текшерүү бир булакка канча секунд жумшай алат |

Marelle өз ишенимдүү күбөлүк берүү мекемелери тизмесин ала жүрөт; анын үстүнөн системада болбогон жерде да, Termux'тагыдай иштейт. `SSL_CERT_FILE` кошо берет.

Жаңыраак нуска табылса, сессия аны бир жолу эскерет — нускасын, сизде турганын жана кайдан алуу керектигин. Алгач чыгарылыш суралат; комиттерин тек белгилери менен көрсөткөн репозиторий эң жаңы белгини берет. Эмне кагылып кетсе, үнсүз калат.

## Ачкычтарыңыз сиздики болуп калат

- Параметрлер ачкыч жаткан өзгөрмөнү же файлды атайт, эч качан ачкычтын өзүн эмес. `providers.x.api_key` сыяктуу параметр четке кагылат.
- Ачкычты `config show`, `config explain` же `providers` басып чыкпайт, жана ал серверден келген ар кайсы ката текстинен алынып ташталат.
- Ал жеткирүүчүнүн дарегине гана, бир сапта жөнөтүлөт, жана уруксат бербесеңиз эч качан жынаа `http` аркылуу башка машинага барбайт.
- Кайра багыттоолорго ээрчишпейт; ошентип ачкыч башка жакка өткөрүлбөй калат.

## Тилдер

Интерфейс англис, орус, франсуз жана украин тилдеринде сүйлөйт. `--lang <code>` же `MARELLE_LANG` менен тандагыла; болбосо `LC_ALL`, `LC_MESSAGES` жана `LANG` чечет. Ар бир билдирүү каталогта турат, жана бир тил бирин таштап кетсе тест жыгылат. Максат — Евразиянын ар тили, андан соң дагы көп.

## Жол картасы

1. **Сүйлөшүү.** OpenAI менен үйлөшкөн жана Anthropic бейімдегичтер, SSE агымы да, толук жоопторду окуу да болгондо. Куралдуу агент кадамдары азыр толук жоопторду колдонот. *Аткарылды.*
2. **Параметрлер жана көрүнүш.** TOML баптоо файлдары, жалпы жана проекеттик, `config set`, `get` жана `unset` менен, ар ачкычтын сыпаттамасы болгондо. [Hermes Agent](https://github.com/NousResearch/hermes-agent) духундагы темалуу интерфейс: баннер, маалымат тактасы, чөйчөктөгү жооптор, айланма, адаттагы гүлзардагы күлгүн түстө, ар түс HEX катары коюлмалуу. *Аткарылды.*
3. **Көп модель.** Модельдештим аталыштары, бир нече моделге бир убакта сураш жана жоопторду салыштыруу, резерв чынжырлары, жана аларды кошкон бийче мадели. *Аткарылды.*
4. **Аракеттенүү.** OpenAI менен үйлөшкөн жана Anthropic жеткирүүчүлөр үчүн түзүлүштүү курал чакырыктары, жергиликтүү файл/издөө/терминал куралдары, URL түшүрүү, жергиликтүү көндүмдөр жана allow/ask/deny укуктары; транскрипт ар курал чакырыгын жана жыйынтыгын көрсөтөт. *Аткарылды.*
5. **Эстөө.** Аталган сүйлөшүүлөр жүгүртүүлөр аралыгында жергиликтүү туруштуу. Узак мөөнөттүк маанидеги эс жана алда.
6. **Кеңейтүү.** MCP серверлери, тышкы кошумчалар, агент алдындагылар жана көндүмдерге байыраак колдоо.
7. **Жетүү.** Браузер автоматтандыруу, кабарлашуу дарбазалары жана бекитилген тапшырмалар.

Дизайндын артындагы ой [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-де.

## Белгилүү чектөөлөр

- Жеткирүүчү жана модель тандагычтар койулган жеткирүүчүлөрдү жана модель аталыштарын тизмектейт. Marelle али алыстагы модель каталогдарын ачпайт.
- Тандагычтар фокус үчүн чычкан басууну жана дөңгөлөк менен бурууну колдойт; транскрипт жана киргизүү редактору клавиатурадан башкарылат.
- Ctrl-C интерактивдүү сессиядан чыгат. Али учурдагы жоопко гана токтотпойт.
- Агент цикли, укуктарды сезген жергиликтүү куралдар жана туруктуу жергиликтүү сессиялар жүзөгө ашырылды; бирок Marelle алдын-алфа прототип болуп турат. Азыркы долбоор жана чектер үчүн [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) окуңуз.

## Өнүктүрүү

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Сыноолор тармакка эч качан тийишпейт: алар `crates/marelle-testkit`-теги жасалма сервер менен сүйлөшөт. GitHub Actions тагы да Android бинарларын топтойт жана айтылган эң төмөн Rust нускасын текшерет. GitLab күзгүсү үчүн GitLab CI түтүгү бар; караңыз [docs/GITLAB.md](docs/GITLAB.md).

## Лицензия

[AGPL-3.0-only](LICENSE). Үчүнчү тарап билдирүүлөрү [NOTICE](NOTICE)-те. Чыгарылыш архивдери бинар ичиндеги Rust сандыктары үчүн `THIRD-PARTY-LICENSES.txt` камтыйт.
