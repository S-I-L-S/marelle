```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Терминалға арналған бапталғыш ЖИ агенті.**
Өз API кілттеріңізді алып келіңіз: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, жергілікті Ollama немесе OpenAI-мен үйлесімді кез келген сервер. Алдымен Linux және Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · Қазақша · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Алдын-алфа, нұсқа 0.1.0-alpha.1.** Іске асырылды: текті TUI интерфейсі, бірнеше жеткізушімен чат, рұқсаттардан хабардар агент циклі, жергілікті құралдар және қайта жалғастырылатын жергілікті сессиялар. API-лер мен параметрлер әлі өзгеруі мүмкін. Бұл прототип, OpenCode немесе Hermes-пен мүмкіндіктері теңдес емес.

*Marelle* — балалардың секірту ойынының француз атауы. Әр шаршы — бір қадам.

## Орнату

**Android-тағы Termux.** Дайын бинар файлды [шығарылымдар](https://github.com/S-I-L-S/marelle/releases) бетінен жүктеп алыңыз, немесе телефонда жинаңыз. Қадам-қадаммен нұсқаулық: [docs/TERMUX.md](docs/TERMUX.md).

**Linux және Rust жүретін әр жерде.** Rust 1.85 немесе жаңасы керек:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Немесе репозиторийді клондаңыз да, `cargo build --release` қосыңыз. Бинар файл — `target/release/marelle`, шамамен 4 МиБ.

## Алғашқы чат

Кілтті тек сіз оқитын файлда сақтаңыз да, Marelle-ге қайда тұрғанын айтыңыз. Marelle кілтті параметрлерінде ешқашан сақтамайды.

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

`MODEL` орнына жеткізушіңіздің тізімінен модель атауын қойыңыз. Пайдаланудың басқа жолдары:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Жауап stdout-қа барады. Ой толғаулар, токен саны, ескертпелер мен қателер stderr-ге барады; солай `marelle chat ... > answer.txt` тек жауапты ғана сақтайды.

## Агент құралдары

Marelle OpenAI-мен үйлесімді және Anthropic API арқылы құрылымдалған құрал анықтамаларын жібереді. Құрамдастырылған құрал жинағы:

| Құрал | Мақсат |
| --- | --- |
| `terminal` | Жегу каталогінен қоршау командасын жүргізу |
| `read_file`, `write_file`, `edit_file` | UTF-8 мәтіндік файлдарды оқу және өзгерту |
| `list_directory`, `search_text` | Жобаның файлдарын қарап шығу және іздеу |
| `fetch_url` | Жария HTTP(S) бетін мәтін ретінде алу (іргелі `curl` қажет) |
| `list_skills`, `load_skill` | Жергілікті `SKILL.md` нұсқауларын тауып, түсіру |

`/tools` командасы құрал тізімін, `/skills` — бар шеберліктерді көрсетеді. Жоба жөнінде көрсетпелер `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` және таңдалған `.marelle/` файлдарынан, рұқсат етілген жолдарданса, жүктеледі.

Файл құралдары `permissions.allowed_paths`-пен шектеледі (бастапқы: жобаның ағымдағы каталогі) және `permissions.denied_paths`-ті ескереді. Оқу бастапқыда рұқсат етілген; файл өзгерту, қоршау командалары және желіден түсіру алдымен сұрайды. Қоршау рұқсаты қожалықтағы командаларға тиесілі және файлдық жүйенің құм жәшігі емес: модельге команданы орындауды білгенбойы сенбесеңіз, `ask`-та ұстаңыз.

Агент `agent.max_turns` құрал турынан кейін тұрады. Құрал жауаптары сенімсіз мазмұн саналады. Сұраныстар модельдің толық жауаптарын қолданады; солай құрал аргументтері ағынның жартылай үзінділерінен жиналмайды.

## Сессиялар

Әңгіме-диалогтер әдеттегідей жергілікті сақталады; құрал шақыру тарихымен, модель мен сессия параметрлерімен бірге. Unix-те сессия каталогінің тәртібі `0700`, JSON файлдікі `0600`. Файл `$XDG_STATE_HOME/marelle/sessions.json` жерінде, ал `XDG_STATE_HOME` қойылмаса `~/.local/state/marelle/sessions.json` жерінде сақталады. API кілттері бұл файлға ешқашан жазылмайды. Сақтауды мынымен өшіріңіз:

```sh
marelle config set agent.persist_sessions false
```

Шығу кодтары: `0` табыс, `1` жеткізуші немесе желі қатесі, `2` пайдалану немесе баптау қатесі.

## Жеткізушілер

| id | Протокол | Кілт айнымалылары | |
| --- | --- | --- | --- |
| `zai` | OpenAI-мен үйлесімді | `ZAI_API_KEY` | Z.ai (Zhipu GLM), халықаралық |
| `zai_coding` | OpenAI-мен үйлесімді | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-пен үйлесімді соңғы нүкте |
| `zhipu` | OpenAI-мен үйлесімді | `ZHIPUAI_API_KEY` | Zhipu BigModel, Қытай |
| `zhipu_coding` | OpenAI-мен үйлесімді | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Қытай |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-мен үйлесімді | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-мен үйлесімді | `OPENROUTER_API_KEY` | бір кілт артында көп модель |
| `ollama` | OpenAI-мен үйлесімді | жоқ | Осы машинаға Ollama |

Z.ai мен Zhipu соңғы нүктелері үшінші жақ құжаттамасынан келеді, жеткізушінің өз беттерінен емес. Оларды өз тарифіңізбен салыстырыңыз да, өзгеше болса кез келген өрісті үстінен жазыңыз.

Жеткізушінің әр өрісі өзгертіледі; жаңа жеткізуші — тек аты бар мекенжай:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Жеткізушінің басқа өрістері: `kind` (`openai-compatible` немесе `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` және `extra_body`. `api_key_env` атаған айнымалы қойылған болса, ол жеңімпаз; басқаша файл оқылады.

## Параметрлер

Параметрлер қабаттармен келеді: құрамдастырылған әдепкі мәндер, кейін орта айнымалылары, кейін жалаушалар. Баптау файлдары әдепкілер мен ортаның арасына тұрады.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model` қояды. Қос асты сызық нүктенің орнына тұрады.
- `marelle config show [prefix]` — параметрлерді және әрқайсысының қай қабаттан келгенін тізеді.
- `marelle config explain <key>` — кілтті қойған әр қабатты, ең төменнен ең жоғары мәртебеге дейін көрсетеді.

| Кілт | Әдепкі | Мағынасы |
| --- | --- | --- |
| `chat.provider`, `chat.model` | жоқ | Кіммен сөйлесу. `chat.model` `provider/model` бола алады. |
| `chat.system` | жоқ | Жүйелік промпт |
| `chat.max_tokens`, `chat.temperature` | жоқ | Қойылмаған — жеткізушіге қалдырылады |
| `chat.stream` | `true` | Билеуші моделдің қалауы бойынша жауабының ағымы; құралды агент қадамдары әзірге толық жауаптарды қолданады, солай құрал шақырулары тұтас қалады |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Ой толғаулар және токен саны stderr-де |
| `agent.max_turns` | `20` | Пайдаланушының бір сұранысына ең көп құрал туры |
| `agent.persist_sessions` | `true` | Жергілікті әңгімелерді жүгірулер арасында сақтау |
| `permissions.shell` | `ask` | Қоршау командаларын орындау: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Файл оқу: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Файл өзгерту: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL түсіру: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Файл құралдарына рұқсат етілген тамырлар |
| `permissions.denied_paths` | жоқ | Тура түрде бұғатталған файл жолдары |
| `network.connect_timeout_secs` | `15` | Қосылуға берілген уақыт |
| `network.idle_timeout_secs` | `300` | Деректер күткенде берілген тыныштық |
| `network.proxy` | `system` | `system`, `none`, немесе прокси мекенжайы |
| `network.ca_bundle` | жоқ | Сенімді сертификаттау мекемелерінің қосымша файлы |
| `network.allow_insecure_http` | `false` | Кілттің жай `http` арқылы бөгде хостқа баруына рұқсат |
| `update.check` | `true` | Интерактивтік сессия басталғанда GitHub мен GitLab-тан жаңалау нұсқа сұрау |
| `update.github_repo` | `S-I-L-S/marelle` | Тексеріс ең соңғы шығарылымды сұрайтын GitHub `owner/repo` |
| `update.gitlab_repo` | жоқ | Сол тексеріс үшін GitLab `owner/repo`. Бос болса GitLab өшіріледі |
| `update.timeout_secs` | `3` | Тексеріс бір көздерге неше секунд жұмсай алады |

Marelle өзінің сенімді сертификаттау мекемелері тізімін ала жүреді; солай болғандағы жүйеде олар болмаса да, Termux-та да істейді. `SSL_CERT_FILE` тағы қосады.

Жаңалау нұсқа табылса, сессия оны бір рет атап өтеді — нұсқаны, сіздегісін және қайдан алу керектігін. Алдымен шығарылым сұралады; комиттерін тек белгілейтін репозиторий ең соңғы белгіні береді. Не сәтсіздік орын алса, үнсіз қалады.

## Кілттеріңіз сіздің қалаңызда

- Параметрлер кілт жатқан айнымалыны немесе файлды атайды, ешқашан кілттің өзін емес. `providers.x.api_key` сияқты параметр қабылданбайды.
- Кілтті `config show`, `config explain` немесе `providers` басып шығармайды, және ол серверден келген әр қате мәтінінен алынады.
- Ол тек жеткізушінің мекенжайына, бір тақырыпта жіберіледі, және рұқсат берсеңіз болмаса ешқашан жай `http` арқылы басқа машинаға бармайды.
- Қайта бағыттауларға еріп жүрмейді; солай кілт басқа жаққа жіберілмейді.

## Тілдер

Интерфейс ағылшын, орыс, француз және украин тілдерінде сөйлейді. `--lang <code>` немесе `MARELLE_LANG` арқылы таңдаңыз; болмаса `LC_ALL`, `LC_MESSAGES` және `LANG` шешеді. Әр хабар каталогта тұрады, және бір тіл біреуін қалдырса сынақ жығылады. Мақсат — Еуразияның әр тілі, сосын тағы.

## Жол картасы

1. **Сөйлеу.** OpenAI-мен үйлесімді және Anthropic бейімдеуіштері, SSE ағынымен де, толық жауаптарды оқумен де. Құралды агент қадамдары әзірге толық жауаптарды қолданады. *Дайын.*
2. **Параметрлер және көрініс.** TOML баптау файлдары, жалпы және жоба бойынша, `config set`, `get` және `unset` әр кілттің сипаттамасымен. [Hermes Agent](https://github.com/NousResearch/hermes-agent) рухында тақырыпталған интерфейс: баннер, мағлұмат тақтасы, жақтаудағы жауаптар, айналма, әдепкі түрде глициния күлгінінде, әр түс HEX ретінде қойылмалы. *Дайын.*
3. **Көп модель.** Модель лақаптары, бірнеше модельге қатар сұрау және жауаптарды салыстыру, резерв тізбектері, және оларды қосылатын билеуші модель. *Дайын.*
4. **Істеу.** OpenAI-мен үйлесімді және Anthropic жеткізушілерге құрылымдалған құрал шақырулары, жергілікті файл/іздеу/терминал құралдары, URL түсіру, жергілікті шеберліктер және allow/ask/deny рұқсаттары; транскрипт әр құрал шақыруын және нәтижесін көрсетеді. *Дайын.*
5. **Есте сақтау.** Аталы әңгіме-диалогтер жүгірулер арасында жергілікті қалады. Ұзақ мерзімдік семантикалық жады әлі алда.
6. **Кеңейту.** MCP серверлері, сыртқы қосылатын модульдер, субагенттер және шеберліктерге байырақ қолдау.
7. **Жету.** Браузер автоматтандыру, хабарландыру қақпалары және жоспарланған істер.

Дизайнның артындағы ой-толғау [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-де.

## Белгілі шектеулер

- Жеткізуші мен модель таңдағыштары қойылған жеткізушілер мен модель лақаптарын тізеді. Marelle әзірге алыстағы модель каталогтерін ашпайды.
- Таңдағыштар фокус үшін тышқан шертуін және дөңгелекпен домалтуды қолдайды; транскрипт пен кіріс редакторы пернетақтадан басқарылады.
- Ctrl-C интерактивтік сессиядан шығады. Әзірге тек ағымдағы жауапты тоқтатпайды.
- Агент циклі, рұқсаттардан хабардар жергілікті құралдар және тұрақты жергілікті сессиялар іске асырылды; бірақ Marelle алдын-алфа прототип болып қалады. Қазіргі жобалау мен шектер үшін [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) оқыңыз.

## Даму

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Сынақтар желіге ешқашан тиіспейді: олар `crates/marelle-testkit`-тегі жасанды сервермен сөйлеседі. GitHub Actions сонымен қатар Android бинарларын жинайды және мәлімделген ең төмен Rust нұсқасын тексереді. GitLab айнасы үшін GitLab CI желісі бар; қараңыз [docs/GITLAB.md](docs/GITLAB.md).

## Лицензия

[AGPL-3.0-only](LICENSE). Үшінші тарап ескертпелері [NOTICE](NOTICE)-де. Шығарылым мұрағаттары бинар ішіндегі Rust қораптары үшін `THIRD-PARTY-LICENSES.txt` қамтиды.
